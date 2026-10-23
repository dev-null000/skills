---
name: python-naming
description: Defines requirements for readable and consistent naming in Python. Use this skill when writing, refactoring, or reviewing Python code.
compatibility: Requires Python 3.11+
---

Follow PEP 8 as the baseline standard. Follow established project conventions when present. Otherwise, apply these semantic naming rules to all Python identifiers, modules, and packages.

**Scope boundary:** Apply these rules to newly written code. Apply these rules to existing code only when the user explicitly requests a rename or refactor. Do NOT rename identifiers outside the requested task scope. You may mention poor existing names as optional suggestions without modifying them.

## Output Style Requirement

You **MUST** write all docstrings, code comments, and optional rename suggestions 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Common Rules

### 1. Express Purpose, Role, Grammar, and Unit in Context

Use the shortest name that states what a value represents, what an action does, or what unit a quantity uses. Rely on enclosing scope (class, module, function) to avoid repeating context:

```python
class Customer:
    email: str  # Good: enclosing class supplies "customer" (avoid customer_email)
```

Do not use generic placeholders (`data`, `value`, `item`, `items`, `info`, `obj`, `flag`, `result`, `res`, `ret`, `tmp`).
Do not use vague standalone verbs (`process`, `handle`, `execute`, `manage`, `do`, `run`).
Generic action verbs are acceptable only when a uniform command contract or protocol specifies the operation (e.g., `CreateDatasetCommand.execute()`).

Follow these patterns:

- **Functions & methods:** use **verb** phrases (`parse_evaluation_response()`, `retry_failed_notifications()`) or
  boolean predicates (`is_expired()`).
- **`@property` & attributes:** use **nouns or adjectives** (`user.full_name`, `order.total_price`,
  `dataset.is_published`), never `get_`/`compute_` verbs (`user.get_full_name`).
- **Classes, `Enum`s, & `TypedDict`s:** use **singular** nouns (`class OrderStatus(StrEnum):`, never `OrderStatuses`).
- **Collections (`list`, `set`, `tuple` of items):** use **plural** nouns (`customer_ids`, `active_users`).
- **Mappings (`dict`):** use `<value>_by_<key>` (`email_by_customer_id`).
- **Booleans:** prefix with `is_`, `has_`, `can_`, `should_`, or an affirmative verb for config flags, and keep
  polarity **affirmative** (`is_enabled`, `is_valid`, `verify_ssl` — never negative names like `is_disabled`,
  `is_not_valid`, `disable_ssl`).
- **Units:** append the physical or time unit (`_km`, `_mm`, `_seconds`, `_ms`, `_bytes`).
- **Timestamps & counts:** suffix timestamps with `_at` (`report_generated_at`, `created_at`) and counts with `_count`
  (`failed_delivery_count`).
- **Collaborators & clients:** append the technical role (`_client`, `_gateway`, `_publisher`, `_repository`). Do not name attributes after vendors or technologies alone (e.g., use `self._stripe_gateway`, not `self._stripe`; use `self._redis_client`, not `self._redis`).
- **Modules:** use nouns for general modules (`evaluation_validator.py`); verb phrases
  (`validate_evaluation_response.py`) are acceptable for single-action command/use-case modules.
- **Test functions:** use `test_<subject>_<condition>_<outcome>` (e.g.,
  `test_withdraw_insufficient_funds_raises_error`).

| Role / Context | Poor | Good |
|---|---|---|
| `str` arg identifying a target stream by name | `publish(stream: str)` | `publish(stream_name: str)` |
| Target distance in kilometres | `distance = 20` | `distance_to_target_km = 20` |
| Drawing a line of a given length in millimetres | `draw(length: float)` | `draw_line(length_mm: float)` |
| Max wait time for an HTTP response (seconds) | `value = 30` | `response_timeout_seconds = 30` |
| Number of failed delivery attempts | `count` | `failed_delivery_count` |
| Permission check for editing a dataset | `flag` | `can_edit_dataset` |
| Affirmative boolean for SSL verification | `disable_ssl = False` | `verify_ssl = True` |
| Computed property on an `Order` class | `@property def get_total(self):` | `@property def total_price(self):` |
| Enum representing order lifecycle states | `class OrderStatuses(StrEnum):` | `class OrderStatus(StrEnum):` |
| Injected API adapter (name the role, not the corporation) | `self._stripe.charge()` | `self._stripe_gateway.create_charge()` |
| Injected Redis client (name the client role, not the server) | `self._redis.get(key)` | `self._redis_client.get(key)` |
| Collection of customer IDs | `items` | `customer_ids` |
| Dict mapping customer IDs to emails | `mapping` | `email_by_customer_id` |
| Timestamp when a report was generated | `timestamp` | `report_generated_at` |
| Path to a file | `path` | `file_path` |
| Base URL used to construct endpoint URLs | `url` | `base_url` |
| Identifier of a streaming server | `srv` | `streaming_server_id` |
| Recording start timestamp from decoded metadata | `start = data["start"]` | `recording_started_at = recording_metadata["start"]` |
| Decoded JSON payload of configuration changes | `data = json.loads(change.data)` | `configuration_changes = json.loads(configuration_change.data)` |
| Parsing an HTTP response into an evaluation result | `process(response)` | `parse_evaluation_response(response)` |
| Retrying notifications whose delivery failed | `handle_notifications()` | `retry_failed_notifications()` |
| Single-action module validating an external evaluation reply | `result_validation.py` | `validate_evaluation_response.py` |

