# Pixel Brew — Phased Roadmap

This roadmap is an execution contract, not a request to build everything at once. Work in order, one phase per task/session, and stop when that phase's acceptance criteria are met.

## Product outcome

Demonstrate one complete local flow:

1. A fake customer interacts with the real Odoo POS UI and places a test order.
2. After the documented confirmation/payment gate, Odoo integration emits an order-ready-for-production intent.
3. An integration service creates a machine job and sends a versioned command through a message broker.
4. A coffee-machine simulator accepts and processes the job, then emits lifecycle events.
5. The integration layer records/normalizes those events and makes them available to Odoo and the visualizer.
6. A pixel-art coffee shop visualizer animates only the states supported by those events.
7. Automated tests verify the successful path and representative failures.

The MVP is a local demonstration with simulated equipment—not a production POS deployment or proof of physical-machine safety.

## Non-goals for the first working demo

Do not expand the MVP into these unless the user explicitly changes scope:

- Real payment processing or handling real card data.
- Direct physical pump, boiler, heater, pressure, or other actuator control.
- Multi-branch, inventory forecasting, loyalty, delivery, or enterprise analytics.
- Kubernetes, microservices for every component, or cloud deployment before local value is proven.
- An autonomous agent that can operate production equipment.
- A large pixel-art world before the end-to-end event path works.

## Proposed logical architecture

```text
Fake Customer (browser automation)
          |
          v
       Odoo POS
          |
          | confirmed test order (document the exact gate)
          v
Odoo add-on / POS integration adapter
          |
          v
Integration service / job orchestrator
          |
          +------> persisted order-to-job correlation and status
          |
          v
Private message broker (e.g. MQTT)
          |
          v
Machine interface
    /             \
Simulator       Future firmware adapter
    |
    v
machine lifecycle events
    |
    v
Integration event handler / read model
    |                         |
    v                         v
Odoo order status        Visualization feed
                              |
                              v
                    Pixel-art coffee shop UI
```

This is a logical view, not a mandate to create every box as a separate deployable service. Keep the initial implementation as simple as practical, while preserving clear interfaces. The browser visualizer must not connect to a privileged broker using embedded credentials and must not access Odoo's database directly.

## Phase 0 — Discovery, product boundaries, and architecture decisions

**Goal:** make the first implementation choices deliberately, before building feature code.

Tasks:
- Inspect the live repository and confirm the actual starting state.
- Clarify and document the MVP user journey and machine/order state boundaries.
- Research current official documentation for the selected Odoo version, Community/Enterprise feature boundaries, POS extensibility, deployment options, the chosen broker, and relevant client libraries.
- Choose a locally reproducible development approach. Prefer tools that can be run without paid cloud services.
- Decide the minimum set of languages and frameworks for Odoo integration, orchestration, visualizer, simulator, and browser automation. Do not add a framework without a clear need.
- Compare a small number of viable options for the pixel-art renderer; decide based on prototype complexity, browser support, testability, and contributor familiarity.
- Record key decisions in short ADRs under `docs/decisions/`, including alternatives considered and consequences.
- Define a threat/safety boundary and a test strategy.
- Identify any unresolved dependency that truly blocks implementation.

Deliverables:
- `docs/decisions/ADR-0001-stack-and-local-development.md`
- `docs/decisions/ADR-0002-order-and-machine-boundaries.md`
- `docs/decisions/ADR-0003-event-transport-and-visualization-feed.md`
- A concise MVP flow and glossary in project docs.

Acceptance criteria:
- The selected stack and versions have links to current official sources.
- Odoo edition/version and how the target POS flow is extended are explicitly justified.
- The local development and testing approach is documented.
- Machine state is not conflated with payment/order state.
- Physical hardware assumptions are explicitly marked unknown, not guessed.
- No application feature code is added merely to get ahead of later phases.

Stop after this phase. Present the decisions and wait for the user to start Phase 1.

## Phase 1 — Repository and local development foundation

**Goal:** make the project easy for another contributor to run without hardware.

Tasks:
- Establish a clear top-level structure only after Phase 0 decisions.
- Add a reproducible local development setup for the selected Odoo edition/version and broker.
- Add environment-variable documentation and a sanitized `.env.example`; keep secrets out of git.
- Add health checks/readiness guidance, useful logs, and deterministic service startup behavior.
- Pin or constrain dependency versions responsibly.
- Add baseline formatting, linting, and test commands appropriate to the chosen stacks.
- Document platform requirements and troubleshooting. Avoid requiring an external paid service.

Acceptance criteria:
- A new contributor can follow the documented instructions from a clean checkout.
- Services start in a documented sequence and report when ready.
- No secrets or machine-specific local state are committed.
- A minimal smoke check verifies the expected local services are reachable.
- Setup and tests are recorded accurately in the README.

## Phase 2 — Shared domain model and versioned contracts

**Goal:** define the language and rules used by Odoo integration, orchestrator, simulator/firmware, and visualizer.

