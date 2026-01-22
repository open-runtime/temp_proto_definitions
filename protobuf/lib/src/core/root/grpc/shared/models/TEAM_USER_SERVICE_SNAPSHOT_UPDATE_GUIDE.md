# Team User Service Snapshot Update Guide

## Overview

This document describes the complete workflow for updating protobuf definitions based on changes to the user-team-service `/aot` endpoint. The process involves capturing new API response snapshots, analyzing changes, and updating the appropriate proto files.

**Canonical Example:** AWS Bedrock Application Inference Profiles (January 2026)

---

## 1. Capturing New Snapshots

### Prerequisites

- OpenVPN connection (required for accessing GCP services)
- Valid Descope account (e.g., `tsavo@pieces.app`)
- Dart SDK installed

### Fetch Scripts Location

| Script | Purpose | Location |
|--------|---------|----------|
| `fetch_user_profile_json.dart` | Fetch `/aot` user profile | `packages/aot/machine_learning/parents/aws_bedrock/examples/` |
| `fetch_bedrock_secrets_json.dart` | Fetch GCP secrets | `packages/aot/machine_learning/parents/aws_bedrock/examples/` |

### Running the Fetch Script

```bash
cd packages/aot/machine_learning/parents/aws_bedrock
dart run examples/fetch_user_profile_json.dart
```

### Authentication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Script starts                                                │
│    └── createAuthenticatedMetadata() called                     │
│                                                                 │
│ 2. Descope OAuth (browser-based)                               │
│    └── Browser window opens automatically                       │
│    └── User logs in with Pieces account                        │
│    └── OAuth callback to localhost:8080                        │
│                                                                 │
│ 3. Token Exchange                                               │
│    └── Access token retrieved from Descope                      │
│    └── Token stored for API calls                              │
│                                                                 │
│ 4. API Call                                                     │
│    └── GET https://user-team-service-*.run.app/aot             │
│    └── Header: Authorization: Bearer <access_token>            │
│                                                                 │
│ 5. Response                                                     │
│    └── Full JSON user profile printed to console               │
│    └── Copy and save to snapshot file                          │
└─────────────────────────────────────────────────────────────────┘
```

### Saving the Snapshot

1. Copy the JSON output from the console (the `/aot` endpoint response)
2. Save to: `packages/definitions/protobuf/lib/src/core/root/grpc/shared/models/team_user_service_profile_snapshot_samples/`
3. Use naming convention: `M_D_YYYY.json` (e.g., `1_14_2026.json`)

---

## 2. Snapshot File Conventions

### Naming Format

```
{month}_{day}_{year}.json
```

Examples:
- `1_14_2026.json` - January 14, 2026
- `12_3_2025.json` - December 3, 2025
- `11_25_2025.json` - November 25, 2025

### Storage Location

```
packages/definitions/protobuf/lib/src/core/root/grpc/shared/models/
└── team_user_service_profile_snapshot_samples/
    ├── PRE_11_25_2025.json     # Legacy format
    ├── 11_25_2025.json         # First modelAccess expansion
    ├── 12_3_2025.json          # Added Azure config
    ├── 12_3_2025_WITH_AZURE.json
    └── 1_14_2026.json          # Added inference_profile_models
```

### Purpose

These snapshots serve as:
1. **Version control** for API response evolution
2. **Reference data** for proto field definitions
3. **Test fixtures** for parsing logic
4. **Documentation** of JSON structure examples

---

## 3. Analyzing JSON for Proto Updates

### Comparison Process

1. Open the new snapshot alongside the previous one
2. Use a diff tool or manual comparison
3. Look for:
   - New top-level fields
   - New nested objects
   - New array structures
   - New map structures (key → value)
   - Changed field types

### Example: Discovering `inference_profile_models`

**Previous snapshot (12_3_2025.json):**
```json
{
  "api_keys": [{
    "name": "BedrockAPIKey-jeft-at-605134436431",
    "key": "...",
    "metadata": {...}
  }]
}
```

**New snapshot (1_14_2026.json):**
```json
{
  "api_keys": [{
    "name": "BedrockAPIKey-jeft-at-605134436431",
    "key": "...",
    "metadata": {...},
    "inference_profile_models": {                    // NEW!
      "anthropic.claude-sonnet-4-5-20250929-v1:0": [
        {
          "inference_profile_id": "5o63oo8uzo4m",
          "inference_profile_arn": "arn:aws:bedrock:...",
          "inference_profile_name": "developer-team-claude-sonnet-4-5",
          ...
        }
      ]
    }
  }]
}
```

### Identifying Data Sources

New fields may come from:
1. **User-team-service `/aot`** - Primary source (snake_case JSON keys)
2. **Direct AWS Bedrock API** - Secondary enrichment (camelCase JSON keys)
3. **AWS CLI output** - Reference data for additional fields

---

## 4. Protobuf Update Strategy

### Decision Flowchart

```
New field discovered in /aot response
           │
           ▼
    ┌──────────────────┐
    │ Where does the   │
    │ data belong?     │
    └────────┬─────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐
│ Provider│    │ User Profile│
│ Specific│    │ Level       │
└────┬────┘    └──────┬──────┘
     │                │
     ▼                ▼
