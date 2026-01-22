# DeepSeek Models on AWS Bedrock

## Overview

DeepSeek models are advanced AI models available on AWS Bedrock, featuring both general-purpose and reasoning-focused variants with chain-of-thought capabilities.

## Available Models

| Model ID | Parameters | Context | Max Output | Reasoning | Released |
|----------|------------|---------|------------|-----------|----------|
| deepseek.v3-v1:0 | 671B MoE | 64K | 163,840 | Optional | 2024 |
| deepseek.r1-v1:0 (us.deepseek.r1-v1:0) | 671B total, 37B active | 64K | 32,768 | **Always-On** | Jan 2025 |

## Key Features

- **Mixture-of-Experts (MoE) Architecture**: 671B total, 37B active parameters per token
- **Always-On Reasoning (R1)**: Chain-of-thought reasoning that cannot be disabled
- **Transparent Reasoning**: R1 exposes full reasoning process via `reasoningContent` field
- **Cross-Region Inference**: R1 supports inference profiles for failover (us., eu., apac.)

## Model Differences

### DeepSeek-V3 (deepseek.v3-v1:0)
- General-purpose text generation
- Supports ON_DEMAND inference
- No built-in reasoning (faster responses)
- Available in: ap-northeast-1, ap-south-1, eu-north-1, eu-west-2, us-east-2, us-west-2

### DeepSeek-R1 (us.deepseek.r1-v1:0)
- Advanced reasoning model with RL fine-tuning
- **Requires inference profile** (cannot use direct model ID)
- Always-on chain-of-thought reasoning
- Reasoning visible in `reasoningContent` field
- Cross-region routing: us-east-1, us-east-2, us-west-2

## API Support

| API | DeepSeek-V3 | DeepSeek-R1 | Notes |
|-----|-------------|-------------|-------|
| InvokeModel | Yes | Yes | OpenAI-compatible format |
| Converse | Yes | Yes | Unified AWS format |
| Streaming | Yes | Yes | Both APIs support streaming |
| Inference Profiles | No | **Required** | R1 only works via inference profile |

## Request Format

### InvokeModel (OpenAI-compatible)

```json
{
  "messages": [
    {"role": "user", "content": "Your question here"}
  ],
  "max_tokens": 100,
  "temperature": 0.7
}
```

### Converse API

```json
{
  "modelId": "deepseek.v3-v1:0",
  "messages": [
    {
      "role": "user",
      "content": [{"text": "Your question here"}]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 100,
    "temperature": 0.7
  }
}
```

## Response Format

### InvokeModel Response (OpenAI-compatible)

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "Response text",
        "role": "assistant"
      }
    }
  ],
  "model": "deepseek.v3-v1:0",
  "usage": {
    "completion_tokens": 41,
    "prompt_tokens": 16,
    "total_tokens": 57
  }
}
```

### Converse Response (with R1 Reasoning)

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [
        {"text": "Final answer here"},
        {
          "reasoningContent": {
            "reasoningText": {
              "text": "Internal reasoning process..."
            }
          }
        }
      ]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 18,
    "outputTokens": 477,
    "totalTokens": 495
  }
}
```

## Critical Constraints

1. **R1 Temperature**: Must be 1.0 when using reasoning (which is always for R1)
2. **Optimal max_tokens**: 8,192 (quality degrades above this)
3. **Maximum max_tokens**: 32,768 for R1, 163,840 for V3
4. **5-minute timeout**: For Knowledge Base integration
5. **Inference Profile Required**: R1 cannot be invoked directly, must use `us.deepseek.r1-v1:0`

## Stop Reasons

| API Value | Common Enum |
|-----------|-------------|
| `stop` | `AWS_BEDROCK_STOP_REASON_END_TURN` |
| `end_turn` | `AWS_BEDROCK_STOP_REASON_END_TURN` |
| `length` | `AWS_BEDROCK_STOP_REASON_MAX_TOKENS` |
| `max_tokens` | `AWS_BEDROCK_STOP_REASON_MAX_TOKENS` |

## Best Practices

1. **Use R1 for complex reasoning**: Math, logic, code analysis
2. **Use V3 for general tasks**: Faster responses, no reasoning overhead
3. **Set adequate max_tokens for R1**: Reasoning consumes tokens (often 200-500)
4. **Parse reasoningContent**: Extract insights from the reasoning field
5. **Plan for latency**: R1 reasoning takes additional time (2-5s typical)

## Sample Files

| File | Description |
|------|-------------|
| `deepseek_v3_converse_request.json` | V3 Converse API request |
| `deepseek_v3_converse_response.json` | V3 Converse response |
| `deepseek_v3_invoke_request.json` | V3 InvokeModel request |
| `deepseek_v3_invoke_response.json` | V3 InvokeModel response |
| `deepseek_r1_converse_request.json` | R1 Converse request with inference profile |
| `deepseek_r1_converse_response.json` | R1 response with reasoningContent |

## References

- [AWS Bedrock DeepSeek Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-deepseek.html)
- [DeepSeek-R1 Model Card](https://github.com/deepseek-ai/DeepSeek-R1)
- [AWS Bedrock Inference Profiles](https://docs.aws.amazon.com/bedrock/latest/userguide/cross-region-inference.html)
