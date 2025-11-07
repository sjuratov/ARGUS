# Task 010: Event-Driven Blob Processing with Event Grid

## Description
Event-driven architecture for automatic document processing triggered by Azure Blob Storage events. Implements Event Grid webhook handling for blob creation events, subscription validation, and asynchronous background processing with concurrency control.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core
- 005-task-document-processing-pipeline

## Technical Requirements

- **Event Grid Integration**: Webhook endpoint for Azure Storage events
- **Subscription Validation**: Handle Event Grid subscription validation protocol
- **Event Filtering**: Process only Microsoft.Storage.BlobCreated events for /datasets/ path
- **Background Processing**: Async task execution without blocking webhook response
- **Concurrency Control**: Semaphore-based limiting of concurrent processing operations
- **Blob URL Parsing**: Extract container and blob name from storage URLs
- **BlobInputStream Creation**: Mock file-like interface for blob reading
- **Thread Pool Execution**: Parallel processing using ThreadPoolExecutor
- **Error Handling**: Timeout detection and error logging per document
- **Dataset Path Recognition**: Automatic dataset extraction from blob path

**Event Grid Event Structure:**

- Event Type: `Microsoft.Storage.BlobCreated` or subscription validation
- Data: Contains blob URL, size, content type, etc.
- Validation Code: For subscription validation handshake

**Processing Flow:**

1. Receive Event Grid webhook POST
2. Validate subscription (if validation event)
3. Parse blob created events
4. Filter for /datasets/ path
5. Acquire concurrency semaphore
6. Submit to thread pool for processing
7. Return 200 OK immediately
8. Process document in background

## Implementation Evidence

- `src/containerapp/api_routes.py` - Event Grid webhook handler (651 lines)
  - Lines 71-125: `handle_blob_created` function
    - Lines 77-90: Event Grid subscription validation handling
    - Lines 92-98: Event list parsing and iteration
    - Lines 100-110: EventGridEvent model instantiation
    - Lines 112-120: Blob URL filtering for /datasets/ path
    - Lines 122-125: Background task submission
- `src/containerapp/blob_processing.py` - Background processing logic (527 lines)
  - Lines 35-60: `create_blob_input_stream` function
    - Parses blob URL to extract account, container, and blob name
    - Creates BlobServiceClient and blob client
    - Wraps in BlobInputStream model for compatibility
  - Lines 62-82: `process_blob_async` function
    - Thread-safe wrapper with thread ID logging
    - Timing and performance tracking
    - Error handling and logging
  - Lines 84-94: `handle_timeout_error_async` function
    - Timeout-specific error handling
    - Document state retrieval for diagnostics
  - Lines 96-140: `process_blob_event` function
    - Semaphore acquisition for concurrency control
    - Thread pool submission
    - Async/await integration with ThreadPoolExecutor
- `src/containerapp/models.py` - Event data models (37 lines)
  - Lines 6-17: EventGridEvent model
    - Parses Event Grid event structure
    - Extracts id, eventType, subject, eventTime, data
  - Lines 20-37: BlobInputStream model
    - Mock file-like object for blob reading
    - Compatible with existing processing functions

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ Event Grid subscription validation handled with validationResponse (api_routes.py lines 86-90)
- ✅ Only Microsoft.Storage.BlobCreated events processed (api_routes.py line 103)
- ✅ Blob URL filtered for /datasets/ path to avoid processing config files (api_routes.py line 108)
- ✅ Event Grid webhook returns 200 OK immediately before processing (api_routes.py line 125)
- ✅ Background tasks submitted via FastAPI BackgroundTasks (api_routes.py line 122)
- ✅ Semaphore controls concurrent processing operations (blob_processing.py lines 100-108)
- ✅ BlobInputStream created from blob URL with proper parsing (blob_processing.py lines 35-60)
- ✅ Thread pool executor processes documents in parallel (blob_processing.py lines 62-82)
- ✅ Processing time tracked per document (blob_processing.py lines 73-79)
- ✅ Errors logged with stack traces (blob_processing.py lines 80-82)
- ✅ Thread ID included in logs for debugging concurrent operations (blob_processing.py line 70)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No retry mechanism for failed Event Grid deliveries (relies on Event Grid retry policy)
- No dead letter queue configuration for permanently failed events
- No validation of Event Grid event signatures (assumes trusted source)
- No metrics on event processing latency or throughput
- No handling of blob deletion or modification events (only creation)
- No duplicate event detection (same blob uploaded multiple times)
- BlobInputStream doesn't support streaming reads (loads entire blob into memory)
- No circuit breaker pattern for downstream service failures

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for Event Grid event parsing
- No tests for subscription validation
- No tests for blob URL parsing
- No tests for BlobInputStream creation
- No tests for background task submission
- No tests for concurrency control with semaphore
- No integration tests with Event Grid
- No tests for error handling scenarios
- **Coverage:** 0%

**Manual Testing Evidence:**

- Event Grid webhook can be tested via Azure Portal Event Grid subscription test
- README.md mentions blob trigger processing (lines 294-320)
- Deployment includes Event Grid subscription configuration (inferred from architecture)

**Recommendations:**

- Add unit tests for Event Grid event models
- Test subscription validation handshake
- Test blob URL parsing with various URL formats
- Add tests for event filtering logic
- Test concurrency control with multiple simultaneous events
- Add integration tests with test Event Grid subscription
- Test error handling and timeout scenarios
- Add tests for background task execution
- Test memory usage with large blob uploads
- Add load tests for high-volume event processing
