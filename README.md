# Skills

Opinionated, reusable agent skills designed for portability across **Codex**, **Claude Code**, and **Antigravity**. Each skill lives in its own directory under [`skills/`](skills/).

## Skill Catalog

The skills form a complete, decoupled lifecycle from feature discovery to verification and review:

| Phase | Skill | Purpose & Trigger |
| :--- | :--- | :--- |
| **Discovery** | [`task-interview`](skills/task-interview) | Classifies task type and scale (`S/M/L`), scans targeted scope, resolves ambiguities via an adaptive interview, and outputs a self-contained Task. |
| **Architecture** | [`folder-structure-preview`](skills/folder-structure-preview) | Renders a diff-annotated tree of files, classes, and method signatures before writing code to verify module boundaries and naming. |
| **Conventions** | [`python-naming`](skills/python-naming) | Enforces semantic naming, PEP 8 baselines, affirmative booleans, and hexagonal architecture port/repository patterns. |
| **Planning** | [`sliced-execution-plan`](skills/sliced-execution-plan) | Decomposes multi-file features into logically cohesive, human-reviewable vertical slices with lower-tier model delegation markers and approval gates. |
| **Execution** | [`implement-plan`](skills/implement-plan) | Executes a plan of any format while enforcing checkpoints, deterministic tool verification, and immediate escalation when code reality contradicts the plan. |
| **Testing (Authoring)** | [`damp-tests`](skills/damp-tests) | Enforces Descriptive And Meaningful Phrases (Arrange-Act-Assert, literal values, no loops/conditionals, 99% comments avoided). |
| **Testing (Auditing)** | [`adversarial-test-verification`](skills/adversarial-test-verification) | Audits tests for tautologies and over-mocking; runs a Hybrid Sabotage gate (static trace or live kill matrix) to report surviving logic mutants. |
| **Review (Basic)** | [`basic-review-handoff`](skills/basic-review-handoff) | Prepares a concise, evidence-based handoff under 50 lines for small or low-risk changes. |
| **Review (Basic)** | [`basic-review`](skills/basic-review) | Runs a single-pass, read-only review against 6 core points (Correctness, Scope, Fit, Coverage, Security, Uncertainty). |
| **Review (Deep)** | [`review-handoff`](skills/review-handoff) | Prepares an in-depth, structured 8-section handoff with file and risk budgets for complex changes. |
| **Review (Deep)** | [`dual-agent-review`](skills/dual-agent-review) | Orchestrates two independent reviewer passes (Specification Compliance + Adversarial Critique) and synthesizes ranked findings. |
| **Review (Deep)** | [`resource-efficiency-review`](skills/resource-efficiency-review) | Audits code strictly for bare-metal physical resource scaling, connection pooling, memory leaks, and saturation limits across 9 pillars. |
| **Triage** | [`review-response`](skills/review-response) | Evaluates and triages external review comments using concrete code facts and technical rebuttals before modifying code. |

---

## Installation & Updating

### Option 1: Clone & Symlink (Recommended — Easy Updates)

Clone this repository once (or navigate to your existing clone) and symlink the skills into your agent's global (or project-local) skills directory from the repository root:

```sh
git clone git@github.com:dev-null000/skills.git
cd skills
```

- **Codex** (Global: `~/.codex/skills/` or Project: `.agents/skills/`):
  ```sh
  mkdir -p ~/.codex/skills
  ln -sfn "$PWD/skills"/* ~/.codex/skills/
  ```
- **Claude Code** (Global: `~/.claude/skills/` or Project: `.claude/skills/`):
  ```sh
  mkdir -p ~/.claude/skills
  ln -sfn "$PWD/skills"/* ~/.claude/skills/
  ```
- **Antigravity** (Global: `~/.gemini/config/skills/` or Project: `.agents/skills/`):
  ```sh
  mkdir -p ~/.gemini/config/skills
  ln -sfn "$PWD/skills"/* ~/.gemini/config/skills/
  ```

**Updating symlinked skills:**
Pull the latest changes from the repository root (and re-run the `ln -sfn` command if new skill directories were added), then start a new agent session:

```sh
git pull
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

