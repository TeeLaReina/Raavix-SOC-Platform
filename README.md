# Raavix SOC Platform

> A multi-tenant, cloud-native, AI-assisted Security Operations Platform built entirely on open source tooling.

**Built by:** Yetunde Duze — Security Infrastructure Engineer
**Status:** 🟡 In Active Development
**Started:** June 2026

---

## What This Is

Raavix is a fully functional Security Operations Centre (SOC) platform that can be
deployed for any organisation needing enterprise-grade security monitoring without
enterprise-grade licensing costs.

It combines industry-standard open source tools into a single, integrated
environment covering detection, alerting, case management, threat intelligence,
automated response, and compliance reporting — with Claude AI providing
intelligent alert triage and analysis.

---

## Who It Is Built For

This environment is designed to be onboarded by:

- Media companies handling high-profile client data
- Law firms and accounting firms with sensitive documents
- Fintech and cloud computing companies
- Foundations and NGOs
- Any SMB needing ISO 27001 or SOC2-aligned security operations

---

## Core Stack

| Layer               | Tool                       | Purpose                               |
| ------------------- | -------------------------- | ------------------------------------- |
| SIEM / XDR          | Wazuh                      | Log collection, correlation, alerting |
| Case Management     | TheHive 5                  | Incident tracking and investigation   |
| Enrichment          | Cortex                     | Automated IOC analysis                |
| Threat Intelligence | MISP                       | Threat feeds and indicator sharing    |
| SOAR                | n8n                        | Automated response workflows          |
| AI Triage           | Claude API (Anthropic)     | Intelligent alert analysis            |
| IAM / SSO / MFA     | Keycloak                   | Identity, single sign-on, MFA         |
| Firewall / VPN      | OPNsense + WireGuard       | Network security and access           |
| Network IDS         | Suricata                   | Packet-level threat detection         |
| Internal PKI        | Step-CA                    | Certificate management                |
| Dashboards          | Grafana                    | SOC and executive reporting           |
| CSPM                | Prowler                    | Cloud security posture                |
| Vuln Management     | OpenVAS + Wazuh            | Vulnerability scanning                |
| Edge Security       | Cloudflare                 | DDoS, WAF, SSL, DNS                   |
| Cloud Infra         | Oracle Cloud (Always Free) | Hosting                               |

---

## Architecture

```
Internet → Cloudflare (WAF/DDoS/SSL)
  → Nginx Proxy Manager
  → WireGuard VPN (authenticated access)
  → Internal Services (Wazuh, TheHive, Keycloak, Grafana)
  → AI Layer (Claude API via n8n)
```

Key design decisions and their reasoning are documented in
[`docs/architecture-decisions.md`](docs/architecture-decisions.md).

---

## Compliance Alignment

- ISO/IEC 27001:2022
- SOC 2 Type II criteria
- NDPR (Nigeria Data Protection Regulation)

Compliance mapping documents: `docs/compliance/`

---

## Project Structure

```
Raavix-SOC-Platform/
├── docs/
│   ├── phase1-detection.md       
│   ├── architecture-decisions.md 
│   ├── architecture/             
│   ├── compliance/               
│   ├── playbooks/                
│   └── onboarding/               
├── evidence/
│   └── phase1-wazuh/            
├── configs/
│   ├── wazuh/                   
│   ├── opnsense/                 
│   ├── keycloak/                 
│   └── nginx/                  
└── automation/
    ├── n8n-workflows/        
    └── claude-prompts/         
```

---

## Build Progress

- [x] Phase 0 — Infrastructure setup and repository
- [x] Phase 1 — Wazuh SIEM deployment & detection validation → [write-up](docs/phase1-detection.md)
- [ ] Phase 2 — TheHive + Cortex + MISP integration
- [ ] Phase 3 — Keycloak SSO + MFA + WireGuard VPN
- [ ] Phase 4 — Claude AI alert triage layer
- [ ] Phase 5 — Grafana dashboards
- [ ] Phase 6 — Vulnerability management
- [ ] Phase 7 — SOAR playbook automation
- [ ] Phase 8 — Compliance documentation
- [ ] Phase 9 — Client onboarding guide

---

## Phase 1 Highlights

Phase 1 deployed Wazuh and validated end-to-end detection of a four-stage attack
on a monitored Windows endpoint, mapped to MITRE ATT&CK:

- **T1110** Brute Force — failed-logon burst + frequency correlation
- **T1078** Valid Accounts — successful breach following the burst
- **T1136.001 / T1098** Local account creation + privilege escalation (persistence)
- Cleanup detection — backdoor account removal also flagged

Full detail, including a documented troubleshooting investigation, in
[`docs/phase1-detection.md`](docs/phase1-detection.md).

---

## Author

**Yetunde Duze**
Security Infrastructure Engineer
[LinkedIn](https://www.linkedin.com/in/yetunde-duze/) | [GitHub](https://github.com/TeeLaReina)
