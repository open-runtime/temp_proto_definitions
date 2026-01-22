# Stability AI Models on AWS Bedrock

## Overview

Stability AI offers a comprehensive suite of image generation and manipulation models on AWS Bedrock, ranging from text-to-image generation to advanced editing operations like inpainting, outpainting, and upscaling.

## Available Models

### Text-to-Image Generation

| Model ID | Name | Description | Input |
|----------|------|-------------|-------|
| stability.stable-image-core-v1:1 | Stable Image Core | Fast, general-purpose generation | TEXT |
| stability.stable-image-ultra-v1:1 | Stable Image Ultra | Highest quality generation | TEXT |
| stability.sd3-5-large-v1:0 | SD 3.5 Large | Stable Diffusion 3.5 flagship | TEXT+IMAGE |

### Image Editing/Manipulation

| Model ID | Name | Operation | Input |
|----------|------|-----------|-------|
| stability.stable-image-inpaint-v1:0 | Inpaint | Fill masked regions | TEXT+IMAGE |
| stability.stable-outpaint-v1:0 | Outpaint | Extend image boundaries | TEXT+IMAGE |
| stability.stable-image-erase-object-v1:0 | Erase Object | Remove objects from image | TEXT+IMAGE |
| stability.stable-image-search-replace-v1:0 | Search & Replace | Replace specific objects | TEXT+IMAGE |
| stability.stable-image-search-recolor-v1:0 | Search & Recolor | Recolor specific objects | TEXT+IMAGE |

### Style Transfer & Control

| Model ID | Name | Operation | Input |
|----------|------|-----------|-------|
| stability.stable-style-transfer-v1:0 | Style Transfer | Apply artistic styles | TEXT+IMAGE |
| stability.stable-image-style-guide-v1:0 | Style Guide | Guided style application | TEXT+IMAGE |
| stability.stable-image-control-sketch-v1:0 | Control Sketch | Generate from sketches | TEXT+IMAGE |
| stability.stable-image-control-structure-v1:0 | Control Structure | Maintain structural integrity | TEXT+IMAGE |

### Upscaling

| Model ID | Name | Quality | Input |
|----------|------|---------|-------|
| stability.stable-fast-upscale-v1:0 | Fast Upscale | Speed-optimized | TEXT+IMAGE |
| stability.stable-conservative-upscale-v1:0 | Conservative Upscale | Detail-preserving | TEXT+IMAGE |
| stability.stable-creative-upscale-v1:0 | Creative Upscale | AI-enhanced details | TEXT+IMAGE |

### Utility

| Model ID | Name | Operation | Input |
|----------|------|-----------|-------|
| stability.stable-image-remove-background-v1:0 | Remove Background | Transparent background | TEXT+IMAGE |

## Key Features

- **Multiple Aspect Ratios**: Support for 1:1, 16:9, 9:16, 4:3, 3:4, etc.
- **High Resolution**: Up to 2048px depending on model
- **Seed Control**: Reproducible generations with seed values
- **Multiple Output Formats**: PNG, JPEG, WEBP

## Request Format

### Text-to-Image (Core/Ultra)

```json
{
  "prompt": "Your detailed prompt here",
  "mode": "text-to-image",
  "output_format": "png",
  "aspect_ratio": "16:9"
}
```

### Image-to-Image (SD 3.5)

```json
{
  "prompt": "Modified scene description",
  "mode": "image-to-image",
  "image": "<base64-encoded-image>",
  "output_format": "png",
  "strength": 0.7
}
```

### Inpaint

```json
{
  "prompt": "What to generate in masked area",
  "image": "<base64-encoded-image>",
  "mask": "<base64-encoded-mask>",
  "output_format": "png"
}
```

### Upscale

```json
{
  "image": "<base64-encoded-image>",
  "output_format": "png"
}
```

## Response Format

All models return a consistent format:

```json
{
  "seeds": [336988488],
  "finish_reasons": [null],
  "images": ["<base64-encoded-image>"]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `seeds` | number[] | Random seeds used for generation |
| `finish_reasons` | (string\|null)[] | Completion status per image |
| `images` | string[] | Base64-encoded output images |

### Finish Reasons

- `null`: Successful completion
- `CONTENT_FILTERED`: Blocked by safety filters
- `ERROR`: Generation error occurred

## API Notes

### InvokeModel Only

Stability AI models use **InvokeModel** API only (not Converse):

```bash
aws bedrock-runtime invoke-model \
  --model-id "stability.stable-image-core-v1:1" \
  --body "$BASE64_BODY" \
  --content-type "application/json" \
  --accept "application/json" \
  /path/to/output.json
```

### Inference Profile Requirements

Some models require inference profiles for on-demand use:
- `stability.stable-fast-upscale-v1:0`
- `stability.stable-conservative-upscale-v1:0`
- `stability.stable-creative-upscale-v1:0`

## Best Practices

1. **Prompt Engineering**: Be specific and detailed in prompts
2. **Aspect Ratio**: Choose ratio appropriate for use case
3. **Seed for Reproducibility**: Save seeds for variations
4. **Output Format**: Use PNG for quality, JPEG for size
5. **Batch Processing**: Process multiple images in parallel

## Supported Aspect Ratios

| Ratio | Description |
|-------|-------------|
| 1:1 | Square (default) |
| 16:9 | Widescreen landscape |
| 9:16 | Vertical/Portrait |
| 4:3 | Standard landscape |
| 3:4 | Standard portrait |
| 21:9 | Ultrawide |
| 9:21 | Ultra-tall |

## Sample Files

| File | Description |
|------|-------------|
| `stability_core_text_to_image_request.json` | Core model request |
| `stability_core_text_to_image_response.json` | Core model response |
| `stability_sd35_text_to_image_request.json` | SD 3.5 Large request |
| `stability_sd35_text_to_image_response.json` | SD 3.5 Large response |
| `stability_ultra_text_to_image_request.json` | Ultra model request |
| `stability_ultra_text_to_image_response.json` | Ultra model response |

## References

- [AWS Bedrock Stability AI Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-stability-diffusion.html)
- [Stability AI Platform Documentation](https://platform.stability.ai/docs)
- [Stable Diffusion 3.5 Model Card](https://stability.ai/news/introducing-stable-diffusion-3-5)
