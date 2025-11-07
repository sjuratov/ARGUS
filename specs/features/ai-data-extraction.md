# AI-Powered Data Extraction

## Overview
AI-Powered Data Extraction feature uses GPT-4 Vision (or GPT-4o) with multimodal processing to extract structured JSON data from documents according to configurable schemas. The system combines OCR text and document images to perform comprehensive extraction, with robust JSON parsing, error recovery, and truncation detection to ensure reliable structured data output.

**Business Purpose**: Transform unstructured documents into structured, machine-readable JSON data that conforms to business-specific schemas, enabling automated data processing, validation, and integration with downstream systems.

## User Stories

Based on implemented functionality:

- **As a user**, I want structured data automatically extracted from my documents, so that I don't need to manually enter information
- **As a business analyst**, I want extraction to follow my custom schema, so that output matches our business data requirements
- **As a user with scanned documents**, I want extraction to work on both images and text, so that data is captured even from low-quality scans
- **As a system administrator**, I want automatic error recovery for malformed responses, so that extraction succeeds despite AI response issues
- **As a data quality manager**, I want to know when extraction is incomplete, so that I can review and correct truncated results

## Functional Requirements

### Multimodal Data Extraction

- Processes both OCR text and document images for comprehensive extraction
- Supports three extraction modalities:
  - Text-only: Extraction from OCR results alone
  - Image-only: Direct visual analysis without OCR
  - Hybrid: Combined OCR + image analysis for cross-validation
- Modality-specific instructions generated based on available inputs
- Cross-modal validation recommended when both text and images available

### Schema-Based Extraction

- Runtime-configurable JSON schema defines expected output structure
- System prompts dynamically constructed with schema templates
- Explicit JSON formatting rules included in GPT instructions
- Validates extracted data has expected structure (not empty objects)
- Supports nested JSON structures, arrays, and complex data types

### Robust JSON Parsing

- Multiple JSON cleanup strategies with fallback logic:
  1. Direct JSON parsing of response
  2. Markdown code block extraction
  3. Text normalization and repair
  4. Bracket/brace balancing
  5. Quote escaping fixes
- Truncation detection identifies incomplete JSON responses
- User-friendly error messages with actionable recommendations
- Processing time tracked for performance monitoring

### Azure OpenAI Integration

- Direct Azure OpenAI SDK integration (not LangChain-based despite module naming)
- GPT-4 Vision/GPT-4o model support
- Base64 image encoding for visual analysis
- Token counting and size validation for images
- Environment-based configuration for deployment names and endpoints

### Error Handling & Recovery

- Graceful handling of malformed GPT responses
- Detection of token-limit-exceeded scenarios
- Detailed error messages guide users on resolution
- Empty extraction detection prevents meaningless outputs
- Processing continues with best-effort results when possible

## Acceptance Criteria

**Multimodal Processing:**

- Given OCR text and images available, when extraction runs, then system uses both modalities
- Given only OCR text available, when extraction runs, then text-only instructions generated
- Given only images available, when extraction runs, then image-only instructions generated
- Given hybrid mode active, when extraction runs, then cross-modal validation recommended in prompt

**Schema Conformance:**

- Given a JSON schema provided, when extraction completes, then output structure matches schema
- Given schema includes nested objects, when extraction runs, then nested data extracted correctly
- Given schema includes arrays, when extraction runs, then array fields populated appropriately
- Given extraction produces empty object, when validation occurs, then user receives warning

**JSON Parsing & Recovery:**

- Given GPT returns valid JSON, when parsing occurs, then data extracted successfully
- Given GPT returns JSON in markdown code blocks, when parsing occurs, then JSON extracted correctly
- Given GPT returns malformed JSON, when parsing occurs, then cleanup strategies attempt repair
- Given JSON truncated due to token limits, when detection occurs, then user receives truncation warning
- Given all parsing strategies fail, when extraction completes, then user receives error with recommendations

**Performance & Tracking:**

- Given extraction completes, when results returned, then processing time included in metadata
- Given extraction starts, when processing, then progress logged for monitoring
- Given multiple extractions run, when concurrent processing, then thread IDs logged for debugging

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No explicit schema validation against provided JSON schema (only structural validation)
- No retry logic for transient API failures
- No caching for repeated extractions of same document
- JSON repair strategies may produce incorrect data without warning
- No extraction confidence scores or quality metrics

