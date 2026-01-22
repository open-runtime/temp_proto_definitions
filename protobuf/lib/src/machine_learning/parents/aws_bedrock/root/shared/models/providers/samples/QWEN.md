# Qwen Models on AWS Bedrock

## Overview

Qwen (Tongyi Qianwen) models from Alibaba Cloud are advanced AI models available on AWS Bedrock, offering general-purpose, coding, and vision capabilities with massive context windows.

## Available Models

| Model ID | Type | Context | Max Output | Vision | Released |
|----------|------|---------|------------|--------|----------|
| qwen.qwen3-235b-a22b-2507-v1:0 | Flagship MoE | 1M | 262,144 | No | July 2025 |
| qwen.qwen3-vl-235b-a22b | Vision MoE | 128K | 32,768 | **Yes** | 2025 |
| qwen.qwen3-next-80b-a3b | Next-gen | 128K | 262,144 | No | 2025 |
| qwen.qwen3-coder-480b-a35b-v1:0 | Coding Flagship | 1M | 131,072 | No | 2025 |
| qwen.qwen3-coder-30b-a3b-v1:0 | Coding Mini | 262K | 262,144 | No | 2025 |
| qwen.qwen3-32b-v1:0 | Dense | 32K | 32,768 | No | 2025 |

## Key Features

- **Massive Context Windows**: Up to 1M tokens (qwen3-235b, qwen3-coder-480b)
- **Mixture-of-Experts (MoE)**: Efficient parameter usage (e.g., 235B total, 22B active)
- **Vision Capabilities**: Qwen3-VL supports PNG, JPEG, GIF, WEBP images and PDF input
- **Code Specialization**: Dedicated coding models with competitive performance
- **Agentic Intelligence**: Strong tool use and reasoning capabilities

## Model Architecture

### Dense Models
- **qwen.qwen3-32b-v1:0**: Dense transformer, all 32B parameters active per token

### MoE Models (A = Active)
- **235B-A22B**: 235B total parameters, 22B active per token
- **480B-A35B**: 480B total parameters, 35B active per token
- **80B-A3B**: 80B total parameters, 3B active per token
- **30B-A3B**: 30B total parameters, 3B active per token

## API Support

| API | Supported | Notes |
|-----|-----------|-------|
| InvokeModel | Yes | Standard format |
| Converse | Yes | **Recommended** |
| Streaming | Yes | Both APIs |
| Guardrails | Yes | All models |
| Vision (VL) | Yes | VL model only |

## Request Format

### Text-Only Request (Converse)

```json
{
  "modelId": "qwen.qwen3-32b-v1:0",
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

### Vision Request (Qwen VL)

```json
{
  "modelId": "qwen.qwen3-vl-235b-a22b",
  "messages": [
    {
      "role": "user",
      "content": [
        {"text": "Describe this image"},
        {
          "image": {
            "format": "png",
            "source": {"bytes": "<base64-encoded-image>"}
          }
        }
      ]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 200,
    "temperature": 0.7
  }
}
```

### Coding Request

```json
{
  "modelId": "qwen.qwen3-coder-30b-a3b-v1:0",
  "messages": [
    {
      "role": "user",
      "content": [{"text": "Write a Python function to sort a list"}]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 500,
    "temperature": 0.3
  }
}
```

## Response Format

### Standard Converse Response

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [{"text": "Response text here"}]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 24,
    "outputTokens": 50,
    "totalTokens": 74
  },
  "metrics": {
    "latencyMs": 500
  }
}
```

## Vision Capabilities (Qwen VL)

### Supported Image Formats
- PNG
- JPEG
- GIF
- WEBP

### Supported Document Input
- PDF

### Vision Use Cases
- Image description and analysis
- Document understanding
- Visual question answering
- Multi-image comparison

## Best Practices

1. **Use Appropriate Model Size**: Dense 32B for fast responses, MoE for complex tasks
2. **Leverage Long Context**: Qwen3-235b supports 1M tokens for extensive documents
3. **Low Temperature for Code**: Use 0.1-0.3 for coding tasks
4. **Vision with VL Model**: Only qwen.qwen3-vl-235b-a22b supports images
5. **Streaming for Long Outputs**: Enable streaming for responses >1000 tokens

## Regional Availability

All Qwen models are available in:
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
| `qwen_32b_converse_request.json` | Dense 32B request |
| `qwen_32b_converse_response.json` | Dense 32B response |
| `qwen_coder_converse_request.json` | Coder model request |
| `qwen_coder_converse_response.json` | Coder response with code |
| `qwen_vl_vision_request.json` | Vision request with image |
| `qwen_vl_vision_response.json` | Vision response |
| `qwen_235b_converse_request.json` | Flagship model request |
| `qwen_235b_converse_response.json` | Flagship model response |

## References

- [AWS Bedrock Qwen Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-qwen.html)
- [Qwen Official Documentation](https://qwen.readthedocs.io/)
- [Alibaba Cloud Qwen Models](https://www.alibabacloud.com/en/solutions/generative-ai/qwen)
