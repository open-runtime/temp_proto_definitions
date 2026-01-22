# Mistral AI Prompting Guide

This guide covers best practices for prompting Mistral AI models on AWS Bedrock, including system prompts, structured outputs, tool use, and streaming.

## Table of Contents

- [Main Concepts](#main-concepts)
- [System Prompts](#system-prompts)
- [Structured Outputs](#structured-outputs)
- [Tool Use (Function Calling)](#tool-use-function-calling)
- [Streaming](#streaming)
- [API Formats](#api-formats)
- [Best Practices](#best-practices)
- [What to Avoid](#what-to-avoid)

## Main Concepts

### The Art of Crafting Prompts

Mastering prompt engineering is essential for generating high-quality responses from Mistral models. The core concepts include:

1. **System Prompts** - Set general context and model behavior
2. **Structured Outputs** - Enforce consistent JSON response formats
3. **Few-Shot Prompting** - Provide examples to guide model behavior
4. **Tool Use** - Enable function calling for agentic capabilities

## System Prompts

### Overview

When providing instructions, there are two levels of input:

- **System prompt**: Provided at the beginning of the conversation, sets general context and instructions for model behavior (typically managed by developers)
- **User prompt**: Provided during conversation to give specific context or instructions for the current interaction

### Role-Separated Format (Recommended)

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant specialized in answering questions about geography."
    },
    {
      "role": "user",
      "content": "What is the capital of France?"
    }
  ]
}
```

### Concatenated Format (Fallback)

If you cannot control the system prompt, you can include general context in the user prompt:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "You are a helpful assistant specialized in answering questions about geography.\n\nUser: What is the capital of France?"
    }
  ]
}
```

### Providing a Purpose (Roleplaying)

Define a clear purpose at the start of your prompt:

```
"You are a <role>, your task is to <task>."
```

**Example:**
```
"You are a language detection model, your task is to detect the language of the given text."
```

### Structure

Organize instructions hierarchically with clear sections:

```
You are a language detection model, your task is to detect the language of the given text.

# Available Languages

Select the language from the following list:
- English: "en"
- French: "fr"
- Spanish: "es"
- German: "de"

Any language not listed must be classified as "other" with the code "on".

# Response Format

Your answer must follow this format:
{"language_iso": <language_code>}

# Examples

Below are sample inputs and expected outputs:

## English
User: Hello, how are you?
Answer: {"language_iso": "en"}

## French
User: Bonjour, comment allez-vous?
Answer: {"language_iso": "fr"}
```

### Formatting

Use Markdown and/or XML-style tags for clarity:

- **Readable**: Easy for humans to scan
- **Parsable**: Simple to extract programmatically
- **Familiar**: Models are trained on these formats

## Structured Outputs

### Overview

Mistral models support structured JSON output formats. This ensures consistent, parseable responses.

### Example: Language Detection

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a language detection model. Your task is to detect the language of the given text.\n\n# Available Languages\n- English: \"en\"\n- French: \"fr\"\n- Spanish: \"es\"\n\n# Response Format\nYour answer must follow this format:\n{\"language_iso\": <language_code>}"
    },
    {
      "role": "user",
      "content": "What is the best French cheese? Return the name and the ingredients in short JSON object."
    }
  ],
  "max_tokens": 100,
  "temperature": 0.7
}
```

**Response:**
```json
{
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "{\"name\": \"Camembert\", \"ingredients\": [\"milk\", \"salt\", \"rennet\"]}"
      },
      "finish_reason": "stop"
    }
  ]
}
```

## Tool Use (Function Calling)

### Overview

Mistral Large 2 (24.07) supports tool use (function calling), enabling the model to invoke external tools/functions.

### Tool Definition

```json
{
  "messages": [
    {
      "role": "user",
      "content": "What's the status of my transaction T1001?"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "retrieve_payment_status",
        "description": "Get payment status of a transaction",
        "parameters": {
          "type": "object",
          "properties": {
            "transaction_id": {
              "type": "string",
              "description": "The transaction id."
            }
          },
          "required": ["transaction_id"]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "retrieve_payment_date",
        "description": "Get payment date of a transaction",
        "parameters": {
          "type": "object",
          "properties": {
            "transaction_id": {
              "type": "string",
              "description": "The transaction id."
            }
          },
          "required": ["transaction_id"]
        }
      }
    }
  ]
}
```

### Tool Choice Configuration

Control when and how tools are used:

- **`auto`**: Model decides whether to use tools (default)
- **`none`**: Model will not use tools
- **`any`**: Model must use at least one tool
- **`tool`**: Force use of specific tool by name

**Example:**
```json
{
  "tool_choice": "auto"
}
```

Or force a specific tool:
```json
{
  "tool_choice": {
    "type": "tool",
    "name": "retrieve_payment_status"
  }
}
```

### Tool Call Response

When the model requests a tool call, the response includes:

```json
{
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "call_abc123",
            "type": "function",
            "function": {
              "name": "retrieve_payment_status",
              "arguments": "{\"transaction_id\": \"T1001\"}"
            }
          }
        ]
      },
      "finish_reason": "tool_calls"
    }
  ]
}
```

### Tool Result Response

After executing the tool, send the result back:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "What's the status of my transaction T1001?"
    },
    {
      "role": "assistant",
      "content": null,
      "tool_calls": [
        {
          "id": "call_abc123",
          "type": "function",
          "function": {
            "name": "retrieve_payment_status",
            "arguments": "{\"transaction_id\": \"T1001\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "{\"status\": \"Paid\"}",
      "tool_call_id": "call_abc123"
    }
  ],
  "tools": [...]
}
```

