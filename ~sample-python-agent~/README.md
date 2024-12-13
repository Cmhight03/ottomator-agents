# Base Sample Python Agent

Author: [Cole Medin](https://www.youtube.com/@ColeMedin)

This is a sample Python FastAPI agent that demonstrates the minimal required components to build an agent for the Live Agent Studio. It serves as a template and reference implementation for creating new Python-based agents.

## Overview

This agent provides a foundation for building AI-powered agents that can:
- Process natural language queries
- Maintain conversation history
- Integrate with external AI models
- Store and retrieve conversation data
- Handle authentication and security

The agent comes in two variants:
1. **Supabase Version** (`sample_supabase_agent.py`): Uses Supabase client for database operations
2. **Postgres Version** (`sample_postgres_agent.py`): Uses direct PostgreSQL connection via asyncpg

## Prerequisites

- Python 3.11 or higher
- pip (Python package manager)
- PostgreSQL database or Supabase account
- Basic understanding of:
  - FastAPI and async Python
  - RESTful APIs
  - Pydantic models
  - Environment variables
  - PostgreSQL (for Postgres version)

## Core Components

### 1. FastAPI Application (`sample_supabase_agent.py and sample_postgres_agent.py`)

The main application is built using FastAPI, providing:

- **Authentication**
  - Bearer token validation via environment variables
  - Secure endpoint protection
  - Customizable security middleware

- **Request Handling**
  - Async endpoint processing
  - Structured request validation
  - Error handling and HTTP status codes

- **Database Integration**
  - Supabase connection management
  - Message storage and retrieval
  - Session-based conversation tracking

### 2. Data Models

#### Request Model
```python
class AgentRequest(BaseModel):
    query: str        # The user's input text
    user_id: str      # Unique identifier for the user
    request_id: str   # Unique identifier for this request
    session_id: str   # Current conversation session ID
```

#### Response Model
```python
class AgentResponse(BaseModel):
    success: bool     # Indicates if the request was processed successfully
```

### 3. Database Schema

The agent uses Supabase tables with the following structure:

#### Messages Table
```sql
messages (
    id: uuid primary key
    created_at: timestamp with time zone
    session_id: text
    message: jsonb {
        type: string       # 'human' or 'assistant'
        content: string    # The message content
        data: jsonb       # Optional additional data
    }
)
```

## Database Setup

### Option 1: Supabase (sample_supabase_agent.py)
Required environment variables:
```plaintext
SUPABASE_URL=your-project-url
SUPABASE_SERVICE_KEY=your-service-key
API_BEARER_TOKEN=your-chosen-token
```

### Option 2: PostgreSQL (sample_postgres_agent.py)
1. **Create Database Tables**
   ```sql
   CREATE TABLE messages (
       id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
       created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
       session_id TEXT NOT NULL,
       message JSONB NOT NULL
   );

   CREATE INDEX idx_messages_session_id ON messages(session_id);
   CREATE INDEX idx_messages_created_at ON messages(created_at);
   ```

2. **Set Environment Variables**
   ```plaintext
   DATABASE_URL=postgresql://user:password@localhost:5432/dbname
   API_BEARER_TOKEN=your-chosen-token
   ```

   The DATABASE_URL format is:
   ```
   postgresql://[user]:[password]@[host]:[port]/[database_name]
   ```

3. **Connection Pool Management**
   The Postgres version automatically manages a connection pool:
   - Pool is created on application startup
   - Connections are automatically acquired and released
   - Pool is properly closed on application shutdown

## Setting Up Your Development Environment

1. **Clone and Install**
   ```bash
   # Clone the repository
   git clone https://github.com/coleam00/ottomator-agents.git
   cd ottomator-agents/~sample-python-agent~

   # Create and activate virtual environment (recommended)
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate

   # Install dependencies
   pip install -r requirements.txt
   ```

2. **Configure Environment**
   ```bash
   # Copy example environment file
   cp .env.example .env

   # Edit .env with your credentials
   nano .env  # or use your preferred editor
   ```

## Making Your First Request

Test your agent using curl or any HTTP client:

### Supabase Version
```bash
curl -X POST http://localhost:8001/api/sample-supabase-agent \
  -H "Authorization: Bearer your-token-here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Hello, agent!",
    "user_id": "test-user",
    "request_id": "test-request-1",
    "session_id": "test-session-1"
  }'
```

### Postgres Version
```bash
curl -X POST http://localhost:8001/api/sample-postgres-agent \
  -H "Authorization: Bearer your-token-here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Hello, agent!",
    "user_id": "test-user",
    "request_id": "test-request-1",
    "session_id": "test-session-1"
  }'
```

## Building Your Own Agent

1. **Fork the Template**
   - Create a copy of this sample agent
   - Rename files and update paths as needed

2. **Customize the Agent Logic**
   - Locate the TODO section in `sample_supabase_agent.py` or `sample_postgres_agent.py`
   - Add your custom agent logic:
     ```python
     # Example: Add AI model integration
     async def get_ai_response(query: str, history: List[Dict]) -> str:
         # Add your AI model logic here
         # e.g., OpenAI, Anthropic, or custom models
         return "AI response"

     @app.post("/api/your-agent", response_model=AgentResponse)
     async def your_agent(request: AgentRequest):
         # Get conversation history
         history = await fetch_conversation_history(request.session_id)
         
         # Process with your AI model
         response = await get_ai_response(request.query, history)
         
         # Store the response
         await store_message(
             session_id=request.session_id,
             message_type="assistant",
             content=response
         )
         
         return AgentResponse(success=True)
     ```

3. **Add Dependencies**
   - Update `requirements.txt` with new packages
   - Document any external services or APIs

4. **Implement Error Handling**
   ```python
   try:
       result = await your_operation()
   except YourCustomError as e:
       raise HTTPException(
           status_code=400,
           detail=f"Operation failed: {str(e)}"
       )
   ```

## Troubleshooting

Common issues and solutions:

1. **Authentication Errors**
   - Verify bearer token in environment
   - Check Authorization header format
   - Ensure token matches exactly

2. **Database Connection Issues**
   - For Supabase:
     - Verify Supabase credentials
     - Validate table permissions
   - For PostgreSQL:
     - Check DATABASE_URL format
     - Verify database user permissions
     - Ensure database is running and accessible
     - Check if tables are created correctly

3. **Performance Problems**
   - Check database query performance
   - Consider caching frequently accessed data
   - For PostgreSQL:
     - Monitor connection pool usage
     - Adjust pool size if needed (default is reasonable for most cases)

## Contributing

This agent is part of the oTTomator agents collection. For contributions or issues, please refer to the main repository guidelines.