# Document Processing Pipeline

## Overview
Document Processing Pipeline orchestrates end-to-end document processing from upload to structured data extraction, coordinating OCR, image preparation, GPT extraction, evaluation, and summarization into a unified workflow. The pipeline handles complex scenarios including PDF chunking for large documents, manages processing state in a database, and implements concurrency control for efficient parallel processing of multiple documents.

**Business Purpose**: Provide automated, hands-free document processing that transforms uploaded documents into structured, validated data through a series of AI-powered processing stages, enabling scale processing of large document volumes.

## User Stories

Based on implemented functionality:

- **As a user**, I want documents automatically processed after upload, so that I don't need to manually trigger each processing step
- **As a user with large documents**, I want automatic PDF splitting, so that documents exceeding token limits can still be processed
- **As a user**, I want to track processing progress, so that I know when my documents are ready
- **As a system administrator**, I want processing state persisted, so that I can monitor and troubleshoot document processing workflows
- **As a user processing many documents**, I want controlled concurrency, so that the system doesn't overwhelm resources
- **As a user**, I want processing to continue despite errors, so that one bad document doesn't block others

## Functional Requirements

### End-to-End Processing Orchestration

- Coordinates multiple processing stages in sequence:
  1. Document initialization with metadata creation
  2. Image preparation (PDF to image conversion)
  3. PDF chunking for large documents
  4. OCR text extraction
  5. GPT structured data extraction
  6. GPT evaluation (optional)
  7. GPT summary generation (optional)
  8. State updates after each stage
- Each stage updates document status and timestamps in database
- Processing options control which stages execute (include_ocr, include_images, enable_summary, enable_evaluation)

### Document Initialization & Metadata

- Creates document record in Cosmos DB with unique ID (blob name with `/` replaced by `__`)
- Extracts dataset from blob path or defaults to 'default-dataset'
- Stores file metadata: blob name, size, page count, upload timestamp
- Initializes processing flags for each stage (all false initially)
- Associates document with dataset-specific configuration

### PDF Processing

- Counts PDF pages using PyMuPDF library
- Splits large PDFs into page subsets based on `max_pages_per_chunk` setting (default: 10 pages)
- Creates manageable chunks for token limit management
- Converts PDF pages to images with configurable DPI and quality
- Enforces maximum image count limit per document
- Temporary file handling for intermediate processing

### State Management & Tracking

- Updates document status flags after each stage completion:
  - `ocr_completed`, `extraction_completed`, `evaluation_completed`, `summary_completed`
- Stores timestamps for each processing stage
- Tracks processing times per stage for performance analysis
- Collects and stores errors in document error array
- Enables resume/retry of failed processing stages

### Concurrency Control

- Global semaphore limits concurrent processing operations
- Thread-safe processing with thread ID logging
- ThreadPoolExecutor with 10 workers for parallel execution
- Semaphore synchronized with Logic App concurrency settings
- Background task execution doesn't block API responses
- Queue-like behavior when concurrency limit reached

### Configuration Loading

- Fetches dataset-specific configuration from Cosmos DB
- Loads system prompts, JSON schemas, and processing options
- Auto-initializes from demo folder if configuration not found
- Dataset discovery and fallback to available datasets
- Validates configuration before processing

### Error Handling

- Errors collected at each stage without stopping pipeline
- Error messages stored in document metadata for review
- Processing continues to next stage even if previous stage has errors
- Timeout handling for long-running operations
- User-friendly error messages with troubleshooting guidance

## Acceptance Criteria

**Document Initialization:**

- Given a blob uploaded, when processing starts, then document record created in Cosmos DB
- Given blob path contains dataset name, when initializing, then dataset extracted from path
- Given blob path has no dataset, when initializing, then defaults to 'default-dataset'
- Given document record created, when saved, then unique ID generated from blob name

**PDF Processing:**

- Given a multi-page PDF, when processing starts, then page count determined
- Given PDF exceeds max_pages_per_chunk, when chunking occurs, then PDF split into subsets
- Given PDF processed, when images needed, then pages converted to images
- Given max image limit set, when converting, then only configured number of images generated

**Processing Stages:**

- Given OCR enabled, when pipeline runs, then OCR stage executes and ocr_completed flag set
- Given extraction needed, when pipeline runs, then GPT extraction executes and extraction_completed flag set
- Given evaluation enabled, when pipeline runs, then evaluation stage executes and evaluation_completed flag set
- Given summary enabled, when pipeline runs, then summary stage executes and summary_completed flag set
- Given stage completes, when updating state, then timestamp recorded for that stage

**Concurrency Control:**

- Given semaphore limit is 5, when 10 documents processing, then only 5 execute concurrently
- Given concurrency limit reached, when new document arrives, then processing waits for available slot
- Given background task submitted, when processing occurs, then API response not blocked

**Configuration:**

- Given dataset configuration exists, when loading, then prompts and schemas retrieved from Cosmos DB
- Given dataset configuration missing, when loading, then auto-initialize from demo folder
- Given requested dataset not found, when loading, then fallback to first available dataset

