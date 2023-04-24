# OpenReview ChatGPT Powered Research Assistant

Building the research assistant involves four steps:

1. Pre-process the accepted submissions of major venues
2. Store embeddings in Postgres with [pgvector](https://supabase.com/docs/guides/database/extensions/pgvector).
3. Perform vector similarity search to find the content that's relevant to the question.
4. Inject content into OpenAI GPT-4 text completion prompt and stream response to the client.

## Build time

Step 1 and 2 are pre-processing steps that are run asynchronously:

```mermaid
sequenceDiagram
    participant OR Server
    participant DB (pgvector)
    participant OpenAI (API)
    loop 1. Pre-process the knowledge base
        OR Server->>OR Server: Chunk .pdf files into sections
        loop 2. Create & store embeddings
            OR Server->>OpenAI (API): request embedding for pdf section
            OpenAI (API)->>OR Server: embedding vector(1536)
            OR Server->>DB (pgvector): store embedding for pdf section
        end
    end
```

## Runtime

Step 3 and 4 happen at runtime, anytime the user submits a question. When this happens, the following sequence of tasks is performed:

```mermaid
sequenceDiagram
    participant Client
    participant OR Server
    participant DB (pgvector)
    participant OpenAI (API)
    Client->>OR Server: { query: lorem ispum }
    critical 3. Perform vector similarity search
        OR Server->>OpenAI (API): request embedding for query
        OpenAI (API)->>OR Server: embedding vector(1536)
        OR Server->>DB (pgvector): vector similarity search
        DB (pgvector)->>OR Server: relevant pdf content
    end
    critical 4. Inject content into prompt
        OR Server->>OpenAI (API): completion request prompt: base prompt + relevant pdf content + query
        OpenAI (API)-->>Client: text/event-stream: completions response
    end
```
