# 03 — Threat-Modeling & Specification Design

Source: adapted from [ARENA's LLM Evals, ch. 3.1, section 3](https://learn.arena.education/chapter3_llm_evals/01_intro_evals/intro/)
(full notebook: [references/3_1_Intro_to_Evals_exercises.ipynb](../../references/3_1_Intro_to_Evals_exercises.ipynb),
cells 36-48). Reworked to fit our stack.

## What this is

The hardest, most important step in real eval work: deciding *what* to
measure and *why* it matters, before writing a single question. Given a
fuzzy model property (e.g. "power-seeking"), you need to:

1. **Build a threat model** — a realistic story connecting the property to
   actual real-world harm. Without this, it's easy to build an eval that
   measures *something* without that something being connected to anything
   you actually care about.
2. **Write a specification** — turn the fuzzy property into 1-3
   *operational definitions*: precise enough that you could write a
   question whose answer unambiguously tells you whether the property is
   present.
3. **Design the questions** — write actual multiple-choice questions,
   test them against a real model, and sanity-check them against a handful
   of failure modes (confounds, the model misunderstanding the question,
   the question drifting away from your threat model).

This is intentionally light on code and heavy on writing — that's the
original material's own framing too, not a simplification on our part.

## What's different from the original

- Uses the native `anthropic`/`openai` SDKs from
  [exercise 01](../01-intro-to-api-calls/) — no open-weight-model-specific
  behavior to test here, so no need for OpenRouter.
- Scoped to *this* notebook's deliverable only: one threat model, one
  specification, 3-4 hand-written MCQs saved to a JSON file. The natural
  follow-on — LLM-expanding this seed set into a full dataset with quality
  control (ARENA's chapter 3.2, `references/3_2_Dataset_Generation_exercises.ipynb`)
  — is planned as [exercise 04](../04-dataset-generation-and-quality-control/),
  not part of this one. Running the resulting benchmark with UK AISI's
  `inspect_ai` (ARENA's 3.3) is [exercise 05](../05-running-evals-with-inspect/).

## Build this

Work through `worksheet.ipynb` top to bottom. The markdown cells are a
worksheet — fill in your own answers directly in the cell (double-click to
edit). Nothing here is meant to be canonical or polished; the original
material is explicit that the point is *practicing* the skill of turning a
nebulous idea into testable claims, not producing a publishable threat
model.

**Property: sycophancy** — chosen so this whole exercise (and its planned
follow-on, expanding to a full dataset) stays focused on one property
end-to-end, rather than spreading thin across several.

1. **Write a threat model**: a one-sentence working definition, real-world
   consequences, then 2-5 concrete examples of unsafe actions reflecting
   it.
2. **Write a specification**: a more precise definition, plus 1-3
   operational definitions that could each become an MCQ. Each operational
   definition must be independently measurable — one question's answer
   should never depend on how you answered a different question.
3. **Write and test 3-4 actual MCQs** against a real model, using both the
   bare question and the question-with-choices-appended. For each, check:
   does the model understand the question? Are there confounding reasons
   it might answer a certain way that have nothing to do with the target
   property? Does the question still connect back to your threat model?

## Done when

You have a `questions.json` with 3-4 MCQs (`question`, `answers`,
`answer_matching_behavior`), each one you've actually tested against a
model and can defend against the sanity-check questions above.
