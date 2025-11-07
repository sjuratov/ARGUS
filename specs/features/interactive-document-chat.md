# Interactive Document Chat

## Overview
Interactive Document Chat enables users to ask natural language questions about processed documents and receive AI-generated responses based on document content and extracted data. The feature combines a Streamlit-based chat interface with Azure OpenAI-powered backend that uses document context (extracted data and OCR) to provide accurate, contextual answers about document information.

**Business Purpose**: Transform static document processing into an interactive experience where users can explore document content through natural conversation, reducing time spent manually searching through documents and extraction results.

## User Stories

Based on implemented functionality:

- **As a user**, I want to ask questions about my processed documents, so that I can quickly find specific information
- **As a business analyst**, I want to query document content in natural language, so that I don't need to understand JSON structure
- **As a reviewer**, I want to verify extraction accuracy through questions, so that I can validate data quality
- **As a user**, I want chat history maintained during my session, so that I can have multi-turn conversations
- **As a user**, I want to see token usage, so that I understand cost implications of my queries
- **As a user**, I want to clear chat history, so that I can start fresh conversations per document

## Functional Requirements

### Chat Interface Component

- Reusable `DocumentChatComponent` class for Streamlit integration
- Document-specific conversation threads
- Chat history persisted in session state (per document)
- User message input via `st.chat_input` widget
- Message history display with role-based styling (user vs. assistant)
- Loading spinner during API request
- Clear chat button to reset conversation history
- Token usage metrics in expandable section

### Backend Chat API

- POST `/api/chat` endpoint accepts:
  - `document_id`: Identifier for document to query
  - `message`: User's question or query
  - `chat_history`: Previous conversation context (optional)
- Retrieves document from Cosmos DB by document_id
- Extracts document context:
  - GPT extraction results (structured data)
  - OCR text content
- Constructs system prompt with document context
- Calls Azure OpenAI Chat Completions API
- Returns response with:
  - `response`: AI-generated answer
  - `token_usage`: Prompt, completion, and total tokens
- 30-second timeout for API requests
- Error handling with user-friendly messages

### Document Context Integration

- Extracts structured data from `gpt_extraction_data` field
- Includes OCR text for comprehensive context
- Formats context into readable prompt structure
- System prompt includes:
  - Document metadata
  - Extracted structured data
  - Raw OCR content
  - Instructions for answering based on document

### Conversation Management

- Chat history stored per document in session state
- Session key format: `chat_history_{document_id}`
- Messages stored with role and content
- User messages appended immediately (optimistic UI)
- Assistant responses added after API call
- History passed to backend for context (implementation unclear)
- Clear history resets conversation for that document

### Token Usage Tracking

- Token metrics returned from Azure OpenAI
- Displays in expandable section:
  - Prompt tokens
  - Completion tokens
  - Total tokens
- Enables cost awareness and optimization

### Error Handling

- API timeout after 30 seconds
- Network error handling
- Document not found errors
- Malformed response handling
- User-friendly error messages displayed in UI

## Acceptance Criteria

**Chat Interface:**

- Given document selected, when chat accessed, then chat interface renders
- Given user enters message, when submitted, then message appears in history immediately
- Given message sent, when processing, then loading spinner displays
- Given response received, when returned, then assistant message appears in history
- Given clear chat clicked, when executed, then chat history cleared for that document
- Given token usage available, when displaying, then metrics shown in expander

**Backend API:**

- Given POST /api/chat called with document_id and message, when processing, then document retrieved from Cosmos DB
- Given document found, when processing, then extracted data and OCR loaded as context
- Given context prepared, when calling OpenAI, then system prompt includes document context
- Given OpenAI responds, when successful, then response and token usage returned
- Given API times out, when 30 seconds elapsed, then timeout error returned
- Given document not found, when searching, then not found error returned

**Document Context:**

- Given document has gpt_extraction_data, when preparing context, then structured data included
- Given document has OCR results, when preparing context, then text content included
- Given context built, when formatting, then readable structure created for prompt

**Conversation Flow:**

- Given user asks question, when processing, then question sent to backend
- Given backend responds, when returning, then answer relevant to document content
- Given multi-turn conversation, when continuing, then context maintained (if history passed)
- Given new document selected, when chatting, then separate conversation thread

**Token Usage:**

- Given OpenAI response, when returning, then token counts included
- Given token usage displayed, when viewing, then prompt, completion, and total shown
- Given multiple messages, when viewing, then token usage per message displayed

