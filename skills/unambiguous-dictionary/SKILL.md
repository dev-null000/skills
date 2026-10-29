---
name: unambiguous-dictionary
description: Freezes and enforces canonical naming and domain terminology across a repository or module using an in-repo dictionary of names, patterns, and meanings. Use when creating or updating project dictionaries, establishing ubiquitous language, freezing naming decisions, or validating identifiers against canonical terminology.
---

Freeze and enforce canonical naming across the codebase or module using an in-repository dictionary. Eliminate lexical drift, synonym proliferation, and ambiguous naming in plans, code, and reviews.

## Output Style Requirement

You **MUST** write all dictionary entries, definitions, rationale, questions, and review findings 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## The Two Core Entry Patterns

Every entry in the dictionary must follow one of two patterns:

### Pattern 1: Exact Domain Term (Ubiquitous Language)

Freezes a single business or domain noun, entity, or action. Binds it to one unambiguous meaning and bans competing synonyms.

- **Name:** The single canonical word or phrase (e.g., `customer`, `workspace`, `line_item`).
- **Meaning:** Authoritative definition. States what the entity represents in the business domain.
- **Disallowed synonyms:** All forbidden alternatives (e.g., `client`, `buyer`, `user`, `account`).

### Pattern 2: Structural & Architectural Pattern (Syntactic Conventions)

Freezes naming shapes, affixes, compound conventions, and architectural roles across files and symbols.

- **Pattern:** Wildcard, prefix, suffix, or compound template (e.g., `*_at`, `*_count`, `*_by_<key>`, `*_gateway`, `is_*`).
- **Meaning:** Authoritative technical contract. States data type, unit, role, or polarity.
- **Disallowed patterns:** Competing formats or misnomers (e.g., `*_timestamp`, `num_*`, `<key>_to_*`, `*_client`).

---

## Dictionary File Location & Discovery

Search for an existing dictionary in the repository in this order:

1. `DICTIONARY.md` (project root)
2. `GLOSSARY.md` (project root)
3. `docs/dictionary.md` or `docs/glossary.md`
4. `.agents/dictionary.md`

### Scoped Dictionaries

For monorepos or distinct subdomains, locate scoped dictionaries within module directories (e.g., `src/billing/DICTIONARY.md`).
- Scoped dictionaries inherit all entries from the root dictionary.
- Scoped entries apply only within their module directory.
- A scoped dictionary must never redefine a root term with a conflicting meaning unless explicitly scoped as an isolated bounded context.

### Initialization

If no dictionary file exists when requested to freeze namings or initialize the dictionary, create `DICTIONARY.md` at the project root using the standard format.

---

## Standard Dictionary Format

Store the dictionary as a clean Markdown table with these columns:

```markdown
# Project Dictionary

Canonical domain terminology and structural naming patterns. Enforce these terms across all code, tests, docs, and plans.

## Domain Terms

| Name | Disallowed Synonyms | Meaning | Example / Context |
| :--- | :--- | :--- | :--- |
| `customer` | `client`, `buyer`, `user`, `account` | External entity with an active subscription contract. | `customer_id: CustomerId` |
| `workspace` | `tenant`, `team`, `organization`, `org` | Top-level tenant container owning projects and billing. | `workspace.projects` |
| `order` | `purchase`, `transaction`, `deal` | Confirmed request to purchase goods or services. | `order_repository.load_by_id()` |
| `dispatch` | `trigger`, `publish`, `emit`, `fire` | Action that schedules an asynchronous background task. | `dispatch_sync_job()` |

## Structural Patterns

| Pattern | Disallowed Patterns | Meaning | Example / Context |
| :--- | :--- | :--- | :--- |
| `*_at` | `*_timestamp`, `*_time`, `*_date` | Point-in-time event timestamp in UTC. | `created_at`, `expires_at` |
| `*_count` | `num_*`, `count_*`, `*_num`, `total_*` | Non-negative integer count of items. | `retry_count`, `active_user_count` |
| `*_by_<key>` | `<key>_to_*`, `*_map`, `*_dict`, `*_mapping` | Hash map or dictionary indexed by `<key>`. | `user_by_email`, `total_by_year` |
| `*_gateway` | `*_client`, `*_service`, `*_api` | Adapter class that encapsulates an external third-party API. | `StripeGateway`, `TwilioGateway` |
| `is_*` | `*_flag`, `check_*`, negative booleans | Affirmative boolean predicate flag. | `is_active`, `is_verified` |
```

