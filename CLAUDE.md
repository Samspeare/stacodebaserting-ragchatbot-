# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a full-stack RAG (Retrieval-Augmented Generation) system for querying course materials built with Python FastAPI backend and vanilla JavaScript frontend. The system uses ChromaDB for vector storage, Anthropic Claude for AI generation, and semantic search with sentence transformers.

## Development Commands

### Environment Setup
```bash
# Install dependencies (requires uv package manager)
uv sync

# Create .env file with required API key
echo "ANTHROPIC_API_KEY=your_key_here" > .env
```

### Running the Application
```bash
# Quick start using provided script
chmod +x run.sh
./run.sh

# Manual start from backend directory
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Development Server
- Web interface: `http://localhost:8000`
- API docs: `http://localhost:8000/docs`
- The FastAPI server serves both the API endpoints and static frontend files

## Architecture Overview

### Core RAG Pipeline
The system follows a modular RAG architecture centered around `RAGSystem` class in `backend/rag_system.py`:

1. **Document Processing** (`document_processor.py`) - Converts course text files into structured chunks
2. **Vector Storage** (`vector_store.py`) - ChromaDB with dual collections (catalog + content)
3. **AI Generation** (`ai_generator.py`) - Claude integration with tool calling support
4. **Search Tools** (`search_tools.py`) - Semantic search with course/lesson filtering

### Key Components

**Backend Structure:**
- `app.py` - FastAPI server with `/api/query` and `/api/courses` endpoints
- `rag_system.py` - Main orchestrator coordinating all components
- `vector_store.py` - ChromaDB wrapper with separate collections for metadata vs content
- `ai_generator.py` - Anthropic Claude client with conversation history and tool support
- `session_manager.py` - Conversation session tracking with configurable history limit
- `models.py` - Pydantic data models for Course, Lesson, and CourseChunk structures

**Frontend Integration:**
- Static files served directly by FastAPI from `frontend/` directory
- JavaScript client uses `/api` endpoints with session management
- Markdown rendering for AI responses using marked.js library

### Data Flow Architecture

1. **Startup**: Documents from `docs/` folder are processed and stored in ChromaDB collections
2. **Query Processing**: RAGSystem coordinates search tools → vector retrieval → Claude generation
3. **Dual Vector Collections**: 
   - `course_catalog` - Course metadata for semantic course name resolution
   - `course_content` - Actual lesson content chunks for retrieval
4. **Tool-Based Search**: Claude uses CourseSearchTool for dynamic vector searches during response generation

### Configuration Management
All settings centralized in `backend/config.py`:
- Anthropic API settings (model: claude-sonnet-4-20250514)
- Embedding model (all-MiniLM-L6-v2)
- Chunking parameters (800 char chunks, 100 char overlap)
- Database paths and conversation limits

### Session Management
Conversation context maintained through SessionManager with configurable history limits. Session IDs track user conversations across API calls.