# Task 005: Document Processing Orchestration Pipeline

## Description
End-to-end document processing orchestration that coordinates OCR, image preparation, GPT extraction, evaluation, and summarization into a unified workflow. Handles PDF chunking for large documents, manages document state in Cosmos DB, tracks processing stages, and implements concurrency control for parallel processing of multiple documents.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core
- 003-task-ocr-processing
- 004-task-gpt-vision-extraction

## Technical Requirements

- **PDF Processing**: PyMuPDF (fitz) for PDF page counting, splitting, and manipulation
- **Image Conversion**: PIL/Pillow for PDF-to-image conversion with configurable DPI
- **Document Chunking**: Split large PDFs into page subsets for token limit management
- **State Management**: Cosmos DB document tracking with status flags and timestamps
- **Blob Storage Integration**: Read documents from Azure Blob Storage
- **Temporary File Handling**: Secure temp file creation and cleanup
- **Concurrent Processing**: ThreadPoolExecutor with semaphore-based concurrency control
- **Progress Tracking**: Multi-stage processing status (OCR → Extraction → Evaluation → Summary)
- **Error Aggregation**: Collect and store errors at each processing stage
- **Configuration Loading**: Dynamic dataset-specific prompts and schemas from Cosmos DB
- **Timeout Management**: Background task timeout handling
- **Metadata Enrichment**: Processing times, page counts, file sizes, dataset association

**Processing Pipeline Stages:**

1. **Initialization**: Create document record in Cosmos DB with metadata
2. **Image Preparation**: Convert PDF pages to images (configurable max images)
3. **PDF Chunking**: Split large PDFs into manageable subsets (configurable pages per chunk)
4. **OCR Processing**: Extract text from document or chunks
5. **GPT Extraction**: Structured data extraction with schema validation
6. **GPT Evaluation**: Quality assessment and data enrichment (optional)
7. **GPT Summary**: Document summarization generation (optional)
8. **State Updates**: Update document status after each stage

## Implementation Evidence

- `src/containerapp/ai_ocr/process.py` - Core orchestration logic (641 lines)
  - Lines 156-168: `connect_to_cosmos` - Database connection
  - Lines 170-227: `initialize_document` - Document record creation
  - Lines 229-257: `update_state` - Status tracking with timestamps
  - Lines 259-280: `write_blob_to_temp_file` - Blob download to temp storage
  - Lines 282-338: `fetch_model_prompt_and_schema` - Configuration loading with dataset discovery
  - Lines 340-382: `split_pdf_into_subsets` - PDF chunking logic
  - Lines 384-444: `prepare_images` - PDF to image conversion with size limits
  - Lines 446-477: `run_ocr_processing` - OCR stage orchestration
  - Lines 479-575: `run_gpt_extraction` - Extraction stage orchestration
  - Lines 576-595: `run_gpt_evaluation` - Evaluation stage orchestration
  - Lines 596-620: `run_gpt_summary` - Summary stage orchestration
  - Lines 621-641: Main `process_blob` function (referenced, likely continues beyond shown lines)
- `src/containerapp/blob_processing.py` - Blob event handling (527 lines)
  - Lines 35-60: `create_blob_input_stream` - Blob URL parsing and client creation
  - Lines 62-82: `process_blob_async` - Async wrapper with timing and logging
  - Lines 84-94: `handle_timeout_error_async` - Timeout error handling
  - Lines 96-527: Background processing with semaphore-based concurrency control
- `src/containerapp/ai_ocr/azure/images.py` - Image conversion utilities
  - PDF to PNG conversion with configurable quality and DPI
- `src/containerapp/dependencies.py` - Concurrency management
  - Lines 42-61: Global ThreadPoolExecutor (10 workers) and processing semaphore initialization

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ Documents initialized in Cosmos DB with unique ID (blob name with `/` replaced by `__`) (process.py lines 172-227)
- ✅ Dataset extracted from blob path or defaults to 'default-dataset' (process.py lines 176-182)
- ✅ Configuration loaded from Cosmos DB with dataset-specific prompts and schemas (process.py lines 282-400)
- ✅ Large PDFs split into chunks based on `max_pages_per_chunk` setting (process.py lines 340-382)
- ✅ PDF pages converted to images with size limits (process.py lines 384-444)
- ✅ Processing stages tracked with timestamps and completion flags (process.py lines 229-257)
- ✅ OCR, extraction, evaluation, and summary run sequentially (process.py lines 446-620)
- ✅ Processing options control which stages execute (include_ocr, include_images, enable_summary, enable_evaluation)
- ✅ Errors collected and stored in document error array (throughout process.py)
- ✅ Thread-safe processing with thread ID logging (blob_processing.py lines 70-73)
- ✅ Concurrency controlled by global semaphore matching Logic App settings (blob_processing.py, dependencies.py)
- ✅ Temporary files cleaned up after processing (process.py uses tempfile.mkdtemp)
- ✅ Background tasks don't block API responses (blob_processing.py lines 96-140)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No explicit validation that temp files are always cleaned up on errors
- No retry mechanism for failed processing stages
- No partial result preservation if later stages fail
- No processing queue persistence (in-memory only)
- No dead letter queue for permanently failed documents
- Chunked document results may not be properly merged (code structure suggests incomplete implementation)

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for document initialization
- No tests for PDF chunking logic
- No tests for image preparation
- No tests for state management and updates
- No integration tests for full pipeline
- No tests for concurrency control
- No tests for error handling and recovery
- No tests for configuration loading
- **Coverage:** 0%

**Manual Testing Evidence:**

- README.md describes document processing workflow (lines 294-320)
- Jupyter notebook `notebooks/evaluator.ipynb` demonstrates partial pipeline execution
- Frontend upload functionality triggers full pipeline (process_files.py)

**Recommendations:**

- Add unit tests for each processing stage independently
- Test PDF chunking with various document sizes
- Test image conversion with different PDF types
- Add integration tests for full pipeline with test documents
- Test concurrent processing with multiple documents
- Add tests for error scenarios (corrupted PDFs, API failures, timeouts)
- Test configuration loading with different dataset structures
- Add tests for state management and Cosmos DB updates
- Test temp file cleanup in error scenarios
- Add performance tests for large document processing
