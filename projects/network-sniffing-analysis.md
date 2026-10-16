# Network Sniffing Analysis & Incident Report

**Course/Lab:** BlackStride Cyber Day 2 Lab – Network Sniffing  
**Target Files:** Packet Captures (`.pcap`)  
**Status:** Complete

> **Lab data disclaimer:** All values and artifacts below come from simulated or authorized educational lab data. Credential-like and secret-looking values are intentionally redacted placeholders for safe public portfolio use.

## Tools Used

- Packet capture analysis workflows (Wireshark-style process)
- HTTP/FTP protocol stream inspection
- File/object reconstruction from captured traffic
- Incident reporting and remediation planning

## What I Did

1. Reviewed unencrypted HTTP and FTP traffic in lab packet captures.
2. Extracted authentication patterns, session behavior, and file-transfer artifacts.
3. Reconstructed transferred file content and analyzed hidden metadata markers.
4. Documented findings, impact, and practical defensive recommendations.

## Key Findings

### Web Authentication (HTTP)

- **Target Server:** `webserver.local` / `portal.local`
- **Endpoint `/login`:** `username=[LAB_USER]` | `[REDACTED_LAB_PASSWORD]`
- **Endpoint `/auth`:** `id=[LAB_USER]` | `sess=[SIMULATED_SESSION_TOKEN]`
- **Endpoint `/session`:** `uid=[LAB_USER]` | `token=[SIMULATED_SESSION_TOKEN]`

### File Transfer Protocol (FTP)

- **Target Server:** BlackStride FTP Service / Local FTP
- **Successful Login (`exam_D_incident_ctf.pcap`):** `USER [LAB_USER]` | `PASS [REDACTED_LAB_PASSWORD]` — Status: `230 Login successful`
- **Successful Login (`cyberlab_incident.pcap`):** `USER [LAB_USER]` | `PASS [REDACTED_LAB_PASSWORD]` — Status: `230 User logged in`

#### Failed Authentication Attempts

- `USER [LAB_USER]` | `PASS [REDACTED_LAB_PASSWORD]` — `530 Login incorrect`
- `USER [LAB_USER]` | `PASS [REDACTED_LAB_PASSWORD]` — `530 Login incorrect`
- `USER [LAB_USER]` | `PASS [REDACTED_LAB_PASSWORD]` — `530 Login incorrect`

### Extracted Files & Secret-Looking Content

#### Reconstructed CSV File (`report.csv`)

```csv
id,name,amount
1,alpha,100
2,beta,200
3,gamma,[SIMULATED_SESSION_TOKEN]
```

#### Reconstructed Text File (`secret_exfil.txt`)

```text
[REDACTED_SIMULATED_SECRET]
```

### Web Source Metadata & Hidden HTML Comments

- Dashboard page contained a hidden session marker: `<!-- sid:[SIMULATED_SESSION_TOKEN] -->`
- Weekly backup report contained an encoded value marker: `<!-- FLAG:[SIMULATED_SESSION_TOKEN] -->`
- HTTPS metadata included a lab exam token placeholder: `[SIMULATED_SESSION_TOKEN]`

## Outcome

This investigation showed how plaintext and weakly handled session data can expose sensitive operational details during transit. I translated packet-level evidence into an actionable incident report suitable for security stakeholders.

## Security Recommendations

1. **Mandate Encrypted Web Communications:** Force all web endpoints to use HTTPS with modern TLS configurations.
2. **Deprecate Plaintext FTP:** Migrate file transfer operations to SFTP or FTPS.
3. **Purge Client-Visible Sensitive Metadata:** Strip sensitive comments/tokens from HTML/JS responses before deployment.
4. **Protect Session Tokens:** Rotate tokens after login, invalidate on logout, and enforce secure cookie attributes.
5. **Monitor Authentication Activity:** Alert on repeated failed logins, unusual file transfers, and sensitive-resource access.

## Skills Demonstrated

- Packet capture analysis
- HTTP and FTP traffic inspection
- Credential exposure detection
- Session token risk analysis
- File reconstruction from network streams
- Security-impact assessment and reporting
- Remediation planning
