# Models

Empty. This is the part of the repo that matters most and the part I have
not done yet.

mesa#3536 calls model-building step one, before PRs, and the template README
says the same: *"build models first, contribute second."*

Planned, simplest first:

- [ ✅] a Boltzmann-wealth variant, to get the basic model/agent/step loop down
- [ ] Schelling on `OrthogonalMooreGrid`, for neighbourhood queries and `empties`
- [ ] the same model on `Network`, since the two spaces are where mesa#4026 came from
- [ ] something original, with data collection and a `SolaraViz` app

Each gets its own folder with the code and a README covering what it does,
which mesa features it uses, what was hard, and what I would do differently.
