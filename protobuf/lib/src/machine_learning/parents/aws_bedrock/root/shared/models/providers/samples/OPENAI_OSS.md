# OpenAI OSS Models on AWS Bedrock

## Overview

OpenAI OSS (Open-Source Style) models on AWS Bedrock are open-weight reasoning models that expose their internal thinking process through `reasoningContent` fields. These models provide transparency into their reasoning similar to DeepSeek-R1.

## Available Models

| Model ID | Size | Context | Max Output | Safeguard | Thinking |
|----------|------|---------|------------|-----------|----------|
| openai.gpt-oss-20b-1:0 | 20B Mini | 128K | 128K | No | **Always-On** |
| openai.gpt-oss-120b-1:0 | 120B Standard | 128K | 128K | No | **Always-On** |
| openai.gpt-oss-safeguard-20b | 20B Mini | 128K | 131K | **Yes** | **Always-On** |
| openai.gpt-oss-safeguard-120b | 120B Standard | 128K | 131K | **Yes** | **Always-On** |

## Key Features

- **Always-On Reasoning**: Internal thinking process always visible via `reasoningContent`
- **Extended/Interleaved Thinking**: Safeguard models support extended thinking capabilities
- **Transparent CoT**: Chain-of-thought reasoning exposed in responses
- **Guardrails Support**: All models support AWS Bedrock Guardrails integration
- **Safeguard Variants**: Enhanced safety filters and content moderation

## Response Structure

All OpenAI OSS models return responses with visible reasoning:

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [
        {
          "reasoningContent": {
            "reasoningText": {
              "text": "Internal thinking process..."
            }
          }
        },
        {
          "text": "Final answer to the user"
        }
      ]
    }
  }
}
```

**Note**: The `reasoningContent` block always appears before the final `text` response, showing the model's thought process.

## API Support

| API | Supported | Notes |
|-----|-----------|-------|
| Converse | Yes | **Recommended** |
| Converse Stream | Yes | Streaming support |
| InvokeModel | Yes | Standard format |
| InvokeModel Stream | Yes | Streaming support |
| Guardrails | Yes | All models |

## Request Format

### Standard Request (Converse)

```json
{
  "modelId": "openai.gpt-oss-120b-1:0",
  "messages": [
    {
      "role": "user",
      "content": [{"text": "Your question here"}]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 300,
    "temperature": 0.7
  }
}
```

### With System Prompt

```json
{
  "modelId": "openai.gpt-oss-safeguard-20b",
  "messages": [
    {
      "role": "user",
      "content": [{"text": "Your question here"}]
    }
  ],
  "system": [{"text": "You are a helpful assistant"}],
  "inferenceConfig": {
    "maxTokens": 300,
    "temperature": 0.7
  }
}
```

## Model Comparison

### Base vs Safeguard Models

| Feature | Base (20b/120b) | Safeguard (20b/120b) |
|---------|-----------------|----------------------|
| Context Window | 128K | 128K |
| Max Output | 128K | 131K |
| Extended Thinking | No | **Yes** |
| Interleaved Thinking | No | **Yes** |
| Safety Filters | Standard | **Enhanced** |
| Content Moderation | Standard | **Enhanced** |

### Size Comparison

| Model | Use Case | Speed | Quality |
|-------|----------|-------|---------|
| 20B | Fast responses, simple tasks | Faster | Good |
| 120B | Complex reasoning, detailed answers | Slower | Better |

## Best Practices

1. **Set Adequate max_tokens**: Reasoning consumes tokens (often 50-150 extra)
2. **Temperature 0.7**: Default works well for most use cases
3. **Parse reasoningContent**: Extract insights from internal thinking
4. **Use Safeguard for Safety-Critical**: Enhanced content moderation
5. **Plan for Extra Latency**: Reasoning adds ~200-500ms

## Stop Reasons

| Value | Meaning |
|-------|---------|
| `end_turn` | Natural completion |
| `max_tokens` | Token limit reached |

## Regional Availability

All models available in:
- ap-northeast-1 (Tokyo)
- ap-south-1 (Mumbai)
- eu-central-1 (Frankfurt)
- eu-north-1 (Stockholm)
- eu-west-2 (London)
- us-east-1 (N. Virginia)
- us-east-2 (Ohio)
- us-west-2 (Oregon)

## Sample Files

| File | Description |
|------|-------------|
| `openai_oss_20b_converse_request.json` | 20B model request |
| `openai_oss_20b_converse_response.json` | 20B response with reasoning |
| `openai_oss_120b_converse_request.json` | 120B model request |
| `openai_oss_120b_converse_response.json` | 120B response with reasoning |
| `openai_oss_safeguard_20b_converse_request.json` | Safeguard 20B request |
| `openai_oss_safeguard_20b_converse_response.json` | Safeguard 20B response |

## References

- [AWS Bedrock Foundation Models](https://docs.aws.amazon.com/bedrock/latest/userguide/foundation-models.html)
- [AWS Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
