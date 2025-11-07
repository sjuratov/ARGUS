# Task 009: Dataset Configuration Management System

## Description
Dynamic dataset configuration system allowing runtime management of document processing settings without redeployment. Stores dataset-specific system prompts, JSON schemas, chunking parameters, and processing options in Cosmos DB, enabling multi-tenant document processing with different extraction rules per dataset.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core

## Technical Requirements

- **Cosmos DB Configuration Container**: Dedicated container for storing configuration documents
- **Dataset-Based Organization**: Configuration grouped by dataset name (folder structure)
- **System Prompts**: Custom extraction instructions per dataset
- **JSON Schemas**: Output structure definitions per dataset
- **Processing Options**: Per-dataset flags for OCR, images, summary, evaluation
- **Chunking Parameters**: Configurable max_pages_per_chunk per dataset
- **Demo Dataset Initialization**: Auto-load default configurations from demo folder on first run
- **Configuration CRUD Operations**: Read, update, refresh configurations via API
- **Fallback Mechanism**: Default to first available dataset if requested dataset not found
- **Legacy Structure Support**: Handle old flat configuration format for backward compatibility

**Configuration Document Structure:**

```json
{
  "id": "configuration",
  "partitionKey": "configuration",
  "datasets": {
    "dataset-name": {
      "model_prompt": "string",
      "example_schema": {},
      "max_pages_per_chunk": 10,
      "processing_options": {
        "include_ocr": true,
        "include_images": true,
        "enable_summary": true,
        "enable_evaluation": true
      }
    }
  }
}
```

## Implementation Evidence

- `src/containerapp/ai_ocr/process.py` - Configuration loading and management (641 lines)
  - Lines 282-400: `fetch_model_prompt_and_schema` function
    - Lines 286-290: Check if configuration exists in Cosmos DB
    - Lines 292-348: Auto-initialize from demo folder if not exists
    - Lines 350-385: Parse configuration (supports datasets key or legacy flat structure)
    - Lines 387-400: Extract dataset-specific prompt, schema, and options
  - Demo folder scanning and system_prompt.txt/output_schema.json loading
  - Validation of dataset configuration structure
- `src/containerapp/api_routes.py` - Configuration API endpoints (651 lines)
  - Lines 130-177: `get_configuration` endpoint
    - Retrieves configuration document from Cosmos DB
    - Handles ResourceNotFoundError gracefully
  - Lines 179-198: `update_configuration` endpoint
    - Updates entire configuration document
    - Validates request body before persistence
  - Lines 200-220: `refresh_configuration` endpoint
    - Forces reload of configuration
    - Re-initializes from demo folder if needed
- `frontend/process_files.py` - Frontend configuration management (369 lines)
  - Lines 70-110: `fetch_configuration` function
    - Reads from Cosmos DB config container
    - Creates default configuration if not exists
  - Lines 112-200: Dataset selection UI
  - Lines 302-369: Configuration editor for prompts and schemas

**Demo Dataset Locations:**

- `demo/default-dataset/` - Default dataset with system_prompt.txt and output_schema.json
- `demo/medical-dataset/` - Medical document dataset configuration
- `demo/mistral-dataset/` - Mistral-specific dataset configuration
- `src/containerapp/datasets/default-dataset/` - Embedded default dataset
- `src/containerapp/example-datasets/` - Example dataset templates

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ Configuration stored in Cosmos DB with id="configuration" (process.py line 286)
- ✅ Demo datasets auto-loaded on first run if configuration missing (process.py lines 292-348)
- ✅ System prompts loaded from system_prompt.txt files (process.py lines 315-322)
- ✅ JSON schemas loaded from output_schema.json files (process.py lines 324-331)
- ✅ Dataset-specific configurations organized under 'datasets' key (process.py lines 356-358)
- ✅ Legacy flat configuration structure supported for backward compatibility (process.py lines 360-362)
- ✅ Fallback to first available dataset if requested dataset not found (process.py lines 376-381)
- ✅ Processing options have default values if not specified (process.py lines 395-400)
- ✅ max_pages_per_chunk defaults to 10 if not configured (process.py line 393)
- ✅ Configuration validated before use (model_prompt and example_schema required) (process.py lines 383-392)
- ✅ Configuration updates persisted via API endpoint (api_routes.py lines 179-198)
- ✅ Configuration refresh forces reload (api_routes.py lines 200-220)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation of JSON schema correctness (only checks if present)
- No versioning of configuration changes (no audit trail)
- No validation of processing options structure
- No default dataset specified (uses first in alphabetical order)
- Demo folder path hardcoded relative to script location (may fail in containers)
- No validation that system_prompt is not empty
- Configuration update doesn't validate against active processing jobs
- No rollback mechanism for bad configurations

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for configuration loading
- No tests for demo dataset initialization
- No tests for legacy configuration migration
- No tests for dataset fallback logic
- No tests for configuration validation
- No integration tests with Cosmos DB
- No tests for API endpoints
- **Coverage:** 0%

**Manual Testing Evidence:**

- Frontend provides configuration management UI (process_files.py)
- Demo datasets included for testing (demo/ directory)
- README.md documents dataset configuration (lines 322-346)

**Recommendations:**

- Add unit tests for configuration loading with mocked Cosmos DB
- Test demo dataset initialization with various folder structures
- Test legacy configuration migration
- Test dataset fallback scenarios
- Add validation tests for configuration structure
- Test configuration updates with invalid data
- Add integration tests with real Cosmos DB
- Test concurrent configuration updates
- Add tests for demo folder path resolution in different environments
- Test configuration versioning and audit trail
