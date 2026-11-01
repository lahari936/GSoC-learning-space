# Investigation record: mesa#4026

Factual record of the work behind
[mesa/mesa#4026](https://github.com/mesa/mesa/pull/4026).
Reflection on it lives in the PR's GSoC checklist, not here.

## Problem

`select_random_empty_cell()` on a space with no empty cells:

| space | result |
|---|---|
| `OrthogonalMooreGrid` | `ValueError: Grid is completely full. No empty cells available.` |
| `OrthogonalVonNeumannGrid` | same |
| `HexGrid` | same |
| `Network` | `IndexError: Cannot choose from an empty sequence` |
| `VoronoiGrid` | `IndexError: Cannot choose from an empty sequence` |

The `IndexError` comes from inside `random.py`, names no mesa object, and
does not say the space is full.

## Reproducer

```python
import networkx as nx
from mesa import Model
from mesa.discrete_space import CellAgent, Network

model = Model()
space = Network(nx.cycle_graph(4), random=model.random)
for cell in space.all_cells:
    CellAgent(model).cell = cell
space.select_random_empty_cell()
# IndexError: Cannot choose from an empty sequence
```

## Root cause

`Grid.select_random_empty_cell` (`grid.py:322`) catches the `IndexError` and
re-raises with a message. `DiscreteSpace.select_random_empty_cell`
(`discrete_space.py:190`) was one line and did not:

```python
return self.random.choice(list(self.empties))
```

`Network` and `VoronoiGrid` inherit it unchanged.

## Fix

Guard the empty case in the base method and raise `SpaceException`. The list
is materialised by the original line anyway, so the check costs a `len`.

Exception type chosen against `docs/development/standards.md`: use a
`MesaException` subclass when it "adds domain context or hides internals",
and pick the most specific existing one. The raw `IndexError` leaks an
internal; `SpaceException` already exists; no new type added.

## Scope deliberately left out

`Grid.select_random_empty_cell` still raises `ValueError` for the same
condition. Changing it is breaking, it is pinned by
`test_infinite_loop_on_full_grid`, and in mesa#4001 a maintainer ruled that
`ValueError` is the wrong type for a method that receives no value, with
that thread moving such cases to `SpaceException`. The grid path belongs in
that migration.

## Verification

| check | result |
|---|---|
| both new tests on `main` | fail with `IndexError` |
| both new tests with the fix | pass |
| `tests/discrete_space/` | 140 passed |
| `pytest tests/` less playwright viz | 1014 passed |
| `ruff check` / `ruff format --check` | clean |

The single unrelated failure in the full run,
`test_actions.py::TestOnIdle::test_suppressed_wake_is_logged_at_debug`, is
pre-existing and tracked in mesa#3935.

## How it was found

Differential probing across all five space types, eight invariants each.
Method written up in [differential-testing.md](differential-testing.md).
