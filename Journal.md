
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

**Reproduction commit link:** [link to commit documenting the reproduced issue][https://github.com/ascherj/pathreview/pull/553]

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
## Week 8 — Reproduction & solution planning

**Reproduction commit link:** 

**Reproduction summary:**
Running the four tests named in the issue (`test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`) against the unpatched code confirmed the bug: `scrub()` left `(555) 123-4567` in the output unchanged, and `detect()` returned an empty list for the same string. A fifth pre-existing failure (`test_mixed_pii_and_text`) was also discovered — the `street_address` regex matched `pl` inside `applications` as the street-type abbreviation `Pl`, causing unrelated text to be redacted.

**PLAN.md link:** [PLAN.md](PLAN.md)

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
None — both root causes were clearly identified from the regex and confirmed locally with pytest.


## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All sub-tasks from PLAN.md are complete. The `phone_us` regex was updated to use `(?<!\w)`/`(?!\w)` lookarounds instead of `\b` and the separator class was widened to `[-. ]?`. The pre-existing `street_address` false-positive (short abbreviation `Pl` matching inside `applications`) was also fixed by replacing the greedy `[A-Za-z\s]+` with `(?:[A-Za-z]+\s+)*`. Two minor pre-existing lint warnings in the same file were cleaned up (import sort `I001`, unused loop variable `B007`). All 25 unit tests in `tests/unit/test_pii_scrubber.py` pass.

**Next steps:**
Push branch to fork, open PR, and confirm submission.

**Blockers:**
None.

---

### Check-in 2 (end of week)

**PR link:** [paste your PR link here after opening it]

**Branch:** `fix/146-pii-scrubber-parenthesized-phone`

**What you built:**
The `PIIScrubber` regex for US phone numbers previously failed to match the parenthesized format `(555) 123-4567` because `\b` requires a word character on one side (a space before `(` has none) and `[-.]?` does not allow the space between `)` and the exchange digits. The fix replaces `\b` with `(?<!\w)`/`(?!\w)` lookarounds and widens each separator to `[-. ]?`, making all common US phone formats — dashed, dotted, parenthesized, and space-separated — match consistently in both `scrub()` and `detect()`. A pre-existing street-address false-positive was fixed in the same pass.

**Tests added or updated:**
No new tests were needed — `tests/unit/test_pii_scrubber.py` already contained the four tests named in the issue (`test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`) plus `test_mixed_pii_and_text`. All 25 tests in that file now pass; none were passing before for the parenthesized format.

**Self-review confirmation:** [x] make check passes (safety/ module clean; pre-existing E501s documented)  [x] make test-unit passes (25/25 in test_pii_scrubber.py; 48 pre-existing failures in other modules unchanged)

**Draft PR feedback received from:** none
