---
name: dual-agent-review
description: Orchestrates a dual-agent code review (specification compliance + adversarial critique) and synthesizes ranked findings. Use when asked to review code, audit a diff or pull request, or run a dual-agent review.
---

This skill orchestrates two independent read-only reviewer passes (Compliance + Adversarial) and synthesizes their findings.

Do NOT modify files in the working tree during review.

## Output Style Requirement

You **MUST** write all review findings, synthesis reports, and remediation recommendations 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Provide concrete `file:line` references for every finding.
- State direct imperative remediations.
- Use one term per concept. Do not rotate synonyms.

---

## Dual-Agent Review Protocol

Spawn **two independent read-only reviewer subagents** in parallel (or run two sequential passes if subagents are unavailable). Synthesize findings afterward.

### Reviewer 1: Specification & Compliance Reviewer
Evaluate the implementation against the original request, approved plan, and project conventions:
- **Scope & Plan Alignment:** Verify full requirement coverage without unrequested changes.
- **Correctness & Regressions:** Check logic, state changes, error handling, and boundaries for defects.
- **Architecture & Naming:** Verify module boundaries, repository patterns, and naming rules from active skills.
- **Test Quality & Evidence:** Confirm tests exercise real behavior and verification claims map to actual output.
- **Output:** Separate **Confirmed Findings** (`file:line` evidence required) from **Optional Suggestions**.

### Reviewer 2: Adversarial & Skeptical Reviewer
Assume the implementation contains defects, premature abstractions, and generated code smells. Find up to 5 concrete findings per category with `file:line` proof. Do not fabricate filler:
1. **Correctness & Edge Cases:** Off-by-one errors, race conditions, partial-failure states, unhandled `None` or empty inputs.
2. **Over-Engineering:** Premature abstractions, unneeded classes or layers where simple functions suffice, speculative parameters.
3. **Naming & Readability:** Vague names (`data`, `result`, `process`), grandiose suffixes (`Manager`, `Engine`), inconsistent domain terms.
4. **Agentic Pitfalls:** Broad `except Exception` blocks, redundant comments restating code, dead helpers, weak types (`Any`, `cast`).
5. **Security & Reliability:** Injection vectors, missing authorization, leaked secrets, unbounded queries, missing network timeouts.

---

## Orchestrator Synthesis (Final Output in Chat)

After both reviewer passes finish:
1. **Verify every finding:** Inspect code at cited lines. Discard false positives, hallucinated issues, and style nitpicks.
2. **Deduplicate and rank findings:**
   - `Critical` / `High`: Bugs, data loss, security vulnerabilities, broken contracts, or missing requirements.
   - `Medium`: Architectural violations, over-engineering, agentic anti-patterns, or missing edge-case tests.
   - `Low`: Minor naming, readability, or cleanup suggestions.
3. **Present ranked report:** Output report in chat using the structure below. Adhere strictly to the 80% ASD-STE100 output rule.
4. **Wait for approval:** Do not edit files until the user explicitly approves the remediation plan.

```markdown
### Code Review Synthesis

**Summary:** Evaluated `<scope>` across Compliance and Adversarial passes. Found `<count>` confirmed issues.

#### Critical & High Severity Findings
| ID | Category | Location | Finding & Evidence | Proposed Remediation |
|---|---|---|---|---|
| C-01 | Correctness | `app/services/auth.py:42` | Missing token expiration check allows expired session reuse. | Add expiration timestamp validation before signature verification. |

#### Medium Severity Findings
| ID | Category | Location | Finding & Evidence | Proposed Remediation |
|---|---|---|---|---|
| M-01 | Architecture | `app/domain/order.py:15` | Domain entity imports infrastructure ORM model directly. | Invert dependency through domain repository port. |

#### Low Severity Findings
- **L-01:** `app/models/user.py:65` — Method `get_name()` violates naming conventions. Rename to property `full_name`.

#### Next Steps
Wait for developer confirmation before making any code changes.
```
