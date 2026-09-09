# LLM-evals

A working repo for learning AI evals from the ground up — hands-on exercises,
larger projects, and article drafts, in that order. This is a workbench, not
a polished showcase: finished articles get cleaned up and published
externally; what lives here is the code and notes behind them.

## Structure

- **[exercises/](exercises/)** — small, focused drills, one concept each
  (e.g. "build a pointwise LLM judge"). Numbered roughly in the order they
  build on each other. See [ROADMAP.md](ROADMAP.md) for the full list.
- **[projects/](projects/)** — larger, self-contained pieces of work that
  combine several concepts (e.g. a full eval harness, a judge-calibration
  study). Each gets its own subfolder with its own README.
- **[articles/](articles/)** — drafts of write-ups based on the exercises
  and projects. Polished before publishing elsewhere.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in API keys
```

## Roadmap

See [ROADMAP.md](ROADMAP.md) for the topic-by-topic plan, tracked as
checkboxes so progress is visible over time.
