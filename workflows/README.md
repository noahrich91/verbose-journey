# n8n workflows

## `channel-b-publish.json`: Channel B publish

This workflow watches a Google Drive folder for finished `.mp4` exports. For each one it uploads the video to YouTube as **private**, then emails a YouTube Studio link so you can add a thumbnail and schedule it.

### Flow

1. **New Finished Video**: Drive trigger. It checks the finished-videos folder every minute for new files.
2. **Is MP4**: lets only `*.mp4` files through.
3. **Derive Run ID**: takes the file name without `.mp4` as the `run_id`. For example, `ep042.mp4` gives `ep042`.
4. **Find Package** and **Download Package**: find `<run_id>-package.json` in the edit-kit folder and download it.
5. **Parse Package**: reads `title`, `description` and `tags` from the package JSON.
6. **Download Video**: downloads the `.mp4`.
7. **Upload to YouTube (Private)**: uploads with category 27 (Education), region US, language `en`, and not made for kids.
8. **Email Upload Link**: sends a Gmail message with the YouTube Studio edit link.

### Package file format

```json
{
  "title": "Video title",
  "description": "Video description",
  "tags": ["tag one", "tag two"]
}
```

### Setup

1. In n8n, go to **Workflows → Import from file** and pick `channel-b-publish.json`.
2. Replace these placeholders:
   - `REPLACE_WITH_FINISHED_VIDEOS_FOLDER_ID` in **New Finished Video**
   - `REPLACE_WITH_EDIT_KIT_FOLDER_ID` in **Derive Run ID**
   - `REPLACE_WITH_YOUR_EMAIL` in **Derive Run ID**
3. Re-select the Google Drive, YouTube and Gmail credentials on each node. The credential IDs in the file are placeholders.

### Notes

- If no matching `-package.json` exists, **Find Package** returns no items and the run stops quietly without an error. Upload the package to the edit-kit folder **before** the `.mp4` lands in the finished folder.
- The Drive trigger polls, so one upload can produce more than one item. All downstream nodes read the first item via `.first()`, so each run handles one video.
