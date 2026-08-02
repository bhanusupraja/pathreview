## Project Journal — PathReview

This journal is a friendly, developer-focused running log for local work: quick notes, test runs, debugging insights, and short-term next steps. Keep entries concise and date-stamped.

---

### 2026-08-02 — Initial tidy-up and test-focused fixes

- Summary: Cleaned up unit tests and improved a few detectors and evaluators for more reliable behavior during CI.
- Files touched:
  - `ingestion/embeddings/batch_processor.py` — ensured per-batch embedding handling is robust
  - `safety/bias_detector.py` — expanded detection patterns for dismissive language and demographic assumptions
  - `rag/evaluator/faithfulness_checker.py` — refined claim extraction and support checks
  - `agent/tools/readme_scorer.py` — scoring thresholds reviewed
  - Several unit tests under `tests/unit/` adjusted to use side_effect mocks and clearer assertions

- Test results (local):
  - Ran targeted unit tests; many tests pass locally; a subset require async test plugin (`pytest-asyncio`) to run in this environment.

- Notes & decisions:
  - Keep detector regexes conservative to reduce false positives.
  - Use per-batch embedding mocking in tests via `side_effect=lambda texts: [...]` so tests reflect true batching behavior.

- Next steps:
  1. Install and run full test suite with `pytest-asyncio` to exercise async tests: `python -m pip install pytest-asyncio` and `python -m pytest -q`.
  2. Review any remaining failing tests and open focused PRs with one behavioral change per PR.
  3. Add a short CONTRIBUTING note on how to run unit tests locally on Windows (Git Bash recommended).

---

If you'd like, I can: (a) expand this journal with a template for daily entries, (b) add automatic test-run snippets, or (c) convert it into `docs/DEVELOPER-JOURNAL.md` for team-wide visibility. Which would you prefer?
## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146#

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** Tier 1

**Problem summary:**
The personal info scrubbery in pii_scrubber.py is not redacting phone numbers with a parenthesized beginning format. The scrub is not redacting and the format is not being detected. A successful fix would lead to the parenthesized format of phone numbers being detected and redacted from text. This file is in the safety folder, but I cannot find reference to it outside the test suite. So I assume it is not integrated into the project and not affecting other files. 

**Branch name:** fix/146-pii-scrubber-phone-number

**Setup confirmation:** Yes, app runs locally at localhost:5173.

**Cohort ledger:** Yes, issue was added to cohort ledger.

## "Is this right for me?" Checklist:
### Tier Fit
- Is this an appropriate tier for my experience? Yes, because I have not done open source contributions, and I have fixed similar bugs before.

### Codebase Readiness
- **Relevant function/module found:** pii_scrubber.py
- **Rough implementation plan:**
  1. Run relevant tests to replicate the bug.
  2. Read the file.
  3. Identify the code likely responsible.
  4. Propose the fix.
  5. Test the fix
  6. Run entire test suite. 
  7. Repeat if issue is not reolved.

- **Relevant test files:**
- test_us_phone_number_redaction, test_us_phone_formats, test_detect_phone_pii, test_phone_at_start_of_text in tests/unit/test_pii_scrubber.py

### Scope & Time
- **Others already working on it?** When I first went to claim it no, but now there are many others working on it.
- **Estimated time:** 2-3 hrs
- **Can I finish before the deadline?** Yes, seems like a minor issue. I have solved similar bugs before.
- **Dependencies/blockers:** None

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [link to commit documenting the reproduced issue](https://github.com/a-maryam/pathreview/commit/7ac52cff2e60e3c8cf7efe38fd7847ba22a823be)

**Reproduction summary:**
I reproduced the issue by following the instructions for the bug on github: 

I ran the following script in the project root:
```
# script to reproduce pii bug
import os
import sys

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from pii_scrubber import PIIScrubber
s = PIIScrubber()
print(s.scrub('Call me at (555) 123-4567 or 555-123-4567'))
# observed: 'Call me at (555) 123-4567 or [REDACTED]'
print(s.detect('Call me at (555) 123-4567'))
# observed: []
```
**PLAN.md link:** [PLAN.md](PLAN.md)

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
[Anything you're still uncertain about going into Week 9, or leave blank]