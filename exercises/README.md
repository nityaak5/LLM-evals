# Exercises

Small, focused drills — one concept per exercise, numbered in the order
they build on each other (see [../ROADMAP.md](../ROADMAP.md)).

Each exercise lives in its own folder: `NN-topic-name/`, with its own
README stating the goal, and whatever code/data it needs. Start a new one
by creating the folder and a README describing what you're trying to
learn or demonstrate before writing code.

Exercises are notebooks (`.ipynb`) — this is where you're prototyping one
concept interactively and poking at raw outputs, not building something
meant to be imported or run repeatably. That's what
[../projects/](../projects/) is for, and it's the point where things move
to `.py`.

**Model access:** [01-intro-to-api-calls](01-intro-to-api-calls/) uses the
native `anthropic`/`openai` SDKs directly, since the point there is
learning the actual provider APIs. From
[02-alignment-faking-case-study](02-alignment-faking-case-study/) onward,
exercises use OpenRouter instead whenever they need an open-weight or
non-Anthropic/OpenAI model — same `openai`-package client, just pointed
at a different `base_url`. Default to whichever an exercise actually
needs; don't add OpenRouter to an exercise that only touches Claude/GPT.
