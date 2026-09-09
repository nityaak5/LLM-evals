# Roadmap

A broad survey of AI evals, organized as phases that roughly build on each
other. Each topic becomes one or more entries in [exercises/](exercises/);
the ones marked with a project link graduate into [projects/](projects/)
once the underlying exercises are done. Check items off as you go.

## Sources

Exercises draw on outside course material where it maps onto a topic here,
adapted rather than copied — reference notebooks live in
[references/](references/). Currently pulling from
[ARENA's LLM Evals chapter](https://learn.arena.education/chapter3_llm_evals/01_intro_evals/intro/)
(threat-modeling, judge design, benchmark construction with `inspect_ai`).

## Phase 1 — Foundations

- [ ] What is an "eval"? Offline evals vs. online monitoring, why they matter
- [ ] Anatomy of an eval: task, output, scoring method, aggregation
- [ ] Reference-based scoring (exact match, F1, ROUGE/BLEU, embedding similarity)
- [ ] Binary/classification evals and reading precision/recall/F1 correctly

## Phase 2 — LLM-as-judge

- [ ] Building a simple pointwise LLM judge
- [ ] Pairwise / comparative judging (A vs. B, Elo-style ranking)
- [ ] Judge calibration: measuring agreement with human labels (Cohen's kappa, correlation)
- [ ] Common judge biases — position, verbosity, self-preference — and mitigations
- [ ] Rubric design: turning vague criteria into a structured, checkable rubric

## Phase 3 — Building eval datasets

- [ ] Where good eval sets actually come from (error analysis on real traffic beats synthetic data)
- [ ] Slicing and stratifying data; avoiding leakage/contamination
- [ ] Synthetic data generation for evals, and its pitfalls
- [ ] How many examples do you need? Sample size and statistical power for evals

## Phase 4 — Domain-specific evals

- [ ] RAG evals: retrieval metrics, faithfulness/groundedness, answer relevancy
- [ ] Agent & tool-use evals: task success, trajectory vs. outcome evaluation
- [ ] Multi-turn conversation evals
- [ ] Safety/robustness evals: red-teaming basics, adversarial prompts

## Phase 5 — Evals infrastructure & practice

- [ ] Building a lightweight eval harness from scratch
- [ ] Survey of eval tooling (OpenAI Evals, Braintrust, promptfoo, W&B Weave, LangSmith, Inspect)
- [ ] CI-integrated evals / regression testing for prompts
- [ ] Closing the loop: error analysis → eval → fix → re-eval

## Phase 6 — Writing it up

- [ ] Turning an exercise/project into an article: what makes evals writing resonate
- [ ] Case-study article structure

---

Not a fixed syllabus — reorder or add topics as real work surfaces gaps.
