# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation

```bash
# Install dependencies
uv sync

# Create environment configuration
echo "ANTHROPIC_API_KEY=your_key_here" > .env
```

### Running the Application

```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Access Points

- Web Interface: `http://localhost:8000`
- API Documentation: `http://localhost:8000/docs`

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for course materials with a tool-calling architecture where Claude intelligently decides when to search.

### Core Components

**Frontend (`/frontend/`)**:

- Vanilla JavaScript ES6+ with Fetch API
- Chat interface with markdown rendering (marked.js)
- Session management for conversation continuity

**Backend (`/backend/`)**:

- **FastAPI** web framework with auto-validation
- **Modular RAG architecture** with specialized components:
  - `rag_system.py` - Main orchestrator, coordinates all components
  - `document_processor.py` - Chunks course documents by lessons with context preservation
  - `vector_store.py` - ChromaDB integration with dual collections (catalog + content)
  - `ai_generator.py` - Anthropic Claude client with tool execution
  - `search_tools.py` - Tool framework for Claude function calling
  - `session_manager.py` - Conversation history management

### Data Flow Pattern

1. **Document Processing**: Structured course documents → sentence-based chunks with lesson context
2. **Vector Storage**: ChromaDB with two collections for course metadata and content
3. **Query Processing**: RAG system orchestrates search and generation
4. **AI Tool Calling**: Claude decides whether to search and executes tools autonomously
5. **Response Generation**: Claude synthesizes final answer from search results

### Key Technologies

- **Vector Search**: ChromaDB + sentence-transformers (all-MiniLM-L6-v2)
- **AI Generation**: Anthropic Claude with tool calling capability
- **Document Structure**: Expects formatted course documents with lesson markers
- **Session Management**: Maintains conversation context across queries

## Configuration

**Environment Variables** (`.env`):

- `ANTHROPIC_API_KEY` - Required for Claude API access

**System Settings** (`backend/config.py`):

- `CHUNK_SIZE: 800` - Text chunk size for vector storage
- `CHUNK_OVERLAP: 100` - Character overlap between chunks
- `MAX_RESULTS: 5` - Maximum search results returned
- `ANTHROPIC_MODEL` - Claude model version
- `CHROMA_PATH` - Vector database storage location

## Document Processing

Course documents must follow this structure:

```text
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [optional_url]
[lesson content...]

Lesson 1: Next Topic
[lesson content...]
```

Documents are processed into:

- **Course objects** with metadata
- **CourseChunk objects** with lesson context for vector search
- **Automatic chunking** preserves sentence boundaries with overlap

## Development Notes

- **Package Manager**: Uses `uv` instead of pip/poetry
- **No Testing Framework**: No test commands currently configured
- **Document Storage**: Course materials in `/docs/` folder
- **Database**: ChromaDB creates persistent storage in `./chroma_db`
- **Tool Architecture**: Plugin-style tool system for Claude function calling
- **Frontend Simplicity**: Vanilla JS chosen for educational clarity over TypeScript complexity

- Use uv to run Python code.
