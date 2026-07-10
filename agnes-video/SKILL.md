---
name: agnes-video
description: Generate videos using Agnes Video V2.0. Supports text-to-video, image-to-video, and keyframe animation. Use when the user asks to create or generate a video. Triggers on: 生成视频, 做视频, 制作视频, 创建视频, 文生视频, 图生视频.
---

# Agnes Video V2.0

Generate videos via the Agnes AI async API. Supports text-to-video, image-to-video, and keyframe animation.

**⚠️ Video generation is ASYNC — it takes minutes, not seconds. After creating a task, you must poll for the result.**

## API Configuration

| Field | Value |
|-------|-------|
| Create Endpoint | `POST https://apihub.agnes-ai.com/v1/videos` |
| Query Endpoint | `GET https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>` |
| Auth Header | `Authorization: Bearer sk-cPAXKxdqflavQHhX2ALG0pjXhppNv1qUbH6B9oNCiHQyW5XH` |
| Model | `agnes-video-v2.0` |

## Workflow

### Step 1: Gather parameters

Ask the user for:
- **prompt** (required): Describe the video content in English
- **mode** (optional): `text-to-video` (default), `image-to-video` (needs an image URL), or `keyframes` (needs 2+ image URLs)
- **width × height** (optional, default `1152×768`): Video resolution
- **duration** (optional, default ~5s): See duration table below

### Step 2: Create the video task

```bash
# Text-to-video (most common)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer YOUT-API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "<USER_PROMPT>",
    "width": 1152, "height": 768,
    "num_frames": 121, "frame_rate": 24
  }'

# Image-to-video (animate a still image)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer YOUT-API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "<USER_PROMPT>",
    "image": "<IMAGE_URL>",
    "num_frames": 121, "frame_rate": 24
  }'

# Keyframe animation (transition between images)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer YOUR-API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Generate a smooth cinematic transition between the keyframes, maintaining visual consistency",
    "num_frames": 121, "frame_rate": 24,
    "extra_body": {
      "image": ["<IMAGE_URL_1>", "<IMAGE_URL_2>"],
      "mode": "keyframes"
    }
  }'
```

The response will be:
```json
{
  "video_id": "video_xxx",
  "task_id": "task_xxx",
  "status": "queued",
  "progress": 0
}
```
**Save the `video_id`** — you need it for polling.

### Step 3: Poll until complete

Poll every 15-20 seconds using the `video_id`:

```bash
curl -s -X GET "https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>" \
  -H "Authorization: Bearer YOUT-API"
```

**Status values:**
| Status | Action |
|--------|--------|
| `queued` | Keep waiting, poll again in 20s |
| `in_progress` | Keep waiting, poll again in 20s |
| `completed` | **Done!** The `url` field has the video link |
| `failed` | Report the error to the user |

Tell the user the current progress each time you poll (e.g. "生成中... 45%"). Poll up to ~15 times (~5 minutes) before telling the user the task may take longer.

### Step 4: Present the result

When `status` is `completed`, the response includes:
```json
{
  "status": "completed",
  "progress": 100,
  "url": "https://platform-outputs.agnes-ai.space/videos/agnes-video-v2.0/video_xxxxxx.mp4",
  "seconds": "5.0",
  "size": "1152x768"
}
```

Show the user the video URL and tell them how long it is.

## Video Duration Control

```
seconds = num_frames / frame_rate
```

`num_frames` MUST be ≤ 441 AND follow the `8n + 1` rule (e.g., 81, 121, 241, 441).

| Target Duration | Parameters |
|----------------|------------|
| ~3 seconds | `num_frames: 81, frame_rate: 24` |
| ~5 seconds (default) | `num_frames: 121, frame_rate: 24` |
| ~10 seconds | `num_frames: 241, frame_rate: 24` |
| ~18 seconds | `num_frames: 441, frame_rate: 24` |

## Resolution Options

| Aspect Ratio | Resolutions | Best For |
|-------------|-------------|----------|
| 16:9 | `1152×768`, `1280×720`, `1920×1080` | Landscape, YouTube |
| 9:16 | `768×1152`, `720×1280` | Vertical, TikTok/Reels |
| 1:1 | `768×768`, `1024×1024` | Square, social feed |

## Parameters Reference

| Parameter | Required | Description |
|-----------|----------|-------------|
| `model` | Yes | Always `agnes-video-v2.0` |
| `prompt` | Yes | Text description of the video |
| `image` | No | Single image URL for image-to-video |
| `mode` | No | `ti2vid` or `keyframes` |
| `width` | No | Video width (default 1152) |
| `height` | No | Video height (default 768) |
| `num_frames` | No | Frame count, must be ≤441 and follow 8n+1 |
| `frame_rate` | No | FPS, range 1–60 (default 24) |
| `seed` | No | Integer for reproducible results |
| `negative_prompt` | No | What to avoid in the video |
| `extra_body.image` | No | Array of image URLs for keyframe mode |
| `extra_body.mode` | No | Set to `"keyframes"` for keyframe animation |

## Prompt Best Practices

- Write in English for best results
- Describe motion, camera movement, and scene dynamics — not just static visuals
- Use cinematic language: "cinematic shot", "slow pan", "tracking shot", "golden hour"
- Include atmosphere: lighting, mood, pace

## Important Rules

- Video generation is **async** — never treat it like a synchronous API
- Always poll with `video_id`, not `task_id` (video_id is the recommended way)
- The `num_frames` parameter MUST follow `8n + 1` (e.g. 81, 121, 241, 441)
- For keyframe mode, use `extra_body.mode: "keyframes"` with `extra_body.image` array
- Tell the user it'll take a few minutes — set expectations upfront
