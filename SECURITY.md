# Security Policy

## Scope

This policy covers NexusKestrel controller firmware, its update and configuration
paths, its operator link, and the NexusEdge Console as it serves NexusKestrel
controllers. These controllers operate HVAC and other building equipment, so we treat
security reports as urgent.

## Supported versions

Only the current firmware release line receives security fixes. Controllers update
over the air, so fixes reach installed units without a site visit.

| Release | Supported |
| ------- | --------- |
| Current | :white_check_mark: |
| Previous | :warning: critical fixes only |
| Older | :x: |

## Reporting a vulnerability

**Do not open a public GitHub issue for a security vulnerability.**

Report it privately:

1. **GitHub private vulnerability reporting.** Use "Report a vulnerability" under
   this repository's Security tab (preferred).
2. **E-mail** **security@automatanexus.com**. Our PGP key is available on request.

Please include:

- the firmware version (shown in the Console and in the integration sheet);
- what you found and its impact;
- steps to reproduce, or a proof of concept;
- any suggested fix.

## What to expect

- **Acknowledgment** within 48 hours.
- **Initial assessment** within 7 days, with a severity rating and plan.
- **Status updates** at least every 14 days until resolved.
- **Credit** in the release notes or advisory, unless you prefer to stay anonymous.

## Disclosure

We follow coordinated disclosure and ask for a reasonable window, typically 90 days,
before public disclosure. Because fixes reach controllers over the air, we can often
move faster than that.

## Rules of engagement

Test only controllers you own or are authorized to test. Never test against equipment
in service at a facility without the owner's written permission: a controller drives
real fans, pumps, valves, and burners. Do not access data that is not yours, and do
not degrade service for others.

## Deployment guidance

- Keep controllers on a building-automation network segment, not on a guest or public
  network.
- Leave automatic updates enabled.
- Keep life-safety and equipment safeties hardwired and independent of any controller
  (see [SAFETY.md](SAFETY.md)).
- Grant Console command rights only to the people who need them.
