# Cabworld — Decisions

Durable conclusions from planning conversations, kept separate from `SPEC.md` (the implementation contract) and any future `CLAUDE.md` (short, behavior-steering instructions for Claude Code). This file is the "why" — link to it from `CLAUDE.md` rather than duplicating it there.

## OOP foundations (mental model)

- Class = blueprint, object = instance built from it.
- The four pillars aren't features to add — they're answers to four questions that arise naturally as a system grows:
  - Encapsulation → "who's allowed to touch this data?"
  - Inheritance → "what do I already have that I shouldn't rewrite?"
  - Polymorphism → "do I need to know exactly what this is before I use it?"
  - Abstraction → the payoff of the other three: minimum surface area to use something safely.
- Pillars are symptoms of good design, not ingredients to sprinkle in. Forcing them where they're not needed is over-engineering.
- OOP earns its keep for stateful, interacting entities — not automatically the right fit for pure data transforms or numerical computation.

## Assignment decoding

- **Core problem, one sentence:** move four independent taxi agents around a shared 10×10 grid, over discrete time steps, until each completes its ordered list of pickup/dropoff jobs — animated and reported.
- **Pipeline:** generate requests → write CSV → read CSV → assemble world (cities, taxis, request assignment) → simulate → report step counts.
- **Class map:** `City` (is a `Rock`), `Request` (plain data, not an `Actor`), `Taxi` (interface — `is_occupied()`, `add_request()`), `SimpleTaxi`/`SmartTaxi` (are `Actor`s, implement `Taxi`), `RequestGeneratorAndWriter`/`RequestReader` (I/O utilities, no simulation role), `SimpleTaxiWorld`/`SmartTaxiWorld` (is a `ActorWorldWithGUI`, owns `set_scenario()`).
- Full detail lives in `SPEC.md`, including the resolved GridWorld framework API.

## Process decisions

- **Priority order** (cone of uncertainty — effort where a wrong decision is expensive to reverse): architecture/class design > data pipeline shape > GridWorld API specifics > tooling/environment > version control/PM.
- **Methodology adopted:** lightweight Spec-Driven Development — write the spec, implement against it, don't over-invest in ceremony disproportionate to a solo coursework project.
- **Tool role split:**
  - This chat — conceptual learning, architecture, decisions, spec-writing. No implementation here.
  - Claude Code (VS Code) — actual implementation, working from `SPEC.md` and `CLAUDE.md`.
  - GitHub — version history, one commit per meaningful checkpoint.
  - `CLAUDE.md` (once created) — short, behavior-steering instructions only, links here for reasoning.
  - `DECISIONS.md` (this file) — durable reasoning and history.
  - Notion — high-level phase/task tracking only, not reasoning.
- A file that just accumulates instructions Claude Code can already derive from code (architecture, file structure) isn't useful — this file should hold what genuinely can't be derived from reading the repo.

## Setup log

- [x] Project named Cabworld, repo created and pushed to GitHub
- [x] `.gitignore`, `assignment5/` package skeleton, empty class stubs
- [x] `SPEC.md` written and resolved against real GridWorld source (framework API confirmed; taxi/request logic left as the actual design work, not copied from a friend's completed solution)
- [ ] `CLAUDE.md` — not yet created
- [ ] Claude Code set up in VS Code
- [ ] First class implemented
