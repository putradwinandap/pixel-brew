# Pixel Brew

Pixel Brew is an open-source coffee-shop automation demo that connects a POS workflow, a simulated coffee machine, and a pixel-art visualization of backend activity.

The goal is to make the system observable and useful for collaboration between software, backend, and embedded/IoT contributors. The first milestone is a fully local demo that works without physical hardware.

## Vision

A virtual customer places an order through Odoo POS. After the order reaches the configured confirmed/paid state, an integration service creates a machine job and sends a command through a message broker. A coffee-machine simulator processes that command and publishes status events. The visualizer turns those real events into an animated pixel-art coffee shop.

The visualization must reflect the actual application state. It must not invent successful activity when a service or machine is disconnected.

## MVP

- One Odoo POS instance and one test product.
- One virtual customer flow through the POS UI.
- One simulated coffee machine.
- A documented command/event contract.
- A local message broker and integration service.
- A pixel-art visualizer driven by backend events.
- Repeatable local setup and automated end-to-end tests.
- Failure demonstrations such as machine offline, timeout, duplicate command, and failed brew.

The MVP does **not** require a physical coffee machine, real payment processing, cloud deployment, multi-branch operations, or production hardware control.

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
