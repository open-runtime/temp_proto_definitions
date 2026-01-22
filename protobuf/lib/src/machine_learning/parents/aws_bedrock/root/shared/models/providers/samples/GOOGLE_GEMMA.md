# Google Gemma Models on AWS Bedrock

## Overview

Google Gemma 3 models are available on AWS Bedrock via the Marketplace. All models are instruction-tuned and support multimodal input (text + images).

> **Naming Convention**: Types are named `AWSBedrockGoogleGemma*` (not `AWSBedrockGoogle*`) to future-proof against potential Google Gemini models on Bedrock.

## Unified Proto Design

The `google.proto` provides a **single unified interface** for all Google Gemma models:

```protobuf
message AWSBedrockGoogleGemmaRequest {
  repeated AWSBedrockGoogleGemmaMessage messages = 1;
  optional float temperature = 2;
  optional float top_p = 3;
  optional uint32 max_tokens = 4;
  repeated string stop = 5;
  optional float presence_penalty = 6;  // text-only
  optional float frequency_penalty = 7; // text-only
  optional int64 seed = 8;              // text-only
}
```

### How It Works

The developer just fills in the request - **the service automatically determines which AWS API to use**:

| Content Type | AWS API Used | Notes |
|-------------|--------------|-------|
| Text-only messages | InvokeModel | Faster, supports penalties/seed |
| Messages with images | Converse | Required for multimodal |

### Message Content Options

```protobuf
message AWSBedrockGoogleGemmaMessage {
  AWSBedrockGoogleGemmaRole role = 1;
  
  // Option 1: Simple text (uses InvokeModel)
  optional string text = 2;
  
  // Option 2: Content blocks for multimodal (uses Converse)
  repeated AWSBedrockGoogleGemmaContentBlock content = 3;
}
```

## Available Models

| Model ID | Parameters | Context | Max Output | Best For |
|----------|-----------|---------|------------|----------|
| `google.gemma-3-4b-it` | 4B | 128K | 8,192 | Edge/mobile, simple tasks |
| `google.gemma-3-12b-it` | 12B | 128K | 8,192 | Balanced performance |
| `google.gemma-3-27b-it` | 27B | 128K | 8,192 | Complex reasoning, images |

## Key Specifications

- **Context Window**: 128,000 tokens
- **Max Output**: 8,192 tokens
- **Languages**: 140+ supported
- **Inference Profiles**: ❌ Not supported (ON_DEMAND only)

### Image Support (via Converse API)

- **Formats**: JPEG, PNG, GIF, WebP
- **Max Size**: 3.75 MB
- **Max Dimensions**: 8,000 × 8,000 pixels
- **Token Cost**: ~100-150 tokens per image

## API Formats

### InvokeModel (Text-Only) - OpenAI Compatible

**Request:**
```json
{
  "messages": [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 100,
  "presence_penalty": 0.5,
  "seed": 42
}
```

**Response:**
```json
{
  "id": "chatcmpl-xxx",
  "model": "google.gemma-3-27b-it",
  "choices": [{
    "index": 0,
    "message": {"role": "assistant", "content": "Hi there!"},
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 14,
    "completion_tokens": 5,
    "total_tokens": 19
  }
}
```

### Converse (Multimodal) - Bedrock Format

**Request:**
```json
{
  "modelId": "google.gemma-3-27b-it",
  "messages": [{
    "role": "user",
    "content": [
      {"text": "What's in this image?"},
      {
        "image": {
          "format": "png",
          "source": {"bytes": "iVBORw0KGgo..."}
        }
      }
    ]
  }]
}
```

**Response:**
```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [{"text": "The image shows..."}]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 281,
    "outputTokens": 15,
    "totalTokens": 296
  }
}
```

## Samples in This Directory

### All Three Model Sizes

| Model | Sample | Description |
|-------|--------|-------------|
| 4B | `google_gemma_3_4b_it_invoke_text_*` | Basic text completion |
| 12B | `google_gemma_3_12b_it_invoke_text_*` | Basic text completion |
| 27B | `google_gemma_3_27b_it_invoke_text_*` | Basic text completion |

### Feature Coverage (27B Model)

| Feature | Sample |
|---------|--------|
| System prompt | `google_gemma_3_27b_it_invoke_with_system_*` |
| All params | `google_gemma_3_27b_it_invoke_with_params_*` |
| Converse text | `google_gemma_3_27b_it_converse_text_*` |
| Red image | `google_gemma_3_27b_it_converse_with_image_*` |
| Blue image | `google_gemma_3_27b_it_converse_with_blue_image_*` |

## Regional Availability

- ap-northeast-1 (Tokyo)
- ap-south-1 (Mumbai)
- eu-south-1 (Milan)
- eu-west-1 (Ireland)
- eu-west-2 (London)
- sa-east-1 (São Paulo)
- us-east-1 (N. Virginia)
- us-east-2 (Ohio)
- us-west-2 (Oregon)

## Verification Results

All samples were created from **actual API calls** (January 2026):

- ✅ InvokeModel uses OpenAI-compatible format
- ✅ Converse API required for images
- ✅ Model correctly identifies colors in images (tested red/blue)
- ✅ All three model sizes respond correctly
- ✅ System prompts work via InvokeModel
- ✅ Parameters (temperature, top_p, penalties, seed, stop) all work

## References

- [AWS Bedrock Google Models](https://aws.amazon.com/bedrock/google/)
- [Gemma 3 on Bedrock Announcement](https://aws.amazon.com/blogs/machine-learning/gemma-3-27b-model-now-available-on-amazon-bedrock-marketplace-and-amazon-sagemaker-jumpstart/)
- [AWS Bedrock Model Parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-google.html)
