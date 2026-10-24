---
name: basic-review
description: Runs a single-agent, read-only review of a change against six basic points and ranks findings by severity with evidence. Use when asked for a basic or quick review, or to review a small or low-risk change.
---

Review a change in one pass against six basic points. Use this skill for small or non-critical changes. For large, risky, or multi-module changes, use a deeper multi-reviewer process.

**MUST NOT change source code.** Do NOT edit, stage, or commit files. Do NOT touch external services.

## Output Style Requirement

You **MUST** write all findings and remediations 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit filler and hedging.
- Use one term per concept. Do not rotate synonyms.

## Review Points

1. **Correctness:** Does the implementation produce the requested behavior?
2. **Scope:** Are all changes necessary for that outcome?
3. **Fit:** Does the implementation follow existing repository patterns?
4. **Coverage:** Do the tests address important behavior, edge cases, and regressions?
5. **Security:** Do the changes introduce risks involving sensitive information, access, or application behavior?
6. **Uncertainty:** Does any part of the result depend on an unverified assumption?

## Rules

- Read the diff and the handoff (if present). Inspect cited code before you report a finding.
- Report a finding only with evidence: a `file:line` reference and a 1-sentence reason. Discard guesses.
- Do not invent findings to fill a point. Write `No findings` for a clean point.

## Output

Rank findings by severity: `Critical`, `High`, `Medium`, `Low`. Use this structure:

```markdown
### Basic Review

**Scope:** `<diff or paths>` | **Findings:** `<count>`

| ID | Severity | Point | Location | Evidence | Remediation |
|---|---|---|---|---|---|
| F-01 | High | Correctness | `app/orders.py:42` | Empty cart raises `KeyError`. | Return a validation error for an empty cart. |

#### Points With No Findings
- <Point names>
```

Stop after the report. Wait for the user before any change.
