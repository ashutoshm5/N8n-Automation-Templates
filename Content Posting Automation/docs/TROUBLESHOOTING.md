# Troubleshooting & Diagnostic Guide

This guide provides diagnoses, root causes, step-by-step troubleshooting, and recommended solutions for errors encountered when operating the **Content Posting Automation** workflow.

---

## 1. Credentials & Import Issues

### Issue 1.1: Red Triangles / Unlinked Credentials After Workflow Import
* **Symptoms**: Nodes such as `Analyze an image`, `Cloudinary - Upload Media `, or `Upload a video` display red warning badges indicating credentials are missing or unlinked.
* **Likely Cause**: Credential references in exported n8n workflow JSON files are internal database identifiers specific to the exporting instance. They do not transfer secrets across instances.
* **Troubleshooting Steps**:
  1. Open the affected node by double-clicking it on the canvas.
  2. Inspect the **Credential to connect with** dropdown.
  3. If marked red or empty, select your own configured credential.
* **Recommended Solution**: Set up your credentials manually as described in [API-CONFIGURATION.md](API-CONFIGURATION.md) and re-link each affected node once.

---

## 2. Media Ingestion & File System Issues

### Issue 2.1: `Read/Write Files from Disk` Returns 0 Items
* **Symptoms**: Triggering the workflow completes instantly with no downstream nodes executing.
* **Likely Cause**: 
  1. The mounted directory in Docker is empty.
  2. The `INPUT_FILES_PATH` environment variable is not defined or points to a path that does not exist inside the container.
  3. `N8N_BLOCK_ENV_ACCESS_IN_NODE` is set to `true`, preventing evaluation of `$env.INPUT_FILES_PATH`.
* **Troubleshooting Steps**:
  1. Run `docker exec -it n8n ls -la /files` to verify files exist inside the container.
  2. Check the node's `fileSelector` parameter. If it shows `/path/to/your/media/*`, the fallback was triggered.
* **Recommended Solution**:
  - Add `-e INPUT_FILES_PATH="/files/*"` and `-e N8N_BLOCK_ENV_ACCESS_IN_NODE="false"` to your Docker container run command or `docker-compose.yml`.
  - Alternatively, change the `fileSelector` expression directly to fixed text: `/files/*`.

### Issue 2.2: Permission Denied / EACCES When Reading Files
* **Symptoms**: Node fails with `EACCES: permission denied, open '/files/sample.mp4'`.
* **Likely Cause**: The n8n container user (`node`, UID 1000) does not have read permissions on the mounted host directory.
* **Troubleshooting Steps**:
  1. Check permissions of your host directory.
  2. In Linux: `ls -ld /opt/media`
* **Recommended Solution**:
  - On Linux: Grant read permissions: `chmod -R o+r /opt/media`.
  - In Docker Compose: Mount as read-only: `/opt/media:/files:ro`.
  - On Windows: Ensure Docker Desktop has permission to access the host drive (Settings -> Resources -> File sharing).

---

## 3. Google Gemini AI Engine Issues

### Issue 3.1: `Gemini response text was not found` in `Result Enhancer`
* **Symptoms**: The workflow halts at node `Result Enhancer` or `Result Enhancer1` with error: `Gemini response text was not found.`
* **Likely Cause**: Gemini returned an empty response, was blocked by safety filters, or the rate limit was hit.
* **Troubleshooting Steps**:
  1. Click on the preceding node (`Message a model` or `Message a model1`).
  2. Check the Output pane for `error` objects or `candidates[0].finishReason`.
  3. If `finishReason: "SAFETY"`, the input media triggered Gemini's safety guidelines.
* **Recommended Solution**:
  - Ensure media does not violate Google AI content policies.
  - Review prompt temperature or safety settings.

### Issue 3.2: `Gemini returned invalid JSON` in `Result Enhancer1`
* **Symptoms**: Error message: `Gemini returned invalid JSON: ...`
* **Likely Cause**: The model generated natural language explanations or incomplete JSON outside the requested format.
* **Troubleshooting Steps**:
  1. Inspect the raw text output in `Message a model1`.
  2. Verify if the JSON string is truncated due to maximum output token limits.
* **Recommended Solution**:
  - The node `Message a model1` has `jsonOutput: true` enabled. Ensure the model selected is `models/gemini-3.7-flash` or `models/gemini-2.0-flash`, which support strict JSON mode.

### Issue 3.3: `Upload a media file` Fails with Rate Limit (429)
* **Symptoms**: Error: `429 Resource Exhausted`.
* **Likely Cause**: Free tier Google AI Studio accounts have strict requests-per-minute limits for video file ingestion.
* **Troubleshooting Steps**:
  1. Node `Analyze video` is already configured with `retryOnFail: true`, `maxTries: 5`, and `waitBetweenTries: 200`.
  2. If retries are exhausted, consecutive video runs are triggering rate limits.
* **Recommended Solution**: Add a `Wait` node before video analysis or upgrade to a pay-as-you-go Gemini API project in Google AI Studio.

---

## 4. Cloudinary Upload Issues

