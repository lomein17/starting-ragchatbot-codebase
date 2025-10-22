# Course Materials RAG System

## Claude Code Preferences

### Task Communication Style
When a task is approved for execution:
- Break the task into clear, actionable steps
- Cross out each step as it's completed (using ~~strikethrough~~ in markdown)
- Keep the user aware of progress throughout the implementation
- This applies to all approved tasks, whether from plan mode or direct execution

## Project Overview

This is a Retrieval-Augmented Generation (RAG) system designed to answer questions about course materials using semantic search and AI-powered responses. The application enables users to query course documents and receive intelligent, context-aware answers with conversation history support.

## Tech Stack

### Backend
- **Python**: 3.13+
- **Web Framework**: FastAPI with Uvicorn
- **LLM Provider**: Anthropic Claude (claude-sonnet-4-20250514)
- **Vector Database**: ChromaDB (v1.0.15)
- **Embeddings**: sentence-transformers (all-MiniLM-L6-v2 model)
- **Package Manager**: uv

### Frontend
- Simple HTML/CSS/JavaScript interface
- Served as static files through FastAPI

### Key Dependencies
- `chromadb==1.0.15` - Vector database for semantic search
- `anthropic==0.58.2` - Claude AI integration
- `sentence-transformers==5.0.0` - Text embeddings
- `fastapi==0.116.1` - Web framework
- `uvicorn==0.35.0` - ASGI server
- `python-dotenv==1.1.1` - Environment configuration

## Architecture

### Core Components

The system follows a modular architecture with clear separation of concerns:

#### 1. **RAG System** ([backend/rag_system.py](backend/rag_system.py))
Main orchestrator that coordinates all components. Handles:
- Adding course documents individually or in bulk
- Processing user queries with conversation context
- Coordinating search tools and AI generation
- Managing course analytics

#### 2. **Document Processor** ([backend/document_processor.py](backend/document_processor.py))
Processes course documents and converts them into structured data:
- Parses PDF, DOCX, and TXT files
- Extracts course metadata (title, lessons, objectives)
- Chunks content into manageable pieces (800 chars with 100 char overlap)
- Creates `Course` and `CourseChunk` objects

#### 3. **Vector Store** ([backend/vector_store.py](backend/vector_store.py))
Manages ChromaDB vector database operations:
- Stores course metadata in "course_metadata" collection
- Stores course content chunks in "course_content" collection
- Performs semantic similarity searches
- Handles duplicate detection by course title

#### 4. **AI Generator** ([backend/ai_generator.py](backend/ai_generator.py))
Interfaces with Anthropic's Claude API:
- Generates responses using Claude Sonnet 4
- Supports tool calling for search functionality
- Handles multi-turn conversations with history
- Implements agentic loop for tool use

#### 5. **Session Manager** ([backend/session_manager.py](backend/session_manager.py))
Manages conversation state:
- Creates and tracks user sessions
- Maintains conversation history (last 2 exchanges by default)
- Provides context for follow-up questions

#### 6. **Search Tools** ([backend/search_tools.py](backend/search_tools.py))
Implements tool-based search pattern:
- `CourseSearchTool` - Semantic search over course content
- `ToolManager` - Registers and executes tools
- Tracks sources from searches for citation

#### 7. **Models** ([backend/models.py](backend/models.py))
Data models using Pydantic/dataclasses:
- `Course` - Course metadata structure
- `Lesson` - Individual lesson structure
- `CourseChunk` - Chunked content with metadata

#### 8. **FastAPI App** ([backend/app.py](backend/app.py))
Web server and API endpoints:
- `POST /api/query` - Process user queries
- `GET /api/courses` - Get course statistics
- Serves frontend static files
- Loads documents on startup from `/docs` folder

#### 9. **Configuration** ([backend/config.py](backend/config.py))
Centralized configuration using environment variables:
- API keys (loaded from `.env`)
- Model settings
- Chunk sizes and search parameters
- Database paths

## Data Flow

1. **Document Ingestion**: Course documents → Document Processor → Structured chunks → Vector Store
2. **Query Processing**: User query → RAG System → AI Generator (with search tools) → ChromaDB search → Claude generation → Response with sources
3. **Session Management**: Each query maintains conversation context through session IDs

## Development Guidelines

### Running the Application

**Quick start:**
```bash
./run.sh
```

**Manual start:**
```bash
cd backend
uv run uvicorn app:app --reload --port 8000
```

Access at:
- Web UI: http://localhost:8000
- API docs: http://localhost:8000/docs

### Environment Setup

Required environment variables in `.env`:
```
ANTHROPIC_API_KEY=your_api_key_here
```

### Adding New Documents

Place course documents (PDF, DOCX, TXT) in the `/docs` folder. They will be automatically loaded on server startup. The system skips documents that have already been processed (based on course title).

### Configuration Parameters

Key settings in [backend/config.py](backend/config.py):
- `CHUNK_SIZE`: 800 characters (adjust for different document types)
- `CHUNK_OVERLAP`: 100 characters (provides context continuity)
- `MAX_RESULTS`: 5 search results (balance between context and token usage)
- `MAX_HISTORY`: 2 conversation turns (controls conversation memory)
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2

### Code Style Notes

- Type hints are used throughout for clarity
- Docstrings follow Google style
- Error handling with try/except and meaningful error messages
- CORS is wide open (`*`) - suitable for development, should be restricted for production
- ChromaDB warnings are suppressed in app.py startup

## Common Tasks

### Adding a New Search Tool

1. Create a new tool class inheriting from base tool pattern in [backend/search_tools.py](backend/search_tools.py)
2. Implement `get_definition()` and `execute()` methods
3. Register with `ToolManager` in RAG system initialization

### Modifying Chunking Strategy

Edit `CHUNK_SIZE` and `CHUNK_OVERLAP` in [backend/config.py](backend/config.py). Larger chunks = more context but fewer precise matches. Smaller chunks = more precise but less context.

### Changing LLM Model

Update `ANTHROPIC_MODEL` in [backend/config.py](backend/config.py) to any supported Claude model.

### Clearing the Vector Database

The ChromaDB is persisted at `./chroma_db`. To rebuild from scratch, either:
- Delete the `chroma_db` folder
- Use `add_course_folder(path, clear_existing=True)`

## Important Notes

- **Document Format**: Course documents should follow a structured format with sections for Overview, Lessons, Objectives, etc.
- **Conversation Context**: Sessions are stored in memory only (not persisted)
- **Tool-Based RAG**: The system uses Claude's tool calling feature for search rather than simple retrieval + generation
- **Duplicate Prevention**: Documents with the same title are automatically skipped
- **Startup Behavior**: Documents in `/docs` are loaded automatically on server startup

## Project Structure

```
.
├── backend/                 # Python backend
│   ├── app.py              # FastAPI application
│   ├── rag_system.py       # Main RAG orchestrator
│   ├── vector_store.py     # ChromaDB interface
│   ├── ai_generator.py     # Claude AI integration
│   ├── document_processor.py # Document parsing and chunking
│   ├── session_manager.py  # Conversation state management
│   ├── search_tools.py     # Tool-based search implementation
│   ├── models.py           # Data models
│   └── config.py           # Configuration management
├── frontend/               # Static web interface
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── docs/                   # Course documents (auto-loaded)
├── .env                    # Environment variables (not in git)
├── pyproject.toml          # Python dependencies
├── run.sh                  # Startup script
└── README.md               # User documentation
```

## Testing

Currently no automated tests. Manual testing via:
- Web UI at http://localhost:8000
- API docs/Swagger UI at http://localhost:8000/docs
- Direct API calls to `/api/query` and `/api/courses`
