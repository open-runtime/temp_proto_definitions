# Moonshot AI Kimi K2 Thinking - AWS Bedrock API Samples

## Model Information

| Property | Value |
|----------|-------|
| Model ID | `moonshot.kimi-k2-thinking` |
| Provider | Moonshot AI |
| Architecture | Mixture-of-Experts (MoE) |
| Parameters | 1T total, 32B activated |
| Input Modalities | TEXT |
| Output Modalities | TEXT |
| Inference Types | ON_DEMAND only |
| Subscription | Required (Bedrock Marketplace) |

## Key Characteristics

### Always-On Reasoning
Kimi K2 is a **reasoning model** with always-on chain-of-thought thinking:
- Reasoning **cannot be disabled**
- Every response includes visible reasoning traces
- Similar to DeepSeek R1

### Reasoning Output Format

**InvokeModel API**: Reasoning is embedded in content as XML-style tags:
```
<reasoning>
  The user asks... I should...
</reasoning>
Final answer here
```

**Converse API**: Reasoning is in a separate structured field:
```json
{
  "content": [
    {
      "reasoningContent": {
        "reasoningText": {
          "text": "The user asks... I should..."
        }
      }
    },
    {
      "text": "Final answer here"
    }
  ]
}
```

## API Support

| API | Supported | Notes |
|-----|-----------|-------|
| InvokeModel | Yes | OpenAI-compatible format |
| Converse | Yes | Structured reasoning output |
| Streaming | Yes | Both APIs support streaming |
| Inference Profiles | No | ON_DEMAND only |

## Regional Availability

- ap-northeast-1 (Tokyo)
- ap-south-1 (Mumbai)
- us-east-1 (N. Virginia)
- us-west-2 (Oregon)

## Request Format

### InvokeModel (OpenAI-compatible)

```json
{
  "messages": [
    {"role": "system", "content": "Optional system prompt"},
    {"role": "user", "content": "Your question here"}
  ],
  "max_tokens": 500,
  "temperature": 0.7,
  "top_p": 0.9
}
```

### Converse API

```json
{
  "modelId": "moonshot.kimi-k2-thinking",
  "messages": [
    {
      "role": "user",
      "content": [{"text": "Your question here"}]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 1000,
    "temperature": 0.3
  }
}
```

## Response Format

### InvokeModel Response

```json
{
  "id": "chatcmpl-{uuid}",
  "object": "chat.completion",
  "created": 1768942633,
  "model": "moonshot.kimi-k2-thinking",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "<reasoning>...</reasoning> Final answer"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 30,
    "completion_tokens": 194,
    "total_tokens": 224
  }
}
```

### Converse API Response

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [
        {
          "reasoningContent": {
            "reasoningText": {
              "text": "Reasoning process..."
            }
          }
        },
        {
          "text": "Final answer"
        }
      ]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 38,
    "outputTokens": 82,
    "totalTokens": 120
  },
  "metrics": {
    "latencyMs": 1561
  }
}
```

## Stop Reasons

| API Value | Common Enum |
|-----------|-------------|
| `stop` | `AWS_BEDROCK_STOP_REASON_END_TURN` |
| `end_turn` | `AWS_BEDROCK_STOP_REASON_END_TURN` |
| `length` | `AWS_BEDROCK_STOP_REASON_MAX_TOKENS` |
| `max_tokens` | `AWS_BEDROCK_STOP_REASON_MAX_TOKENS` |

## Best Practices

1. **Set adequate max_tokens**: Reasoning consumes tokens. For complex tasks, use 1000+.
2. **Lower temperature for reasoning**: 0.3-0.7 often produces better reasoning quality.
3. **Plan for latency**: Reasoning takes time. Set appropriate timeouts.
4. **Parse reasoning**: Extract insights from `<reasoning>` tags or `reasoningContent` field.

## Sample Files

| File | Description |
|------|-------------|
| `moonshot_kimi_k2_thinking_invoke_request.json` | Basic InvokeModel request |
| `moonshot_kimi_k2_thinking_invoke_response.json` | InvokeModel response with reasoning |
| `moonshot_kimi_k2_thinking_invoke_with_system_request.json` | Request with system prompt |
| `moonshot_kimi_k2_thinking_invoke_with_system_response.json` | Response to system prompt |
| `moonshot_kimi_k2_thinking_converse_request.json` | Converse API request |
| `moonshot_kimi_k2_thinking_converse_response.json` | Converse response with structured reasoning |

## References

- [AWS Bedrock Supported Models](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)
- [AWS Bedrock Model Regions](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html)
- [Kimi K2 Official Documentation](https://moonshotai.github.io/Kimi-K2/)