Tasks:
- Define separate order/payment state and machine-job state models.
- Define command and event envelopes with schema version, unique ID, type, UTC timestamp, source, correlation ID, aggregate/job ID, and typed payload.
- Define a minimal menu/recipe identifier contract without assuming a particular machine's capabilities.
- Define state transitions, retryable vs terminal errors, timeouts, duplicate delivery behavior, and cancellation rules.
- Define an error model for machine offline, rejected command, timeout, brew failure, and malformed/unsupported payload.
- Store representative examples and validation rules in the repository.
- Add schema validation and contract tests before integrating components.

Suggested initial job lifecycle (adjust with evidence during Phase 0/2):
`queued → accepted → heating → brewing → completed`, with explicit `failed` and `cancelled` terminal states where valid. Not every machine must implement every intermediate state; document capability-dependent states.

Acceptance criteria:
- Contracts are documented and versioned.
- Every state transition has clear semantics and a test.
- Duplicate commands/events cannot create duplicate machine jobs or falsely complete orders.
- Event ordering, timeout behavior, and unknown states have explicit behavior.
- Contracts do not include secrets or assume real hardware details.

## Phase 3 — Machine simulator and integration orchestration

**Goal:** exercise the command/event protocol without Odoo or physical hardware.

Tasks:
- Implement a machine simulator behind the documented machine interface.
- Implement a small orchestrator that accepts an authorized job request, assigns/correlates a job ID, publishes a machine command, and consumes lifecycle events.
- Simulate realistic but configurable stages, delays, success, and failure.
- Make the simulator support deterministic test mode so tests do not rely on arbitrary long sleeps.
- Add duplicate command, disconnected machine, timeout, unsupported recipe, and failed-brew scenarios.
- Expose a narrow status/read interface for diagnostics; keep broker credentials private.

Acceptance criteria:
- A test command produces a full simulated lifecycle and final result.
- Every emitted event is validated against the shared contract.
- Retries and repeated messages do not silently create duplicate work.
- Failure cases result in explicit error states and useful logs.
- The simulator makes no claim that coffee or hardware action occurred physically.

## Phase 4 — Odoo POS integration

**Goal:** have a real test order create a machine job only at a clearly defined, documented business-state gate.

Tasks:
- Implement the smallest Odoo add-on/adapter appropriate for the version selected in Phase 0.
- Decide and test the exact gate (for example, confirmed/paid according to the configured demo payment flow); never trigger on an unfinished cart or merely selected product.
- Map supported POS products to recipe IDs/capabilities without trusting arbitrary browser input.
- Reliably correlate the Odoo order, orchestrator job, and machine IDs.
- Handle retries safely if Odoo or the integration service sends the same request more than once.
- Reflect machine progress separately from financial/payment state.
- Keep a clear manual recovery/error state if machine execution fails after the POS order is confirmed.

Acceptance criteria:
- One manually created test order crosses the documented gate and produces exactly one machine job.
- Unconfirmed/cancelled carts do not start a machine job.
- Integration errors are visible and diagnosable without corrupting payment/order state.
- Automated tests cover duplicate requests, unsupported recipe, unavailable broker, and failed machine job.
- No real payment gateway or hardware is needed.

## Phase 5 — Fake customer simulator

**Goal:** reproduce the customer journey through the actual POS interface.

Tasks:
- Use browser automation (for example, Playwright if selected in Phase 0) as a separate development/test tool.
- Create a clearly labeled demo customer flow that opens the POS, selects a test item, and completes the configured mock/test checkout path.
- Avoid hard-coded fragile selectors when stable labels, test IDs, or documented extension points are available.
- Keep test identities and credentials local and non-production.
- Capture enough browser logs/screenshots/traces to diagnose failures without recording sensitive data.
- Make the scenario deterministic and callable from one documented command.

Acceptance criteria:
- A clean local run can execute the customer journey end-to-end in the browser.
- The fake customer uses the real POS UI rather than writing directly to the application database.
- The test does not use real payment methods or real customer data.
- Failures produce actionable diagnostics and a non-zero test result.
- The customer simulator can be disabled without affecting the machine simulator or visualizer.

## Phase 6 — Pixel-art coffee shop visualizer

**Goal:** create the distinctive interface after real event flows exist.

Tasks:
- Select and implement the smallest renderer/UI architecture supported by the Phase 0 ADR.
- Create a first scene with cashier/order point, one customer, one espresso machine, and an order pickup point.
- Build a read-only visualization feed from normalized backend state/events; use SSE/WebSocket or another documented narrow interface as justified by the chosen stack.
- Define a visual state mapping: arrival/order, waiting/queued, machine accepted, heating, brewing, completed/pickup, and error/offline.
- Show selected order/job details and a compact event timeline so the scene is useful for debugging as well as presentation.
- Add connection-lost and stale-data indicators.
- Provide a way to select or inspect one demo order and trace its correlation ID.
- Keep pixel assets replaceable and document their sources/licenses. Do not copy assets from Pixel Agents or other projects without compatible permission.

