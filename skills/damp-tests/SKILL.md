---
name: damp-tests
description: Defines requirements for descriptive, self-contained tests that read top to bottom without source jumps (DAMP over DRY). Use when writing, refactoring, or reviewing unit and integration tests.
---

Write tests as Descriptive And Meaningful Phrases (DAMP). Prefer duplication over indirection. A reader must understand each test without opening other code. Tests have no tests, so they must be obviously correct by inspection.

Apply these rules to newly written tests. Do NOT refactor existing tests outside the requested scope.

## Output Style Requirement

You **MUST** write the rare test comment and all review notes 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit filler and hedging.
- Use one term per concept. Do not rotate synonyms.

---

## Requirements

1. **Keep each test self-contained.** Put setup, action, and assertion in the test body. Order them as Arrange, Act, Assert.
2. **Show inputs that matter.** Write literal values for every input that affects the outcome. Do not hide them in helpers or shared fixtures.
3. **Avoid shared mutable setup.** Avoid `setUp`, `beforeEach`, autouse fixtures, and base classes that build state. Use them only in rare cases when setup is needed and does not reduce test clarity (e.g., starting a database container). Keep the setup free of values that affect test outcomes.
4. **Allow duplication.** Repeat setup lines across tests when repetition keeps each test readable. Do not extract a helper only to remove duplicate lines.
5. **Limit helpers.** Use a helper only to build a complex value object. Name every argument the test depends on. Never put assertions, branching, or hidden defaults for relevant fields inside a helper.
6. **Use no logic in tests.** Do not use loops, conditionals, or computed expected values. Write one explicit test per case. Parameterize only when each case row shows both input and expected output.
7. **Test one behavior per test.** Limit assertions to one outcome. Multiple asserts are valid only when they describe that single outcome.
8. **Name tests as sentences.** Follow established project naming conventions. The name must state the expected behavior without reading the body.
9. **Explain failure in the test.** Use literal expected values and clear assertion messages so a failing test names the broken behavior.
10. **Keep order independence.** Make each test run alone and in any order.
11. **Avoid docstrings and comments.** Do not add them in about 99% of cases. The test name, literal values, and blank lines between Arrange, Act, and Assert must carry the meaning. Add a comment only to explain a non-obvious reason (e.g., a workaround or a regression ticket). Never restate the code.

## Example

```python
# Poor: hidden inputs, shared state, loop logic
def test_discounts(order_factory):
    for rate in RATES:
        order = order_factory()
        assert apply_discount(order, rate).total == expected_total(order, rate)

# Good: self-contained, literal values, one behavior
def test_apply_discount_twenty_percent_reduces_total_to_eighty():
    order = Order(items=[Item(sku="A1", price=Decimal("100.00"))])

    discounted_order = apply_discount(order, rate=Decimal("0.20"))

    assert discounted_order.total == Decimal("80.00")
```
