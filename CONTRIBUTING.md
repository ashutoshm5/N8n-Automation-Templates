# Contributing to the n8n Automation Templates Library

Thank you for your interest in contributing to this collection of production-grade n8n automation templates! 

This repository aims to be a trusted, modular library of self-hosted workflows across multiple fields of automation (Content & Social Media, AI Agents, CRM & Sales, DevOps, Data Pipelines, and Customer Support).

---

## 1. Quality Standards for New Templates

Every template added to this repository must meet the following criteria:

1. **Deterministic & Production-Ready**: Workflows must include proper error handling, retries where appropriate, and clean data transformations.
2. **Zero Embedded Secrets**: Workflows must **never** contain plain-text API keys, passwords, client secrets, or private tokens. All authentication must be handled via n8n's Credentials Manager.
3. **Self-Contained Modularity**: Each template must live in its own directory with its own documentation suite, workflow JSON, and asset folders.
4. **Beginner-Friendly Documentation**: A user without prior knowledge should be able to configure credentials, mount storage, and run a test execution by following the template's guides.

---

## 2. Standard Template Directory Structure

When creating a new template, create a dedicated folder under the root repository following this structure:

```
Template-Name/
│
├── README.md                          # Template landing page & summary
├── LICENSE                            # Template license notice
├── CHANGELOG.md                       # Versioned change history (starting at 1.0.0)
├── SECURITY.md                        # Security notes & sanitization rules
├── .gitignore                         # Template-level git exclusions
│
├── docs/
│   ├── SETUP-GUIDE.md                 # Step-by-step installation (Docker / Local)
│   ├── API-CONFIGURATION.md           # External service setup & permission scopes
│   ├── WORKFLOW-EXPLANATION.md        # Node-by-node audit with Mermaid flowcharts
│   ├── DIRECTORY-STRUCTURE.md         # Local storage & volume mapping guide (if applicable)
│   ├── TROUBLESHOOTING.md             # Common errors and step-by-step fixes
│   └── PRE-DEPLOYMENT-CHECKLIST.md    # Pre-launch sanity checklist
│
├── workflow/
│   └── Template-Name.json             # Sanitized production workflow JSON
│
└── assets/
    ├── README.md                      # Asset guidelines
    ├── workflow-overview.png          # High-resolution canvas screenshot
    └── setup-screenshots/             # Setup view captures (sensitive info masked)
```

---

## 3. Mandatory Pre-Commit Sanitization Checklist

Before committing any workflow JSON file:

- [ ] **Strip Instance ID**: Delete `"instanceId"` from the root `"meta"` block.
- [ ] **Remove Workflow ID**: Remove the root `"id"` property so n8n generates a fresh ID upon import.
- [ ] **Sanitize Account IDs**: Replace account numbers, cloud names, or channel IDs with uppercase placeholders (e.g., `YOUR_ACCOUNT_ID`, `YOUR_CLOUD_NAME`).
- [ ] **Replace Local Paths**: Replace hardcoded file paths (e.g., `C:\Users\...`) with expressions like `{{ $env.INPUT_PATH || '/path/to/files/*' }}`.
- [ ] **Set Safe Defaults**: Ensure any upload or publishing nodes default to `private` or `draft` mode.
- [ ] **Sanitize Prompts**: Replace specific personal or company brand names in AI prompts with `[Your Brand]`.
- [ ] **Check Node Names**: Ensure node names are descriptive and match between the JSON and documentation.

---

## 4. Pull Request Process

1. Fork the repository and create your branch:
   ```bash
   git checkout -b template/your-automation-name
   ```
2. Build and thoroughly test your workflow in a clean n8n instance.
3. Export the workflow JSON and perform the sanitization checklist above.
4. Write the complete documentation suite inside your template folder.
5. Update the template catalog in the root [README.md](README.md).
6. Commit your changes and submit a Pull Request with a clear description of the automation's business use case and architecture.