## Streaming

### Overview

Mistral models support streaming responses via two APIs:

1. **InvokeModelWithResponseStream** - Native Mistral streaming format
2. **ConverseStream** - Unified Bedrock streaming format

### InvokeModelWithResponseStream

**Request:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "What is the best French cheese?"
    }
  ]
}
```

**Response Format:**
Stream chunks contain `outputs` arrays with **incremental text** (deltas):

```json
{
  "outputs": [
    {
      "text": "The best French cheese",
      "stop_reason": null
    }
  ]
}
```

**Important**: Each chunk contains **incremental text**, not the full accumulated response. You must accumulate the text across chunks:

```dart
String accumulated = '';
await for (final response in stream) {
  if (response.hasSuccess() && response.success.hasMistral()) {
    for (final choice in response.success.mistral.choices) {
      if (choice.message.hasText()) {
        accumulated += choice.message.text; // Append incremental chunk
      }
    }
  }
}
```

### ConverseStream

**Request:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "text": "What is the best French cheese?"
        }
      ]
    }
  ]
}
```

**Response Format:**
Uses Bedrock's unified streaming format with events:
- `messageStart`
- `contentBlockDelta`
- `messageStop`
- `metadata`

## API Formats

### InvokeModel (Non-Streaming)

**Request Format:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "What is the capital of France?"
    }
  ],
  "max_tokens": 100,
  "temperature": 0.7
}
```

**Response Format (Mistral Large 2402+):**
```json
{
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The capital of France is Paris."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 8,
    "total_tokens": 18
  },
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "model": "mistral.mistral-large-2402-v1:0"
}
```

**Response Format (Older Models):**
```json
{
  "outputs": [
    {
      "text": "The capital of France is Paris.",
      "stop_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "total_tokens": 18
  }
}
```

### Converse (Non-Streaming)

**Request Format:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "text": "What is the capital of France?"
        }
      ]
    }
  ]
}
```

**Response Format:**
Same as InvokeModel response format.

## Best Practices

### 1. Stay Flexible and Experiment

Different models and updates can change behavior. Iterate on prompts similar to code:

- Test different prompt structures
- Evaluate impact of changes
- Monitor model updates

### 2. Use Clear, Objective Language

**Good:**
```
"If the record is longer than 100 characters, split it into multiple records."
```

**Bad:**
```
"If the record is too long, split it into multiple records."
```

### 3. Provide Examples (Few-Shot Prompting)

Include examples in your prompt to guide model behavior:

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a language detection model..."
    },
    {
      "role": "user",
      "content": "Hello, how are you?"
    },
    {
      "role": "assistant",
      "content": "{\"language_iso\": \"en\"}"
    },
    {
      "role": "user",
      "content": "Bonjour, comment allez-vous?"
    },
    {
      "role": "assistant",
      "content": "{\"language_iso\": \"fr\"}"
    }
  ]
}
```

### 4. Use Worded Scales Instead of Numeric

**Good:**
```
Rate these options using this scale:
- Very Low: if the option is highly irrelevant
- Low: if the option is not good enough
- Neutral: if the option is not particularly interesting
- Good: if the option is worth considering
- Very Good: for highly relevant options
```

**Bad:**
```
"Rate these options on a 1 to 5 scale, 1 being highly irrelevant and 5 being highly relevant."
```

## What to Avoid

### 1. Avoid Subjective and Blurry Words

**Avoid:**
- "too long", "too short", "many", "few"
- "things", "stuff"
- "write an interesting report"
- "make it better"

**Instead:**
- Provide objective measures
- State exactly what you mean

### 2. Avoid Contradictions

**Bad:**
```
"If the new data is related to an existing database record, update this record."
"If the data is new, create a new record."
```

**Good:**
Use a decision tree:
```
## How to update database records

