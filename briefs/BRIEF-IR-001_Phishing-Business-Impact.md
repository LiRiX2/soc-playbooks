# Phishing Campaigns — Business Impact & Management Response Brief

> *Example template — fictional scenario. Contains no confidential, client-specific, or proprietary information; all identifiers and figures are illustrative.*

| Field | Value |
|---|---|
| **Document ID** | BRIEF-IR-001 |
| **Title** | Phishing Campaigns — Business Impact & Management Response |
| **Version** | 1.0 |
| **Audience** | IT/Security leadership, IR Lead, business stakeholders |
| **Companion to** | SOP-IR-001 — Phishing Email Triage & Initial Response |
| **Classification** | TLP:CLEAR |
| **Author** | Tobias Kastenhuber (LiRiX2) |
| **Effective date** | 2026-06-15 |

---

## 1. Executive Summary

Phishing is not a mail-filter problem; it is a business-risk problem that happens to arrive by email. A single successful campaign can translate within hours into financial loss, operational outage, a reportable data breach, and lasting reputational damage. This brief sets out **what a phishing campaign can mean for the organization**, **which decisions belong to management**, and shows — through one worked example — how a technical detection on the SOC bench becomes a business decision at the leadership table.

The operational handling of individual emails is defined in the companion **SOP-IR-001**. This document covers the layer above it: impact and command.

---

## 2. What a Phishing Campaign Can Mean for the Business

A phishing email is the *delivery mechanism*; the business impact depends on the attacker's objective. The table frames the realistic outcomes leadership should plan for.

| Impact area | Mechanism | Realistic outcome | Primarily affects |
|---|---|---|---|
| **Financial — direct** | Business Email Compromise / fraudulent payment instruction | Wire transfer to attacker; invoice fraud | Finance, treasury |
| **Financial — indirect** | Ransomware delivered via attachment/link | Recovery cost, downtime, ransom decision | Whole organization |
| **Confidentiality / data** | Credential theft → account takeover → lateral movement | Unauthorized access to regulated or sensitive data | Affected business unit, customers |
| **Operational** | Endpoint compromise / malware foothold | Service disruption, system isolation, lost productivity | IT operations, service delivery |
| **Regulatory & legal** | Breach of personal or regulated data | Mandatory notification, potential fines, audit findings | Legal, compliance, DPO |
| **Reputational** | Public disclosure, customer/partner impact | Loss of trust, contract risk, press exposure | Executive, communications |

**Key management insight:** the same inbound email can land anywhere on this table. Triage exists to determine *which* outcome is in play as early as possible, because the cost of response rises sharply the longer the attacker's objective goes unidentified.

---

## 3. Management Response & Decision Points

Most of the procedure is operational and owned by the SOC. A defined set of decisions, however, can only be made by management. The point of pre-agreeing them is that they are made calmly *now*, not improvised under pressure during an incident.

### 3.1 Decisions management owns

| Decision | Trigger | Owner |
|---|---|---|
| **Declare a formal incident** | Confirmed phishing with exposure, or signs of a broader campaign | IR Lead / CISO |
| **Authorize disruptive containment** | Containment action would affect business operations (e.g., disabling accounts, isolating systems) | IR Lead + affected business owner |
| **Regulatory notification** | Breach likely affects personal or regulated data | CISO / DPO / Legal |
| **Customer / partner communication** | External parties are or may be affected | Executive + Communications |
| **Law enforcement / insurance / external IR** | Financial loss, ransomware, or scope beyond internal capacity | Executive + Legal |
| **Crisis communications (internal & external)** | Incident becomes visible or material | Communications + Executive |

### 3.2 The regulatory clock

If a campaign results in unauthorized access to personal data, notification timelines apply and start running early. Under the GDPR, a notifiable personal-data breach must be reported to the supervisory authority **without undue delay and where feasible within 72 hours** of becoming aware of it (Art. 33); affected individuals must be informed where the risk to them is high (Art. 34). Sector-specific obligations may apply in addition (e.g., financial supervision, or critical-infrastructure reporting where the organization qualifies). 

The operational consequence: **the breach-assessment decision cannot wait for full forensic certainty.** Management must decide, on the available facts, whether the 72-hour clock has started. This is why the SOC's exposure assessment (§4 below) is escalated immediately, not at case closure.

### 3.3 Management actions across the incident lifecycle

