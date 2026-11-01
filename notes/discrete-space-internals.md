# Notes: how mesa's discrete spaces fit together

Reading notes taken while working on mesa#3934 and mesa#4026. Everything
here was checked against the source or measured, not read off the docs.

## The two layers

`DiscreteSpace` (`mesa/discrete_space/discrete_space.py`) is the abstract base.
`Grid` (`grid.py`) extends it and adds the array-backed machinery that only
makes sense on a regular lattice: property layers, coordinate arithmetic,
`_celllist`.

Five concrete spaces ship:

| class | base | backing |
|---|---|---|
| `OrthogonalMooreGrid` | `Grid` | property layers + coordinate array |
| `OrthogonalVonNeumannGrid` | `Grid` | same |
| `HexGrid` | `Grid` | same |
| `Network` | `DiscreteSpace` | a networkx graph |
| `VoronoiGrid` | `DiscreteSpace` | a point list + KD-tree |

The split matters more than it looks. Anything `Grid` overrides silently
diverges for `Network` and `VoronoiGrid`, which inherit the base version.
That is the shape of mesa#4026: `Grid.select_random_empty_cell` grew a
friendly error for a full space, the base method never did, so two of the
five spaces still leaked `IndexError: Cannot choose from an empty sequence`
straight out of `random.py`.

## Cells and the neighbourhood cache

`Cell` holds `connections: dict[key, Cell]` and `_agents`. Neighbourhood
lookup is two methods, both decorated with `functools.cache`:

```
get_neighborhood(radius, include_center)   ->  CellCollection
    _neighborhood(radius, include_center)  ->  dict[Cell, list[Agent]]
```

`_neighborhood` has exactly one caller, `get_neighborhood` at `cell.py:218`,
and both cache on the same `(self, radius, include_center)` key. So every
call that reaches `_neighborhood` is already a `get_neighborhood` miss, and
the inner cache can never serve a read. Measured on BoltzmannWealth large
(10000 agents, 100x100, 10 steps):

```
Cell.get_neighborhood  CacheInfo(hits=90002, misses=9998, currsize=9998)
Cell._neighborhood     CacheInfo(hits=0,     misses=9998, currsize=9998)
```

Two things worth remembering from that:

1. `functools.cache` on a *method* is one cache for the whole class, keyed
   on `self`. Calling `cache_clear()` from any instance wipes every
   instance's entries, and the cache holds a strong reference to every cell.
2. `Cell.connect()` calls `_clear_cache()` (`cell.py:124`). Building a
   100x100 grid makes 78804 `connect()` calls, so removing one of the two
   caches removed 78804 `cache_clear()` calls from model construction.

## Invariants that hold across all five spaces

Checked by running the same probes against each (see
[differential-testing.md](differential-testing.md)):

- a cell is not in its own `radius=1` neighbourhood
- `include_center=True` puts it back
- `radius=0` and `radius=-1` both raise `ValueError`
- neighbourhood is symmetric: `a in nbh(b)` iff `b in nbh(a)`
- `radius=2` is a superset of `radius=1`
- `empties` agrees with `cell.is_empty`

The only invariant that did not hold uniformly was the full-space error
type, which became mesa#4026.
