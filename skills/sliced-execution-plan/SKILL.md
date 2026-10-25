---
name: sliced-execution-plan
description: Decomposes implementation into small, verifiable execution slices with approval gates and lower-tier model delegation markers. Use when planning multi-file features, refactorings, or fixes, or when asked for a sliced execution plan.
---

Generate a complete, self-contained **Sliced Execution Plan** document. This skill produces a **Plan artifact** (Markdown document), NOT implemented code.

Do NOT modify production code or test files. Any implementer (current agent, fresh agent in a clean session, different harness, or human engineer) can execute the plan.

## Output Style Requirement

You **MUST** write all slice plans, boundary rules, delegation notes, and handoff instructions 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Step 1: Establish Inputs (Standalone or Composed)

Operate seamlessly with or without prior skills:
1. **Composed Mode:** If a Task summary or a structure preview already exists in the conversation or workspace docs, reuse its boundaries, file paths, and signatures. Do not re-scan inspected files.
2. **Standalone Mode:** If invoked directly from a user prompt without prior artifacts, run a targeted lookup (at most 5 relevant files) to identify target modules, public signatures, and test commands before slicing.

---

## Step 2: Slicing & Delegation Rules

### 1. Prioritize Logical Coherence & Human Reviewability
- **Logical Consistency First (Mandatory):** Make every slice a self-contained, testable unit that leaves the repository green. Never split a cohesive change into broken half-steps to satisfy a file or line count.
- **Single-Slice Fast-Track (`Scale S`):** If the task touches `1–2 files` (`<100 LOC`), output a single slice (`Slice 1 of 1`).
- **Reviewability Target (`Scale M/L`):** Aim for **at most 10 substantive files** and **`<= 500 LOC` of production code** per slice. Test code is exempt from the 500-LOC target.
- **Exclude Package Boilerplate:** Do not count empty package markers (`__init__.py`), pure re-export barrels (`index.ts`, `mod.rs`), or auto-generated files toward the file limit.
- **Agile Exception:** If a logically indivisible slice requires `>10 substantive files` or `>500 production LOC`, keep it together and state a 1-sentence justification in the slice plan for user approval.
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

## Step 3: Output the Sliced Execution Plan

Output the complete, self-contained **Plan document** in chat (and offer to save it to `.docs/<task-name>/PLAN.md`):

```markdown
### Sliced Execution Plan: <Task Title>

**Context & Inputs:** Task: `<link or title>` | Slices: `<count>` | Estimated files: `<count>` | **Global DoD:** `<test + lint + typecheck commands>`
**Reference Artifacts (Optional):** <Links to structure previews, UI mockups, task docs, or architectural ADRs if present; omit if none>

#### Slice 1: <Slice Title>
- **Objective:** <1-sentence goal>
- **Target Files & Symbols:**
  - `<path/to/file.py>`: `<classes/functions to add or modify>`
- **Forbidden in Slice 1:** <Explicit out-of-scope files or abstractions>
- **Lower-Tier Model Delegation:** <Sub-tasks safe for a cheaper/faster model or subagent, or "None">
- **Relevant Artifacts:** <Links to specific mockup/schema/preview files for this slice, or "None">
- **Slice Verification:** `<exact test/typecheck command for Slice 1> -> expected result`

#### Slice 2: <Slice Title>
- **Objective:** <1-sentence goal>
- **Target Files & Symbols:**
  - `<path/to/file.py>`: `<classes/functions to add or modify>`
- **Forbidden in Slice 2:** <Explicit out-of-scope files or abstractions>
- **Lower-Tier Model Delegation:** <Sub-tasks safe for a cheaper/faster model or subagent, or "None">
- **Slice Verification:** `<exact test/typecheck command for Slice 2> -> expected result`

---

### Implementation Handoff & Next Steps

This plan is ready for execution by an AI agent or human developer:
- **Option A (Fresh Session — Recommended):** Start a new conversation or harness session with a clean context window:
  > *"Implement [Task](<path/to/TASK.md>) according to [Plan](<path/to/PLAN.md>). Execute Slice 1 first, verify with `<command>`, and pause for approval before Slice 2."*
- **Option B (Current Session):** Confirm approval to begin Slice 1 in this thread.
- **Option C (Human Implementation):** Implement the slices manually following the target symbols and verification commands above.
```

Stop execution after outputting the Plan. Wait for developer confirmation before making any code modifications.
