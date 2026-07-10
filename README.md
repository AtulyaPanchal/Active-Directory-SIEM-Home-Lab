# Enterprise Active Directory & SIEM Home Lab

A self-contained, isolated home lab that builds a real Active Directory forest, forwards
Windows Event Log telemetry into a Splunk SIEM, and simulates attacker activity from Kali
Linux to validate detection coverage — end to end, on a single host machine.

![Status](https://img.shields.io/badge/status-active-brightgreen)
![Platform](https://img.shields.io/badge/hypervisor-VMware%20Workstation%20Pro-blue)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

## Overview

This lab stands up four VMs on an isolated, host-only/NAT network and wires them together
into a small but realistic enterprise identity + monitoring environment:

| VM | Role | IP |
|---|---|---|
| `DC01` | Windows Server 2025 — AD DS + DNS | `192.168.50.10` |
| `WIN-CLIENT` | Windows 10/11 — domain-joined workstation | `192.168.50.20` |
| `KALI-ATTACK` | Kali Linux — recon / credential-attack simulation | `192.168.50.30` |
| `SPLUNK-SIEM` | Ubuntu Server — Splunk Enterprise indexer/search head | `192.168.50.40` |

```
┌────────────────────────────────────────────┐
│ Isolated Lab Network (Host-Only/NAT)        │
│ Subnet: 192.168.50.0/24                     │
└───────────────────┬────────────────────────┘
                     │
     ┌──────────┬────┴─────┬──────────────┐
     │          │          │              │
  ┌──▼──┐  ┌────▼───┐ ┌────▼─────┐  ┌─────▼──────┐
  │DC01 │  │WIN-CLIENT│ │KALI-ATTACK│ │SPLUNK-SIEM │
  │.10  │  │  .20    │ │   .30    │  │   .40      │
  └─────┘  └─────────┘ └──────────┘  └────────────┘

Log flow: DC01 & WIN-CLIENT ─▶ Splunk Universal Forwarder ─▶ SPLUNK-SIEM:9997
```

## What this lab demonstrates

- Provisioning an isolated multi-VM network in VMware Workstation Pro
- Standing up a real Windows Server 2025 AD forest (OUs, users, group policy, account lockout policy)
- Domain-joining a Windows client and authenticating as a domain user
- Forwarding Security/System/Application Windows Event Logs into Splunk via Universal Forwarder
- Simulating recon and failed-authentication activity from Kali Linux
- Building detections for failed logons (4625), account lockouts (4740), and Kerberos ticket
  requests (4768/4769) — the same event IDs used to detect password spraying and Kerberoasting
  in production environments

## Prerequisites

| Component | Requirement |
|---|---|
| RAM | 16 GB minimum (32 GB recommended) |
| Disk | 250 GB free, SSD recommended |
| CPU | 6+ cores, VT-x/AMD-V enabled |
| Hypervisor | VMware Workstation Pro 17+ (free for personal use) |

Software (all free): VMware Workstation Pro, Windows Server 2025 evaluation ISO, Windows
10/11 ISO, Kali Linux prebuilt VMware image, Splunk Enterprise (free Developer license) +
Universal Forwarder.

## Repo structure

```
.
├── README.md
├── docs/
│   └── AD_SIEM_HomeLab_Guide.pdf     # full step-by-step build guide
├── configs/
│   ├── inputs.conf                   # Splunk forwarder log inputs
│   └── sysmonconfig.xml              # (optional) Sysmon baseline config
└── scripts/
    ├── new-lab-user.ps1              # AD OU/user creation helper
    └── kali-static-ip.sh             # Kali static IP setup
```

*(Adjust this section to match whatever you actually commit — see note below.)*

## Quick start

The full walkthrough — network setup, DC promotion, domain join, Splunk install, and attack
simulation — is in [`docs/AD_SIEM_HomeLab_Guide.pdf`](docs/AD_SIEM_HomeLab_Guide.pdf). At a
high level:

1. Build the isolated `192.168.50.0/24` lab network in VMware's Virtual Network Editor.
2. Stand up `DC01`, promote it to a domain controller for `corp.local`, and configure an
   account lockout policy.
3. Domain-join `WIN-CLIENT` and confirm authentication.
4. Install Splunk Enterprise on `SPLUNK-SIEM`, enable receiving on port 9997, and deploy the
   Universal Forwarder to `DC01` and `WIN-CLIENT`.
5. From `KALI-ATTACK`, generate failed-logon and Kerberos telemetry and confirm detections
   surface in Splunk.

## Detections built

| Detection | Event ID(s) | Source |
|---|---|---|
| Failed logon | 4625 | Windows Security log |
| Account lockout | 4740 | Windows Security log |
| Kerberos TGT / service ticket request | 4768 / 4769 | Windows Security log |

## Scope & responsible use

This lab is built and operated entirely inside an isolated, host-only/NAT network that is
never bridged to a home or production network. All "attacks" are run by the lab owner
against infrastructure they own, for detection-engineering practice only. If you clone this
repo, do the same: keep it isolated, and only point these techniques at systems you own or
are explicitly authorized to test.

## Further exploration

- **BloodHound** — map AD attack paths and trust relationships
- **Atomic Red Team** — run mapped MITRE ATT&CK test cases and validate in Splunk
- **Additional domain-joined hosts** — widen the detection surface
- **Wazuh / Elastic** — stand up an alternate SIEM for comparison
- **AD CS** — add a certificate authority role and explore cert-based attack/detection
- **Tiered admin model** — rebuild with tiered privileged access and measure detection impact

## License

MIT — see [`LICENSE`](LICENSE).
