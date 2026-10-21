# Pixel Brew — Phased Roadmap

This roadmap is an execution contract, not a request to build everything at once. Work in order, one phase per task/session, and stop when that phase's acceptance criteria are met.

## Product outcome

Demonstrate a semi-automated coffee shop that feels alive and is backed by real system state:

1. A virtual customer arrives and orders through the configured Odoo-backed ordering screen.
2. For cash orders, the customer joins the cashier queue. A cashier character receives the virtual cash payment and records the transaction through Odoo POS.
3. Only after Odoo reports the configured payment/confirmation gate is satisfied does the integration emit an order-ready-for-production intent.
4. An integration service creates a machine job and sends a versioned command through a message broker.
5. A coffee-machine simulator processes the job and emits lifecycle events.
6. When the machine reports the drink ready, a waiter character collects it and delivers it to the correct customer/table or pickup location.
7. The customer receives the order, completes the visit, and leaves.
8. A pixel-art coffee shop scene shows many customer visits and overlapping orders, driven by observed system state rather than invented success animations.
9. Automated tests verify the cash path, happy path, and representative failure/recovery scenarios.

Default simulated staff: one cashier and one waiter. A second waiter is an optional high-traffic configuration, not a required dependency for the initial MVP.

Build this incrementally: first one customer, one order, one simulated machine, and one waiter; then multiple concurrent customers, queues, simple table/pickup assignment, and minimal staff scheduling. The MVP is a local demonstration with simulated equipment—not a production POS deployment or proof of physical-machine safety.

## Non-goals for the first working demo

Do not expand the MVP into these unless the user explicitly changes scope:

- Real-money transactions, real card processing, or a production QRIS/payment-provider integration.
- Making digital payment a prerequisite. Cash-first payment via Odoo POS is the required MVP path.
- A parallel custom payment ledger that duplicates Odoo's financial records.
- Direct physical pump, boiler, heater, pressure, or other actuator control.
- Multi-branch, inventory forecasting, loyalty, delivery, or enterprise analytics.
- Kubernetes, microservices for every component, or cloud deployment before local value is proven.
- An autonomous agent that can operate production equipment.
- A large pixel-art world before the end-to-end event path works.
- Complex staff rostering, wage/payroll systems, table-service optimization, full restaurant economics, or a complete business-management game.
- Treating customer/waiter animation as authoritative when backend/order/machine state is missing or stale.

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
- Clarify and document the MVP user journey and machine/order/payment state boundaries.
- Document the intended shop scene and operating model: multiple virtual customers, automated drink preparation, one cashier, one waiter, and an optional second waiter for high-traffic scenarios.
- Decide how the selected Odoo version supports the ordering screen/self-order path and cashier-based cash payments. Do not assume that cash can be paid directly inside the customer self-order UI.
- Define the cash checkout flow: order at screen → cashier queue → cash received/tendered amount recorded in Odoo POS → Odoo payment/validation state observed → machine job allowed.
- Keep QRIS/digital payment optional until the Odoo version, configuration/provider, and desired validation behavior have been verified.
- Define explicit state boundaries for customer visit, order, machine job, waiter delivery task, and table/pickup location; do not collapse them into one status.
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
- `docs/decisions/ADR-0004-payment-and-staffing-flow.md`
- A concise MVP flow and glossary in project docs.

Acceptance criteria:
- The selected stack and versions have links to current official sources.
- Odoo edition/version and how the target ordering/self-order screen and cashier payment flow are implemented are explicitly justified with official documentation.
- The cash payment gate is precise and does not depend on a visual animation.
- Staff responsibilities and default roster (one cashier, one waiter; optional second waiter later) are documented.
- Digital payment/QRIS is clearly optional and its future integration requirements are listed rather than assumed.
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
- Define separate customer-visit, order, payment, machine-job, cashier/payment action, waiter/delivery-task, and table/pickup-location state models.
- Define cash checkout semantics and authoritative payment fields from Odoo (amount due, amount tendered/change where supported, payment method, paid/validated state). Avoid duplicating Odoo's accounting/payment ledger.
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
- One cash order is created through the configured ordering/POS flow, paid at the cashier through Odoo POS, and produces exactly one machine job only after Odoo reports paid/validated.
- The cashier flow correctly records amount tendered and change using Odoo's supported POS workflow.
- Unpaid, partially paid, cancelled, or invalid orders do not start a machine job.
- Integration errors are visible and diagnosable without corrupting payment/order state.
- Automated tests cover duplicate requests, unsupported recipe, unavailable broker, and failed machine job.
- No real payment gateway or hardware is needed.

## Phase 5 — Fake customer simulator

**Goal:** reproduce the customer journey through the actual POS interface.

