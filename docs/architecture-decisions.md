# Architecture Decisions

This document records the key engineering decisions behind Raavix and the
reasoning for each. The intent is to show *why* each choice was made, not just
*what* was chosen because infrastructure decisions are trade-offs, and the reasoning
matters more than the outcome.

---

## 1. Local-first build with resource rotation, opportunistic cloud polling

**Decision:** Build and run the platform on local virtualised infrastructure
(VirtualBox), rotating RAM between services as needed, while a lightweight
automation process polls Oracle Cloud for free-tier ARM (Ampere A1) capacity in
the background.

**Why:** Free-tier cloud ARM capacity is persistently contended in the chosen
home region and cannot be guaranteed on any given day. Rather than treating cloud
availability as a prerequisite — and stalling the entire build waiting for it —
the platform is designed to run locally within real resource constraints, with
cloud capacity treated as an opportunistic enhancement rather than a dependency.

A dedicated, no-passphrase automation API key (separate from the interactive
credential, and independently revocable) performs the capacity polling unattended,
with graceful handling of out-of-capacity and transient API errors.

**Result:** The build is never blocked on infrastructure scarcity. When free ARM
capacity becomes available it can be adopted; until then, local resource rotation
keeps delivery moving. This mirrors a real-world principle: infrastructure
availability is a constraint to design around, not a wall to wait at.

---

## 2. Credential separation: interactive vs automation

**Decision:** Use a passphrase-protected key for interactive human use, and a
separate, dedicated, no-passphrase key scoped solely to unattended automation.

**Why:** A passphrase prompt cannot be satisfied by an unattended process, but a
no-passphrase key reused for everything weakens the human credential. Separating
the two by purpose means the interactive key stays passphrase-protected, the
automation key is isolated and independently revocable, and a compromise of one
does not compromise the other. This is the same separation-of-identity principle
behind service accounts and CI/CD deploy keys in production environments.

---

## 3. Open-source stack as a design driver

**Decision:** Build the entire platform on free and open-source tooling
(Wazuh, TheHive, Cortex, MISP, OPNsense, Suricata, and others).

**Why:** Cost is a genuine design constraint, and working within it demonstrates
the ability to deliver enterprise-grade security operations without
enterprise-grade licensing. Every tool chosen is production-credible in its own
right; the constraint shapes the architecture rather than limiting its
seriousness.

A specific example: **OPNsense was chosen over pfSense** for the firewall/VPN
layer. It is fully free with no paywalled features, built on HardenedBSD, ships
weekly security patches, has native WireGuard support, and exposes a full REST
API for automation. The reasoning was documented at decision time rather than
defaulting to the more familiar option.

---

*These decisions are revisited as the platform evolves. The constraints
(resource limits, cloud scarcity, cost) are treated as part of the engineering
problem, not obstacles to it.*
