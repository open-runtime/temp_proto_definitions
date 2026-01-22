# JSON Name Migration Plan

## Overview

This document outlines the plan to use `json_name` field options in proto files to simplify runtime field name normalization. 

**Status:** ✅ **COMPLETED** - All `json_name` options added, protobufs regenerated, tests passing (17/17)

## Current State

- **API Response Format**: Mixed camelCase and snake_case
  - camelCase: `userId`, `createdTime`, `loginIds`, `isVerifiedEmail`
  - snake_case: `user_id`, `created_at`, `login_ids`, `is_verified_email`
  
- **Proto Fields**: All snake_case (`user_id`, `created_at`)

- **Protobuf Default**: Converts snake_case → camelCase for JSON (`userId`, `createdAt`)

- **Current Solution**: Runtime normalization in `TeamUserServiceProfileSnapshotHelpers` that converts snake_case → camelCase

## Proposed Solution

Add `json_name` options to proto fields to match the API's snake_case format. This makes field mappings declarative in proto files and simplifies the normalization logic.

**Note:** When `json_name` is specified, protobuf's `mergeFromProto3Json()` prioritizes that exact format. While `supportNamesWithUnderscores=true` accepts both formats by default, `json_name` overrides this behavior. Therefore, we still need minimal camelCase → snake_case normalization for fields that come as camelCase from the API.

**Benefits:**
1. ✅ Field mappings are declarative (in proto files)
2. ✅ Reduced normalization code (~200 lines removed, ~30 lines remain for camelCase → snake_case)
3. ✅ Timestamp normalization still needed (Unix → RFC 3339)
4. ✅ Type conversions still needed (string → int for `quantity`)

## Fields That Need `json_name`

Based on the API response, these fields come as snake_case and should get `json_name`:

### Top-level fields:
- `user_id` → `[json_name = "user_id"]` (API also sends `userId`)
- `autho_id` → `[json_name = "autho_id"]`
- `given_name` → `[json_name = "given_name"]`
- `middle_name` → `[json_name = "middle_name"]`
- `family_name` → `[json_name = "family_name"]`
- `paddle_id` → `[json_name = "paddle_id"]`
- `paddle_last_webhook_at` → `[json_name = "paddle_last_webhook_at"]`
- `created_at` → `[json_name = "created_at"]` (API also sends `createdTime`)
- `updated_at` → `[json_name = "updated_at"]` (API also sends `updatedTime`)
- `login_ids` → `[json_name = "login_ids"]` (API also sends `loginIds`)
- `role_names` → `[json_name = "role_names"]`
- `sso_app_ids` → `[json_name = "sso_app_ids"]`
- `is_verified_email` → `[json_name = "is_verified_email"]` (API also sends `isVerifiedEmail`)
- `is_verified_phone` → `[json_name = "is_verified_phone"]` (API also sends `isVerifiedPhone`)
- `welcome_email` → `[json_name = "welcome_email"]`
- `cloud_key` → `[json_name = "cloud_key"]`
- `open_ai` → `[json_name = "open_ai"]`
- `api_keys` → `[json_name = "api_keys"]` (API also sends `apiKeys`)

### Organization fields:
- `paddle_id` → `[json_name = "paddle_id"]`
- `tenant_id` → `[json_name = "tenant_id"]`
- `created_time` → `[json_name = "createdTime"]` (API sends camelCase)
- `updated_time` → `[json_name = "updatedTime"]` (API sends camelCase)

### Subscription fields:
- `product_id` → `[json_name = "product_id"]`
- `price_id` → `[json_name = "price_id"]`
- `subscription_id` → `[json_name = "subscription_id"]`
- `billing_term` → `[json_name = "billing_term"]`
- `created_at` → `[json_name = "created_at"]`
- `nextbilled_at` → `[json_name = "nextbilled_at"]`
- `paused_at` → `[json_name = "paused_at"]`
- `canceled_at` → `[json_name = "canceled_at"]`

### Bedrock credentials (AWSBedrockCredentialsConfig):
- `access_key_credentials` → `[json_name = "access_key_credentials"]`
- `api_keys` → `[json_name = "api_keys"]`
- `is_stored` → `[json_name = "is_stored"]`
- `updated_at` → `[json_name = "updated_at"]`