**Error Handling:**

- Given OCR fails, when error occurs, then error stored and extraction stage still attempts
- Given processing times out, when timeout detected, then timeout error recorded and processing stops
- Given malformed configuration, when loading, then validation error reported to user

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No explicit validation that temp files cleaned up on errors
- No retry mechanism for failed processing stages
- No partial result preservation if later stages fail
- Chunked document results may not be properly merged (implementation unclear)
- No explicit timeout configuration per stage

## Non-Functional Requirements

**Performance:**

- Processing stages execute sequentially (not parallel) for single document
- Multiple documents processed in parallel up to concurrency limit
- ThreadPoolExecutor with 10 workers for parallelism
- ⚠️ PERFORMANCE GAPS: No specific SLA for total processing time
- Processing time tracked per stage for monitoring

**Scalability:**

- Concurrency control prevents resource exhaustion
- Semaphore limits adjustable via Logic App settings
- Cosmos DB scales with document volume
- ⚠️ SCALABILITY GAPS: In-memory queue (no persistence) limits scaling

**Reliability:**

- State persistence enables monitoring and troubleshooting
- Error collection allows diagnosis without losing context
- ⚠️ RELIABILITY GAPS: No retry mechanism for transient failures
- ⚠️ No dead letter queue for permanently failed documents
- Temporary file cleanup implemented but validation unclear

**Maintainability:**

- Modular stage functions enable independent testing and updates
- Configuration externalized for easy adjustments
- Thread ID logging enables concurrent operation debugging

## Dependencies & Constraints

**Dependencies:**

- Requires Azure infrastructure (Blob Storage, Cosmos DB, Container Apps) - Task 001
- Requires backend FastAPI application - Task 002
- Requires OCR processing capability - Task 003
- Requires GPT extraction capability - Task 004
- Requires configuration management - Task 009

**Constraints:**

- Sequential stage processing limits throughput per document
- Cosmos DB query limits may impact high-volume scenarios
- ThreadPoolExecutor fixed at 10 workers (not dynamically scalable)
- Temporary file storage limited by container disk space
- PDF chunking may not properly merge results (unclear from implementation)
- No processing queue persistence (lost on container restart)

**Technical Constraints:**

- PyMuPDF (fitz) for PDF manipulation
- PIL/Pillow for image conversion
- Python tempfile for temporary storage
- Azure SDK for Blob Storage and Cosmos DB access

## Task Traceability

This feature is implemented by the following tasks:

- **Task 005**: Document Processing Pipeline (`specs/tasks/005-task-document-processing-pipeline.md`) - Core orchestration logic
- **Task 002**: Backend FastAPI Core (`specs/tasks/002-task-backend-fastapi-core.md`) - API endpoints and background task execution
- **Task 003**: OCR Processing (`specs/tasks/003-task-ocr-processing.md`) - OCR stage implementation
- **Task 004**: GPT Vision Extraction (`specs/tasks/004-task-gpt-vision-extraction.md`) - Extraction, evaluation, and summary stages
- **Task 009**: Configuration Management (`specs/tasks/009-task-configuration-management.md`) - Configuration loading
- **Task 010**: Event Grid Integration (`specs/tasks/010-task-event-grid-integration.md`) - Automatic pipeline triggering

## Implementation Status

**Status**: ✅ Mostly Complete / ⚠️ Partial

**Complete:**

- End-to-end orchestration of all processing stages
- Document initialization with metadata in Cosmos DB
- PDF page counting and chunking logic
- Image conversion with configurable limits
- State management with completion flags and timestamps
- Concurrency control with semaphore and thread pool
- Configuration loading from Cosmos DB
- Error collection and storage
- Processing time tracking per stage
- Background task execution
- Thread-safe concurrent processing

**Incomplete/Missing:**

- ⚠️ Retry mechanism for failed stages not implemented
- ⚠️ Partial result preservation incomplete (if later stage fails, earlier results may be lost)
- ⚠️ Chunked document result merging unclear/incomplete
- ⚠️ Temporary file cleanup validation not explicit
- ⚠️ No processing queue persistence (in-memory only)
- ⚠️ No dead letter queue for permanently failed documents
- ⚠️ No explicit timeout configuration per stage

**Testing:**

- ❌ No automated tests (0% coverage)
- Manual testing via web interface uploads
- Jupyter notebook demonstrates partial pipeline execution
- README documentation describes workflow

**Known Issues:**

- Sequential processing of stages may be slow for large documents
- No validation that temp files always cleaned up on errors
- Chunked PDF processing may not properly merge results
- Processing queue lost on container restart

**Recommendations:**

- Add retry logic with exponential backoff for transient failures
- Implement processing queue persistence (Redis, Service Bus)
- Add explicit temp file cleanup verification
- Implement chunked result merging logic
- Add stage-level timeout configuration
- Add comprehensive testing for each stage and full pipeline
- Implement partial result preservation
- Add dead letter queue for failed documents
- Consider parallel stage execution where possible
