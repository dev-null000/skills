---
name: sliced-execution-plan
description: Decomposes implementation into small, verifiable execution slices with approval gates and lower-tier model delegation markers. Use when planning multi-file features, refactorings, or fixes, or when asked for a sliced execution plan.
---

Break multi-file implementations into small, self-contained slices. Enforce strict scope boundaries, identify tasks suitable for lower-tier models, and pause after each slice for verification and human approval.

Do NOT modify production files until the user approves the sliced execution plan.

## Output Style Requirement

You **MUST** write all slice plans, boundary rules, delegation notes, and checkpoint summaries 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Step 1: Establish Inputs (Standalone or Composed)

Operate seamlessly with or without prior skills:
1. **Composed Mode:** If a requirements summary or a structure preview already exists in the conversation, reuse its boundaries, file paths, and signatures. Do not re-scan inspected files.
2. **Standalone Mode:** If invoked directly from a user prompt without prior artifacts, run a targeted lookup (at most 5 relevant files) to identify target modules, public signatures, and test commands before slicing.

---

## Step 2: Slicing & Delegation Rules

### 1. Bound Slice Size and Order
- **Single-Slice Fast-Track (`Scale S`):** If the task touches `1–2 files` (`<100 LOC`), output a single slice (`Slice 1 of 1`).
- **Multi-Slice Cap (`Scale M/L`):** Split changes across `3+ files` into **2 to 5 sequential slices**. Cap each slice at **at most 4 files** or **`~250 LOC`**.
- **Green-Tree Ordering:** Order slices along dependency boundaries so every completed slice compiles and passes tests:
  1. Domain entities, value objects, port protocols, and pure unit tests.
  2. Application command/query handlers and behavior tests using fakes.
  3. Infrastructure adapters (repositories, gateways, migrations) and integration tests.
  4. API routes, CLI handlers, or UI wiring.

### 2. Lock Scope & Prevent Over-Engineering
- List the exact files and public symbols allowed in each slice.
- State explicit **Forbidden Actions** per slice (e.g., no unapproved base classes, no cross-slice imports, no touching adjacent modules).

### 3. Mark Lower-Tier Model Delegation
Identify bounded tasks inside each slice that can be delegated to a lower-tier model or fast subagent to save tokens:
- **Safe to Delegate to Lower-Tier Model:**
  - Read-only file, symbol, or reference lookups.
  - Mechanical boilerplate, DTOs, fixtures, or SQL/migration scaffolding where signatures are already locked.
  - Running test suites, linters, and typecheckers and extracting failure lines from logs.
- **Keep on Primary Model:**
  - Complex domain logic, state transitions, concurrency control, security guards, and architectural trade-offs.

---

## Step 3: Present the Sliced Execution Plan

Output the plan using the Markdown template below and stop for user approval:

```markdown
### Sliced Execution Plan: <Task Title>

**Baseline & Scope:** `<Total slices>` slices | `<Total estimated files>` files | **Global DoD:** `<test + lint + typecheck commands>`

#### Slice 1: <Slice Title>
- **Objective:** <1-sentence goal>
- **Target Files & Symbols:**
  - `<path/to/file.py>`: `<classes/functions to add or modify>`
- **Forbidden in Slice 1:** <Explicit out-of-scope files or abstractions>
- **Lower-Tier Model Delegation:** <Sub-tasks safe for a cheaper/faster model or subagent, or "None">
- **Slice Verification:** `<exact test/typecheck command for Slice 1> -> expected result`

#### Slice 2: <Slice Title>
- **Objective:** <1-sentence goal>
- **Target Files & Symbols:**
  - `<path/to/file.py>`: `<classes/functions to add or modify>`
- **Forbidden in Slice 2:** <Explicit out-of-scope files or abstractions>
- **Lower-Tier Model Delegation:** <Sub-tasks safe for a cheaper/faster model or subagent, or "None">
- **Slice Verification:** `<exact test/typecheck command for Slice 2> -> expected result`
```

---

## Step 4: Checkpointed Slice Execution Gate

After the user approves the plan, execute **one slice at a time**:
1. Implement **only** the current slice (`Slice N`). Do not write code for `Slice N+1`.
2. Run the **Slice Verification** commands and report the observed terminal output in 1–3 lines.
3. **Stop and wait:** Present a compact checkpoint summary (`Changed Files`, `Verification Result`, `Ready for Slice N+1`) and wait for user approval before starting the next slice.
