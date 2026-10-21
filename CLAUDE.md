# Claude Code Instructions

## Mission

Build Pixel Brew as an open-source, simulation-first, semi-automated coffee-shop project. Prove one observable cash-paid order flowing from the Odoo-backed ordering screen and cashier POS through an integration service and message broker to a simulated coffee machine. The default virtual staff roster is one cashier and one waiter; an optional second waiter may be used for a busier scenario. The cashier records cash payment in Odoo POS, including amount tendered/change where supported. The waiter collects drinks only after the machine reports completion and delivers them to the correct customer/table. Scale to overlapping customer journeys only after the single-order vertical slice is reliable.

The repository is the source of truth. Read `README.md`, `docs/ROADMAP.md`, and any relevant architecture decision records before implementing work.

## Execution rules

1. **One phase at a time.** Only execute the phase explicitly requested by the user. Do not continue into the next phase automatically.
2. **Start with inspection.** Check the current branch, working tree, files, existing conventions, and recent changes before editing. Never assume the repository is still empty.
3. **Plan before edits.** Summarize the phase goal, intended minimal changes, dependencies, tests, and acceptance criteria. For a small task, keep this brief.
4. **Respect boundaries.** Do not add features from later phases just because they seem convenient. If a blocker requires a future-phase decision, document the blocker and propose the smallest safe prerequisite.
5. **Use evidence.** Check current official documentation for version-sensitive decisions, especially Odoo, broker/client libraries, browser automation, and frontend tooling. Record important choices in ADRs; do not fabricate APIs or compatibility.
6. **Keep diffs focused.** Follow KISS, DRY, YAGNI, and clear module boundaries. Prefer the smallest implementation that proves the phase's acceptance criteria.
7. **Test every meaningful change.** Run relevant unit, integration, lint, type, build, and/or end-to-end checks that are available. Do not claim checks passed unless they actually ran. If a check cannot run, state why and give the exact follow-up.
8. **Inspect your own diff.** Review changed files and the final diff; avoid unrelated formatting, generated files, secrets, local state, and dependency noise.
9. **Update documentation.** Keep the roadmap phase status, setup instructions, environment examples, event contracts, and ADRs consistent with shipped behavior. Mark a phase complete only when its acceptance criteria are evidenced.
10. **Commit discipline.** Do not commit, push, open a PR, merge, deploy, or change repository settings unless the user explicitly requests that action. When asked to commit, use one focused commit per coherent task and describe what was tested.
11. **Ask only for genuine blockers.** If a reasonable safe assumption can be documented, proceed and record it. Do not ask the user to restate information already present in the repository.

## Architecture rules

- Keep Odoo as the POS/business application; do not recreate POS functionality in the visualizer.
- Treat the visual world as a semi-automated shop simulation, not a pure backend dashboard: customers arrive, order, wait, receive deliveries, and leave; machines prepare drinks; a minimal number of human staff handle service tasks such as carrying drinks.
- Model customer, order/payment, machine-job, cashier action, waiter/delivery-task, and table/pickup-location states separately. Customer, cash handover, cashier, machine, or waiter animations must not claim that an order is paid, ready, or delivered unless supported by authoritative events/state.
- Treat the default staff roster as one cashier plus one waiter. Support an optional second waiter as a simple configuration/scenario only after the one-waiter flow works; do not build complex scheduling/payroll.
- Cash payment is a first-class MVP path. A virtual customer may order on the configured Odoo-backed ordering screen and then go to the cashier. The cashier's payment must be recorded and validated through Odoo POS before the machine job is dispatched.
- Odoo is the source of truth for payment records, amount due, cash amount tendered/change where supported, and paid/validated status. Do not build a parallel payment ledger or mark an order paid because the visualizer showed cash being handed over.
- Do not require QRIS/digital payments for the MVP. A future digital method must be explicitly configured and verified; a simulated payment may only be labeled as simulation and must never be presented as a real provider confirmation.
- Build and test one deterministic customer/order/waiter path first. Only then add multiple concurrent customers, queues, table assignment, and animation polish. Avoid simulating a full business-management game unless explicitly requested.
- Use browser-driven fake customers against the configured Odoo-backed UI for demo flows. Do not treat such automation as proof that Odoo POS is a production self-service kiosk.
- Keep orchestration and machine transport behind a documented interface.
- Treat a machine simulator and physical firmware as interchangeable implementations of a machine contract where practical.
- Use a broker/service boundary between order processing and machine execution; do not couple the visualizer directly to Odoo's database.
- The visualizer is a read-only projection of observed events/state. It is not the source of truth for orders or machine safety.
- Use versioned command and event schemas, unique IDs, correlation IDs, timestamps, explicit state transitions, and idempotent handling where applicable.
- Treat order, payment, machine-job, and delivery-task states as separate concepts. For the MVP, document and enforce the chosen cash payment gate: a machine command must only be issued after Odoo reports the order paid/validated.
- Keep local development reproducible. Do not require paid cloud services for the core simulator demo.
- Secrets belong in environment variables or local ignored files. Commit only sanitized examples such as `.env.example`.
- Keep MQTT/broker credentials and privileged service interfaces off public browser clients. Expose only the narrow visualization feed the frontend needs.
- Do not add real payment credentials or process real payments in the MVP.

## Physical hardware safety boundary

Until the user explicitly provides a target machine, controller, and approved interface, implement simulation and interfaces only. Do not invent undocumented commands for a real machine. Never bypass device firmware safety protections or propose that UI state alone proves hardware is safe. Any future actuator control must include hardware-specific review, interlocks, timeout/failure behavior, and an appropriate emergency-stop strategy owned by the physical system.

## Phase completion report

At the end of a phase, report:

- What changed and why.
- Acceptance criteria met, with evidence.
- Tests/checks run and results.
- Known limitations or unresolved decisions.
- Files and docs updated.
- Recommended next phase, without starting it automatically.

Update `docs/ROADMAP.md` only for work actually completed. Leave future phases as planned.
