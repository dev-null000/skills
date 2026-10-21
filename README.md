# Skills

Opinionated, reusable Codex workflows, designed for portability to Claude and Antigravity. Each skill lives in its own directory.

## Contributing

This project uses [Conventional Commits](https://www.conventionalcommits.org/). Follow this convention when writing commit messages.

Requires [uv](https://docs.astral.sh/uv/getting-started/installation/). [Task](https://taskfile.dev/installation/) is optional.

Create `.venv` and install dependencies:

```sh
task venv
# Or: uv sync --locked
```

Validate a skill:

```sh
task validate -- ./skills/python-naming
# Or: .venv/bin/agentskills validate ./skills/python-naming
```
