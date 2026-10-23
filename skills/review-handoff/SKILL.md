---
name: review-handoff
description: Prepares a concise, evidence-based implementation handoff for review. Use when asked for a review handoff, handoff for review, review request, or summary of changes for review.
---

This skill prepares a concise, evidence-based summary of an implemented change. It pairs the handoff with a compact prompt for an external or fresh reviewer agent.

## Output Style Requirement

You **MUST** write all handoff sections, table entries, and reviewer instructions 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use single-sentence bullets and compact tables.
- Use one term per concept. Do not rotate synonyms.

---

## Brevity & Deduplication Rules

- **Link, do not restate:** Reference plan files, specs, ADRs, `AGENTS.md`, and skills by path. Do not copy rules that exist on disk.
- **State each fact once:** Do not repeat limitations, caveats, or safety rules across sections.
- **No prose essays or inline scripts:** Use single-sentence bullets and compact tables. Do not paste multi-line shell or Python wrapper scripts.

---

## Required Handoff Sections

Output the review handoff with these 8 sections:

1. **Goal & Implementation Summary** *(max 5 bullets)*: State original objective and summarize implemented changes.
2. **Context & Reference Links:** State repository path, base commit, diff command, and paths to plans and rules.
3. **Changed Files & Components:** Provide a compact table. Include a 1-line summary per file and total diff stats. For changes over 15 files, group by module into at most 15 rows.
4. **Verification Evidence:** Provide a compact list formatted as `command -> 1-line observed result`. Map every completion claim to a test, linter, or type check.
5. **Resolved Issues & Key Decisions** *(max 5 bullets)*: State non-obvious edge cases or design trade-offs.
6. **Unverified Areas** *(max 5 bullets)*: List items not verified automatically and state why.
7. **Remaining Risks & Next Steps** *(max 5 bullets)*: State operational risks or follow-up tasks. Do not repeat unverified items.
8. **Reviewer Prompt:** Append a copy-pasteable instruction block under 15 lines invoking the `dual-agent-review` skill:

```markdown
### Reviewer Instructions
Review changes in `<repo-path>` (scope: `<diff-command-or-paths>`, base: `<commit>`) against the handoff above and `<paths-to-plan-and-AGENTS.md>`. Do NOT edit files, stage commits, or touch external services.
Execute the `dual-agent-review` skill to audit these changes and present a ranked report. Do not edit files before developer approval.
```
