---
name: implement-plan
description: Executes an implementation plan of any format while enforcing checkpoints, deterministic verification, and early escalation when code reality contradicts the plan. Use when asked to implement, execute, or follow a plan, spec, or task document.
---

Execute an implementation plan with engineering discipline. Follow planned steps, boundaries, and approval gates, while halting immediately when codebase evidence proves a plan assumption is invalid or unsafe.

## Output Style Requirement

You **MUST** write all checkpoint reports, blocker escalations, and verification summaries 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Step 1: Load Plan, Constraints, and Linked Artifacts

Work with plans of any format (sliced plans, checklists, ADRs, or task documents):
1. Read the plan and any linked workspace artifacts (task descriptions, structure previews, UI mockups, or schemas).
2. Identify the ordered execution steps, forbidden scope boundaries, required verification commands, and human stop gates.

---

## Step 2: Execute & Respect Checkpoints

1. **Stay within scope:** Modify only the files and symbols required for the active step. Do not add unrequested abstractions or refactor adjacent code.
2. **Resolve routine friction autonomously:** Fix normal local syntax, import, or type errors that fit within the planned boundaries without interrupting the user.
3. **Run deterministic tools:** Execute all tests, linters, and typecheckers required for the current step. Report each as `command -> 1-line observed result`.
4. **Stop at gates:** Whenever the plan specifies a checkpoint or approval gate, output a compact status summary (`Completed Step`, `Changed Files`, `Verification Evidence`, `Next Step`) and wait for user approval before continuing.

---

## Step 3: Agility & Blocker Escalation Protocol

Never force a flawed plan through silent hacks, boundary violations, or skipped checks. **Stop immediately and escalate to the user** when any of these 4 conditions occur:

1. **Invalidated Premise:** Codebase or runtime evidence proves a planned API, schema, library behavior, or domain invariant does not exist or works differently.
2. **Constraint Conflict:** Completing the step requires violating a declared architectural rule, editing a forbidden file, or adding an unapproved dependency.
3. **Unforeseen Blast Radius:** A planned change breaks existing callers, exposes a race condition or data-loss risk, or requires out-of-scope changes.
4. **Tooling or Environment Blocker:** A required test, linter, or migration command cannot run (e.g., sandbox restriction, missing service, broken environment) or fails outside the current scope. Never claim a check passed without running it.

### Blocker Report Format

When escalating, stop editing files and output this report:

```markdown
### Plan Blocker / Deviation Report

- **Blocked Step:** <Step or slice name>
- **Observed Evidence:** <Exact `file:line` or command error showing why the plan cannot proceed as written>
- **Impact / Conflict:** <Which plan assumption or constraint is blocked>
- **Proposed Options:**
  1. **(Recommended)** <Minimal plan adjustment and why>
  2. <Alternative approach and trade-off>
```

Wait for user direction before resuming implementation.
