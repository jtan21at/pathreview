## Week 7 -- Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/157

**Issue title:** Relevance scorer “partial overlap” test fixture actually has full query overlap

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:** `test_query_with_partial_overlap` in `tests/unit/test_relevance_scorer.py` uses a query whose four tokens all occur in the test chunk. `RelevanceScorer.score()` correctly reports complete keyword overlap as `1.0`, but the test expects a value below `0.9`, so the test fails despite correct production behavior. The fix will adjust only the fixture so it has genuine partial overlap while preserving the test's intent to validate a middle-range relevance score.

**Branch name:** `test/157-relevance-partial-overlap`

**Issue fit and selection reasoning:** This Tier 1 issue has a single, reproducible failing test and a narrowly scoped change in the test fixture. The scorer behavior and expected outcome are clear from the implementation, so no API credentials, database, or cross-module changes are required. The relevant test can be run independently, making it realistic to implement and validate within the module timeline.

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/jtan21at/pathreview/commit/884386a26d10d1017a3c0c6fa64ba0d3d6e3e217

**Reproduction summary:** I ran `.venv/Scripts/python -m pytest tests/unit/test_relevance_scorer.py -q` before changing the fixture. The `test_query_with_partial_overlap` case failed because the query and chunk shared all four tokens, so `RelevanceScorer.score()` correctly returned `1.0` while the test asserted that the score must be below `0.9`.

**PLAN.md link:** https://github.com/jtan21at/pathreview/blob/test/157-relevance-partial-overlap/PLAN.md

**Walkthrough video (recommended):** Not recorded.

**Blockers or open questions:** Docker cannot currently be accessed from this Windows session because access to the Docker named pipe is denied; it is not required for the isolated relevance-scorer reproduction. GitHub CLI is not installed, and the previous push attempt waited for interactive GitHub authentication.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:** All #157 plan steps are complete. I reproduced the failure, confirmed `RelevanceScorer.score()` was correctly computing full overlap, changed the test fixture to two matching and two non-matching terms, and formatted the touched test file with Black. The targeted suite `tests/unit/test_relevance_scorer.py` passes all 19 tests.

**Next steps:** Open a pull request from `test/157-relevance-partial-overlap` to `main`, request peer feedback in the cohort Slack channel, and update Check-in 2 with the submitted PR URL.

**Blockers:** The complete `make check` and `make test-unit` commands are not green because of pre-existing repository-wide failures. The issue-specific test file passes Ruff and Black checks, and `rag/evaluator/relevance_scorer.py` passes Mypy.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/jtan21at/pathreview/pull/1

**Branch:** `test/157-relevance-partial-overlap`

**What you built:** The partial-overlap relevance test now uses the query `Python Django Rust Go` against the existing Django/Python chunk. `RelevanceScorer.score()` therefore receives two overlapping tokens from four query tokens and returns `0.5`, which correctly exercises the test's expected middle range without changing production scoring behavior.

**Tests added or updated:** Updated `tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_partial_overlap` so its fixture has genuine partial keyword overlap. The targeted module passes 19 tests.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

`make lint` reports 182 repository-wide Ruff findings in unrelated files; `make test-unit` reports 52 unrelated failures and 376 passes. The #157 test module passes its targeted run, and the touched test file passes Ruff and Black checks. `mypy rag/evaluator/relevance_scorer.py` also passes.

**Draft PR feedback received from:** none