Follow these steps:
- If the data does not include new information (i.e., it already exists in a record):
  - Ignore this data.
- Otherwise, if the data is not related to any existing record in the same table:
  - Create a new record.
- Otherwise, if the related record is larger than 100 characters:
  - Create a new record.
- Otherwise, if the data directly contradicts the existing record:
  - Delete the existing record and create a new one.
- Otherwise:
  - Update the existing record to include the new data.
```

### 3. Do Not Make LLMs Count Words

**Avoid:**
```
"If the record is too long, split it into multiple records."
"If the record is longer than 100 characters, split it into multiple records."
```

**Instead:**
Provide character counts as input:
```json
{
  "existing_records": [
    {"record": "User: Alice, Age: 30", "charCount": 15},
    {"record": "User: Bob, Age: 25", "charCount": 13}
  ],
  "new_data": {
    "data": "User: Charlie, Age: 35",
    "charCount": 17
  }
}
```

### 4. Do Not Generate Too Many Tokens

Models are faster at ingesting tokens than generating them. Only ask for what is strictly necessary.

**Bad:**
- Generating full record content for a NO_OP operation
- Generating an entire book in one shot

**Good:**
- Only generate the update or necessary data
- Use structured outputs to limit response size

## Model-Specific Notes

### Mistral Large 2 (24.07)

- **Model ID**: `mistral.mistral-large-2407-v1:0` or `mistral.mistral-large-2402-v1:0`
- **Features**: Tool use, structured outputs, streaming
- **Response Format**: Uses `choices` array format (not `outputs`)
- **Max Tokens**: 32K

### Mistral Small (24.02)

- **Model ID**: `mistral.mistral-small-2402-v1:0`
- **Features**: Basic chat completion
- **Response Format**: Uses `outputs` array format

### Pixtral Large (25.02)

- **Model ID**: `mistral.pixtral-large-2502-v1:0`
- **Features**: Multimodal (text + images)
- **Image Support**: Base64 or S3 locations

## Unified Request Helpers

### Overview

The unified request helpers (`AWSBedrockUnifiedRequestHelpers.composeMistralRequest`) automatically implement Mistral prompting best practices:

- **Instructions** → System message (role-separated format - recommended)
- **Context** → Formatted with clear separators (`## Context` / `## Query`)
- **Query** → User message content
- **Images** → Content blocks for Pixtral models

### Example with Unified Helpers

