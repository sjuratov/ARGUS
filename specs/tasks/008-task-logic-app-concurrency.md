# Task 008: Logic App Concurrency Management

## Description
Dynamic concurrency control system for managing Azure Logic App trigger settings via Azure Management API. Allows runtime adjustment of concurrent document processing operations and synchronizes backend processing semaphore with Logic App concurrency limits to prevent resource exhaustion.

## Dependencies

- 001-task-infrastructure-deployment
- 002-task-backend-fastapi-core
- 005-task-document-processing-pipeline

## Technical Requirements

- **Azure Management SDK**: Python client for Logic App management (azure-mgmt-logic)
- **DefaultAzureCredential**: Managed identity authentication for management operations
- **Workflow Definition Management**: Read and update Logic App workflow configurations
- **Trigger Concurrency Configuration**: Modify runtimeConfiguration.concurrency.runs settings
- **Backend Semaphore Synchronization**: Update in-memory semaphore to match Logic App limits
- **RBAC Requirements**: Managed identity needs Contributor or Logic App Contributor role
- **Environment Configuration**: Subscription ID, resource group, and Logic App name from env vars
- **Validation**: Enforce concurrency limits between 1-100 concurrent runs

**Management Operations:**

1. **Get Concurrency Settings**: Retrieve current workflow configuration and extract concurrency
2. **Update Concurrency Settings**: Modify workflow trigger concurrency and redeploy
3. **Synchronize Backend**: Update global processing semaphore to match new limits
4. **Validation**: Ensure requested concurrency within Azure limits (1-100)

## Implementation Evidence

- `src/containerapp/logic_app_manager.py` - Logic App management client (268 lines)
  - Lines 1-25: LogicAppManager class initialization with credentials
  - Lines 27-37: Logic Management client factory
  - Lines 39-85: `get_concurrency_settings` - Extract concurrency from workflow definition
  - Lines 87-268: `update_concurrency_settings` - Modify workflow trigger configuration
  - Lines 100-120: Validation logic (1-100 range check)
  - Lines 122-150: Workflow retrieval and definition update
  - Lines 152-200: Trigger configuration modification
  - Lines 202-240: Workflow update via Azure Management API
- `src/containerapp/api_routes.py` - API endpoints for concurrency management (651 lines)
  - Lines 218-245: `get_concurrency_settings` endpoint with error handling
  - Lines 247-290: `update_concurrency_settings` endpoint with semaphore sync
  - Lines 260-266: Global semaphore update after successful Logic App update
- `src/containerapp/dependencies.py` - Semaphore initialization (131 lines)
  - Lines 42-61: Initial semaphore creation with Logic App concurrency on startup
  - Default semaphore: 5 concurrent operations if Logic App not configured
  - Lines 98-103: `set_global_processing_semaphore` setter function

## Acceptance Criteria

**Based on observed behavior in implementation:**

- ✅ LogicAppManager initializes with Azure credentials and resource identifiers (logic_app_manager.py lines 16-28)
- ✅ Environment variables validated on initialization (AZURE_SUBSCRIPTION_ID, AZURE_RESOURCE_GROUP_NAME, LOGIC_APP_NAME)
- ✅ Get concurrency endpoint retrieves workflow definition and extracts runs limit (logic_app_manager.py lines 39-85)
- ✅ Concurrency range validated (1-100) before update (logic_app_manager.py lines 114-115)
- ✅ Workflow definition deep copied and modified with new concurrency (logic_app_manager.py lines 122-150)
- ✅ Updated workflow deployed via Azure Management API (logic_app_manager.py lines 202-240)
- ✅ Backend semaphore updated to match new concurrency limit (api_routes.py lines 260-266)
- ✅ Graceful degradation if Logic App not configured (returns error, doesn't crash) (logic_app_manager.py lines 20-23)
- ✅ Trigger name and workflow state included in response (logic_app_manager.py lines 72-79)
- ✅ Last modified timestamp returned for audit (logic_app_manager.py lines 78-79)

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- No validation that Logic App actually exists before attempting operations
- No rollback mechanism if workflow update fails
- No verification that updated workflow deployed successfully
- No monitoring of actual concurrent executions vs. configured limit
- Frontend UI for concurrency management exists (concurrency_management.py) but integration unclear
- No audit logging of concurrency changes (who changed, when, from what to what)
- No rate limiting on concurrency update API (could be abused)

## Testing Requirements

⚠️ **NO TESTS FOUND:**

- No unit tests for LogicAppManager class
- No tests for concurrency validation logic
- No tests for workflow definition manipulation
- No integration tests with Azure Management API
- No tests for semaphore synchronization
- No tests for error handling (invalid credentials, missing resources)
- No tests for API endpoints
- **Coverage:** 0%

**Manual Testing Evidence:**

- API endpoints exposed for testing (GET /api/concurrency, PUT /api/concurrency)
- Concurrency management UI in frontend (concurrency_management.py)
- README.md mentions concurrency control as a feature (line 54)

**Recommendations:**

- Add unit tests with mocked Azure Management API
- Test concurrency validation with boundary values (0, 1, 100, 101)
- Test workflow definition updates with various trigger configurations
- Add integration tests with test Logic App
- Test semaphore synchronization across multiple requests
- Test error scenarios (missing permissions, non-existent Logic App)
- Add tests for concurrent update requests (race conditions)
- Test rollback behavior on update failures
- Add audit logging tests
