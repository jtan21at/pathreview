## Solution plan

**Issue:** [Relevance scorer “partial overlap” test fixture actually has full query overlap](https://github.com/ascherj/pathreview/issues/157)

### Understand
`TestRelevanceScorer.test_query_with_partial_overlap` is intended to verify that `RelevanceScorer.score()` returns a middle-range relevance score when only some query terms occur in a chunk. The fixture currently uses the query `Python Django web framework` and a chunk containing all four tokens, so `RelevanceScorer.score()` correctly calculates `4 / 4 = 1.0`; the test then incorrectly asserts that the score is below `0.9`. The expected behavior is unchanged: a genuinely partial-overlap fixture should produce a score between `0.3` and `0.9`.

### Map
- `tests/unit/test_relevance_scorer.py`
  - `TestRelevanceScorer.test_query_with_partial_overlap` contains the incorrect fixture and middle-range assertion.
- `rag/evaluator/relevance_scorer.py`
  - `RelevanceScorer.score()` tokenizes the query and chunk, then computes `overlap / len(query_tokens)`.
  - `RelevanceScorer._tokenize()` lowercases and splits text into tokens.

### Plan
1. Run `python -m pytest tests/unit/test_relevance_scorer.py -q` to reproduce the failure and confirm the fixture returns `1.0`.
2. Compare the query and chunk tokens with `RelevanceScorer.score()` to verify that the scorer implementation is correct and the failure is isolated to test data.
3. Replace the partial-overlap query with four distinct tokens, two of which occur in the existing chunk, while retaining the middle-range assertion.
4. Re-run the targeted test module and verify the intended case produces a score of `0.5` and the full module passes.
5. Check the final diff to ensure no production scoring logic or unrelated fixtures changed.

### Inputs & outputs
The test passes a query string and a list containing a chunk dictionary with a `text` value to `RelevanceScorer.score()`. The corrected fixture uses `Python Django Rust Go` as input against the existing Django/Python chunk. It should produce a `float` score of `0.5`, satisfying the existing `0.3 < score < 0.9` assertion and accurately representing partial keyword overlap.

### Risks & unknowns
- `RelevanceScorer._tokenize()` uses whitespace splitting only, so changing fixtures to include punctuation could accidentally test tokenization rather than overlap; the replacement query should use simple distinct words.
- A fixture with three of four matching terms would score `0.75` and still pass, but two matching terms make the intended partial-overlap behavior clearer and less sensitive to assertion bounds.
- The repository-wide setup requires Docker-backed services, but this isolated scorer test imports only `pytest` and `structlog`; Docker availability should not be required to validate this issue.

### Edge cases
- A full-overlap fixture should continue to be covered by `test_query_with_perfect_keyword_match`, not by the partial-overlap test.
- Zero-overlap behavior remains covered by `test_query_with_zero_keyword_overlap` and must stay below the middle range.
- Empty queries and chunks return `0.0`; this fixture must remain non-empty so it specifically validates keyword-overlap scoring.
- Token matching is case-insensitive; using ordinary lowercase/uppercase-free terms avoids conflating this case with `test_case_insensitive_matching`.