```dart
import 'package:runtime_isomorphic_library/machine_learning/parents/aws_bedrock/root/shared/extensions/unified_request_helpers.dart';

// Automatically formats according to Mistral best practices:
// - System message for instructions (role-separated)
// - Clear formatting for context and query
final request = AWSBedrockUnifiedRequestHelpers.composeMistralRequest(
  instructions: 'You are a helpful assistant specialized in geography.',
  context: 'France is a country in Western Europe.',
  query: 'What is the capital of France?',
);
```

**Resulting Request Structure:**
```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant specialized in geography."
    },
    {
      "role": "user",
      "content": "## Context\n\nFrance is a country in Western Europe.\n\n## Query\n\nWhat is the capital of France?"
    }
  ]
}
```

This automatically follows Mistral best practices:
- ✅ Role-separated system prompt (recommended)
- ✅ Clear formatting with markdown headers
- ✅ Proper separation of context and query

## Code Examples

### Basic Request

```dart
final request = AWSBedrockInferenceRequest(
  region: 'us-east-1',
  modelIdentifier: AWSBedrockModelIdentifier(
    customModelId: 'mistral.mistral-large-2402-v1:0',
  ),
  mistral: AWSBedrockMistralRequest(
    messages: [
      AWSBedrockMistralMessage(
        role: AWSBedrockMistralRole.AWS_BEDROCK_MISTRAL_ROLE_SYSTEM,
        text: 'You are a helpful assistant.',
      ),
      AWSBedrockMistralMessage(
        role: AWSBedrockMistralRole.AWS_BEDROCK_MISTRAL_ROLE_USER,
        text: 'What is the capital of France?',
      ),
    ],
    config: AWSBedrockMistralInferenceConfig()
      ..maxTokens = 100
      ..temperature = 0.7,
  ),
);
```

### Tool Use Request

```dart
final request = AWSBedrockInferenceRequest(
  region: 'us-east-1',
  modelIdentifier: AWSBedrockModelIdentifier(
    customModelId: 'mistral.mistral-large-2402-v1:0',
  ),
  mistral: AWSBedrockMistralRequest(
    messages: [
      AWSBedrockMistralMessage(
        role: AWSBedrockMistralRole.AWS_BEDROCK_MISTRAL_ROLE_USER,
        text: "What's the status of transaction T1001?",
      ),
    ],
    tools: [
      AWSBedrockMistralTool()
        ..name = 'retrieve_payment_status'
        ..description = 'Get payment status of a transaction'
        ..inputSchema = Struct()
          ..fields['transaction_id'] = Value(stringValue: 'string'),
    ],
    toolChoice: AWSBedrockMistralToolChoice()
      ..auto = AWSBedrockMistralToolChoiceAuto(),
    config: AWSBedrockMistralInferenceConfig()
      ..maxTokens = 1024,
  ),
);
```

### Streaming Request

```dart
final stream = client.predictWithStream(
  AWSBedrockInferenceRequest(
    region: 'us-east-1',
    modelIdentifier: AWSBedrockModelIdentifier(
      customModelId: 'mistral.mistral-large-2402-v1:0',
    ),
    mistral: AWSBedrockMistralRequest(
      messages: [
        AWSBedrockMistralMessage(
          role: AWSBedrockMistralRole.AWS_BEDROCK_MISTRAL_ROLE_USER,
          text: 'What is the best French cheese?',
        ),
      ],
      config: AWSBedrockMistralInferenceConfig()
        ..maxTokens = 1024
        ..temperature = 0.7,
    ),
  ),
);

await for (final response in stream) {
  if (response.hasSuccess() && response.success.hasMistral()) {
    final mistral = response.success.mistral;
    for (final choice in mistral.choices) {
      if (choice.message.hasText()) {
        print(choice.message.text);
      }
    }
  }
}
```

## References

- [AWS Bedrock Mistral Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-mistral.html)
- [Mistral AI Documentation](https://docs.mistral.ai/)
- [AWS Bedrock API Reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/)

