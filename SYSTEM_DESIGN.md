# DocuMind AI System Design

## 1. Project Overview

DocuMind AI is a full-stack AI document assistant. It allows authenticated users to upload PDF, DOCX, or TXT documents, extract text, generate summaries, extract structured information, ask document-grounded questions, and export summaries as PDF or DOCX.

The project has three main runtime areas:

- Frontend: React, Vite, TypeScript, Material UI.
- Backend: FastAPI, SQLAlchemy, Pydantic, Alembic.
- Data and AI infrastructure: PostgreSQL for relational data, ChromaDB for vector search, and configurable LLM providers such as OpenAI, Gemini, or Ollama.

## 2. High-Level Architecture

```text
User Browser
    |
    | React UI over HTTP
    v
Frontend App
    |
    | /api requests with JWT
    v
FastAPI Backend
    |
    | relational metadata
    v
PostgreSQL

FastAPI Backend
    |
    | uploaded document files
    v
Filesystem / uploads

FastAPI Backend
    |
    | vector embeddings
    v
ChromaDB / vector_store

FastAPI Backend
    |
    | prompts, summaries, chat answers, embeddings
    v
AI Provider
OpenAI / Gemini / Ollama / sentence-transformers
```

## 3. Main Components

### Frontend

The frontend is responsible for user interaction, routing, authentication state, document management UI, and calling backend APIs.

Key files:

- `frontend/src/main.tsx`: creates the React app, sets the Material UI theme, and installs routing.
- `frontend/src/App.tsx`: defines public and protected routes.
- `frontend/src/api/client.ts`: central Axios client that attaches JWT tokens to requests.
- `frontend/src/context/AuthContext.tsx`: stores user login state, restores sessions, and handles logout.
- `frontend/src/pages/DashboardPage.tsx`: shows upload UI and document library.
- `frontend/src/pages/DocumentPage.tsx`: shows document details, summaries, chat, and extraction tools.

Important UI components:

- `DocumentUploader`: uploads files with frontend validation.
- `DocumentLibrary`: lists and deletes user documents.
- `SummaryPanel`: generates and displays saved summaries.
- `ExtractionPanel`: extracts key points, dates, names, statistics, and definitions.
- `ChatInterface`: sends document-grounded questions and displays chat history.
- `ExportButton`: downloads generated summaries as PDF or DOCX.

### Backend

The backend exposes REST APIs, enforces authentication, validates ownership, processes uploaded documents, calls AI providers, and persists results.

Key files:

- `backend/app/main.py`: FastAPI entry point and router registration.
- `backend/app/config.py`: environment-based configuration.
- `backend/app/database.py`: SQLAlchemy engine and session dependency.
- `backend/app/models.py`: ORM models.
- `backend/app/schemas.py`: request and response models.
- `backend/app/exceptions.py`: custom domain exceptions.
- `backend/app/middleware/error_handler.py`: centralized error response mapping.

Main routers:

- `auth.py`: registration, login, logout, current user.
- `documents.py`: upload, list, detail, delete.
- `summaries.py`: summary generation and structured extraction.
- `chat.py`: RAG question answering and chat history.
- `export.py`: PDF and DOCX summary export.

Main services:

- `AuthenticationService`: email validation, password hashing, JWT creation and validation.
- `DocumentUploader`: validates files, stores uploads, runs parse/chunk/embed pipeline.
- `DocumentParser`: extracts text from PDF, DOCX, and TXT files.
- `TextChunker`: splits extracted text into token-bounded chunks.
- `EmbeddingGenerator`: generates chunk embeddings and stores them in ChromaDB.
- `SummarizationEngine`: generates summaries and extracts information with LLMs.
- `RAGEngine`: retrieves relevant chunks and generates document-grounded answers.
- `ExportGenerator`: creates PDF and DOCX exports from summaries.

## 4. Data Model

PostgreSQL stores user data and application metadata.

```text
users
  id
  email
  password_hash
  created_at
  updated_at

documents
  id
  user_id -> users.id
  filename
  file_path
  file_type
  file_size
  upload_date
  text_content
  chunk_count

summaries
  id
  document_id -> documents.id
  summary_type
  summary_text
  created_at
  generation_time_seconds
  model_used

chat_history
  id
  document_id -> documents.id
  session_id
  user_id -> users.id
  question
  answer
  source_chunks
  created_at
```

Deletion behavior:

- Deleting a user cascades to documents and chat history.
- Deleting a document cascades to summaries and chat history.
- Document deletion also attempts to remove the uploaded file and ChromaDB embeddings.

## 5. Vector Store Design

ChromaDB stores document chunk embeddings in the `document_embeddings` collection.

Each vector record contains:

- id: composed from document id and chunk id.
- document text: the chunk text.
- embedding: numeric vector.
- metadata:
  - `document_id`
  - `chunk_id`
  - `chunk_position`
  - `user_id`
  - `created_at`

The RAG engine queries ChromaDB with a question embedding and filters results by `document_id`, ensuring answers are grounded in the selected document.

## 6. Core Workflows

### User Authentication

```text
User submits email/password
    -> POST /api/auth/login
    -> backend verifies bcrypt password
    -> backend returns JWT
    -> frontend stores JWT in localStorage
    -> Axios sends JWT on future requests
```

The backend validates the token through `get_current_user`. Protected endpoints reject missing or invalid tokens.

### Document Upload Pipeline

