# Portfolio Maintenance Guide

Use this guide when adding new portfolio content so the repository stays internship-ready and safe to share publicly.

## Adding a New Project

1. Add the writeup as a new Markdown file in `projects/`.
2. Include concise sections: **Tools Used**, **What I Did**, **Key Findings**, and **Outcome**.
3. Add or update a project link in `README.md` with a one-line outcome and tools summary.
4. Keep all links relative (example: `projects/new-project.md`).

## Adding a New Certificate

1. Place certificate files in `certificates/` only.
2. Add a row in `certificates/README.md` for every new file.
3. Use only verifiable filename details; mark unknown details as **Not verified from filename**.
4. Keep links relative (example: `certificates/certificate-file.pdf`).

## Safety & Privacy Checklist (Before Commit)

- Remove or replace any credential-like values using placeholders:
  - `[REDACTED_LAB_PASSWORD]`
  - `[SIMULATED_SESSION_TOKEN]`
  - `[REDACTED_SIMULATED_SECRET]`
- Do not include real personal contact details unless intentionally publishing them.
- Do not commit IDs, private keys, API tokens, or account secrets.
- Keep all project evidence explicitly labeled as simulated/authorized lab data.
