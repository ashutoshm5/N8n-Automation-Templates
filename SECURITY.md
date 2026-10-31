# Repository Security Policy & Template Safety Standards

This repository serves as a curated library of **n8n automation templates**. Because workflow files frequently interact with private APIs, cloud infrastructure, and databases, maintaining rigorous security practices is critical.

---

## 1. Security Architecture of Shared Templates

All templates published in this repository adhere to the **Zero-Embedded-Secrets Standard**:

1. **Decoupled Credentials**: 
   - Workflow JSON files do **not** contain API keys, access tokens, client secrets, or basic authentication passwords.
   - All authentication is delegated to n8n's internal **Credentials Manager**, which encrypts sensitive tokens at rest using AES-256.
   - When importing any template from this repository, you must connect your own credentials on your local instance.
2. **Sanitized Identifiers**:
   - Host-specific instance identifiers (`instanceId`) are scrubbed from exported JSON metadata.
   - Workflow database primary keys (`id`) are omitted or set to null to avoid database collision errors on import.
   - Account-specific IDs (e.g., Instagram Business Account IDs, Cloudinary Cloud Names, private webhook URLs) are replaced with visible placeholders (e.g., `YOUR_INSTAGRAM_ACCOUNT_ID`).
3. **Safe Defaults**:
   - Publishing operations (such as YouTube or social video uploads) are configured to **Private** or **Draft** mode by default to prevent accidental public disclosure during initial testing.

---

## 2. Best Practices for Users Importing Templates

When importing workflows from this repository into your n8n environment:

* **Use Environment Variables for System Paths**: Never hardcode direct local machine paths (e.g., `C:\Users\...`). Mount storage volumes cleanly inside Docker containers and reference them via expressions like `{{ $env.INPUT_FILES_PATH }}`.
* **Keep Secrets in the Credentials Manager**: Never hardcode API keys or authorization tokens inside JavaScript Code nodes, URL parameters, or plain HTTP header fields.
* **Review Code Nodes Prior to Execution**: Always inspect JavaScript code nodes within imported workflows to understand data transformations before running production workloads.
* **Verify Service Permissions**: Apply the principle of least privilege when generating API tokens for third-party platforms (Google Cloud, Meta, AWS, Cloudinary). Grant only the exact scopes needed for the workflow.

---

## 3. Contributing & Sharing Workflows Safely

If you contribute new templates to this repository, you must complete the pre-commit sanitization audit detailed in [CONTRIBUTING.md](CONTRIBUTING.md):
- Confirm that `.env` files are ignored and never committed.
- Verify that exported JSON files contain zero plain-text tokens.
- Mask all sensitive details in screenshots, logs, and sample responses.

---

## 4. Reporting Security Vulnerabilities

If you discover a credential leak, security flaw, or vulnerability in any template within this repository, please do not open a public issue.

Contact the repository maintainer directly via private channels so the issue can be remediated promptly.
