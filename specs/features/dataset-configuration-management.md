# Dataset Configuration Management

## Overview
Dataset Configuration Management enables runtime control of document processing behavior through dataset-specific configurations stored in Cosmos DB. Users can define custom system prompts, JSON schemas, PDF chunking parameters, and processing options per dataset without code changes or redeployment, supporting multi-tenant document processing with different extraction rules for different document types.

**Business Purpose**: Enable business users and administrators to configure and customize extraction behavior for different document types without technical deployment, allowing the same platform to process medical forms, invoices, legal documents, and other document types with appropriate extraction rules.

## User Stories

Based on implemented functionality:

- **As a business administrator**, I want to configure extraction rules per dataset, so that different document types can be processed appropriately
- **As a user**, I want to define custom JSON schemas, so that extracted data matches my business requirements
- **As a prompt engineer**, I want to update system prompts without redeployment, so that I can iterate on extraction quality
- **As a system administrator**, I want processing options configurable per dataset, so that resource-intensive features can be selectively enabled
- **As a developer**, I want demo configurations auto-loaded, so that new deployments work out-of-the-box
- **As a user**, I want dataset-specific chunking settings, so that large document processing optimizes per document type

## Functional Requirements

### Dataset-Based Configuration Structure

- Configuration stored in Cosmos DB with id="configuration"
- Organized under 'datasets' key with dataset names as sub-keys
- Each dataset configuration includes:
  - `model_prompt`: Custom system prompt for extraction instructions
  - `example_schema`: JSON schema defining expected output structure
  - `max_pages_per_chunk`: PDF chunking parameter (default: 10)
  - `processing_options`: Flags controlling processing stages
    - `include_ocr`: Enable/disable OCR processing
    - `include_images`: Enable/disable image processing
    - `enable_summary`: Enable/disable document summarization
    - `enable_evaluation`: Enable/disable quality evaluation
- Configuration shared across all documents in same dataset

### Automatic Demo Dataset Initialization

- On first run, scans demo folder for available datasets
- Loads configurations from demo folder structure:
  - `demo/{dataset-name}/system_prompt.txt`
  - `demo/{dataset-name}/output_schema.json`
- Creates initial configuration document in Cosmos DB if missing
- Supports multiple demo datasets (default-dataset, medical-dataset, mistral-dataset)
- Enables out-of-the-box functionality without manual configuration

### Configuration CRUD Operations

**Read Configuration:**

- GET `/api/configuration` endpoint retrieves full configuration document
- Configuration loaded at processing time per document
- Graceful handling of missing configuration (ResourceNotFoundError)

**Update Configuration:**

- POST `/api/configuration` endpoint updates entire configuration
- Validates request body before persistence
- Changes immediately effective for new document processing
- Frontend UI provides configuration editor

**Refresh Configuration:**

- POST `/api/configuration/refresh` endpoint forces reload
- Re-initializes from demo folder if configuration cleared
- Useful for resetting to defaults or recovering from bad config

### Dataset Selection & Fallback

- Dataset extracted from document blob path (first folder in path)
- Defaults to 'default-dataset' if path doesn't indicate dataset
- If requested dataset not found in configuration, falls back to first available
- Validates configuration has required fields (model_prompt, example_schema)

### Legacy Configuration Support

- Supports old flat configuration structure for backward compatibility
- Automatically migrates from flat to datasets structure
- Ensures existing deployments continue functioning after upgrade

### Frontend Configuration Management

- Process Files tab shows dataset-specific configuration
- Configuration editor allows prompt and schema editing
- Dataset selection dropdown for uploads
- Visual feedback on configuration changes
- Dataset creation and management UI

## Acceptance Criteria

**Configuration Storage:**

- Given configuration document, when stored, then id="configuration" with partitionKey="configuration"
- Given multiple datasets, when storing, then all organized under 'datasets' key
- Given dataset configuration, when saved, then includes model_prompt, example_schema, max_pages_per_chunk, processing_options

**Demo Initialization:**

- Given first application run, when starting, then demo folder scanned for datasets
- Given demo datasets found, when initializing, then system_prompt.txt and output_schema.json loaded
- Given configuration missing, when accessing, then auto-created from demo folder
- Given multiple demo datasets, when initializing, then all loaded into configuration

**Configuration Retrieval:**

- Given GET /api/configuration called, when successful, then full configuration returned
- Given configuration missing, when GET called, then ResourceNotFoundError handled gracefully
- Given document processing, when loading config, then dataset-specific config retrieved

**Configuration Update:**

- Given POST /api/configuration called with valid data, when updating, then configuration persisted to Cosmos DB
- Given configuration updated, when new document processes, then new configuration used
- Given invalid configuration, when updating, then validation error returned

**Configuration Refresh:**

- Given POST /api/configuration/refresh called, when executing, then configuration reloaded
- Given configuration cleared, when refreshing, then re-initialized from demo folder

**Dataset Selection:**

