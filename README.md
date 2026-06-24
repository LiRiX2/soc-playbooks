# SOC Playbooks

A curated collection of security operations documentation — incident response SOPs, management briefs, and process diagrams. Each document is written to reflect real SOC practice: operational where it needs to be actionable, and strategic where it needs to inform leadership decisions.

> All documents in this repository are **illustrative examples built around fictional scenarios**. They contain no confidential, client-specific, or proprietary information.

## Why this exists

Tooling shows you can build; documentation shows you can operate. This repository demonstrates how a detection translates into a defensible, repeatable response — from the analyst bench up to the management decision that the incident forces.

## Contents

### Phishing

| ID | Document | Audience | Description |
|---|---|---|---|
| SOP-IR-001 | [Phishing Email Triage & Initial Response](sops/SOP-IR-001_Phishing-Email-Triage.md) | SOC / IR | Operational runbook for triaging and responding to reported phishing emails, including a process flow diagram. |
| BRIEF-IR-001 | [Phishing — Business Impact & Management Response](briefs/BRIEF-IR-001_Phishing-Business-Impact.md) | Leadership / IR | What a phishing campaign means for the business, which decisions management owns, and a worked example bridging the technical and management layers. |

The phishing documents are built around **[EmailAnalyzer](https://github.com/LiRiX2/EmailAnalyzer)**, a Python tool for email header, URL, and attachment analysis — used as the core detection step in the triage SOP.

### Malware Containment

| ID | Document | Audience | Description |
|---|---|---|---|
| SOP-IR-002 | [Malware Containment — Phishing-Delivered Payload](sops/SOP-IR-002_Malware-Containment.md) | SOC / IR | Operational runbook for containing and eradicating malware delivered via phishing. Covers detection, host isolation, lateral movement check, evidence preservation, and recovery. |
| BRIEF-IR-002 | [Malware Infection — Business Impact & Management Response](briefs/BRIEF-IR-002_Malware-Business-Impact.md) | Leadership / IR | What a malware infection means for the business, which decisions management owns, and how to handle regulatory obligations including GDPR Article 33. |

SOP-IR-002 continues directly from SOP-IR-001: a phishing email has been triaged, a malicious attachment was opened, and a payload is confirmed or suspected to have executed on an endpoint.

## Structure

```
soc-playbooks/
├── sops/        Operational runbooks (step-by-step procedures)
├── briefs/      Management-facing impact & response documents
└── diagrams/    Process and decision-flow diagrams
```

## Frameworks referenced

These documents map to the standards a SOC is typically audited against: NIST SP 800-61 (incident handling), NIST CSF, ISO/IEC 27001:2022, MITRE ATT&CK, and GDPR Article 33.

## Author

Tobias Kastenhuber ([LiRiX2](https://github.com/LiRiX2))
