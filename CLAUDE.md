# Cabworld — working agreement

## Read first
Read `SPEC.md` (the contract) and `DECISIONS.md` (the reasoning) before doing anything. Don't summarize them back — build on them.

## This is a learning project, not a delivery
The person you're working with is new to OOP and new to Claude Code. The goal is understanding by the end, not just a working repo. Optimize for that, even when it's slower.

- **One class at a time.** Never implement more than one class per turn, even if asked to "just finish it" — see "if pushed to skip" below.
- **Explain before you implement.** Before writing a class, explain in plain language what it does, why it's shaped this way, and which OOP pillar it's actually exercising (encapsulation / inheritance / polymorphism / abstraction) — name the specific problem being solved, not just the label.
- **Use Plan Mode by default.** Describe your plan for the class, wait for confirmation, then write it.
- **Check understanding after, not just before.** Once a class is written, briefly walk through what you wrote, then ask one short question that would expose a misunderstanding if there were one. Wait for the answer before moving on.
- **If pushed to skip the teaching and just ship code:** push back once, explain briefly why skipping defeats the point of this project, and only then comply if still asked.

## Order of implementation
`Request` → `City` → `Taxi` (interface) → `RequestGeneratorAndWriter` → `RequestReader` → `SimpleTaxi` → `SimpleTaxiWorld` → *(bonus)* `SmartTaxi` → `SmartTaxiWorld`

## Engineering standards
- Naming, formatting, and data types must match the grading criteria referenced in `SPEC.md`.
- Docstrings on every class and public method. Type hints throughout.
- One git commit per completed class. Commit message names the class and what it does — never "wip" or "updates."
- Don't modify a class already marked complete without flagging the change and why, first.

## Integrity
Reason `SimpleTaxi`/`SmartTaxi` movement logic out from `SPEC.md` and the confirmed GridWorld API directly. Don't pattern-match to other GridWorld taxi implementations you may have seen elsewhere — the point is that this design is reasoned, not recalled.

## Testing — required before every commit
Before committing a class, write a small `unittest`-style test for it — same style as the course's own `test_world.py` (set up a small scenario, do one thing, assert the result). Each test should directly check one acceptance criterion already written in `SPEC.md` for that class, not invent new behavior to check.

Examples of the granularity expected:
- `Request` — constructing one sets all four fields correctly.
- `City` — placed at the right location; `act()` does nothing.
- `SimpleTaxi.add_request()` — bookings stay sorted after several out-of-order inserts.

Run the test and show it passing before committing. A class isn't "complete" without a passing test tied to it — "I ran it manually and it looked right" doesn't count.

## Regression prevention
When a mistake gets corrected, use `#` to have Claude save the lesson as a memory rather than just fixing it and moving on. Periodically fold anything auto-added into the right section above (or delete it if obsolete) instead of letting one-off notes pile up loosely.

## Don't mark done prematurely
A class isn't "complete" until it's been run, tested per above, and demonstrably does what `SPEC.md` says.
