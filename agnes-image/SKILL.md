---
name: agnes-image
description: Generate images using Agnes Image 2.1 Flash. Supports text-to-image and image-to-image. Use when the user asks to create, generate, or draw an image. Triggers on: 生成图片, 画图, 做图, 生成图像, 文生图, 图生图.
---

# Agnes Image 2.1 Flash

Generate images via the Agnes AI API. Supports text-to-image and image-to-image modes.

## API Configuration

| Field | Value |
|-------|-------|
| Endpoint | `https://apihub.agnes-ai.com/v1/images/generations` |
| Auth Header | `Authorization: Bearer sk-cPAXKxdqflavQHhX2ALG0pjXhppNv1qUbH6B9oNCiHQyW5XH` |
| Model | `agnes-image-2.1-flash` |
| Method | POST |

## Workflow

When the user asks to generate an image:

### 1. Gather required parameters

Ask the user if they haven't provided enough info:
- **prompt** (required): What to generate — be specific and descriptive
- **size** (optional, default `1024x768`): Output dimensions
- **reference image** (optional): URL of an existing image to transform (for img2img mode)

### 2. Generate the image

Run this curl command:

**Text-to-image (URL output — preferred):**
```bash
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer YOUR-API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "<USER_PROMPT>",
    "size": "<SIZE>",
    "extra_body": { "response_format": "url" }
  }'
```

**Image-to-image (transform an existing image):**
```bash
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer YOUT-API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "<USER_PROMPT>",
    "size": "<SIZE>",
    "extra_body": {
      "image": ["<IMAGE_URL>"],
      "response_format": "url"
    }
  }'
```

### 3. Present the result

Parse the response JSON. The generated image URL is at `data[0].url`. Show the URL to the user and tell them to open it in a browser.

Response format:
```json
{
  "created": 1780000000,
  "data": [{ "url": "https://storage.googleapis.com/agnes-aigc/xxx.png", ... }]
}
```

## Prompt Best Practices

- Write in English for best results
- Be specific about subject, setting, lighting, style, and mood
- Include technical quality descriptors: "cinematic lighting", "high detail", "4K", "photorealistic"
- For img2img: describe the transformation clearly, mention "preserving the original composition"

## Parameters Reference

| Parameter | Required | Description |
|-----------|----------|-------------|
| `model` | Yes | Always `agnes-image-2.1-flash` |
| `prompt` | Yes | Text description of the desired image |
| `size` | Yes | Output size, e.g. `1024x768`, `768x1024`, `1024x1024` |
| `image` | No (img2img) | Array of public image URLs for image-to-image |
| `return_base64` | No | Set `true` for Base64 output (text-to-image only) |
| `extra_body.response_format` | No | `"url"` (default) or `"b64_json"` |

## Important Rules

- **NEVER** put `response_format` at the top level of the request body — always inside `extra_body`
- **NEVER** pass `tags: ["img2img"]` — it's not needed
- For img2img, put the image URL inside `extra_body.image` array
- Image input URLs must be publicly accessible
