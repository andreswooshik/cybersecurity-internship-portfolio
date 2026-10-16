# Network Sniffing Analysis & Incident Report

**Course/Lab:** BlackStride Cyber Day 2 Lab – Network Sniffing  
**Target Files:** Packet Captures (`.pcap`)  
**Status:** Complete  

> **Portfolio note:** This report is based on an authorized educational lab environment. The credentials, tokens, flags, and file contents below are lab-generated values.

---

## 1. Executive Summary

This report documents the analysis of raw packet captures (`.pcap`) containing unencrypted network communications across HTTP and FTP protocols. By inspecting cleartext session traffic, credentials, session tokens, hidden HTML metadata, and exfiltrated file contents were successfully recovered. This exercise demonstrates the critical risk of transmitting sensitive authentication data over unencrypted channels.

---

## 2. Captured Credentials & Session Data

### Web Authentication (HTTP)

- **Target Server:** `webserver.local` / `portal.local`
- **Endpoint `/login`:** `username=student1` | `password=P@ssw0rd123`
- **Endpoint `/auth`:** `id=student1` | `sess=BSC-LPD8W`
- **Endpoint `/session`:** `uid=student1` | `token=TEMP1`

### File Transfer Protocol (FTP)

- **Target Server:** BlackStride FTP Service / Local FTP
- **Successful Login (`exam_D_incident_ctf.pcap`):** `USER student` | `PASS Spring!2026` — Status: `230 Login successful`
- **Successful Login (`cyberlab_incident.pcap`):** `USER backup` | `PASS backup123` — Status: `230 User backup logged in`

#### Failed Authentication Attempts

- `USER audit` | `PASS Backup2025` — `530 Login incorrect`
- `USER admin` | `PASS Admin123` / `PASS tryme` — `530 Login incorrect`
- `USER backup` | `PASS Password1` — `530 Login incorrect`

---

## 3. Extracted Files & Secret Content

### Reconstructed CSV File (`report.csv`)

Interception of the FTP `STOR report.csv` stream yielded the following record payload:

```csv
id,name,amount
1,alpha,100
2,beta,200
3,gamma,BSC-HQZ08
```

### Reconstructed Text File (`secret_exfil.txt`)

Interception of FTP exfiltration traffic revealed the plaintext secret payload:

```text
SECRET_DATA: The project's passwords are 12345
```

### Web Source Metadata & Hidden HTML Comments

- **Dashboard Page (`exam_B_http_objects_ctf.pcap`):** Header contained session ID comment: `<!-- sid:BSC-5A0JQ -->`
- **Weekly Backup Report (`exam_D_incident_ctf.pcap`):** Body contained encoded flag comment: `<!-- FLAG:QlNDLTlLUTJB                          -->`
- **HTTPS Metadata Capture (`exam_C_https_meta_ctf.pcap`):** Extracted secure exam token: `BSC-Z0F15`

---

## 4. Master Flag Inventory

| Flag / Token ID | Protocol / Source | Details / Encoding |
| :--- | :--- | :--- |
| `BSC-LPD8W` | HTTP (`/auth`) | Session parameter |
| `BSC-HQZ08` | FTP (`report.csv`) | Exfiltrated CSV record |
| `BSC-5A0JQ` | HTTP (HTML Header) | Client session ID (`sid`) |
| `BSC-Z0F15` | HTTPS Metadata | Exam session identifier |
| `QlNDLTlLUTJB` | HTTP (HTML Comment) | Base64 encoded flag (`BSC-9KQ2A` decoded) |

---

## 5. Security Recommendations

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