models.proto    team_user_service_profile_snapshot.proto
(aws_bedrock)   (core)
```

### File Locations

| Proto File | Purpose | Location |
|------------|---------|----------|
| `models.proto` | AWS Bedrock specific types | `packages/definitions/protobuf/lib/src/machine_learning/parents/aws_bedrock/root/shared/models/` |
| `team_user_service_profile_snapshot.proto` | User profile structure | `packages/definitions/protobuf/lib/src/core/root/grpc/shared/models/` |

### Update Steps

1. **Define new enums/messages** in the provider-specific proto (e.g., `models.proto`)
2. **Add fields** to existing messages that need the new data
3. **Reference types** from `team_user_service_profile_snapshot.proto` if needed
4. **Run proto generation** to verify syntax
5. **Check for lint errors**

---

## 5. Naming Conventions

### Pattern: `AWSBedrock[Resource][Property/Representation]`

| Type | Pattern | Example | Rationale |
|------|---------|---------|-----------|
| Generic Attribute | `[Provider][Resource][Property]` | `AWSBedrockInferenceProfileStatus` | Profile-level attribute, reusable |
| Protobuf Model | `[Provider][Resource]Model` | `AWSBedrockInferenceProfileModel` | Our representation of the resource |
| Nested Model Type | `[Provider][Resource]Model[Subtype]` | `AWSBedrockInferenceProfileModelVariant` | Specifically model-related |

### Why This Matters

```protobuf
// CORRECT: Generic profile attributes (no "Model")
// Rationale: These describe properties OF an inference profile,
// not our protobuf representation. Enables reuse across different
// profile representations (Summary, Details, etc.)
enum AWSBedrockInferenceProfileStatus { ... }
enum AWSBedrockInferenceProfileType { ... }
message AWSBedrockInferenceProfileTag { ... }

// CORRECT: Our protobuf model (has "Model")
// Rationale: This IS our data model representing the profile entity
message AWSBedrockInferenceProfileModel { ... }

// CORRECT: Nested model-specific type (has "Model")
// Rationale: Specifically represents a foundation model ARN in a region
message AWSBedrockInferenceProfileModelVariant { ... }
```

### Documentation in Proto Files

Add naming convention comments in the proto file section header:

```protobuf
// NAMING CONVENTION:
// - AWSBedrockInferenceProfile[Property] (e.g., Status, Type, Tag) - Generic profile attributes
//   These DO NOT include "Model" because they describe properties OF an inference profile,
//   not our protobuf representation of one. This allows reuse if we add different profile
//   representations (e.g., Summary vs Details) or if AWS expands the API.
//
// - AWSBedrockInferenceProfileModel - Our protobuf model/representation of an inference profile
//   Includes "Model" because it's our data model representing the full profile entity.
//
// - AWSBedrockInferenceProfileModelVariant - A regional foundation model variant within a profile
//   Includes "Model" because it specifically represents a model ARN in a region.
```

---

## 6. json_name Mapping Strategy

### Two JSON Formats

| Source | Format | Example |
|--------|--------|---------|
| User-team-service `/aot` | snake_case | `inference_profile_id`, `model_id` |
| Direct AWS Bedrock API | camelCase | `inferenceProfileId`, `modelArn` |

### When to Use json_name

| Scenario | Use json_name? | Example |
|----------|---------------|---------|
| Field maps to `/aot` response | YES | `[json_name = "inference_profile_id"]` |
| Field maps to AWS API response | YES | `[json_name = "modelArn"]` |
| Field is parsed/extracted | NO | `string region = 2;` (extracted from ARN) |
| Field is a request parameter | NO | `string inference_profile_arn = 6;` |

### Documentation Pattern

Always document which JSON source the `json_name` maps to:

```protobuf
// json_name maps to user-team-service /aot response format
string inference_profile_id = 1 [json_name = "inference_profile_id"];

// json_name maps to AWS Bedrock API "modelArn" field
string model_variant_arn = 1 [json_name = "modelArn"];

// Populated by parsing the ARN - not present in raw API response
string region = 2;
```

### Request Fields

Request fields going TO an API (not coming FROM a response) should NOT have `json_name`:

```protobuf
message AWSBedrockInferenceRequest {
  // These are request parameters, not response mappings
  string inference_profile_arn = 6;    // NO json_name
  string inference_profile_id = 7;     // NO json_name
  string inference_profile_name = 8;   // NO json_name
}
```

---

## 7. Complete Example: AWS Bedrock Inference Profiles

### Changes Made (January 14, 2026)

#### New Enums in `models.proto`

```protobuf
enum AWSBedrockInferenceProfileStatus {
  AWS_BEDROCK_INFERENCE_PROFILE_STATUS_UNSPECIFIED = 0;
  AWS_BEDROCK_INFERENCE_PROFILE_STATUS_ACTIVE = 1;
  AWS_BEDROCK_INFERENCE_PROFILE_STATUS_CREATING = 2;
}

