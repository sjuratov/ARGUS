# Extraction Quality Evaluation

## Overview
Extraction Quality Evaluation feature provides a modular framework for assessing the accuracy of extracted data against ground truth datasets. The system implements field-level evaluators with configurable comparison strategies including exact matching, fuzzy matching, semantic similarity, and recursive JSON comparison, enabling quantitative measurement of extraction quality for continuous improvement.

**Business Purpose**: Ensure extraction accuracy meets business requirements by providing quantitative quality metrics, enabling data-driven refinement of extraction models, prompts, and configurations.

## User Stories

Based on implemented functionality:

- **As a data quality manager**, I want to compare extracted data against ground truth, so that I can measure extraction accuracy
- **As a developer**, I want field-specific evaluation strategies, so that different data types can be validated appropriately
- **As a business analyst**, I want correctness ratios calculated, so that I can report on system performance
- **As a user with fuzzy matching needs**, I want configurable string comparison, so that minor variations don't cause false failures
- **As a machine learning engineer**, I want semantic similarity evaluation, so that meaning-based matches are recognized
- **As a developer**, I want to evaluate nested JSON structures, so that complex extraction outputs can be validated

## Functional Requirements

### Evaluator Framework

- Abstract base class defines evaluator interface for extensibility
- Field-level evaluators for granular comparison
- Composable architecture enables custom evaluation pipelines
- Evaluation schema specifies which evaluator to use per field
- Default evaluators used when no schema provided

### Evaluator Types

**1. CustomStringEvaluator:**

- Configurable exact/fuzzy string matching
- Ignore flags for normalization:
  - `IGNORE_DOTS`: Removes periods from comparison
  - `IGNORE_COMMAS`: Removes commas from comparison
  - `IGNORE_DOLLAR_SIGN`: Removes dollar signs from comparison
  - `IGNORE_CASE`: Case-insensitive comparison
- Returns boolean match result

**2. FuzzStringEvaluator:**

- Token-based fuzzy matching using RapidFuzz library
- Uses `token_set_ratio` algorithm for comparison
- Configurable threshold (0.0-1.0, default: 0.8)
- Returns match boolean and similarity ratio score
- Handles word order variations and partial matches

**3. CosineSimilarityStringEvaluator:**

- Semantic similarity using sentence embeddings
- Meaning-based comparison beyond literal text
- Returns similarity score (0.0-1.0)
- Useful for semantically equivalent but differently worded extractions

**4. JsonEvaluator:**

- Recursive JSON structure comparison
- Orchestrates field-level evaluators across complex structures
- Supports nested objects and arrays
- Per-field evaluator selection via evaluation schema
- Handles missing fields in actual data
- Calculates correctness ratios (correct_fields / total_fields)

### Evaluation Workflow

- Load ground truth data from reference dataset
- Load actual extraction results from document processing
- Apply evaluation schema to determine evaluator per field
- Recursively traverse JSON structures comparing fields
- Aggregate results and calculate metrics
- Return evaluation report with correctness ratios

### Configuration & Customization

- Evaluation schema defines field-to-evaluator mappings
- Schema supports nested field paths
- Multiple evaluators can run on same data (returns results for each)
- Configurable thresholds and ignore flags per evaluator
- Default evaluators when schema not provided

## Acceptance Criteria

**Evaluator Framework:**

- Given an evaluator class created, when implemented, then it inherits from FieldEvaluatorBase
- Given evaluation needed, when evaluator called, then it implements `__call__` method
- Given custom evaluator needed, when created, then it integrates with existing framework

**CustomStringEvaluator:**

- Given IGNORE_DOTS enabled, when comparing "Dr." and "Dr", then match succeeds
- Given IGNORE_CASE enabled, when comparing "ABC" and "abc", then match succeeds
- Given multiple ignore flags set, when comparing, then all normalizations applied
- Given no ignore flags set, when comparing, then exact match required

**FuzzStringEvaluator:**

- Given threshold 0.8, when similarity is 0.85, then match succeeds
- Given threshold 0.8, when similarity is 0.75, then match fails
- Given token order differs, when comparing, then match succeeds if tokens present
- Given evaluation complete, when results returned, then ratio score included

**CosineSimilarityStringEvaluator:**

- Given semantically similar strings, when compared, then high similarity score returned
- Given semantically different strings, when compared, then low similarity score returned

**JsonEvaluator:**

- Given nested JSON structure, when evaluated, then all levels traversed recursively
- Given evaluation schema provided, when evaluating, then schema-specified evaluators used per field
- Given no evaluation schema, when evaluating, then default evaluators applied
- Given missing field in actual data, when evaluated, then counted as incorrect
- Given array of objects, when evaluated, then elements compared element-wise
- Given evaluation complete, when results returned, then correctness ratio calculated (correct/total)

**Metrics & Reporting:**

