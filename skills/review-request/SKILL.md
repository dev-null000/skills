---
name: review-request
description: Prepares a concise, evidence-based implementation handoff and orchestrates a dual-agent code review (specification compliance + adversarial critique). Use when asked for a review request, review handoff, handoff for review, or multi-agent code review.
---

This skill defines a two-part workflow:
1. **Review Handoff:** A concise, evidence-based summary of an implemented change, paired with a compact prompt for an external or fresh reviewer agent.
2. **Dual-Agent Review Protocol:** Instructions for an orchestrating reviewer agent to run two independent read-only reviewers (Compliance + Adversarial) and synthesize their findings.

When invoked after implementing a change, output the **Review Handoff**. When invoked to *perform* a review on a handoff or diff, execute the **Dual-Agent Review Protocol** directly.

---

## Part 1: Review Handoff Structure

### Brevity & Deduplication Rules
- **Link, do not restate:** Cite plan files, specs, ADRs, `AGENTS.md`, and active skills by file path. Do **not** copy-paste architecture rules, naming conventions, or specifications that already exist in referenced files; state inline only requirements that are not recorded on disk.
- **State each fact once:** Never repeat the same limitation, test caveat, or safety rule across multiple sections.
- **No prose essays or wrapper scripts:** Use single-sentence bullets and compact tables. Never paste multi-line shell/Python wrapper scripts in the verification section.

### Required Sections

1. **Goal & Implementation Summary** *(3–5 bullets max)*: The original objective and a high-level summary of what was implemented.
2. **Context & Reference Links:** Repository path, base commit / diff scope (`git diff` command), and paths to the approved plan, `AGENTS.md`, and active skills (plus brief bullets *only* for unwritten task-specific constraints).
3. **Changed Files & Components:** Compact table or bullet list with a 1-line summary per entry and total diff stat. For large changes (>15 files), group by module or layer into at most 10–15 rows.
4. **Verification Evidence:** Compact list of `command -> 1-line observed result` for tests, linters, and type checks. Every completion claim must map to a check; summarize one-off test commands in one line rather than pasting inline scripts.
5. **Resolved Issues & Key Decisions** *(max 5 single-sentence bullets)*: Non-obvious edge cases or design trade-offs resolved during implementation.
6. **Unverified Areas** *(max 5 single-sentence bullets)*: Anything not verified automatically (e.g., production DB engine, live browser, type checker) and why.
7. **Remaining Risks & Next Steps** *(max 5 single-sentence bullets)*: Operational/migration risks or follow-up tasks (do not repeat Section 6 items).
8. **Reviewer Prompt:** Append a short copy-pasteable block following this template (keep under 15 lines):

```markdown
### Reviewer Instructions
Review the changes in `<repo-path>` (scope: `<diff-command-or-paths>`, base: `<commit>`) against the handoff above and `<paths-to-plan-and-AGENTS.md>`. Do NOT modify files, stage/commit, or touch shared services/databases.
1. Execute the `review-request` Dual-Agent Review Protocol: run two independent read-only review passes (in parallel via subagents if available, otherwise sequentially):
   - **Pass 1 (Compliance):** Verify requirement coverage, scope boundaries, correctness/regressions, architecture/naming rules, and whether verification evidence proves each claim. Separate Confirmed Findings from Optional Suggestions.
   - **Pass 2 (Adversarial):** Assume the code is flawed until proven otherwise. Find up to 5 concrete, `file:line`-backed issues per category (fewer if no real evidence exists; never invent filler): (a) Correctness/edge cases, (b) Over-engineering, (c) Naming/readability, (d) Typical agentic pitfalls (broad `except`, dead helpers/shims, redundant comments, weak typing), (e) Security/reliability.
2. Verify every reported finding against the actual code, discard false positives and rules that conflict with project conventions, deduplicate, and rank validated findings as `Critical`/`High`, `Medium`, or `Low` with `file:line` evidence and recommended fixes. Wait for developer approval before editing code.
```

---

## Part 2: Dual-Agent Review Protocol

When conducting the review, **MUST NOT** modify any files in the working tree.

Spawn **two independent, read-only reviewer subagents** in parallel (or run two separate review passes sequentially if subagents are unavailable), then synthesize their findings:

### Reviewer 1: Specification & Compliance Reviewer
Evaluates the implementation against the original request, approved plan, and project conventions:
- **Scope & Plan Alignment:** Does the change fulfill every requirement without unrequested scope creep?
- **Correctness & Regressions:** Are logic, state transitions, error handling, and boundary conditions sound? Could existing behavior break?
- **Architecture & Naming:** Does the code follow established repository patterns, module boundaries, and active naming skills (such as `python-naming`)?
- **Test Quality & Evidence:** Do tests verify real behavior and edge cases, and does the cited validation output actually prove each completion claim?
- **Output:** Separate **Confirmed Findings** (concrete bugs, regressions, or rule violations with `file:line` evidence) from **Optional Suggestions**.

### Reviewer 2: Adversarial & Skeptical Reviewer
Adopts a zero-trust mindset—assuming the implementation contains hidden bugs, over-engineering, and generated-code smells until proven otherwise. Hunts for concrete evidence (**up to 5 findings per category**; report fewer if no genuine evidence exists—never fabricate filler):
1. **Correctness & Edge-Case Failures:** Off-by-one errors, race conditions, partial-failure state corruption, unhandled `None`/empty inputs, or broken invariants.
2. **Over-Engineering & Accidental Complexity:** Premature abstractions, unnecessary classes/layers where a plain function suffices, speculative configurability, or indirection without value.
3. **Naming & Readability Violations:** Vague placeholders (`data`, `result`, `process`, `handle`), grandiose suffixes (`Manager`, `Orchestrator`, `Engine`, `Helper`), inconsistent domain terms, or misleading boolean/method grammar.
4. **Typical Agentic Pitfalls:** Overly broad `try / except Exception` blocks that swallow errors, redundant comments restating the code, dead/unreferenced helpers, unrequested backwards-compatibility shims, or weak typing (`Any`, `cast`) masking type mismatches.
5. **Security & Reliability Risks:** Injection vectors, missing authorization/tenant scoping, leaked secrets/PII in logs, unbounded queries/loops, or missing timeouts on external I/O.

### Orchestrator Synthesis (Final Output in Chat)
After both reviewers finish:
1. **Verify every reported finding** against the actual codebase; discard false positives, hallucinated issues, and style nitpicks that contradict project rules.
2. **Deduplicate and rank** all validated findings by severity:
   - `Critical` / `High`: Correctness bugs, data loss/security risks, broken contracts, or missing requirements.
   - `Medium`: Architectural violations, over-engineering, agentic anti-patterns, or missing edge-case tests.
   - `Low`: Minor naming, readability, or cleanup suggestions.
3. **Present the ranked report in chat** with exact `file:line` references, concrete evidence, and recommended fixes, then wait for developer approval before making any code changes.
