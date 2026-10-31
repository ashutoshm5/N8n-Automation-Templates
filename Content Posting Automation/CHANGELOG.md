# Changelog

All notable changes to the **Content Posting Automation** workflow and documentation will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-10-31

### Added
- **First Documented Release** of the Content Posting Automation workflow for self-hosted n8n.
- **Media Ingestion & Routing**:
  - `Read/Write Files from Disk` node configured with dynamic environment expression `{{ $env.INPUT_FILES_PATH || '/path/to/your/media/*' }}`.
  - Multi-condition switch routing for image (`image/*`) and video (`video/*`) MIME types.
- **AI Media Analysis & Copywriting Engine**:
  - Google Gemini 3.7 Flash integration for automated image scene understanding and brand-focused caption drafting.
  - Google Gemini File API media upload step followed by Gemini 3.8 Flash video analysis with automated retries.
  - JSON-enforced content generation for Instagram, LinkedIn, and YouTube.
- **Publishing Integrations**:
  - Cloudinary HTTP Multipart integration for automated public hosting of Instagram media assets.
  - Meta Instagram Graph API integration for two-stage Reels and Feed Photo publishing (Media Container creation followed by `media_publish`).
  - YouTube Data API v3 direct binary upload for video publishing with tags, description formatting, and private default status.
- **Robust JavaScript Processing**:
  - `Edit Fields` node for clean filename sanitization.
  - `Result Enhancer` and `Result Enhancer1` nodes for markdown fence stripping, JSON schema enforcement, and cross-branch data pairing.
- **Documentation Suite**:
  - Comprehensive `README.md` landing page.
  - Step-by-step beginner `docs/SETUP-GUIDE.md` for Docker, Windows, and Linux.
  - Exhaustive `docs/API-CONFIGURATION.md` for Gemini, Meta Instagram Graph API, YouTube OAuth 2.0, and Cloudinary.
  - In-depth node-by-node `docs/WORKFLOW-EXPLANATION.md` with Mermaid execution flowcharts.
  - Media volume and mapping guide in `docs/DIRECTORY-STRUCTURE.md`.
  - Security and secret sanitization practices in `SECURITY.md`.
  - Diagnostic and error resolution guide in `docs/TROUBLESHOOTING.md`.
  - Production readiness audit in `docs/PRE-DEPLOYMENT-CHECKLIST.md`.

### Known Limitations in Initial Release
- **LinkedIn Publishing Not Implemented**: The workflow generates structured LinkedIn captions, keywords, hashtags, and CTAs, but does not contain a LinkedIn publishing node. Content must be scheduled or posted manually.
- **Cloudinary Dependency for Instagram**: The Meta Graph API requires public HTTPS asset URLs; local media cannot be passed directly into Instagram containers.
