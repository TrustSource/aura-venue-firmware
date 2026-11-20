# Architecture: aura-venue-firmware

Version 0.1.0 · 2026-11-20 · Diagram: [ARCHITECTURE.drawio](ARCHITECTURE.drawio) (pages "System" and
"Hardware")

## System overview and purpose

The Venue Amp is a network amplifier. A building has many of them. Each contains a microcontroller,
a signal processor and a power stage. The controller tells it what to play and how loud; the
firmware receives audio and commands over the network, runs the signal processing and supervises
the power stage.

## Components and responsibilities

| Component | Responsibility | Origin |
|---|---|---|
| Application | command handling, device state, supervision | Aurelia Audio |
| Real-time kernel | tasks, timers, drivers | open source |
| Network stack and TLS | communication with the controller | open source, copied into the project |
| Update | receives a new image, stores it in the external flash, switches over | Aurelia Audio, partly open source |
| Signal-processor program | filters, limiter, delay | Aurelia Audio, loaded from a serial EEPROM or by the microcontroller |
| Mainboard | microcontroller, flash, network, signal processor, converter, power stage, service port | Aurelia Audio, real parts (`hardware/`) |

## Key interfaces and data flows

| From | To | What |
|---|---|---|
| Controller | amplifier | control commands and audio over Ethernet |
| Controller | amplifier | firmware update |
| Legacy systems | amplifier | remote control over RS-485 |
| Microcontroller | signal processor | parameters and program over SPI or I2C |
| Service technician | amplifier | debug port (SWD) |

Protocols are not fixed yet.

## Infrastructure and deployment

Nothing is deployed. The repository is input for analysis.

Communication with the TrustSource API (uploads and imports from CI) uses the organisation secret
`AURA_UPLOAD_KEY`, sent as the `x-api-key` header. The base URL including the API version comes from
`TS_API_BASE_URL`; it is not hard-coded. No workflow exists yet.

Mapping to TrustSource: one project "Aura Venue Firmware". The hardware bill of materials is
attached to the product node of the amplifier.

## Architecture decision records

### ADR-001: Four repositories, operating system separate

- **Date:** 2026-11-20 (v0.1.0)
- **Status:** Accepted
- **Context:** The example has to reflect how device manufacturers organise their work: a platform
  team builds the Linux, product teams build devices on top of it, and engineering software is a
  product of its own. A single repository would hide exactly the hand-over points that cause the
  difficulties we want to study. A separate operating-system repository also leaves room for OS
  variants.
- **Decision:** Four repositories: `aura-venue-studio`, `aura-venue-controller`, `aura-venue-os`,
  `aura-venue-firmware`. In TrustSource each one is a project of its own. The operating system is
  included in the controller project as a **linked module** (a module that refers to another project
  and one of its approvals). The purchased runtime is a third-party component of the controller.
- **Consequences:** The controller depends on a specific approved state of the operating system, not
  on its source. Cross-repository topics (system overview, measurement log) need one agreed home.
  Linked-module behaviour across OS versions becomes part of what is tested (see aura-venue-os).

### ADR-002: Realistic artefacts instead of running software, measured against a ground truth

- **Date:** 2026-11-20 (v0.1.0)
- **Status:** Accepted
- **Context:** The purpose is to learn where scanning and vulnerability matching succeed and where
  manual work remains. That needs realistic input and a reference to compare with, not a working
  product.
- **Decision:** Repositories contain realistic files (manifests, vendored sources, binaries, images,
  supplier documents) that need not build or run. Each repository carries a `truth.yaml` with the
  real content and the expected vulnerabilities, each entry backed by a source. Only freely licensed
  third-party files are used; no proprietary binaries of real suppliers.
- **Consequences:** Results are reproducible and can be published. Building `truth.yaml` is real
  effort and has to be kept in step with every change to the content.

### ADR-003: Hardware concept and HBOM live in the device repository

- **Date:** 2026-11-20 (v0.1.0)
- **Status:** Accepted
- **Context:** Hardware has no repository of its own. The hardware bill of materials (HBOM) is needed
  for the product record and as evidence for the device threat analysis (MITRE EMB3D).
- **Decision:** The hardware concept of the amplifier lives in `hardware/` of this repository; that
  of the controller in aura-venue-controller. HBOM documents are CycloneDX using hardware component
  types. Parts are real, common types so that manufacturer plus part number is a valid identity.
- **Consequences:** Hardware and firmware of the device are versioned together.

### ADR-004: Third-party C sources are copied into the project

- **Date:** 2026-11-20 (v0.1.0)
- **Status:** Proposed
- **Context:** Embedded C projects usually have no package manager. Third-party code is copied into
  the source tree, often modified, and its version is visible only in header files or not at all.
  This is the case in which a machine-readable component list is hardest to obtain.
- **Decision:** Third-party sources are kept as copies under `third_party/`, some unmodified, some
  with local changes. In addition a manifest of the real-time kernel's module tool describes part of
  the dependencies, so that both situations occur in one project.
- **Consequences:** Scan results will be incomplete by design; the gap against `truth.yaml` is the
  finding. Whether real projects look like this is to be confirmed with an embedded development
  partner.

## Open decisions

- Real-time kernel and module tool (current idea: Zephyr with its `west.yml` manifest).
- Type of the digital-to-analogue converter and of the signal processor's EEPROM.
- Protocols between controller and amplifier.
