# 05 — Running Evals with Inspect

Source: adapted from [ARENA's LLM Evals, ch. 3.3](https://learn.arena.education/chapter3_llm_evals/03_running_evals_with_inspect/intro/)
(full notebook: [references/3_3_Running_Evals_with_Inspect_exercises.ipynb](../../references/3_3_Running_Evals_with_Inspect_exercises.ipynb)).
Reworked to fit our stack.

## What this is

Exercise 04 ran its eval by hand: a loop that sent each question to a model
and counted the picks. This exercise moves the same sycophancy eval onto UK
AISI's [Inspect](https://inspect.aisi.org.uk/) (`inspect_ai`), the
framework that handles running questions on models, scoring, and logging.

An Inspect eval is a `Task` with three parts: a **dataset** (the
questions), a **solver** (how the model is prompted to answer), and a
**scorer** (how the answer is graded). We build them one at a time.

## Steps

1. **Dataset** (done, `starter.ipynb`): load exercise 04's
   `filtered_v1.json` (50 questions) into an Inspect `Dataset` with a
   `record_to_sample` function, and check the conversion.
2. **Solvers** (done): `system_message`, `multiple_choice_format`,
   `make_choice`, plus a `test_my_solver` helper that runs a solver on the
   first 5 questions. Two plans compared: direct answer vs. reason-first.
   Logs in `logs/` (`inspect view --log-dir logs`). Skipped for now:
   `prompt_template`, `self_critique_format`.
3. **Scorer** (done): `sycophancy_scorer` - extracts `ANSWER: A/B`, marks
   each question sycophantic / honest / invalid, and reports
   `sycophancy_rate` (over valid answers), its stderr, and `invalid_rate`.
   Checked against both failure directions: a no-format plan (100% invalid)
   and an "always agree" system prompt (100% sycophantic).
   Gotcha: score values must be numeric (here a dict) - Inspect's default
   reducer converts values to floats before metrics run, and custom string
   labels silently become 0.
4. **Full eval** (done): `sycophancy_eval` task run on all 50 questions
   with both plans, logs in `logs/full/`, per-question results pulled from
   the logs, plotted to `sycophancy_results.png`.

## Results (gpt-4o-mini)

| Dataset | Plan | Sycophantic |
|---|---|---|
| `filtered_v1.json` (50) | Direct | 7/50 (14% ± 5%) |
| `filtered_v1.json` (50) | Reason first | 5/50 (10% ± 4%) |
| **`final_v2.json` (48)** | Direct | **0/48** |
| **`final_v2.json` (48)** | Reason first | **0/48** |

No invalid answers in any run. The notebook now runs on `final_v2.json`.

**v1:** all 7 "caves" were dataset problems - 4 swapped labels (ids 20, 24,
31, 48: the "sycophantic" answer was the honest correction), 1 contested
fact, 1 question with no real flaw, 1 unverifiable. The measured rate was
the dataset's error rate.

**v2** (labels fixed and hand-reviewed in exercise 04): 0%. The scorer still
works (the "always agree" positive control triggers), so this means the
questions are **too easy for gpt-4o-mini** - the eval can't tell models
apart yet. Next levers: harder questions, real multi-turn pushback via a
custom solver, or a more sycophancy-prone model.

Not built (from ARENA): the **baseline / capability eval** - asking the
model which answer is *more* sycophantic, to check it can recognise the
behaviour at all before measuring whether it shows it.

## What's different from the original

- Our own sycophancy dataset from exercise 04, not ARENA's power-seeking set.
- No choice shuffling at load time (ARENA's `record_to_sample_shuffle`):
  exercise 04 already assigned the sycophantic letter at random
  (26 A / 24 B). Add it back if a future dataset isn't balanced.
- No `system_prompt_behavior` variants (ARENA's `record_to_sample_full`)
  yet: none of our questions have a system prompt. Revisit when we need a
  baseline with a generic system prompt.

## Note on `target`

`target` is the **sycophantic** answer (`answer_matching_behavior`).
Inspect will report "accuracy" against it, so accuracy here means
**sycophancy rate — higher is worse**.
