# Network Forensics & Capture The Flag (CTF) Lab Writeups

**Course/Lab:** BlackStride Cyber Day 2 Lab – Network Sniffing  
**Project Type:** Network forensics, packet analysis, and incident response  
**Status:** Complete

> **Responsible-use disclaimer:** This writeup uses simulated or authorized educational lab artifacts only. Credential-like fields, session tokens, and secret-looking payloads are redacted placeholders for safe public sharing.

## Tools Used

- Packet capture and protocol analysis workflows
- HTTP object export and payload reconstruction
- FTP command/data stream tracing
- TLS metadata and certificate inspection
- Base64 decoding for encoded lab artifacts

## What I Did

- Investigated eight protocol-focused lab scenarios.
- Traced authentication and transfer activity across HTTP, FTP, and TLS-related flows.
- Reconstructed artifacts from packet evidence and documented findings clearly.
- Mapped evidence to impact and defensive recommendations.

## Key Findings

1. **Cleartext HTTP credential exposure** can reveal submitted login fields in transit.
2. **HTTP object transfers** can be reconstructed when encryption is absent.
3. **TLS sessions** protect payload content but may still expose metadata.
4. **FTP authentication and file transfers** expose sensitive material when unencrypted.
5. **Client-visible comments/metadata** can leak session identifiers or encoded values.

## Scenario Walkthrough Summaries

### 1) Cleartext HTTP Credential Sniffing

- **Target File:** `cyberlab_1-cleartext-password.pcap`
- **Evidence pattern observed:** `username=[LAB_USER] credential=[REDACTED_LAB_PASSWORD]`
- **Impact:** Anyone with network visibility can intercept plaintext login submissions.

### 2) HTTP Object Export and Payload Extraction

- **Target File:** `cyberlab_2-HTML-export.pcap`
- **Evidence pattern observed:** downloadable HTML attachment transmitted over HTTP.
- **Impact:** Unencrypted object transfers can be inspected, copied, and altered in transit.

### 3) TLS Handshake and Certificate Analysis

- **Target File:** `cyberlab_3_https.pcap`
- **Evidence pattern observed:** synthetic certificate exchange during TLS negotiation.
- **Impact:** Certificate and handshake metadata remain useful for identity/configuration validation.

### 4) FTP Brute Force and Data Exfiltration

- **Target File:** `cyberlab_incident.pcap`
- **Evidence pattern observed:** repeated failed logins followed by successful FTP access and upload activity.
- **Redacted payload sample:** `[REDACTED_SIMULATED_SECRET]`
- **Impact:** Plaintext FTP can expose both authentication attempts and transferred data.

### 5) Authentication and Session Token Recovery

- **Target File:** `exam_A_cleartext_ctf.pcap`
- **Evidence pattern observed:** `/session` and `/auth` POST parameters containing token-like values.
- **Safe representation:** `token=[SIMULATED_SESSION_TOKEN]`, `sess=[SIMULATED_SESSION_TOKEN]`
- **Impact:** Session identifiers can be captured and abused if transmitted insecurely.

### 6) Hidden Metadata and Source Code Analysis

- **Target File:** `exam_B_http_objects_ctf.pcap`
- **Evidence pattern observed:** HTML comment contained session marker-style metadata.
- **Safe representation:** `<!-- sid:[SIMULATED_SESSION_TOKEN] -->`
- **Impact:** Client-visible comments should never carry sensitive context.

### 7) Encrypted Session Metadata Leakage

- **Target File:** `exam_C_https_meta_ctf.pcap`
- **Evidence pattern observed:** metadata string included token-like value during negotiation.
- **Safe representation:** `[SIMULATED_SESSION_TOKEN] secure-exam net`
- **Impact:** Encrypted channels can still leak useful contextual metadata.

### 8) Multi-Protocol Forensics and Base64 Decoding

- **Target File:** `exam_D_incident_ctf.pcap`
- **Evidence pattern observed:** FTP evidence plus encoded HTML marker required decoding workflow.
- **Safe representation:** encoded value `[SIMULATED_SESSION_TOKEN]` decoded to `[SIMULATED_SESSION_TOKEN]`
- **Impact:** Cross-protocol correlation improves incident reconstruction quality.

## Outcome

I produced a structured set of forensics-focused writeups that preserve investigation quality while redacting sensitive-looking artifacts for responsible public portfolio use.

## Summary Table of Artifacts

| PCAP File | Protocol | Evidence Type (Sanitized) | Recruiter-Relevant Takeaway |
| :--- | :--- | :--- | :--- |
| `cyberlab_1-cleartext-password.pcap` | HTTP | Login field capture (`[REDACTED_LAB_PASSWORD]`) | Demonstrates plaintext credential risk |
| `cyberlab_2-HTML-export.pcap` | HTTP | Object export/reconstruction | Demonstrates file-level traffic analysis |
| `cyberlab_3_https.pcap` | TLS | Handshake/certificate metadata review | Demonstrates encrypted-session analysis |
| `cyberlab_incident.pcap` | FTP | Auth attempts + exfil stream (`[REDACTED_SIMULATED_SECRET]`) | Demonstrates incident tracing |
| `exam_A_cleartext_ctf.pcap` | HTTP | Session parameter capture (`[SIMULATED_SESSION_TOKEN]`) | Demonstrates token-risk awareness |
| `exam_B_http_objects_ctf.pcap` | HTTP | Hidden comment analysis | Demonstrates source-level artifact review |
| `exam_C_https_meta_ctf.pcap` | TLS metadata | Token-like metadata exposure | Demonstrates metadata leakage analysis |
| `exam_D_incident_ctf.pcap` | FTP / HTTP | Multi-protocol + encoded marker workflow | Demonstrates end-to-end forensics workflow |

## Lessons Learned

- Plaintext protocols expose credentials and transferred data to observers.
- Session-style tokens require the same protection level as passwords.
- Client-visible metadata can leak operationally sensitive context.
- Forensics reporting should connect evidence, impact, and mitigation clearly.

## Recommended Remediation

1. Enforce HTTPS on authenticated and sensitive web endpoints.
2. Replace FTP with SFTP/FTPS or secure managed transfer mechanisms.
3. Rotate exposed credentials and invalidate potentially captured sessions.
4. Remove sensitive comments/tokens from client-visible source content.
5. Monitor failed logins, unusual transfer behavior, and large outbound flows.
