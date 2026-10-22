---
name: review-response
description: Evaluates, triages, and responds to external agentic code review findings using evidence-based senior engineering judgment before implementing fixes. Use when the user provides findings, critique, or reports from an external code review (as a markdown prompt, report, or file path).
---

When presented with findings from an external code review (pasted as markdown or referenced via a file path), execute this protocol to critically evaluate, verify, and triage each finding before modifying code.

**Bypass rule:** If the user explicitly instructs to fix all review findings unconditionally, bypass this evaluation and proceed directly to implementation. Otherwise, follow the three-step workflow below.

---

## 1. Evidence-Based Critical Evaluation

Evaluate every finding independently with senior-level engineering rigor. Do not assume the reviewer is correct, and do not defend prior decisions out of bias or ego—let concrete codebase evidence drive every verdict.

### Verification Protocol
For each reported finding:
1. **Inspect actual code:** Navigate to the cited `file:line` and trace execution paths, runtime invariants, and call hierarchies.
2. **Verify against reality:** Check the claim against the repository's type system, runtime constraints, active skills (e.g., naming conventions), architecture boundaries, and test suite.
3. **Dispassionate evaluation:** If the reviewer demonstrates an actual bug, oversight, or design flaw with valid evidence, accept it immediately without rationalization.

### Rejection Criteria (Decline Finding)
Firmly decline findings that meet any of the following criteria, providing a concrete technical rebuttal backed by code evidence:
- **Incorrect or mistaken:** Reviewer misread the code, hallucinated nonexistent APIs or behaviors, or misdiagnosed execution flow.
- **Imaginary or speculative problems:** Theoretical concerns that cannot occur due to domain constraints, upstream guarantees, framework invariants, or strict typing (e.g., defensive null checks when schemas/types guarantee presence, speculative thread-safety on single-threaded workers).
- **Inappropriate or scope violation:** Demands modifications outside the agreed task scope, enforces unadopted external idioms, or introduces premature abstractions and accidental complexity.
- **Rule contradiction:** Proposes changes that violate established repository patterns, architecture guides, or active skills.

### Acceptance & Prioritization Criteria (Accept Finding)
Accept findings supported by concrete codebase evidence and categorize by severity:
- **Critical / High:**
  - Security vulnerabilities (injection, authorization bypass, secret exposure, insecure deserialization).
  - Correctness bugs, data corruption risks, state leakage, or unhandled runtime exceptions.
  - Architectural or module boundary violations.
- **Medium:**
  - Spaghetti, unclear, or unreadable code that obscures intent or increases cognitive load.
  - Missing error handling or unhandled failure states in realistic execution paths.
  - Missing tests for critical, reachable edge cases.
  - Verifiable performance bottlenecks with tangible impact.
- **Low:**
  - Minor naming or readability inconsistencies.
  - Redundant comments restating obvious code.
  - Dead code or minor refactoring cleanups.

---

## 2. Structured Triage Report

Present findings in chat using a compact, structured format. Do not modify files during this phase.

```markdown
### Review Findings Triage

**Summary:** `<total>` findings evaluated — `<accepted_count>` accepted, `<declined_count>` declined, `<open_count>` requiring clarification.

#### Accepted Findings (Prioritized)
| ID | Priority | Location | Finding & Evidence | Proposed Remediation |
|---|---|---|---|---|
| F-01 | Critical | `path/to/file.py:42` | Concrete bug description and verification evidence. | Minimal, targeted fix approach. |
| F-02 | High | `path/to/file.py:88` | Architecture violation or unreadable code evidence. | Refactoring or cleanup strategy. |

#### Declined Findings
| ID | Location | Reviewer Claim | Technical Rebuttal & Code Evidence |
|---|---|---|---|
| F-03 | `path/to/file.py:15` | Claimed issue. | Why it is invalid, mistaken, or an imaginary problem (with concrete proof from code/types). |

#### Open Questions & Trade-offs *(if applicable)*
- **F-04:** Ambiguity or trade-off requiring human developer decision before action.
```

---

## 3. Human Alignment & Approval Gate

1. **Stop and wait:** After presenting the triage report, **MUST NOT** edit files, stage commits, or run destructive commands.
2. **Seek explicit user approval:** Ask the user to confirm the accepted findings, review the declined justifications, and decide on any open trade-offs.
3. **Resolve ambiguities:** Discuss and align with the human developer until no unclarity remains.
4. **Implement approved fixes only:** Once approval is granted, implement targeted fixes for the agreed findings, keeping changes minimal, verifying with relevant tests, and adhering to repository conventions.
