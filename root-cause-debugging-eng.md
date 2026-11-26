# Skill: Root-Cause Debugging (v1 - Anti-Regression & Circuit Breaker)

## 0. Objective & Scope
The sole purpose of this skill is to **completely prevent Shotgun Debugging (blindly editing code and breaking working features) and Symptom Masking** when resolving build errors, runtime exceptions, or bugs.

- **Mandatory:** Fixing build/compile errors, runtime crashes/exceptions, failing tests, or behavioral bugs.
- **Skip Allowed:** Implementing new features with no active errors, or trivial typo/formatting fixes.

---

## 1. Definitions (No Arbitrary Interpretation)
- **Verified Error:** The exact, raw error message, compiler output, or stack trace directly observed in terminal output during the current session or pasted verbatim by the user.
- **Symptom Masking:** Hiding an error instead of fixing its root cause—such as swallowing exceptions with empty `try-catch` blocks, adding unjustified `if (x == null) return;` guards, or disabling compiler warnings/checks.
- **Shotgun Debugging:** Editing multiple files, functions, or unrelated logic simultaneously without isolating and verifying a single root-cause hypothesis.

---

## 2. Mandatory Execution Protocol
Before modifying a single line of code to fix a bug or error, you MUST execute these 3 steps in strict order.

### Step 1. Inspect Error & Origin
- Identify the **first (root) error message, file path, and exact line number** in the log. (When cascading errors occur, never fix downstream errors first.)
- Directly read the failing line and its immediate call sites / definitions.

### Step 2. Mandatory Pre-Fix Gate (`[DIAGNOSIS]` Block)
Immediately before editing any code, output the exact `[DIAGNOSIS]` block below:

```text
[DIAGNOSIS]
- Raw Error: (Quote the exact error message and File:Line verbatim)
- Root-Cause Hypothesis (1 ONLY): (State exactly ONE code-grounded hypothesis explaining why this error occurs)
- Minimal Fix Plan: (Exact minimal file and line range to change to test this hypothesis)
- Attempt Count: (e.g., 1/2)
```

### Step 3. 2-Strike Circuit Breaker
- **After Attempt 1 Fails:** If the first edit does not resolve the error, you MUST **revert the failed edit completely** before forming a new hypothesis for Attempt 2. (Never stack a second speculative edit on top of a failed one.)
- **STOP After 2 Consecutive Failures:** If the same error persists after Attempt 2, **STOP EDITING IMMEDIATELY.** Do not attempt a 3rd speculative fix. Summarize both failed hypotheses and the resulting error logs for the user, and wait for direction.

---

## 3. Zero-Tolerance Rules
1. **No Symptom Masking:** Never hide errors using empty `catch` blocks, unjustified null-check early returns, or `@SuppressWarnings` / `#pragma warning(disable)`.
2. **No Shotgun Debugging:** Test **ONLY ONE hypothesis** per attempt. Never touch unrelated code, refactor, or reformat while fixing a bug.
3. **No Stacking Failed Edits:** Never leave an ineffective fix in place while modifying other lines; always revert failed changes first.
4. **No Test/Assertion Bypass:** Never delete, weaken, or comment out failing tests or `assert` statements to force a pass.

---

## 4. Few-Shot Examples (Correct vs. Violation)

### Scenario: `Access Violation` / `NullReference` at `Module.cpp:L142`

#### ❌ Violation (Strictly Forbidden)
> Since the pointer might be null, I added `if (!target) return;` and wrapped the entire `Update()` function in a `try-catch` block just in case.

#### ✅ Correct (Must Follow)
> ```text
> [DIAGNOSIS]
> - Raw Error: Access violation reading location 0x00000000 at Module.cpp:L142
> - Root-Cause Hypothesis (1 ONLY): In Module.cpp:L88 (onDisable), the target pointer is freed, but the event listener is never unregistered, causing onTick() at L142 to dereference the freed pointer on the next frame.
> - Minimal Fix Plan: Add 1 line to unregister the event listener inside Module.cpp:L88 (onDisable).
> - Attempt Count: 1/2
> ```
> Applying the minimal 1-line fix to `Module.cpp:L88` based on this hypothesis.
