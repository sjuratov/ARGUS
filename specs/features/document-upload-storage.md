# Document Upload & Storage

## Overview
Document upload and storage feature enables users to upload documents through a web interface or automatically via event-driven processing. Documents are stored in Azure Blob Storage organized by dataset, with automatic event triggering for downstream processing. This feature provides the foundational data ingestion capability for the ARGUS document intelligence platform.

**Business Purpose**: Enable seamless document ingestion through multiple channels (manual upload and event-driven) to initiate document processing workflows without manual intervention.

## User Stories

Based on implemented functionality:

- **As a user**, I want to upload documents through a web interface, so that I can process documents without technical knowledge
- **As a user**, I want to organize my documents by dataset, so that different document types can be processed with appropriate extraction rules
- **As a system administrator**, I want documents automatically processed when uploaded to blob storage, so that batch processing can occur without manual triggering
- **As a developer**, I want event-driven processing triggered by blob creation, so that the system scales automatically with document volume

## Functional Requirements

### Document Upload via Web Interface
- Users can select and upload one or multiple files through the Streamlit frontend
- Files are uploaded directly to Azure Blob Storage in the `datasets` container
- Upload path follows the pattern: `{dataset_name}/{filename}`
- Supported file types include PDF and image formats (JPEG, PNG)
- Upload progress feedback provided to users during file transfer
- Files stored with managed identity authentication (no credential management required)

### Event-Driven Blob Processing
- Azure Event Grid monitors blob storage for new document creation events
- System responds only to `Microsoft.Storage.BlobCreated` events in the `/datasets/` path
- Event Grid webhook validates subscription via validation handshake protocol
- Events trigger background processing without blocking the webhook response
- Blob URL automatically parsed to extract container and blob name for processing

### Dataset Organization
- Documents organized into dataset-specific folders within blob storage
- Dataset name extracted from blob path for appropriate configuration selection
- Multiple datasets supported (default-dataset, medical-dataset, mistral-dataset, etc.)
- Dataset structure enables multi-tenant document processing with different extraction rules

### Background Processing Coordination
- Event Grid webhook returns 200 OK immediately to prevent timeout
- Document processing submitted to background task queue
- Concurrency controlled by configurable semaphore limits
- Thread pool executor manages parallel processing of multiple documents

## Acceptance Criteria

**Web Upload:**
- Given a user on the "Process Files" tab, when they select files and click upload, then files are stored in blob storage under the selected dataset path
- Given multiple files selected, when upload initiated, then all files upload successfully with progress indication
- Given invalid file type selected, when upload attempted, then user receives clear error message

**Event-Driven Processing:**
- Given a blob created in /datasets/ path, when Event Grid delivers the event, then webhook validates and triggers background processing
- Given an Event Grid subscription validation request, when received, then system responds with validation code
- Given a blob created outside /datasets/, when event received, then event is filtered and not processed
- Given webhook receives event, when processing initiated, then 200 OK returned within 30 seconds

**Dataset Organization:**
- Given a document uploaded to "medical-dataset" folder, when processing begins, then system loads medical-dataset configuration
- Given a document with no dataset specified, when processing begins, then system defaults to "default-dataset"

**Concurrency Control:**
- Given concurrency limit set to 5, when 10 documents uploaded simultaneously, then only 5 process concurrently
- Given semaphore at capacity, when new event arrives, then processing waits for available slot

## Non-Functional Requirements

**Performance:**
- File uploads complete within reasonable time based on file size and network speed (observed in implementation)
- Event Grid webhook responses within 30 seconds to prevent timeout and retry
- Blob URL parsing and event validation complete in milliseconds

**Scalability:**
- System handles multiple simultaneous uploads via blob storage scaling
- Event Grid delivers events reliably with built-in retry policy
- Thread pool executor supports configurable parallelism (10 workers observed)

**Reliability:**
- ⚠️ NON-FUNCTIONAL REQUIREMENTS PARTIALLY SPECIFIED: Retry mechanism for failed Event Grid deliveries relies on Event Grid's built-in retry policy (not explicit in implementation)
- Errors logged with thread ID for debugging concurrent operations
- Timeout detection for long-running background tasks

**Security:**
- Managed identity authentication for blob storage access (no stored credentials)
- Network-level security assumed (no explicit authorization in upload flow)

⚠️ **GAPS IN NON-FUNCTIONAL REQUIREMENTS:**
- No explicit dead letter queue configuration for permanently failed events
- No duplicate event detection for repeated uploads
- No Event Grid event signature validation (assumes trusted source)

## Dependencies & Constraints

**Dependencies:**
- Requires Azure infrastructure provisioned (Storage Account, Event Grid, Container Apps) - Task 001
- Requires backend FastAPI application running to handle webhooks - Task 002
- Requires document processing pipeline to consume uploaded documents - Task 005

**Constraints:**
- Blob storage limited by Azure storage account quotas
- Event Grid delivery latency typically within seconds but not guaranteed
- Thread pool limited to 10 concurrent workers (configurable but hardcoded in implementation)
- BlobInputStream loads entire blob into memory (not suitable for very large files)
- Only monitors blob creation events (no deletion or modification handling)

## Task Traceability

This feature is implemented by the following tasks:
- **Task 001**: Azure Infrastructure Deployment (`specs/tasks/001-task-infrastructure-deployment.md`) - Azure Blob Storage, Event Grid, and Container Apps provisioning
- **Task 010**: Event Grid Integration (`specs/tasks/010-task-event-grid-integration.md`) - Event Grid webhook handling, subscription validation, and background processing
- **Task 011**: Containerization and Deployment (`specs/tasks/011-task-containerization-deployment.md`) - Container deployment enabling web upload functionality
- **Task 007**: Streamlit Frontend (`specs/tasks/007-task-streamlit-frontend.md`) - Web interface for manual file uploads

## Implementation Status

**Status**: ✅ Complete

**Complete:**
- Web-based file upload fully functional via Streamlit frontend
- Azure Blob Storage integration with managed identity authentication
- Event Grid webhook handling with subscription validation
- Background processing triggered by blob creation events
- Dataset-based organization and path parsing
- Concurrency control with semaphore-based limiting
- Thread pool executor for parallel processing

**Known Limitations:**
- ⚠️ No dead letter queue for permanently failed events (relies on Event Grid retry)
- ⚠️ No duplicate event detection (same file uploaded multiple times triggers reprocessing)
- ⚠️ No validation of Event Grid event signatures (assumes network-level security)
- ⚠️ BlobInputStream loads entire blob into memory (memory-intensive for large files)
- ⚠️ No handling of blob deletion or modification events

**Testing:**
- ❌ No automated tests (0% coverage)
- Manual testing via web interface and Event Grid subscription test
- README documentation describes upload workflow
