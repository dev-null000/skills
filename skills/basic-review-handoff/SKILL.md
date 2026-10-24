---
name: basic-review-handoff
description: Prepares a short, evidence-based handoff for small or low-risk changes. Use when asked for a basic or lightweight review handoff, or when a change is too small for a detailed handoff.
---

Prepare a short handoff of an implemented change for a reviewer. Use this skill for small or non-critical changes. For large, risky, or multi-module changes, use a detailed handoff instead.

## Output Style Requirement

You **MUST** write the handoff 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit filler and hedging.
- Use single-sentence bullets.
- Use one term per concept. Do not rotate synonyms.

## Rules

- Base every statement on evidence. Map each completion claim to a command and its observed result.
- Link plans, specs, and rule files by path. Do not copy their content.
- State each fact once. Keep the whole handoff under 50 lines.

## Handoff Sections

1. **Goal & Implementation:** State the original goal and what the change implements.
2. **Changed Files:** List changed files or application areas with a 1-line summary each.
3. **Verification:** List completed tests and checks as `command -> 1-line observed result`.
4. **Issues & Resolutions:** List issues found during development and how each was resolved. Write `None` if empty.
5. **Unverified:** List what you could not verify and why. Write `None` if empty.
6. **Remaining Risks:** List open risks. Do not repeat unverified items. Write `None` if empty.

## Reviewer Prompt

Append this block:

```markdown
### Reviewer Instructions
Review changes in `<repo-path>` (scope: `<diff-command-or-paths>`) against the handoff above. Do NOT edit files. Rank findings by severity with evidence.
```
