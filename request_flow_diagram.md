# RAG System Request Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend<br/>(script.js)
    participant API as FastAPI<br/>(app.py)
    participant RAG as RAG System<br/>(rag_system.py)
    participant AI as AI Generator<br/>(ai_generator.py)
    participant TM as Tool Manager<br/>(search_tools.py)
    participant VS as Vector Store<br/>(vector_store.py)
    participant DB as ChromaDB
    participant Claude as Anthropic Claude API

    U->>FE: Types question & clicks send
    Note over FE: sendMessage() triggered
    FE->>FE: Disable input, show loading
    FE->>API: POST /api/query<br/>{query, session_id}
    
    Note over API: FastAPI endpoint
    API->>API: Validate QueryRequest (Pydantic)
    API->>RAG: rag_system.query(query, session_id)
    
    Note over RAG: Main orchestrator
    RAG->>RAG: Build prompt, get history
    RAG->>AI: generate_response(query, tools, history)
    
    Note over AI: Anthropic client
    AI->>Claude: API call with tools<br/>{messages, system, tools}
    Claude-->>AI: Response with tool_use
    
    Note over AI: Tool execution needed
    AI->>TM: execute_tool("search_course_content")
    TM->>VS: search(query, course_name, lesson)
    
    Note over VS: Vector search
    VS->>DB: ChromaDB query<br/>(semantic similarity)
    DB-->>VS: Similar documents + metadata
    VS-->>TM: SearchResults
    TM->>TM: Format results, track sources
    TM-->>AI: Formatted search results
    
    AI->>Claude: Second API call<br/>{messages + tool_results}
    Claude-->>AI: Final natural language response
    AI-->>RAG: Generated answer
    
    RAG->>TM: get_last_sources()
    TM-->>RAG: Source list
    RAG-->>API: (answer, sources)
    
    Note over API: Response assembly
    API->>API: Create QueryResponse
    API-->>FE: JSON response<br/>{answer, sources, session_id}
    
    Note over FE: UI update
    FE->>FE: Remove loading, parse JSON
    FE->>FE: Render markdown (marked.js)
    FE->>FE: Display answer + sources
    FE->>U: Show response in chat
```

## Key Technologies at Each Layer

### Frontend Layer

- **JavaScript ES6+**: Event handling, DOM manipulation
- **Fetch API**: HTTP requests
- **Marked.js**: Markdown to HTML conversion

### API Layer  

- **FastAPI**: Web framework with automatic validation
- **Pydantic**: Request/response models
- **CORS Middleware**: Cross-origin support

### Processing Layer

- **RAG System**: Orchestrates components
- **Session Manager**: Conversation history
- **Tool Manager**: Plugin architecture

### AI Layer

- **Anthropic Client**: Claude API integration  
- **Tool Calling**: Function calling capability
- **System Prompts**: Specialized instructions

### Search Layer

- **Vector Store**: Semantic search orchestration
- **ChromaDB**: Vector database with persistence
- **Sentence Transformers**: Text embeddings

## Data Flow Summary

1. **User Input** → Frontend JavaScript
2. **HTTP Request** → FastAPI endpoint  
3. **Query Processing** → RAG System orchestration
4. **AI Generation** → Claude with tool access
5. **Vector Search** → ChromaDB semantic similarity
6. **Result Synthesis** → Claude final response
7. **JSON Response** → Frontend rendering
8. **UI Update** → User sees answer + sources
