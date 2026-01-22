# Amazon Nova Models on AWS Bedrock

## Overview

Amazon Nova is Amazon's family of foundation models spanning understanding (text/multimodal), generation (image/video), speech, and embeddings. All models are first-party Amazon models with native Bedrock integration.

## Available Models

### Understanding Models (Generation 1)

| Model ID | Name | Input | Output | Context |
|----------|------|-------|--------|---------|
| amazon.nova-micro-v1:0 | Nova Micro | TEXT | TEXT | 128K |
| amazon.nova-micro-v1:0:24k | Nova Micro 24K | TEXT | TEXT | 24K |
| amazon.nova-lite-v1:0 | Nova Lite | TEXT, IMAGE, VIDEO | TEXT | 300K |
| amazon.nova-lite-v1:0:24k | Nova Lite 24K | TEXT, IMAGE, VIDEO | TEXT | 24K |
| amazon.nova-pro-v1:0 | Nova Pro | TEXT, IMAGE, VIDEO | TEXT | 300K |
| amazon.nova-pro-v1:0:24k | Nova Pro 24K | TEXT, IMAGE, VIDEO | TEXT | 24K |
| amazon.nova-premier-v1:0 | Nova Premier | TEXT, IMAGE, VIDEO | TEXT | 1000K |

### Understanding Models (Generation 2)

| Model ID | Name | Input | Output | Context |
|----------|------|-------|--------|---------|
| amazon.nova-2-lite-v1:0 | Nova 2 Lite | TEXT, IMAGE, VIDEO | TEXT | 256K |

### Generation Models

| Model ID | Name | Input | Output | Notes |
|----------|------|-------|--------|-------|
| amazon.nova-canvas-v1:0 | Nova Canvas | TEXT, IMAGE | IMAGE | Image generation |
| amazon.nova-reel-v1:0 | Nova Reel | TEXT, IMAGE | VIDEO | Async video gen |
| amazon.nova-reel-v1:1 | Nova Reel v1.1 | TEXT, IMAGE | VIDEO | Improved quality |

### Speech Models

| Model ID | Name | Input | Output | Notes |
|----------|------|-------|--------|-------|
| amazon.nova-sonic-v1:0 | Nova Sonic | SPEECH | SPEECH, TEXT | TTS/STT |
| amazon.nova-2-sonic-v1:0 | Nova 2 Sonic | SPEECH | SPEECH, TEXT | Real-time voice |

### Embedding Models

| Model ID | Name | Input | Output | Dimensions |
|----------|------|-------|--------|------------|
| amazon.nova-2-multimodal-embeddings-v1:0 | Nova Multimodal Embeddings | TEXT, IMAGE, AUDIO, VIDEO | EMBEDDING | 256/384/1024/3072 |

## Key Features

- **Native Bedrock Integration**: First-party models with full API support
- **Multimodal Understanding**: Image, video, and text comprehension
- **Long Context**: Up to 1M tokens for Premier
- **Matryoshka Embeddings**: Flexible dimension selection (256-3072)

## API Support

| API | Understanding | Canvas | Reel | Embeddings |
|-----|---------------|--------|------|------------|
| InvokeModel | ✅ | ✅ | ❌ | ✅ |
| Converse | ✅ | ❌ | ❌ | ❌ |
| StartAsyncInvoke | ❌ | ❌ | ✅ | ❌ |
| Streaming | ✅ | ❌ | ❌ | ❌ |

## Request/Response Formats

### Understanding Models (InvokeModel)

**Request:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": [{"text": "What is 2+2?"}]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 100,
    "temperature": 0.7
  }
}
```

**Response:**
```json
{
  "output": {
    "message": {
      "content": [{"text": "2 + 2 equals 4."}],
      "role": "assistant"
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 10,
    "outputTokens": 10,
    "totalTokens": 20,
    "cacheReadInputTokenCount": 0,
    "cacheWriteInputTokenCount": 0
  }
}
```

### Nova Canvas (Image Generation)

**Request:**
```json
{
  "taskType": "TEXT_IMAGE",
  "textToImageParams": {
    "text": "A futuristic city at sunset"
  },
  "imageGenerationConfig": {
    "numberOfImages": 1,
    "quality": "standard",
    "height": 512,
    "width": 512
  }
}
```

**Response:**
```json
{
  "images": ["<base64_encoded_png>"]
}
```

### Nova Multimodal Embeddings

**Request:**
```json
{
  "taskType": "SINGLE_EMBEDDING",
  "singleEmbeddingParams": {
    "embeddingPurpose": "GENERIC_INDEX",
    "embeddingDimension": 256,
    "text": {
      "truncationMode": "END",
      "value": "What is the capital of France?"
    }
  }
}
```

**Response:**
```json
{
  "embeddings": [
    {
      "embeddingType": "TEXT",
      "embedding": [0.008150992, 0.06255241, -0.075534984, ...]
    }
  ]
}
```

## Key Notes

### Understanding Models
- `content` is ALWAYS an array of content blocks, not a string
- `system` is also an array of content blocks
- `inferenceConfig` uses camelCase (maxTokens, topP)
- Response wraps message in `output.message` (singular)

### Nova Canvas
- `taskType`: "TEXT_IMAGE" for text-to-image
- `quality`: "standard" (faster) or "premium" (higher quality)
- `numberOfImages`: 1-4 images per request
- Returns array of base64-encoded PNG images

### Embeddings
- `embeddingPurpose`: "GENERIC_INDEX" or "GENERIC_SEARCH"
- `embeddingDimension`: 256, 384, 1024, or 3072
- `truncationMode`: "NONE", "END", or "START"
- Trained with Matryoshka Representation Learning (MRL)

## Stop Reasons

| Value | Maps To |
|-------|---------|
| `end_turn` | `AWS_BEDROCK_STOP_REASON_END_TURN` |
| `max_tokens` | `AWS_BEDROCK_STOP_REASON_MAX_TOKENS` |
| `stop_sequence` | `AWS_BEDROCK_STOP_REASON_STOP_SEQUENCE` |
| `content_filtered` | `AWS_BEDROCK_STOP_REASON_CONTENT_FILTERED` |
| `guardrail_intervened` | `AWS_BEDROCK_STOP_REASON_GUARDRAILS` |

## Sample Files

| File | Description |
|------|-------------|
| `amazon_nova_micro_invoke_request.json` | Micro InvokeModel request |
| `amazon_nova_micro_invoke_response.json` | Micro InvokeModel response |
| `amazon_nova_micro_converse_request.json` | Micro Converse API request |
| `amazon_nova_micro_converse_response.json` | Micro Converse API response |
| `amazon_nova_lite_invoke_request.json` | Lite InvokeModel request |
| `amazon_nova_lite_invoke_response.json` | Lite InvokeModel response |
| `amazon_nova_pro_invoke_request.json` | Pro InvokeModel request |
| `amazon_nova_pro_invoke_response.json` | Pro InvokeModel response |
| `amazon_nova_canvas_invoke_request.json` | Canvas text-to-image request |
| `amazon_nova_canvas_invoke_response.json` | Canvas response with image |
| `amazon_nova_multimodal_embeddings_invoke_request.json` | Embeddings request |
| `amazon_nova_multimodal_embeddings_invoke_response.json` | Embeddings response |

## References

- [AWS Bedrock Nova Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-nova.html)
- [Nova Canvas User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/nova-canvas.html)
- [Nova Embeddings Blog](https://aws.amazon.com/blogs/aws/amazon-nova-multimodal-embeddings-now-available-in-amazon-bedrock/)
