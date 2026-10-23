---
name: review-response
description: Evaluates, triages, and responds to external code review findings using concrete codebase evidence before changing code. Use when given review findings, critique, or reports.
---

When given findings from an external code review, execute this protocol. Evaluate, verify, and triage each finding before modifying code.

**Bypass rule:** If the user explicitly commands to fix all findings unconditionally, bypass evaluation and implement fixes directly. Otherwise, follow the three steps below.

## Output Style Requirement

You **MUST** write all triage reports, table entries, technical rebuttals, and remediations 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, polite padding, and hedging.
- Write dispassionate technical rebuttals backed by code facts. Never write defensive prose.
- Use one term per concept. Do not rotate synonyms.

---

## 1. Evidence-Based Verification

Evaluate each finding independently. Base every decision on verifiable code evidence. Do not assume the reviewer is correct.

### Verification Steps
For each reported finding:
1. **Inspect code:** Open the cited `file:line`. Trace call hierarchies, data flows, and runtime invariants.
2. **Verify against reality:** Check the claim against static types, tests, runtime behavior, and active skill rules.
3. **Accept valid defects:** If code evidence confirms a defect or design flaw, accept the finding immediately.

### Rejection Criteria (Decline Finding)
Decline a finding when it meets any of these conditions. Provide a factual technical rebuttal with line references:
- **Incorrect claim:** The reviewer misread code, assumed wrong execution flow, or cited nonexistent APIs.
- **Speculative issue:** The scenario cannot occur because types, framework invariants, or domain boundaries prevent it.
- **Scope violation:** The request demands changes outside the task scope, adds unneeded abstractions, or forces alien idioms.
- **Rule contradiction:** The suggested change violates established project architecture, conventions, or active skills.

### Acceptance Criteria (Accept Finding)
Accept findings supported by concrete code evidence. Assign severity:
- **Critical / High:**
  - Security vulnerabilities (injection, authorization bypass, secret leaks, unsafe deserialization).
  - Correctness bugs, data loss, state corruption, or unhandled exceptions.
  - Module boundary or architecture violations.
- **Medium:**
  - Logic that obscures intent or increases cognitive load.
  - Missing error handling for realistic failure paths.
  - Missing tests for reachable edge cases.
  - Measurable performance bottlenecks.
- **Low:**
  - Minor naming or style inconsistencies.
  - Redundant comments that restate code.
  - Dead code or minor cleanup opportunities.

---

## 2. Structured Triage Report

Present findings in chat using this Markdown structure. Do not edit files during triage. Write all text adhering to the 80% ASD-STE100 rule.

```markdown
### Review Findings Triage

**Summary:** `<total>` findings evaluated — `<accepted_count>` accepted, `<declined_count>` declined, `<open_count>` requiring clarification.

#### Accepted Findings (Prioritized)
| ID | Priority | Location | Finding & Evidence | Proposed Remediation |
|---|---|---|---|---|
| F-01 | Critical | `path/to/file.py:42` | Function raises unhandled `KeyError` on empty payload. | Add explicit dictionary check and return `400` status. |
| F-02 | High | `path/to/file.py:88` | Domain entity imports infrastructure database model directly. | Invert dependency through domain repository port. |

#### Declined Findings
| ID | Location | Reviewer Claim | Technical Rebuttal & Code Evidence |
|---|---|---|---|
| F-03 | `path/to/file.py:15` | Add null check for `user.id`. | Schema enforces non-nullable `user.id`. `User` type guarantees string value. |

#### Open Questions & Trade-offs *(if applicable)*
- **F-04:** Ambiguity or trade-off requiring human developer decision before action.
```

---

## 3. Human Alignment & Approval Gate

1. **Stop execution:** Stop after presenting the triage report. Do not edit files or run destructive commands.
2. **Request user approval:** Ask the user to confirm accepted findings, review declined rebuttals, and resolve open trade-offs.
3. **Wait for confirmation:** Wait until the user approves the triage plan.
4. **Implement approved fixes only:** Implement targeted fixes for approved findings only. Keep changes minimal, run tests, and follow project style.