enum AWSBedrockInferenceProfileType {
  AWS_BEDROCK_INFERENCE_PROFILE_TYPE_UNSPECIFIED = 0;
  AWS_BEDROCK_INFERENCE_PROFILE_TYPE_APPLICATION = 1;
  AWS_BEDROCK_INFERENCE_PROFILE_TYPE_SYSTEM_DEFINED = 2;
}
```

#### New Messages in `models.proto`

```protobuf
message AWSBedrockInferenceProfileTag {
  string key = 1;
  string value = 2;
}

message AWSBedrockInferenceProfileModelVariant {
  string model_variant_arn = 1 [json_name = "modelArn"];
  string region = 2;
  string model_id = 3;
  string partition = 4;
}

message AWSBedrockInferenceProfileModel {
  string inference_profile_id = 1 [json_name = "inference_profile_id"];
  string inference_profile_arn = 2 [json_name = "inference_profile_arn"];
  string inference_profile_name = 3 [json_name = "inference_profile_name"];
  string model_id = 4 [json_name = "model_id"];
  string model_name = 5 [json_name = "model_name"];
  string description = 6;
  google.protobuf.Timestamp created_at = 7 [json_name = "created_at"];
  google.protobuf.Timestamp updated_at = 8 [json_name = "updated_at"];
  AWSBedrockInferenceProfileStatus status = 9;
  AWSBedrockInferenceProfileType type = 10;
  repeated AWSBedrockInferenceProfileModelVariant model_variants = 11 [json_name = "models"];
  repeated AWSBedrockInferenceProfileTag tags = 12;
  int32 model_count = 13 [json_name = "model_count"];
}

message AWSBedrockInferenceProfileModels {
  repeated AWSBedrockInferenceProfileModel all = 1;
}
```

#### Updated Messages

**`AWSBedrockApiKey` (models.proto):**
```protobuf
message AWSBedrockApiKey {
  string name = 1;
  string bearer_token = 2 [json_name = "key"];
  AWSBedrockApiKeyMetadata metadata = 3;
  // NEW: Map of model_id to available inference profiles
  map<string, AWSBedrockInferenceProfileModels> inference_profile_models = 4 [json_name = "inference_profile_models"];
}
```

**`AWSBedrockInferenceRequest` (models.proto):**
```protobuf
message AWSBedrockInferenceRequest {
  // ... existing fields ...
  
  // NEW: Inference profile selection
  AWSBedrockInferenceProfileModel selected_inference_profile = 5;
  string inference_profile_arn = 6;
  string inference_profile_id = 7;
  string inference_profile_name = 8;
}
```

**`TeamUserServiceProfileOrganizationModelAccess` (team_user_service_profile_snapshot.proto):**
```protobuf
message TeamUserServiceProfileOrganizationModelAccess {
  // ... existing fields 1-14 ...
  
  // NEW: Link to inference profiles
  repeated runtime.aot.machine_learning.parents.aws_bedrock.AWSBedrockInferenceProfileModel inference_profile_models = 15 [json_name = "inference_profile_models"];
}
```

### Data Flow

```
┌────────────────────────────────────────────────────────────────────┐
│ models.proto (aws_bedrock)                                         │
│                                                                    │
│ AWSBedrockInferenceProfileModel ◄─── AWSBedrockInferenceProfileTag │
│         │                       ◄─── AWSBedrockInferenceProfileModelVariant
│         │                       ◄─── AWSBedrockInferenceProfileStatus
│         │                       ◄─── AWSBedrockInferenceProfileType
│         │                                                          │
│         ▼                                                          │
│ AWSBedrockInferenceProfileModels                                   │
│         │                                                          │
│         ▼                                                          │
│ AWSBedrockApiKey.inference_profile_models                          │
│         │                                                          │
│         ▼                                                          │
│ AWSBedrockCredentialsConfig                                        │
└────────┬───────────────────────────────────────────────────────────┘
         │
         │ (imported into)
         ▼
┌────────────────────────────────────────────────────────────────────┐
│ team_user_service_profile_snapshot.proto                           │
│                                                                    │
│ TeamUserServiceProfileOrganizationConfiguration.bedrock            │
│         │                                                          │
│         ▼                                                          │
│ TeamUserServiceProfileOrganizationModelAccess.inference_profile_models
└────────────────────────────────────────────────────────────────────┘
```

---

## 8. Validation Checklist

After making proto changes:

- [ ] Run protobuf generation: `melos run generate:protos` (or equivalent)
- [ ] Check for lint errors in proto files
- [ ] Verify imports are correct (especially cross-package references)
- [ ] Test JSON parsing with the new snapshot
- [ ] Update snapshot samples if format changed significantly
- [ ] Document naming decisions in proto file comments

---

## Related Files

| File | Purpose |
|------|---------|
| `JSON_NAME_MIGRATION_PLAN.md` | Strategy for json_name field options |
| `team_user_service_profile_snapshot_samples/*.json` | Historical API responses |
| `packages/aot/core/lib/grpc/shared/interceptors/event_sourcing/per_user_key_auth/` | Runtime code that calls `/aot` |

---

## Changelog

| Date | Author | Changes |
|------|--------|---------|
| 2026-01-14 | tsavo@pieces.app | Initial document, AWS Bedrock Inference Profiles example |
