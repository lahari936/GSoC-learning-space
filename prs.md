# Pull requests

Status as of 2026-11-26.

## mesa/mesa

### Merged (10)

| PR | merged | change |
|---|---|---|
| [#4061](https://github.com/mesa/mesa/pull/4061) | 2026-11-20 | Keep agent ids and field names in `JSONDataRecorder` numpy snapshots |
| [#4072](https://github.com/mesa/mesa/pull/4072) | 2026-11-19 | Make `JSONDataRecorder` honour `window_size` |
| [#3970](https://github.com/mesa/mesa/pull/3970) | 2026-11-10 | Answer the meta-agent class lookup from the dict key, not from a live agent |
| [#3956](https://github.com/mesa/mesa/pull/3956) | 2026-11-07 | Spread any sized sequence across agents, not four hard-coded types |
| [#3934](https://github.com/mesa/mesa/pull/3934) | 2026-11-07 | Remove the dead inner cache on `Cell._neighborhood` |
| [#4026](https://github.com/mesa/mesa/pull/4026) | 2026-11-02 | Explain a full space instead of leaking `IndexError` from `random.choice` |
| [#4012](https://github.com/mesa/mesa/pull/4012) | 2026-10-28 | Stop pytest 9.2 from failing at plugin load on solara's websockets warning |
| [#4011](https://github.com/mesa/mesa/pull/4011) | 2026-10-28 | Run docstring doctests in CI as their own job |
| [#4010](https://github.com/mesa/mesa/pull/4010) | 2026-10-28 | Fix the docstring examples that cannot run under doctest |
| [#3974](https://github.com/mesa/mesa/pull/3974) | 2026-10-24 | Refuse an `Event` that is already queued in the same `EventList` |

### Open (2)

| PR | opened | change | state |
|---|---|---|---|
| [#4000](https://github.com/mesa/mesa/pull/4000) | 2026-10-27 | Anchor a `ContinuousState`'s rate whenever the state is assigned | approved by codebreaker32, follow-up pushed, waiting for final review |
| [#4060](https://github.com/mesa/mesa/pull/4060) | 2026-11-10 | Let `SQLDataRecorder` store values with their own type | checked by arzaanxeng, waiting for maintainer review |

### Closed without merging (3)

| PR | closed | change |
|---|---|---|
| [#4059](https://github.com/mesa/mesa/pull/4059) | 2026-11-10 | Record a copy of mutable values, not a reference to them |
| [#3864](https://github.com/mesa/mesa/pull/3864) | 2026-10-28 | Run docstring doctests in CI via `pytest --doctest-modules` |
| [#3823](https://github.com/mesa/mesa/pull/3823) | 2026-08-30 | Fix `FixedAgent.remove` for unplaced agents |

## mesa/mesa-examples

| PR | state | change |
|---|---|---|
| [#502](https://github.com/mesa/mesa-examples/pull/502) | merged 2026-11-01 | Report an unimportable example as one failing test, not a collection error |
| [#499](https://github.com/mesa/mesa-examples/pull/499) | merged 2026-10-28 | Import `MetaAgents` from its new home in `mesa.experimental` |
| [#498](https://github.com/mesa/mesa-examples/pull/498) | merged 2026-10-28 | Give a duplicated virus its own dna list |
| [#487](https://github.com/mesa/mesa-examples/pull/487) | open | Fix forest_fire notebook compatibility with Mesa 4 |
| [#485](https://github.com/mesa/mesa-examples/pull/485) | closed 2026-08-30 | Add energy expenditure example |
