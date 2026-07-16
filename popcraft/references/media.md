# Media — getting reference files into PopCraft

Reference media (images/videos/audio the generation should use) must live in
PopCraft-owned GCS storage before a generate tool can reference it. Three routes in,
one confirm step, one browse tool. All facts below are the live connector contract.

## Route decision

| The media is… | Use | Then |
|---|---|---|
| A public web URL (user pasted a link, search result, another tool's output) | `popcraft_media_import_url` | Returned `media_id` (a GCS https URL) goes straight into `medias[].value` |
| Already a PopCraft GCS URL (`gs://` or `storage.googleapis.com`, e.g. a prior generation's output) | Nothing — pass it through | `import_url` passes these through unchanged; never re-import |
| A local file, and the client renders widgets (claude.ai / desktop) | `popcraft_media_upload_widget` | Widget handles signed-URL, PUT, and confirm end-to-end; the confirmed `media_id` comes back as the next user message |
| A local file, no widget support (CLI-style host) | `popcraft_media_upload` → PUT bytes → `popcraft_media_confirm` | Confirm MUST succeed before using the `media_id` |

**Hard rule (from the connector):** chat attachments are NOT readable by the remote
server. When the user has a local file, go straight to the widget (or upload tool) —
never ask them to "attach it here".

## Tool contracts

### `popcraft_media_import_url`
- `url` — publicly fetchable http(s) URL. Private/internal addresses are blocked (SSRF protection) — if an import fails on a corporate/localhost link, ask for a public link instead.
- If a public URL fails with a 4xx, try the canonical/original asset URL before bouncing
  back to the user (e.g. a Wikimedia `…/thumb/…/640px-X.jpg` link 400s when the original
  is smaller than the requested thumb — import the original `…/commons/…/X.jpg` instead)
  and tell the user what you substituted.
- Returns a stable `media_id` (GCS https URL).

### `popcraft_media_upload` → PUT → `popcraft_media_confirm`
- `media_upload(filename)` → returns `upload_url` (signed, **expires in 15 minutes**) + `media_id`.
- PUT the raw bytes to `upload_url` with the returned headers.
- `media_confirm(media_id)` → verifies the object exists and is non-empty. **Skipping confirm and generating anyway is how you get a failed generation charged to a bad reference** — always confirm.

### `popcraft_media_upload_widget`
- Optional `type`: `"image" | "video" | "audio"` to restrict the picker (default auto), optional `label` (e.g. "Upload your product photo").
- Prefer this over raw upload whenever widgets render — the user drags a file, everything else is automatic.

### `popcraft_show_medias`
- Browse the user's saved media library (thumbnail grid widget on Apps hosts). Use when the user says "use the photo I uploaded before" — find it here instead of re-asking for the file.

## Using media in generations — `medias[].role`

Pass `medias: [{ value: <media_id/GCS URL>, role: <role> }]`:

| role | Tool | Meaning |
|---|---|---|
| `"start_image"` | generate_video | First frame. Alone = animate-this-image (image-to-video) |
| `"end_image"` | generate_video | Last frame. With `start_image` = first-to-last-frame video |
| `"image"` | generate_video | Style/subject reference (reference-to-video) |
| `"video"` | generate_video | Extend / continue / transform an existing clip — **model-dependent**; check the model's capabilities via `popcraft_models_explore` before offering |
| `"reference"` | generate_image | Condition the image on this reference (style / composition / subject transfer, i.e. "edit this image") |

The connector validates references against the model **before charging** (count, media
type, size, format) — a validation failure costs nothing. Known per-model reference
limits live in [video.md](video.md) § Reference limits.

## Workflow recipes

**"Animate this image" (URL):** `media_import_url(url)` → `generate_video` with
`medias:[{value, role:"start_image"}]` + a motion prompt (see video.md — write the
motion prompt yourself if the user gave none).

**"Edit this photo" (local file, Apps host):** `media_upload_widget(type:"image")` →
wait for the confirmed `media_id` message → `generate_image` with
`medias:[{value, role:"reference"}]` + an edit prompt.

**"Continue this clip":** the clip must be a PopCraft GCS URL (a prior result works
as-is; an external URL needs `import_url` first) → check the target model supports
video input (`models_explore`) → `generate_video` with `medias:[{value, role:"video"}]`.

**Chaining a prior result:** outputs of `popcraft_generate_*` are already GCS URLs —
pass them directly as `medias[].value` for the next step (image → video, video →
upscale, etc.). No import, no re-upload.
