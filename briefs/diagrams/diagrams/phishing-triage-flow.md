# Phishing Triage — Process Flow

> Companion diagram to [SOP-IR-001](../sops/SOP-IR-001_Phishing-Email-Triage.md).  
> Illustrative example. Contains no confidential or client-specific information.

This diagram shows the decision flow from a reported email to verdict, containment, and escalation.

```mermaid
flowchart TD
    A[Suspicious email reported<br/>abuse mailbox / report button] --> B[Phase 1: Preserve .eml<br/>open ticket, ask: did user interact?]
    B --> C[Phase 2: Run EmailAnalyzer<br/>headers · URLs · attachments]

    C --> D{Spoofing?<br/>SPF/DKIM/DMARC fail<br/>+ Visual ≠ Truth Sender}
    C --> E{Malicious URL?<br/>lookalike / recent domain<br/>credential path}
    C --> F{Malicious attachment?<br/>auto-exec macro<br/>suspicious API calls}

    D --> G[Consolidated Risk Score]
    E --> G
    F --> G

    G --> H{Verdict}
    H -->|Benign / Spam| I[Filter · inform reporter · close]
    H -->|Phishing - credentials| J[Phase 4: Block IOCs<br/>+ exposure assessment]
    H -->|Phishing - malware| J
    H -->|BEC / fraud| K[Escalate to IR Lead<br/>notify Finance]

    J --> L{Exposure assessment<br/>how many interacted?}
    L -->|No interaction| M[Block · purge · document · close]
    L -->|Clicked, no creds/payload| N[Monitor affected users · document]
    L -->|Credentials submitted| O[Escalate: credential-compromise<br/>reset passwords · revoke sessions]
    L -->|Payload executed| P[Escalate to SOP-IR-002<br/>Malware Containment]

    O --> Q{Regulated data<br/>at risk?}
    P --> Q
    K --> Q
    Q -->|Yes| R[Notify IR Lead / DPO<br/>assess GDPR Art. 33 clock<br/>see BRIEF-IR-001]
    Q -->|No| S[Document · lessons learned]
    R --> S

    style A fill:#1e2327,color:#fff
    style P fill:#7a1f1f,color:#fff
    style O fill:#7a1f1f,color:#fff
    style R fill:#8a6d1f,color:#fff
    style I fill:#1f5f3f,color:#fff
    style M fill:#1f5f3f,color:#fff
```

## Reading the flow

- **Top half (Phases 1–3)** is pure analysis: preserve the evidence, run EmailAnalyzer, reach a verdict.
- **Bottom half (Phase 4 onward)** is where triage becomes incident response. The single most important branch is the **exposure assessment** — one reported email is not the incident; the campaign is.
- **Red nodes** are escalation points where the incident leaves L1/L2 hands.
- **Amber node** is the regulatory decision point that starts the GDPR clock — escalated immediately, never deferred to case closure.
