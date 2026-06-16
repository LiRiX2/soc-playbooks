# Phishing Triage & Escalation — Process Flow

Visual representation of the triage and escalation logic in [SOP-IR-001](../sops/SOP-IR-001_Phishing-Email-Triage.md). Decision points are diamonds; escalation steps are highlighted; green nodes are start/end states.

```mermaid
flowchart TD
    A([Suspicious email reported]) --> B[Preserve evidence<br/>copy .eml + SHA-256 hash]
    B --> C[Run EmailAnalyzer<br/>headers - URLs - attachments]
    C --> D{Verdict?}
    D -->|Benign or Spam| E([Close ticket])
    D -->|Phishing or Malicious| F[Assess exposure<br/>recipients and interactions]
    F --> G{User interaction?}
    G -->|No interaction| H[Contain and block<br/>sender - IP - URL - hash]
    G -->|Clicked / entered credentials| I[Credential-compromise response<br/>reset - revoke sessions]
    G -->|Attachment executed| J[Isolate endpoint<br/>full IR]
    H --> K{Regulated data<br/>in scope?}
    K -->|No| N[Notify reporter]
    K -->|Yes| M[[Escalate: IR Lead and CISO / DPO]]
    I --> L[[Escalate: IR Lead]]
    J --> L
    L --> O{Broader campaign<br/>indicators?}
    M --> O
    N --> P[Post-incident<br/>finalize IOCs - lessons learned]
    O -->|Yes| Q[[Declare incident - CISO]]
    O -->|No| P
    Q --> P
    P --> R([Close and feed back to Preparation])

    classDef terminal fill:#e8f0e8,stroke:#4a7a55,color:#143322;
    classDef escalate fill:#fdeaea,stroke:#c55555,color:#611111;
    class A,E,R terminal;
    class L,M,Q escalate;
```
