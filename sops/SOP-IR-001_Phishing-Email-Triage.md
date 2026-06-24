# SOP-IR-001 — Phishing Email Triage & Initial Response

> Example template — fictional scenario. Contains no confidential, client-specific, or proprietary information; all identifiers and figures are illustrative.

| Field | Value |
|-------|-------|
| Document ID | SOP-IR-001 |
| Title | Phishing Email Triage & Initial Response |
| Version | 1.0 |
| Audience | SOC Analyst (L1/L2), Incident Responder |
| Companion to | BRIEF-IR-001 — Phishing Campaigns: Business Impact & Management Response |
| Classification | TLP:CLEAR |
| Author | Tobias Kastenhuber (LiRiX2) |
| Effective date | 2026-06-15 |
| Frameworks | NIST SP 800-61 Rev. 2, MITRE ATT&CK, ISO/IEC 27001:2022 |

---

## 1. Purpose

This SOP defines the operational procedure for triaging and responding to a reported suspicious email. It takes the analyst from "an email landed in the abuse mailbox" to a defensible verdict and the correct escalation path. The business-impact and management layer is covered in the companion BRIEF-IR-001.

The goal of triage is simple to state and hard to do under pressure: **determine the attacker's objective as early as possible**, because the cost of response rises the longer that objective goes unidentified.

---

## 2. Scope

Applies to all emails reported as suspicious through the abuse mailbox, the report-phishing button, or direct user report — regardless of whether the user interacted with the email. Covers triage, verdict, containment initiation, and exposure assessment. Hands off to SOP-IR-002 if a payload is confirmed or suspected to have executed.

---

## 3. Trigger Conditions

This SOP is initiated when any of the following occurs:

- A user reports a suspicious email via the report button or abuse mailbox
- The email security gateway flags a message for analyst review
- A SOAR alert correlates a reported email with other suspicious activity
- Threat intel indicates an active campaign matching inbound mail

---

## 4. Response Phases

### Phase 1 — Intake & Evidence Preservation (0–5 min)

**1.1** Acknowledge the report. If a user submitted it, confirm receipt.

**1.2** Preserve the original email:
- Obtain the email as a `.eml` or `.msg` file (full headers intact)
- Do **not** forward the suspicious email inline — this strips headers and can re-trigger links. Always attach the original.
- Store the sample in the case directory with the incident ID

**1.3** Open an incident ticket and record: reporter, reported time, mailbox, and whether the user states they interacted (clicked / opened attachment / entered credentials).

> In practice this first question — "did you click?" — drives everything downstream. Ask it early, but never rely on the answer alone; users under-report interaction.

---

### Phase 2 — Technical Analysis with EmailAnalyzer (5–20 min)

