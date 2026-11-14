# Safety Notice and Intended Use

**NexusKestrel** — AutomataNexus, LLC

## Intended use

NexusKestrel is a building-automation controller and supervisor for heating,
ventilating, air-conditioning, and related equipment in commercial and institutional
buildings. It is intended to be installed, configured, and commissioned by qualified
controls technicians.

## Not a life-safety device

NexusKestrel is **not** a safety device and must not be the only protection against a
hazard. In particular:

- **Freeze stats, high-limit and low-limit cut-outs, flame safeguards, fire and smoke
  dampers and shutdowns, refrigerant and gas detection, and other code-required
  safeties must be hardwired and must act independently of the controller.**
- The controller may monitor and alarm on these devices, but it must never be the
  means by which they act.
- Plant equipment that carries its own safeties and sequencing (boilers, chillers) is
  enabled and disabled by the controller, not operated around its own controls.

## Electrical

- Wire to the controller's rated supply and I/O ratings only, and follow the wiring
  documentation for each board.
- Disconnect power before wiring. Follow local electrical codes.
- Relay outputs switch pilot-duty control circuits, not motor or line loads directly.

## Behavior on failure

- Outputs go to their configured fail-safe state on loss of control, on a firmware
  fault, and during updates.
- A firmware update that does not come up healthy is rolled back automatically.
- Loss of the network or the Console does not stop local control.

## Remote commands

Commands sent from the Console act on real equipment. Each command is confirmed by
read-back; an unconfirmed command is reported as such. Command rights should be given
only to trained people, and site command policy should be set before the controller
is placed in service.

## Report a safety concern

safety@automatanexus.com
