---
name: folder-structure-preview
description: Renders a diff-annotated tree of directories, files, classes, and method signatures to verify architecture and naming before writing code. Use when planning new modules, features, or structural refactorings, or when requested by the user.
---

Before creating or refactoring modules, classes, or public signatures, output a unified **Structure & Symbol Preview** to verify module boundaries, layer placement, and naming conventions before generating code.

## Output Style Requirement

You **MUST** write all architectural questions, design notes, and rationale outside the `diff` block 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Rules

1. **Unify filesystem and code symbols in one tree:**
   - Render directories (with trailing `/`), files (e.g., `.py`), classes/protocols/enums/type aliases, injected collaborator attributes (including protected `_` attributes), and public functions, methods, properties, and fields with parameter and return types.
   - Format each declaration on a single line: place key decorators inline and omit `def` and trailing colons (e.g., `@dataclass class Bid`, `@property is_active(self) -> bool`).
   - Omit trivial constructors/dunder methods (e.g., `__init__`), private helper methods, and package boilerplate files (e.g., `__init__.py`) unless they are the primary subject of the change.
   - Collapse unaffected directories, unaffected files, and wholly removed files (`- file.py`) to a single line; expand only unaffected symbols that establish local naming or architectural context (such as a neighboring `View` or key fields on a modified class), never dumping the entire repository tree.

2. **Mark planned changes in `diff` format:**
   - Wrap the tree in a ```` ```diff ```` code block so Markdown renderers highlight additions and removals.
   - Prefix added lines with `+`, removed lines with `-`, and unchanged context lines (including existing parent directories, files, and classes that only have child additions or removals) with two spaces (`  `).
   - Represent modified or renamed declarations/signatures as a `-` removal line followed by a `+` addition line (never invent `~` or `M` markers).
   - Repeat the `+` or `-` marker next to the symbol or filename inside the tree so the diff remains unambiguous in plain text.

3. **Verify architecture, scope, and naming before presenting:**
   - **Follow established architecture or ask:** Derive module boundaries and layer placement from user instructions, active skills, project documentation, or existing code. If the architectural style cannot be determined from any of these sources, ask the user for architectural requirements before proposing a structure.
   - **Keep changes minimal and naming consistent:** Propose the smallest coherent change and never invent grandiose abstractions. Follow naming and style conventions from active skills, project documentation, existing code, and linter/formatter configs; when none specify a rule, use the language's standard style guide (e.g., PEP 8 for Python) as the baseline.
   - **Wait for approval:** Unless instructed to proceed immediately, stop after printing the preview and ask for confirmation before writing a detailed implementation plan, creating, or modifying files.

## Format Example

```diff
  procurements/
  └── modules/
      ├── bidding/
      │   ├── domain/
      │   │   ├── entities.py
      │   │   │   ├── class BidStatus(StrEnum)
      │   │   │   └── @dataclass class Bid
      │   │   │       ├── bid_id: BidId
      │   │   │       ├── tender_id: TenderId
      │   │   │       ├── supplier_edrpou: str
      │   │   │       ├── amount_uah: Decimal
      │   │   │       ├── submitted_at: datetime
+     │   │   │       ├── + @property is_active(self) -> bool
+     │   │   │       ├── + disqualify(self, reason: str) -> None
-     │   │   │       └── - check_valid(self) -> bool
      │   │   └── ports.py
      │   │       ├── class BidReadRepositoryPort(Protocol)
      │   │       │   ├── load_by_id(self, bid_id: BidId) -> Bid | None
+     │   │       │   └── + load_active_by_tender_id(self, tender_id: TenderId) -> list[Bid]
+     │   │       ├── + class BidWriteRepositoryPort(Protocol)
+     │   │       │   ├── + create(self, bid: Bid) -> None
+     │   │       │   └── + update(self, bid: Bid) -> None
+     │   │       └── + class EdrpouGatewayPort(Protocol)
+     │   │           └── + is_supplier_active(self, supplier_edrpou: str) -> bool
      │   ├── application/
      │   │   ├── views.py
      │   │   │   └── @dataclass(frozen=True) class BidDisclosureView
+     │   │   └── + submit_bid.py
+     │   │       ├── + @dataclass(frozen=True) class SubmitBidCommand
+     │   │       │   ├── + tender_id: TenderId
+     │   │       │   ├── + supplier_edrpou: str
+     │   │       │   └── + amount_uah: Decimal
+     │   │       └── + class SubmitBidCommandHandler
+     │   │           ├── + _bid_write_repository: BidWriteRepositoryPort
+     │   │           ├── + _edrpou_gateway: EdrpouGatewayPort
+     │   │           └── + execute(self, command: SubmitBidCommand) -> BidId
      │   └── infrastructure/
+     │       ├── + repositories.py
+     │       │   └── + class SqlBidWriteRepository(BidWriteRepositoryPort)
+     │       │       ├── + create(self, bid: Bid) -> None
+     │       │       └── + update(self, bid: Bid) -> None
-     │       └── - legacy_bid_manager.py
      ├── tendering/
      ├── qualification/
      ├── contracting/
      └── state_financial_audit/
```