### Issue 4.1: Cloudinary Returns 401 Unauthorized
* **Symptoms**: Node `Cloudinary - Upload Media ` throws `401 Unauthorized` with message `Invalid Signature` or `Must supply api_key`.
* **Likely Cause**: The HTTP Basic Auth credential is misconfigured.
* **Troubleshooting Steps**:
  1. Verify the `Cloudinary API` credential in n8n.
  2. Ensure **User** = Cloudinary **API Key** (numeric string).
  3. Ensure **Password** = Cloudinary **API Secret** (alphanumeric string).
* **Recommended Solution**: Re-copy the API Key and API Secret directly from the Cloudinary Console dashboard.

### Issue 4.2: Cloudinary URL Contains `YOUR_CLOUDINARY_CLOUD_NAME`
* **Symptoms**: Error: `404 Not Found` or `Resource not found` when uploading.
* **Likely Cause**: The node URL still contains the placeholder `YOUR_CLOUDINARY_CLOUD_NAME`.
* **Recommended Solution**: Open node `Cloudinary - Upload Media ` and replace the placeholder with your actual Cloudinary Cloud Name in the URL:
  `https://api.cloudinary.com/v1_1/<YOUR_CLOUD_NAME>/auto/upload`

---

## 5. Meta Instagram Graph API Issues

### Issue 5.1: `Video Content Merge` Fails: `The video file could not be downloaded from the provided URL`
* **Symptoms**: Instagram API returns error code `2207001` or `2207008` (OAuthException): `The video file could not be downloaded from the provided URL.`
* **Likely Cause**: 
  1. Cloudinary upload has not finished processing/encoding the video before Instagram attempted to fetch it.
  2. The `secure_url` is not publicly accessible over HTTPS.
* **Troubleshooting Steps**:
  1. Copy the `secure_url` output from `Cloudinary - Upload Media ` and open it in an incognito browser tab.
  2. If it downloads or plays immediately, the file is public.
* **Recommended Solution**:
  - For larger videos, add a 10–20 second `Wait` node between `Switch1` and `Video Merge` to allow Cloudinary encoding to finalize before Instagram fetches it.

### Issue 5.2: `Instagram Image Post ` Fails: `Media ID is not ready`
* **Symptoms**: Node `Instagram Image Post ` or `Instagram Reel Post ` throws error: `Media container is not ready for publishing`.
* **Likely Cause**: Video containers (Reels) require processing time on Meta's servers before they transition to status `FINISHED`.
* **Troubleshooting Steps**:
  1. Inspect the container creation node response (`Video Content Merge`). It returns an `id`.
  2. Query container status:
     ```http
     GET https://graph.facebook.com/v26.0/<CONTAINER_ID>?fields=status_code&access_token=<TOKEN>
     ```
* **Recommended Solution**: If Reels frequently fail with processing errors, insert a `Wait` node (configured for 30–60 seconds) between `Video Content Merge` and `Instagram Reel Post `.

### Issue 5.3: `190 Invalid OAuth 2.0 Access Token`
* **Symptoms**: Meta returns error `Error validating access token: Session has expired`.
* **Likely Cause**: You are using a short-lived user access token (valid for 1–2 hours) that has expired.
* **Recommended Solution**: Generate a **Long-Lived Page Access Token** (valid for 60 days) or create a **System User Token** in Meta Business Manager with permanent validity.

---

## 6. YouTube Data API v3 Issues

### Issue 6.1: `Upload a video` Fails: `Tags must be an array or string`
* **Symptoms**: Upload node fails with invalid tags format.
* **Likely Cause**: The AI output for tags in `Message a model1` did not return an array, or array joining failed.
* **Verified Configuration**: The node expression is:
  ```javascript
  ={{ $json.youtube.tags.join(', ') }}
  ```
* **Recommended Solution**: The JavaScript validator in `Result Enhancer1` verifies that `result.youtube.tags` is present. If tags are occasionally returned as a plain string, update the expression to:
  ```javascript
  ={{ Array.isArray($json.youtube.tags) ? $json.youtube.tags.join(', ') : $json.youtube.tags }}
  ```

### Issue 6.2: `Upload a video` Fails: `The user has exceeded the upload limit`
* **Symptoms**: Error `403 quotaExceeded` or `uploadLimitExceeded`.
* **Likely Cause**:
  1. Unverified YouTube channels are subject to daily upload limits (typically 5–10 videos per day).
  2. Google Cloud project default YouTube Data API quota is 10,000 units/day. A single video upload consumes approximately 1,600 units.
* **Recommended Solution**: Verify your YouTube channel via phone number in YouTube Studio to raise limits, or request an API quota increase in the Google Cloud Console.

---

## 7. Merge Node Key Mismatch Issues

### Issue 7.1: `Video Merge` or `Image Merge` Outputs 0 Items
* **Symptoms**: Both branches run, but the Merge node produces no items and halts publishing.
* **Likely Cause**: The join keys do not match.
  - `field1` expects `original_filename` from Cloudinary.
  - `field2` expects `originalFileName` from `Result Enhancer` / `Result Enhancer1`.
* **Troubleshooting Steps**:
  1. Inspect the JSON from `Switch1`: check if `original_filename` exists.
  2. Inspect the JSON from `Result Enhancer`: check if `originalFileName` exists.
* **Recommended Solution**: Cloudinary always returns `original_filename` matching the sanitized filename produced by `Edit Fields`. If Cloudinary sanitized special characters differently, verify that the input filename does not contain spaces or non-ASCII symbols.
