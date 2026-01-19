# Image Generation Provider APIs

Reference documentation for supported image generation providers.

## SVG Generation (Default)

SVG is the default provider. Claude generates clean vector graphics directly - no external API needed.

### Requirements
- None (built into Claude)

### Output Format
- Format: SVG (Scalable Vector Graphics)
- Dimensions: 1280x640 viewBox
- File size: Typically 10-50KB

### SVG Styles

| Style | Description | Shapes | Best For |
|-------|-------------|--------|----------|
| `minimal` | Clean, focused | 3-5 | Professional, understated |
| `geometric` | Complex patterns | 8-15 | Technical projects |
| `illustrated` | Organic, warm | Varies | Approachable projects |

### Template Structure

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1280 640">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#0f172a"/>
      <stop offset="100%" style="stop-color:#1e293b"/>
    </linearGradient>
  </defs>
  <rect width="1280" height="640" fill="url(#bg)"/>
  <!-- Domain-specific shapes -->
  <text x="640" y="340" text-anchor="middle" fill="#f8fafc"
        font-family="-apple-system, sans-serif" font-size="72" font-weight="700">
    Project Name
  </text>
</svg>
```

### Benefits
- **Free**: No API costs
- **Instant**: No network latency
- **Editable**: Text-based, easily modified
- **Small**: 10-50KB file size
- **Consistent**: Predictable results

---

## Gemini (Google)

Google's Gemini API provides native image generation (codename: Nano Banana).

### Requirements
- Environment variable: `GEMINI_API_KEY`
- API endpoint: `https://generativelanguage.googleapis.com/v1beta/models/`

### Models

| Model | Description | Best For |
|-------|-------------|----------|
| `gemini-2.5-flash-image` | Fast, efficient | High-volume, quick iterations |
| `gemini-3-pro-image-preview` | Higher quality with reasoning | Final production, text rendering |

### API Call

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "[YOUR_PROMPT]"}]
    }],
    "generationConfig": {
      "responseModalities": ["IMAGE"],
      "imageConfig": {
        "aspectRatio": "16:9"
      }
    }
  }'
```

### Response Processing

```bash
# Extract base64 image data from response
IMAGE_DATA=$(echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].inlineData.data')

# Decode and save
echo "$IMAGE_DATA" | base64 -d > .github/social-preview.png

# Resize to exact GitHub dimensions if needed
convert .github/social-preview.png -resize 1280x640! .github/social-preview.png
```

### Error Handling

Common errors:
- `401`: Invalid API key
- `429`: Rate limit exceeded (free tier has limits)
- `400`: Invalid prompt or unsupported content

### Cost
- **gemini-2.5-flash-image**: ~$0.039 per image (1290 output tokens at $30/1M tokens)
- Free tier available with rate limits

### Getting an API Key

1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Create or select a project
3. Generate an API key
4. Set environment variable: `export GEMINI_API_KEY="your-key"`

---

## DALL-E 3 (OpenAI)

### Requirements
- Environment variable: `OPENAI_API_KEY`
- API endpoint: `https://api.openai.com/v1/images/generations`

### API Call

```bash
curl -s https://api.openai.com/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "dall-e-3",
    "prompt": "[YOUR_PROMPT]",
    "n": 1,
    "size": "1792x1024",
    "quality": "hd",
    "response_format": "b64_json"
  }'
```

### Size Options
- `1024x1024` - Square (not recommended for social preview)
- `1792x1024` - Landscape (closest to 2:1, recommended)
- `1024x1792` - Portrait (not recommended)

### Response Format
- `url` - Returns temporary URL (expires in 1 hour)
- `b64_json` - Returns base64-encoded image (recommended for saving)

### Post-Processing

DALL-E 3 outputs 1792x1024. Resize to exact GitHub requirements:

```bash
# Extract base64 from response and decode
echo "$BASE64_DATA" | base64 -d > temp_dalle.png

# Resize to exact GitHub dimensions (1280x640)
# Using ImageMagick:
convert temp_dalle.png -resize 1280x640! -quality 95 .github/social-preview.png

# Using ffmpeg:
ffmpeg -i temp_dalle.png -vf "scale=1280:640:force_original_aspect_ratio=decrease,pad=1280:640:(ow-iw)/2:(oh-ih)/2" .github/social-preview.png

# Using sips (macOS):
sips -z 640 1280 temp_dalle.png --out .github/social-preview.png
```

### Error Handling

Common errors:
- `401`: Invalid API key
- `429`: Rate limit exceeded
- `400`: Invalid prompt (content policy violation)

### Cost
- Standard quality: ~$0.040 per image
- HD quality: ~$0.080 per image

## Stable Diffusion (Local/API)

### Local Installation (Automatic1111)

If user has local SD installation:

```bash
# API endpoint (default)
curl -X POST http://127.0.0.1:7860/sdapi/v1/txt2img \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[YOUR_PROMPT]",
    "negative_prompt": "[NEGATIVE_PROMPT]",
    "width": 1280,
    "height": 640,
    "steps": 30,
    "cfg_scale": 7.5,
    "sampler_name": "DPM++ 2M Karras"
  }'
```

### Stability AI API

