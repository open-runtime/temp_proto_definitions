# TwelveLabs Pegasus v1.2 on AWS Bedrock

## Overview

TwelveLabs Pegasus v1.2 (`twelvelabs.pegasus-1-2-v1:0`) is a state-of-the-art video understanding model that can analyze video content and answer questions about it. It supports both S3 URI and base64 video input.

## Model Details

| Property | Value |
|----------|-------|
| Model ID | `twelvelabs.pegasus-1-2-v1:0` |
| Provider | TwelveLabs |
| Input | TEXT + VIDEO |
| Output | TEXT |
| Context Window | 32,000 tokens |
| Max Output | 4,096 tokens |
| Inference Type | **INFERENCE_PROFILE** (required) |

## Key Features

- **Video Understanding**: Analyze video content for objects, actions, events
- **Temporal Awareness**: Timestamps for detected elements
- **Question Answering**: Ask specific questions about video content
- **Summarization**: Generate video summaries
- **Structured Output**: Optional JSON schema for structured responses

## Inference Profiles

Pegasus requires an inference profile - direct model invocation is not supported:

| Profile | Regions |
|---------|---------|
| `us.twelvelabs.pegasus-1-2-v1:0` | us-east-1, us-east-2, us-west-1, us-west-2 |
| `eu.twelvelabs.pegasus-1-2-v1:0` | eu-central-1, eu-north-1, eu-south-1, eu-south-2, eu-west-1, eu-west-3 |

## Request Format

### With S3 URI (up to 2GB / 1 hour)

```json
{
  "inputPrompt": "Describe the key events in this video",
  "mediaSource": {
    "s3Location": {
      "uri": "s3://bucket-name/video.mp4",
      "bucketOwner": "123456789012"
    }
  },
  "temperature": 0.2,
  "maxOutputTokens": 2000
}
```

### With Base64 (up to 25MB)

```json
{
  "inputPrompt": "Summarize the main activities shown",
  "mediaSource": {
    "base64String": "<base64-encoded-video>"
  },
  "temperature": 0.7,
  "maxOutputTokens": 1000
}
```

### Request Parameters

| Parameter | Type | Required | Limits |
|-----------|------|----------|--------|
| `inputPrompt` | string | Yes | Max 2000 tokens |
| `mediaSource.s3Location.uri` | string | Conditional | Max 2GB, 1 hour |
| `mediaSource.base64String` | string | Conditional | Max 25MB |
| `temperature` | float | No | 0.0 - 1.0 (default 0.2) |
| `maxOutputTokens` | int | No | Max 4096 |
| `responseFormat.json_schema` | object | No | For structured output |

## Response Format

```json
{
  "message": "Video analysis response text...",
  "finishReason": "stop"
}
```

### Finish Reasons

| Value | Meaning |
|-------|---------|
| `stop` | Completed normally |
| `length` | Hit token limit |

## CLI Example

```bash
aws bedrock-runtime invoke-model \
  --model-id "us.twelvelabs.pegasus-1-2-v1:0" \
  --body '{"inputPrompt":"Describe this video","mediaSource":{"s3Location":{"uri":"s3://bucket/video.mp4"}}}' \
  --content-type "application/json" \
  --region us-west-2 \
  output.json
```

## S3 Bucket Permissions

Ensure Bedrock can access your S3 video:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "bedrock.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

## Supported Video Formats

- MP4 (recommended)
- AVI
- MOV
- WebM
- Other common formats

## Use Cases

1. **Video Summarization**: Generate concise summaries of long videos
2. **Object Detection**: Identify objects and their timestamps
3. **Action Recognition**: Detect and describe activities
4. **Content Moderation**: Analyze video for specific content types
5. **Accessibility**: Generate video descriptions for visually impaired
6. **Search Indexing**: Extract searchable metadata from videos

## Best Practices

1. **Use S3 for Large Videos**: Base64 limited to 25MB
2. **Be Specific in Prompts**: Clear questions get better answers
3. **Request Timestamps**: Ask for temporal references when needed
4. **Use Structured Output**: JSON schema for consistent parsing
5. **Lower Temperature**: 0.2 for factual analysis, higher for creative

## Streaming Support

Pegasus supports streaming via `InvokeModelWithResponseStream`:

```bash
aws bedrock-runtime invoke-model-with-response-stream \
  --model-id "us.twelvelabs.pegasus-1-2-v1:0" \
  --body '{"inputPrompt":"Describe this video","mediaSource":{"s3Location":{"uri":"s3://bucket/video.mp4"}}}' \
  --region us-west-2
```

## Sample Files

| File | Description |
|------|-------------|
| `twelvelabs_pegasus_video_s3_request.json` | Request with S3 video URI |
| `twelvelabs_pegasus_video_base64_request.json` | Request with base64 video |
| `twelvelabs_pegasus_video_response.json` | Analysis response |

## Related Model: Marengo Embed

TwelveLabs also offers **Marengo Embed v2.7** for video/image embeddings:
- Model ID: `twelvelabs.marengo-embed-2-7-v1:0`
- Use case: Video similarity search, clustering
- See existing samples: `twelvelabs_marengo_*` files

## References

- [AWS Bedrock TwelveLabs Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-pegasus.html)
- [TwelveLabs Pegasus Documentation](https://docs.twelvelabs.io/docs/cloud-partner-integrations/amazon-bedrock)
- [TwelveLabs API Reference](https://docs.twelvelabs.io/reference)
