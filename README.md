# Skills

Opinionated, reusable agent skills designed for portability across **Codex**, **Claude Code**, and **Antigravity**. Each skill lives in its own directory under [`skills/`](skills/).

## Installation & Updating

### Option 1: Clone & Symlink (Recommended — Easy Updates)

Clone this repository once and symlink the skills into your agent's global (or project-local) skills directory.

```sh
git clone git@github.com:dev-null000/skills.git ~/skills
```

- **Codex** (Global: `~/.codex/skills/` or Project: `.agents/skills/`):
  ```sh
  mkdir -p ~/.codex/skills
  ln -sfn ~/skills/skills/* ~/.codex/skills/
  ```
- **Claude Code** (Global: `~/.claude/skills/` or Project: `.claude/skills/`):
  ```sh
  mkdir -p ~/.claude/skills
  ln -sfn ~/skills/skills/* ~/.claude/skills/
  ```
- **Antigravity** (Global: `~/.gemini/config/skills/` or Project: `.agents/skills/`):
  ```sh
  mkdir -p ~/.gemini/config/skills
  ln -sfn ~/skills/skills/* ~/.gemini/config/skills/
  ```

**Updating symlinked skills:**
Pull the latest changes in your cloned repository (and run the `ln -sfn` command again if new skill directories were added), then start a new agent session:

```sh
git -C ~/skills pull
```

### Option 2: Codex `$skill-installer` (Direct from GitHub)

In Codex, you can install individual skills directly from GitHub without cloning the repository:

```text
$skill-installer install https://github.com/dev-null000/skills/tree/main/skills/python-naming
$skill-installer install https://github.com/dev-null000/skills/tree/main/skills/folder-structure-preview
```

**Updating `$skill-installer` skills:**
Because `$skill-installer` downloads a static copy into `~/.codex/skills/<skill-name>`, update a skill by removing the existing directory and reinstalling (or asking Codex to replace it), then restart Codex:

```sh
rm -rf ~/.codex/skills/python-naming
# Then re-run $skill-installer in Codex
```

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

## License

Licensed under the [MIT License](LICENSE). Provided "as is", without warranty of any kind. Third-party adaptations retain their original attribution noted in the respective skill files.

