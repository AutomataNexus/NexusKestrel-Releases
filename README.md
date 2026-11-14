<div align="center">
  <img src="https://raw.githubusercontent.com/AutomataNexus/NexusKestrel-Releases/main/docs/img/nexuskestrel-lockup.png" alt="NexusKestrel" width="640"/>

  <h1>NexusKestrel</h1>

  <p><strong>Mini-JACE, protocol gateway and equipment controller: a small, secure field controller for building automation, in pure Rust on a single microcontroller.</strong></p>

  [![Firmware](https://img.shields.io/badge/firmware-Rust%20no__std-orange.svg?logo=rust)](https://www.rust-lang.org)
  [![MCU](https://img.shields.io/badge/MCU-RP2350%20Cortex--M33-c51a4a.svg?logo=raspberrypi)](#hardware)
  [![Secure Boot](https://img.shields.io/badge/secure%20boot-signed%20firmware-2da44e.svg)](#security)
  [![OTA](https://img.shields.io/badge/OTA-signed%2C%20auto%20rollback-2da44e.svg)](#security)
  [![BACnet](https://img.shields.io/badge/BACnet-MS%2FTP%20%7C%20IP-14b8a6.svg)](#protocols)
  [![Modbus](https://img.shields.io/badge/Modbus-RTU%20master%20%7C%20slave-14b8a6.svg)](#protocols)
  [![Legacy](https://img.shields.io/badge/legacy-N2%20%C2%B7%20LON%20%C2%B7%20CCN%20%C2%B7%20Comm4%20%C2%B7%20APOGEE-14b8a6.svg)](#protocols)
  [![Sequences](https://img.shields.io/badge/Talos-69%20equipment%20sequences-14b8a6.svg)](#local-control)
  [![Supervision](https://img.shields.io/badge/supervises-160%20devices%20%2F%204096%20points-2d2a26.svg)](#capacity)
  [![Fault detection](https://img.shields.io/badge/fault%20detection-on--device%2C%20per%20unit-purple.svg)](#fault-detection)
  [![License](https://img.shields.io/badge/license-Proprietary-blue.svg)](LICENSE)

  [What it is](#what-it-is) ·
  [Protocols](#protocols) ·
  [Local control](#local-control) ·
  [Fault detection](#fault-detection) ·
  [Security](#security) ·
  [Hardware](#hardware) ·
  [Licensing](COMMERCIAL.md)

</div>

---

This is the public **product, documentation, and legal** home for NexusKestrel. The
firmware is designed and built in-house by AutomataNexus, LLC, and its source is
private; this repository holds no code. It contains the product description, the
license, the end-user license agreement, the terms, and the privacy, security, and
safety policies.

## What it is

Kestrels are the smallest falcons. They hover over one patch of ground and work it.

The NexusKestrel fills the role of a **mini-JACE**. It talks to equipment and building
systems over the field protocols used in our industry, runs the same control sequences
as our NexusEdge controllers, monitors its equipment for faults, and reports to the
NexusEdge Console. It does all of this on one low-power microcontroller, with no
Rockwell or Tridium licensing and no third-party runtime.

| Use | What it does |
|---|---|
| **Protocol gateway / supervisor** | Reads and writes BACnet, Modbus, Allen-Bradley PLCs, Niagara stations, and legacy trunks, and brings those points into the Console. This is how we take over a Niagara or other aging front end. |
| **Point controller** | Sensors, valves, dampers, relays, and drives are wired to it and it runs the unit locally, with the Console supervising. |
| **Integration bridge** | Presents a unit to a customer's existing BAS as a native BACnet or Modbus device. |

Each Kestrel is built for its job. Only the boards that unit needs are fitted, and its
firmware image carries only the sequences that unit runs. A different sequence later
is a signed over-the-air update of about a minute.

## Protocols

| Protocol | What the Kestrel does |
|---|---|
| **BACnet MS/TP** | Full BACnet device on either RS-485 port; MS/TP master and client that reads and writes other devices at a set priority, each write read back |
| **BACnet/IP** | Full BACnet device (UDP 47808, on Ethernet, Wi-Fi or both); supervisory client for routed BACnet networks. Off until enabled for a job, so a writable device never appears on a customer LAN unannounced |
| **Modbus RTU** | Slave on RS-485 #1 for a CM5 or BAS; master on RS-485 #2 for drives, meters, and boilers. Built-in ABB ACH580 / ACS580 / ACS550 drive profiles plus a generic profile |
| **Allen-Bradley PLCs** | MicroLogix 1100 / 1400 and SLC 5/05 over EtherNet/IP, using RSLogix addresses (N7:0, B3:1/0, T4:0.ACC, O:0/2). PLC mode and fault code monitored and logged |
| **Niagara AX / N4** | Through the station over oBIX (JACEs and FX80s, using the station's watch service so the front end barely notices), or under it, directly on the field trunk |
| **Niagara R2 (UNC-510)** | Reads points over HTTP and writes analog outputs through the station's own command interface, each write read back |
| **Siemens APOGEE** | Supervisor of APOGEE field panels over P2 |
| **Johnson Controls N2** | Decodes the trunk's polls and replies, so N2 points are read without disturbing the existing system; can ask each address whether a device is present |
| **LonWorks FT-10** | Reads and writes LON devices directly: decodes network variables and sends updates as its own node on the channel (Trane Tracer ZN fan coils and other LonMark devices) |
| **Carrier CCN** | Reads Carrier equipment directly as a CCN master |
| **Trane Comm4** | Reads Trane Comm4 unit controllers and writes their occupied / unoccupied heating and cooling setpoints, each write read back; stays off the trunk whenever another Comm4 master is active |
| **Staefa SMART II / NCRS** | Connects to the Staefa front end and commands its digital points |

**Trunk identification.** Pointed at an unknown RS-485 trunk, the Kestrel listens and
reports what it hears (N2, CCN, Siemens P1/FLN, MS/TP or Comm4) and why.

**Integration sheet.** The Kestrel generates its own integration documents: identity,
addresses and port settings, the full BACnet object list with units and current values,
an **EDE file** for the front end's BACnet import, and the Modbus map.

**Every command is read back.** Whether a command comes from the Console, the service
shell, Modbus, BACnet, or the Kestrel's own control, it is reported as **confirmed**,
**FAILED**, or **unconfirmed**, and is only reported complete when the read-back
confirms it.

## Local control

The Kestrel runs **Talos**, the same control engine as our CM5 and NexusEdge
controllers. Every Talos equipment sequence is ported with its logic unchanged and its
full test suite, so a Kestrel runs a unit exactly the way the CM5 beside it would.

**69 equipment sequences**, by family:

| Family | Sequences |
|---|---|
| Air handlers | TMC AHU · TMC AHU + Exhaust Damper · TMC AHU (Cooling-only) · TMC AHU 4-pipe Lead/Lag · TMC Zone AHU · TMC DX AHU · DX + CW AHU · TMC Make-Up Air · TMC Steam AHU · Rooftop Unit |
| Dedicated outdoor air | TMC Direct-Fired DOAS + DX · TMC DOAS + DX + Season Relay · DOAS Gas Heat + 2-Stage DX · DOAS Permissive |
| Zones and terminals | Fan Coil Unit (2-Way / 4-Way) · Classroom Unit Ventilator · TMC VAV · TMC VAV + DX · Zone Reheat · TMC Electric Heat · Heat Pump |
| Chillers, towers, geothermal | TMC Chiller · TMC Chiller (simplified) · TMC Chiller RTAA · TMC Chiller Plant · Water-Cooled Chiller · Chiller Remote Enable · Lead/Lag Chillers · Cooling Tower · TMC Geo Stage · TMC Seasonal Changeover |
| Boilers and steam | Lead/Lag Boiler · Cascade Boiler Plant · Cascade Boiler Plant (Lochinvar PowerFin) · TMC Domestic Boiler · TMC Comfort Boiler · Steam-Boiler Safety Monitor · TMC Steambundle · TMC Steam Reset |
| Pumps and loops | VFD Pump Pack · Lead/Lag Pumps · TMC Loop Pump Plant · TMC Circulator Pump · TMC Lead/Lag Enable · Pressure Booster Pumps · Pressure Booster Pumps (DP Proof) |
| Pools and natatoriums | Natatorium PoolPak · TMC Natatorium · TMC Natatorium AHU · Pool Controller (Heat Pump) · Pool Controller (Gas Heater) |
| Greenhouses | TMC Greenhouse · TMC Greenhouse Plant |
| Residential | Residential Furnace · Residential Heat Pump · Residential Boiler · Smart Home Controller · Smart Garden |
| Lighting and monitoring | Commercial Lighting · Electrical Load Monitor · Water Monitor (City) · Water Monitor (Well) |
| General-purpose loops | PID · Linear Proportional · On/Off · Cascade · TMC · Always-On Enable · Model Predictive Control (advisory) |

- **Custom logic from NexusFoundry.** Technicians open any sequence as a wire sheet of
  logic blocks, modify it or build new logic, test it in the live simulator by tripping
  safeties and alarms, and export it, once it passes Foundry's verification gates and
  is signed, as a native sequence for the Kestrel.
- **Point assignments.** Each sequence input is assigned to a point, a combination of
  points, or a scaled reading (for example loop DP = AI 1 × 20 − AI 2 × 20, or a high
  limit built from a supply sensor). A sensor that is not reading is treated as
  missing; the sequence never substitutes a value.
- **Life safety.** Every sequence honors the safety inputs it uses (smoke, fire alarm,
  freeze stat, flow, high limit, airflow proof, damper end switch), all normally
  closed. A failed sensor opens its limit and trips; it never reads as safe.
- **Hand and overrides.** An output placed in hand from the Console, the shell, Modbus
  or a BACnet priority write is left alone until released, and the sequence resumes
  smoothly. Relays keep a minimum off time, including after power-up, so a reset
  cannot short-cycle a compressor.
- **Plant templates.** Tested configurations for common plants, such as lead/lag
  boilers with outdoor-air reset and lead/lag loop pumps on differential pressure, and
  classroom unit ventilators, filled in per site.
- **Settings change live**, one at a time or as a signed settings bundle, with no
  rebuild and no restart. Sequences on a local schedule use the site's local time.

## Fault detection

**On-device fault detection.** The Kestrel carries a fault-detection model trained for
the unit it monitors. Every 3 minutes it scores that unit; three abnormal scores in a
row raise a flag in the Console with the likely fault. The model runs on the Kestrel's
own processor, with no accelerator and no cloud round trip. An indicator light on the
board is lit while it scores.

**On-device learning.** The Kestrel can refine its own model from the unit's install
labels while it runs, in small steps and only when control has spare capacity.
**Control always takes priority.** A refined model is not used until the AutomataNexus
server has re-checked it against the model the unit was commissioned with (no worse on
known conditions, no new false alarms) and signed it for that controller. The Kestrel
checks that signature before using the model and again at every start. Anything
unsigned or unverified is discarded.

## Capacity

| Limit | Per Kestrel |
|---|---|
| Supervised devices and points (supervisory image) | 160 devices / 4,096 points, from a point table in flash |
| Equipment under local control at once | 16, each with its own sequence, parameters, and inputs (8 with on-device learning running) |
| Field points | 128, shared by all equipment and the BACnet / Modbus mirrors |
| BACnet objects | 192, with 16-level priority arrays and change-of-value subscriptions |
| Inputs per sequence | 32 named inputs, 32 outputs |
| Settings | 16 KB, kept as two alternating copies in flash; changes take effect live |
| Control pass | every 2 seconds, using a small fraction of the processor's time |

## The NexusEdge Console

The Kestrel connects **outbound only** to the NexusEdge Console over TLS 1.3, verifying
the server against our own certificate authority built into its firmware. It sends
status and readings every 30 seconds and checks for commands every 5 seconds. It uses
wired Ethernet when present and falls back to Wi-Fi automatically.

Each unit the Kestrel runs appears in the Console as its own equipment, with the same
field names as every other controller in the fleet, so a Kestrel's boilers and pumps
sit beside a CM5's on the same pages and alarms.

## Security

Security is layered from the chip up, and each layer holds even if another fails.

- **Secure boot.** The RP2350 checks the firmware's signature at every start and runs
  only firmware signed with the AutomataNexus key, which never leaves our build
  machines. Altered firmware will not start, and a flash dump copied to another board
  will not start there. The partition layout is signed too.
- **Device identity.** Every Kestrel has its own key, accepted by the Console only for
  that controller. A lost or retired unit is shut out by revoking its key.
- **No internet-facing services.** The Console link is outbound only. Remote service
  runs only inside AutomataNexus's own encrypted, peer-authenticated mesh. BACnet/IP is
  off by default.
- **Encrypted direct link on site.** On a LAN with no internet, only enrolled
  technician laptops can connect, over our own protocol authenticated in both
  directions with fresh session keys; any replayed or altered message ends the session.
- **Signed updates with automatic rollback.** An update is verified on arrival, again
  after it is written, and by the processor on first start. It runs on trial; if it is
  not healthy within five minutes, or crashes or hangs, the previous firmware is
  restored automatically.
- **Signed settings and signed models.** Settings bundles and refined fault models are
  signed for one specific Kestrel and refused by every other. A bundle is checked as a
  whole before anything is saved.
- **NexusShield on the Kestrel.** A write governor (per-output change limits and relay
  minimum off times) on every write from every source; a tamper-evident hash-chained
  audit log whose head is reported to the Console; and bus protection that refuses
  writes from a misbehaving Modbus bus for 10 minutes while reads continue. Turning an
  output **off** is always allowed, and our own service access is never locked out.
- **Fail-safe.** Outputs power up off / 0 V. If the supervisor is silent for 15 minutes
  while outputs are commanded, they return to off / 0 V; with local control running,
  only outputs placed in hand are released. Boards that brown out are restored to the
  commanded state on the next pass.
- **Hardening.** Hardware stack limits, secrets obfuscated in the firmware, no symbol
  table, no source paths, and no error-message text in shipped images. Optional
  per-chip lock-down of the debug port and USB read-back for the highest-security
  installations.
- **Penetration tested** with AutomataNexus's own penetration-testing tool against the
  firmware, its tools, and its Console endpoint.

See [SECURITY.md](SECURITY.md) to report a vulnerability.

## Hardware

| Item | Specification |
|---|---|
| Processor | Raspberry Pi RP2350 (dual Arm Cortex-M33), 520 KB RAM, 4 MB flash |
| Wireless | 2.4 GHz Wi-Fi (802.11n) |
| Wired network | 10/100 Ethernet, DHCP; preferred over Wi-Fi when present |
| Serial | Two RS-485 ports, each independently Modbus RTU, BACnet MS/TP, or a legacy protocol |
| LonWorks | FT-10 on board |
| Analog inputs | Up to 16, each 0-10 V or 4-20 mA, 16-bit |
| Analog outputs | Up to 16, 0-10 V, 12-bit |
| Relay outputs | Up to 8, through DC solid-state relays to RIB coils |
| Digital inputs | Up to 8 dry contacts |
| Built-in sensors | Three 10K Type II thermistors, up to eight DS18B20 pipe / air sensors, differential pressure, PIR occupancy |
| Power | 5 V DC from a DIN-rail supply |
| Enclosure | Kestrel-Aero enclosure, boards on removable decks |

| Configuration | Hardware | Typical use |
|---|---|---|
| **Protocol gateway** | Controller, RS-485, Ethernet | Take over or integrate a front end or field trunk; no local point wiring |
| **Point controller** | Gateway plus the AI, AO, and relay boards the unit needs | Run any equipment type locally with any of the 69 sequences or custom logic |
| **Fully loaded** | 16 AI, 16 AO, 8 relays, 8 DI, two RS-485 ports, Ethernet, Wi-Fi | A small plant, or several units on one controller |

**Status light.** A repeating blink code shows heartbeat, Console data, traffic,
Ethernet, and Wi-Fi at a glance, even with the network down. A second light is lit
while the fault model scores.

## Licensing and purchase

NexusKestrel is proprietary hardware and firmware from AutomataNexus, LLC, installed
and commissioned by our controls technicians or authorized partners. See
[COMMERCIAL.md](COMMERCIAL.md).

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
NexusKestrel, NexusEdge, NexusShield, NexusFoundry, and Talos are trademarks of AutomataNexus, LLC.

</div>
