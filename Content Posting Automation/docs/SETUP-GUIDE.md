# Complete Beginner Setup Guide

This guide walks you through setting up and running the **Content Posting Automation** workflow on a self-hosted n8n instance from scratch.

Because the original template was built and validated on **Windows 11 with Docker Desktop**, detailed PowerShell examples are provided, alongside Linux/macOS command equivalents.

---

## 1. Prerequisites & Required Software

Before starting, ensure you have the following installed:

1. **Docker Desktop** (Windows / macOS) or **Docker Engine & Docker Compose** (Linux).
   - [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) (Ensure WSL2 backend is enabled).
2. **Web Browser** (Chrome, Firefox, Edge, Brave).
3. **API Accounts**:
   - Google AI Studio (Gemini API Key).
   - Meta for Developers (Instagram Graph API access).
   - Google Cloud Console (YouTube Data API v3 OAuth Client).
   - Cloudinary (Cloud Name, API Key, API Secret).

---

## 2. Docker Installation & Environment Setup

### Step 1: Create Local Host Folders

On your host machine, create directories for n8n persistent data and your local media staging queue.

**Windows PowerShell:**
```powershell
# Create persistent storage folder for n8n and queue folder for media
New-Item -ItemType Directory -Force -Path "D:\n8n_data"
New-Item -ItemType Directory -Force -Path "D:\SocialMediaQueue"
```

**Linux / macOS Terminal:**
```bash
mkdir -p ~/n8n_data
mkdir -p ~/SocialMediaQueue
```

---

## 3. Starting n8n via Docker

You can launch n8n either with a single `docker run` command or with `docker-compose`.

### Option A: Running via Docker Run (Recommended for Quick Start)

**Windows PowerShell:**
```powershell
docker run -d `
  --name n8n `
  --restart unless-stopped `
  -p 5678:5678 `
  -v "D:/n8n_data:/home/node/.n8n" `
  -v "D:/SocialMediaQueue:/files:ro" `
  -e GENERIC_TIMEZONE="Asia/Kolkata" `
  -e TZ="Asia/Kolkata" `
  -e N8N_BLOCK_ENV_ACCESS_IN_NODE="false" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS="true" `
  -e INPUT_FILES_PATH="/files/*" `
  docker.n8n.io/n8nio/n8n:latest
```

**Linux / macOS Terminal:**
```bash
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v "$HOME/n8n_data:/home/node/.n8n" \
  -v "$HOME/SocialMediaQueue:/files:ro" \
  -e GENERIC_TIMEZONE="Asia/Kolkata" \
  -e TZ="Asia/Kolkata" \
  -e N8N_BLOCK_ENV_ACCESS_IN_NODE="false" \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS="true" \
  -e INPUT_FILES_PATH="/files/*" \
  docker.n8n.io/n8nio/n8n:latest
```

### Option B: Running via Docker Compose

