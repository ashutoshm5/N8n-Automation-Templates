# n8n Automation Templates Library

[![n8n](https://img.shields.io/badge/Platform-n8n%20Workflow%20Automation-EA4B71?logo=n8n)](https://n8n.io/)
[![Self-Hosted](https://img.shields.io/badge/Deployment-Docker%20%7C%20Self--Hosted-2496ED?logo=docker)](https://www.docker.com/)
[![Zero Embedded Secrets](https://img.shields.io/badge/Security-Zero%20Embedded%20Secrets-brightgreen)](SECURITY.md)
[![Modular Architecture](https://img.shields.io/badge/Design-Fully%20Modular%20Templates-blueviolet)](#-template-catalog)

Welcome to the **n8n Automation Templates Library** — a curated, open-source collection of production-grade, battle-tested automation workflows designed for self-hosted [n8n](https://n8n.io/) environments.

Whether you are automating social media content pipelines, deploying autonomous AI agents, orchestrating DevOps alerts, syncing CRM leads, or processing multi-source data, this repository provides **ready-to-import blueprints** complete with exhaustive documentation, configuration steps, and security hardening.

---

## 📖 What is n8n?

[n8n](https://n8n.io/) is an extensible, open-source workflow automation platform. Often described as a developer-friendly, self-hostable alternative to Zapier or Make, n8n lets you connect any application with an API, manipulate data using native JavaScript/Python code nodes, and build complex, branching business logic without vendor lock-in or recurring per-task cloud fees.

---

## ⚡ What are n8n Templates?

An **n8n template** is a pre-built, serialized JSON blueprint of an automation pipeline. When imported into an n8n canvas, it instantly recreates the full operational workflow:
* Node arrangements and topological execution connections.
* Parameter configurations, expressions, and routing logic.
* Data transformation scripts and filtering rules.
* Integration endpoints for external webhooks, APIs, and AI models.

Instead of spending days designing workflows from scratch, debugging API authentication headers, and handling edge cases, you simply **import the template, connect your credentials, and start automating in minutes**.

---

## 🎁 What You Get in This Repository

Every template in this repository is built to **commercial-grade production standards**:

* 🔒 **Zero Embedded Secrets**: All workflows strictly utilize n8n's encrypted **Credentials Manager**. No API keys, passwords, client secrets, or private server instance IDs exist inside the exported JSON files.
* 📦 **Self-Contained Modularity**: Each template lives in its own dedicated directory with its own workflow JSON, step-by-step setup guide, external API configuration manual, and troubleshooting handbook.
* 🛡️ **Defensive Error Handling**: Workflows include automated retry policies, fallback logic, JSON schema validation, and safe defaults (e.g., publishing as Private/Draft by default).
* 📚 **Exhaustive Documentation**: Every template includes an architectural node-by-node audit, Mermaid execution flowcharts, Docker volume mapping instructions, and a pre-deployment sanity checklist.
* 🚀 **Multi-Domain Scalability**: A structured, growing catalog spanning Content Creation, AI Engineering, Sales & CRM, DevOps, Data Pipelines, and Customer Operations.

---

## 🗂️ Template Catalog

Explore our growing collection of automation blueprints. **Click on any template** to view its dedicated documentation, architecture, and installation guide:

| Category | Template Name | Key Integrations | Status | Documentation & Blueprint |
| :--- | :--- | :--- | :---: | :--- |
| **Content & Social Media** | **Content Posting Automation** | Google Gemini (3.7 & 3.8 Flash), Meta Instagram Graph API, YouTube Data API v3, Cloudinary | `Production Ready` | [Explore Template](Content%20Posting%20Automation/README.md) |
| **AI & Intelligent Agents** | *Multi-Model Research & Summarization Agent* | OpenAI, Anthropic Claude, Perplexity API, Google Drive | `In Development` | *Coming Soon* |
| **Sales & CRM** | *Inbound Lead Enrichment & Slack Routing* | HubSpot CRM, Apollo.io, Clearbit, Slack | `In Development` | *Coming Soon* |
| **DevOps & Cloud Ops** | *Automated Incident Response & Docker Monitor* | GitHub, Docker API, Telegram / Discord, Grafana | `Planned` | *Roadmap* |
| **Customer Support** | *AI Ticket Triage & Zendesk Auto-Responder* | Zendesk API, Pinecone Vector DB, Gemini Vision | `Planned` | *Roadmap* |
| **Data & Analytics** | *E-commerce ETL & Google BigQuery Sync* | Shopify API, Stripe, PostgreSQL, BigQuery | `Planned` | *Roadmap* |

> 💡 **Future Expansions**: This repository is actively expanding. New templates across diverse fields of automation are regularly audited, documented, and published here. Check our [Roadmap](#-upcoming-domains--roadmap) or [CONTRIBUTING.md](CONTRIBUTING.md) to suggest a workflow.

---

## 🚀 How to Use Any Template in 4 Steps

Each template folder contains everything required to deploy the automation independently. Follow these 4 universal steps:

```
[1. Choose a Template] ──> [2. Review Setup Guide] ──> [3. Import Workflow JSON] ──> [4. Connect Credentials & Run]
```

### Step 1: Choose Your Template
Browse the [Template Catalog](#-template-catalog) above and open the directory of the workflow you wish to use (for example, [`Content Posting Automation/`](Content%20Posting%20Automation/README.md)).

### Step 2: Review the Template's Setup Guide
Open the template's `docs/SETUP-GUIDE.md` and `docs/API-CONFIGURATION.md`. These files provide:
* Required Docker run / compose configurations and volume mounts.
* External API permissions, developer portal setup, and OAuth scopes.
* Required environment variables (e.g., input folders, timezone, port settings).

### Step 3: Import the Workflow JSON into n8n
1. Open your n8n web dashboard (e.g., `http://localhost:5678`).
2. Click **Workflows** -> **Add Workflow** -> **Import from File**.
3. Select the sanitized workflow JSON from the template's `workflow/` folder (e.g., `Content-Posting-Automation.json`).
4. The complete node graph will render on your n8n canvas.

### Step 4: Link Your Credentials & Execute
1. Open the n8n **Credentials Manager** and add your own API credentials for the services used in the workflow.
2. Replace any capitalized configuration placeholders (e.g., `YOUR_ACCOUNT_ID` or `YOUR_CLOUD_NAME`).
3. Complete the template's `docs/PRE-DEPLOYMENT-CHECKLIST.md` and trigger a test execution!

---

## 📂 Anatomy of Every Template Folder

To ensure consistency and ease of use, every template folder in this repository follows a standardized, modular structure:

```
Template-Name/
│
├── README.md                          # Template landing page, feature matrix & summary
├── LICENSE                            # Template license notice
├── CHANGELOG.md                       # Versioned release notes (starting at v1.0.0)
├── SECURITY.md                        # Template-specific security guidelines
├── .gitignore                         # Local exclusion rules (logs, media, .env)
│
├── docs/                              # In-depth technical documentation
│   ├── SETUP-GUIDE.md                 # Beginner installation (Docker, Windows, Linux)
│   ├── API-CONFIGURATION.md           # External developer portals & OAuth scopes
│   ├── WORKFLOW-EXPLANATION.md        # Node-by-node audit with Mermaid flowcharts
│   ├── DIRECTORY-STRUCTURE.md         # File queue & volume mapping guides (if applicable)
│   ├── TROUBLESHOOTING.md             # Error diagnoses, rate limits & solutions
│   └── PRE-DEPLOYMENT-CHECKLIST.md    # Pre-launch sanity checklist
│
├── workflow/
│   └── Template-Name.json             # Production-ready, sanitized workflow JSON
│
└── assets/                            # Canvas captures & architecture schematics
    ├── README.md                      # Asset guidelines
    ├── workflow-overview.png          # High-resolution canvas screenshot
    └── setup-screenshots/             # Masked setup walkthrough captures
```

---

## 🛠️ General Prerequisites

To run any template from this repository, you will need:

1. **Docker & Docker Compose**: Recommended deployment method for self-hosting n8n.
   - [Install Docker Desktop (Windows / macOS)](https://docs.docker.com/desktop/)
   - [Install Docker Engine (Linux)](https://docs.docker.com/engine/install/)
2. **n8n Instance (v1.x or v2.x)**: Self-hosted via Docker or deployed on a VPS / Cloud VM.
3. **Accounts for Integrations**: Accounts with the relevant platforms used by your selected template (e.g., Google AI Studio, Meta Developer, YouTube Cloud Console, Cloudinary, Slack, etc.).

---

## 🗺️ Upcoming Domains & Roadmap

We are developing new templates across several automation domains:

* **📢 Social Media & Content Marketing**: Multi-channel syndication, automated subtitle generation, podcast clipping, and newsletter syndication.
* **🤖 Autonomous AI Workflows**: Retrieval-Augmented Generation (RAG) pipelines, multi-agent research swarms, automated document analysis, and AI voice call routing.
* **💼 CRM, Sales & Lead Gen**: Automated LinkedIn profile enrichment, cold email sequence dispatchers, inbound webhook lead routing, and pipeline velocity monitors.
* **🛠️ Cloud & DevOps Automation**: Docker container auto-healing, GitHub PR review bots, SSL certificate expiration monitors, and cloud cost alert dispatchers.
* **🛒 E-commerce & Logistics**: Shopify inventory reconciliation, abandoned cart recovery sequences, and invoice extraction pipelines.

---

## 🤝 Contributing New Templates

Have an innovative, reliable n8n workflow that solves a real business problem? We welcome contributions!

Please review our [CONTRIBUTING.md](CONTRIBUTING.md) guide for folder structure conventions, sanitization rules, and documentation standards before opening a Pull Request.

---

## 🔐 Security & Responsible Disclosure

This repository enforces strict sanitization on all published workflows. For our complete security policy, secret isolation guidelines, and responsible disclosure process, see [SECURITY.md](SECURITY.md).

---

## 📄 License

See [LICENSE](LICENSE) for details.

> **Notice:** The repository owner must select and approve a formal software license prior to public distribution.
#   N 8 n - A u t o m a t i o n - T e m p l a t e s  
 