# Pixel Brew

Pixel Brew is an open-source semi-automated coffee-shop simulation and automation platform. It connects an Odoo-backed ordering flow, simulated coffee machines, a small human staff, and a pixel-art world that visualizes both customer-facing activity and backend events.

The project is intended to be built collaboratively by software, backend, frontend/game-UI, and embedded/IoT contributors. The first milestone is a reproducible local demo that needs no physical hardware or paid cloud service.

## Vision

The coffee shop should feel alive: many virtual customers arrive, order through an Odoo-backed ordering screen, pay at the cashier when using cash, wait while drinks are prepared, and receive drinks from a minimal human staff. The intended initial roster is **one cashier and one waiter**. A second waiter can be enabled as a configurable scenario for busier periods; the default demo does not need more staff.

The cashier operates the Odoo POS register, receives the customer's virtual cash payment, enters the amount tendered, and handles any change. Odoo's recorded payment state is authoritative: showing a cash handover animation does not mean an order is paid. For the first demo, the cash workflow is simulated and uses a configured Odoo cash payment method; no real money is processed. The coffee machine is automated, while waiter(s) pick up drinks only after the machine reports them ready and deliver them to the right customer or table.

Behind the scenes, an order becomes a machine job only after the documented Odoo payment/confirmation gate. A machine simulator processes jobs and publishes lifecycle events. Those real system states drive the world: customers queue at the ordering screen and cashier, the machine works, waiter(s) collect ready drinks, and orders are delivered. The visualizer must never fake a successful payment, machine job, or delivery when the underlying event has not occurred.

The MVP is **cash-first but payment-method-aware**. QRIS or other digital methods may be evaluated as a later optional integration; they must not be required for the local demo, and a simulated digital success must never be presented as a real payment. Customer behavior, cashier work, machine behavior, and waiter delivery should be separate entities with explicit state transitions.

## MVP

- One local Odoo POS instance and a small demo menu.
- Virtual customers interacting with the configured ordering/POS UI; start with one deterministic browser journey, then scale to several concurrent customers.
- One simulated coffee machine with an explicit job lifecycle.
- A minimal staff roster: one cashier and one waiter by default; an optional second waiter can be enabled in a higher-traffic scenario.
- A cash-first checkout flow: a virtual customer orders at the Odoo-backed screen, then pays the cashier through the configured Odoo POS cash method; the cashier records amount tendered/change through Odoo.
- A simple shop scene with entry/exit, ordering screen, cashier/payment queue, waiting area, machine/counter, and a few tables or pickup locations.
- Payment and machine-job state sourced from Odoo/integration events, never inferred from character animation.
- A documented command/event contract and local message broker/integration service.
- A pixel-art visualizer driven by observed system events and current state.
- Repeatable local setup and automated success/failure tests.

The first vertical slice may use one customer, one drink, one machine, and one waiter to prove the flow. The intended visual milestone then expands to multiple customers whose orders overlap naturally, while machine and waiter activity remains consistent with backend state.

The MVP does **not** require a physical coffee machine, real-money transactions, a real QRIS/payment-provider integration, paid cloud hosting, multi-branch operations, or production hardware control. Cash is simulated and recorded using Odoo's configured cash method. QRIS/digital payment can be added later after the exact Odoo version, provider, and requirements are confirmed. Browser-driven customers are demo/test actors, not a production self-service product.

## Project principles

- **Simulation first:** the software demo must work without hardware.
- **Real events, honest visuals:** animations represent observed states, not assumptions.
- **Loose coupling:** Odoo, orchestration, machine implementations, and visualizer communicate through documented contracts.
- **Hardware adapter boundary:** firmware and physical device support are separate implementations of the machine contract.
- **Safety first:** no physical actuator control until the target hardware, its documented interface, and required safety interlocks have been reviewed.
- **Open collaboration:** architecture decisions, setup, tests, and contribution paths are documented.

## Start here

1. Read [CLAUDE.md](CLAUDE.md) before making changes.
2. Read [the phased roadmap](docs/ROADMAP.md).
3. Work on one phase at a time. Do not implement the entire roadmap in one pass.
4. Start with Phase 0: inspect assumptions, select a justified stack, and record architecture decisions before writing application features.

## Project status

Early planning / bootstrap. No hardware integration or production readiness is implied.

## Contributions

Contributions are welcome across Odoo/POS, backend integration, frontend visualization, testing, DevOps, and embedded/IoT. Until the machine protocol and hardware target are defined, embedded contributions should focus on interfaces, documentation, and simulation-compatible contracts—not unverified hardware actuation.
