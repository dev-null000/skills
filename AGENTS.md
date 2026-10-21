# Repository guidance

This repository packages opinionated workflows as reusable skills for Codex, Claude, and Antigravity.

## Structure

- `skills/<skill-name>/SKILL.md` contains each skill's metadata and instructions.
- `Taskfile.yaml` defines development commands; `pyproject.toml` and `uv.lock` manage tooling.
- See [README.md](README.md#contributing) for prerequisites, setup, and direct command alternatives.

## Conventions

- Make the smallest coherent change. Keep instructions concise, explicit, and portable.
- Follow the [Agent Skills specification](https://agentskills.io/specification). Match the skill directory and frontmatter `name`; use relative paths for bundled resources.
- Add supporting files only when they serve the skill's workflow. Add nested `AGENTS.md` files only when a directory needs distinct guidance.
- Use Conventional Commits.

## Validation

- Run `task validate -- skills/<skill-name>` for each added or changed skill.
- Review instruction changes for clear scope, actionable guidance, and valid resource links; format validation alone does not establish usefulness.
- When changing tooling, run the affected command. Report checks performed and any unresolved failures.

## Local files

- Keep credentials and `.venv/` out of version control. Recreate environments through the documented setup instead of editing installed packages.
- Update `uv.lock` through uv when changing dependencies.
