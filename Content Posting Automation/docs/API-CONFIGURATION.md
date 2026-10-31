# External API Configuration Guide

This guide walks through configuring the external APIs and credentials required by the **Content Posting Automation** workflow.

The workflow interacts with four external platforms:
1. **Google Gemini API** (Multimodal Vision & Copywriting)
2. **Meta Instagram Graph API** (Photo & Reels Publishing)
3. **YouTube Data API v3** (Video Publishing)
4. **Cloudinary API** (Public Media Staging Host for Instagram)

> [!WARNING]
> **Credential Decoupling Notice:** When importing this workflow into your n8n instance, existing credential references will display missing or unlinked icons. You must generate your own API credentials following the steps below and link them inside n8n.

---

## 1. Google Gemini API (Google AI Studio)

The workflow relies on Google Gemini for image analysis, video analysis, and platform-tailored copywriting across five distinct nodes:
* `Analyze an image`: Uses `models/gemini-3.7-flash` (Image inspection)
* `Message a model`: Uses `models/gemini-3.7-flash` (Copy generation from image)
* `Upload a media file`: Uses Gemini File API (Uploads video binary for URI)
* `Analyze video`: Uses `models/gemini-3.8-flash` (Video scene & dialogue inspection)
* `Message a model1`: Uses `models/gemini-3.7-flash` (Copy generation from video)

All five nodes share the same credential type: **Google Gemini(PaLM) Api account** (`googlePalmApi`).

