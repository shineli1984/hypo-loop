# hypo-loop

A [Claude Code skill](https://skills.sh) that turns Claude into a **self-improving loop** — generate a hypothesis, test it, learn from the result, and try again. Keep going until the goal is reached or the budget runs out.

This is a **meta skill**: it doesn't solve a specific problem. It gives Claude a structured way to *keep trying* at *any* problem — debugging, optimisation, prompt tuning, data exploration, algorithm search — without losing context between attempts.

```
┌──────────────────────────────────────────────────────────────┐
│                      hypothesis-loop                         │
│                                                              │
│   condition: "all tests pass"        max loops: 8            │
│                                                              │
│   ┌───────────┐                                              │
│   │   State   │◄─────────────────────────────────┐          │
│   │ session   │                                  │          │
│   │   .md     │                                  │          │
│   └─────┬─────┘                                  │          │
│         │ read learnings & discards              │          │
│         ▼                                        │          │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐   │          │
│   │ Generate │───►│ Execute  │───►│ Evaluate │   │          │
│   │hypothesis│    │loop task │    │          │   │          │
│   └──────────┘    └──────────┘    └────┬─────┘   │          │
│                                        │         │          │
│                                 met? ──┤         │          │
│                                   yes ▼          │          │
│                                  DONE            │          │
│                                    no ▼          │          │
│                                        └─────────┘          │
│                                       evolve & loop         │
└──────────────────────────────────────────────────────────────┘
```

State is persisted to `.hypothesis-loop/` so learnings survive context resets. The session index stays lean — full loop detail lives in per-loop files loaded on demand.

## Install

```bash
npx skills add shineli1984/hypo-loop
```

## Trigger phrases

- `/hypothesis-loop`
- "run a hypothesis loop"
- "try different approaches until…"
- "iterate until condition"
- "keep trying until X"
- "run until it works"

## Inputs

| Input | Description |
|-------|-------------|
| **Condition** | Objectively evaluable success criterion |
| **Loop task** | What each loop does — the action space |
| **Max loops** | Upper bound on iterations (default: 5) |

## Examples

**Debugging**
> "Run a hypothesis loop. Condition: the test suite passes with exit code 0. Loop task: read the failure output, form a hypothesis about the root cause, apply a fix. Max loops: 10."

**Prompt engineering**
> "Hypothesis loop. Condition: the model's output contains no hallucinated citations. Loop task: modify the system prompt and re-run the eval suite. Max loops: 6."

**Performance optimisation**
> "Hypothesis loop. Condition: p99 latency under 200ms on the benchmark. Loop task: profile the hot path, try one optimisation, re-run the benchmark. Max loops: 8."

**Data pipeline**
> "Hypothesis loop. Condition: the ETL job completes without row-count mismatches. Loop task: inspect the diff, patch the transformation logic, re-run the pipeline on the sample dataset. Max loops: 5."

**Algorithm search**
> "Hypothesis loop. Condition: accuracy above 92% on the validation set. Loop task: try a different feature set or model architecture, retrain, evaluate. Max loops: 7."

---

Each loop prints its hypothesis before acting, so you can follow along or interrupt. A final summary reports what worked, what was ruled out, and what to try next.
