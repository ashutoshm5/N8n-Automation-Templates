# Pre-Deployment & Production Readiness Checklist

Before activating or deploying this workflow for automated social media publishing, complete the following verification checklists to ensure reliability, security, and data integrity.

---

## 1. Infrastructure & Container Environment

- [ ] **Docker Engine / Desktop Running**: Docker is active and running with WSL2 backend (Windows) or native daemon (Linux).
- [ ] **n8n Container Accessible**: The n8n dashboard is reachable at `http://localhost:5678` (or your domain).
- [ ] **Persistent Volume Mounted**: Host folder is mapped to `/home/node/.n8n` to prevent database loss on container recreation.
- [ ] **Media Queue Mounted Read-Only**: Host media folder is mounted to `/files:ro` inside the container.
- [ ] **Environment Variables Loaded**:
  - [ ] `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` (required for `$env` expressions).
  - [ ] `INPUT_FILES_PATH=/files/*` (or explicit container path).
  - [ ] `GENERIC_TIMEZONE` and `TZ` configured for your locale.

---

## 2. API Credentials Verification

- [ ] **Google Gemini (`googlePalmApi`)**:
  - [ ] API Key generated in Google AI Studio.
  - [ ] Credential created in n8n and linked to all 5 Gemini nodes.
  - [ ] Quota checked (ensure account is not rate-limited).
- [ ] **Cloudinary (`httpBasicAuth`)**:
  - [ ] Cloud Name noted.
  - [ ] API Key (User) and API Secret (Password) paired in n8n.
  - [ ] Node `Cloudinary - Upload Media ` URL updated with actual Cloud Name (no `YOUR_CLOUDINARY_CLOUD_NAME`).
- [ ] **Meta Instagram Graph API (`httpHeaderAuth`)**:
  - [ ] Facebook Page connected to an Instagram Professional (Business/Creator) account.
  - [ ] Long-lived or System User token generated with `instagram_content_publish` permission.
  - [ ] Header Auth credential configured (`Authorization: Bearer <TOKEN>`).
  - [ ] 17-digit Instagram Business Account ID replaced in all 4 Instagram node URLs (no `YOUR_INSTAGRAM_ACCOUNT_ID`).
- [ ] **YouTube Data API v3 (`youTubeOAuth2Api`)**:
  - [ ] Google Cloud project created with YouTube Data API v3 enabled.
  - [ ] OAuth consent screen configured with YouTube upload scope.
  - [ ] OAuth Client ID & Secret connected via OAuth2 flow in n8n.
  - [ ] Status confirmed as "Connected".

---

## 3. Workflow Logic & Safety Sanity Check

- [ ] **YouTube Upload Privacy**: Node `Upload a video` parameter `options.privacyStatus` is verified as `"private"` for initial staging tests.
- [ ] **Brand Customization**: Node `Message a model` prompt updated from `[Your Brand]` to your actual company/creator name.
- [ ] **Merge Node Keys Verified**: Both `Image Merge` and `Video Merge` nodes verify `field1: "original_filename"` matches `field2: "originalFileName"`.
- [ ] **MIME Routing Checked**: Ensure test filenames do not contain spaces or unusual characters that alter MIME detection.

---

## 4. End-to-End Staging Execution Tests

### Test 1: Image Processing & Instagram Photo Post
1. Place **one** test image (`.jpg` or `.png`) into your media queue folder.
2. Trigger the workflow manually.
3. Verify:
   - [ ] `Read/Write Files from Disk` loads 1 item.
   - [ ] `Switch` routes item to `Analyze an image`.
   - [ ] `Message a model` produces structured JSON (`instagram`, `linkedin`, `youtube`).
   - [ ] `Result Enhancer` parses JSON and attaches `originalFileName`.
   - [ ] `Cloudinary - Upload Media ` successfully returns a public `secure_url`.
   - [ ] `Image Merge` merges metadata without dropping items.
   - [ ] `Image Content Merge` creates container ID.
   - [ ] `Instagram Image Post ` publishes successfully.
   - [ ] Verify test image appears on Instagram profile.

### Test 2: Video Processing, Instagram Reels & YouTube Upload
1. Remove the test image and place **one** short test video (`.mp4`, 5–15 seconds) into your queue folder.
2. Trigger the workflow manually.
3. Verify:
   - [ ] `Switch` routes item to `Upload a media file`.
   - [ ] Gemini File API upload succeeds and returns `fileUri`.
   - [ ] `Analyze video` generates scene/dialogue inspection.
   - [ ] `Message a model1` creates structured copy.
   - [ ] `Result Enhancer1` sanitizes markdown and validates schema.
   - [ ] `Cloudinary - Upload Media ` uploads video and returns `secure_url`.
   - [ ] `Video Merge` merges data for Instagram.
   - [ ] `Video Content Merge` creates Reels container.
   - [ ] `Instagram Reel Post ` publishes Reel.
   - [ ] `Switch3` routes binary video to `Video Merge1`.
   - [ ] `Video Merge1` pairs AI metadata with raw local binary.
   - [ ] `Upload a video` uploads to YouTube Data API v3.
   - [ ] Verify video appears in **YouTube Studio -> Content -> Private Videos**.

---

## 5. Security & GitHub Sharing Audit

Before exporting, committing, or sharing any workflow JSON:
- [ ] **No Hardcoded Tokens**: Verify no tokens or keys in headers or URLs.
- [ ] **No Server Instance ID**: Verify `instanceId` is absent from `meta`.
- [ ] **No Local Account IDs**: Verify `178414...` and Cloudinary cloud names are replaced with placeholders.
- [ ] **No Local Paths**: Verify `fileSelector` uses `{{ $env.INPUT_FILES_PATH || ... }}`.
- [ ] **.gitignore Configured**: Confirm `.env` and local media files are ignored.
