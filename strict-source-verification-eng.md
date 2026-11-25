# Skill: Strict Source-Code Verification (v5 - Complete)

## 0. Objective & Scope
The sole purpose of this skill is to **completely eliminate code guessing (hallucination, silent rewrites, unverified dependencies, silent omissions, and lazy truncation)** when merging, modifying, or porting source code.

- **Mandatory:** Merging existing tools/modules, cross-language or cross-framework porting, modifying integrations with external functions/classes/APIs, or migrating bulk data (configs, registry keys, etc.).
- **Skip Allowed:** Trivial typo/comment fixes, or standalone one-off test scripts completely independent of existing codebases.

---

## 1. Definitions (No Arbitrary Interpretation)
Terms in this document MUST be interpreted strictly as defined below:

- **Verified Code:** Code that you have directly read via a file-reading tool in the current session, or exact raw code pasted by the user in chat. (Summaries from prior turns, filenames, function names, or comments alone are NOT Verified Code.)
  - **Standard File (< 500 lines):** Read in its entirety from start to finish.
  - **Large File (≥ 500 lines):** Read the **entire definition of the target symbol(s) + all relevant imports/headers/type definitions**, with exact line numbers recorded.
- **Guessing:** Assuming any parameter order, default value, return type (dict keys, list structure, object attributes), exception type, or external utility/library behavior not explicitly written in Verified Code.
- **Glue Code:** The minimal code strictly required to connect two tools in the same language/framework (imports, data format adapters, call-site wiring, naming conflict resolution).
- **Silent Rewrite:** Modifying, simplifying, "improving," or swapping internal algorithms, variable names, error handling, logging, or libraries of an existing tool without being explicitly asked.
- **Silent Omission:** Migrating only a subset of repetitive items (e.g., 25 out of 300 registry keys) and declaring the task "complete" without leaving omission comments (`...`).
- **Verified Dependency:** A library explicitly confirmed to exist in the target project's build/package configuration files (`CMakeLists.txt`, `.vcxproj`, `vcpkg.json`, `package.json`, `requirements.txt`, `.csproj`, `build.gradle`, etc.), or the target language's standard library.

---

## 2. Mandatory Execution Protocol
For any Mandatory task, you MUST execute these 3 steps in strict order before writing or editing a single line of code.

### Step 1. Read Code, State & Dependencies
- **Standard File (< 500 lines):** Read the entire file.
- **Large File (≥ 500 lines):** Read the target symbol definitions and top-level headers/imports/types. **If reading or modifying member variables or shared state, you MUST use a search tool (e.g., grep) to inspect EVERY reference to that variable across the file (including init, reset/teardown, and event loops).**
- **External Dependencies & Frameworks:** List all external utilities and framework APIs called by the source module (e.g., `EventBus`, `RotationUtils`, mapping APIs) and verify whether they exist in the target project.
- **Cross-Language Tasks:** Read the target project's build/package configuration files to establish Verified Dependencies.

### Step 2. Mandatory Pre-Code Gate (`[VERIFICATION]` Block)
Immediately before writing any code, output the exact `[VERIFICATION]` block below. Every entry MUST be grounded strictly in Verified Code and Verified Dependencies.

```text
[VERIFICATION]
- Verified Files: (File paths. For partial reads of large files, specify exact line ranges, e.g., Module.cpp:L1-L60, L820-L940)
- Target Symbols: (File:Function/Class | Exact parameter signature | Exact return type/keys)
- Shared State Audit (Required for large files, else "N/A"): (Modified member/global vars -> All references & lifecycle/init locations verified)
- External Dependency / Framework Boundary (Else "N/A"): (External utils/APIs called by source -> Existence in target & resolution plan)
- Language / Dependency Mapping (Required for cross-language, else "N/A"): (Source API -> Verified Target API & OS/language semantic caveats)
- Bulk Data Count Check (Required for ≥ 30 repetitive items, else "N/A"): (Total source count / Current batch range & cumulative count, e.g., 1-50 of 300)
- Edit Scope: (Exact file paths and locations for glue code or ported code)
- Missing / Ambiguous Items: (Unverified files/utilities, undecided integration method, missing libraries. MUST state "None" if everything is verified)
```

