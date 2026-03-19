# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A RAG (Retrieval-Augmented Generation) chatbot for querying course materials. Uses ChromaDB for vector storage, Anthropic Claude for AI generation, and FastAPI for the backend.

## Commands

```bash
# Install dependencies
uv sync

# Run the application (from project root)
./run.sh
# Or manually:
cd backend && uv run uvicorn app:app --reload --port 8000

# Access points
# Web UI: http://localhost:8000
# API docs: http://localhost:8000/docs
```

## Environment Setup

Uses AWS Bedrock via SSO. Before running:
```bash
GRANTED_ALIAS_CONFIGURED="true" assume --ex --es genai-internal-access-bedrock-access
```

The `.env` file configures `AWS_PROFILE` and `AWS_REGION` (defaults already set).

## Architecture

```
User Query → FastAPI (app.py) → RAGSystem → AIGenerator (Claude API)
                                    ↓
                              ToolManager → CourseSearchTool → VectorStore (ChromaDB)
```

**RAGSystem** (`backend/rag_system.py`): Main orchestrator that coordinates document processing, vector storage, AI generation, and session management.

**VectorStore** (`backend/vector_store.py`): Manages two ChromaDB collections:
- `course_catalog`: Course metadata for semantic course name matching
- `course_content`: Chunked course content for RAG retrieval

**AIGenerator** (`backend/ai_generator.py`): Handles Claude API calls with tool-based search. Claude decides when to call `search_course_content` tool.

**DocumentProcessor** (`backend/document_processor.py`): Parses course documents with expected format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]
Lesson N: [title]
Lesson Link: [url]
[content...]
```

**Search Flow**: Queries go through `CourseSearchTool` which uses vector similarity search with optional course/lesson filtering. Course names are resolved via semantic search against the catalog before filtering content.

## Key Configuration (backend/config.py)

- `AWS_PROFILE`: genai-internal-access-bedrock-access
- `AWS_REGION`: us-east-1
- `BEDROCK_MODEL`: us.anthropic.claude-sonnet-4-20250514-v1:0
- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters
- `MAX_RESULTS`: 5 search results
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2

## Data Models (backend/models.py)

- `Course`: Title (unique ID), course_link, instructor, lessons list
- `Lesson`: lesson_number, title, lesson_link
- `CourseChunk`: content, course_title, lesson_number, chunk_index
