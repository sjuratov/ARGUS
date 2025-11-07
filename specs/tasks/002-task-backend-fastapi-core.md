# Task 002: Backend FastAPI Application Core

## Description
Core FastAPI application that serves as the ARGUS backend API, providing RESTful endpoints for document processing, configuration management, and health monitoring. Implements modular architecture with dependency injection for Azure service clients, async request handling, and comprehensive logging.

## Dependencies

- 001-task-infrastructure-deployment

## Technical Requirements

- **FastAPI Framework**: Modern Python async web framework with automatic OpenAPI documentation
- **Uvicorn ASGI Server**: High-performance async server with standard workers
- **Lifespan Management**: Async context manager for startup/shutdown of Azure clients
- **Dependency Injection**: Centralized Azure client initialization and management
- **Route Handlers**: Modular route organization separating concerns
- **Background Tasks**: Async background processing for long-running document operations
- **Error Handling**: HTTP exception handling with appropriate status codes
- **Request/Response Models**: Pydantic models for Event Grid events and blob operations
- **Logging**: Structured logging with thread ID tracking for concurrent operations
- **Health Checks**: Comprehensive health endpoints testing all Azure dependencies

**API Endpoints:**

- `GET /` - Root health check
- `GET /health` - Detailed service health with connection verification
- `POST /api/blob-created` - Event Grid webhook for blob creation events
- `POST /api/process-blob` - Manual blob processing trigger
- `POST /api/process-file` - Direct file upload and processing
- `GET /api/configuration` - Retrieve current configuration
- `POST /api/configuration` - Update configuration
- `POST /api/configuration/refresh` - Reload configuration from Cosmos DB
- `GET /api/concurrency` - Get Logic App concurrency settings
- `PUT /api/concurrency` - Update Logic App concurrency settings

## Implementation Evidence

- `src/containerapp/main.py` - Main FastAPI application (139 lines)
  - Lines 1-35: Imports, logging setup, lifespan manager for Azure client initialization
  - Lines 37-48: FastAPI app instantiation with metadata
  - Lines 51-98: Route endpoint definitions delegating to api_routes module
- `src/containerapp/api_routes.py` - Route handler implementations (651 lines)
  - Lines 1-28: Imports and dependencies
  - Lines 30-69: Health check implementations with Azure service verification
  - Lines 71-150: Blob event handling with Event Grid validation
  - Lines 152-220: Configuration management endpoints
  - Lines 222-290: Logic App concurrency management
- `src/containerapp/dependencies.py` - Dependency injection and global state (131 lines)
  - Lines 1-30: Global Azure client variables
  - Lines 32-80: Azure client initialization with credential management
  - Lines 82-95: Cleanup handlers
  - Lines 97-131: Client accessor functions
- `src/containerapp/models.py` - Data models (37 lines)
  - Lines 6-17: EventGridEvent model
  - Lines 20-37: BlobInputStream model for blob reading
- `src/containerapp/requirements.txt` - Python dependencies

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ FastAPI application starts with lifespan context manager (main.py lines 24-34)
- ✅ Azure clients initialized on startup using DefaultAzureCredential (dependencies.py lines 32-80)
- ✅ Health endpoint returns connection status for storage and Cosmos DB (api_routes.py lines 36-69)
- ✅ Event Grid subscription validation handled correctly (api_routes.py lines 86-90)
- ✅ Blob created events trigger background processing (api_routes.py lines 92-125)
- ✅ Background tasks don't block request responses (verified by BackgroundTasks usage)
- ✅ Configuration fetched from Cosmos DB with dataset structure (api_routes.py lines 130-177)
- ✅ Configuration updates persisted to Cosmos DB (api_routes.py lines 179-198)
- ✅ Logic App concurrency dynamically managed via Azure Management API (api_routes.py lines 218-290)
- ✅ Global semaphore updated when concurrency settings change (api_routes.py lines 260-266)
- ✅ Proper HTTP status codes and error messages returned (verified throughout api_routes.py)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No explicit request validation beyond Event Grid events
- No rate limiting implementation observed
- No API authentication/authorization middleware (relies on network security)
- No request timeout configuration visible
- No API versioning strategy implemented

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for route handlers
- No integration tests for API endpoints
- No tests for Event Grid event handling
- No tests for background task processing
- No tests for health check endpoints
- No tests for configuration management
- **Coverage:** 0%

**Manual Testing Evidence:**

- README.md describes health check curl command (lines 187-199)
- Health endpoint provides diagnostic information for manual verification

**Recommendations:**

- Add pytest-based unit tests for all route handlers
- Implement integration tests with mocked Azure services
- Add tests for Event Grid validation logic
- Test background task execution and error handling
- Add load testing for concurrent request handling
- Implement API contract tests against OpenAPI schema
