---
name: task-interview
description: Classifies task type and scale, checks targeted codebase context, and conducts an adaptive technical interview to define a complete, self-contained task before planning. Use when asked to plan a feature, fix, or refactoring, clarify requirements, define a task, or run a task interview.
---

Before writing an implementation plan, classify the task, run a targeted codebase lookup, resolve ambiguities through an adaptive interview, and output a complete, self-contained **Task**.

Do NOT modify production files during discovery.

## Output Style Requirement

You **MUST** write all questions, multiple-choice options, stated assumptions, task summary, and the final Task 80% of the way to ASD-STE100:
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
4. Inspect user-provided context if present (idea notes, images, logs, stack traces, snippets, or URLs). Extract existing constraints before asking questions.

Classify the request across two axes:
- **Type:** `feat` | `fix` | `refactor` | `perf` | `chore` | `docs` | `test`
- **Scale:**
  - `S (Local)`: 1–2 files (`<100 LOC`), single layer, no schema or public contract change, zero new dependencies.
  - `M (Module)`: 3–6 files (`100–300 LOC`), crosses layers within one module, local contract or additive schema change.
  - `L (Cross-Cutting)`: 7+ files (`>300 LOC`), crosses modules or external services, alters core schemas/auth/state machines, or adds new dependencies/patterns.

**Fast-Track Gate:** If Scale is `S` or the prompt and local code leave zero high-blast-radius ambiguities, **ask zero questions**. Proceed directly to **Step 3**.

---

## Step 2: Adaptive High-Impact Interview

Ask questions to eliminate unclarity, hidden assumptions, and architectural ambiguity.

### Anti-Annoyance Rules
- **Never ask what targeted files already show:** Do not ask about existing frameworks, naming rules, or patterns visible in the inspected scope.
- **Never ask low-impact details:** Do not ask about private helpers or local names. Choose a convention-compliant default and record it under **Assumed Defaults**.
- **Interview rounds and question quotas:**
  - `Scale S`: Baseline up to **3 rounds** $\times$ max **4 questions per round** (0 questions if zero ambiguities remain).
  - `Scale M`: Baseline up to **3 rounds** $\times$ max **6 questions per round**.
  - `Scale L`: Baseline up to **3 rounds** $\times$ max **10 questions per round**.
- **Adaptive continuation:** If critical ambiguities remain after baseline rounds, assess the remaining gaps and continue with another round.
- **Early completion:** Stop asking questions immediately when all high-impact ambiguities are resolved, even if rounds or question quotas remain.
- **Format questions in 80% ASD-STE100:** State the codebase fact in 1 sentence (<20 words), ask the direct question, and provide 2–3 imperative options. Place `(Recommended)` first. Use the interactive question tool when available.

### Ambiguity Checklist (Filter by Task Type)
Ask only about unresolved items relevant to the classified **Type**:
1. **Scope & Non-Goals (`feat`, `refactor`):** What exact behavior is in scope, and what adjacent work is excluded?
2. **User Interaction & UI Surface (`feat`):** If the change affects user workflows, what screens, controls, status indicators, or data views are added or changed?
3. **Reuse & Boundaries (`feat`, `refactor`):** Which existing ports, entities, or services must be reused?
4. **Novelty (`feat`, `perf`, `chore`):** Are new libraries, external APIs, or patterns required that differ from the rest of the project?
5. **State & Compatibility (`feat`, `fix`, `refactor`):** Does this alter schemas, migrations, or break existing callers/data?
6. **Failure, Concurrency & Security (`feat`, `fix`, `perf`):** How must timeouts, retries, duplicate requests, or authorization checks behave?
7. **Definition of Done & Execution Mode (`all`):** Which test/lint commands verify completion, and should execution run in a single pass or checkpointed slices?

---

## Step 3: Output the Task

The Task **MUST be self-contained, context-independent, and descriptive enough to act as direct input for planning**. A developer, reviewer, or downstream planning agent must understand the complete technical scope, domain rules, system interactions, and invariants directly from the Task without reading prior chat messages or external context. There is no arbitrary line limit; make it as thorough as required.

Output this **Task** in chat:

```markdown
### Task: <Task Title>

- **Task Summary:** <2–4 sentences describing the feature, what problem it solves, how it works end-to-end, and the collaborating systems/services involved>
- **Classification:** Type: `<feat|fix|refactor|...>` | Scale: `<S|M|L>` (`~<N> files`) | Execution: `<Single-Pass | Sliced>`
- **In-Scope Behavior:** <1–3 bullets>
- **Explicit Non-Goals:** <1–3 bullets>
- **Reuse & Architecture Boundaries:** <Modules/ports to reuse; forbidden cross-imports>
- **Net-New Additions:** <None, or approved new libraries/patterns>
- **State, Failure & Security Invariants:** <Data changes, failure/idempotency rules, auth checks>
- **Assumed Defaults:** <Low-risk decisions inferred from codebase conventions>
- **Reference Context (Optional):** <Repository file paths, public URLs, or brief inline notes if provided; omit if none>
- **Definition of Done:** <Exact test, linter, and typecheck commands + required edge cases>
- **Execution Slices (if Scale M/L):** <Ordered list of 2–5 checkpointed slices, max 3–4 files per slice>
- **Attached Artifacts:** <Links to attached structure previews, UI mockups, or task files if created, or "None">
```

---

## Step 4: Propose Relevant Follow-Up Artifacts

After presenting the Task, verify the context and offer to generate only relevant companion artifacts:
1. **Filter by relevance:**
   - Offer a **structure preview** only if new modules, public ports, or cross-layer refactorings are planned. Do not offer if folder structure is already established or obvious.
   - Offer a **UI mockup** whenever user-facing views, forms, or data displays (e.g., metrics, tables, run statuses) are added or modified.
   - Offer to **save the Task to a file** (e.g., `.docs/<task-name>/TASK.md`) for persistent team reference.
2. **Ask one combined question:** Present a single interactive choice offering only the relevant artifact options for the task, plus an option to proceed directly without extra artifacts.
3. **User declination is standard:** The user can refuse any or all offered artifacts. Treat a refusal as normal workflow. Do not treat it as an error or block progress. Proceed directly to the next phase requested by the user.
4. **Attach links:** When companion artifacts are generated, append their relative file links directly into the **Attached Artifacts** section of the Task.