### 2. Avoid Single-Character Names

Allow single-character names **only** in small, immediate scopes for:

- Short-lived loop counters/indices (`i`, `j`, `k`) or comprehension variables (`k`, `v`).
- Caught exceptions (`except ValueError as e:`).
- Context-managed file handles (`with open(...) as f:`).
- Intentionally unused values (`for _ in range(3):`) or established aliases
  (`from django.utils.translation import gettext as _`). Do not shadow an active `_()` callable with a discarded
  variable.
- Unconstrained private type variables and parameter specs (`_T = TypeVar("_T")`, `_P = ParamSpec("_P")`).
- Standard mathematical/algorithmic notation matching a reference paper.

| Role / Context | Poor | Good |
|---|---|---|
| Loading a user record | `u = User.objects.get(pk=user_id)` | `user = User.objects.get(pk=user_id)` |
| Loading application settings | `s = load_settings()` | `application_settings = load_settings()` |
| Today's calendar date | `d = date.today()` | `current_date = date.today()` |
| Current date and time | `n = datetime.now()` | `now = datetime.now()` |
| Dict mapping product IDs to prices | `p = {}` | `price_by_product_id = {}` |

### 3. Restrict Abbreviations

Use abbreviations **only** from these three categories:

1. **Standard acronyms & units:** `id`, `ip`, `udp`, `tcp`, `vpn`, `url`, `http`, `ttl`, `km`, `dto`, `iso`, `tx`, `rx`,
   `bps`, etc., plus framework conventions when unambiguous in context (e.g., Django `qs`, or `active_users_qs` /
   `active_users_queryset` when multiple querysets exist).
2. **Ecosystem import aliases:** `import numpy as np`, `import pandas as pd`, `import matplotlib.pyplot as plt`.
3. **Universally understood short forms** (prefer full words when clarity benefits):
   `clt` (client), `cfg` (configuration), `req` (request), `resp` (response), `err` (error), `gw` (gateway), `mgmt`
   (management), `msg` (message), `img` (image), `proc` (process), `repo` (repository), `uow` (unit of work).

Never invent ad-hoc truncations, consonant-stripped words, or project-only initials:

| Role / Context (and why the abbreviation fails) | Poor | Good |
|---|---|---|
| Customer record (arbitrary truncation) | `cust` | `customer` |
| Watch object (opaque domain initials) | `wo` | `watch_object` |
| List of orders (names container type, hides domain item) | `lst` | `orders` |
| Regex matching a filename timestamp (consonant stripping) | `ptrn` | `filename_timestamp_pattern` |
| Regex match for a timestamp (opaque single letter) | `m` | `timestamp_match` |
| Count of updated records (hides subject and that it is a count) | `aff` | `updated_record_count` |
| Row identifier (`r` is ambiguous) | `r_id` | `row_id` |
| File extension (ambiguous: extension, external, or extra) | `ext` | `extension` |
| `python-docx` text run from `paragraph.add_run()` | `r` | `text_run` |

### 4. Disallowed Patterns

- **Dashes in module/package names:** use `report_export.py`, never `report-export.py`.
- **Invented dunder names:** never define custom `__name__` identifiers; use only standard Python data-model methods
  (`__init__`, `__call__`, etc.).
- **Double-leading-underscore methods (`__private_method`):** use a single leading underscore (`_private_method`) for
  all non-public methods and attributes unless avoiding subclass name collisions.
- **Redundant container types in names:** use `name_by_id` or `users`, never `id_to_name_dict` or `user_list`.
- **Non-neutral or derogatory terms:** use accurate domain terminology.

