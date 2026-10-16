# Portfolio Maintenance Guide

## Add a New Project
1. Create a new Markdown file in `projects/` with a clear kebab-case filename.
2. Use this section structure when relevant: **What I did / Tools / Findings / Recommendations**.
3. Add the project link to the Featured Projects section in `README.md`.

## Add a New Certificate or Badge
1. Save the file in `certificates/` using a professional kebab-case filename.
2. Update `certificates/README.md` with file link, type, issuer, title, and verification status.
3. If issuer/title cannot be confirmed from the document content, label it as **Not verified** instead of guessing.

## Keep Sensitive Data Out
- Do not upload real credentials, personal access tokens, API keys, private keys, or production system details.
- For lab artifacts that resemble credentials, keep clear disclaimers and redact values when full values are not necessary.
- Remove hidden metadata if it contains personal or sensitive information before publishing files.
