# Redoland sample run

A sample world for [Redoland](https://github.com/kilyig/redoland), a village of LLM agents living under food scarcity where every action is a git commit. This repository *is* the run: the engine reads and writes it directly, and the Redoland repo mounts it as a submodule at `runs/sample_run`.

## The world

- 10 founders (seed 123), no premise, default rules: a shared pile of `max(15, round(0.9 × living))` food a year, 10 actions and 6 lines of speech per villager per year, food of the dead returns to the pile.
- Every villager's decisions were made by `claude-sonnet-4-6` through the `claude -p` CLI.
- Simulated between 2026-07-05 and 2026-08-02. Each commit is one villager action; each completed year is tagged `<branch>-y<N>`.

## The two worldlines

**`main`** — 15 years. The founders had six children (Dell, Eron, Nessa, Zinn, Fina, Joren) and seven villagers starved; nobody died in combat. Population at year 15: 9.

**`eron8`** — a counterfactual, forked from the end of year 8. At that point Eron (born in year 3 to Sela and Roan) held 12 food, the most in the village; Dell had 11 and no one else had more than 6. The fork removes him with a single injected event, announced to everyone as *"Eron has died."* His 12 food went to the pile, so year 9 opened with 27 food for 13 people instead of 15 for 14.

Years 9–15 then played out differently:

| | `main` | `eron8` |
|---|---|---|
| Starvation deaths, years 9–15 | 6 (Bram, Taric, Sela, Nessa, Zinn, Dell) | 3 (Taric, Roan, Dell) |
| Births, years 9–15 | 1 (Joren) | 1 (Talia) |
| Population at year 15 | 9 | 11 |

Treat this as one roll of the dice, not a result: the model is stochastic, so a fork with no injected event would also diverge. Running the fork several times is how you'd tell whether removing Eron matters beyond that natural spread.

## Looking at it

From a clone of the Redoland repo made with `git clone --recurse-submodules`:

```bash
.venv/bin/python -m redoland serve            # then open http://127.0.0.1:8000
.venv/bin/python -m redoland log sample_run
.venv/bin/python -m redoland timeline sample_run --branch eron8
.venv/bin/python -m redoland state sample_run --branch main --year 8
.venv/bin/python -m redoland diff sample_run main 15 eron8 15
```

To try your own counterfactual, fork from any action commit and run it forward:

```bash
.venv/bin/python -m redoland fork sample_run --at <commit> --name myfork
.venv/bin/python -m redoland inject sample_run --branch myfork --narrate "..." --changes '{...}'
.venv/bin/python -m redoland run sample_run --branch myfork --years 3
```

Running costs Claude usage: a year is many `claude -p` calls per villager.

## Layout

- `meta.json` — world settings, model, random-number state and where the run left off
- `agents/aNNN.json` — one file per villager, living or dead
- `events/year-N.jsonl` — everything that happened in year N