- Given evaluation completes, when ratio calculated, then formula is correct_fields / total_fields
- Given multiple evaluators on same data, when running, then results returned for each evaluator
- Given nested structures, when evaluating, then field-level results aggregated correctly

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation that evaluation schema is correctly formatted
- No handling of type mismatches between ground truth and actual data
- No detailed mismatch reporting (only boolean match results)
- No support for custom evaluator plugins beyond built-ins
- No evaluation metrics beyond simple ratios (no precision/recall/F1)

## Non-Functional Requirements

**Accuracy:**

- Evaluators provide accurate comparison per their algorithm
- Fuzzy matching reduces false negatives from minor variations
- Semantic similarity captures meaning-based equivalence
- ⚠️ ACCURACY GAPS: No validation of evaluator correctness beyond basic tests

**Performance:**

- Recursive JSON traversal efficient for typical document structures
- ⚠️ PERFORMANCE GAPS: No performance benchmarks for large nested JSON
- No optimization for repeated evaluations

**Extensibility:**

- Abstract base class enables custom evaluator implementation
- Field-level design allows mix-and-match strategies
- Evaluation schema provides flexible configuration

**Usability:**

- Multiple evaluator types cover common comparison needs
- Configurable thresholds and ignore flags
- Default evaluators work without configuration
- ⚠️ USABILITY GAPS: No detailed error reporting on mismatches

## Dependencies & Constraints

**Dependencies:**

- Requires ground truth datasets for comparison
- Requires GPT extraction results - Task 004
- RapidFuzz library for fuzzy string matching
- Sentence transformers for semantic similarity (inferred from CosineSimilarity evaluator)
- Jupyter notebook environment for evaluation workflows - Task 012

**Constraints:**

- Evaluation requires ground truth data (not available for all documents)
- Semantic similarity evaluator requires sentence embedding model
- Type mismatches between ground truth and actual data may cause errors
- No real-time evaluation (manual/batch process)
- Evaluation schema must be manually created and maintained

**Technical Constraints:**

- Python-based implementation
- RapidFuzz dependency for fuzzy matching
- Sentence transformers or similar for semantic similarity
- JSON structure must be comparable (similar keys and nesting)

## Task Traceability

This feature is implemented by the following tasks:

- **Task 006**: Custom Evaluator Framework (`specs/tasks/006-task-evaluator-framework.md`) - Core evaluator implementation and framework
- **Task 012**: Jupyter Evaluation Notebook (`specs/tasks/012-task-jupyter-evaluation-notebook.md`) - Evaluation workflow and LLM-as-judge experimentation

## Implementation Status

**Status**: ✅ Partial / ⚠️ Incomplete

**Complete:**

- Abstract base class for evaluator framework
- CustomStringEvaluator with configurable ignore flags
- FuzzStringEvaluator with threshold-based fuzzy matching
- JsonEvaluator with recursive traversal
- Evaluation schema support for per-field evaluator selection
- Correctness ratio calculation
- Unit tests for CustomStringEvaluator
- Integration tests for JsonEvaluator with schema

**Incomplete/Missing:**

- ⚠️ CosineSimilarityStringEvaluator implementation not fully visible/verified
- ⚠️ No validation of evaluation schema correctness
- ⚠️ No detailed mismatch reporting (only boolean/ratio results)
- ⚠️ No support for custom evaluator plugins (only built-in types)
- ⚠️ No evaluation metrics beyond ratios (no precision/recall/F1)
- ⚠️ No handling of type mismatches between ground truth and actual data
- ⚠️ LLM-as-judge evaluation in notebook incomplete (only imports shown)

**Testing:**

- ✅ Partial test coverage (40-50% estimated)
- CustomStringEvaluator has unit tests with various configurations
- JsonEvaluator has integration tests (250 lines) with and without schema
- Tests cover nested objects, arrays, missing fields
- Tests validate ratio calculations
- ⚠️ No tests for FuzzStringEvaluator threshold variations
- ⚠️ No tests for CosineSimilarityStringEvaluator
- ⚠️ No tests for error handling (invalid schemas, type mismatches)

**Known Issues:**

- Duplicate evaluator implementations in `src/containerapp/evaluators/` and `src/evaluators/` (code organization in progress)
- CosineSimilarityStringEvaluator not fully tested or verified
- No detailed reporting when fields don't match (just boolean result)

**Recommendations:**

- Complete CosineSimilarityStringEvaluator implementation and testing
- Add comprehensive tests for all evaluator types
- Implement evaluation schema validation
- Add detailed mismatch reporting (expected vs. actual values)
- Implement precision, recall, F1 metrics
- Add error handling for type mismatches
- Complete LLM-as-judge evaluation in notebook
- Remove duplicate evaluator implementations (consolidate to one location)
- Add performance benchmarks for large JSON structures
- Implement custom evaluator plugin system
