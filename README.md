<div align="center">
  <img src="https://raw.githubusercontent.com/AutomataNexus/NexusKestrel-Releases/main/docs/img/nexuskestrel-lockup.png" alt="NexusKestrel" width="640"/>

  <h1>NexusKestrel</h1>

  <p><strong>A secure, multi-protocol field controller and supervisor for building automation, in pure Rust on a single microcontroller.</strong></p>

  [![Firmware](https://img.shields.io/badge/firmware-Rust%20no__std-orange.svg?logo=rust)](https://www.rust-lang.org)
  [![Runtime](https://img.shields.io/badge/runtime-Embassy%20async-orange.svg)](#architecture)
  [![MCU](https://img.shields.io/badge/MCU-RP2350%20Cortex--M33-c51a4a.svg?logo=raspberrypi)](#hardware)
  [![Secure Boot](https://img.shields.io/badge/secure%20boot-signed%20images-2da44e.svg)](#security)
  [![OTA](https://img.shields.io/badge/OTA-A%2FB%20signed%2C%20auto%20rollback-2da44e.svg)](#security)
  [![BACnet](https://img.shields.io/badge/BACnet-MS%2FTP%20%7C%20IP-14b8a6.svg)](#protocols)
  [![Modbus](https://img.shields.io/badge/Modbus-RTU%20master%20%7C%20slave-14b8a6.svg)](#protocols)
  [![Control](https://img.shields.io/badge/Talos-73%20control%20algorithms-14b8a6.svg)](#local-control)
  [![Supervision](https://img.shields.io/badge/supervises-160%20devices%20%2F%204096%20points-2d2a26.svg)](#supervision)
  [![On-device ML](https://img.shields.io/badge/on--device%20ML-per--unit%20fault%20models-purple.svg)](#on-device-fault-detection)
  [![License](https://img.shields.io/badge/license-Proprietary-blue.svg)](LICENSE)

  [What it is](#what-it-is) ·
  [Capabilities](#capabilities) ·
  [Hardware](#hardware) ·
  [Security](#security) ·
  [Licensing](COMMERCIAL.md) ·
  [Contact](#contact)

</div>

---

This is the public **product, documentation, and legal** home for NexusKestrel. The
firmware source is maintained privately by AutomataNexus, LLC; this repository holds
no code. It contains the product description, the license, the end-user license
agreement, the terms, the privacy, security, and safety policies, and (as they are
published) release notes and signed firmware releases.

## What it is

Kestrels are the smallest falcons. They hover over one patch of ground and work it.
A NexusKestrel sits at the equipment: it reads what is wired to it, drives relays and
0-10 V outputs, talks to the field bus, runs the control sequence locally, and reports
to the NexusEdge Console.

One firmware image runs in three roles, set per installation:

| Role | What it does |
|---|---|
| **Equipment controller** | Runs the control sequence for one or more pieces of equipment (AHU, boiler plant, heat pump, unit ventilator, pumps) from its own inputs and outputs. |
| **Supervisory controller** | Reads, trends, alarms, and commands a building's existing field controllers over their own protocols, the job a JACE or a legacy supervisor does, without replacing the field controllers. |
| **Protocol gateway** | Presents what it reads as a BACnet device and a Modbus slave, so another BMS can see equipment it could not reach before. |

With no sensors fitted it is a compact multi-protocol supervisor. With I/O boards fitted
it is a full equipment controller. Both run on the same microcontroller and the same
signed firmware.

## Capabilities

### Protocols

| Protocol | Direction | Status |
|---|---|---|
| BACnet MS/TP | device (server) and master; reads and writes field devices | Available |
| BACnet/IP | device (server), and a client for routed networks (ReadPropertyMultiple, COV) | Available |
| Modbus RTU | slave to a supervisor or BMS; master to field devices and I/O boards | Available |
| Allen-Bradley PCCC over EtherNet/IP | MicroLogix / SLC data-table reads and writes | Available |
| Niagara oBIX | client to an existing Niagara station | Available |
| Siemens APOGEE P2 | supervisor of APOGEE field panels | Available |
| Johnson Controls N2 | field bus | Available |
| Carrier CCN | reads | Available |
| LonWorks FT-10 | reads and writes (transmit and receive) | Available |

### Supervision

- Supervises up to **160 field devices and 4,096 points** per Kestrel. The point table
  lives in flash; RAM holds only live values.
- Change-of-value subscriptions where the field device supports them, polled reads
  where it does not, with per-device offline tracking and back-off.
- **Every command is confirmed by read-back.** Each write reports *confirmed*,
  *failed*, or *unconfirmed*; a command is never assumed to have landed.
- Operator overrides with priority and release, an audit record for every command,
  and per-site command policy (who may command what).
- Integration sheet on demand: identity, network settings, per-port protocol settings,
  the full BACnet object list, an EDE export, and the Modbus register map.

### Local control

- Control logic is **Talos**, the same Rust control-algorithm library that runs on the
  NexusEdge controller fleet, ported to the microcontroller with its tests.
  **73 algorithms** are available: AHU and zone sequences, staged and modulating boiler
  plants, lead/lag pumps, VFD pump packs, heat pumps, unit ventilators, two-pipe
  changeover, chillers, PID, and more.
- Each image carries the algorithms its job needs. Up to **16 local control loops** run
  per image (8 when the on-device fault model is also loaded). The loop count is a
  memory budget per image, not a limit on the devices a Kestrel supervises.
- Setpoints, schedules, and algorithm parameters are delivered as **signed
  configuration bundles**, so a sequence change never needs a firmware change.
- Fail-safe outputs on loss of control, and plant equipment with its own safeties is
  enabled and disabled, never second-guessed.

### On-device fault detection

Each unit can carry a small fault model trained on **that unit's own operating
history**, not on a generic equipment class. The models are trained with
[AxonML](https://github.com/AutomataNexus/AxonML), our Rust ML framework, and run on
the microcontroller. They flag short cycling, capacity loss, stuck sensors, and
failure to reach setpoint, and publish their verdicts to the Console.

### NexusEdge Console

A Kestrel appears in the NexusEdge Console as its own controller: live values, trends,
alarms, equipment graphics, commands with read-back, firmware and configuration
updates, and the integration sheet. Alarm e-mail and the operator command path are
handled by the Console.

## Hardware

| | |
|---|---|
| Microcontroller | Raspberry Pi RP2350: dual Arm Cortex-M33, 520 KB SRAM, 4 MB flash, OTP key storage |
| Networking | 2.4 GHz Wi-Fi; 10/100 wired Ethernet (falls back to Wi-Fi if the wired link fails) |
| Field bus | 2 x RS-485 ports (one supervisor/BMS-facing, one field-facing) |
| I/O (with expansion boards) | up to 16 analog inputs, 16 analog outputs (0-10 V), 8 relay outputs, 8 digital inputs |
| Sensors | 10K Type II and 1-Wire temperature, differential pressure, occupancy |
| Enclosure | DIN-rail enclosure with lever-style field terminals |

## Security

The Kestrel is designed for networks where a controller must not be the way in.

- **Secure boot.** The chip runs only firmware signed with AutomataNexus keys, held
  in one-time-programmable memory.
- **A/B signed over-the-air updates with automatic rollback.** A new image runs on
  trial; if it does not come up healthy, the Kestrel returns to the previous image on
  its own. Updates are compressed for slow links.
- **Signed configuration bundles.** Configuration is verified before it is applied.
- **Encrypted uplink.** TLS 1.3 to the Console with a per-device credential, scoped to
  that one controller and revocable.
- **Our own encrypted operator link.** Local operator access uses an AutomataNexus
  protocol built on vetted primitives (X25519, Ed25519, ChaCha20-Poly1305), not a
  general-purpose remote shell.
- **NexusShield-Kestrel.** A tamper-evident SHA-256 audit chain, a write-rate governor
  that protects equipment from rapid cycling, and field-bus threat scoring that bans
  writes from a misbehaving bus peer while leaving off-commands and operator access
  intact.

See [SECURITY.md](SECURITY.md) for how to report a vulnerability.

## Architecture

- Pure Rust, `no_std`, on the Embassy async runtime. No operating system, no C runtime
  in the control path.
- Protocol stacks are AutomataNexus code, host-tested and gated before every build.
- Every build passes a gate of lint-clean builds across all supported feature sets and
  the full host test suites before it can be signed.

## Licensing and purchase

NexusKestrel is proprietary hardware and firmware, sold and supported by
AutomataNexus, LLC, and installed by our controls technicians or authorized partners.
See [COMMERCIAL.md](COMMERCIAL.md).

| Document | |
|---|---|
| [LICENSE](LICENSE) | Copyright and license notice |
| [EULA.md](EULA.md) | End User License Agreement for the firmware |
| [TERMS.md](TERMS.md) | Terms of Service |
| [PRIVACY.md](PRIVACY.md) | What the device and the Console collect |
| [SECURITY.md](SECURITY.md) | Vulnerability reporting and disclosure |
| [SAFETY.md](SAFETY.md) | Safety notice and intended use |
| [TRADEMARKS.md](TRADEMARKS.md) | Trademarks and third-party marks |
| [COMMERCIAL.md](COMMERCIAL.md) | Purchasing, licensing, and support |
| [CHANGELOG.md](CHANGELOG.md) | Release notes |

## Contact

**AutomataNexus, LLC** · Wolcottville, Indiana, USA

- Sales and integration: [devops@automatanexus.com](mailto:devops@automatanexus.com)
- Security: [security@automatanexus.com](mailto:security@automatanexus.com)
- Legal: [legal@automatanexus.com](mailto:legal@automatanexus.com)
- Web: [automatanexus.com](https://automatanexus.com)

---

<div align="center">

**© 2026 AutomataNexus, LLC.** All rights reserved.
NexusKestrel, NexusEdge, NexusShield, Talos, and AxonML are trademarks of AutomataNexus, LLC.

</div>
