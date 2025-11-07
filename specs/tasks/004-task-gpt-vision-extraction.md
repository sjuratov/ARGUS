# Task 004: GPT-4 Vision Data Extraction with Azure OpenAI SDK

## Description
AI-powered structured data extraction from documents using GPT-4 Vision (or GPT-4o) with multimodal input support. Implements direct Azure OpenAI SDK-based extraction pipeline that processes both OCR text and document images to extract structured JSON data according to configurable schemas, with robust JSON parsing and error handling.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core
- 003-task-ocr-processing

## Technical Requirements

- **Azure OpenAI SDK**: Direct Python SDK for GPT-4 Vision/GPT-4o model access (openai==1.58.1)
- **LangChain Dependencies**: Installed but only used in Jupyter notebooks for JSON parsing utilities (parse_json_markdown)
- **Multimodal Processing**: Combined OCR text + image analysis for comprehensive extraction
- **Dynamic Prompting**: Runtime-configurable system prompts and extraction instructions
- **JSON Schema Validation**: Structured output conforming to provided schemas
- **Robust JSON Parsing**: Multiple fallback strategies for parsing malformed GPT responses
- **Truncation Detection**: Detection and handling of token-limit-exceeded scenarios
- **Base64 Image Handling**: Image encoding for GPT-4 Vision API
- **Error Recovery**: Graceful degradation with detailed error messages and recommendations

**Extraction Pipeline:**

1. **Input Analysis**: Determine available modalities (text, images, or both)
2. **Prompt Construction**: Build context-aware system prompt based on input type
3. **GPT Invocation**: Call Azure OpenAI with multimodal content
4. **JSON Parsing**: Parse GPT response with multiple cleanup strategies
5. **Validation**: Verify extracted data against schema
6. **Error Handling**: Detect truncation, malformed JSON, and provide actionable feedback

**Supported Modalities:**

- Text-only extraction (OCR results)
- Image-only extraction (direct visual analysis)
- Hybrid extraction (OCR + images for cross-validation)

## Implementation Evidence

- `src/containerapp/ai_ocr/chains.py` - Azure OpenAI extraction functions (561 lines)
  - **NOTE**: Despite the filename "chains.py", this module does NOT use LangChain
  - Uses direct Azure OpenAI SDK (AzureOpenAI client from openai package)
  - Lines 1-90: JSON cleaning utilities with multiple repair strategies
  - Lines 92-100: Azure OpenAI client initialization with get_client()
  - Lines 102-210: `get_structured_data` function with multimodal prompt construction
  - Lines 115-145: Modality-specific instruction generation
  - Lines 147-190: System prompt with critical JSON formatting rules
  - Lines 210-250: Message construction with text and image content
  - Lines 218-225: Direct client.chat.completions.create() call (Azure OpenAI SDK, not LangChain)
  - Lines 227-260: Response validation and truncation detection
  - Lines 262-340: Multiple JSON cleanup strategies with fallback logic
  - Lines 350-450: `get_summary_with_gpt` function for document summarization
  - Lines 360-500: `perform_gpt_evaluation_and_enrichment` function for quality assessment
- `src/containerapp/ai_ocr/process.py` - Extraction orchestration (641 lines)
  - Lines 1-155: `safe_parse_json` with truncation detection (150 lines)
  - Lines 479-575: `run_gpt_extraction` orchestration function
  - Lines 485-495: Debug logging for input validation
  - Lines 497-510: GPT response parsing with safe_parse_json
  - Lines 512-570: Error handling with user-friendly messages
- `src/containerapp/ai_ocr/azure/openai_ops.py` - OpenAI utilities
  - Image loading and base64 encoding functions
  - Token counting and size validation for images
- `src/containerapp/ai_ocr/model.py` - Configuration models
  - Config dataclass for Azure OpenAI settings
- `src/containerapp/requirements.txt` - Dependencies
  - Lines 22-25: LangChain packages included but NOT used in backend code
  - LangChain only used in notebooks/evaluator.ipynb for parse_json_markdown utility

**Architecture Note**: The module is named "chains.py" suggesting LangChain usage, but the implementation uses direct Azure OpenAI SDK calls throughout. LangChain dependencies are installed but only utilized in the Jupyter notebook for JSON parsing utilities, not in the production backend code.

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ GPT-4 Vision processes both OCR text and images (chains.py lines 115-145)
- ✅ Modality-specific instructions generated based on available inputs (chains.py lines 115-145)
- ✅ System prompt includes explicit JSON formatting rules (chains.py lines 147-190)
- ✅ JSON schema template provided to GPT for structured output (chains.py lines 192-193)
- ✅ Multiple JSON parsing strategies attempt to repair malformed responses (process.py lines 57-135)
- ✅ Truncation detection identifies incomplete JSON responses (process.py lines 20-55)
- ✅ User-friendly error messages with actionable recommendations (process.py lines 43-52, 147-155)
- ✅ Extracted data validated and stored in document metadata (process.py lines 571-573)
- ✅ Processing time tracked for performance monitoring (process.py lines 497, 571)
- ✅ Cross-modal validation recommended when both text and images available (chains.py lines 126-131)
- ✅ Empty object detection prevents meaningless extractions (chains.py line 177)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No explicit schema validation against provided JSON schema (only structural validation)
- No retry logic for transient API failures
- No caching mechanism for repeated extractions
- No cost tracking or token usage optimization
- No quality metrics (extraction confidence scores)
- JSON repair strategies may produce incorrect data without warning

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for JSON parsing and repair logic
- No tests for multimodal prompt construction
- No tests for truncation detection
- No integration tests with Azure OpenAI
- No tests for different extraction scenarios (text-only, image-only, hybrid)
- No tests for error handling and recovery
- **Coverage:** 0%

**Manual Testing Evidence:**

- Jupyter notebook `notebooks/evaluator.ipynb` demonstrates extraction workflow
  - Cell #VSC-19c76f18: End-to-end extraction with `get_structured_data`
  - Shows OCR + image extraction on medical and default datasets
- README.md describes extraction capabilities (lines 22-40, 322-346)

**Recommendations:**

- Add comprehensive unit tests for JSON parsing with malformed inputs
- Test all JSON repair strategies with real GPT failure cases
- Add integration tests with mocked OpenAI responses
- Test truncation detection with artificially truncated JSON
- Add tests for all three modality types (text, image, hybrid)
- Implement regression tests using captured GPT responses
- Add tests for prompt construction with different schema types
- Test error message generation and user guidance
- Add performance tests for large documents and multiple images
