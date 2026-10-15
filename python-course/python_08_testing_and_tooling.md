# Testing & Tooling (this fills a gap flagged earlier in your prep)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · **Testing/Tooling**

---

Your resume leans heavily on PyTest, CI integration, and automated regression testing — "automated 90% of system validation in Python; integrated regression tests into CI, reducing post-deployment issues by 40%" is a real, strong bullet. This file gives you the precise vocabulary to talk about that work at the depth an interviewer might actually probe, not just name the tools.

## PyTest Fixtures — the core mechanism

```python
import pytest

@pytest.fixture
def db_connection():
    conn = create_test_db()
    yield conn                    # everything before yield = setup; everything after = teardown
    conn.close()

def test_insert(db_connection):   # pytest matches the parameter NAME to a fixture — dependency injection
    db_connection.insert("row")
    assert db_connection.count() == 1
```
- A fixture is PyTest's dependency-injection mechanism: a test function requesting a parameter named `db_connection` automatically receives whatever the `db_connection` fixture produces — no manual setup call needed in the test body itself, and PyTest resolves the wiring by name matching.
- **The `yield`-based fixture is exactly the same shape as the `@contextmanager` pattern from file 03** — setup before `yield`, teardown after, and teardown is guaranteed to run even if the test itself fails (PyTest handles this the same way `contextlib.contextmanager`'s `try`/`finally` does). Worth explicitly naming this parallel if asked to explain fixtures — it signals you understand the *mechanism*, not just "fixtures do setup and teardown."
- **Fixture scope** (`@pytest.fixture(scope="module")`, `"session"`, etc.) controls how often a fixture actually re-runs — `scope="function"` (the default) re-runs for every test; `scope="session"` runs once for the entire test run and is shared. Real trade-off worth naming: a wider scope (session) is faster (expensive setup, like spinning up a test database, happens once) but risks **test pollution** — state leaking between tests that were supposed to be independent, exactly the kind of subtle bug that makes a previously-passing test fail only when run in a particular order.

## `@pytest.mark.parametrize` — one test, many inputs

```python
@pytest.mark.parametrize("input,expected", [
    (2, 4), (3, 9), (-2, 4), (0, 0),
])
def test_square(input, expected):
    assert square(input) == expected
```
Runs the same test body once per parameter set, each reported as a **separate, individually-named test result** — a failing case at `input=-2` is immediately identifiable, not buried inside a single test with an internal loop and one aggregated pass/fail. This distinction (separate reported results vs. a hand-rolled loop inside one test) is worth stating if asked "why not just write a `for` loop in the test" — the real answer is granular, individually-actionable failure reporting, not just "less code."

## Mocking — `unittest.mock`

```python
from unittest.mock import patch, MagicMock

@patch("myapp.network.send_request")
def test_retry_logic(mock_send):
    mock_send.side_effect = [ConnectionError, ConnectionError, {"status": "ok"}]
    result = call_with_retry()
    assert mock_send.call_count == 3
    assert result == {"status": "ok"}
```
- **What mocking is actually for, precisely:** isolating the code under test from its real dependencies (network calls, real hardware, real timing) — `@patch` performs the monkey-patching from file 07's trivia list, but in the specifically sanctioned, good use case: swapping out `myapp.network.send_request` for a fake during the test, then automatically restoring the real one afterward.
- **`side_effect` vs. `return_value`:** `return_value` gives a fixed value every call; `side_effect` (a list, exception, or callable) lets you script a *sequence* of different behaviors across successive calls — exactly what's needed to simulate "fails twice, then succeeds," the canonical retry-logic test shape above.
- **The precise trade-off worth stating if asked "should you mock this":** over-mocking makes tests fast and isolated but can drift from reality (the mock's behavior doesn't actually match what the real dependency does, so the test passes while the real system is broken) — a genuine, common failure mode of test suites that mock too aggressively. Mock true external dependencies (network, hardware, other services); avoid mocking the actual logic you're trying to verify.

## Structuring a Test Suite for CI

Given your CI-integration background, this is worth having a clean, general framing ready:
- **Fast, isolated unit tests** (mocked dependencies, no real I/O) run on every commit/PR — this is where PyTest + fixtures + mocking above live.
- **Slower integration tests** (real database, real network calls against a test environment) run less frequently — on merge to main, or nightly — since they're too slow/flaky for tight per-commit feedback.
- **Regression tests specifically** (the ones tied to previously-found bugs) deserve their own marker/tag (`@pytest.mark.regression`) so they can be run as a distinct, trackable suite — directly the kind of structure that would produce a concrete "reduced post-deployment issues by 40%" result, since it's measuring a *specific*, named category of test rather than an undifferentiated pile.
- **Test markers** (`@pytest.mark.slow`, custom marks registered in `pytest.ini`/`pyproject.toml`) let CI selectively run subsets (`pytest -m "not slow"` for a fast pre-commit pass, full suite on a scheduled nightly run) — a real, practical technique for balancing fast feedback against thorough coverage.

## `assert` — plain, and why PyTest's is special

PyTest lets you write plain `assert expected == actual` (no special assertion-method API like `assertEqual` from the older `unittest` module) and still get a **detailed failure report** — showing the actual values on both sides of a failed comparison. This works via **assertion rewriting**: PyTest rewrites the test module's bytecode at collection time to capture intermediate values for a rich failure message, rather than relying on you calling a specially-named method. Worth knowing this is genuinely happening (not just "PyTest is smart about asserts") if asked why plain `assert` gives such detailed output compared to a bare Python `assert` run outside PyTest.

## Practice signal

If asked to describe your testing background, the strongest framing ties a specific technique to a specific outcome you can defend: *"I used PyTest fixtures for setup/teardown of test environments, parametrize for systematic input coverage, and mocking to isolate hardware/network dependencies — structured with markers so CI could run a fast subset per-commit and the full regression suite on a schedule, which is what got us the drop in post-deployment issues."* That's concrete and checkable, not just a tool-name list.

---

## Related Modules

- [Python 03 — Decorators/Generators (contextmanager == fixture shape)](python_03_functional_features_and_decorators.md)
