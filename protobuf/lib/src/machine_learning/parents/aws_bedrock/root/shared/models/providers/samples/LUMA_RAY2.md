# Luma AI Ray v2 on AWS Bedrock

## Overview

Luma AI Ray v2 (`luma.ray-v2:0`) is a state-of-the-art video generation model available on AWS Bedrock. It generates high-quality videos from text prompts using the **async inference API**.

## Model Details

| Property | Value |
|----------|-------|
| Model ID | `luma.ray-v2:0` |
| Provider | Luma AI |
| Input | TEXT |
| Output | VIDEO (MP4) |
| Streaming | Not supported |
| Inference Type | ON_DEMAND |
| Region | **us-west-2 only** |

## Key Features

- **Text-to-Video Generation**: Create videos from text prompts
- **Multiple Aspect Ratios**: 16:9, 9:16, 1:1
- **Variable Durations**: 5s or 9s videos
- **Resolution Options**: 540p, 720p
- **Loop Support**: Generate seamlessly looping videos
- **Cinematic Quality**: High-fidelity video output (5-50 MB)

## API: StartAsyncInvoke (Required)

Luma Ray v2 uses **async inference** only - synchronous `InvokeModel` is not supported.

### Request Format

```json
{
  "modelId": "luma.ray-v2:0",
  "modelInput": {
    "prompt": "Your detailed video description",
    "aspect_ratio": "16:9",
    "loop": false,
    "duration": "5s",
    "resolution": "720p"
  },
  "outputDataConfig": {
    "s3OutputDataConfig": {
      "s3Uri": "s3://your-bucket-name/output-path/"
    }
  }
}
```

### modelInput Parameters

| Parameter | Type | Required | Default | Values |
|-----------|------|----------|---------|--------|
| `prompt` | string | **Yes** | - | Text description |
| `aspect_ratio` | string | No | "16:9" | "16:9", "9:16", "1:1" |
| `loop` | boolean | No | false | true, false |
| `duration` | string | No | "5s" | "5s", "9s" |
| `resolution` | string | No | "720p" | "540p", "720p" |

### Response Format

```json
{
  "invocationArn": "arn:aws:bedrock:us-west-2:123456789012:async-inference-job/abc123...",
  "status": "IN_PROGRESS"
}
```

## Async Workflow

### 1. Start Job

```bash
aws bedrock-runtime start-async-invoke \
  --model-id "luma.ray-v2:0" \
  --model-input '{"prompt":"A cat playing in sunlight"}' \
  --output-data-config '{"s3OutputDataConfig":{"s3Uri":"s3://bucket/path/"}}' \
  --region us-west-2
```

### 2. Poll Status

```bash
aws bedrock-runtime get-async-invoke \
  --invocation-arn "arn:aws:bedrock:us-west-2:123456:async-inference-job/xyz..." \
  --region us-west-2
```

### 3. Download Result

When `status` is `COMPLETED`, retrieve the MP4 file from your S3 bucket.

## Job Status Values

| Status | Description |
|--------|-------------|
| `IN_PROGRESS` | Video generation in progress |
| `COMPLETED` | Video ready in S3 bucket |
| `FAILED` | Generation failed (check error details) |
| `STOPPED` | Job was cancelled |

## Completed Job Response

```json
{
  "invocationArn": "arn:aws:bedrock:us-west-2:123456:async-inference-job/abc...",
  "modelArn": "arn:aws:bedrock:us-west-2::foundation-model/luma.ray-v2:0",
  "status": "COMPLETED",
  "submitTime": "2026-01-20T16:30:00.000Z",
  "endTime": "2026-01-20T16:32:15.000Z",
  "outputDataConfig": {
    "s3OutputDataConfig": {
      "s3Uri": "s3://your-bucket-name/output-path/"
    }
  }
}
```

## Prerequisites

### 1. Enable Model Access

Navigate to AWS Bedrock Console → Model Access → Enable `luma.ray-v2:0`

### 2. S3 Bucket Setup

Create an S3 bucket with Bedrock write permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "bedrock.amazonaws.com"
      },
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

### 3. IAM Permissions

Ensure your role has:
- `bedrock:InvokeModelWithResponseStream`
- `bedrock:StartAsyncInvoke`
- `bedrock:GetAsyncInvoke`
- `s3:PutObject`
- `s3:GetObject`

## Prompt Best Practices

1. **Be Descriptive**: Include details about scene, lighting, movement
2. **Specify Style**: "cinematic", "anime", "documentary", etc.
3. **Describe Motion**: "slowly panning", "zooming in", "camera follows"
4. **Set Atmosphere**: "golden hour", "dramatic lighting", "foggy morning"

### Example Prompts

- "A drone shot flying over a futuristic city at sunset, neon lights reflecting on wet streets, cinematic"
- "A golden retriever running through a field of sunflowers, slow motion, warm summer light"
- "An astronaut floating in space, Earth visible in background, camera slowly rotating"

## Pricing & Performance

- **Generation Time**: 2-5 minutes typically
- **Output Size**: 5-50 MB depending on duration and resolution
- **Cost**: Per-video pricing (check AWS Bedrock pricing page)

## Sample Files

| File | Description |
|------|-------------|
| `luma_ray2_start_async_request.json` | StartAsyncInvoke request body |
| `luma_ray2_start_async_response.json` | Initial response with invocationArn |
| `luma_ray2_get_async_completed_response.json` | Completed job response |

## References

- [AWS Bedrock Luma Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-luma.html)
- [Luma AI Documentation](https://lumalabs.ai/docs)
- [AWS Bedrock Async Invoke](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_StartAsyncInvoke.html)
