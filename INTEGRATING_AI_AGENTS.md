# Integrating AI Agents with Airweave

This guide explains how to integrate AI agents with Airweave to enable semantic search capabilities across your connected data sources.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Integration Methods](#integration-methods)
  - [REST API](#rest-api)
  - [Python SDK](#python-sdk)
  - [TypeScript/JavaScript SDK](#typescriptjavascript-sdk)
  - [MCP (Message Control Protocol)](#mcp-message-control-protocol)
- [Implementation Examples](#implementation-examples)
  - [Frontend AI Agent Integration](#frontend-ai-agent-integration)
  - [Backend AI Agent Integration](#backend-ai-agent-integration)
- [Advanced Usage](#advanced-usage)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

Airweave serves as a backend service that provides APIs for AI agents to access and search data from various sources. It handles the complex tasks of connecting to data sources, processing and transforming the data, and making it available through simple API endpoints.

The typical architecture looks like this:

```
[Your Data Sources] → [Airweave Backend] → [APIs] → [Your AI Agent]
```

## Prerequisites

Before integrating your AI agent with Airweave, ensure you have:

1. Airweave running (locally or deployed)
2. Connected at least one data source
3. Created a collection or sync
4. Generated an API key

## Integration Methods

### REST API

The simplest way to integrate is through direct REST API calls.

#### Search Endpoint

```http
GET /collections/{collection_id}/search?query=your_query
```

**Example cURL request:**

```bash
curl -X 'GET' \
  'http://localhost:8001/collections/your-collection-id/search?query=your_search_query' \
  -H 'accept: application/json' \
  -H 'x-api-key: your-api-key' \
  -H 'Content-Type: application/json'
```

**Response:**

```json
{
  "results": [
    {
      "id": "doc-123",
      "content": "This is the content that matched your query",
      "metadata": {
        "source": "notion",
        "last_updated": "2023-06-15T10:30:00Z"
      },
      "score": 0.92
    },
    // More results...
  ],
  "status": "success"
}
```

### Python SDK

For Python-based agents, use the official Airweave SDK:

```bash
pip install airweave-sdk
```

**Example usage:**

```python
from airweave import AirweaveSDK

# Initialize the client
client = AirweaveSDK(api_key="your-api-key")

# Search in a collection
results = client.collections.search_collection(
    readable_id="your-collection-id",
    query="your search query"
)

# Process results in your agent
for item in results.results:
    print(f"Found match: {item.content} (Score: {item.score})")
    # Your agent logic here...
```

### TypeScript/JavaScript SDK

For JavaScript/TypeScript agents, use the official Airweave SDK:

```bash
npm install @airweave/sdk
# or
yarn add @airweave/sdk
```

**Example usage:**

```typescript
import { AirweaveSDKClient } from "@airweave/sdk";

// Initialize the client
const client = new AirweaveSDKClient({ 
  apiKey: "your-api-key",
  baseUrl: "http://localhost:8001" // Your Airweave instance URL
});

// Search for data
async function searchWithAgent(query) {
  const results = await client.collections.searchCollection("your-collection-id", {
    query: query
  });
  
  // Process results in your agent
  results.results.forEach(item => {
    console.log(`Found match: ${item.content} (Score: ${item.score})`);
    // Your agent logic here...
  });
  
  return results;
}
```

### MCP (Message Control Protocol)

Airweave supports the MCP standard, which is a protocol for AI agents to interact with tools and data sources.

```javascript
import { MCPClient } from "@mcp/client";

// Initialize the MCP client with your Airweave endpoint
const client = new MCPClient({
  endpoint: "http://localhost:8001/search",
  version: "0.1.0-beta"
});

// Create a context with your data sources
const context = client.createContext({
  syncId: "your-sync-id"
});

// Execute a search query with MCP
async function searchWithMCP() {
  const result = await context.search({
    query: "your search query",
    // Optional MCP-specific parameters
    retrieval: {
      strategy: "hybrid",
      depth: 3,
      sources: ["primary", "knowledge_graph"]
    }
  });

  console.log(result.data);
  console.log(result.metadata.context_used);
}
```

## Implementation Examples

### Frontend AI Agent Integration

Here's an example of integrating Airweave with a React-based frontend AI agent:

```jsx
import { useState } from 'react';
import { AirweaveSDKClient } from "@airweave/sdk";

// Initialize the client
const client = new AirweaveSDKClient({ 
  apiKey: "your-api-key",
  baseUrl: "http://localhost:8001"
});

function AIAgentComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState(null);
  const [agentResponse, setAgentResponse] = useState('');
  const [loading, setLoading] = useState(false);
  
  const handleSearch = async () => {
    setLoading(true);
    try {
      // 1. Search Airweave for relevant data
      const searchResults = await client.collections.searchCollection("your-collection-id", {
        query: query
      });
      
      setResults(searchResults);
      
      // 2. Process the results with your agent logic
      // This could be a local model or an API call to an LLM
      const agentReply = processResultsWithAgent(query, searchResults);
      setAgentResponse(agentReply);
      
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  // Your agent's logic to process the search results
  const processResultsWithAgent = (query, results) => {
    // This is where your agent would process the data
    // For example, you might:
    // 1. Format the data for an LLM prompt
    // 2. Call an LLM API with the context
    // 3. Process and format the response
    
    // Simplified example:
    return `Based on the ${results.results.length} results found, here's what I know about "${query}"...`;
  };
  
  return (
    <div>
      <h2>AI Assistant</h2>
      <div>
        <input 
          type="text" 
          value={query} 
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Ask about your data..."
        />
        <button onClick={handleSearch} disabled={loading}>
          {loading ? 'Searching...' : 'Search'}
        </button>
      </div>
      
      {agentResponse && (
        <div>
          <h3>Agent Response:</h3>
          <p>{agentResponse}</p>
        </div>
      )}
    </div>
  );
}
```

### Backend AI Agent Integration

For a backend service that uses Airweave:

```python
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel
from airweave import AirweaveSDK
import openai

app = FastAPI()

# Initialize Airweave client
airweave_client = AirweaveSDK(api_key="your-api-key")

class QueryRequest(BaseModel):
    query: str
    collection_id: str

class AgentResponse(BaseModel):
    answer: str
    sources: list

@app.post("/ask", response_model=AgentResponse)
async def ask_agent(request: QueryRequest):
    # 1. Search for relevant information using Airweave
    try:
        search_results = airweave_client.collections.search_collection(
            readable_id=request.collection_id,
            query=request.query
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Airweave search failed: {str(e)}")
    
    # 2. Format the context for the LLM
    context = "\n\n".join([
        f"Source {i+1}: {result.content}"
        for i, result in enumerate(search_results.results[:5])  # Use top 5 results
    ])
    
    # 3. Generate a response using an LLM
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "You are a helpful assistant. Answer based only on the provided context."},
                {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {request.query}"}
            ]
        )
        answer = response.choices[0].message.content
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"LLM processing failed: {str(e)}")
    
    # 4. Return the agent's response with sources
    return AgentResponse(
        answer=answer,
        sources=[{
            "content": result.content,
            "metadata": result.metadata
        } for result in search_results.results[:5]]
    )
```

## Advanced Usage

### Hybrid Search

Airweave supports hybrid search combining semantic and keyword search:

```python
results = client.collections.search_collection(
    readable_id="your-collection-id",
    query="your search query",
    search_type="hybrid",  # Options: "semantic", "keyword", "hybrid"
    hybrid_ratio=0.7  # Weight for semantic search (0.0 to 1.0)
)
```

### Response Types

You can request AI-generated completions based on search results:

```python
results = client.collections.search_collection(
    readable_id="your-collection-id",
    query="your search query",
    response_type="completion"  # Options: "raw", "completion"
)

# Access the AI-generated completion
print(results.completion)
```

## Troubleshooting

Common issues and solutions:

1. **Authentication errors**: Ensure your API key is valid and properly formatted
2. **No results returned**: Check that your collection has been properly synced with data
3. **CORS issues**: When calling from a browser, ensure CORS is properly configured
4. **Rate limiting**: Be aware of API rate limits, especially in production

## Best Practices

1. **Cache results** when appropriate to reduce API calls
2. **Use specific queries** rather than broad ones for better results
3. **Implement error handling** for a better user experience
4. **Consider pagination** for large result sets
5. **Provide source attribution** when displaying information from search results
6. **Use response streaming** for a more responsive user experience
