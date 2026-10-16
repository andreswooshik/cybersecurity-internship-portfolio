# Network Sniffing Analysis & Incident Report

**Course/Lab:** BlackStride Cyber Day 2 Lab – Network Sniffing  
**Target Files:** Packet Captures (`.pcap`)  
**Status:** Complete  

> **Portfolio note:** This report is based on an authorized educational lab environment. The credentials, tokens, flags, and file contents below are lab-generated values.
>
> **Public-safety note:** Credential-like and token-like values are partially redacted or explicitly marked as lab artifacts to avoid confusion with real secrets.

---

## 1. Executive Summary

This report documents the analysis of raw packet captures (`.pcap`) containing unencrypted network communications across HTTP and FTP protocols. By inspecting cleartext session traffic, credentials, session tokens, hidden HTML metadata, and exfiltrated file contents were successfully recovered. This exercise demonstrates the critical risk of transmitting sensitive authentication data over unencrypted channels.

## 2. At-a-Glance

### What I did
- Investigated multiple lab packet captures to identify exposed authentication data and hidden application metadata.
- Reconstructed transferred files from FTP and reviewed HTTP response content for embedded artifacts.
- Consolidated evidence into a single incident-style report with practical security actions.

### Tools
- Wireshark-style packet inspection workflows
- Protocol filtering and stream reconstruction (HTTP, FTP, TLS metadata)
- Base64 decoding for encoded lab artifacts

### Findings
- Plaintext HTTP/FTP traffic exposed usernames, password fields, session values, and transferred content.
- Client-visible metadata (such as HTML comments) revealed lab session/flag markers.
- Metadata remains partially observable even when sessions use encrypted protocols.

### Recommendations
- Enforce HTTPS and remove plaintext FTP.
- Rotate credentials/session identifiers and improve authentication monitoring.
- Remove sensitive comments/metadata from client-delivered responses.

---

## 3. Captured Credentials & Session Data

### Web Authentication (HTTP)

- **Target Server:** `webserver.local` / `portal.local`
- **Endpoint `/login`:** `username=student1` | `password=P@ssw0rd123`
- **Endpoint `/auth`:** `id=student1` | `sess=[LAB_SESSION_ID_REDACTED]`
- **Endpoint `/session`:** `uid=student1` | `token=[LAB_TOKEN_REDACTED]`

### File Transfer Protocol (FTP)

- **Target Server:** BlackStride FTP Service / Local FTP
- **Successful Login (`exam_D_incident_ctf.pcap`):** `USER student` | `PASS [LAB_PASSWORD_REDACTED]` — Status: `230 Login successful`
- **Successful Login (`cyberlab_incident.pcap`):** `USER backup` | `PASS [LAB_PASSWORD_REDACTED]` — Status: `230 User backup logged in`

#### Failed Authentication Attempts

- `USER audit` | `PASS [LAB_PASSWORD_REDACTED]` — `530 Login incorrect`
- `USER admin` | `PASS [LAB_PASSWORD_REDACTED]` / `PASS [LAB_PASSWORD_REDACTED]` — `530 Login incorrect`
- `USER backup` | `PASS [LAB_PASSWORD_REDACTED]` — `530 Login incorrect`

---

## 4. Extracted Files & Secret Content

### Reconstructed CSV File (`report.csv`)

Interception of the FTP `STOR report.csv` stream yielded the following record payload:

```csv
id,name,amount
1,alpha,100
2,beta,200
3,gamma,[LAB_FLAG_VALUE]
```

### Reconstructed Text File (`secret_exfil.txt`)

Interception of FTP exfiltration traffic revealed the plaintext secret payload:

```text
SECRET_DATA: [LAB_CONTENT_REDACTED]
```

### Web Source Metadata & Hidden HTML Comments

- **Dashboard Page (`exam_B_http_objects_ctf.pcap`):** Header contained lab session marker comment: `<!-- sid:[LAB_FLAG_VALUE] -->`
- **Weekly Backup Report (`exam_D_incident_ctf.pcap`):** Body contained encoded lab marker comment: `<!-- FLAG:[LAB_BASE64_VALUE] -->`
- **HTTPS Metadata Capture (`exam_C_https_meta_ctf.pcap`):** Extracted lab token marker: `[LAB_FLAG_VALUE]`

---

## 5. Master Flag Inventory

| Flag / Token ID | Protocol / Source | Details / Encoding |
| :--- | :--- | :--- |
| `[LAB_SESSION_MARKER]` | HTTP (`/auth`) | Session parameter |
| `[LAB_FLAG_VALUE]` | FTP (`report.csv`) | Exfiltrated CSV record |
| `[LAB_FLAG_VALUE]` | HTTP (HTML Header) | Client session marker (`sid`) |
| `[LAB_FLAG_VALUE]` | HTTPS Metadata | Exam session identifier |
| `[LAB_BASE64_VALUE]` | HTTP (HTML Comment) | Base64-encoded lab flag value |

---

## 6. Security Recommendations

1. **Mandate Encrypted Web Communications:** Force all web endpoints to use HTTPS, preferably with modern TLS configurations, to prevent eavesdropping on user credentials and session tokens.
2. **Deprecate Plaintext FTP:** Upgrade file transfer operations from legacy FTP to SFTP or FTPS to secure authentication and file transfer streams.
3. **Purge Comments Prior to Production:** Implement build pipelines to automatically strip sensitive metadata, tokens, and developer comments from HTML/JS response bodies.
4. **Protect Session Tokens:** Rotate session tokens after authentication, invalidate them on logout, and use secure cookie attributes such as `Secure`, `HttpOnly`, and `SameSite`.
5. **Monitor Authentication Activity:** Alert on repeated failed logins, unusual file transfers, and access to sensitive resources.

---

## Skills Demonstrated

- Packet capture analysis
- HTTP and FTP traffic inspection
- Plaintext credential identification
- Session token analysis
- FTP file reconstruction
- Base64 decoding
- Evidence documentation
- Security impact assessment
- Incident reporting
- Remediation planning
