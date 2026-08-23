# Stage 4: Plan (Atomic Step-by-Step Checklist)

- **Feature:** `<feature-slug>`
- **Date & Time:** `YYYY-MM-DD HH:MM:SS UTC`
- **Cognitive Load / Model Tier:** `HIGH (Deep Reasoning / Extended Thinking)`
- **Status:** `[DRAFT | READY_FOR_EXECUTION]`

---

## 4.1 Step-by-Step Atomic Checklist

- [ ] **Step 1: Declare Contracts & Types**
  - *Action:* Add interface definitions in `src/types/index.ts`.
  - *Verification Command:* `pnpm tsc --noEmit`
- [ ] **Step 2: Test-First Harness (TDD)**
  - *Action:* Write test suite in `tests/feature.test.ts`.
  - *Verification Command:* `pnpm test tests/feature.test.ts` (Expected: Fail / Red)
- [ ] **Step 3: Core Implementation**
  - *Action:* Implement logic in `src/core/feature.ts`.
  - *Verification Command:* `pnpm test tests/feature.test.ts` (Expected: Pass / Green)
- [ ] **Step 4: Integration & Full Suite**
  - *Action:* Wire exports into `src/main.ts`.
  - *Verification Command:* `pnpm test && pnpm lint`

## 4.2 Blast Radius & Forbidden Orthogonal Edits
*Delimit strict boundaries to ensure changes are 100% surgical:*
- **Allowed Target Files:**
  - `[CREATE]` `src/core/feature.ts`
  - `[MODIFY]` `src/types/index.ts`
  - `[CREATE]` `tests/feature.test.ts`
- **Strictly Forbidden Orthogonal Changes:**
  - ❌ Do NOT reformat or re-indent adjacent functions or unrelated files.
  - ❌ Do NOT mutate or remove pre-existing comments or docstrings.
  - ❌ Do NOT delete unrelated dead code (report in summary instead).

## 4.3 Rollback / Checkpoint Strategy
- If Step 3 breaks backward compatibility: revert to checkpoint commit or fallback adapter.

## 4.4 Hard Gate Confirmation
- [ ] **Gate Passed:** Plan verified with atomic steps, blast radius containment, and automated validation commands.

## 4.5 Mandatory User Review & Plan Approval Gate
> 🛑 **MANDATORY HARD STOP:** The agent must present the step-by-step plan, blast radius, and test commands to the user, STOP calling tools, and END ITS TURN. Do not modify source code or proceed to Phase 5 until user approval is confirmed below.
- [ ] **User Approval Confirmed:** `[PENDING | APPROVED]`
- **Approved by:** `<User Name / Handle>`
- **Approval Timestamp:** `YYYY-MM-DD HH:MM:SS UTC`
