# Mistral Voxtral Audio Models on AWS Bedrock

> **Last Verified: January 2026** via actual API testing with boto3 SDK

## Overview

Mistral Voxtral models are multimodal audio understanding models available on AWS Bedrock. They support:
- Audio transcription (speech-to-text)
- **Timestamps** (Voxtral Small only)
- **Speaker diarization** (multi-speaker identification)
- Audio understanding with text instructions
- Function calling from voice input (Small model only)
- 100+ language support with auto-detection

## Available Models

| Model ID | Parameters | Timestamps | Speaker ID | Function Calling |
|----------|------------|------------|------------|------------------|
| `mistral.voxtral-mini-3b-2507` | 3B | ❌ No | ⚠️ Prompt-only | ❌ No |
| `mistral.voxtral-small-24b-2507` | 24B | ✅ Yes | ✅ Yes | ✅ Yes |

## API Endpoint

Voxtral models use the **Converse API** (NOT InvokeModel):

```
POST /model/{modelId}/converse
```

## Supported Audio Formats (VERIFIED)

> ⚠️ **IMPORTANT**: Only MP3 and WAV are supported! Other formats are rejected.

| Format | Status |
|--------|--------|
| WAV | ✅ Verified working |
| MP3 | ✅ Documented as supported |
| FLAC | ❌ Rejected by API |
| M4A | ❌ Rejected by API |
| OGG | ❌ Rejected by API |

## Audio Constraints

| Parameter | Min | Max |
|-----------|-----|-----|
| Duration | 500ms | 240 minutes (4 hours) |
| File Size | - | 2GB |
| Sample Rate | 8kHz | 48kHz (16-22kHz recommended) |
| Channels | 1 (mono) | 2 (stereo) |
| Files per request | 1 | 8 |

## Timestamps (Voxtral Small 24B ONLY)

Timestamps are returned **embedded in the text response**, not as structured JSON fields.

### How to Enable Timestamps

Use `additionalModelRequestFields`:

```python
response = client.converse(
    modelId='mistral.voxtral-small-24b-2507',
    messages=messages,
    inferenceConfig={'maxTokens': 2000},
    additionalModelRequestFields={
        'timestamp_granularities': ['segment']
    }
)
```

### Timestamp Output Format

Timestamps appear in the text with format: `[ {time_start} - {time_end} ] {text}`

**Example (actual API response):**
```
[ 0m0s110ms - 0m0s420ms ] Hello.
[ 0m0s770ms - 0m3s890ms ] This is a test of the Voxtral speech recognition system.
[ 0m4s180ms - 0m7s160ms ] I am testing timestamps and speaker identification.
```

### Voxtral Mini 3B: Ignores Timestamps

When `timestamp_granularities` is set on Mini, it's **silently ignored**:
```
"Here's the transcription of your audio:\n\n\"Hello, this is a test...\""
```

## Speaker Diarization

Speaker identification is **prompt-driven**, not parameter-driven.

### How to Enable Speaker Diarization

Request it in your prompt:

```python
messages = [{
    'role': 'user',
    'content': [
        {'audio': {'format': 'wav', 'source': {'bytes': audio_bytes}}},
        {'text': 'Please transcribe this audio and identify different speakers with labels (Speaker 1, Speaker 2, etc).'}
    ]
}]
```

### Speaker Diarization Output

**Example (actual API response):**
```
[ Speaker 1 ] Hello, I am the first speaker. How are you today?
[ Speaker 2 ] Hello, I am the second speaker. I am doing well, thank you.
```

### Combined Timestamps + Speaker Labels

You can get BOTH by requesting it in your prompt:

**Prompt:** "Please transcribe this audio with BOTH timestamps and speaker labels."

**Example (actual API response):**
```
[00:00:00] Speaker 1: Hello, I am the first speaker. How are you today?
[00:00:03] Speaker 2: Hello, I am the second speaker. I am doing well, thank you.
```

## Request Format

Audio is passed via the `audio` content block in the Converse API:

```json
{
  "modelId": "mistral.voxtral-small-24b-2507",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "audio": {
            "format": "wav",
            "source": {
              "bytes": "<raw-bytes-for-SDK-or-base64-for-REST>"
            }
          }
        },
        {
          "text": "Please transcribe this audio with timestamps and speaker labels."
        }
      ]
    }
  ],
  "inferenceConfig": {
    "maxTokens": 2000,
    "temperature": 0.1
  },
  "additionalModelRequestFields": {
    "timestamp_granularities": ["segment"],
    "language": "en"
  }
}
```

### Audio Source Options

**Option 1: Inline bytes (raw bytes for SDK, base64 for REST)**
```json
{
  "audio": {
    "format": "wav",
    "source": {
      "bytes": "<audio-data>"
    }
  }
}
```

**Option 2: S3 Reference (for large files)**
```json
{
  "audio": {
    "format": "mp3",
    "source": {
      "s3Location": {
        "uri": "s3://my-bucket/audio/recording.mp3"
      }
    }
  }
}
```

## Response Format

