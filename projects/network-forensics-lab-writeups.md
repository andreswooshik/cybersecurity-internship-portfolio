# Network Forensics & Capture The Flag (CTF) Lab Writeups

**Course/Lab:** BlackStride Cyber Day 2 Lab – Network Sniffing  
**Project Type:** Network forensics, packet analysis, and incident response  
**Status:** Complete  

> **Lab disclaimer:** This document describes authorized educational exercises. The credentials, tokens, flags, hostnames, and file contents are simulated lab artifacts and should not be reused outside the lab.

## Contents

1. [Cleartext HTTP Credential Sniffing](#1-walkthrough-cleartext-http-credential-sniffing)
2. [HTTP Object Export and Payload Extraction](#2-walkthrough-http-object-export--payload-extraction)
3. [TLS Handshake and Certificate Analysis](#3-walkthrough-tls-handshake--certificate-analysis)
4. [FTP Brute Force and Data Exfiltration](#4-incident-walkthrough-ftp-brute-force--data-exfiltration)
5. [Authentication and Session Token Recovery](#5-ctf-walkthrough-authentication--session-token-recovery)
6. [Hidden Metadata and Source Code Analysis](#6-ctf-walkthrough-hidden-metadata--source-code-analysis)
7. [Encrypted Session Metadata Leakage](#7-ctf-walkthrough-encrypted-session-metadata-leakage)
8. [Multi-Protocol Forensics and Base64 Decoding](#8-incident-walkthrough-multi-protocol-forensics--base64-decoding)
9. [Summary of Artifacts and Flags](#summary-table-of-artifacts--flags)

---

## 1. Walkthrough: Cleartext HTTP Credential Sniffing

**Target File:** `cyberlab_1-cleartext-password.pcap`  
**Protocol Focus:** HTTP / TCP  

### Objective

Intercept cleartext web authentication requests to identify plaintext credentials transmitted over unencrypted HTTP.

### Methodology and Findings

1. **HTTP traffic filtering:** Filtered for HTTP `POST` requests directed to `webserver.local`.
2. **Credential extraction:** Identified an unencrypted login request sent to `/login`:

```http
POST /login HTTP/1.1
Host: webserver.local
User-Agent: CyberLabAgent/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

username=student1&password=P@ssw0rd123
```

3. **Session response:** The server returned `HTTP/1.1 200 OK` and the banner `hello from 10.0.0.13`.
4. **Follow-up request:** A subsequent `GET` request to `webserver.local/index.html` used the User-Agent `Mozilla/5.0`.

**Security impact:** Credentials sent over HTTP can be captured by anyone able to monitor the network path.

---

## 2. Walkthrough: HTTP Object Export & Payload Extraction

**Target File:** `cyberlab_2-HTML-export.pcap`  
**Protocol Focus:** HTTP / HTML payload extraction  

### Objective

Reconstruct and extract file objects transmitted across unencrypted HTTP streams.

### Methodology and Findings

1. Monitored HTTP `GET` requests sent from `CyberLabAgent/2.0` to `webserver.local`.
2. Detected an attachment download request for `/blackstride_day2.html`.
3. Confirmed the response header `Content-Disposition: attachment; filename="blackstride_day2.html"`.
4. Reconstructed the HTML payload from packet bytes:

```html
<html>
<head><title>BlackStride Cyber Day 2 Lab - Network Sniffing</title></head>
<body>
<h1>BlackStride Cyber Day 2 Lab - Network Sniffing</h1>
<p>This HTML file was served in the lab capture so students can extract it using Wireshark's Export → HTTP Objects.</p>
</body>
</html>
```

**Security impact:** Unencrypted HTTP allows transferred objects to be inspected, copied, and modified in transit.

---

## 3. Walkthrough: TLS Handshake & Certificate Analysis

**Target File:** `cyberlab_3_https.pcap`  
**Protocol Focus:** TLS / SSL negotiation  

### Objective

Analyze TLS handshake traffic and extract the server certificate exchanged during negotiation.

### Methodology and Findings

1. Inspected packet frames initiating the TLS session.
2. Carved the synthetic PEM-formatted X.509 certificate block from the packet payload:

```text
-----BEGIN CERTIFICATE-----
MIIFakeCert
-----END CERTIFICATE-----
```

**Security note:** TLS encrypts application data, but handshake metadata and certificates can still be observable. Certificate analysis helps validate the server identity and TLS configuration.

---

## 4. Incident Walkthrough: FTP Brute Force & Data Exfiltration

**Target File:** `cyberlab_incident.pcap`  
**Protocol Focus:** FTP / data carving  

### Objective

Investigate unauthorized access attempts against an internal FTP server and recover transferred secret data.

### Methodology and Findings

1. **Service banner identification:** The service returned `220 Welcome to BlackStride FTP Service`.
2. **Authentication analysis:**

   - `USER admin` | `PASS wrongpass` → `530 Login incorrect.`
   - `USER admin` | `PASS tryme` → `530 Login incorrect.`
   - `USER backup` | `PASS backup123` → `230 User backup logged in.`

3. **Data-stream tracking:** After login, user `backup` enabled binary transfer mode with `TYPE I` and issued `STOR secret_exfil.txt`.
4. **Data reconstruction:** The data-channel stream contained:

```text
SECRET_DATA: The project's passwords are 12345
```

**Security impact:** FTP exposes both authentication credentials and file contents in plaintext.

---

## 5. CTF Walkthrough: Authentication & Session Token Recovery

**Target File:** `exam_A_cleartext_ctf.pcap`  
**Protocol Focus:** HTTP POST / session tokens  

### Objective

Locate session tokens embedded within cleartext web authentication POST requests.

### Methodology and Findings

1. Filtered for HTTP `POST` requests sent to `portal.local`.
2. Recovered the following session parameters:

   - `/session`: `uid=student1&token=TEMP1&remember=1`
   - `/auth`: `id=student1&sess=BSC-LPD8W&mode=1`

3. **Flag identified:** `BSC-LPD8W`

---

## 6. CTF Walkthrough: Hidden Metadata & Source Code Analysis

**Target File:** `exam_B_http_objects_ctf.pcap`  
**Protocol Focus:** HTTP body / source comments  

### Objective

Carve HTTP response payloads from `site.local` to uncover hidden developer comments and session metadata.

### Methodology and Findings

1. Rebuilt multiple HTTP endpoints, including a PNG object, an HTML news page, and a text-based system health page.
2. Inspected the dashboard HTML response:

```html
<html><head><!-- sid:BSC-5A0JQ --></head><body>Dashboard</body></html>
```

3. **Flag identified:** `BSC-5A0JQ`

**Security impact:** HTML comments are client-visible and should never contain secrets, session identifiers, or internal security metadata.

---

## 7. CTF Walkthrough: Encrypted Session Metadata Leakage

**Target File:** `exam_C_https_meta_ctf.pcap`  
**Protocol Focus:** TLS SNI / Client Hello metadata  

### Objective

Extract unencrypted hostname and metadata strings observable during SSL/TLS setup.

### Methodology and Findings

1. Examined metadata fields transmitted during encrypted-session negotiation.
2. Identified the string `BSC-Z0F15 secure-exam net`.
3. **Flag identified:** `BSC-Z0F15`

**Security note:** Encryption protects application content but does not necessarily conceal all connection metadata.

---

## 8. Incident Walkthrough: Multi-Protocol Forensics (FTP & Base64 Decoding)

**Target File:** `exam_D_incident_ctf.pcap`  
**Protocol Focus:** FTP / HTML comments / Base64  

### Objective

Analyze FTP uploads and HTTP backup reports to recover hidden flags across multiple protocol streams.

### Part 1: FTP Traffic Analysis

1. **Credential audit:**

   - `USER audit` | `PASS Backup2025` → `530 Login incorrect`
   - `USER backup` | `PASS Password1` → `530 Login incorrect`
   - `USER student` | `PASS Spring!2026` → `230 Login successful`

2. **File-stream carving:** Recovered the payload from `STOR report.csv`:

```csv
id,name,amount
1,alpha,100
2,beta,200
3,gamma,BSC-HQZ08
```

3. **CSV flag identified:** `BSC-HQZ08`

### Part 2: HTTP HTML Comment Analysis

1. Inspected the HTML response for the Weekly Backup Report:

```html
<!doctype html>
<html>
<head><title>Incident Report</title></head>
<body>
<h1>Weekly Backup Report</h1>
<p>Status: OK</p>
<!-- FLAG:QlNDLTlLUTJB                          -->
</body>
</html>
```

2. Decoded the Base64 value:

   - Encoded payload: `QlNDLTlLUTJB`
   - Decoded value: `BSC-9KQ2A`

3. **HTML flag identified:** `BSC-9KQ2A`

---

## Summary Table of Artifacts & Flags

| PCAP File | Protocol | Key Evidence / Credential | Recovered Flag / Token |
| :--- | :--- | :--- | :--- |
| `cyberlab_1-cleartext-password.pcap` | HTTP | `student1` : `P@ssw0rd123` | N/A |
| `cyberlab_2-HTML-export.pcap` | HTTP | `blackstride_day2.html` export | N/A |
| `cyberlab_3_https.pcap` | TLS | PEM certificate (`MIIFakeCert`) | N/A |
| `cyberlab_incident.pcap` | FTP | `backup` : `backup123` | `SECRET_DATA` payload |
| `exam_A_cleartext_ctf.pcap` | HTTP | POST `/auth` parameter | `BSC-LPD8W` |
| `exam_B_http_objects_ctf.pcap` | HTTP | HTML `<head>` SID comment | `BSC-5A0JQ` |
| `exam_C_https_meta_ctf.pcap` | TLS metadata | Domain metadata leak | `BSC-Z0F15` |
| `exam_D_incident_ctf.pcap` | FTP / HTTP | `student` : `Spring!2026` and Base64 comment | `BSC-HQZ08` and `BSC-9KQ2A` |

## Lessons Learned

- Plaintext protocols expose credentials and transferred data to network observers.
- Session tokens must be protected as carefully as passwords.
- HTML comments and client-visible metadata are not secure storage locations.
- TLS improves confidentiality, but connection metadata may remain observable.
- Packet captures can preserve valuable evidence for incident response and digital forensics.
- Security findings should be documented with evidence, impact, and practical remediation steps.

## Recommended Remediation

1. Enforce HTTPS for every authenticated web endpoint.
2. Replace FTP with SFTP, FTPS, or a secure managed file-transfer service.
3. Rotate exposed credentials and invalidate captured sessions.
4. Remove secrets, tokens, and internal metadata from HTML and JavaScript.
5. Monitor failed logins, unusual transfers, and large outbound data flows.
6. Use least-privilege access controls and strong authentication for file-transfer services.