- Given blob path "medical-dataset/form.pdf", when processing, then medical-dataset configuration loaded
- Given blob path "invoice.pdf", when processing, then default-dataset configuration loaded
- Given requested dataset missing, when loading, then first available dataset used
- Given configuration loaded, when validation occurs, then model_prompt and example_schema verified present

**Processing Options:**

- Given include_ocr=false, when processing, then OCR stage skipped
- Given enable_summary=true, when processing, then summary stage executes
- Given processing options not specified, when loading, then defaults used (all true)
- Given max_pages_per_chunk=20, when chunking PDF, then 20 pages per chunk

**Legacy Compatibility:**

- Given old flat configuration, when loading, then migrated to datasets structure automatically
- Given legacy config loaded, when processing, then extraction works correctly

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation of JSON schema correctness (only checks presence)
- No versioning of configuration changes (no audit trail)
- No validation that system_prompt is not empty
- Configuration updates don't validate against active processing jobs
- No rollback mechanism for bad configurations
- Demo folder path hardcoded relative to script location (may fail in containers)

## Non-Functional Requirements

**Flexibility:**

- Runtime configuration changes without redeployment
- Per-dataset customization enables multi-tenant use cases
- Processing options enable resource optimization per dataset

**Usability:**

- Auto-initialization from demo folder provides out-of-box functionality
- Frontend configuration editor makes updates accessible to non-technical users
- Clear dataset naming and organization

**Reliability:**

- Configuration validation before use (checks required fields)
- Fallback to available datasets prevents processing failures
- Legacy configuration support ensures backward compatibility
- ⚠️ RELIABILITY GAPS: No rollback for bad configurations

**Performance:**

- Configuration loaded once per document processing
- ⚠️ PERFORMANCE GAPS: No configuration caching (reloaded each time)
- Cosmos DB queries for configuration on each load

## Dependencies & Constraints

**Dependencies:**

- Requires Cosmos DB configuration container - Task 001
- Requires backend API for CRUD operations - Task 002
- Requires document processing pipeline to consume configuration - Task 005
- Requires frontend for configuration management UI - Task 007
- Requires demo datasets in file system

**Constraints:**

- Configuration shared across all documents in dataset (no per-document config)
- Demo folder path hardcoded (relative to application location)
- No configuration versioning or history
- No validation of JSON schema validity (syntax check only)
- No locking mechanism for concurrent configuration updates
- Changes affect all new processing immediately (no staged rollout)

**Technical Constraints:**

- Cosmos DB for storage (requires connectivity)
- JSON format for schemas and configuration
- Text file format for system prompts
- Python-based configuration loading

## Task Traceability

This feature is implemented by the following tasks:

- **Task 009**: Configuration Management System (`specs/tasks/009-task-configuration-management.md`) - Core configuration implementation
- **Task 002**: Backend FastAPI Core (`specs/tasks/002-task-backend-fastapi-core.md`) - Configuration API endpoints
- **Task 005**: Document Processing Pipeline (`specs/tasks/005-task-document-processing-pipeline.md`) - Configuration consumption
- **Task 007**: Streamlit Frontend (`specs/tasks/007-task-streamlit-frontend.md`) - Configuration management UI

## Implementation Status

**Status**: ✅ Complete / ⚠️ Partial

**Complete:**

- Cosmos DB-based configuration storage
- Dataset-based configuration structure
- Auto-initialization from demo folder
- Configuration loading from Cosmos DB
- Configuration CRUD API endpoints (GET, POST, refresh)
- Frontend configuration editor UI
- Dataset selection and fallback logic
- Legacy configuration migration support
- Processing options support
- max_pages_per_chunk configuration
- Required field validation (model_prompt, example_schema)

**Incomplete/Missing:**

- ⚠️ No JSON schema validation (syntax/structure checking)
- ⚠️ No configuration versioning or audit trail
- ⚠️ No validation that system_prompt is not empty
- ⚠️ No check against active processing jobs before updates
- ⚠️ No rollback mechanism for bad configurations
- ⚠️ Demo folder path hardcoded (container deployment risk)
- ⚠️ No configuration caching (reloaded each time)
- ⚠️ No concurrent update locking

**Testing:**

- ❌ No automated tests (0% coverage)
- Manual testing via frontend configuration UI
- Demo datasets serve as test configurations
- README documentation describes dataset configuration

**Known Issues:**

- Demo folder path resolution may fail in container environments
- No protection against configuration corruption
- Concurrent updates may cause race conditions
- No validation that configuration changes are safe

**Recommendations:**

- Add JSON schema validation library integration
- Implement configuration versioning with history
- Add validation for system_prompt content
- Implement configuration rollback mechanism
- Make demo folder path configurable via environment variable
- Add configuration caching with TTL
- Implement optimistic locking for concurrent updates
- Add comprehensive configuration tests
- Add configuration validation against active jobs
- Implement staged configuration rollout
- Add audit logging for configuration changes
