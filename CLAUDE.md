# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Starter code for the AI110 **Tinker 1B / 2B / 3B** in-class activities — a small Streamlit app called StudySync. The app "ships complete" but each module has one or more intentionally unfinished tickets (marked with `TODO`, `TICKET`, or `BUG` comments in the docstrings/code). Each tinker is scoped to a specific file:

| Tinker | Focus | Files |
|---|---|---|
| 1B, Split the Logic | Write a pytest test, then a cross-file refactor | `scoring.py`, `scoring_helpers.py`, `test_scoring.py` |
| 2B, Wire It Up | Streamlit `session_state`, input validation, dataclasses, recurring dates | `sessions.py`, `app.py` |
| 3B, Rank & Explain | CSV loading, weighted scoring with reasons, ranking, data-flow diagram | `ranking.py`, `data/study_spots.csv`, `diagram.mmd` |

When asked to work on "the tinker" or a specific ticket, read the module's top-of-file docstring first — it states the exact scope of what's broken/missing and what NOT to touch yet. Don't fix tickets that belong to a different tinker unless asked.

## Commands

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt   # streamlit>=1.30, pytest>=7.0

python -m streamlit run app.py    # run the app
pytest                             # run all tests
pytest test_scoring.py -k boundary_90   # run a single test
```

There is no build step, linter, or type checker configured in this repo.

## Architecture

`app.py` is the Streamlit entry point: it sets page config and renders three tabs, each delegating to a `render_*_tab()` function imported from the corresponding module. There is no shared state or model layer across tabs beyond `st.session_state` — each module owns its own tab's UI and logic.

- **`scoring.py`** — `session_rating(combined_score)` maps a score to a rating string; correct and intended to gain test coverage in Tinker 1B. `apply_streak_bonus()` currently lives here but is slated to move to `scoring_helpers.py` (the import in `scoring.py` must be updated after the move so `render_session_scorer_tab()` and `run_demo()` keep working).
- **`scoring_helpers.py`** — destination module for logic extracted out of `scoring.py`; currently just a stub with a TODO.
- **`sessions.py`** — `PlainSession` is a hand-written class slated to be reimplemented as a `@dataclass` (`SessionDC`). `next_occurrence()` and `find_conflicts()` are stubs (`raise NotImplementedError`) called from `render_session_log_tab()`, which already catches `NotImplementedError` and shows a `st.warning` placeholder — this is expected until the tinker is done. The tab also demonstrates a deliberately broken counter (plain local variable, forgotten on Streamlit rerun) alongside a not-yet-wired-up `st.session_state` counter.
- **`ranking.py`** — all core functions (`load_study_spots`, `score_study_spot`, `rank_study_spots`, `format_results`) are stubs. `load_study_spots` reads `data/study_spots.csv` via `csv.DictReader`; `distance_miles` must become `float` and `seats_available` must become `int`. `score_study_spot` scores a spot against a profile dict (`max_noise`, `max_distance`, `min_seats`) using `NOISE_ORDER` for noise comparison, returning `(score, reasons)`. `render_study_spot_tab()` also catches `NotImplementedError` and shows a placeholder warning.
- **`data/study_spots.csv`** — sample data consumed by `ranking.py` (columns: `id, name, noise_level, distance_miles, wifi_quality, seats_available`).
- **`diagram.mmd`** — placeholder for a Mermaid flowchart (input → process/score → output) to be filled in as part of Tinker 3B.

A recurring pattern across `sessions.py` and `ranking.py`: unfinished functions `raise NotImplementedError`, and the Streamlit render functions wrap calls to them in `try/except NotImplementedError` to show a friendly in-app warning instead of crashing — preserve this pattern when implementing stubs rather than removing the try/except.
