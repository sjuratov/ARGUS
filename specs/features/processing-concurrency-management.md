# Processing Concurrency Management

## Overview
Processing Concurrency Management provides dynamic control over concurrent document processing operations through Azure Logic App integration and backend semaphore synchronization. System administrators can adjust concurrency limits at runtime via API or UI, with automatic synchronization between Logic App trigger settings and backend processing semaphore to prevent resource exhaustion and optimize throughput.

**Business Purpose**: Enable operational control over system resource utilization by dynamically adjusting how many documents process simultaneously, balancing throughput with resource consumption based on workload patterns and infrastructure capacity.

## User Stories

Based on implemented functionality:

- **As a system administrator**, I want to control concurrent processing, so that I can balance throughput with resource limits
- **As an operations manager**, I want to adjust concurrency without redeployment, so that I can respond to changing workload patterns
- **As a cost optimizer**, I want to limit concurrency during low-priority periods, so that I can reduce cloud resource costs
- **As a performance engineer**, I want to increase concurrency during peak periods, so that documents process faster when needed
- **As a developer**, I want backend and Logic App synchronized, so that system doesn't accept more work than it can handle

## Functional Requirements

### Azure Logic App Concurrency Control

- Manages Azure Logic App workflow trigger concurrency settings
- Modifies `runtimeConfiguration.concurrency.runs` in workflow definition
- Validates concurrency values between 1-100 (Azure limits)
- Retrieves current workflow definition and configuration
- Updates workflow via Azure Management API
- Tracks trigger name and last modified timestamp

### Backend Semaphore Synchronization

- Global in-memory semaphore controls concurrent document processing
- Semaphore limit synchronized with Logic App concurrency settings
- Automatic semaphore update after successful Logic App configuration change
- Default semaphore: 5 concurrent operations if Logic App not configured
- ThreadPoolExecutor with 10 workers for parallel execution

### Configuration Management

- Environment variables define Logic App resource:
  - `AZURE_SUBSCRIPTION_ID`: Azure subscription identifier
  - `AZURE_RESOURCE_GROUP_NAME`: Resource group containing Logic App
  - `LOGIC_APP_NAME`: Logic App workflow name
- Managed identity authentication for Azure Management API
- Requires Contributor or Logic App Contributor role for managed identity

### API Endpoints

**Get Concurrency Settings:**

- GET `/api/concurrency` endpoint
- Returns current concurrency limit from Logic App
- Includes trigger name and workflow state
- Includes last modified timestamp for audit
- Error handling for missing or inaccessible Logic App

**Update Concurrency Settings:**

- PUT `/api/concurrency` endpoint
- Validates requested concurrency (1-100 range)
- Updates Logic App workflow trigger configuration
- Updates backend semaphore to match new limit
- Returns updated configuration on success
- Rolls back on failure (to extent possible)

### Frontend Integration

- Concurrency management UI in frontend (concurrency_management.py)
- Current concurrency display
- Slider or input for new concurrency value
- Update button to apply changes
- Visual feedback on success/failure

### Graceful Degradation

- System continues if Logic App not configured
- Returns informative error messages
- Doesn't crash application on Logic App access failures
- Validates environment variables on initialization

## Acceptance Criteria

**Logic App Management:**

- Given environment variables set, when LogicAppManager initializes, then Azure credentials and resource identifiers configured
- Given missing environment variable, when initializing, then validation error raised
- Given valid credentials, when getting concurrency, then workflow definition retrieved from Azure
- Given workflow retrieved, when extracting concurrency, then runtimeConfiguration.concurrency.runs value returned
- Given concurrency update requested, when validating, then range checked (1-100)
- Given valid concurrency value, when updating, then workflow definition modified with new value
- Given workflow updated, when deploying, then changes pushed via Azure Management API

**Semaphore Synchronization:**

- Given concurrency limit updated in Logic App, when update succeeds, then backend semaphore updated to match
- Given semaphore updated, when new documents arrive, then new concurrency limit enforced
- Given concurrency limit 5, when 10 documents processing, then only 5 execute concurrently
- Given default initialization, when Logic App not configured, then semaphore defaults to 5

**API Endpoints:**

- Given GET /api/concurrency called, when executing, then current Logic App concurrency returned
- Given PUT /api/concurrency called with value 10, when successful, then concurrency set to 10
- Given PUT /api/concurrency called with value 0, when validating, then rejected as invalid
- Given PUT /api/concurrency called with value 101, when validating, then rejected as invalid
- Given API call fails, when error occurs, then descriptive error message returned

**Frontend Integration:**

- Given concurrency UI accessed, when loading, then current concurrency displayed
- Given new concurrency entered, when update clicked, then API called with new value
- Given update succeeds, when response received, then UI shows success message
- Given update fails, when error returned, then UI shows error message

**Error Handling:**

