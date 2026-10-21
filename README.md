# Pixel Brew

Pixel Brew is an open-source semi-automated coffee-shop simulation and automation platform. It connects an Odoo-backed ordering flow, simulated coffee machines, a small human staff, and a pixel-art world that visualizes both customer-facing activity and backend events.

The project is intended to be built collaboratively by software, backend, frontend/game-UI, and embedded/IoT contributors. The first milestone is a reproducible local demo that needs no physical hardware or paid cloud service.

## Vision

The coffee shop should feel alive: many virtual customers arrive, place orders through the configured Odoo-backed ordering/POS flow, wait while drinks are prepared, and receive their drinks from a minimal number of human staff characters. The coffee machine is automated; a waiter still performs the human service step of picking up completed drinks and delivering them to the right customer or table. Customers can then finish their visit and leave.

Behind the scenes, confirmed demo orders become machine jobs through an integration service and message broker. A machine simulator processes jobs and publishes lifecycle events. Those real system states drive the world: customers queue, the machine works, the waiter collects ready drinks, and orders are delivered. The visualizer must never fake a successful machine or delivery state when the underlying event has not occurred.

The intended operating model is **high automation with minimal staff**, not a staffless shop. Customer behavior, machine behavior, and waiter delivery should be separate entities with explicit state transitions.

## MVP

- One local Odoo POS instance and a small demo menu.
- Virtual customers interacting with the configured ordering/POS UI; start with one deterministic browser journey, then scale to several concurrent customers.
- One simulated coffee machine with an explicit job lifecycle.
- A small staff roster, initially one waiter who collects completed drinks and delivers them to the correct customer/table.
- A simple shop scene with entry/exit, ordering point, waiting area, machine/counter, and a few tables or pickup locations.
- A documented command/event contract and local message broker/integration service.
- A pixel-art visualizer driven by observed system events and current state.
- Repeatable local setup and automated success/failure tests.

The first vertical slice may use one customer, one drink, one machine, and one waiter to prove the flow. The intended visual milestone then expands to multiple customers whose orders overlap naturally, while machine and waiter activity remains consistent with backend state.

The MVP does **not** require a physical coffee machine, real payment processing, paid cloud hosting, multi-branch operations, or production hardware control. Browser-driven customers are demo/test actors, not a production self-service product.

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