- **On detection:** Confirm decision ownership; ensure the IR Lead has authority to act; stand by for an exposure figure.
- **During containment:** Authorize disruptive actions; convene legal/DPO if regulated data is in scope; prepare holding statements.
- **Post-incident:** Decide on notifications and external communication; commission lessons-learned; fund any control gaps the incident revealed.

---

## 4. Worked Example — Credential-Harvesting Phishing

*This walks one fictional incident through both layers: the technical analysis on the SOC bench and the business decision at the management table. It uses the **EmailAnalyzer** tool and the escalation logic from SOP-IR-001.*

### 4.1 Scenario (fictional)

An employee in the finance department reports an email titled **"Action required: your Microsoft 365 password expires today"**, asking them to "re-validate" their account via a link. They report it because the wording felt urgent. They state they did *not* click.

### 4.2 Technical analysis (SOC bench)

The reported `.eml` is preserved and run through `email_analyzer.py`. Illustrative findings:

```
HEADER ANALYSIS
  Visual Sender : Microsoft 365 <no-reply@microsoft.com>
  Truth Sender  : Return-Path bounce@m365-secure-login[.]com
                  Originating IP 203.0.113.45 (geo: outside expected region)
  Authentication: SPF=fail  DKIM=none  DMARC=fail
  >> SPOOFING DETECTED: visual sender does not match authenticated origin

URL ANALYSIS
  hxxps://m365-secure-login[.]com/verify?user=...
  WHOIS: domain registered 3 days ago, registrant anonymized
  Heuristic verdict: HIGH RISK (lookalike domain, recent registration,
                     credential-themed path, auth mismatch)

CONSOLIDATED RISK SCORE: HIGH
```

**Analyst interpretation:** This is a credential-harvesting phishing email. The "Visual vs. Truth Sender" split, the failed authentication, and a lookalike domain registered three days ago are consistent and mutually reinforcing. Tagged **MITRE ATT&CK T1566.002 (Spearphishing Link)**.

**Verdict:** Phishing — credential harvesting.

### 4.3 From verdict to escalation (per SOP-IR-001)

The reporter did not interact, so the immediate user is low-risk. But the SOC must answer the question management actually cares about: **how many *other* people received this, and did any of them click or enter credentials?** A mailbox-wide search shows the campaign reached **40 recipients**, of whom **3 submitted credentials** before the domain was blocked.

That fact flips the escalation tier: from "contain and block" to **credential-compromise response**, escalated to the IR Lead — because the affected users are in **finance**, with access to payment systems and personal data.

### 4.4 The same incident, at the management table

Translated into business terms, the SOC's finding reads:

> *Three finance accounts with access to payment systems and personal data may be under attacker control. Account takeover could enable fraudulent payments (financial impact) and access to regulated personal data (regulatory impact).*

Management decisions triggered:

1. **Declare incident** and authorize immediate password resets and session revocation for the 3 users (disruptive containment, pre-authorized for confirmed compromise).
2. **Convene DPO/Legal** to assess whether personal data was accessed — i.e., whether the **GDPR 72-hour clock** has started. The assessment cannot wait for full forensics.
3. **Brief Finance leadership** to watch for fraudulent payment instructions over the coming days (BEC frequently follows successful credential theft).
4. **Hold external communication** pending the breach assessment; prepare a holding statement in case escalation is needed.

### 4.5 What this example demonstrates

A nine-line tool output became a board-relevant decision in one chain: *spoofed sender → high-risk verdict → exposure of 3 finance accounts → potential payment fraud and a regulatory notification clock.* The technical rigor and the management response are the same incident viewed from two ends — and the value of the SOC function is making that translation fast and accurate.

---

## 5. Metrics That Matter to Management

These reframe the SOC's operational KPIs (SOP-IR-001 §8) for a leadership audience:

- **Exposure rate** — recipients who interacted ÷ recipients targeted. The single best indicator of campaign impact.
- **Time to containment** — how quickly the window of attacker opportunity was closed.
- **Breach-assessment turnaround** — detection → DPO/Legal decision on reportability. Directly governs regulatory exposure.
- **Repeat-victim rate** — informs targeted user-awareness investment.

---

## 6. References

- GDPR — Regulation (EU) 2016/679, Art. 33 & 34 (personal data breach notification)
- NIST SP 800-61 — Computer Security Incident Handling Guide
- MITRE ATT&CK — T1566 Phishing and sub-techniques
- Companion: SOP-IR-001 — Phishing Email Triage & Initial Response

---

## 7. Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-06-15 | Tobias Kastenhuber | Initial example version. |