**2.1** Run the preserved `.eml` through **[EmailAnalyzer](https://github.com/LiRiX2/EmailAnalyzer)** (`email_analyzer.py`).

**2.2** Review the **header analysis**:
- Compare the **Visual Sender** (display name / From) against the **Truth Sender** (Return-Path, authenticated origin, originating IP)
- Check authentication results: **SPF / DKIM / DMARC**
- A spoofing flag = visual sender does not match authenticated origin

**2.3** Review the **URL analysis**:
- Inspect each extracted URL (defanged) for lookalike domains, credential-themed paths, redirects
- Check WHOIS: recently registered domains (days old) are a strong signal
- Note the heuristic verdict per URL

**2.4** Review **attachment analysis** (if present):
- Check file type vs. claimed type
- For Office documents: macro analysis (oletools) — auto-exec macros, suspicious API calls, obfuscation
- Record file hashes; submit to threat intel

**2.5** Read the **consolidated Risk Score**. Treat it as a decision aid, not a verdict — the analyst owns the final call. Corroborating signals (auth failure + lookalike domain + credential path) reinforce each other; a single weak signal does not.

---

### Phase 3 — Verdict & Classification (per analysis)

**3.1** Assign a verdict:

| Verdict | Meaning |
|---------|---------|
| **Benign** | Legitimate mail, false report — close and inform reporter |
| **Spam / Low-risk** | Unwanted but not malicious — filter, no incident |
| **Phishing — credential harvesting** | Link/page designed to steal credentials |
| **Phishing — malware delivery** | Malicious attachment or payload link |
| **BEC / fraud** | Social-engineering for payment / data, often no payload |

**3.2** Tag with MITRE ATT&CK:
- T1566.001 — Spearphishing Attachment
- T1566.002 — Spearphishing Link
- T1566.003 — Spearphishing via Service

**3.3** For any malicious verdict, proceed to Phase 4.

---

### Phase 4 — Containment & Exposure Assessment (20–40 min)

> This is the phase that determines incident severity. A single reported email is not the incident — **the campaign is the incident**.

**4.1** Block the indicators:
- Submit malicious URLs, domains, sender addresses, and file hashes to the email gateway, proxy, and blocklist
- Confirm blocks are applied

**4.2** Assess campaign exposure — the question management actually cares about:
- Mailbox-wide search: how many recipients received this or a variant?
- How many **delivered** vs. **quarantined**?
- How many **interacted** (clicked / opened / submitted credentials)?

**4.3** Calculate the **exposure rate** = recipients who interacted ÷ recipients targeted.

**4.4** Remediate delivered copies:
- Purge / claw back undelivered and delivered copies of the malicious mail from mailboxes where possible

**4.5** Decide the escalation tier based on exposure:

| Finding | Action |
|---------|--------|
| No interaction, contained | Block, purge, document, close |
| Interaction but no credential/payload | Monitor affected users, document |
| **Credentials submitted** | Escalate to credential-compromise response → IR Lead |
| **Payload executed** | Escalate to **SOP-IR-002 (Malware Containment)** |
| BEC / payment fraud risk | Escalate to IR Lead + notify Finance |

---

### Phase 5 — Escalation & Handoff

**5.1** For confirmed compromise (credentials or payload), escalate to the IR Lead and trigger BRIEF-IR-001 for the management response.

**5.2** For credential compromise:
- Force password reset and session revocation for affected users
- Flag accounts with access to sensitive/regulated systems (these drive regulatory assessment)

**5.3** For payload execution: hand off to SOP-IR-002.

**5.4** If regulated personal data may be exposed: notify IR Lead so the breach-assessment / GDPR Article 33 clock can be evaluated (see BRIEF-IR-001 §3.2). The exposure assessment is escalated **immediately**, not at case closure.

---

### Phase 6 — Closure & Lessons Learned

**6.1** Close the ticket with full timeline: intake → verdict → containment → exposure → escalation.

**6.2** Inform the reporter of the outcome — this reinforces reporting behaviour.

**6.3** Feed findings back:
- New indicators → detection rules / blocklists
- Repeat-victim users → targeted awareness training
- Gaps in the triage flow → update this SOP and the SOAR playbook

---

## 5. Process Flow

The full triage decision flow is documented as a diagram:  
**[diagrams/phishing-triage-flow.md](../diagrams/phishing-triage-flow.md)**

---

## 6. Operational Metrics (KPIs)

| Metric | Definition |
|--------|-----------|
| Time to triage | Report received → verdict assigned |
| Time to containment | Verdict → indicators blocked |
| Exposure rate | Recipients who interacted ÷ recipients targeted |
| Breach-assessment turnaround | Detection → DPO/Legal reportability decision |
| Repeat-victim rate | Users repeatedly interacting with phishing |

(These map to the leadership-facing metrics in BRIEF-IR-001 §5.)

---

## 7. References

- NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide
- MITRE ATT&CK — T1566 Phishing and sub-techniques
- ISO/IEC 27001:2022 — A.5.26 Response to information security incidents
- GDPR — Regulation (EU) 2016/679, Art. 33 & 34
- Companion: BRIEF-IR-001 — Phishing Campaigns: Business Impact & Management Response
- Tool: [EmailAnalyzer](https://github.com/LiRiX2/EmailAnalyzer)

---

## 8. Revision History

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-06-15 | Tobias Kastenhuber | Initial example version. |
