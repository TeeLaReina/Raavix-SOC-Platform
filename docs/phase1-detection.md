# Phase 1 — SIEM Detection Engineering (Wazuh)

This document walks through the first detection scenario built on the Raavix
platform: deploying a Wazuh agent to a Windows endpoint, generating a realistic
multi-stage attack, and validating that the SIEM detected and correctly
classified each stage against the MITRE ATT&CK framework.

The goal of Phase 1 was not simply to install a SIEM. It was to prove the full
detection chain end to end — that known-bad activity on a monitored host
surfaces as correctly-classified, correctly-prioritised alerts an analyst can
act on.

---

## Environment

| Component | Detail |
|-----------|--------|
| SIEM | Wazuh (all-in-one: indexer, manager, dashboard) |
| Manager host | Ubuntu 22.04, internal SOC segment `10.10.10.10/24` |
| Monitored endpoint | Windows (debloated build), static IP `10.10.10.20` |
| Agent | `win-endpoint-01`, Windows MSI |
| Network | Isolated internal lab segment (`raavix-soc`) |

Both the manager and the endpoint sit on an isolated internal network, so all
telemetry is generated and contained within the lab.

---

## The detection chain (why this works)

A failed login only becomes an alert if four independent links all hold. This
mental model is the foundation for debugging *any* "why didn't my SIEM catch X"
problem:

1. **Windows generates the event** — e.g. a failed logon writes Event ID 4625 to
   the Security log, *but only if the audit policy is configured to log it*.
2. **The event lands in the Security channel** of the Windows Event Log.
3. **The Wazuh agent forwards that channel** to the manager.
4. **The manager's decoders and rules match it** and raise an alert with a rule
   ID and severity level.

Break any one link and there is no alert. Phase 1 validated all four.

---

## The attack scenario

A single failed login proves logging. A *sequence* that mirrors a real intrusion
proves detection engineering. The scenario was built as a four-stage narrative,
each stage mapped to a MITRE ATT&CK technique.

### Stage 1 — Brute force access attempt
Multiple failed authentication attempts in rapid succession, generating a burst
of **Event ID 4625** (failed logon).

- **MITRE:** T1110 — Brute Force
- **Detection:** individual failure events plus a higher-severity correlation
  alert when the frequency threshold was crossed. The dashboard recorded
  **38 authentication failures** and **4 alerts at level 12 or above** across the
  test window — the correlation alerts are the signal that the SIEM detected a
  *pattern*, not just isolated events.

### Stage 2 — The breach
A successful interactive logon immediately following the failure burst —
**Event ID 4624** — representing the attacker finally gaining access. The
dashboard recorded **8 authentication successes**, allowing the failure-then-
success pattern to be reconstructed on a timeline.

- **MITRE:** T1078 — Valid Accounts

### Stage 3 — Persistence
Creation of a backdoor local account and its addition to the Administrators
group:

- Account creation → **Event ID 4720**
- Added to privileged group → **Event ID 4732**, surfaced as Wazuh
  **rule 60160** ("Domain Users Group Changed"), level 5

- **MITRE:** T1136.001 — Create Account: Local; T1098 — Account Manipulation

### Stage 4 — Cleanup (also detected)
The backdoor account was removed — and the removal is itself detectable:

- Account deletion → **Event ID 4726**, surfaced as Wazuh **rule 60111**
  ("User account disabled or deleted"), level 8

The lesson: even an attacker covering their tracks leaves a trail the SIEM
records.

---

## Validation and results

All four stages were confirmed in the Wazuh dashboard:

- The **Threat Hunting** module showed the individual events with their rule IDs,
  severities, and the `win-endpoint-01` source.
- The **MITRE ATT&CK** module mapped the activity to the expected techniques —
  Brute Force, Valid Accounts, Account Manipulation, and related categories.
- Severity tiers behaved correctly: routine Windows housekeeping decoded at low
  levels (3–5), while the meaningful security events surfaced at higher levels
  (8 and 12+).

This severity separation is the core of real SOC work, distinguishing the
handful of alerts that matter from the constant background of benign system
telemetry.

---

## Troubleshooting: a real detection-chain failure

During testing, the account-management events (Stages 3 and 4) initially
appeared to be missing from the dashboard while the brute-force events came
through cleanly. The investigation followed the detection chain from the source
rather than assuming a SIEM misconfiguration:

1. **Checked the source first.** Querying the Windows Security log directly with
   `Get-WinEvent` confirmed that events 4720, 4732, and 4726 *were* present on the
   endpoint. This immediately ruled out an event-generation or audit-policy
   problem — the events existed, so the issue was downstream of Windows.
2. **Identified the real cause.** The Wazuh manager had degraded under memory
   pressure during the test (the indexer and manager compete for RAM on a
   constrained host). While the manager was unhealthy, the dashboard could not be
   queried — so the events looked "missing" when they were simply not yet visible.
3. **Confirmed recovery.** After restarting the Wazuh services in dependency
   order (indexer → manager → dashboard) and allowing time for each to come up,
   the previously-missing events appeared. The agent had buffered the events
   locally and re-delivered them once the manager recovered.

The takeaway: the events were never lost. Diagnosing the source first — instead
of immediately editing Wazuh rules — isolated the problem to the manager's health,
and Wazuh's own agent-buffering resilience preserved the data through the outage.

---

## Skills demonstrated

- SIEM deployment and agent enrollment (Wazuh manager + Windows endpoint)
- Detection validation against MITRE ATT&CK (T1110, T1078, T1136.001, T1098)
- Reading and interpreting alerts by rule ID and severity
- Distinguishing security signal from benign system noise
- Systematic troubleshooting of the detection chain from source to dashboard
- Understanding of resource constraints and service-dependency ordering

---

*Phase 1 status: complete. Next: Phase 2 — case management and threat
intelligence (TheHive, Cortex, MISP).*
