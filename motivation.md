---

## GSoC contributor checklist

### Context & motivation
<!-- Why are you addressing this problem? How did you find it? What interests you about it? -->
<!-- ⚠️ Write this yourself. Do not use LLM/AI tools for this section — we want to hear your voice and your understanding. -->

**Why I'm addressing it**

I wanted to find a useful improvement in Mesa that no open PR already covered.

**How I found it**

- I went through open issues and PRs, then read `mesa/discrete_space/cell.py`.
- Both `get_neighborhood()` and `_neighborhood()` use `functools.cache`, but `_neighborhood()` is only ever called through `get_neighborhood()`.

**What caught my attention**

In my benchmark, the inner cache had zero hits and built up 20,000 entries. It was holding memory and adding overhead for nothing.

**Why it interests me**

It's a small, contained fix. It saves memory without changing the public API or how neighborhood queries behave.

**How I investigated it**

- Compared the two implementations on memory and execution time.
- Ran regression tests to confirm neighborhood invalidation still works.

### What I learned
As a new contributor, the best part was digging through Mesa's codebase until I found the exact lines behind an unnecessary cache. What looked like a small optimization took hours of tracing function calls, reading existing code, and working out how the pieces fit together. I used AI to help me understand unfamiliar code, but I still had to check the behavior myself and tie each explanation back to what the project was actually doing. Finding and fixing a real issue in a real codebase felt great.

On the technical side, I learned how `functools.cache` holds references to its arguments, including `self`, and how a redundant cache can cost memory without giving anything back. Tracing `Cell.get_neighborhood()` and `Cell._neighborhood()` showed me why the inner cache never got a hit: the outer cache always answered first. I also learned to back an optimization with evidence. I compared memory and execution time, tested cold and warm runs, and ran regression tests to make sure nothing broke.

I also saw how open-source contribution works in practice. Finding an issue is only the start. You have to understand the surrounding code, check existing issues and PRs, follow the project's conventions, write tests, and be honest about limitations. A three-line change can need a lot of investigation before it's safe.

Mostly, I learned to read unfamiliar code patiently before writing any. I have a lot more to learn about Mesa, but I'm now more comfortable finding my way around an established codebase, asking sharper questions, and using AI to learn rather than to skip understanding the code.

- How `functools.cache` keys results on arguments, and that caching a method keeps a reference to `self`, so objects can live longer than they should.
- To trace who calls a function before deciding its cache is redundant.
- To benchmark fairly: same workload, interleaved runs, cold and warm cache measured separately.
- That removing an optimization means re-checking what depends on it. Here, that was neighborhood invalidation after cells connect or disconnect.
- To check existing issues and PRs before picking one, so I don't duplicate someone's work.

**What was harder than I expected**

- Finding an open issue. Most easy bugs already had competing PRs.
- Spotting the redundant cache. One function didn't show it. I had to trace the callers and look at cache stats.
- The deleted lines were few, but proving the change safe took much longer than writing it.
- The gains were clear for memory and cold runs but small for warm runs. That showed me why testing different conditions matters.
- Some example tests failed because the `solara_test` fixture was missing. Running them on the unchanged baseline showed they were environment failures, not regressions I caused.

### Learning repo
🔗 My learning repo: https://github.com/lahari936/GSoC-learning-space
🔗 Relevant model(s): n/a — this fix came out of reading and profiling `discrete_space` rather than from a model

### Readiness checks
- [x] This PR addresses an agreed-upon problem (linked issue or discussion with maintainer approval), **or** is a small/trivial fix
- [x] I have read the [contributing guide](https://github.com/mesa/mesa/blob/main/CONTRIBUTING.md) and [deprecation policy](https://github.com/mesa/mesa/blob/main/CONTRIBUTING.md#deprecation-policy)
- [x] I have performed a self-review: I reviewed my own PR as if I were a reviewer and left comments on anything that needs explanation
- [x] Another GSoC contributor has reviewed this PR: @AshNicolus
- [x] Tests pass locally (`pytest --cov=mesa tests/`)
- [x] Code is formatted (`ruff check . --fix`)
- [x] If applicable: documentation, examples, and/or migration guide are updated — n/a, `_neighborhood` is private and no public behaviour changes

**Test run:** `pytest --cov=mesa tests/` → 861 passed, 1 failed, 86% coverage. The failure is
`tests/experimental/test_actions.py::TestOnIdle::test_suppressed_wake_is_logged_at_debug`, which is
pre-existing: a clean `upstream/main` worktree gives 859 passed and the same single failure. It passes
in isolation and only fails in full-suite order, so it looks like a test-isolation issue in logging
capture rather than anything in this PR.