### 5. Strictly Distinguish Public and Internal Names

Prefix all non-public (internal or protected) modules, classes, constants, variables, attributes, functions, and methods
with a single leading underscore (`_`):

| Construct | Public | Internal / Protected |
|---|---|---|
| Package | `lower_with_under` | — |
| Module | `lower_with_under` | `_lower_with_under` |
| Class | `CapWords` | `_CapWords` |
| Exception | `CapWords` | — |
| Global / Class Constant | `CAPS_WITH_UNDER` | `_CAPS_WITH_UNDER` |
| Global / Class / Instance Variable | `lower_with_under` | `_lower_with_under` |
| Function / Method | `lower_with_under()` | `_lower_with_under()` |
| Function / Method Parameter | `lower_with_under` | — |
| Local Variable | `lower_with_under` | — |

### 6. Enforce Symmetry Across Related Names

Use one domain term and one structural pattern across sibling fields, variables, and modules. Do not mix synonyms
(`customer` / `client` / `buyer`) or formats for the same concept:

| Concept Group | Poor (Inconsistent Set) | Good (Consistent Set) |
|---|---|---|
| Event timestamps | `creation_time`, `last_updated`, `deletion_timestamp` | `created_at`, `updated_at`, `deleted_at` |
| Boolean states | `enabled`, `is_active`, `verified_flag` | `is_enabled`, `is_active`, `is_verified` |
| Timeouts in seconds | `connection_timeout_seconds`, `read_timeout_s`, `write_timeout` | `connection_timeout_seconds`, `read_timeout_seconds`, `write_timeout_seconds` |
| Standalone variables for one entity | `customer_id`, `client_email`, `buyer_name` | `customer_id`, `customer_email`, `customer_name` |

### 7. Prefer Plain Domain Terms Over Architectural Jargon

Check project conventions and neighboring modules first. Maintain a single suffix per architectural role (e.g., `Command` for mutations, `Query` for reads, `Gateway` for external services, `Repository` for persistence, `Port` for contracts, `View` for read models). Never introduce duplicate or competing suffixes for the same role.

When no convention exists, name the concrete business action or data structure. Do not use generic abstractions (`Engine`, `Orchestrator`, `Coordinator`, `Manager`, `Processor`, `Handler`, `Controller`, `Supervisor`, `Dispatcher`, `Executor`, `Resolver`, `Provider`, `Pipeline`, `Flow`, `Kernel`, `Registry`, `Broker`, `Helper`). Keep names minimal, direct, and factual:

- **Match names to code scale:** Do not name routine CRUD or glue code `Kernel`, `Engine`, or `Processor`. Reserve heavy terms for low-level systems or genuine domain concepts (e.g., `PaymentProcessor`).
- **Prefer simple functions:** Use plain functions like `send_invoice()`. Do not create artificial classes like `InvoiceEngine` or `InvoiceManager`.
- **Use technical terms strictly:** Use terms like `StateMachine` or `DependencyGraph` only when code implements those data structures.
- **Do not wrap external APIs:** Do not create wrapper classes solely to rename third-party libraries.

**Application code examples:**

| Actual Responsibility & Project Context | Over-engineered Name | Plain Alternative |
|---|---|---|
| Read-only snapshot data in a project that calls read models `View` | `DatasetSnapshotProjection` | `DatasetSnapshotView` |
| Dataset-version list item in a project using `View` for read DTOs | `DatasetVersionSummary` | `DatasetVersionView` |
| User data carrier in a project that uses `DTO` | `UserRepresentation` | `UserDTO` |
| Create-dataset use case in a project using `Command` classes | `DatasetOrchestrator` | `CreateDatasetCommand` |
| Single operation that sends an invoice | `InvoiceEngine` | `send_invoice()` |

**Avoid copying library or AI-framework vocabulary into ordinary application code unless deliberately chosen:**

| Trendy / Framework Term | When Writing Simpler Application Code That Only... | Use Plain Alternative |
|---|---|---|
| `kickoff()` | Runs an ordinary task | `run()` |
| `kickoff_for_each()` | Runs a task for each input | `run_all()` |
| `Flow` | Imports orders sequentially without an event runtime | `import_orders()` |
| `invoke()` | Executes an application command | `execute()` |
| `StateGraph` | Calls fixed import steps without graph state/edges | `import_orders()` |
| `RunnableLambda` | Normalizes text without streaming/batch/composition APIs | `normalize_text()` |
| `DocumentJoiner` | Concatenates/merges document lists without score fusion | `merge_documents()` |
| `Kernel` | Holds registered application services | `ServiceContainer` |