```bash
curl -X POST https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $STABILITY_API_KEY" \
  -d '{
    "text_prompts": [
      {"text": "[YOUR_PROMPT]", "weight": 1},
      {"text": "[NEGATIVE_PROMPT]", "weight": -1}
    ],
    "cfg_scale": 7,
    "width": 1344,
    "height": 768,
    "samples": 1,
    "steps": 30
  }'
```

### Size Constraints
SDXL requires dimensions divisible by 64. Closest to 1280x640:
- `1344x704` - Slightly larger, crop to 1280x640
- `1280x640` - Exact (if supported by model)

### Recommended Models
- SDXL 1.0 - Best quality for abstract/artistic
- Juggernaut XL - Good for detailed compositions
- DreamShaper XL - Artistic style flexibility

## Replicate API

For cloud-hosted Stable Diffusion:

```bash
curl -s -X POST https://api.replicate.com/v1/predictions \
  -H "Authorization: Token $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "version": "[SDXL_VERSION_HASH]",
    "input": {
      "prompt": "[YOUR_PROMPT]",
      "negative_prompt": "[NEGATIVE_PROMPT]",
      "width": 1344,
      "height": 768,
      "num_outputs": 1
    }
  }'
```

> **Note**: Get the current SDXL version hash from [Replicate's SDXL page](https://replicate.com/stability-ai/sdxl). Version hashes change when models are updated.

### Polling for Result

Replicate is async. Poll the returned URL:

```bash
# Initial response contains prediction URL
PREDICTION_URL=$(echo $RESPONSE | jq -r '.urls.get')

# Poll until complete
while true; do
  STATUS=$(curl -s -H "Authorization: Token $REPLICATE_API_TOKEN" $PREDICTION_URL | jq -r '.status')
  if [ "$STATUS" = "succeeded" ]; then
    IMAGE_URL=$(curl -s -H "Authorization: Token $REPLICATE_API_TOKEN" $PREDICTION_URL | jq -r '.output[0]')
    curl -s "$IMAGE_URL" -o .github/social-preview.png
    break
  elif [ "$STATUS" = "failed" ]; then
    echo "Generation failed"
    break
  fi
  sleep 2
done
```

## Midjourney (Manual Process)

Midjourney doesn't have a public API. Output optimized prompt for manual use.

### Prompt Format

```
[YOUR_PROMPT] --ar 2:1 --q 2 --stylize 500 --no text watermark
```

### Parameters
- `--ar 2:1` - Aspect ratio matching GitHub requirements
- `--q 2` - High quality mode
- `--stylize 500` - Balanced stylization (100-1000 range)
- `--no` - Negative prompt elements

### User Instructions

When outputting for Midjourney:
1. Display the complete prompt with parameters
2. Instruct user to paste in Midjourney Discord
3. Recommend using `/imagine` command
4. Suggest upscaling the result (U1-U4)
5. Note to download at highest resolution

## Image Optimization

After generation, ensure file meets GitHub requirements:

### Check Dimensions
```bash
# Using ImageMagick
identify -format "%wx%h" image.png

# Using sips (macOS)
sips -g pixelWidth -g pixelHeight image.png

# Using file
file image.png
```

### Check File Size
```bash
# Get size in bytes
stat -f%z image.png  # macOS
stat -c%s image.png  # Linux

# Human readable
ls -lh image.png
```

### Compress if Over 1MB

```bash
# PNG optimization (lossless)
pngquant --quality=80-95 --output optimized.png image.png

# Convert to optimized JPEG (lossy but smaller)
convert image.png -quality 85 image.jpg

# Progressive compression
optipng -o5 image.png
```

### Resize if Needed

```bash
# Exact resize (may distort)
convert image.png -resize 1280x640! output.png

# Resize maintaining aspect, then crop
convert image.png -resize 1280x640^ -gravity center -extent 1280x640 output.png

# Resize with padding
convert image.png -resize 1280x640 -gravity center -background "#0d1117" -extent 1280x640 output.png
```

## Environment Variable Setup

### For Users

Document required environment variables:

```bash
# DALL-E 3
export OPENAI_API_KEY="sk-..."

# Stability AI
export STABILITY_API_KEY="sk-..."

# Replicate
export REPLICATE_API_TOKEN="r8_..."
```

### Checking Availability

```bash
# Check if key is set
if [ -z "$OPENAI_API_KEY" ]; then
  echo "OPENAI_API_KEY not set"
fi
```

> **Note**: OpenAI API keys may have different prefixes depending on key type and creation date. Validate by making a test API call rather than checking prefix format.

## Provider Comparison

| Provider | Quality | Speed | Cost | Setup |
|----------|---------|-------|------|-------|
| **SVG** (default) | Professional | Instant | Free | None |
| **Gemini** | Very Good | Fast | ~$0.039/img | API key only |
| **DALL-E 3** | Excellent | Fast | ~$0.08/img | API key only |
| SDXL (local) | Very Good | Varies | Free | Local install |
| Stability API | Very Good | Fast | ~$0.02/img | API key only |
| Replicate | Good | Medium | ~$0.01/img | API key only |
| Midjourney | Excellent | Manual | $10-60/mo | Discord sub |

## Fallback Strategy

If configured provider fails:

1. **SVG (default)**: Always available, no external dependencies
2. **DALL-E 3 / Gemini**: Check if API key environment variable is set
3. If API key not set, fall back to SVG generation
4. If API call fails, capture error and fall back to SVG generation
5. If SVG also fails, output prompt for manual use with Midjourney or other tools
