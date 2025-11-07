# Task 006: Custom Evaluator Framework for Extraction Quality

## Description
Modular evaluation framework for assessing the quality of extracted data against ground truth datasets. Implements field-level evaluators with configurable comparison strategies including exact matching, fuzzy matching, cosine similarity, and JSON structural comparison. Provides a composable architecture for building domain-specific evaluation pipelines.

## Dependencies

- 001-task-infrastructure-deployment
- 004-task-gpt-vision-extraction

## Technical Requirements

- **Field Evaluator Base Class**: Abstract base class defining evaluator interface
- **Custom String Evaluator**: Configurable string comparison with ignore rules (dots, commas, dollar signs, case)
- **Fuzz String Evaluator**: Fuzzy string matching using RapidFuzz library
- **Cosine Similarity Evaluator**: Semantic similarity using sentence embeddings
- **JSON Evaluator**: Recursive JSON comparison with per-field evaluator configuration
- **Evaluation Schema Support**: Field-specific evaluator selection and configuration
- **Ratio Metrics**: Calculate correctness ratios (correct fields / total fields)
- **Nested Structure Handling**: Deep traversal of JSON objects and arrays
- **Ground Truth Comparison**: Compare extracted data against reference datasets

**Evaluator Types:**

1. **CustomStringEvaluator**: Configurable exact/fuzzy string matching with punctuation/case handling
2. **FuzzStringEvaluator**: Token-based fuzzy matching with configurable threshold
3. **CosineSimilarityStringEvaluator**: Semantic similarity for meaning-based comparison
4. **JsonEvaluator**: Orchestrates field-level evaluators across JSON structures

**Configuration Options:**

- `IGNORE_DOTS`, `IGNORE_COMMAS`, `IGNORE_DOLLAR_SIGN`: Punctuation normalization
- `IGNORE_CASE`: Case-insensitive comparison
- `THRESHOLD`: Fuzzy match threshold (0.0-1.0)

## Implementation Evidence

- `src/containerapp/evaluators/field_evaluator_base.py` - Abstract base class
  - Lines 1-10: FieldEvaluatorBase ABC with abstract `__call__` method
- `src/containerapp/evaluators/custom_string_evaluator.py` - Configurable string evaluator
  - Lines 1-150: CustomStringEvaluator implementation with ignore flags
  - Normalizes strings based on configuration
  - Returns boolean match result
- `src/containerapp/evaluators/fuzz_string_evaluator.py` - Fuzzy matching evaluator
  - Lines 1-80: FuzzStringEvaluator using RapidFuzz token_set_ratio
  - Threshold-based matching (default 0.8)
  - Returns match boolean and ratio score
- `src/containerapp/evaluators/cosine_similarity_string_evaluator.py` - Semantic evaluator
  - Lines 1-60: CosineSimilarityStringEvaluator using sentence embeddings
  - Calculates semantic similarity between strings
  - Returns similarity score (0.0-1.0)
- `src/containerapp/evaluators/json_evaluator.py` - JSON structure evaluator
  - Lines 1-300: Recursive JSON comparison with per-field evaluator selection
  - Lines 5-50: FieldEvaluatorWrapper for evaluator instantiation
  - Lines 52-150: Recursive traversal of JSON structures
  - Lines 152-250: Ratio calculation and result aggregation
  - Supports evaluation schema for field-specific evaluators
- `src/containerapp/evaluators/tests/test_custom_string_evaluator.py` - Unit tests
  - Tests for CustomStringEvaluator with various configurations
- `src/containerapp/evaluators/tests/test_json_evaluator.py` - Integration tests (250 lines)
  - Lines 1-100: Test setup with ground truth and actual data
  - Lines 8-60: Test without evaluation schema (default evaluators)
  - Lines 62-170: Test with evaluation schema (custom per-field evaluators)
  - Tests nested objects, arrays, and missing fields

**Note:** Duplicate evaluator implementations exist in both `src/containerapp/evaluators/` and `src/evaluators/` directories (appears to be code organization in progress).

## Acceptance Criteria

**Based on observed behavior in implementation and tests:**

- ✅ Field evaluators implement abstract base class interface (field_evaluator_base.py)
- ✅ CustomStringEvaluator normalizes strings based on ignore flags (test_custom_string_evaluator.py)
- ✅ FuzzStringEvaluator uses token-based fuzzy matching (fuzz_string_evaluator.py)
- ✅ JsonEvaluator recursively traverses nested JSON structures (test_json_evaluator.py lines 8-60)
- ✅ Evaluation schema supports field-specific evaluator configuration (test_json_evaluator.py lines 62-170)
- ✅ Ratio metrics calculated as correct_fields / total_fields (test_json_evaluator.py lines 50, 147)
- ✅ Missing fields in actual data counted as incorrect (test_json_evaluator.py lines 38-40)
- ✅ Arrays of objects evaluated element-wise (test_json_evaluator.py lines 23-25)
- ✅ Multiple evaluators can run on same data (returns results for each) (test_json_evaluator.py lines 50-51)
- ✅ Default evaluators used when no schema provided (test_json_evaluator.py lines 8-51)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation of evaluation schema correctness
- No handling of type mismatches between ground truth and actual data
- CosineSimilarityStringEvaluator implementation not fully visible (may be incomplete)
- No support for custom evaluator plugins beyond built-ins
- No detailed mismatch reporting (only boolean match results)
- No evaluation metrics beyond simple ratios (no precision/recall/F1)

## Testing Requirements

✅ **PARTIAL TEST COVERAGE:**

- `src/containerapp/evaluators/tests/test_custom_string_evaluator.py` - CustomStringEvaluator tests
  - Tests various ignore flag combinations
  - Tests case sensitivity handling
- `src/containerapp/evaluators/tests/test_json_evaluator.py` - JsonEvaluator tests (250 lines)
  - Lines 8-60: Test without evaluation schema
  - Lines 62-170: Test with custom evaluation schema
  - Tests nested objects, arrays, missing fields
  - Validates ratio calculations

**Coverage:** Estimated 40-50% (core evaluators tested, but incomplete coverage)

⚠️ **MISSING TESTS:**

- No tests for FuzzStringEvaluator threshold variations
- No tests for CosineSimilarityStringEvaluator
- No tests for field_evaluator_base abstract interface
- No tests for error handling (invalid schemas, type mismatches)
- No tests for performance with large JSON structures
- No integration tests with actual extraction pipeline

**Recommendations:**

- Add comprehensive tests for all evaluator types
- Test edge cases (null values, empty strings, type mismatches)
- Add performance tests for large nested JSON structures
- Test evaluation schema validation and error handling
- Add integration tests with real extraction results
- Test concurrent evaluation of multiple documents
- Add benchmark tests comparing evaluator accuracy
