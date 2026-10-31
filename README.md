# n8n Automation Templates Library

[![n8n Platform](https://img.shields.io/badge/Platform-n8n%20Workflow%20Automation-EA4B71?logo=n8n)](https://n8n.io/)
[![Deployment](https://img.shields.io/badge/Deployment-Docker%20%7C%20Self--Hosted-2496ED?logo=docker)](https://www.docker.com/)
[![Security Standard](https://img.shields.io/badge/Security-Zero%20Embedded%20Secrets-brightgreen)](SECURITY.md)
[![License Notice](https://img.shields.io/badge/License-Review%20Required-yellow)](LICENSE)
[![Templates Status](https://img.shields.io/badge/Templates-Production%20Ready-blueviolet)](#-template-catalog)

A curated collection of production-grade, battle-tested automation workflows for self-hosted [n8n](https://n8n.io/). 

Each template is **modular**, **fully documented**, and **decoupled from secrets**—ready to import and run in minutes.

---

## ⚡ Quick Navigation

| Section | Description |
| :--- | :--- |
| **[📂 Template Catalog](#-template-catalog)** | Browse available and upcoming automation blueprints |
| **[🚀 4-Step Quickstart](#-how-to-use-any-template)** | How to import and run any template on your instance |
| **[🧩 What is n8n & Templates?](#-what-is-n8n--what-are-templates)** | A quick primer on n8n and workflow templates |
| **[🏗️ Template Standards](#-template-folder-structure)** | Standard folder structure and quality checklist |
| **[🗺️ Future Roadmap](#-upcoming-automation-domains)** | Planned templates across AI, Sales, DevOps, and Data |
| **[🔒 Security & Policies](#-security--credentials)** | Credential isolation and safe sharing policies |

---

## 🧩 What is n8n & What are Templates?

### What is n8n?
* **Open-Source & Self-Hostable**: An extensible workflow automation tool with no per-execution cloud fees.
* **API Integration Engine**: Connects native nodes, webhooks, and REST APIs with JavaScript and Python logic.
* **Full Data Ownership**: Runs entirely on your private server, Docker container, or cloud VM.

### What is an n8n Template?
* **Pre-Built Architecture**: A standardized JSON blueprint containing all node connections, settings, and routing.
* **Plug-and-Play**: Import the file into n8n, map your API credentials, and start executing immediately.
* **Zero Guesswork**: Bypasses the need to design complex API payloads, retries, and error handling from scratch.

---

## 🎁 What You Get in This Repository

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       PRODUCTION TEMPLATE STANDARDS                     │
├───────────────────┬───────────────────┬─────────────────────────────────┤
│ 🔒 Zero Secrets   │ 📦 Modular Design │ 📚 Full Documentation           │
│ No API keys or    │ Self-contained in │ Dedicated setup, API config,    │
│ private IDs saved │ dedicated folders │ and troubleshooting for each    │
├───────────────────┼───────────────────┼─────────────────────────────────┤
│ 🛡️ Error Handling │ 🔄 Safe Defaults  │ 🚀 Real-World Business Cases    │
│ Built-in retries, │ Uploads default   │ Content pipelines, AI agents,   │
│ checks & schemas  │ to private mode   │ lead flows, and DevOps monitors │
└───────────────────┴───────────────────┴─────────────────────────────────┘
```

---

## 🗂️ Template Catalog

Click any template name to view its dedicated documentation and workflow JSON:

| Domain | Template | Integrations | Status | Documentation & Blueprint |
| :--- | :--- | :--- | :---: | :--- |
| 📢 **Content & Social Media** | **Content Posting Automation** | Google Gemini (3.7 & 3.8 Flash), Meta Instagram Graph API, YouTube Data API v3, Cloudinary | `Production Ready` | [Open Template Guide](Content%20Posting%20Automation/README.md) |
| 🤖 **AI & Autonomous Agents** | *Multi-Model Research Agent* | OpenAI, Anthropic Claude, Perplexity API, Google Drive | `In Development` | *Coming Soon* |
| 💼 **Sales, CRM & Leads** | *Inbound Lead Enrichment* | HubSpot CRM, Apollo.io, Clearbit, Slack | `In Development` | *Coming Soon* |
| 🛠️ **DevOps & Cloud Ops** | *Docker Auto-Healing & Alerts* | Docker API, Telegram / Discord, GitHub API | `Planned` | *Roadmap* |
| 💬 **Customer Operations** | *AI Support Ticket Triage* | Zendesk API, Vector DB, Google Gemini Vision | `Planned` | *Roadmap* |
| 📊 **Data & E-Commerce** | *Multi-Store ETL & BigQuery Sync* | Shopify, Stripe, PostgreSQL, BigQuery | `Planned` | *Roadmap* |

---

## 🚀 How to Use Any Template

Every template in this library follows the same **4-step deployment cycle**:

```
[1. Select Template] ──> [2. Review Setup Docs] ──> [3. Import JSON] ──> [4. Connect & Run]
```

### Step 1: Select Your Template
* Open the template directory from the catalog (e.g., [`Content Posting Automation/`](Content%20Posting%20Automation/README.md)).

### Step 2: Review the Dedicated Documentation
Each template folder includes a `docs/` subfolder with specific guides:
* `docs/SETUP-GUIDE.md`: Docker configurations, volume mounts, and environment variables.
* `docs/API-CONFIGURATION.md`: Third-party developer portal setup and permission scopes.
* `docs/TROUBLESHOOTING.md`: Common error messages and step-by-step solutions.

### Step 3: Import the Workflow into n8n
1. Open your n8n web dashboard (`http://localhost:5678`).
2. Navigate to **Workflows** -> **Add Workflow** -> **Import from File**.
3. Select the sanitized JSON file from the template's `workflow/` folder.

### Step 4: Link Credentials & Run
1. Open n8n **Credentials Manager** and create your credentials for the required services.
2. Replace any capitalized placeholder values (e.g., `YOUR_ACCOUNT_ID`).
3. Follow the template's `docs/PRE-DEPLOYMENT-CHECKLIST.md` and execute your first test!

---

## 🏗️ Template Folder Structure

Every template is strictly organized using this predictable layout:

```
Template-Name/
├── README.md                          # Template landing page & architecture overview
├── LICENSE                            # Template-specific license notice
├── CHANGELOG.md                       # Release notes & version history (from v1.0.0)
├── SECURITY.md                        # Secret sanitization guidelines
├── .gitignore                         # Local exclusion rules (.env, test media)
│
├── docs/                              # Technical documentation suite
│   ├── SETUP-GUIDE.md                 # Step-by-step installation instructions
│   ├── API-CONFIGURATION.md           # Developer portal setup & permission scopes
│   ├── WORKFLOW-EXPLANATION.md        # Node-by-node audit & Mermaid diagrams
│   ├── DIRECTORY-STRUCTURE.md         # File queue & volume mapping guides
│   ├── TROUBLESHOOTING.md             # Error diagnoses & solutions
│   └── PRE-DEPLOYMENT-CHECKLIST.md    # Pre-launch production verification checklist
│
├── workflow/                          # Production workflow
│   └── Template-Name.json             # Sanitized, import-ready workflow JSON
│
└── assets/                            # Visual documentation
    ├── README.md                      # Asset guidelines
    ├── workflow-overview.png          # Full canvas screenshot
    └── setup-screenshots/             # Masked setup walkthrough captures
```

---

## 🛠️ General Prerequisites

Before running any template, ensure your host environment is ready:

- [x] **Docker & Docker Compose**: [Install Docker Desktop](https://docs.docker.com/desktop/) (Windows/macOS) or Docker Engine (Linux).
- [x] **n8n Instance**: Running locally or on a cloud server (v1.x or v2.x supported).
- [x] **Platform Accounts**: Active developer accounts for the third-party APIs used by your chosen template.

---

## 🗺️ Upcoming Automation Domains

New templates are actively developed and documented across these categories:

* **📢 Social Media & Content**: Multi-platform clipping, transcription pipelines, and newsletter syndication.
* **🤖 Autonomous AI Agents**: RAG knowledge bases, multi-agent research swarms, and AI document parsers.
* **💼 CRM & Inbound Sales**: Lead scoring, email verification waterfalls, and meeting booking assistants.
* **🛠️ Cloud & Site Reliability**: SSL renewal alerts, GitHub PR automated reviewers, and server disk monitors.
* **🛒 E-commerce & Payments**: Abandoned cart triggers, inventory sync, and refund notifications.

---

## 🔒 Security & Credentials

* **No Secrets in Files**: API keys, access tokens, and private server hashes are never committed to this repository.
* **Safe Placeholders**: Identifiers are clearly marked with uppercase placeholders (e.g., `YOUR_API_KEY`).
* **Environment Protection**: `.env` files and local media queues are ignored by root [.gitignore](.gitignore).
* **Security Policy**: For responsible disclosure and template safety guidelines, read [SECURITY.md](SECURITY.md).

---

## 🤝 Contributing

Contributions of tested, production-grade workflows are welcome!

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for quality criteria, folder naming conventions, and the pre-commit sanitization checklist.

---

## 📄 License

See [LICENSE](LICENSE) for details.

> [!IMPORTANT]
> The repository owner must select and approve a formal open-source or commercial license prior to public distribution.
