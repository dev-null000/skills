---
name: review-request
description: Prepares an evidence-based implementation handoff and orchestrates a dual-agent code review (specification compliance + adversarial critique). Use when asked for a review request, review handoff, handoff for review, or multi-agent code review.
---

This skill defines a two-part workflow:
1. **Review Handoff:** An evidence-based summary of an implemented change, paired with a ready-to-run review prompt for an external or fresh reviewer agent.
2. **Dual-Agent Review Protocol:** Instructions for an orchestrating reviewer agent to run two independent read-only reviewers (Compliance + Adversarial) and synthesize their findings.

When invoked after implementing a change, output the **Review Handoff** (including the embedded **Reviewer Instructions** block). When invoked to *perform* a review on a handoff or diff, execute the **Dual-Agent Review Protocol** directly.

---

## Part 1: Review Handoff Structure

Generate a concise, evidence-backed handoff containing these exact sections:

1. **Goal & Implementation Summary:** The original user request/problem and a concise summary of what was implemented.
2. **Context & Reference Links:** Paths or links to the approved implementation plan, architectural docs, ADRs, or skills (e.g., `python-naming`) that govern the change.
3. **Changed Files & Components:** List of added, modified, and removed files/modules with a 1-line summary of each change.
4. **Verification Evidence:** Exact test, linter, and type-check commands executed and their observed results. Every completion claim must cite its verification check.
5. **Resolved Issues During Implementation:** Notable edge cases, failures, or design trade-offs encountered and how they were resolved.
6. **Unverified Areas:** Anything that could not be tested or verified automatically (e.g., external APIs, missing credentials, production-only migrations) and why.
7. **Remaining Risks & Next Steps:** Known limitations, operational risks, or recommended follow-up tasks.
8. **Reviewer Prompt:** Append a copy-pasteable prompt instructing the reviewing agent to execute **Part 2: Dual-Agent Review Protocol** below.

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
