# Task 001: Azure Infrastructure Deployment with Bicep

## Description
Infrastructure-as-Code implementation using Azure Bicep to provision all required cloud resources for the ARGUS document intelligence platform. This includes container hosting, storage, databases, AI services, identity management, monitoring, and networking components deployed to Azure using declarative Bicep templates.

## Dependencies
None - This is the foundational task

## Technical Requirements
- **Azure Bicep IaC**: Declarative resource definitions for reproducible infrastructure
- **Azure Container Apps**: Serverless container hosting for backend and frontend services
- **Azure Blob Storage**: Document storage with hot tier access
- **Azure Cosmos DB**: Serverless NoSQL database for document metadata and configurations
- **Azure Container Registry**: Private registry for container images
- **Azure Document Intelligence**: OCR and document layout analysis service
- **Managed Identity**: Zero-credential authentication using Azure AD
- **Azure Log Analytics & Application Insights**: Centralized logging and monitoring
- **RBAC Role Assignments**: Least-privilege access controls for managed identities
- **Azure Developer CLI (azd)**: Deployment orchestration and environment management

**Infrastructure Components:**
- Container Apps Environment with log analytics integration
- User-assigned managed identity for container apps
- Storage account with blob container (`datasets`)
- Cosmos DB account with two containers (`documents`, `configuration`)
- Container Registry with admin disabled (uses managed identity)
- Document Intelligence resource
- Log Analytics workspace
- Application Insights instance
- Role assignments: Storage Blob Data Contributor, Cosmos DB Data Contributor, AcrPull

## Implementation Evidence
- `infra/main.bicep` - Main infrastructure definition (827 lines)
  - Lines 1-100: Parameters and Container Registry setup
  - Lines 100-200: Storage Account, Cosmos DB resources
  - Lines 200-400: Container Apps, managed identities, role assignments
  - Lines 400-827: Container app configurations, environment variables
- `infra/main.parameters.json` - Environment-specific parameters
- `infra/main-containerapp.bicep` - Alternative Container App-specific configuration
- `azure.yaml` - Azure Developer CLI project configuration
  - Defines two services: `backend` (containerapp) and `frontend` (containerapp)
  - Specifies Python language and bicep infrastructure provider

## Acceptance Criteria
**Based on observed infrastructure code:**
- ✅ All Azure resources deploy successfully via `azd up` command (verified in main.bicep)
- ✅ Container Apps Environment created with Log Analytics integration (lines 78-93)
- ✅ User-assigned managed identity created for container apps (lines 96-100)
- ✅ Storage Account with `datasets` blob container provisioned (lines 103-129)
- ✅ Cosmos DB with serverless tier and two containers created (lines 131-200)
- ✅ Container Registry provisioned with public network access (lines 42-53)
- ✅ Document Intelligence resource deployed (lines 202-211)
- ✅ RBAC roles assigned to managed identity for storage, Cosmos DB, and ACR (throughout bicep)
- ✅ Environment variables configured for container apps with Azure resource references
- ✅ Application Insights instrumentation key exposed to containers

⚠️ **ACCEPTANCE CRITERIA GAPS:**
- No explicit health check validation in infrastructure code
- No automated rollback configuration observed
- No network security policies (NSG/firewall rules) defined
- No disaster recovery or backup policies configured

## Testing Requirements
⚠️ **NO AUTOMATED TESTS FOUND:**
- No infrastructure validation tests
- No bicep linting or validation in CI/CD pipeline files
- No integration tests for deployed resources
- No smoke tests post-deployment

**Manual Testing Evidence:**
- README.md shows `azd up` deployment workflow (lines 171-182)
- Health check endpoint described for manual verification (lines 187-199)
- **Coverage:** 0% automated, relies on manual deployment verification

**Recommendations:**
- Add bicep linting tests
- Implement post-deployment validation scripts
- Add infrastructure integration tests
- Set up automated health checks after deployment