Create a `docker-compose.yml` file in your project directory:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-automation
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - GENERIC_TIMEZONE=Asia/Kolkata
      - TZ=Asia/Kolkata
      # Allows expressions like {{ $env.INPUT_FILES_PATH }} in nodes
      - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      # Media folder glob expression
      - INPUT_FILES_PATH=/files/*
    volumes:
      - "D:/n8n_data:/home/node/.n8n" # For Linux: "$HOME/n8n_data:/home/node/.n8n"
      - "D:/SocialMediaQueue:/files:ro" # For Linux: "$HOME/SocialMediaQueue:/files:ro"
```

Start the container:
```powershell
docker compose up -d
```

---

## 4. Initial n8n Onboarding

1. Open your browser and navigate to:
   ```
   http://localhost:5678
   ```
2. When prompted, create your initial owner account (Name, Email, Password).
3. Complete the onboarding wizard and enter the n8n canvas dashboard.

---

## 5. Importing the Workflow JSON

1. Locate the workflow file inside this repository:
   ```
   workflow/Content-Posting-Automation.json
   ```
2. In n8n:
   - Click **Workflows** in the left sidebar.
   - Click the **Add Workflow** button (or **+**).
   - In the top-right corner of the workflow editor, click the **Options menu (`...`)** and select **Import from File**.
   - Select `Content-Posting-Automation.json`.
3. The complete 22-node workflow graph will render on your canvas.

---

## 6. Configuring Credentials

Because credentials are decoupled from workflow exports for security, several nodes will display a credential warning indicator. Configure your credentials as detailed in [API-CONFIGURATION.md](API-CONFIGURATION.md):

1. **Google Gemini API (`googlePalmApi`)**:
   - Go to **Credentials** -> **Add Credential** -> **Google Gemini(PaLM) Api**.
   - Paste your Google AI Studio API key.
   - Link this credential to:
     - `Analyze an image`
     - `Message a model`
     - `Upload a media file`
     - `Analyze video`
     - `Message a model1`
2. **Cloudinary (`httpBasicAuth`)**:
   - Go to **Credentials** -> **Add Credential** -> **Basic Auth**.
   - Enter your Cloudinary API Key as **User** and API Secret as **Password**.
   - Link this credential to `Cloudinary - Upload Media `.
3. **Meta Instagram API (`httpHeaderAuth`)**:
   - Go to **Credentials** -> **Add Credential** -> **Header Auth**.
   - Name: `Authorization`, Value: `Bearer <YOUR_META_LONG_LIVED_TOKEN>`.
   - Link this credential to:
     - `Video Content Merge`
     - `Image Content Merge`
     - `Instagram Reel Post `
     - `Instagram Image Post `
4. **YouTube (`youTubeOAuth2Api`)**:
   - Go to **Credentials** -> **Add Credential** -> **YouTube OAuth2 API**.
   - Enter your Google Cloud OAuth Client ID & Client Secret.
   - Click **Connect my account** and complete authorization.
   - Link this credential to `Upload a video`.

---

## 7. Configuring Parameters & Placeholders in Nodes

Search for and replace these placeholder values inside the workflow:

1. **Cloudinary Cloud Name**:
   - Open node `Cloudinary - Upload Media `.
   - Change URL from:
     ```
     https://api.cloudinary.com/v1_1/YOUR_CLOUDINARY_CLOUD_NAME/auto/upload
     ```
     to your actual Cloudinary Cloud Name.
2. **Instagram Account ID**:
   - Open nodes: `Video Content Merge`, `Image Content Merge`, `Instagram Reel Post `, and `Instagram Image Post `.
   - In each node's URL, replace `YOUR_INSTAGRAM_ACCOUNT_ID` with your 17-digit Instagram Business Account ID:
     ```
     https://graph.facebook.com/v26.0/YOUR_INSTAGRAM_ACCOUNT_ID/media
     https://graph.facebook.com/v26.0/YOUR_INSTAGRAM_ACCOUNT_ID/media_publish
     ```
3. **Brand Customization**:
   - Open node `Message a model`.
   - Inspect the prompt string:
     `You are the social media SEO strategist for [Your Brand], a premium technology brand.`
   - Replace `[Your Brand]` with your company or personal brand name.

---

## 8. Executing Your First Test Run

### Step 1: Queue a Test Media File
Place a single sample image or video into your mounted host queue folder (`D:\SocialMediaQueue` on Windows or `~/SocialMediaQueue` on Linux):
- For an Image test: Place a `.jpg` or `.png` file (e.g., `test_product.jpg`).
- For a Video test: Place a short `.mp4` clip (e.g., `test_clip.mp4`).

> [!TIP]
> Ensure only **one** test file is in the folder during your first run to make log tracking straightforward.

### Step 2: Trigger the Workflow
1. Return to the n8n canvas.
2. Ensure YouTube node privacy is set to `"private"` (default).
3. Click the **Test step** or **Execute workflow** button on the `When clicking ‘Execute workflow’` node.

---

## 9. Verifying Successful Execution

Observe the execution nodes light up green in sequence:

1. **`Read/Write Files from Disk`**: Should output 1 item with binary metadata.
2. **`Edit Fields`**: Emits `originalFilename` without file extension.
3. **`Switch`**:
   - Routes to `Analyze an image` if testing an image.
   - Routes to `Upload a media file` -> `Analyze video` if testing a video.
4. **`Message a model` / `Message a model1`**: Generates a valid JSON object containing `instagram`, `linkedin`, and `youtube` blocks.
5. **`Result Enhancer` / `Result Enhancer1`**: Cleans JSON and attaches metadata.
6. **`Cloudinary - Upload Media `**: Returns Cloudinary `secure_url`.
7. **`Merge` Nodes**: Combine AI text with media URLs.
8. **`Instagram Content Merge` + `Instagram Post`**:
   - First node creates a container (`id: "178..."`).
   - Second node publishes the container (`id: "178..."`).
9. **`Upload a video`** *(Video branch only)*:
   - Uploads to YouTube and outputs YouTube video ID. Check your YouTube Studio Videos tab under "Private" videos to confirm.

Congratulations! Your automated social media publishing pipeline is fully operational.
