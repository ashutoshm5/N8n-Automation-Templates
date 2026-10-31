# Security Policy & Best Practices

This document outlines the security architecture of the **Content Posting Automation** workflow, operational security guidelines for self-hosting n8n, and steps to prevent credential leaks when sharing workflows or contributing to public repositories.

---

## 1. n8n Credentials Manager Architecture

n8n utilizes an internal **Credentials Manager** to decouple sensitive authentication details from workflow execution definitions:

* **Encryption at Rest:** All credentials stored in the n8n database (SQLite/PostgreSQL) are encrypted using an instance-specific encryption key defined by the `N8N_ENCRYPTION_KEY` environment variable.
* **Separation of Concerns:** Nodes in workflow JSON files reference credentials by an internal identifier (`id`) and a human-readable descriptor (`name`).
* **Export Decoupling:** When you export a workflow via the n8n UI, **raw credentials (API keys, OAuth tokens, client secrets) are NEVER exported in the workflow JSON**. Only placeholder references are included.

> [!IMPORTANT]
> The credential references (e.g., `"id": "Rf1bIPI2vzxGifEd"`, `"name": "Google Gemini(PaLM) Api account"`) present in the exported JSON file are database pointers belonging to the author's local n8n instance. **They do not contain secrets**, nor will they function on your instance. You must configure your own credentials in your local n8n instance.

---

## 2. API Key & OAuth Token Protection

### Google Gemini (Google AI Studio)
* Restrict API keys in the [Google Cloud Console / Google AI Studio](https://aistudio.google.com/) where possible.
* Never hardcode your Gemini API key inside JavaScript expressions or code nodes. Always bind it to an n8n `googlePalmApi` credential.

### Meta Instagram Graph API
* Use short-lived tokens only during testing; generate system user or long-lived page access tokens for production.
* Grant only the minimum permissions required (`instagram_basic`, `instagram_content_publish`, `pages_show_list`, `pages_read_engagement`).
* Never commit access tokens to Git. Manage tokens strictly inside n8n's **Header Auth** (`httpHeaderAuth`) credential manager, configured to inject the `Authorization: Bearer <TOKEN>` header.

### YouTube Data API v3
* Configure your OAuth 2.0 Client credentials in the Google Cloud Console.
* Set the Authorized Redirect URI strictly to your n8n instance callback URL (`https://<your-n8n-domain>/rest/oauth2-credential/callback`).
* Never distribute your OAuth Client Secret or refresh tokens.

### Cloudinary
* Cloudinary credentials require an API Key and API Secret paired via HTTP Basic Authentication (`httpBasicAuth`).
* Cloudinary API Secrets grant administrative capabilities over your media library. Protect this credential with high priority and rotate it immediately if exposed.

---

## 3. Environment Variable Security

Certain settings are passed into n8n via container environment variables (`.env` file):

```bash
# Example: Local media path
INPUT_FILES_PATH=/files/*

# Security hardening for n8n instances
N8N_ENCRYPTION_KEY=generate_a_random_32_char_hex_key_here
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

### `.env` File Protection
1. **Never commit `.env` files to source control.** Verify that `.env` is listed in [.gitignore](.gitignore).
2. Distribute only `.env.example` templates containing sanitized placeholder values.
3. Restrict host permissions on the `.env` file (`chmod 600 .env` on Linux / macOS).

---

## 4. Preventing Leaks Before Publishing to GitHub

Before committing changes or sharing workflow JSON files publicly, execute the following audit:

| Target Item | What to Look For | Safe State / Replacement |
| :--- | :--- | :--- |
| **Meta Instagram URLs** | Hardcoded Instagram Business Account IDs (e.g., `178414...`) | Replace with `YOUR_INSTAGRAM_ACCOUNT_ID` |
| **Cloudinary URLs** | Personal Cloud Names (e.g., `my_cloud_name`) | Replace with `YOUR_CLOUDINARY_CLOUD_NAME` |
| **Local File Paths** | Direct system paths (e.g., `C:\Users\username\...`) | Use `{{ $env.INPUT_FILES_PATH \|\| '/path/to/your/media/*' }}` |
| **Root Metadata** | Server `instanceId` hash | Delete `"instanceId"` from the root `"meta"` block |
| **Workflow ID** | Database primary key (`"id": "bKAzYGEx..."`) | Omit or remove the root `"id"` field |
| **YouTube Privacy** | Upload default visibility | Set `"privacyStatus": "private"` |
| **Brand Names in Prompts** | Specific corporate or personal trademarks | Replace with generic placeholder `[Your Brand]` |

---

## 5. Security in Custom Code & HTTP Request Nodes

* **Zero Hardcoded Secrets in Code Nodes:**
  Review nodes like `Edit Fields`, `Result Enhancer`, and `Result Enhancer1`. These nodes process strings and clean JSON output. Ensure no static API keys or webhook URLs are pasted into their scripts.
* **Header and Query Parameters:**
  In HTTP Request nodes (`Cloudinary - Upload Media `, `Video Content Merge`, `Image Content Merge`, `Instagram Reel Post `, `Instagram Image Post `), all authentication is delegated to generic credential types (`httpBasicAuth` and `httpHeaderAuth`). Never pass raw tokens in `parameters.queryParameters` or plain body fields.

---

## 6. Safe Screenshot & Log Sharing

When capturing screenshots or sharing logs for debugging:
* Mask the browser address bar if it displays private internal domain names or IP addresses.
* Mask Instagram Business Account IDs, Cloudinary Cloud Names, and Cloudinary upload URLs.
* Mask YouTube channel IDs and Google Cloud project numbers.
* Never share raw Docker logs without grepping and stripping sensitive authorization headers or request bodies.

---

## 7. Credential Rotation & Incident Response

If you suspect an API key or access token has been compromised:
1. **Immediately revoke the key** in the respective provider dashboard (Google AI Studio, Meta App Dashboard, Cloudinary Console, Google Cloud Console).
2. Generate a new key/secret pair.
3. Open n8n, navigate to **Credentials**, open the affected credential, update the value, and click **Save**. All nodes referencing that credential will automatically update without altering the workflow JSON.
4. Review API audit logs on Meta and Google Cloud to inspect unauthorized operations.

---

## 8. Reporting Security Issues

If you discover a security vulnerability or accidental secret exposure within this repository, please do not open a public issue. Contact the repository maintainer directly via private channels.
