# Skill: Verify Before Done (v1 - Proof-Based Completion)

## 0. Objective & Scope
The sole purpose of this skill is to **completely prevent Premature Completion—claiming a task is "done" or "working" based on mental assumptions without actually executing build, compile, or test commands.**

- **Mandatory:** Immediately before finishing any source-code creation, modification, merge, porting, or bug-fix task.
- **Skip Allowed:** Pure Q&A with zero code changes, or edits strictly limited to documentation (`.md`) and comments.

---

## 1. Definitions (No Arbitrary Interpretation)
- **Verified Proof:** The **actual Exit Code (`0`) and raw terminal output** from a build, compile, or test command executed directly via a terminal tool in the current session **strictly AFTER the final code edit**.
- **Premature Completion:** Declaring a task complete by assuming "the syntax looks fine" or "it should work now" without running commands, OR citing **stale pass logs recorded before the latest edit**.
- **Unverified Scope:** Areas that cannot be directly executed or verified in a terminal environment (e.g., GUI rendering, in-game client behavior, elevated UAC Windows Registry writes).

---

## 2. Mandatory Execution Protocol
Before telling the user "Done," "Fixed," or "Completed," you MUST execute these 3 steps in strict order.

### Step 1. Execute Verification After Final Edit
- Run the project's actual verification command in the terminal (e.g., `cmake --build`, `dotnet build`, `gradlew compileJava`, `pytest`, linter/type-checker).
- **Re-Verification Rule:** If you edit even a single character after a passing run, the previous proof is immediately invalidated—you MUST re-run the verification command.

### Step 2. Mandatory Pre-Completion Gate (`[DONE-CHECK]` Block)
Immediately before your final report, output the exact `[DONE-CHECK]` block below:

```text
[DONE-CHECK]
- Command Executed: (Exact raw command run in the terminal)
- Exit Code / Output: (Actual Exit Code and 1–3 lines of raw key output)
- Run After Final Edit: (Yes / No — if "No", re-run immediately before reporting)
- Unverified Scope: (Runtime, UI, or in-game behavior not verifiable via terminal; state "None" if fully verified)
```

### Step 3. Pass-or-Fail Gate
- **Verification Failed (Exit Code ≠ 0 or Errors Present):** NEVER declare completion. Switch immediately to debugging (`root-cause-debugging`) or report the raw failure log honestly.
- **No Runnable Build/Test Tool Available:** Never claim "it works." Explicitly state *"No runnable build/test tool available (static code review only)"* inside `[DONE-CHECK]`.
- **Verification Passed (Exit Code = 0):** Output `[DONE-CHECK]` and report completion, clearly listing any `Unverified Scope` items the user must test manually.

---

## 3. Zero-Tolerance Rules
1. **No Speculative Completion:** Never write "there should be no compile errors" or "this works now" without terminal execution proof.
2. **No Stale Logs:** Never submit a build/test pass log captured before the most recent file edit.
3. **No Fabricated or Hidden Output:** Never invent terminal output or hide failing tests/warnings to claim a pass.
4. **No Conflating Compile with Runtime:** If only compilation succeeded, NEVER claim "the feature works perfectly." Report accurately as "Build passed (runtime unverified)."

---

## 4. Few-Shot Examples (Correct vs. Violation)

### Scenario: Modified Windows Registry logic in a C# optimizer tool

#### ❌ Violation (Strictly Forbidden)
> I added 64-bit registry view handling. I double-checked all syntax and types, so it will now build and work properly.

#### ✅ Correct (Must Follow)
> ```text
> [DONE-CHECK]
> - Command Executed: dotnet build Optimizer.csproj -c Release
> - Exit Code / Output: Exit Code 0 | Build succeeded. 0 Warning(s), 0 Error(s)
> - Run After Final Edit: Yes
> - Unverified Scope: Actual Windows Registry writes under elevated UAC privileges and UI button binding cannot be executed in this terminal.
> ```
> Release build passed. Please run the binary with administrator privileges to verify the UI and actual registry writes.