---

## Operating Workflows

### Mode 1: Audit & Verification (Read Mode)

Use when planning changes, generating code, or reviewing pull requests.

1. **Scan candidate symbols:**
   - Extract domain nouns, class names, functions, variables, attributes, and file names from the planned or modified code.
2. **Match against dictionary:**
   - Verify every domain concept matches a registered `Name`.
   - Verify every compound identifier follows registered `Pattern` rules.
3. **Detect naming violations:**
   - **Disallowed Synonym:** Code uses a banned alternative (e.g., `client_id` instead of `customer_id`).
   - **Semantic Collision:** Code uses a registered term with a conflicting meaning (e.g., `workspace` for a temporary directory).
   - **Pattern Drift:** Code violates a structural pattern (e.g., `delivery_time` instead of `delivered_at`).
   - **Unregistered Domain Entity:** Code introduces a new business concept not present in the dictionary.
4. **Report findings:**
   - Present violations in a concise table with exact resolution steps:

| Symbol / Location | Violation | Canonical Rule | Resolution |
| :--- | :--- | :--- | :--- |
| `client_id` in `invoice.py` | Disallowed synonym | Use `customer` | Rename to `customer_id` |
| `creation_time` in `job.py` | Pattern drift | Suffix timestamps with `*_at` | Rename to `created_at` |

---

### Mode 2: Freezing & Registration (Write Mode)

Use when establishing new terminology or adding entities during feature development.

1. **Identify the candidate term or pattern:**
   - Formulate the exact canonical name.
   - List common synonyms, misnomers, or abbreviations to ban explicitly.
2. **Verify uniqueness:**
   - Confirm the concept does not duplicate an existing entry.
   - Confirm the name does not overload an existing term with a different meaning.
3. **Define meaning:**
   - Write a definition under 20 words stating the entity, role, or technical contract.
4. **Update dictionary:**
   - Insert the new row into the repository dictionary file in alphabetical order.
   - Present the addition to the user for confirmation.

---

### Mode 3: Repository Initialization (Bootstrap Mode)

Use when creating a dictionary for an existing repository that lacks one.

1. **Extract candidate terms:**
   - Inspect database schemas, core entity models, domain events, and primary service interfaces.
   - Collect recurring domain nouns and compound identifier patterns.
2. **Identify active synonyms:**
   - Detect synonym fragmentation across files (e.g., `user_id` vs. `account_id` for the same entity).
   - Select the dominant or most accurate term as canonical; mark the other as disallowed.
3. **Generate initial `DICTIONARY.md`:**
   - Populate Domain Terms and Structural Patterns tables.
   - Present the generated dictionary to the user before committing.

---

## Rules and Invariants

1. **One Concept, One Name:**
   - Never assign multiple names to one domain concept.
   - Pick one canonical term and record all alternatives as disallowed synonyms.

2. **One Name, One Concept:**
   - Never reuse a registered domain name for an unrelated concept within the same scope.

3. **Explicit Negative Constraints:**
   - Every domain entry must record disallowed synonyms to block agent synonym rotation.

4. **Scope Boundaries:**
   - Do NOT record ephemeral variables (loop indices `i`, `j`, caught exceptions `e`, context file handles `f`).
   - Focus strictly on domain entities, ubiquitous business terms, public interfaces, and architectural patterns.

5. **Enclosing Scope Precedence:**
   - Rely on enclosing class or module scope to avoid redundancy.
   - Inside class `Customer`, attribute is `email`, not `customer_email`.
   - Outside class `Customer`, reference is `customer_email` or `customer.email`.
