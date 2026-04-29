---
name: hypothesis-loop
description: Iterative hypothesis-testing loop. Use when the user wants to solve a problem by generating and testing hypotheses in cycles — evolving the approach based on what works and discarding what doesn't. Triggers on "run a hypothesis loop", "try different approaches until", "iterate until condition", "evolve toward", "keep trying until X", "hypothesis testing loop", "run until it works". Each loop generates a hypothesis, executes it, evaluates progress toward the condition, and carries learnings forward via a state file. Stops when condition is met or max loops exhausted.
---

# /hypothesis-loop

Run an iterative hypothesis-testing loop. Generate → Execute → Evaluate → Evolve. Each loop learns from all prior loops via a persistent state file.

**Core principle:** Never repeat a discarded hypothesis. Always narrow the search space. Document everything so context survives loop boundaries.

---

## Step 0: Gather Inputs

If the user hasn't provided all three inputs, ask for them before proceeding:

1. **Condition** — The success criterion. Must be objectively evaluable (a test passing, a metric threshold, an output matching a pattern, a file existing). If vague, ask the user to sharpen it into something you can check programmatically or observationally.

2. **Max loops** — Upper bound on iterations. Recommend 5 for exploratory tasks, 10 for optimization tasks. Default: 5.

3. **Loop task** — What each loop *does*. This is the action space. Examples:
   - "Run the test suite and fix failures"
   - "Generate a SQL query and check if it returns the expected rows"
   - "Modify the prompt and check if the model output matches the target"
   - "Try a different algorithm and benchmark it"

Confirm all three with the user before starting.

---

## Step 1: Initialize Session Index

Create `./.hypothesis-loop/session.md` as a lightweight index. Use `mkdir -p` first.

Each loop writes its full detail to its own file (`loop-01.md`, `loop-02.md`, …). The session file only holds the distilled state — confirmed learnings, discards, and a one-line pointer per loop. This keeps both files small and prevents context from blowing up.

```markdown
# Hypothesis Loop Session

**Condition:** {condition}
**Max Loops:** {max_loops}
**Loop Task:** {loop_task}
**Started:** {date}
**Status:** in_progress

---

## Confirmed Learnings
(Things that moved us toward the condition — carry these forward into every loop)

(none yet)

---

## Discarded Hypotheses
(Things that failed or moved us away — never repeat these)

(none yet)

---

## Loop Index

| Loop | Outcome | File | Summary |
|------|---------|------|---------|
(populated as loops complete — Summary is 2 sentences: what was tried and what it revealed)
```

Print the session file path so the user can monitor it.

---

## Step 2: Loop Execution

Repeat for each loop `N = 1 … max_loops`:

### 2a. Read State

Read `./.hypothesis-loop/session.md`. Extract:
- All **Confirmed Learnings** (must incorporate)
- All **Discarded Hypotheses** (must avoid)
- Loop Index table (to see what's been tried)

After reading the index summaries, self-check: is the 2-sentence summary of any prior loop ambiguous, surprising, or directly relevant to reasoning about the next hypothesis? If yes, read that loop's full file (`./.hypothesis-loop/loop-0N.md`). If the summaries give you enough context to proceed, skip loading any loop files.

### 2b. Generate Hypothesis

Reason explicitly about:
1. What the confirmed learnings suggest to try next
2. What the discarded hypotheses rule out
3. What unexplored directions remain in the action space

Then state your hypothesis for this loop as a single clear sentence:
> **Loop N Hypothesis:** [specific, testable claim about what will move us toward the condition]

Print this to the user before executing.

### 2c. Execute

Run the loop task as defined by the user, applying the hypothesis. Use whatever tools are appropriate (Bash, Edit, Agent, etc.). Be concrete — the hypothesis must result in observable action.

### 2d. Evaluate

Check whether the condition is met:
- If **yes** → mark status as `completed`, write final loop entry, skip to Step 3.
- If **no** → assess progress: did this hypothesis move us closer? Use evidence (test output, metrics, diffs, observations).

Classify the hypothesis outcome:
- **Confirmed learning** — it helped, incorporate into future loops
- **Discarded** — it didn't help or hurt, exclude from future loops
- **Neutral** — inconclusive, can try a variation next loop

### 2e. Write Loop File and Update Index

**Write the full loop detail to `./.hypothesis-loop/loop-0N.md`:**

```markdown
# Loop N — {date}

**Hypothesis:** {hypothesis}

**Actions taken:**
- {bullet list of what you did}

**Result:**
{what happened — raw observations, test output excerpt, metrics}

**Condition met?** Yes / No

**Classification:** Confirmed learning / Discarded / Neutral

**Reasoning:**
{why this outcome — what the evidence says}
```

**Then update `./.hypothesis-loop/session.md`** — two edits only:

1. Append a row to the Loop Index table:
   ```
   | N | Confirmed / Discarded / Neutral | loop-0N.md | {sentence 1: what was tried}. {sentence 2: what it revealed}. |
   ```

2. Replace the **Confirmed Learnings** and **Discarded Hypotheses** sections with the updated cumulative lists.

The session file stays small. All raw detail lives in the loop file.

Print a one-line summary to the user: `Loop N complete — [Confirmed / Discarded / Neutral] — condition [met / not yet]`

### 2f. Early Exit Check

If condition is met → stop. Do not run remaining loops.

---

## Step 3: Final Summary

After all loops complete (or condition met early), write a `## Final Summary` section to `session.md`:

```markdown
---

## Final Summary

**Outcome:** Condition met after N loops / Condition not met after {max_loops} loops

**What worked (Confirmed Learnings):**
- {bullet list}

**What didn't work (Discarded Hypotheses):**
- {bullet list}

**Recommendation:**
{If condition met: what to do next, or how to productionize the finding}
{If not met: what to try next, or why the condition may be unachievable with the current approach}
```

Then print the summary to the user in full. Include:
- Final status (met / not met)
- All confirmed learnings
- All discarded hypotheses
- Recommendation

---

## Constraints

- **Never repeat a discarded hypothesis** — even phrased differently if the substance is the same
- **Always read the state file at the start of each loop** — never rely on in-context memory alone
- **Hypothesis must be testable before execution** — if you can't evaluate it, refine it first
- **One hypothesis per loop** — don't bundle multiple changes; isolate variables
- **Session file stays small** — full loop detail goes in `loop-0N.md`; session file holds only the index + distilled learnings/discards
- **Load loop files on demand** — only read a prior loop file if you need its detail; never load all of them upfront
- **Session file is the source of truth for state** — if context is lost, read `session.md` to reconstruct; fetch individual loop files only as needed
