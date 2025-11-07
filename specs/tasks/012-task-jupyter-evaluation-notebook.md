# Task 012: Jupyter Notebook for Evaluation and Analysis

## Description
Interactive Jupyter notebook for evaluation workflow experimentation, ground truth comparison, and extraction quality analysis. Provides a development environment for testing document processing pipeline components, evaluating results against ground truth datasets, and generating evaluation reports.

## Dependencies

- 003-task-ocr-processing
- 004-task-gpt-vision-extraction
- 006-task-evaluator-framework

## Technical Requirements

- **Jupyter Notebook Environment**: Interactive Python development and documentation
- **Azure SDK Integration**: Direct access to Azure Document Intelligence and OpenAI
- **LangChain Integration**: Structured data extraction chains
- **Evaluator Framework**: Import and use custom evaluators
- **Ground Truth Datasets**: Load and compare against reference data
- **Demo Dataset Access**: Access to default, medical, and mistral datasets
- **Output Generation**: Export evaluation results to JSON files
- **Visualization Support**: Plotting and analytics capabilities (implicit)
- **Module Path Configuration**: Dynamic sys.path manipulation to import from src/
- **Environment Configuration**: .env file loading for API credentials

**Notebook Workflow:**

1. **Setup**: Import dependencies and configure paths
2. **Configuration Loading**: Load system prompts and output schemas
3. **OCR Processing**: Extract text from demo documents
4. **Image Preparation**: Convert PDFs to images
5. **GPT Extraction**: Run structured data extraction
6. **Result Export**: Save output to JSON files
7. **Evaluation**: Compare against ground truth using evaluators
8. **LLM-as-Judge**: Use GPT for qualitative evaluation

## Implementation Evidence

- `notebooks/evaluator.ipynb` - Main evaluation notebook (336 lines)
  - Cell #VSC-ebcc21ef (Lines 1-30): Imports and module path configuration
    - Adds `src/functionapp` to sys.path for module imports
    - Imports OCR, GPT, and image processing functions
    - Loads environment variables from .env
  - Cell #VSC-fb858208: Section header for solution execution
  - Cell #VSC-19c76f18 (Lines 40-100): Full pipeline execution
    - Loads system_prompt.txt and output_schema.json from demo datasets
    - Runs OCR on all PDFs in input directory
    - Converts PDFs to images with size limits
    - Calls get_structured_data for extraction
    - Parses and saves output to /tmp/output.json
  - Cell #VSC-d2567508: Section header for LLM evaluation
  - Cell #VSC-ef75d097 (Lines 110-130): LLM-as-judge evaluation setup
    - Imports LangChain components for evaluation
- `notebooks/README.md` - Notebook documentation and setup instructions
  - Environment variable configuration
  - Azure OpenAI deployment name requirements
- `notebooks/requirements.txt` - Notebook-specific dependencies
  - Includes jupyter, azure-ai-documentintelligence, langchain, etc.
- `notebooks/outputs/` - Saved evaluation outputs directory
  - `output_07_31.15.32.50.json` - Example output
  - `output_07_31.15.32.50_randomized-1.json` - Randomized test output
  - Multiple timestamped output files for comparison

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ Notebook imports from src/functionapp via sys.path manipulation (evaluator.ipynb cell 1)
- ✅ Supports multiple demo datasets (default, medical, mistral) (evaluator.ipynb cell 3)
- ✅ Loads system_prompt.txt and output_schema.json from dataset folders (cell 3 lines 40-48)
- ✅ Runs OCR on all PDFs in input directory (cell 3 lines 55-58)
- ✅ Converts PDFs to images for GPT-4 Vision (cell 3 lines 61-63)
- ✅ Calls get_structured_data with OCR results and images (cell 3 lines 80-81)
- ✅ Parses JSON output with parse_json_markdown (cell 3 line 84)
- ✅ Saves results to /tmp/output.json (cell 3 lines 89-91)
- ✅ Environment variables loaded from .env file (cell 1 line 30)
- ✅ Config object used for image limits and processing settings (cell 3 lines 68-70)
- ✅ Multiple output files saved with timestamps for comparison (outputs/ directory)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- LLM-as-judge evaluation section incomplete (only imports shown)
- No automatic comparison between output.json and ground truth
- No evaluation metrics calculation visible in notebook
- No visualization of evaluation results
- Custom evaluators imported but not used in shown cells
- No error handling for failed OCR or extraction
- Hardcoded paths (../demo/, /tmp/) may fail in different environments
- Windows/Unix path compatibility issues noted in comments (line 18)

## Testing Requirements

⚠️ **NO AUTOMATED TESTS:**

- Jupyter notebooks are inherently manual/interactive
- No notebook execution tests in CI/CD
- No validation of notebook outputs
- **Coverage:** N/A (manual testing tool)

**Manual Testing Evidence:**

- Multiple output files in notebooks/outputs/ directory show manual testing
- README.md provides setup and execution instructions
- Comments in notebook indicate manual testing of different datasets

**Recommendations:**

- Add automated notebook execution tests using nbconvert or papermill
- Implement notebook smoke tests in CI/CD
- Add validation scripts for notebook outputs
- Complete LLM-as-judge evaluation section
- Add visualization cells for evaluation metrics
- Implement automatic ground truth comparison
- Add error handling and retry logic
- Make paths configurable via environment variables
- Add notebook for regression testing with saved outputs
- Create notebook for cost analysis and token usage tracking
