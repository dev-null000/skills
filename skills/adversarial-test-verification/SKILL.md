---
name: adversarial-test-verification
description: Audits test suites for tautologies, over-mocking, weak assertions, and surviving logic mutants using static trace or git-guarded live sabotage. Use when verifying agent-generated tests, auditing test quality, or checking if tests catch real bugs.
---

Evaluate whether tests genuinely falsify production bugs or merely create an illusion of coverage. Run an anti-tautology audit and construct a **Mutation Kill Matrix** via read-only static trace (default) or git-guarded live sabotage.

Do NOT modify test or production files until the user approves the remediation plan.

## Output Style Requirement

You **MUST** write all audit findings, mutation tables, and remediation instructions 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and hedging.
- Provide concrete `file:line` references for every production mutation and test assertion.
- Use one term per concept. Do not rotate synonyms.

---

## Step 1: Anti-Tautology & Mock Boundary Audit

Inspect the target test files and corresponding production modules against these 4 rules:

1. **No Tautological Oracles:** Flag tests that copy production formulas, regexes, or queries into the `assert` statement. Require literal expected values or independent invariants (e.g., round-trip serialization, conservation of totals, directional bounds):
   ```python
   # Poor (copies buggy production formula into test)
   assert calculate_discount(price, rate) == price * (Decimal("1") + rate)

   # Good (literal expected value + directional bound)
   discounted = calculate_discount(Decimal("100.00"), Decimal("0.20"))
   assert discounted == Decimal("80.00")
   assert Decimal("0") <= discounted <= Decimal("100.00")
   ```
2. **Boundary-Only Fakes & Mocks:** Allow fakes, stubs, or mocks **only** at external I/O or architectural port boundaries (`*Port`, HTTP clients, clocks, gateways). Flag any test that mocks the system under test, internal domain services, or private `_methods`.
3. **Strong State & Value Assertions:** Flag superficial assertions (`assert result is not None`, `assert status == 200`, or bare `mock.assert_called()`) that omit exact return payloads, persisted database/repository state, or raised exception messages.
4. **Boundary & Failure Coverage:** Verify explicit test cases for:
   - Boundary thresholds (`0`, `-1`, empty collections, `None`, off-by-one limits `<` vs. `<=`).
   - Scenarios from user-provided incident logs, stack traces, or payload snippets (if provided).
   - Authorization denials and cross-tenant access attempts.
   - Idempotency, duplicate inputs, and downstream port failures.

---

## Step 2: Hybrid Sabotage Verification (Mutation Kill Matrix)

Select **5 to 8 high-risk lines** in the changed production code (conditional guards, boundary comparisons, state mutations, authorization checks, error handlers, return values).

### Mode A: Static Sabotage Trace (Default — 100% Read-Only)
Use Mode A by default:
1. Formulate a concrete single-line fault (`Mutant`) for each selected production line (e.g., flip `<=` to `<`, invert a boolean guard, remove a repository write, return an empty list).
2. Trace execution against the existing test suite without editing files.
3. Record whether a specific `test_file::test_name:line` assertion fails (`KILLED`) or if all existing tests still pass (`SURVIVED`).

### Mode B: Live Sabotage Execution (On User Request or Isolated Branch Only)
Run Mode B **only** when the user explicitly requests live sabotage or when executing inside an isolated subagent branch/worktree:
1. **Git Safety Gate (Mandatory):** Run `git status --porcelain -- <target-prod-files>`. If any target production file has unstaged changes, **abort live sabotage immediately** and fall back to Mode A (or ask the user to stage/commit first).
2. **Single-Mutant Loop:** For each mutant, apply 1 edit to the production file, run the targeted test command, record `KILLED` (test failed) or `SURVIVED` (test passed), and immediately restore the file using `git checkout -- <file>`.
3. **Post-Run Verification:** Run `git diff -- <target-prod-files>` after the loop to verify zero leftover mutations exist on disk.

---

## Step 3: Output the Test Verification Report

Present the audit in chat using this Markdown structure and wait for user approval before editing any files:

```markdown
### Adversarial Test Verification Report

**Scope:** `<production & test files>` | **Mode:** `<Static Trace | Live Sabotage>` | **Result:** `<K>/<N>` mutants killed, `<S>` survived

#### 1. Surviving Mutants (Unprotected Production Logic)
| ID | Production Location | Sabotage Mutation | Why Existing Tests Pass | Required Test & Assertion Fix |
|---|---|---|---|---|
| M-01 | `app/domain/bid.py:42` | `amount <= limit` -> `amount < limit` | Tests only check `amount = 90` (`limit = 100`). Exact boundary `100` is untested. | Add `test_submit_bid_at_exact_limit_succeeds` asserting `status == ACTIVE` for `amount = 100`. |
| M-02 | `app/services/order.py:78` | Remove `if user.id != order.owner_id: raise ForbiddenError` | All tests pass `user.id == order.owner_id`. | Add `test_cancel_order_by_non_owner_raises_forbidden` asserting `pytest.raises(ForbiddenError)`. |

#### 2. Tautological, Over-Mocked, or Weak Tests
| ID | Issue Type | Test Location | Evidence | Proposed Remediation |
|---|---|---|---|---|
| T-01 | Over-Mocking | `tests/test_order.py:55` | Mocks internal `_calculate_tax()` on the service under test. | Remove internal mock. Pass real inputs and assert final `order.total_amount`. |
| T-02 | Weak Assertion | `tests/test_order.py:89` | Asserts `result is not None` without checking persisted state. | Assert `fake_order_repo.load_by_id(result.id).status == OrderStatus.PAID`. |

#### 3. Verified Killed Mutants (Strong Coverage)
- `app/domain/bid.py:30` (`not supplier.is_active` guard) -> Killed by `tests/test_bid.py:24` (`test_inactive_supplier_rejected`).

#### Next Steps
Wait for developer confirmation before adding or modifying tests.
```
