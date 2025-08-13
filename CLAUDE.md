# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install/sync dependencies
uv sync

# Add new dependency
uv add package_name
```

### Environment Setup
Create `.env` file in root with:
```
ANTHROPIC_API_KEY=your_api_key_here
```

## Architecture Overview

This is a Retrieval-Augmented Generation (RAG) system for querying course materials using semantic search and AI-powered responses.

### Core Components Flow

**RAG System (`rag_system.py`)**: Central orchestrator that coordinates all components
- Manages document processing, vector storage, AI generation, and session state
- Handles tool-based search workflow where Claude decides when to search vs. use general knowledge

**Document Processing Pipeline**:
1. `DocumentProcessor` parses structured course documents with expected format:
   ```
   Course Title: [title]
   Course Link: [url]
   Course Instructor: [instructor]
   
   Lesson 0: Introduction
   Lesson Link: [lesson_url]
   [content...]
   ```
2. Text is chunked using sentence-based splitting with configurable overlap
3. Chunks get contextual prefixes: "Course {title} Lesson {number} content: {chunk}"

**Vector Storage (`vector_store.py`)**: ChromaDB integration with sentence transformers
- Stores course metadata and content chunks with embeddings
- Supports filtered semantic search by course name and lesson number

**AI Generation (`ai_generator.py`)**: Anthropic Claude integration with tool calling
- Uses tool-based architecture where Claude can choose to search or answer from knowledge
- Handles multi-step tool execution workflow: initial response → tool calls → final synthesis

**Search Tools (`search_tools.py`)**: Extensible tool system
- `CourseSearchTool` provides semantic search with course/lesson filtering
- `ToolManager` handles tool registration and execution
- Tracks sources for UI display

### Query Processing Flow

1. Frontend sends query to `/api/query` endpoint
2. `RAGSystem.query()` builds prompt and calls AI generator with available tools
3. Claude decides whether to use `search_course_content` tool based on query type
4. If searching: Vector store performs semantic search with optional filters
5. Tool results are sent back to Claude for final response synthesis
6. Response includes both answer and source attributions

### Session Management

- `SessionManager` maintains conversation history per session
- Configurable history length (default: 2 exchanges)
- Session IDs are created automatically and maintained by frontend

### Configuration

All settings centralized in `config.py`:
- Chunk size/overlap: 800/100 characters
- Embedding model: `all-MiniLM-L6-v2`
- Claude model: `claude-sonnet-4-20250514`
- Max search results: 5
- ChromaDB path: `./chroma_db`

### Document Loading

On startup, the application automatically loads course documents from the `docs/` folder. Documents should follow the structured format above for proper parsing into courses and lessons.

### Frontend Architecture

Single-page application with vanilla HTML/CSS/JS:
- Real-time chat interface with markdown rendering
- Session management and conversation history
- Collapsible sources display for result traceability
- Course statistics sidebar with auto-loading
- always use uv to run the server do not use pip directly
- use uv to run Python files