Acceptance criteria:
- Visual states are driven by received system events or a documented state snapshot, not local timers pretending the backend succeeded.
- A single order can be followed visually from queue to completion.
- Disconnecting the feed produces a visible stale/disconnected indicator, not a false success animation.
- The UI remains usable at common desktop viewport sizes and has an accessible textual status/event fallback.
- No privileged MQTT credentials or direct database access exist in frontend code.

## Phase 7 — End-to-end demo and failure observability

**Goal:** prove the components work together and show what happens when things go wrong.

Tasks:
- Add a documented one-command or small, explicit command sequence to start the stack and run the demo.
- Run the fake customer → Odoo → orchestrator → broker → simulator → events → visualizer journey.
- Add repeatable integration/end-to-end tests where practical.
- Add scenarios for machine offline, command timeout, duplicate command, rejected recipe, broker restart, simulator crash, failed brew, and visualizer disconnection.
- Ensure logs include correlation IDs across services and avoid leaking secrets.
- Document reset/cleanup steps for test data.
- Add a small demo script or recording checklist only after the flow is repeatable.

Acceptance criteria:
- The success scenario is repeatable from documented local instructions.
- Failure scenarios produce explicit and consistent states.
- An operator can identify the affected order/job and trace it across components.
- Tests pass or known environmental limitations are stated with evidence.
- Screenshots/demo claims match the actual working software.

## Phase 8 — Embedded/IoT adapter collaboration

**Goal:** prepare for a real hardware contributor without coupling the project to unknown hardware.

Prerequisites:
- The hardware target, controller, command interface, electrical/mechanical safety constraints, and responsible hardware partner must be identified. Until then, do not claim physical-machine support.

Tasks:
- Publish the machine interface specification, event schemas, example traces, simulator, and a contributor issue for a first hardware adapter.
- Record device-specific capabilities and mapping from supported recipes to machine operations.
- Implement firmware in a dedicated area or package appropriate to the selected controller and toolchain.
- Add local command validation, bounded execution, watchdog/timeout handling, error reporting, and safe handling of lost network connectivity.
- Require physical safety mechanisms and interlocks to be enforced by the hardware/controller design, independently of the browser UI or cloud/backend.
- Use fake/simulated commands and tests first. Add hardware-in-the-loop testing only when real approved equipment is available.
- Document emergency stop, recovery, calibration, and operator responsibilities for the chosen hardware.

Acceptance criteria:
- The adapter conforms to the versioned interface and passes shared contract tests where feasible.
- Unsupported commands fail safely and explicitly.
- Network loss or backend failure cannot be interpreted as successful brewing.
- Hardware-specific safety review and test evidence exist before any real actuation demo.
- Documentation clearly distinguishes simulator-tested behavior from hardware-tested behavior.

## Phase 9 — Open-source contributor readiness

**Goal:** make collaboration approachable and sustainable.

Tasks:
- Add contribution guidelines, code of conduct if desired, issue templates, and a pull request template.
- Document development commands, architecture diagrams, component ownership boundaries, contract changes, and test expectations.
- Add CI for the checks the project can run reliably.
- Publish a roadmap status table, known limitations, and starter issues for backend, Odoo, visualizer, testing, and embedded contributors.
- Review dependency licenses and third-party asset licenses.
- Add a demo walkthrough and a clear warning that the early project is not certified for production equipment.

Acceptance criteria:
- A new contributor can locate the right component, run tests, and submit a focused change.
- CI runs documented checks and does not imply physical hardware safety.
- All committed assets and dependencies have reviewed license terms.
- Open issues distinguish software-only contributions from hardware-dependent work.

## Phase gates and working method

For every phase:

1. Inspect the current branch, working tree, files, and existing implementation.
2. Restate the phase goal and acceptance criteria.
3. Create a short implementation plan and note assumptions.
4. Implement only the smallest slice required for that phase.
5. Run relevant tests and inspect the final diff.
6. Update docs and phase status based on evidence.
7. Report results, tests, limitations, and the next recommended phase.
8. Stop. Do not start the next phase until the user asks.

Do not mark a phase completed merely because its files exist. Mark it complete only when its acceptance criteria pass or clearly state which criteria remain blocked.

## Roadmap status

- [ ] Phase 0 — Discovery and architecture decisions
- [ ] Phase 1 — Repository and local development foundation
- [ ] Phase 2 — Shared domain model and contracts
- [ ] Phase 3 — Machine simulator and orchestration
- [ ] Phase 4 — Odoo POS integration
- [ ] Phase 5 — Fake customer simulator
- [ ] Phase 6 — Pixel-art visualizer
- [ ] Phase 7 — End-to-end demo and failure observability
- [ ] Phase 8 — Embedded/IoT adapter collaboration
- [ ] Phase 9 — Open-source contributor readiness