- Given Logic App doesn't exist, when accessing, then error returned without crash
- Given insufficient permissions, when accessing, then permission error returned
- Given network failure, when calling Azure API, then network error handled gracefully
- Given invalid workflow state, when accessing, then error message explains issue

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation that Logic App actually exists before attempting operations
- No rollback mechanism if workflow update fails (partially updated state possible)
- No verification that updated workflow deployed successfully
- No monitoring of actual concurrent executions vs. configured limit
- No audit logging of concurrency changes (who, when, from/to what)
- No rate limiting on concurrency update API (could be abused)

## Non-Functional Requirements

**Flexibility:**

- Runtime concurrency adjustments without redeployment
- Immediate effect on new document processing
- Supports range of 1-100 concurrent operations

**Reliability:**

- Semaphore synchronization prevents overload
- Graceful degradation if Logic App unavailable
- ⚠️ RELIABILITY GAPS: No rollback on failed updates
- ⚠️ No verification of deployment success

**Performance:**

- Concurrency control optimizes resource utilization
- Prevents resource exhaustion from excessive parallelism
- Enables performance tuning based on workload

**Security:**

- Managed identity authentication (no stored credentials)
- RBAC controls who can modify Logic App configuration
- ⚠️ SECURITY GAPS: No API-level authorization (assumes network security)
- ⚠️ No rate limiting on configuration updates

**Auditability:**

- Last modified timestamp returned for basic audit
- ⚠️ AUDITABILITY GAPS: No detailed audit log of changes
- ⚠️ No tracking of who made changes

## Dependencies & Constraints

**Dependencies:**

- Requires Azure Logic App provisioned (optional) - Task 001
- Requires Azure Management SDK (azure-mgmt-logic)
- Requires backend API for endpoints - Task 002
- Requires document processing pipeline consuming semaphore - Task 005
- Requires managed identity with Contributor role

**Constraints:**

- Concurrency limited to Azure Logic App range (1-100)
- Logic App configuration is optional (system works without it)
- Semaphore is in-memory only (lost on container restart)
- Changes affect all new processing immediately (no staged rollout)
- No coordination between multiple backend instances (if scaled out)
- ThreadPoolExecutor fixed at 10 workers (may bottleneck at high concurrency)

**Technical Constraints:**

- Azure Management SDK for Logic App operations
- DefaultAzureCredential for authentication
- Environment variables for configuration
- In-memory semaphore (not distributed)

## Task Traceability

This feature is implemented by the following tasks:

- **Task 008**: Logic App Concurrency Management (`specs/tasks/008-task-logic-app-concurrency.md`) - Core concurrency control implementation
- **Task 002**: Backend FastAPI Core (`specs/tasks/002-task-backend-fastapi-core.md`) - Concurrency API endpoints
- **Task 005**: Document Processing Pipeline (`specs/tasks/005-task-document-processing-pipeline.md`) - Semaphore consumption
- **Task 007**: Streamlit Frontend (`specs/tasks/007-task-streamlit-frontend.md`) - Concurrency management UI

## Implementation Status

**Status**: ✅ Complete / ⚠️ Partial

**Complete:**

- LogicAppManager class for Azure Management API integration
- Concurrency get and update operations
- Validation of concurrency range (1-100)
- Workflow definition retrieval and modification
- Azure Management API integration for updates
- Backend semaphore synchronization
- API endpoints (GET, PUT) for concurrency management
- Frontend UI for concurrency control (concurrency_management.py)
- Graceful degradation if Logic App not configured
- Environment variable validation
- Managed identity authentication

**Incomplete/Missing:**

- ⚠️ No validation that Logic App exists before operations
- ⚠️ No rollback mechanism for failed updates
- ⚠️ No verification of successful deployment
- ⚠️ No monitoring of actual vs. configured concurrency
- ⚠️ No detailed audit logging of changes
- ⚠️ No rate limiting on API endpoints
- ⚠️ Frontend-backend integration for UI unclear (may not be complete)
- ⚠️ No coordination for multi-instance backend deployments

**Testing:**

- ❌ No automated tests (0% coverage)
- Manual testing via API endpoints
- Frontend UI available for manual testing
- README mentions concurrency control feature

**Known Issues:**

- Semaphore is in-memory (lost on restart)
- No distributed semaphore for scaled-out backends
- ThreadPoolExecutor limited to 10 workers (may bottleneck)
- No validation of Logic App existence
- No protection against concurrent configuration updates

**Recommendations:**

- Add unit tests with mocked Azure Management API
- Test concurrency validation with boundary values
- Add integration tests with test Logic App
- Implement rollback mechanism for failed updates
- Add verification of deployment success
- Implement audit logging with change tracking
- Add rate limiting on API endpoints
- Implement distributed semaphore (Redis) for scaled deployments
- Add monitoring of actual concurrent executions
- Test concurrent configuration update scenarios
- Make ThreadPoolExecutor workers configurable
- Add health checks for Logic App connectivity