```text
User uploads file
    -> frontend validates type and size
    -> POST /api/documents/upload
    -> backend validates extension and size
    -> backend writes file to uploads/{user_id}/{document_id}.{ext}
    -> DocumentParser extracts text
    -> TextChunker creates chunks
    -> EmbeddingGenerator creates embeddings
    -> ChromaDB stores embeddings
    -> PostgreSQL stores document metadata and extracted text
```

Supported formats:

- PDF
- DOCX
- TXT

Maximum file size:

- 50 MB

### Summary Generation

```text
User selects summary type
    -> POST /api/summaries/generate
    -> backend verifies document ownership
    -> SummarizationEngine loads extracted text
    -> TextChunker chunks long text
    -> LLM generates summary
    -> summary is saved in PostgreSQL
    -> frontend displays saved summary
```

Supported summary types:

- SHORT
- DETAILED
- EXECUTIVE
- BULLETS
- ELI10

For long documents, the backend uses a map-reduce pattern:

1. Summarize each chunk.
2. Combine partial summaries into a final summary.

### Structured Extraction

```text
User selects info type
    -> POST /api/summaries/extract
    -> backend verifies document ownership
    -> SummarizationEngine prompts LLM for requested info
    -> backend returns list of extracted items
```

Supported extraction types:

- KEY_POINTS
- DATES
- NAMES
- STATISTICS
- DEFINITIONS

### Document Chat / RAG

```text
User asks question
    -> POST /api/chat/query
    -> backend verifies document ownership
    -> RAGEngine embeds the question
    -> ChromaDB retrieves top matching chunks for that document
    -> backend loads recent chat history
    -> LLM answers using retrieved context
    -> backend saves question and answer
    -> frontend displays answer and sources
```

If no relevant chunks are found, the system returns a clear "not found in the document" answer.

### Export

```text
User clicks PDF or DOCX
    -> POST /api/export/pdf or /api/export/docx
    -> backend verifies summary ownership through document ownership
    -> ExportGenerator creates file bytes
    -> frontend downloads the file
```

## 7. API Surface

Authentication:

- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/me`

Documents:

- `POST /api/documents/upload`
- `GET /api/documents`
- `GET /api/documents/{document_id}`
- `DELETE /api/documents/{document_id}`

Summaries and extraction:

- `POST /api/summaries/generate`
- `GET /api/summaries/{document_id}`
- `POST /api/summaries/extract`

Chat:

- `POST /api/chat/query`
- `GET /api/chat/{document_id}/history`

Export:

- `POST /api/export/pdf`
- `POST /api/export/docx`

Health:

- `GET /health`

## 8. Security Design

Authentication:

- JWT bearer tokens are issued after login.
- Passwords are stored as bcrypt hashes.
- Protected routes use `get_current_user`.

Authorization:

- Every document, summary, chat, and export action verifies user ownership.
- Users can only access documents that belong to them.

Input validation:

- Frontend and backend both validate upload type and size.
- Backend rejects unsupported formats and corrupted files.

Secrets:

- API keys and JWT secrets come from environment variables.
- `.env` is ignored by Git.

## 9. Error Handling

The backend uses custom exceptions and a centralized error handler.

Examples:

- `UnsupportedFormatError`: unsupported upload type.
- `FileSizeError`: file exceeds size limit.
- `FileCorruptionError`: invalid file content.
- `AuthenticationError`: invalid credentials or token.
- `DocumentNotFoundError`: missing document.
- `LLMQuotaExceededError`: AI provider quota or rate limit reached.
- `SummarizationError`, `EmbeddingError`, `ExportError`: processing failures.

The frontend displays user-facing alerts and snackbars for failed uploads, failed summary generation, chat errors, and export errors.

## 10. Deployment Design

Docker Compose services:

- `postgres`: PostgreSQL database.
- `chromadb`: vector database.
- `backend`: FastAPI API server.
- `frontend`: Nginx serving the React build and proxying `/api` to backend.

Development mode:

- Vite frontend runs on port `5173`.
- FastAPI backend runs on port `8000`.
- Vite proxies `/api` requests to `localhost:8000`.

Production container mode:

- Frontend is served by Nginx on port `3000`.
- Nginx serves React routes through SPA fallback.
- Nginx proxies `/api` requests to the backend container.

## 11. Scalability Considerations

Current design works well for a small to medium local deployment. For larger usage, these improvements would be useful:

- Move uploads from local filesystem to object storage such as S3.
- Run embedding and summarization jobs asynchronously with a queue.
- Add background job status tracking for long documents.
- Add pagination for documents, summaries, and chat history.
- Use remote managed ChromaDB or another vector database.
- Add rate limiting per user.
- Add refresh tokens or token revocation if logout must invalidate tokens server-side.
- Add streaming responses for chat and summarization.

## 12. Testing Strategy

The backend includes unit and property-based tests.

Covered areas include:

- File upload validation.
- Document parsing.
- Text chunking and overlap behavior.
- Embedding storage behavior.
- Summary and extraction behavior.
- RAG source attribution and not-found responses.
- Export generation.
- Authentication invariants.
- Database constraints and cascade deletion.
- Error response behavior.

## 13. Design Summary

DocuMind AI is organized around a clear layered architecture:

```text
React UI
  -> API client
  -> FastAPI routers
  -> service layer
  -> PostgreSQL / ChromaDB / filesystem / AI providers
```

This separation keeps responsibilities clear:

- Frontend handles user experience.
- Routers handle HTTP and authorization.
- Services handle business logic.
- PostgreSQL stores durable application records.
- ChromaDB stores searchable semantic chunks.
- AI providers generate summaries, extracted facts, embeddings, and chat answers.

