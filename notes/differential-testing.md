# Notes: differential testing as a way to find bugs in mesa

A method that has now produced three results in mesa, two bugs and one
clean bill of health. Writing it down because the useful part is the
procedure, not the bugs.

## The idea

When a codebase has two or more implementations that are supposed to honour
the same contract, run identical operations against each and diff the
results. Any disagreement is either a bug in one of them or a gap in the
contract. You do not need to know what correct looks like in advance, which
is the expensive part of ordinary test writing.

mesa has several of these families:

- five discrete spaces implementing `DiscreteSpace`
- `InMemoryStore` and `DiskStore` implementing the `Store` protocol
- `AgentSet` and `_HardKeyAgentSet`

## The procedure

1. Write each check as a small function returning either `("ok", value)` or
   `("raise", ExceptionName)`. Never let the probe itself throw; catching
   and recording the exception type is half the signal.
2. Run every check against every implementation.
3. Print a table and flag rows where the set of results has more than one
   distinct value.
4. For each flagged row, read the source of both paths before deciding
   which side is wrong. Sometimes the divergence is deliberate and the
   contract just fails to say so.

## What it found

**mesa#4026 — five discrete spaces, eight invariants.** Seven invariants
agreed everywhere. One did not: `select_random_empty_cell()` on a full space
raised a descriptive `ValueError` on the three `Grid` subclasses and a raw
`IndexError` from `random.py` on `Network` and `VoronoiGrid`. Root cause was
that `Grid` overrides the method and the base was never given the same
guard.

**`InMemoryStore` vs `DiskStore` — twelve protocol probes.** Eleven matched
exactly, including the exception type for every error path. The one
divergence, `write_scenarios` called twice, turned out to be deliberate and
documented on the `DiskStore` side. No bug. Worth recording that a probe run
finding nothing is a result too.

**mesa#3974** came from the same habit applied differently: taking a
documented promise in `EventList` and checking it held under randomised
operation sequences.

## Mistakes I made running these

- A probe that silently skips every case reports zero mismatches, which
  looks exactly like a pass. One early run on `ContinuousState` had a
  `if predicted is None: continue` that skipped all 120 trials. Always
  print the number of cases actually compared.
- `id(object())` is reused after the object is collected, so using it to
  name temporary directories gave every `DiskStore` the same path and a
  pile of `FileExistsError`s that looked like a mesa bug. Use a counter.
- Guessing an enum member name or a function signature produces a uniform
  failure across every implementation, which reads as a finding until you
  look at the message. Check the error before believing the table.
