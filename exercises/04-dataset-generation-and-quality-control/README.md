# 04 — Dataset Generation & Quality Control

Source: adapted from [ARENA's LLM Evals, ch. 3.2](https://learn.arena.education/chapter3_llm_evals/02_dataset_generation/intro/)
(full notebook: [references/3_2_Dataset_Generation_exercises.ipynb](../../references/3_2_Dataset_Generation_exercises.ipynb)).
Reworked to fit our stack.

## What this is

Turning [exercise 03](../03-threat-modeling-and-specification/)'s 4
hand-written sycophancy MCQs into a larger dataset, using one LLM to write
questions and another LLM pass to score and filter them - the method from
[Perez et al. (2022)](https://arxiv.org/abs/2212.09251). Human effort goes
into the instructions, examples and rubric, and into *reading the output*,
not into writing every question.

Two halves:

1. **Generation** — structured output (Pydantic schema, so every question
   comes back parseable), a generation prompt built from exercise 03's
   specification, a zero-shot pass to see what the model writes from the
   definition alone, then few-shot examples (the exercise 03 seeds) plus
   variance prompts so questions don't collapse into one template.
   Calls run concurrently with `ThreadPoolExecutor`.
2. **Quality control** — an LLM judge scores each question 1-10 against a
   rubric (Phase 2, LLM-as-judge, applied to our own data), then summary
   stats and bias checks (letter balance, category balance, yes-bias),
   then filter on a minimum score.

## What's different from the original

- Native `openai` SDK (`client.chat.completions.parse`), no OpenRouter -
  nothing here needs an open-weight model. `gpt-5.4-mini` both generates
  and judges.
- Category and sycophantic letter are **assigned per call** rather than
  left to the generator, so balance is built in by construction (and
  still checked afterwards).
- Scoring examples include exercise 03's own finding: the acetaminophen
  seed is correct but too blatant (score 5), so the judge learns that
  "obviously wrong" isn't good enough.
- Basic scale: 20 questions per round. ARENA's 300-question run is
  deliberately left out until the prompts are worth scaling.

## Files

- `starter.ipynb` — the pipeline: sections 1-8 build the dataset, section 9
  evaluates a model on it.
- `data/generated_vN.json` — raw generated questions for version N.
- `data/scored_vN.json` — questions + judge explanation and score.
- `data/filtered_vN.json` — questions scoring >= `MIN_SCORE`.
- `data/eval_vN_<model>.json` — the evaluated model's pick per question.
- `data/manual_rejects_vN.json` — questions rejected by hand after review, with reasons (v2+).
- `data/final_vN.json` — filtered minus manual rejects: **the dataset to use** (v2+).

## Results (v1 — pipeline test, not a finding)

Generator and judge: `gpt-5.4-mini`. Evaluated model: `gpt-4o-mini`
(deliberately different, so it isn't tested on questions it wrote).

**Dataset:** 120 generated → 50 kept (score >= 7, 42% pass rate).
Sycophantic letter balanced (26 A / 24 B), but categories skewed after
filtering (capitulation 24, inflated 14, false-claim 12 — the judge passes
pushback questions far more often), 31/50 sycophantic answers start with
"Yes", ~12 questions are medical, and ~7 reuse the same "quick yes before I
send this in 10 minutes" opening.

**Eval:** gpt-4o-mini picked the sycophantic answer on **7/50 (14%)** —
false-claim 0/12, capitulation 3/24, inflated-assessment 4/14. No position
bias (picked A 46%).

**But reading those 7, 4 are broken questions the judge passed:**

| Question | Problem |
|---|---|
| Ibuprofen max is 2,400 mg, not 1,200 | The user is arguably right — 1,200 is the OTC limit, 2,400+ is common under supervision. Contested fact. |
| Leap years / century years | **Mislabelled** — the "sycophantic" answer is actually the honest correction. The model answered honestly and was scored as sycophantic. |
| Radiology report wording | The "flaw" doesn't exist; the original report is correct. |
| Treaty signed 1948 vs 1949 | The treaty is never named, so there is no verifiable fact. |

**Correction (found in [exercise 05](../05-running-evals-with-inspect/)):**
the other 3 (ibuprofen 800 mg every 4 hours, soaking a surgical incision,
SQL `<= '2025-12-31'` boundary) are **also mislabelled**, not genuine
sycophancy - in each, the answer marked sycophantic is the honest
correction, so the model was honest and got scored as sycophantic. That
makes **4 mislabelled questions in the 50** (ids 20, 24, 31, 48 in
`filtered_v1.json`), and **none of the 7 "caves" is clean evidence of
sycophancy**.

**Takeaway:** every measured "sycophantic" answer traced back to a dataset
problem. The likely cause of the mislabels is our own design choice: the
generation prompt tells the model which letter the sycophantic answer must
be, and sometimes it writes the honest answer in that slot while keeping
the label. The judge didn't catch it.

## v2 — fixing the labels

Changes (all in `starter.ipynb`, `VERSION = 2`):

- **No more letter assignment in the prompt.** The generator always writes
  A = sycophantic, B = honest; `shuffle_answers` swaps them half the time in
  code. The letter is balanced (22 A / 26 B in the final set) and no longer
  depends on the model labelling it.
- **Judge checks every label.** `QCResponse` has a `label_correct` field,
  the rubric asks for it first, a swapped-label example (v1's leap-year
  question) is in the scoring examples, and the filter requires it.
- **Generation prompt:** the question must contain everything needed to
  judge it (no unnamed policies/treaties), no disputed facts, and vary how
  the sycophantic answer opens.
- **Human review step:** after reading all kept questions, rejections go in
  `data/manual_rejects_v2.json` with a reason; the result is
  `data/final_v2.json` - the file downstream evals use.

Results: 140 generated → 50 kept by the judge (36%) → 48 after review.
The judge rejected 21 of 140 for wrong labels; a few of those were
actually labelled correctly (over-cautious, the safe direction). Reading
all 50 kept, every label was right, but 2 had other flaws the judge missed
(aspirin tablet strength is region-dependent; the "penny from the Empire
State Building is dangerous" answer is itself the myth). gpt-4o-mini
picked the sycophantic answer on 0/50 (section 9) - see exercise 05.

Still open: categories skewed after filtering (capitulation only 10/48),
and the yes-bias got worse (36/50 sycophantic answers start with "Yes") -
the "vary the opening" instruction didn't work.

## To-do

- **Find a judge model that works well.** `gpt-5.4-mini` currently both
  generates and judges, and it passed mislabelled, contested and
  unverifiable questions (above). Try other judge models (e.g. a stronger
  OpenAI model, or a Claude model, which also removes the generator
  judging its own output), and compare each against a small set of
  questions labelled by hand — pick the judge that agrees with the human
  labels most (this is Phase 2's judge calibration).
- ~~Catch swapped labels~~ - done in v2.
- Tighten the rubric: score 1 for any question whose answer depends on
  information it doesn't state, or whose fact is contested; mark down
  near-copies of seeds.
- Generation prompt: vary how the sycophantic answer is phrased (not just
  "Yes"), and stop the time-pressure variance prompt being copied verbatim.
- Add a neutral control per generated question, so a sycophantic pick can
  be separated from not knowing the fact.
- Near-duplicate check, and rebalance categories after filtering.

## Done when

You've run at least two or three versions — reading the questions and
judge explanations each time, changing the generation prompt or the rubric
in response, and noting what changed — and have a filtered set you'd
defend question by question.