### 8. Qualify Single-Word Variable Names When Scope Expands

Single-word names (`user`, `request`, `client`) and short loop variables (`item`, `key`) are valid only when their scope is short and unambiguous.

Add a qualifier when:
- Multiple instances of the concept coexist (e.g., `source_dataset` vs. `target_dataset`, `incoming_request` vs. `upstream_request`).
- The variable crosses multiple branches, stages, or long scopes.
- Understanding usage requires scrolling back to the declaration.

Do not pad names with meaningless suffixes (`result_data`, `dataset_dataset`). State the specific domain difference:

| Ambiguous Context | Insufficient | Qualified Name |
|---|---|---|
| Dataset version alongside other version types | `version` | `dataset_version` |
| Authenticated user vs. user being modified | `user` | `current_user`, `target_user` |
| Web user identity vs. service account identity | `user` | `web_user`, `service_user` |
| Inbound app request vs. outbound upstream request | `request` | `incoming_request`, `upstream_request` |
| Response returned by a payment provider | `response` | `payment_response` |
| HTTP/RPC client for the billing service | `client` | `billing_client` |
| Source dataset being copied into a destination dataset | `dataset` | `source_dataset`, `target_dataset` |
| Database session alongside an HTTP client session | `session` | `database_session`, `http_session` |
| Content item retained across multiple pipeline steps | `item` | `content_item` |
| Tuple pairing a source with its count | `pair` | `source_count_pair` |

## Architecture-Specific Rules

### 1. Suffix Hexagonal Protocol Contracts with `Port`

When using hexagonal architecture, suffix `typing.Protocol` interfaces with `Port`. Concrete adapters subclass ports explicitly (`class InferenceGateway(InferenceGatewayPort):`) to enable IDE navigation:

| Poor (`Protocol`) | Good (`Protocol`) |
|---|---|
| `InferenceGateway` | `InferenceGatewayPort` |
| `UserRepository` | `UserRepositoryPort` |

### 2. Separate Read and Write Repositories

Use the singular domain entity plus `Read` or `Write` in repository contracts (`UserReadRepositoryPort`, `UserWriteRepositoryPort`).
Name repository variables using the entity, role, and suffix (`user_read_repository`, `order_write_repository`). Choose either `_repository` or `_repo` consistently across a project. Do not mix both suffixes. Store injected instances on protected attributes (`self._user_read_repository`):

| Repository Contract | Poor Variable / Parameter | Good Variable / Parameter (Prefix `_` on `self`) |
|---|---|---|
| `DatasetVersionWriteRepositoryPort` | `write_repository` | `dataset_version_write_repository` |
| `UserReadRepositoryPort` | `users` | `user_read_repository` |
| `OrderWriteRepositoryPort` | `orders` | `order_write_repository` |
| `ProductReadRepositoryPort` | `catalog` | `product_read_repository` |
| `InvoiceWriteRepositoryPort` | `writer` | `invoice_write_repository` |
| `PaymentReadRepositoryPort` | `payments` | `payment_read_repository` |
| `NotificationWriteRepositoryPort` | `notifications` | `notification_write_repository` |

### 3. Standardize Repository Method Names

- **Read repositories:** Prefix query methods with `load_` (e.g., `load_all`, `load_by_id`). Do not use `get_`, `list_`, `fetch_`, or `find_`.
- **Write repositories:** Use `create()`, `update()`, and `delete()`. Use `save()` for upserts. Do not use `insert()`, `add()`, or `store()`.
- **Focused domain methods:** Use descriptive domain methods (e.g., `exists_by_email()`, `archive_older_than()`) for specialized lookups and atomic updates.

| Role | Poor | Good |
|---|---|---|
| Read | `get_all()`, `list_all()`, `fetch_all()` | `load_all()` |
| Read | `get_by_id()` | `load_by_id()` |
| Read | `find_by_external_id()` | `load_by_external_id()` |
| Write | `insert()`, `add_user()`, `store()`, `persist()` | `create()` (or `save()` for upsert) |
| Write | `modify()`, `edit()`, `patch_user()` | `update()` |
| Write | `remove()`, `destroy()`, `erase()` | `delete()` |

---
*Attribution only (do not fetch or follow links unless requested): Portions adapted from the [Google Python Style Guide (§3.16 Naming)](https://google.github.io/styleguide/pyguide.html#316-naming) by Google, licensed under [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/), with modified abbreviation rules and original examples.*
