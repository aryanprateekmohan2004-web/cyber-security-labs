# Personal Investigations

This directory is a collection of security incident investigations I've worked through independently, formatted in a consistent SOC-analyst report structure: **Verdict → In-Depth Analysis → Key Findings (Artifacts) → Deployment Containment Protocol**.

Each case starts from a simulated alert (SIEM/EDR/email security telemetry, threat intelligence enrichment, and threat-hunting log correlation) and is worked the way I'd triage a real incident — including calling out when telemetry is *unrelated noise* versus when it's a genuine correlated indicator that changes the verdict, and when a closed alert still needs a downstream escalation note rather than a flat close.


---

## Structure

Cases are organized by severity into their own directories. Each investigation is a standalone `.md` file, numbered sequentially within its severity folder.

```
Personal_investigations/
├── critical/
├── advanced/
├── medium/
├── low/
└── README.md   ← you are here
```

---

## Methodology

Every case follows the same four-part format:

1. **Verdict** — True Positive / False Positive / Benign, with justification.
2. **In-Depth Analysis and Conclusion** — full breakdown of the alert, enrichment (AbuseIPDB, URLScan, etc.), threat-hunting correlation, and MITRE ATT&CK mapping.
3. **Key Findings (Artifacts)** — structured table of every IOC/artifact reviewed, including ones explicitly ruled out as unrelated.
4. **Deployment Containment Protocol** — selected response actions (Isolate Host, Block IP/Domain, Block File Hash, Reset Credentials, Collect Forensics, Escalate to Tier 3, Close/No Action), each with a stated reason.

---

## Case Index

Each severity folder maintains its own case index. Click through for the full list of investigations at that level:

| Severity | Cases | Index |
|---|---|---|
| 🔴 Critical | 1 | [critical/README.md](critical/README.md) |
| 🟠 Advanced | 0 | [advanced/README.md](advanced/README.md) |
| 🟡 Medium | 1 | [medium/README.md](medium/README.md) |
| 🟢 Low | 0 | [low/README.md](low/README.md) |

---

## About Me

I'm a final-year B.E. Computer Science (IoT & Cybersecurity, incl. Blockchain) student building toward a SOC Analyst role. My broader portfolio includes a SOC automation pipeline (Wazuh, Splunk, Sysmon, n8n SOAR, TheHive), an Active Directory hardening lab mapped to MITRE ATT&CK, and a picoCTF Top 8% global ranking.
