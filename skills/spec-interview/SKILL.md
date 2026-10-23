---
name: spec-interview
description: Classifies task type and scale, checks targeted codebase context, and conducts a bounded technical interview before planning. Use when asked to plan a feature, fix, or refactoring, clarify requirements, or run a spec interview.
---

Before writing an implementation plan, classify the task, run a targeted codebase lookup, resolve high-impact ambiguities through a bounded interview, and output a compact **Task Contract**.

Do NOT modify files during discovery.

## Output Style Requirement

You **MUST** write all questions, multiple-choice options, stated assumptions, and the Task Contract 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Step 1: Targeted Lookup & Task Classification

Do NOT scan the entire repository. Restrict file reads strictly to the requested scope:
1. Read only files or symbols explicitly named in the user prompt.
2. If no files are named, run a targeted search (`grep`/glob) for the domain term and inspect at most **5 directly relevant files** (or delegate to a fast read-only subagent).
3. Inspect `pyproject.toml`, `package.json`, or `Taskfile.yaml` **only** if dependencies or verification commands are relevant.

Classify the request across two axes:
- **Type:** `feat` | `fix` | `refactor` | `perf` | `chore` | `docs` | `test`
- **Scale:**
  - `S (Local)`: 1–2 files (`<100 LOC`), single layer, no schema or public contract change, zero new dependencies.
  - `M (Module)`: 3–6 files (`100–300 LOC`), crosses layers within one module, local contract or additive schema change.
  - `L (Cross-Cutting)`: 7+ files (`>300 LOC`), crosses modules or external services, alters core schemas/auth/state machines, or adds new dependencies/patterns.

**Fast-Track Gate:** If Scale is `S` or the prompt and local code leave zero high-blast-radius ambiguities, **ask zero questions**. Proceed directly to **Step 3**.

---

## Step 2: Bounded High-Impact Interview

Ask questions **only** when a wrong assumption would force multi-file rework, break public contracts/schemas, or violate security/architecture boundaries.

### Anti-Annoyance Rules
- **Never ask what targeted files already show:** Do not ask about existing frameworks, naming rules, or patterns visible in the inspected scope.
- **Never ask low-impact details:** Do not ask about private helpers or local names. Choose a convention-compliant default and record it under **Assumed Defaults**.
- **Respect scale question caps:**
  - `Scale S`: 0 questions (unless prompt contains a direct contradiction).
  - `Scale M`: Max **3 questions** in **1 round**.
  - `Scale L`: Max **4 questions per round**, max **2 rounds**.
- **Format questions in 80% ASD-STE100:** State the codebase fact in 1 sentence (<20 words), ask the direct question, and provide 2–3 imperative options. Place `(Recommended)` first. Use the interactive question tool when available.

### Ambiguity Checklist (Filter by Task Type)
Ask only about unresolved items relevant to the classified **Type**:
1. **Scope & Non-Goals (`feat`, `refactor`):** What exact behavior is in scope, and what adjacent work is excluded?
2. **Reuse & Boundaries (`feat`, `refactor`):** Which existing ports, entities, or services must be reused?
3. **Novelty (`feat`, `perf`, `chore`):** Are new libraries, external APIs, or patterns required that differ from the rest of the project?
4. **State & Compatibility (`feat`, `fix`, `refactor`):** Does this alter schemas, migrations, or break existing callers/data?
5. **Failure, Concurrency & Security (`feat`, `fix`, `perf`):** How must timeouts, retries, duplicate requests, or authorization checks behave?
6. **Definition of Done & Execution Mode (`all`):** Which test/lint commands verify completion, and should execution run in a single pass or checkpointed slices?

---

## Step 3: Output the Task Contract

After resolving questions (or immediately on Fast-Track), output this **Task Contract** (under 80 lines) as the verified input for the planning step (e.g., `folder-structure-preview`):

```markdown
### Task Contract: <Task Title>

- **Classification:** Type: `<feat|fix|refactor|...>` | Scale: `<S|M|L>` (`~<N> files`) | Execution: `<Single-Pass | Sliced>`
- **In-Scope Behavior:** <1–3 bullets>
- **Explicit Non-Goals:** <1–3 bullets>
- **Reuse & Architecture Boundaries:** <Modules/ports to reuse; forbidden cross-imports>
- **Net-New Additions:** <None, or approved new libraries/patterns>
- **State, Failure & Security Invariants:** <Data changes, failure/idempotency rules, auth checks>
- **Assumed Defaults:** <Low-risk decisions inferred from codebase conventions>
- **Definition of Done:** <Exact test, linter, and typecheck commands + required edge cases>
- **Execution Slices (if Scale M/L):** <Ordered list of 2–5 checkpointed slices, max 3–4 files per slice>
```
