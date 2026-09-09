# 01 — Intro to API Calls

Source: adapted from [ARENA's LLM Evals, ch. 3.1, section 1](https://learn.arena.education/chapter3_llm_evals/01_intro_evals/intro/)
(full notebook: [references/3_1_Intro_to_Evals_exercises.ipynb](../../references/3_1_Intro_to_Evals_exercises.ipynb)).
Reworked to fit our stack. 

## Goal

Get comfortable with the primitives almost every eval needs: sending chat
messages to a model API, understanding message roles, and writing a
reusable, retry-safe `generate_response` wrapper. Later exercises
(LLM-as-judge, rubric scoring, etc.) import this function rather than
re-deriving it.

## What's different from the original

- ARENA routes every model through **OpenRouter** so one client/key can hit
  `gpt-4o-mini` and `anthropic/claude-sonnet-4` interchangeably. We already
  have native `anthropic` + `openai` SDKs in
  [requirements.txt](../../requirements.txt), so `generate_response` dispatches
  by provider instead of adding a proxy + third API key. Say the word if
  you'd rather switch to OpenRouter later — it does make multi-model
  comparisons simpler.
- Skipping the alignment-faking case study (section 2 of the notebook) and
  the threat-modeling/specification-design exercise (section 3) for now —
  those are their own, much bigger pieces of work (see "how far this
  notebook goes" below).

## Build this

1. **Basic call** — send a `system` + `user` message to a model, print both
   the raw response object and just the text content. Do it for an
   Anthropic model and an OpenAI model so you see how the two response
   shapes differ (Anthropic takes `system` as a separate argument, not a
   message in the list).
2. **Roles** — try a `system` prompt that changes tone/behavior, and
   (Anthropic only) a trailing `assistant`-role message as a prefill.
   Confirm OpenAI ignores a trailing assistant message; Anthropic
   continues from it.
3. **`generate_response(model, messages, temperature, max_tokens)`** — one
   function that picks the right client based on the model name and
   returns just the text. This is the function every later exercise will
   import.
4. **(Optional) retry with exponential backoff** — wrap `generate_response`
   so a rate-limit error triggers a sleep-and-retry with increasing
   backoff instead of crashing. Low value in isolation, but you'll want it
   once you're running batches of eval questions (exercise 02+).

## Files

- `starter.ipynb` — one cell per piece above, each with a TODO and no
  implementation filled in. Write the code yourself; ask for help on a
  specific stuck point rather than a full solution.

## Done when

`generate_response` returns a real completion for at least one Anthropic
model and one OpenAI model, using messages you constructed by hand.
