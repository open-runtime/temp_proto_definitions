# NVIDIA Nemotron Models on AWS Bedrock

## Overview

NVIDIA Nemotron models are available on AWS Bedrock with OpenAI-compatible API format. The models use a hybrid Mamba-Transformer mixture-of-experts architecture for efficient inference.

## Available Models

| Model ID | Parameters | Active Params | Context | Max Output | Vision | Released |
|----------|------------|---------------|---------|------------|--------|----------|
| nvidia.nemotron-nano-9b-v2 | 9B | 9B | 128K | 8,192 | No | 2024 |
| nvidia.nemotron-nano-12b-v2 | 12B | 12B | 128K | 8,192 | **Yes** | 2024 |
| nvidia.nemotron-nano-3-30b | 31.6B | 3.2B | 256K (1M native) | 8,192 | No | Dec 2025 |

## Key Features

- **Built-in Reasoning**: Models include internal reasoning with `</think>` tags (similar to DeepSeek R1)
- **OpenAI-Compatible API**: Same format as OpenAI Chat Completions
- **Mixture-of-Experts**: 30B model uses 128 experts + 1 shared, 5 active per token
- **Vision Support**: 12B VL model supports images (PNG, JPEG, GIF, WEBP)
- **Tool Calling**: XML-style tool definitions for agentic workflows

## Vision Support (12B VL Only)

Only `nvidia.nemotron-nano-12b-v2` supports vision. The 9B and 30B models are text-only.

**Tested Results:**
- Converse API: ✅ **Recommended** - Works reliably with `image` content block
- InvokeModel API: ⚠️ Works with OpenAI `image_url` format but may have inconsistent results

**Recommendation:** Use Converse API for vision tasks for most reliable results.

**Image Format Differences:**

| API | Image Format |
|-----|--------------|
| Converse | `{"image": {"format": "png", "source": {"bytes": "<base64>"}}}` |
| InvokeModel | `{"type": "image_url", "image_url": {"url": "data:image/png;base64,<base64>"}}` |

**Limitations:**
- Max recommended image size: ~500KB base64 (resize large images)
- Supported formats: PNG, JPEG, GIF, WEBP

## API Endpoints

Both InvokeModel and Converse API are supported:

### InvokeModel (OpenAI-compatible format)

```bash
aws bedrock-runtime invoke-model \
  --model-id nvidia.nemotron-nano-9b-v2 \
  --region us-east-1 \
  --body "$(echo '{"messages":[{"role":"user","content":"Hello"}],"max_tokens":100}' | base64)" \
  --content-type application/json \
  output.json
```

### Converse API

```bash
aws bedrock-runtime converse \
  --model-id nvidia.nemotron-nano-9b-v2 \
  --region us-east-1 \
  --messages '[{"role":"user","content":[{"text":"Hello"}]}]' \
  --inference-config '{"maxTokens":100}'
```

## Request Parameters

| Parameter | Type | Range | Default | Description |
|-----------|------|-------|---------|-------------|
| max_tokens | int | 1-8192 | Model-specific | Maximum tokens to generate |
| temperature | float | 0.0-2.0 | 1.0 | Controls randomness (lower = deterministic) |
| top_p | float | 0.0-1.0 | 1.0 | Nucleus sampling threshold |
| top_k | int | 1-1000+ | Model-specific | Number of top tokens to consider |
| stop | array | - | [] | Stop sequences to terminate generation |
| repetition_penalty | float | 1.0+ | 1.0 | Penalty for repeated tokens (>1 reduces repetition) |

### Recommended Settings

- **Reasoning tasks**: `temperature=1.0, top_p=1.0, max_tokens=10000`
- **Tool calling**: `temperature=0.6, top_p=0.95`
- **Factual answers**: `temperature=0.1-0.3, top_p=0.9`

## Response Format

### InvokeModel Response (OpenAI-compatible)

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "message": {
        "content": "Response with optional </think> reasoning tags",
        "refusal": null,
        "role": "assistant"
      }
    }
  ],
  "created": 1768943131,
  "id": "chatcmpl-uuid",
  "model": "nvidia.nemotron-nano-9b-v2",
  "object": "chat.completion",
  "service_tier": "default",
  "usage": {
    "completion_tokens": 100,
    "prompt_tokens": 24,
    "total_tokens": 124
  }
}
```

### Converse API Response

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [{"text": "Response text"}]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 24,
    "outputTokens": 100,
    "totalTokens": 124
  },
  "metrics": {
    "latencyMs": 1729
  }
}
```

## Stop Reasons

| InvokeModel | Converse | Meaning |
|-------------|----------|---------|
| `stop` | `end_turn` | Natural completion |
| `length` | `max_tokens` | Token limit reached |
| `tool_calls` | `tool_use` | Model requested tool execution |
| `content_filter` | `content_filtered` | Content safety triggered |

## Regional Availability

- US: us-east-1, us-east-2, us-west-2
- Europe: eu-west-1, eu-west-2, eu-south-1
- Asia Pacific: ap-northeast-1 (Tokyo), ap-south-1 (Mumbai)
- South America: sa-east-1 (São Paulo)

## Pricing (January 2026)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| nemotron-nano-9b-v2 | $0.07 | $0.27 |
| nemotron-nano-12b-v2 | ~$0.10 | ~$0.40 |
| nemotron-nano-3-30b | $0.06 | $0.24 |

## Knowledge Cutoff Dates

- **nemotron-nano-9b-v2**: September 1, 2024
- **nemotron-nano-12b-v2**: September 1, 2024  
- **nemotron-nano-3-30b**: June 25, 2025

## References

- [AWS Bedrock NVIDIA Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-nvidia.html)
- [NVIDIA Nemotron 3 Nano Technical Report](https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Nano-Technical-Report.pdf)
- [NVIDIA Build Nemotron Model Card](https://build.nvidia.com/nvidia/nemotron-3-nano-30b-a3b/modelcard)