### Bedrock API Key (AWSBedrockApiKey):
- `bearer_token` → `[json_name = "key"]` ⚡ **CRITICAL CHANGE**
  - Both user-team-service and inference service APIs use `"key"`
  - Proto Dart property: `bearerToken` (clear intent)
  - JSON wire format: `"key"` (matches external APIs)
  - **Eliminated** unnecessary key ↔ bearer_token conversions in extensions
  - **Result**: Cleaner code, consistent format across all systems

### Bedrock Access Key Credential:
- `access_key` → `[json_name = "access_key"]`
- `secret_key` → `[json_name = "secret_key"]`
- **Note**: Profile sends `key`/`secret`, extensions normalize to `access_key`/`secret_key`

## Implementation Steps

1. **Add `json_name` to proto files**
   - Update `team_user_service_profile_snapshot.proto`
   - Update `models.proto` (AWSBedrockCredentialsConfig)

2. **Regenerate protobufs**
   ```bash
   melos delete:generated:protobufs
   melos generate:protobufs
   ```

3. **Simplify normalization code**
   - ✅ Removed old snake_case → camelCase mappings (~200 lines)
   - ⚠️ Still need camelCase → snake_case normalization (~30 lines) because `json_name` prioritizes exact format
   - ✅ Keep timestamp normalization (Unix → RFC 3339)
   - ✅ Keep type conversions (string → int)

4. **Update tests**
   - Verify tests still pass with `json_name`
   - Test that both camelCase and snake_case are accepted

## Benefits

- **Less Code**: Removed ~200 lines of old field mapping logic (snake_case → camelCase)
- **Declarative**: Field mappings in proto files (single source of truth via `json_name`)
- **Maintainable**: Changes to API format only require proto updates
- **Performance**: Reduced runtime transformation (still need minimal camelCase → snake_case normalization)
- **Type Safety**: Fixed proto type mismatches (e.g., `welcome_email` changed from `string` to `bool`)

## Considerations

- **Mixed Formats**: API sends both camelCase and snake_case - we normalize camelCase → snake_case to match `json_name` values
- **`json_name` Behavior**: When `json_name` is specified, `mergeFromProto3Json()` prioritizes that exact format, so normalization is still needed
- **Backward Compatibility**: Protobuf parsers accept both formats via `supportNamesWithUnderscores`, but `json_name` overrides this
- **Timestamp Normalization**: Still needed (Unix → RFC 3339) for `google.protobuf.Timestamp` fields
- **Type Conversions**: Still needed (string → int for `quantity` when API sends strings)
- **Proto Type Fixes**: Fixed `welcome_email` from `string` to `bool` to match API type

## Testing Strategy

1. ✅ Test with real API response (mixed camelCase and snake_case)
2. ✅ Test with camelCase JSON (normalized to snake_case)
3. ✅ Verify timestamp normalization still works
4. ✅ Verify type conversions still work
5. ✅ Run all existing tests (17/17 passing)

## Implementation Status

### ✅ Completed
- Added `json_name` options to all relevant fields in proto files
- Regenerated protobufs with `melos`
- Simplified normalization code (removed ~200 lines of old mappings)
- Fixed proto type mismatches (`welcome_email`: `string` → `bool`)
- Updated all tests to pass with `json_name`
- Updated documentation to reflect camelCase → snake_case normalization

### ⚠️ Current State
- Still need ~30 lines of camelCase → snake_case normalization because `json_name` prioritizes exact format
- Using `mergeFromProto3Json()` for deserialization (correct method)
- Using `toProto3Json()` for serialization (correct method)
- All tests passing (17/17)

### 📝 Methods Used
- **Deserialization**: `mergeFromProto3Json(Map<String, dynamic>)` - Proto3 JSON format
  - Uses `supportNamesWithUnderscores=true` by default (accepts both formats)
  - When `json_name` is specified, it prioritizes that exact format
- **Serialization**: `toProto3Json()` - Proto3 JSON format with `json_name` support
  - Uses `json_name` values when specified, otherwise defaults to camelCase
- Both methods respect `json_name` options specified in proto files

### 📊 Code Metrics
- **Before**: ~850 lines (with extensive field name mappings)
- **After**: ~653 lines (simplified normalization)
- **Reduction**: ~200 lines removed, ~30 lines remain for camelCase → snake_case normalization

