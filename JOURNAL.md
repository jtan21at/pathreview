## Week 7 -- Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/157

**Issue title:** Relevance scorer “partial overlap” test fixture actually has full query overlap

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:** `test_query_with_partial_overlap` in `tests/unit/test_relevance_scorer.py` uses a query whose four tokens all occur in the test chunk. `RelevanceScorer.score()` correctly reports complete keyword overlap as `1.0`, but the test expects a value below `0.9`, so the test fails despite correct production behavior. The fix will adjust only the fixture so it has genuine partial overlap while preserving the test's intent to validate a middle-range relevance score.

**Branch name:** `test/157-relevance-partial-overlap`

**Issue fit and selection reasoning:** This Tier 1 issue has a single, reproducible failing test and a narrowly scoped change in the test fixture. The scorer behavior and expected outcome are clear from the implementation, so no API credentials, database, or cross-module changes are required. The relevant test can be run independently, making it realistic to implement and validate within the module timeline.

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
