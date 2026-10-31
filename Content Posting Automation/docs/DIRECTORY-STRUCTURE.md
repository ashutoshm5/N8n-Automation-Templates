# Media Directory Structure & Volume Mapping Guide

This document explains how the **Content Posting Automation** workflow reads files from your local storage, how Docker container volume mounts bridge your host machine with n8n, and how to configure file paths using environment variables.

---

## 1. Overview of File Ingestion Architecture

The workflow starts with the `Read/Write Files from Disk` node (`n8n-nodes-base.readWriteFile`), which scans a directory pattern and loads binary files into the n8n execution pipeline.

In the workflow JSON, the node parameter `fileSelector` is defined as:

```javascript
={{ $env.INPUT_FILES_PATH || '/path/to/your/media/*' }}
```

> [!IMPORTANT]
> The path `/path/to/your/media/*` is an **unconfigured placeholder fallback**. In a real deployment, n8n cannot read files from this path unless you explicitly configure the `INPUT_FILES_PATH` environment variable or edit the node's selector with your actual container-mounted directory.

---

## 2. How Docker Volume Mapping Works

When running n8n inside a Docker container, the container operates in its own isolated filesystem. 

To allow n8n to read files from your host machine (e.g., Windows `D:\SocialMediaQueue` or Linux `/opt/media`), you must mount your host folder to an internal container folder using a Docker volume:

```
+-------------------------------------------------------------+
| HOST MACHINE (Windows / Linux / macOS)                      |
| Host Path: D:\SocialMediaQueue  or  /opt/media             |
| Contains: clip1.mp4, banner.jpg, promo.png                 |
+------------------------------+------------------------------+
                               |
                   Docker Volume Mount (-v)
                               |
                               v
+-------------------------------------------------------------+
| N8N DOCKER CONTAINER                                        |
| Container Path: /files                                      |
| Accessible to n8n as: /files/clip1.mp4, /files/banner.jpg   |
+-------------------------------------------------------------+
```

---

## 3. Recommended Host Directory Structure

Create a dedicated directory on your host machine to store media files scheduled for posting:

```
D:\SocialMediaQueue\          <-- (Windows Host Example)
├── banner_launch.png
├── product_demo.mp4
├── tech_infographic.jpg
└── feature_overview.mov
```

### Supported Media Formats

The workflow uses MIME type inspection in the `Switch` and `Switch3` nodes:

| Media Type | MIME Type Condition | Confirmed File Extensions | Downstream Destination |
| :--- | :--- | :--- | :--- |
| **Video** | Starts with `video/` | `.mp4`, `.mov`, `.mkv`, `.avi` | Gemini Video API + Cloudinary + Instagram Reels + YouTube |
| **Image** | Starts with `image/` | `.jpg`, `.jpeg`, `.png`, `.webp` | Gemini Image API + Cloudinary + Instagram Photo Post |

> [!NOTE]
> Ensure filenames do not contain unusual special characters (such as `%`, `#`, `?`, or emojis) before placing them in the queue. The `Edit Fields` node sanitizes the filename by stripping the file extension:
> `{{ $binary.data.fileName.replace(/\.[^/.]+$/, '') }}`

---

## 4. Docker Run & Docker Compose Configuration

### Option A: Docker Run (Windows PowerShell)

```powershell
docker run -d `
  --name n8n `
  -p 5678:5678 `
  -v n8n_data:/home/node/.n8n `
  -v "D:/SocialMediaQueue:/files:ro" `
  -e INPUT_FILES_PATH="/files/*" `
  -e N8N_BLOCK_ENV_ACCESS_IN_NODE="false" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS="true" `
  docker.n8n.io/n8nio/n8n:latest
```

* `-v "D:/SocialMediaQueue:/files:ro"`: Mounts host folder `D:\SocialMediaQueue` to `/files` inside the container as **read-only** (`:ro`) for security.
* `-e INPUT_FILES_PATH="/files/*"`: Supplies the environment variable read by `Read/Write Files from Disk`.
* `-e N8N_BLOCK_ENV_ACCESS_IN_NODE="false"`: Enables n8n expressions to read `$env.INPUT_FILES_PATH`.

### Option B: Docker Run (Linux / macOS Terminal)

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -v "/opt/media/queue:/files:ro" \
  -e INPUT_FILES_PATH="/files/*" \
  -e N8N_BLOCK_ENV_ACCESS_IN_NODE="false" \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS="true" \
  docker.n8n.io/n8nio/n8n:latest
```

### Option C: Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-automation
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - GENERIC_TIMEZONE=Asia/Kolkata
      - TZ=Asia/Kolkata
      # Allow n8n expressions to read system env variables
      - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
      # Media queue path expression input
      - INPUT_FILES_PATH=/files/*
      # Instagram & Cloudinary placeholders (optional env overrides)
      - INSTAGRAM_ACCOUNT_ID=YOUR_INSTAGRAM_ACCOUNT_ID
      - CLOUDINARY_CLOUD_NAME=YOUR_CLOUDINARY_CLOUD_NAME
    volumes:
      - n8n_data:/home/node/.n8n
      # Host folder mapped to internal container directory
      - "D:/SocialMediaQueue:/files:ro" # Change for Linux: "/opt/media:/files:ro"

volumes:
  n8n_data:
```

---

## 5. Security & Permission Considerations

1. **Read-Only Volume Mount (`:ro`):**
   Because the workflow only *reads* files from disk during execution, always append `:ro` to the Docker volume mount. This prevents n8n or any third-party node from accidentally modifying or deleting original media files on your host machine.
2. **n8n v2.0+ File Access Security (`N8N_RESTRICT_FILE_ACCESS_TO`):**
   In modern n8n versions, accessing host files may require explicitly listing the directory in `N8N_RESTRICT_FILE_ACCESS_TO`. If n8n throws a file access error, add this environment variable:
   ```bash
   N8N_RESTRICT_FILE_ACCESS_TO=/files/
   ```
3. **Empty Folder Handling:**
   If the mounted directory is empty, the `Read/Write Files from Disk` node will output zero items, and downstream nodes will not execute. Place at least one sample media file in the folder before running a manual trigger test.
