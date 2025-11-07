# Task 003: Document Intelligence OCR Processing

## Description
OCR processing pipeline supporting multiple providers (Azure Document Intelligence and Mistral Document AI) for extracting structured text and layout information from documents. Implements provider abstraction allowing runtime selection between Azure's prebuilt-layout model and Mistral's document AI API.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core

## Technical Requirements

- **Azure Document Intelligence SDK**: Python client for Azure's OCR service using prebuilt-layout model
- **Mistral Document AI API**: HTTP-based integration with Mistral's document processing service
- **Provider Abstraction**: Environment-variable-driven OCR provider selection (`OCR_PROVIDER`)
- **Base64 Encoding**: Document encoding for API submission (Mistral)
- **Managed Identity Authentication**: Credential-less authentication for Azure services
- **Thread-Safe Processing**: Support for parallel document processing with thread ID logging
- **Polling & Async**: Long-running operation polling for OCR completion
- **Error Handling**: Provider-specific error handling with fallback logic
- **Configuration Management**: Dynamic configuration loading from Cosmos DB

**OCR Providers:**

1. **Azure Document Intelligence**: Uses prebuilt-layout analyzer with DocumentAnalysisFeature support
2. **Mistral Document AI**: Uses HTTP API with base64-encoded document submission

**Supported File Types:**

- PDF documents
- Images (JPEG, PNG)

## Implementation Evidence

- `src/containerapp/ai_ocr/azure/doc_intelligence.py` - Azure Document Intelligence integration (38 lines)
  - Lines 1-16: Client initialization with DefaultAzureCredential and custom headers
  - Lines 18-38: OCR result retrieval with prebuilt-layout model and polling
- `src/containerapp/ai_ocr/azure/mistral_doc_intelligence.py` - Mistral Document AI integration (181 lines)
  - Lines 1-48: Base64 encoding and file type detection
  - Lines 50-72: Mistral client configuration from environment
  - Lines 74-181: OCR result retrieval via HTTP API with JSON schema support
- `src/containerapp/ai_ocr/azure/config.py` - Configuration management
  - Loads configuration from Cosmos DB or environment variables
  - Manages API endpoints, keys, and model names for both providers
- `src/containerapp/ai_ocr/process.py` - OCR orchestration (641 lines)
  - Lines 446-477: `run_ocr_processing` function with provider selection logic
  - Lines 156-165: Cosmos DB connection and configuration container access
  - OCR provider determined by `OCR_PROVIDER` environment variable (default: 'azure')

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ Azure Document Intelligence client created per request to avoid connection pooling issues (doc_intelligence.py lines 11-16)
- ✅ OCR processing uses prebuilt-layout model for structure extraction (doc_intelligence.py line 31)
- ✅ Mistral Document AI encodes documents to base64 for API submission (mistral_doc_intelligence.py lines 15-45)
- ✅ OCR provider selection based on `OCR_PROVIDER` environment variable (process.py lines 453-466)
- ✅ Thread-safe processing with thread ID logging for debugging (doc_intelligence.py lines 20-22)
- ✅ Polling mechanism waits for long-running OCR operations (doc_intelligence.py line 33)
- ✅ OCR results returned as markdown/text content (doc_intelligence.py line 34)
- ✅ Processing time tracked and stored in document metadata (process.py lines 468-475)
- ✅ Custom solution header added to Azure requests (`solution: ARGUS-1.0`) (doc_intelligence.py line 15)
- ✅ Mistral supports optional JSON schema for structured extraction (mistral_doc_intelligence.py line 72)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No retry logic for transient OCR failures observed
- No timeout configuration for long-running operations
- No quality validation of OCR results
- No confidence score extraction or reporting
- Mistral implementation not fully tested in production (based on code structure)

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for Azure Document Intelligence integration
- No unit tests for Mistral Document AI integration
- No tests for OCR provider selection logic
- No integration tests with actual OCR services
- No tests for base64 encoding functionality
- No tests for error handling and retries
- **Coverage:** 0%

**Manual Testing Evidence:**

- README.md describes OCR provider configuration (lines 322-346)
- Jupyter notebook `notebooks/evaluator.ipynb` shows OCR usage in cells
  - Cell #VSC-19c76f18 demonstrates OCR processing on demo datasets
  - Uses `get_ocr_results` function from doc_intelligence module

**Recommendations:**

- Add unit tests with mocked Azure/Mistral APIs
- Implement integration tests with test documents
- Add tests for provider switching logic
- Test edge cases (empty documents, corrupted files, large files)
- Add performance tests for OCR processing time
- Test concurrent OCR processing with multiple threads
- Add tests for configuration loading from different sources
