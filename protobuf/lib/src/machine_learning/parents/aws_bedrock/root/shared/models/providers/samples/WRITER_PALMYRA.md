# Writer Palmyra API Samples

## Verification Status

All samples in this directory were captured from **actual AWS Bedrock API calls** on January 20, 2026.

### API Endpoint Used
- **InvokeModel**: All `writer_palmyra_x5_v1_0_*.json` files
- **Converse**: `writer_palmyra_x5_v1_0_converse_*.json` files

### Model IDs Tested
- `us.writer.palmyra-x5-v1:0` (System Inference Profile - X5)
- `us.writer.palmyra-x4-v1:0` (System Inference Profile - X4)

> **Note:** Direct model invocation (`writer.palmyra-x5-v1:0`) returns `ValidationException`.
> Writer models REQUIRE inference profiles.

## Sample Files

### InvokeModel API (OpenAI-compatible format)

| File | Description | Verified |
|------|-------------|----------|
| `writer_palmyra_x5_v1_0_text_request.json` | Basic text completion | ✅ Real API |
| `writer_palmyra_x5_v1_0_text_response.json` | Basic response | ✅ Real API |
| `writer_palmyra_x5_v1_0_tool_calling_request.json` | Tool/function calling | ✅ Real API |
| `writer_palmyra_x5_v1_0_tool_calling_response.json` | Tool call response | ✅ Real API |
| `writer_palmyra_x5_v1_0_multi_turn_with_tool_result_request.json` | Multi-turn with tool result | ✅ Real API |
| `writer_palmyra_x5_v1_0_multi_turn_with_tool_result_response.json` | Tool result response | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_seed_request.json` | Deterministic generation | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_seed_response.json` | Seeded response | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_system_request.json` | System message | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_system_response.json` | System response | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_stop_sequences_request.json` | Stop sequences | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_stop_sequences_response.json` | Stop response (shows stop_reason) | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_penalties_request.json` | Presence/frequency penalty | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_penalties_response.json` | Penalty response | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_top_p_request.json` | Top-P sampling | ✅ Real API |
| `writer_palmyra_x5_v1_0_with_top_p_response.json` | Top-P response | ✅ Real API |
| `writer_palmyra_x4_v1_0_text_request.json` | Palmyra X4 text | ✅ Real API |
| `writer_palmyra_x4_v1_0_text_response.json` | X4 response | ✅ Real API |

### Converse API (AWS native format)

| File | Description | Verified |
|------|-------------|----------|
| `writer_palmyra_x5_v1_0_converse_request.json` | Converse API request | ✅ Real API |
| `writer_palmyra_x5_v1_0_converse_response.json` | Converse response | ✅ Real API |
| `writer_palmyra_x5_v1_0_converse_with_document_request.json` | Document input | 📝 Based on docs |
| `writer_palmyra_x5_v1_0_converse_with_document_response.json` | Document response | 📝 Based on docs |

## Key Findings

### InvokeModel Format (OpenAI-compatible)

**Request:**
```json
{
  "messages": [{"role": "user", "content": "Simple string content"}],
  "temperature": 0.7,
  "max_tokens": 100,
  "stop": ["sequence"],
  "seed": 42
}
```

**Response:**
```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "model": "writer.palmyra-x5-v1:0",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Response text",
      "tool_calls": []
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 252,
    "completion_tokens": 8,
    "total_tokens": 260
  }
}
```

### Converse Format (AWS native)

**Response:**
```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [{"text": "Response"}]
    }
  },
  "stopReason": "max_tokens",
  "usage": {
    "inputTokens": 247,
    "outputTokens": 200,
    "totalTokens": 447
  },
  "metrics": {"latencyMs": 2534}
}
```

## Notes

1. **X5 vs X4 response differences**: X5 has additional fields (`refusal`, `annotations`, `audio`, `function_call`, `service_tier`, `system_fingerprint`, `kv_transfer_params`). X4 has a simpler response.

2. **finish_reason values**: `"stop"` (natural), `"length"` (max_tokens), `"tool_calls"` (tool use)

3. **stop_reason**: Only populated when a custom stop sequence triggered termination.

4. **Tool calling**: Uses OpenAI format with `type: "function"` and `function: {name, parameters}`.
