# hypo-loop

A [Claude Code skill](https://skills.sh) that runs an iterative hypothesis-testing loop — generate, execute, evaluate, evolve — until a condition is met or a loop budget is exhausted.

## Install

```bash
npx skills add shineli1984/hypo-loop
```

## Usage

Trigger with any of:

- `/hypothesis-loop`
- "run a hypothesis loop"
- "try different approaches until…"
- "iterate until condition"
- "keep trying until X"
- "run until it works"

## How it works

Each loop follows four steps:

1. **Generate** — form a hypothesis based on confirmed learnings and ruled-out approaches
2. **Execute** — run the loop task applying the hypothesis
3. **Evaluate** — check whether the condition is met; classify the result
4. **Evolve** — update the session state; carry learnings forward

State is persisted to `.hypothesis-loop/session.md` (distilled index) and `.hypothesis-loop/loop-0N.md` (full detail per loop). If context is lost mid-session, the skill reconstructs from the state file.

## Inputs

| Input | Description |
|-------|-------------|
| **Condition** | Objectively evaluable success criterion (test passing, metric threshold, output match) |
| **Loop task** | What each loop does — the action space (e.g. "run tests and fix failures") |
| **Max loops** | Upper bound on iterations. Default: 5 |

## Example

> "Run a hypothesis loop. Condition: the benchmark scores above 90. Loop task: modify the prompt and re-run the benchmark. Max loops: 8."

The skill will iterate, learning from each result, until the score exceeds 90 or 8 loops are exhausted — then print a final summary with what worked, what didn't, and a recommendation.