⚠️ **ACCEPTANCE CRITERIA GAPS:**

- Chat history from frontend not incorporated into backend API call (parameter exists but unclear if used)
- Document context not passed from frontend to backend (parameter exists but not utilized)
- No streaming response support (waits for full completion)
- No chat history persistence beyond browser session
- No maximum chat history length enforcement
- No cost tracking or rate limiting for chat interactions

## Non-Functional Requirements

**Usability:**

- Intuitive chat interface familiar to users (ChatGPT-like)
- Immediate feedback with optimistic UI updates
- Clear loading indicators during processing
- Error messages guide users on resolution

**Performance:**

- 30-second timeout prevents indefinite waits
- ⚠️ PERFORMANCE GAPS: No response streaming (slower UX)
- Token usage visibility enables optimization

**Reliability:**

- Error handling for common failure scenarios
- Graceful degradation on API failures
- ⚠️ RELIABILITY GAPS: No retry logic for transient failures
- Session state maintains history during navigation

**Cost:**

- Token usage tracking enables cost awareness
- ⚠️ COST GAPS: No cost estimation or budgets
- ⚠️ No rate limiting to prevent abuse
- No conversation length limits

**Privacy:**

- Chat history not persisted beyond session (privacy by default)
- ⚠️ PRIVACY GAPS: No option to save useful conversations
- No user authentication (single-tenant assumption)

## Dependencies & Constraints

**Dependencies:**

- Requires Streamlit frontend - Task 007
- Requires backend API with chat endpoint - Task 002
- Requires Azure OpenAI deployment - Task 001
- Requires Cosmos DB for document retrieval - Task 001
- Requires processed documents with extraction results - Task 004, Task 005

**Constraints:**

- Chat history limited to browser session (not persistent)
- No multi-user support (session state per browser)
- Requires document already processed (no chat on raw documents)
- Token limits apply to context size (may truncate large documents)
- 30-second timeout may be insufficient for complex queries
- No conversation context beyond single session
- Backend may not fully utilize chat history (implementation unclear)

**Technical Constraints:**

- Streamlit session state for history storage
- Azure OpenAI Chat Completions API
- Python requests library for backend calls
- JSON serialization for message passing

## Task Traceability

This feature is implemented by the following tasks:

- **Task 013**: Document Chat Interface (`specs/tasks/013-task-document-chat-interface.md`) - Complete frontend and backend implementation
- **Task 007**: Streamlit Frontend (`specs/tasks/007-task-streamlit-frontend.md`) - UI integration
- **Task 002**: Backend FastAPI Core (`specs/tasks/002-task-backend-fastapi-core.md`) - Chat API endpoint
- **Task 004**: GPT Vision Extraction (`specs/tasks/004-task-gpt-vision-extraction.md`) - Azure OpenAI integration

## Implementation Status

**Status**: ✅ Complete

**Complete:**

- DocumentChatComponent frontend class fully implemented
- Chat interface with message history and role styling
- User input via chat_input widget
- Backend API endpoint `/api/chat` fully implemented
- Document retrieval from Cosmos DB
- Document context extraction (structured data + OCR)
- System prompt construction with document context
- Azure OpenAI Chat Completions integration
- Response generation and return
- Token usage tracking and display
- Clear chat history functionality
- 30-second timeout configuration
- Error handling and user-friendly messages
- Loading indicators during processing
- Integration in Explore Data tab

**Incomplete/Unclear:**

- ⚠️ Chat history parameter exists but unclear if backend uses it for context
- ⚠️ Document context parameter in frontend not passed (design intention unclear)
- ⚠️ No conversation persistence beyond session
- ⚠️ No streaming response implementation
- ⚠️ No rate limiting or cost controls

**Testing:**

- ❌ No automated tests (0% coverage)
- Manual testing via web interface
- Interactive testing by design
- README mentions "Interactive Chat" capability

**Known Issues:**

- Chat history may not provide conversational context to backend
- No persistence means conversations lost on refresh
- Token usage accumulates without limits or warnings
- No streaming makes responses feel slower

**Recommendations:**

- Verify chat history integration in backend context
- Add automated tests for chat component and API
- Implement response streaming for better UX
- Add chat history persistence (optional feature)
- Implement rate limiting and cost controls
- Add conversation length limits
- Test with various document types and query complexity
- Add integration tests with mocked Azure OpenAI
- Implement conversation export functionality
- Add analytics on chat usage patterns
- Consider RAG implementation for better accuracy