### Step 1: Obtain a Gemini API Key
1. Navigate to [Google AI Studio](https://aistudio.google.com/).
2. Sign in with your Google Account.
3. Click **Get API key** in the left sidebar navigation.
4. Click **Create API key in new project** (or select an existing Google Cloud project).
5. Copy the generated API key.

### Step 2: Configure the Credential in n8n
1. Open your n8n dashboard and click **Credentials** in the left sidebar.
2. Click **Add Credential** and search for **Google Gemini(PaLM) Api**.
3. Name the credential (e.g., `Google Gemini(PaLM) Api account`).
4. Paste your API key into the **API Key** field.
5. Click **Save**.

### Step 3: Link Nodes in the Workflow
Open each of the five Gemini nodes in the n8n canvas and select your newly created credential under the **Credential to connect with** dropdown.

### Common Gemini Errors
* **`400 Invalid Argument / Model Not Found`**: Verify that your Google Cloud project has access to `gemini-3.7-flash` and `gemini-3.8-flash`.
* **`429 Resource Exhausted / Rate Limit`**: The video pipeline contains automated retry policies (`Analyze video` retries 5 times with a 200ms backoff). For production scaling, consider a paid Google AI Studio tier.
* **`File Size Exceeded`**: Video uploads via the Gemini File API must adhere to Google's file size limits (up to 2 GB per file).

---

## 2. Meta Instagram Graph API

The workflow publishes images to the Instagram Feed and videos to Instagram Reels using the Meta Graph API v26.0 endpoint (`https://graph.facebook.com/v26.0/...`).

### Prerequisites
* A **Facebook Page** linked to an **Instagram Professional (Creator or Business) Account**. Personal Instagram profiles cannot be automated via the Graph API.
* A [Meta for Developers](https://developers.facebook.com/) account.

### Step 1: Create a Meta Developer App
1. Go to the [Meta Developer Dashboard](https://developers.facebook.com/apps/).
2. Click **Create App** and select **Other** -> **Business** (or **Consumer** with Instagram Graph API product).
3. Name your app (e.g., `Social-Posting-Automation`).
4. Under **Add products to your app**, locate **Instagram Graph API** and click **Set Up**.

### Step 2: Permissions Required
Ensure your User or System User Token has the following approved permissions:
* `instagram_basic`: Required to read account information and profile metadata.
* `instagram_content_publish`: Mandatory for publishing photos, carousels, and Reels.
* `pages_show_list`: Required to discover connected Facebook Pages.
* `pages_read_engagement`: Required to read page-linked Instagram assets.

### Step 3: Obtain Long-Lived Token & Instagram Business Account ID
1. Open the [Meta Graph API Explorer](https://developers.facebook.com/tools/explorer/).
2. Select your App and User. Request permissions: `instagram_basic`, `instagram_content_publish`, `pages_show_list`.
3. Generate a User Access Token.
4. Query the Facebook Pages endpoint to find your connected Instagram Business ID:
   ```http
   GET https://graph.facebook.com/v26.0/me/accounts?fields=name,instagram_business_account
   ```
5. Note your **`instagram_business_account.id`** (a 17-digit numeric string, e.g., `17841400000000000`).
6. Exchange your short-lived token for a **Long-Lived Page Access Token** or create a **System User Token** in Meta Business Manager to prevent token expiration every 60 days.

### Step 4: Configure n8n Header Auth Credential
The workflow nodes use **Header Auth** (`httpHeaderAuth`) named `Meta Instagram API`:
1. In n8n, click **Credentials** -> **Add Credential**.
2. Select **Header Auth**.
3. Credential Name: `Meta Instagram API`.
4. Set **Name**: `Authorization`.
5. Set **Value**: `Bearer YOUR_LONG_LIVED_ACCESS_TOKEN`.
6. Click **Save**.

### Step 5: Update Workflow Node URLs
The workflow contains four HTTP Request nodes interacting with Instagram:
1. `Video Content Merge`
2. `Image Content Merge`
3. `Instagram Reel Post `
4. `Instagram Image Post `

Replace `YOUR_INSTAGRAM_ACCOUNT_ID` in their URL parameters:
```http
# Media Container Creation
https://graph.facebook.com/v26.0/YOUR_INSTAGRAM_ACCOUNT_ID/media

# Media Publish Step
https://graph.facebook.com/v26.0/YOUR_INSTAGRAM_ACCOUNT_ID/media_publish
```

---

## 3. YouTube Data API v3

YouTube video publishing is handled by the native n8n node `Upload a video` (`n8n-nodes-base.youTube`), utilizing OAuth 2.0 authentication.

### Step 1: Google Cloud Project Setup
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create or select a project.
3. In the search bar, search for **YouTube Data API v3** and click **Enable**.

### Step 2: Configure OAuth Consent Screen
1. Navigate to **APIs & Services** -> **OAuth consent screen**.
2. Select User Type: **External** (or **Internal** for Google Workspace organizations).
3. Fill in required App Name and Support Email.
4. In the **Scopes** step, click **Add or Remove Scopes** and add:
   * `https://www.googleapis.com/auth/youtube.upload` (Manage your YouTube videos)
   * `https://www.googleapis.com/auth/youtube` (Full YouTube management)
5. Under **Test Users**, add the Google email address of the YouTube channel owner.

### Step 3: Create OAuth 2.0 Client Credentials
1. Navigate to **APIs & Services** -> **Credentials** -> **Create Credentials** -> **OAuth client ID**.
2. Select Application Type: **Web application**.
3. Set Name: `n8n YouTube Uploader`.
4. In **Authorized redirect URIs**, add your n8n OAuth callback URL:
   ```
   https://<your-n8n-domain>/rest/oauth2-credential/callback
   ```
   *(For local testing on localhost: `http://localhost:5678/rest/oauth2-credential/callback`)*.
5. Click **Create** and copy your **Client ID** and **Client Secret**.

### Step 4: Configure the Credential in n8n
1. In n8n, click **Credentials** -> **Add Credential**.
2. Select **YouTube OAuth2 API**.
3. Name: `YouTube account`.
4. Enter your **Client ID** and **Client Secret**.
5. Click **Connect my account**. A Google authorization window will prompt you to grant channel permissions.
6. Once connected, click **Save**.

### Step 5: Node Privacy Settings
In node `Upload a video`:
* Privacy Status is configured to `"private"` by default to ensure safe testing.
* Category ID is set to `28` (Science & Technology). Adjust this value in the node settings if your content belongs to another genre (e.g., `27` for Education, `24` for Entertainment).

---

## 4. Cloudinary API

Meta's Instagram Graph API cannot ingest binary media directly from private servers; it requires a publicly accessible HTTPS URL. The workflow uses Cloudinary as an automated media staging host via node `Cloudinary - Upload Media `.

### Step 1: Obtain Cloudinary Credentials
1. Register or log into [Cloudinary](https://cloudinary.com/).
2. Open the **Dashboard / API Keys** page.
3. Note your:
   * **Cloud Name**: (e.g., `my-cloud-name`)
   * **API Key**: (e.g., `123456789012345`)
   * **API Secret**: (e.g., `abcdefghijklmnopqrstuvwxyz01234`)

### Step 2: Configure n8n HTTP Basic Auth Credential
The node authenticates with Cloudinary using **HTTP Basic Authentication** (`httpBasicAuth`):
1. In n8n, click **Credentials** -> **Add Credential**.
2. Select **Basic Auth**.
3. Name: `Cloudinary API`.
4. Set **User**: Paste your Cloudinary **API Key**.
5. Set **Password**: Paste your Cloudinary **API Secret**.
6. Click **Save**.

### Step 3: Update Cloudinary URL in the Node
Open node `Cloudinary - Upload Media ` in the workflow and replace `YOUR_CLOUDINARY_CLOUD_NAME` in the URL:
```http
POST https://api.cloudinary.com/v1_1/YOUR_CLOUDINARY_CLOUD_NAME/auto/upload
```

### Video Considerations for Cloudinary
* For large video files (>100 MB), Cloudinary requires chunked uploads or elevated storage tiers.
* Standard free tier accounts have maximum video duration and payload limits. Monitor usage in the Cloudinary Console.

---

## 5. Credential Mapping Summary Table

| n8n Credential Name in Template | Credential Type in n8n | Target Provider | Nodes Utilizing Credential |
| :--- | :--- | :--- | :--- |
| `Google Gemini(PaLM) Api account` | `googlePalmApi` | Google AI Studio | `Analyze an image`<br>`Message a model`<br>`Upload a media file`<br>`Analyze video`<br>`Message a model1` |
| `Cloudinary API` | `httpBasicAuth` | Cloudinary | `Cloudinary - Upload Media ` |
| `Meta Instagram API` | `httpHeaderAuth` | Meta Instagram Graph API | `Video Content Merge`<br>`Image Content Merge`<br>`Instagram Reel Post `<br>`Instagram Image Post ` |
| `YouTube account` | `youTubeOAuth2Api` | YouTube Data API v3 | `Upload a video` |
