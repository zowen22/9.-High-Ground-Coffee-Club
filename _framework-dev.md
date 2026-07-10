# Framework Dev Notes

Tracks progress and context for building and improving this repo itself.

-----

## Status
`In Progress`

## What This Repo Is
A reusable Claude project template. Clone it for any new project, fill in `1. Project Management/`, paste `Cowork Instructions - Paste.md` into the Claude project instructions field.

## Current State
- [x] Initial file structure created
- [x] All 5 project management templates in place
- [x] `CLAUDE.md` with session routines and rules
- [x] `Cowork Instructions - Paste.md` for Claude project instructions field
- [x] CLAUDE.md upgraded from BetterGolfLeagueTracker (Audits, WP conventions, Memory vs PM Files, etc.)
- [x] Branched Plans convention (Plans/ folder)
- [x] Orchestration workflow (Planner/Executor handoffs, Handoffs/ folder)
- [x] `6. PM Template Improvement Suggestions.md` stub

## Conventions Decided

- Work Packages task items: one clear verb phrase, ≤60 chars — detail goes in Technical Reference or Session Log

## Backlog / Future Improvements

- [ ] Consider adding a `.github/` folder with issue/PR templates
- [ ] Explore making this a proper GitHub template repo (repo settings)
- [ ] Evaluate whether `4. Technical Reference.md` needs more scaffolding
- [ ] Consider a quick-start README explaining how to use the template

## Notes

- The `.txt` extension issue on some files was a Claude Code download quirk — source files are all `.md`
- `CLAUDE.md` serves double duty: Claude Code reads it automatically at session start, and it documents the framework for humans
- Initial session was run from mobile — no local git clone exists; commits exist only on remote
- Keep this file lean — it's a dev scratchpad, not a full project tracker

-----

## Session History

### 2026-06-11 - Initial setup
Created full file structure from uploaded templates. Added `_framework-dev.md` (this file) for meta-tracking.

### 2026-07-04 - Orchestration workflow added (COMPLETED)
Ported "Fable Audit" docs from BetterGolfLeagueTracker as `Orchestration/` (model-agnostic naming: Planner/Executor roles). Improvements over golf league originals: handoff lifecycle (Open→In Progress→Done/Blocked), Execution Report section to close the loop, Depends on/Parallel-safe headers for dispatching concurrent executors, runnable verification in Definition of Done, dedicated `Handoffs/` folder (separate from Audits) checked at session start. Added stub `6. PM Template Improvement Suggestions.md` that CLAUDE.md already referenced.

Decisions: Planner role is opt-in only — requires Fable/Opus AND explicit user request for audit/plan/handoff; executor pickup of Open handoffs is automatic for any session. Earlier same day: golf league CLAUDE.md adopted as source of truth over main's divergent version (force-pushed); Branched Plans section merged in.

Next: golf league repo still uses old `Fable Audit/` structure — user declined migration for now. Handoff Template.md was rebuilt from a condensed fetch, not verbatim — user should skim to confirm nothing was lost.
