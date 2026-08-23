# Stage 5: Implement (Execution Log & Audit Trail)

- **Feature:** `<feature-slug>`
- **Date & Time:** `YYYY-MM-DD HH:MM:SS UTC`
- **Cognitive Load / Model Tier:** `LOW / FAST (Atomic Code Execution)`
- **Status:** `[IN_PROGRESS | COMPLETED | FAILED]`
- **Handoff Ready:** `[YES | NO]`

---

## 5.1 Step Execution Log
| Step | Status | Verification Output | Timestamp |
| :--- | :--- | :--- | :--- |
| Step 1 | `SUCCESS` | `Found 0 errors in 15ms` | `10:14:02` |
| Step 2 | `SUCCESS` | `2 tests failed as expected (TDD)` | `10:15:20` |
| Step 3 | `SUCCESS` | `2 tests passed (45ms)` | `10:18:05` |
| Step 4 | `SUCCESS` | `Suite passed. 0 lint errors.` | `10:20:10` |

## 5.2 Verification Summary
- **Linter Status:** `[PASSED | FAILED]`
- **Typecheck / Compiler:** `[PASSED | FAILED]`
- **Automated Tests:** `X passed, 0 failed`
- **Acceptance Criteria Sign-Off:** All criteria from `1-question.md` verified.

## 5.3 Surgical Execution & Code Integrity Audit
- [ ] **Zero Orthogonal Edits:** Modified ONLY the files planned in `4-plan.md`. No adjacent refactoring, style tampering, or re-formatting.
- [ ] **Comments & Docstrings Preserved:** No pre-existing comments or documentation were mutated or deleted.
- **Unrelated Dead Code / Tech Debt Observations:**
  - *(List any unrelated dead code observed during implementation for future cleanup — DO NOT DELETE IT HERE)*

## 5.4 Handoff & Merge Notes
- **Key Changes Summary:**
- **Known Limitations / Next Steps:**

## 5.5 Mandatory Final User Acceptance & Sign-Off
> 🛑 **MANDATORY HARD STOP:** The agent must present the implementation summary, test evidence, surgical changes audit, and verification logs to the user, STOP calling tools, and END ITS TURN.
- [ ] **Final User Acceptance:** `[PENDING | ACCEPTED]`
- **Accepted by:** `<User Name / Handle>`
- **Approval Timestamp:** `YYYY-MM-DD HH:MM:SS UTC`