The response is always a standard Converse API response with text content.
**The text content format is 100% controlled by your prompt!**

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [
        {
          "text": "<format depends on your prompt>"
        }
      ]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 401,
    "outputTokens": 58,
    "totalTokens": 459
  },
  "metrics": {
    "latencyMs": 2500
  }
}
```

## Output Format Control (IMPORTANT!)

**Voxtral's output format is 100% prompt-driven.** Unlike OpenAI Whisper's `response_format` parameter, Voxtral doesn't have predefined formats. Instead, you describe the format you want in your prompt!

### Available Formats (All Verified)

| Format | Prompt | Example Output |
|--------|--------|----------------|
| **Plain** | "Transcribe verbatim" | `Hello, I am the first speaker...` |
| **JSON** | "Return as JSON: {segments: [{speaker, text, start_ms, end_ms}]}" | Valid JSON object |
| **SRT** | "Return in SRT subtitle format" | `1\n00:00:00,000 --> 00:00:02,000\n...` |
| **Markdown Table** | "Return as markdown table" | `| Start | End | Speaker | Text |` |
| **Analysis** | "Provide transcription, speaker count, duration, sentiment" | Full analysis report |

### JSON Format Example

**Prompt:**
```
Transcribe this audio and return the output as JSON with this structure:
{"segments": [{"speaker": "...", "text": "...", "start_ms": 0, "end_ms": 0}]}
```

**Response:**
```json
{
  "segments": [
    {"speaker": "Speaker 1", "text": "Hello, I am the first speaker. How are you today?", "start_ms": 0, "end_ms": 3000},
    {"speaker": "Speaker 2", "text": "Hello, I am the second speaker. I am doing well, thank you.", "start_ms": 3000, "end_ms": 6000}
  ]
}
```

### SRT Format Example

**Prompt:**
```
Transcribe this audio in standard SRT subtitle format.
Example SRT format:
1
00:00:00,000 --> 00:00:03,000
First subtitle text
```

**Response:**
```
1
00:00:00,000 --> 00:00:02,000
Hello, I am the first speaker.

2
00:00:02,000 --> 00:00:03,000
How are you today?

3
00:00:03,000 --> 00:00:05,000
Hello, I am the second speaker.

4
00:00:05,000 --> 00:00:07,000
I am doing well, thank you.
```

### Analysis Format Example

**Prompt:**
```
Analyze this audio and provide:
1. Full transcription
2. Number of unique speakers detected
3. Duration of each speaker's speech
4. Overall sentiment
```

**Response:**
```
1. **Full transcription:**
   - Speaker 1: "Hello, I am the first speaker. How are you today?"
   - Speaker 2: "Hello, I am the second speaker. I am doing well, thank you."

2. **Number of unique speakers detected:** 2

3. **Duration of each speaker's speech:**
   - Speaker 1: 4 seconds
   - Speaker 2: 3 seconds

4. **Overall sentiment:** Positive. Both speakers are polite and engaging in a friendly conversation.
```

## additionalModelRequestFields

| Parameter | Type | Description | Voxtral Small | Voxtral Mini |
|-----------|------|-------------|---------------|--------------|
| `timestamp_granularities` | string[] | `["segment"]` | ✅ Works | ❌ Ignored |
| `language` | string | ISO 639-1 code (e.g., "en") | ✅ Works | ✅ Works |

**Note:** `timestamp_granularities` and `language` may be mutually exclusive per Mistral docs.

## Streaming Support

Voxtral supports streaming via `ConverseStream` for real-time transcription output.

## Region Availability

Voxtral models are available in:
- `ap-northeast-1` (Tokyo) ✅ Verified
- (Check AWS docs for latest availability)

## Token Usage

Audio processing consumes approximately 5-15 input tokens per second of audio.

**Example from tests:**
- 2-second tone: ~401 input tokens
- 7-second speech: ~395 input tokens

## Python SDK Example (VERIFIED)

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='ap-northeast-1')

with open('audio.wav', 'rb') as f:
    audio_bytes = f.read()

response = client.converse(
    modelId='mistral.voxtral-small-24b-2507',
    messages=[{
        'role': 'user',
        'content': [
            {
                'audio': {
                    'format': 'wav',
                    'source': {'bytes': audio_bytes}  # Raw bytes, not base64!
                }
            },
            {'text': 'Please transcribe with timestamps and speaker labels.'}
        ]
    }],
    inferenceConfig={'maxTokens': 2000, 'temperature': 0.1},
    additionalModelRequestFields={
        'timestamp_granularities': ['segment']
    }
)

print(response['output']['message']['content'][0]['text'])
```

## Model Comparison

| Feature | Voxtral Mini 3B | Voxtral Small 24B |
|---------|-----------------|-------------------|
| Transcription | ✅ Yes | ✅ Yes |
| Timestamps | ❌ No | ✅ Yes (in text) |
| Speaker Diarization | ⚠️ Prompt only | ✅ Better |
| Function Calling | ❌ No | ✅ Yes |
| Latency | ~2s | ~3s |
| Cost | Lower | Higher |

## Integration Notes

- Use **Converse API**, not InvokeModel
- Boto3 SDK expects **raw bytes**, not base64 string
- REST API expects **base64-encoded** bytes
- Output is always **text** (not audio)
- Timestamps and speakers are **embedded in text**, not structured fields
- For S3 references, ensure proper IAM permissions
- **Only WAV and MP3 formats work** - others are rejected

## Sample Files in This Directory

- `mistral_voxtral_mini_3b_converse_request.json` - Request format
- `mistral_voxtral_mini_3b_converse_response.json` - Basic response
- `mistral_voxtral_mini_3b_speech_response.json` - Speech transcription
- `mistral_voxtral_small_24b_speech_response.json` - With timestamps
- `mistral_voxtral_small_24b_multispeaker_response.json` - With speaker labels

## Test Verification Details

**Tests conducted January 2026:**

1. **Tone Test (440Hz):** Both models correctly identified "a single, steady tone"
2. **Speech Test:** Single speaker with macOS TTS (Samantha voice)
3. **Multi-speaker Test:** Two speakers (Alex + Samantha voices)
4. **Timestamp Test:** Verified segment-level timestamps on Small model
5. **Speaker Diarization Test:** Verified Speaker 1/Speaker 2 labels
6. **Format Test:** M4A rejected, WAV accepted

**Region:** ap-northeast-1 (Tokyo)