Tasks:
- Use browser automation (for example, Playwright if selected in Phase 0) as a separate development/test tool.
- Create a clearly labeled demo customer flow that opens the ordering screen, selects a test item, and then moves through the cashier queue for a cash payment.
- Automate the cashier's Odoo POS workflow in a separate, clearly identified test actor/session where possible; do not bypass Odoo by writing payment rows directly to the database.
- Keep the cash interaction explicitly simulated: the customer character hands over virtual cash, while Odoo POS records the test payment and calculates/records change.
- Avoid hard-coded fragile selectors when stable labels, test IDs, or documented extension points are available.
- Keep test identities and credentials local and non-production.
- Capture enough browser logs/screenshots/traces to diagnose failures without recording sensitive data.
- Make the scenario deterministic and callable from one documented command.

Acceptance criteria:
- A clean local run can execute the customer journey end-to-end in the browser.
- The fake customer uses the real configured ordering/POS UI rather than writing orders directly to the application database.
- The cash payment is recorded by the supported Odoo POS UI/workflow; the fake customer does not create a payment by writing directly to the database.
- The test does not use real money, real payment gateways, or real customer data.
- Failures produce actionable diagnostics and a non-zero test result.
- The customer simulator can be disabled without affecting the machine simulator or visualizer.

## Phase 6 — Pixel-art coffee shop visualizer

**Goal:** create the distinctive interface after real event flows exist.

Tasks:
- Select and implement the smallest renderer/UI architecture supported by the Phase 0 ADR.
- Create an initial scene with entrance/exit, ordering screen, cashier and cashier queue, waiting area, one coffee machine/counter, one waiter, and a few simple tables or pickup locations.
- Visualize the default staff roster of one cashier and one waiter. Allow one optional second waiter in a higher-traffic scenario only after the baseline service flow works.
- Implement the baseline customer journey: arrive → order → wait → order becomes ready → waiter collects drink → waiter delivers to the assigned customer/table → customer finishes and leaves.
- Make waiter delivery a separate task/state transition triggered only after authoritative machine/order-ready state. The visualizer must show delays, blocked deliveries, or errors honestly.
- After the single-customer path works, allow multiple customer characters and overlapping orders. Use a small initial staff roster (one waiter) and simple rules before considering more staff or scheduling logic.
- Distinguish a customer-facing ordering animation from the actual Odoo-backed order event. Never create a successful order solely because an animation played.
- Build a read-only visualization feed from normalized backend state/events; use SSE/WebSocket or another documented narrow interface as justified by the chosen stack.
- Define a visual state mapping: arrival/order, waiting/queued, machine accepted, heating, brewing, completed/pickup, and error/offline.
- Show selected order/job details and a compact event timeline so the scene is useful for debugging as well as presentation.
- Add connection-lost and stale-data indicators.
- Provide a way to select or inspect one demo order and trace its correlation ID.
- Keep pixel assets replaceable and document their sources/licenses. Do not copy assets from Pixel Agents or other projects without compatible permission.

Acceptance criteria:
- Visual states are driven by received system events or a documented state snapshot, not local timers pretending the backend succeeded.
- At least one customer can be followed visually from arrival, order placement, cashier cash payment, machine preparation, waiter pickup/delivery, and departure.
- An order cannot trigger machine preparation until Odoo's paid/validated state has been observed.
- The cashier queue and waiter delivery tasks are separate, observable activities; adding a second waiter does not change order/payment truth.
- Multiple customers can appear in the scene and at least two order lifecycles can overlap without confusing customer, order, machine job, or delivery task identities.
- The initial waiter only collects a drink after the machine/order reports ready and delivers it to the intended customer/table.
- A single order can be followed visually from queue to completion.
- Disconnecting the feed produces a visible stale/disconnected indicator, not a false success animation.
- The UI remains usable at common desktop viewport sizes and has an accessible textual status/event fallback.
- No privileged MQTT credentials or direct database access exist in frontend code.

## Phase 7 — End-to-end demo and failure observability

**Goal:** prove the components work together and show what happens when things go wrong.

Tasks:
- Add a documented one-command or small, explicit command sequence to start the stack and run the demo.
- Run the fake customer → Odoo → orchestrator → broker → simulator → readiness event → waiter delivery task → visualizer journey.
- Verify multiple customer arrivals and at least two overlapping demo orders after the single-order scenario is reliable.
- Verify unpaid orders do not trigger machine jobs, the cashier records test cash payments through Odoo POS, and any change calculation shown matches Odoo's recorded transaction.
- Verify the waiter does not collect an unfinished drink, a delivered order is linked to the right customer/table, and a failed/offline machine does not produce a false delivery.
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

- [ ] Phase 0 — Discovery and architecture decisions (including cash payment/staffing ADR)
- [ ] Phase 1 — Repository and local development foundation
- [ ] Phase 2 — Shared domain model and contracts
- [ ] Phase 3 — Machine simulator and orchestration
- [ ] Phase 4 — Odoo POS integration
- [ ] Phase 5 — Fake customer simulator
- [ ] Phase 6 — Pixel-art visualizer
- [ ] Phase 7 — End-to-end demo and failure observability
- [ ] Phase 8 — Embedded/IoT adapter collaboration
- [ ] Phase 9 — Open-source contributor readiness