### Step 3. Stop-or-Go Gate
- **If `Missing / Ambiguous Items` is NOT "None":** STOP IMMEDIATELY. Do not write a single line of code. Ask the user for the exact missing files or decisions needed.
- **ONLY if `Missing / Ambiguous Items` is "None":** Proceed to write code strictly within the symbols, dependencies, and Edit Scope declared in Step 2.

---

## 3. Zero-Tolerance Rules
1. **No Omission Comments:** Never write `// ... existing code ...`, `# rest unchanged`, `...`, or similar placeholders when outputting or writing code.
2. **No Silent Omissions:** Never silently cherry-pick a subset of repetitive data or branching logic and claim the task is complete.
3. **No Full Overwrites on Large Files:** For files ≥ 500 lines, NEVER rewrite the entire file. Use **targeted partial edits (Diff / Search-and-Replace)** only.
4. **No Fake Implementations:** Never fill gaps with `TODO`, `pass`, `return {}`, or dummy/mock data when information is missing.
5. **No API Fabrication:** Never invent methods, flags, or config keys that do not exist in Verified Code or Verified Dependencies.
6. **No Silent Rewrites:** When merging within the same language/framework, preserve original internal logic and libraries verbatim; write only Glue Code.

---

## 4. Special Case Rules

### 4.1 Cross-Language Merging & Porting
1. **Confirm Integration Method First:** If not specified by the user, ask whether to **directly port (re-implement logic)** or **integrate externally (IPC, CLI, FFI, DLL)** before writing code.
2. **Verified Dependency Mapping:** Use ONLY **Verified Dependencies** and the target's standard library. Never `#include` or import unverified third-party libraries (e.g., `Boost`, `nlohmann/json`, `libcurl`). If a replacement is needed, stop and present options to the user.
3. **1:1 Behavioral Fidelity:** Adapt syntax to the target language, but NEVER simplify or drop original branching conditions, error handling, edge cases, math/bitwise operations, or data formats.
4. **Semantic & OS Mismatch Report:** Explicitly note behavioral differences in `[VERIFICATION]`—such as memory lifetime (GC vs. RAII), string encoding (UTF-16 vs. UTF-8), OS registry views (e.g., C++ `KEY_WOW64_64KEY` vs. C# `RegistryView.Registry64`), or UAC/privilege handling.

### 4.2 External Dependency & Framework Boundary
- Even within the same language, if frameworks, mappings, or event buses differ (e.g., `@EventTarget` vs. `@SubscribeEvent`, MCP vs. Yarn), or the source module calls external utilities (`RotationUtils`, `PacketUtils`, etc.) missing from the target project:
  - Do NOT guess or fabricate the missing utility's implementation.
  - List all missing items under `External Dependency / Framework Boundary` in `[VERIFICATION]`, then stop and ask the user whether to **(1) port the utility alongside the module** or **(2) map it to a specific existing API in the target project**.

### 4.3 Bulk Data Chunking & Count Check
- When porting or converting **30 or more repetitive items** (config entries, registry keys, packet tables, etc.), enforce the following to prevent output-token truncation:
  1. **Batch Processing:** Split work into **batches of 30–50 items** instead of attempting everything in one response.
  2. **Explicit Count Reconciliation:** After each batch, report `Cumulative Completed / Total Source Count` (e.g., `50 / 300 completed; next batch: 51–100`). Do NOT declare full completion until the count matches 100%.

---

## 5. Few-Shot Examples (Correct vs. Violation)

### Example A (External Dependency Boundary): Porting a 1,500-line module that calls `RotationUtils.getYaw()`, which does not exist in the target project
- ❌ **Violation:** Guessing standard yaw calculation math and writing a fake `getYaw()` without reading `RotationUtils`.
- ✅ **Correct:** Recording `RotationUtils.getYaw() missing in target` under `External Dependency / Framework Boundary` in `[VERIFICATION]`, stopping, and asking whether to port `RotationUtils` or substitute an existing target API.

### Example B (Bulk Data Porting): Porting 300 C++ Windows Registry optimizations to C#
- ❌ **Violation:** Porting 25 representative keys and reporting "All 300 registry optimizations have been ported."
- ✅ **Correct:** Recording `Batch 1–50 of 300` and `C++ KEY_WOW64_64KEY -> C# RegistryView.Registry64` in `[VERIFICATION]`, migrating in 50-item batches, and reporting `50 / 300 completed` until all 300 are verified.