## Non-Functional Requirements

**Accuracy:**

- GPT-4 Vision provides high-quality extraction for most document types
- Multimodal processing improves accuracy over text-only approaches
- ⚠️ ACCURACY GAPS: No quantitative accuracy metrics or confidence scores
- JSON repair may introduce errors during malformed response recovery

**Performance:**

- Processing time tracked and stored in document metadata
- ⚠️ PERFORMANCE GAPS: No specific performance SLAs defined
- No timeout configuration visible for long-running extractions
- No optimization for repeated or similar documents (no caching)

**Reliability:**

- Multiple JSON parsing strategies improve extraction success rate
- Truncation detection prevents silently incomplete extractions
- ⚠️ RELIABILITY GAPS: No retry mechanism for transient API failures
- Error handling provides actionable guidance to users

**Cost:**

- ⚠️ COST GAPS: No cost tracking or token usage optimization
- No limits on extraction attempts per document
- Base64 encoding increases token consumption for images

## Dependencies & Constraints

**Dependencies:**

- Requires Azure OpenAI resource with GPT-4 Vision/GPT-4o deployment - Task 001
- Requires OCR processing for text extraction - Task 003
- Requires document processing pipeline for image preparation - Task 005
- Requires configuration management for schema and prompt loading - Task 009

**Constraints:**

- GPT-4 Vision API token limits may truncate extraction for large documents
- Image size limits and token costs constrain number of images processed
- API rate limits apply to Azure OpenAI endpoint
- JSON schema complexity limited by GPT context window
- Extraction quality depends on document quality and clarity
- No guarantee of perfect JSON structure (requires repair strategies)

**Technical Constraints:**

- Azure OpenAI SDK version: openai==1.58.1
- LangChain installed but not used in production code (only in notebooks)
- Module named "chains.py" misleading (no LangChain implementation)
- Base64 encoding required for image submission
- Memory usage scales with image count and size

## Task Traceability

This feature is implemented by the following tasks:

- **Task 004**: GPT-4 Vision Data Extraction (`specs/tasks/004-task-gpt-vision-extraction.md`) - Core extraction implementation with multimodal processing
- **Task 003**: OCR Processing (`specs/tasks/003-task-ocr-processing.md`) - Provides OCR text for extraction
- **Task 005**: Document Processing Pipeline (`specs/tasks/005-task-document-processing-pipeline.md`) - Image preparation and orchestration
- **Task 009**: Configuration Management (`specs/tasks/009-task-configuration-management.md`) - Schema and prompt management

## Implementation Status

**Status**: ✅ Mostly Complete

**Complete:**

- Multimodal extraction fully implemented (text, image, hybrid)
- Schema-based extraction with dynamic prompt construction
- Robust JSON parsing with multiple cleanup strategies
- Truncation detection and user-friendly error messages
- Azure OpenAI SDK integration (direct, not LangChain)
- Base64 image encoding and processing
- Processing time tracking
- Thread-safe concurrent extraction
- Empty extraction detection

**Incomplete/Missing:**

- ⚠️ No explicit JSON schema validation (only checks structure, not conformance)
- ⚠️ No retry logic for transient OpenAI API failures
- ⚠️ No caching mechanism for repeated extractions
- ⚠️ No cost tracking or token usage reporting
- ⚠️ No extraction confidence scores or quality metrics
- ⚠️ JSON repair strategies may silently produce incorrect data

**Testing:**

- ❌ No automated tests (0% coverage)
- Jupyter notebook demonstrates extraction workflow
- Manual testing with demo datasets (default, medical, mistral)
- README documentation describes extraction capabilities

**Known Issues:**

- Module naming misleading: "chains.py" suggests LangChain but uses direct SDK
- LangChain dependencies installed but unused in production code
- JSON repair may fix structure but introduce semantic errors
- No validation that repaired JSON matches original intent

**Recommendations:**

- Add comprehensive unit tests for JSON parsing with malformed inputs
- Implement schema validation against provided JSON schemas
- Add retry logic with exponential backoff for API failures
- Implement caching for repeated extraction requests
- Add cost tracking and token usage monitoring
- Implement extraction confidence scoring
- Refactor module names to reflect actual implementation (remove "chains" naming)
- Remove unused LangChain dependencies or implement LangChain integration
- Add validation that JSON repairs don't introduce semantic errors
