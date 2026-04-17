# CrewAI WebUI - Implementation Package

> Version: 1.0 | Created: April 2026

---

# Part 1: API Design & Contracts

## Base URL
```
http://localhost:8000/api/v1
```

## Authentication
```
Header: Authorization: Bearer <token>
```

---

## Tools API

### Create Tool
```
POST /tools
```

**Request:**
```json
{
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "code": "@tool(\"Web Search\")\ndef search(query: str) -> str:\n    \"\"\"\"Search the web\"\"\"\n    return f\"Results for: {query}\"",
  "input_schema": {
    "query": {
      "type": "string",
      "description": "Search query",
      "required": true
    }
  },
  "output_schema": {
    "results": {
      "type": "string",
      "description": "Search results"
    }
  }
}
```

**Response (201):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "is_active": true,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
}
```

### List Tools
```
GET /tools
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "tool-001",
      "name": "web_search",
      "description": "Search the web for information",
      "type": "decorator",
      "is_active": true,
      "version": 1
    },
    {
      "id": "tool-002",
      "name": "pdf_reader",
      "description": "Read PDF documents",
      "type": "basetool",
      "is_active": true,
      "version": 1
    }
  ],
  "total": 2,
  "page": 1,
  "page_size": 20
}
```

### Get Tool
```
GET /tools/{id}
```

**Response (200):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "code": "@tool(\"Web Search\")\ndef search(query: str) -> str:\n    \"\"\"Search the web\"\"\"\n    return f\"Results for: {query}\"",
  "input_schema": {
    "query": {
      "type": "string",
      "description": "Search query",
      "required": true
    }
  },
  "output_schema": {
    "results": {
      "type": "string",
      "description": "Search results"
    }
  },
  "is_active": true,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
}
```

### Update Tool
```
PUT /tools/{id}
```

**Request:**
```json
{
  "name": "web_search",
  "description": "Search the web for information",
  "code": "@tool(\"Web Search\")\ndef search(query: str, limit: int = 10) -> str:\n    \"\"\"Search the web\"\"\"\n    return f\"Results for: {query}\"",
  "input_schema": {
    "query": {
      "type": "string",
      "description": "Search query",
      "required": true
    },
    "limit": {
      "type": "integer",
      "description": "Max results",
      "required": false,
      "default": 10
    }
  }
}
```

**Response (200):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "version": 2,
  "updated_at": "2026-04-17T11:00:00Z"
}
```

### Delete Tool
```
DELETE /tools/{id}
```

**Response (204):** No content

---

## Agents API

### Create Agent
```
POST /agents
```

**Request:**
```json
{
  "name": "Market Researcher",
  "role": "Market Research Analyst",
  "goal": "Research and analyze market trends",
  "backstory": "Expert market researcher with 10+ years experience in competitive analysis",
  "llm_provider": "openai",
  "llm_model": "gpt-4o",
  "llm_config": {
    "temperature": 0.7,
    "max_tokens": 2000
  },
  "tools": ["tool-001", "tool-002"],
  "verbose": true,
  "allow_delegation": false,
  "max_iterations": 10
}
```

**Response (201):**
```json
{
  "id": "agent-001",
  "name": "Market Researcher",
  "role": "Market Research Analyst",
  "goal": "Research and analyze market trends",
  "backstory": "Expert market researcher with 10+ years experience",
  "llm_provider": "openai",
  "llm_model": "gpt-4o",
  "llm_config": {
    "temperature": 0.7,
    "max_tokens": 2000
  },
  "tools": ["tool-001", "tool-002"],
  "verbose": true,
  "allow_delegation": false,
  "max_iterations": 10,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### List Agents
```
GET /agents
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "agent-001",
      "name": "Market Researcher",
      "role": "Market Research Analyst",
      "goal": "Research and analyze market trends",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": ["tool-001", "tool-002"],
      "verbose": true
    },
    {
      "id": "agent-002",
      "name": "Content Writer",
      "role": "Technical Writer",
      "goal": "Create compelling content",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": ["tool-003"],
      "verbose": true
    }
  ],
  "total": 2
}
```

### Get Agent
```
GET /agents/{id}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "name": "Market Researcher",
  "role": "Market Research Analyst",
  "goal": "Research and analyze market trends",
  "backstory": "Expert market researcher with 10+ years experience",
  "llm_provider": "openai",
  "llm_model": "gpt-4o",
  "llm_config": {
    "temperature": 0.7,
    "max_tokens": 2000
  },
  "tools": [
    {"id": "tool-001", "name": "web_search"},
    {"id": "tool-002", "name": "pdf_reader"}
  ],
  "verbose": true,
  "allow_delegation": false,
  "max_iterations": 10,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### Add Tool to Agent
```
POST /agents/{agent_id}/tools
```

**Request:**
```json
{
  "tool_id": "tool-003"
}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "tools": ["tool-001", "tool-002", "tool-003"]
}
```

### Remove Tool from Agent
```
DELETE /agents/{agent_id}/tools/{tool_id}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "tools": ["tool-001", "tool-002"]
}
```

---

## Tasks API

### Create Task
```
POST /tasks
```

**Request:**
```json
{
  "title": "Research Competitors",
  "description": "Research top 5 competitors in the AI market",
  "expected_output": "List of competitors with market share and analysis",
  "agent_id": "agent-001",
  "context_tasks": [],
  "async_execution": false,
  "output_format": "text",
  "priority": "high",
  "timeout_seconds": 300
}
```

**Response (201):**
```json
{
  "id": "task-001",
  "title": "Research Competitors",
  "description": "Research top 5 competitors in the AI market",
  "expected_output": "List of competitors with market share and analysis",
  "agent_id": "agent-001",
  "context_tasks": [],
  "async_execution": false,
  "output_format": "text",
  "priority": "high",
  "timeout_seconds": 300,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### List Tasks
```
GET /tasks
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "task-001",
      "title": "Research Competitors",
      "description": "Research top 5 competitors",
      "expected_output": "List of competitors",
      "agent_id": "agent-001",
      "priority": "high"
    },
    {
      "id": "task-002",
      "title": "Write Report",
      "description": "Write market analysis report",
      "expected_output": "PDF report",
      "agent_id": "agent-002",
      "priority": "normal"
    }
  ],
  "total": 2
}
```

### Assign Agent to Task
```
POST /tasks/{task_id}/agent
```

**Request:**
```json
{
  "agent_id": "agent-002"
}
```

**Response (200):**
```json
{
  "id": "task-001",
  "agent_id": "agent-002"
}
```

### Set Task Context (Dependency)
```
POST /tasks/{task_id}/context
```

**Request:**
```json
{
  "context_tasks": ["task-001"]
}
```

**Response (200):**
```json
{
  "id": "task-002",
  "context_tasks": ["task-001"]
}
```

---

## Crews API

### Create Crew
```
POST /crews
```

**Request:**
```json
{
  "name": "Market Analysis Team",
  "description": "Team for comprehensive market analysis",
  "process": "sequential",
  "verbose": 1,
  "agents": ["agent-001", "agent-002", "agent-003"],
  "tasks": ["task-001", "task-002", "task-003"],
  "manager_agent_id": null,
  "memory": false,
  "cache": true,
  "can_trigger_other_crews": true,
  "trigger_chain_on": "completed",
  "output_fields": {
    "research_results": "string",
    "summary": "string",
    "recommendations": "string"
  }
}
```

**Response (201):**
```json
{
  "id": "crew-001",
  "name": "Market Analysis Team",
  "description": "Team for comprehensive market analysis",
  "process": "sequential",
  "verbose": 1,
  "agents": ["agent-001", "agent-002", "agent-003"],
  "tasks": ["task-001", "task-002", "task-003"],
  "manager_agent_id": null,
  "memory": false,
  "cache": true,
  "can_trigger_other_crews": true,
  "trigger_chain_on": "completed",
  "is_active": true,
  "version": 1,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### List Crews
```
GET /crews
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "crew-001",
      "name": "Market Analysis Team",
      "process": "sequential",
      "agents_count": 3,
      "tasks_count": 3,
      "can_trigger_other_crews": true,
      "is_active": true
    },
    {
      "id": "crew-002",
      "name": "Content Pipeline",
      "process": "hierarchical",
      "agents_count": 4,
      "tasks_count": 5,
      "can_trigger_other_crews": false,
      "is_active": true
    }
  ],
  "total": 2
}
```

### Get Crew with Full Details
```
GET /crews/{id}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "name": "Market Analysis Team",
  "description": "Team for comprehensive market analysis",
  "process": "sequential",
  "verbose": 1,
  "agents": [
    {
      "id": "agent-001",
      "name": "Market Researcher",
      "role": "Research Analyst"
    },
    {
      "id": "agent-002",
      "name": "Data Analyst",
      "role": "Data Expert"
    },
    {
      "id": "agent-003",
      "name": "Content Writer",
      "role": "Writer"
    }
  ],
  "tasks": [
    {
      "id": "task-001",
      "title": "Research Competitors",
      "agent_id": "agent-001",
      "order": 1
    },
    {
      "id": "task-002",
      "title": "Analyze Data",
      "agent_id": "agent-002",
      "order": 2,
      "context_tasks": ["task-001"]
    },
    {
      "id": "task-003",
      "title": "Write Report",
      "agent_id": "agent-003",
      "order": 3,
      "context_tasks": ["task-002"]
    }
  ],
  "manager_agent_id": null,
  "memory": false,
  "cache": true,
  "can_trigger_other_crews": true,
  "trigger_chain_on": "completed",
  "is_active": true,
  "version": 1
}
```

### Run Crew
```
POST /crews/{id}/run
```

**Request:**
```json
{
  "inputs": {
    "topic": "AI market trends",
    "region": "global"
  }
}
```

**Response (202):**
```json
{
  "execution_id": "exec-001",
  "crew_id": "crew-001",
  "status": "queued",
  "started_at": "2026-04-17T10:00:00Z"
}
```

### Get Execution Result
```
GET /executions/{execution_id}
```

**Response (200):**
```json
{
  "id": "exec-001",
  "crew_id": "crew-001",
  "status": "completed",
  "input_data": {
    "topic": "AI market trends",
    "region": "global"
  },
  "output_data": {
    "research_results": "Top competitors: OpenAI, Anthropic, Google...",
    "summary": "The AI market is valued at $150B...",
    "recommendations": "Invest in LLM infrastructure..."
  },
  "started_at": "2026-04-17T10:00:00Z",
  "completed_at": "2026-04-17T10:05:00Z",
  "duration_seconds": 300,
  "token_usage": {
    "input": 5000,
    "output": 3000
  }
}
```

---

## Triggers API

### Create Trigger (External Event)
```
POST /triggers
```

**Request:**
```json
{
  "name": "New Lead Alert",
  "description": "Triggered when new lead is added",
  "type": "event",
  "event_source": "database",
  "event_type": "new_row",
  "filter_condition": {
    "table": "leads",
    "where": "status = 'new'"
  },
  "crew_id": "crew-001"
}
```

**Response (201):**
```json
{
  "id": "trigger-001",
  "name": "New Lead Alert",
  "description": "Triggered when new lead is added",
  "type": "event",
  "event_source": "database",
  "event_type": "new_row",
  "filter_condition": {
    "table": "leads",
    "where": "status = 'new'"
  },
  "crew_id": "crew-001",
  "enabled": true,
  "trigger_count": 0,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### Create Trigger (Crew Completion)
```
POST /triggers
```

**Request:**
```json
{
  "name": "After Research → Analysis",
  "description": "Trigger analysis after research completes",
  "type": "crew_completion",
  "event_source": "crew",
  "source_crew_id": "crew-001",
  "event_type": "completed",
  "crew_id": "crew-002",
  "output_mapping": {
    "research_data": "{{source.output.research_results}}",
    "summary": "{{source.output.summary}}"
  },
  "continue_on_failure": false
}
```

**Response (201):**
```json
{
  "id": "trigger-002",
  "name": "After Research → Analysis",
  "description": "Trigger analysis after research completes",
  "type": "crew_completion",
  "event_source": "crew",
  "source_crew_id": "crew-001",
  "event_type": "completed",
  "crew_id": "crew-002",
  "output_mapping": {
    "research_data": "{{source.output.research_results}}",
    "summary": "{{source.output.summary}}"
  },
  "continue_on_failure": false,
  "enabled": true,
  "trigger_count": 0,
  "created_at": "2026-04-17T10:00:00Z"
}
```

### List Triggers
```
GET /triggers
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "trigger-001",
      "name": "New Lead Alert",
      "type": "event",
      "event_source": "database",
      "crew_id": "crew-001",
      "enabled": true,
      "trigger_count": 5
    },
    {
      "id": "trigger-002",
      "name": "After Research → Analysis",
      "type": "crew_completion",
      "event_source": "crew",
      "source_crew_id": "crew-001",
      "crew_id": "crew-002",
      "enabled": true,
      "trigger_count": 3
    }
  ],
  "total": 2
}
```

### Enable/Disable Trigger
```
POST /triggers/{id}/enable
```
```
POST /triggers/{id}/disable
```

**Response (200):**
```json
{
  "id": "trigger-001",
  "enabled": true
}
```

---

## Executions API

### List Executions
```
GET /executions
```

**Query Parameters:**
- `crew_id` - Filter by crew
- `status` - Filter by status (queued/running/completed/failed)
- `limit` - Number of results
- `offset` - Pagination offset

**Response (200):**
```json
{
  "data": [
    {
      "id": "exec-001",
      "crew_id": "crew-001",
      "crew_name": "Market Analysis Team",
      "status": "completed",
      "started_at": "2026-04-17T10:00:00Z",
      "completed_at": "2026-04-17T10:05:00Z",
      "duration_seconds": 300
    },
    {
      "id": "exec-002",
      "crew_id": "crew-002",
      "crew_name": "Content Pipeline",
      "status": "running",
      "started_at": "2026-04-17T11:00:00Z",
      "duration_seconds": 60
    }
  ],
  "total": 2
}
```

### Get Execution Details
```
GET /executions/{id}
```

**Response (200):**
```json
{
  "id": "exec-001",
  "crew_id": "crew-001",
  "crew_name": "Market Analysis Team",
  "trigger_id": "trigger-002",
  "status": "completed",
  "input_data": {
    "topic": "AI market trends"
  },
  "output_data": {
    "research_results": "...",
    "summary": "...",
    "recommendations": "..."
  },
  "error_message": null,
  "started_at": "2026-04-17T10:00:00Z",
  "completed_at": "2026-04-17T10:05:00Z",
  "duration_seconds": 300,
  "token_usage": {
    "input": 5000,
    "output": 3000,
    "total": 8000
  },
  "task_results": [
    {
      "task_id": "task-001",
      "task_title": "Research Competitors",
      "status": "completed",
      "output": "...",
      "duration_seconds": 120
    },
    {
      "task_id": "task-002",
      "task_title": "Analyze Data",
      "status": "completed",
      "output": "...",
      "duration_seconds": 100
    }
  ]
}
```

---

## Webhook API

### External Webhook
```
POST /webhooks/{trigger_name}
```

**Request:**
```json
{
  "event": "payment_succeeded",
  "data": {
    "amount": 5000,
    "currency": "USD",
    "customer_id": "cus-123"
  }
}
```

**Response (202):**
```json
{
  "success": true,
  "trigger_id": "trigger-001",
  "execution_id": "exec-003"
}
```

---

# Part 2: Mock Data (JSON)

## Full Mock Data for Frontend

```json
{
  "tools": [
    {
      "id": "tool-001",
      "name": "web_search",
      "description": "Search the web for information",
      "type": "decorator",
      "is_active": true,
      "linked_agents": ["agent-001", "agent-003"]
    },
    {
      "id": "tool-002",
      "name": "pdf_reader",
      "description": "Read and extract text from PDF",
      "type": "basetool",
      "is_active": true,
      "linked_agents": ["agent-002"]
    },
    {
      "id": "tool-003",
      "name": "calculator",
      "description": "Perform mathematical calculations",
      "type": "decorator",
      "is_active": true,
      "linked_agents": ["agent-002", "agent-004"]
    },
    {
      "id": "tool-004",
      "name": "email_sender",
      "description": "Send emails via SMTP",
      "type": "basetool",
      "is_active": true,
      "linked_agents": ["agent-003"]
    }
  ],
  "agents": [
    {
      "id": "agent-001",
      "name": "Market Researcher",
      "role": "Market Research Analyst",
      "goal": "Research and analyze market trends",
      "backstory": "Expert in competitive analysis and market research",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": [
        {"id": "tool-001", "name": "web_search"},
        {"id": "tool-002", "name": "pdf_reader"}
      ],
      "verbose": true
    },
    {
      "id": "agent-002",
      "name": "Data Analyst",
      "role": "Data Science Expert",
      "goal": "Analyze data and find insights",
      "backstory": "PhD in Statistics with 10 years experience",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": [
        {"id": "tool-002", "name": "pdf_reader"},
        {"id": "tool-003", "name": "calculator"}
      ],
      "verbose": true
    },
    {
      "id": "agent-003",
      "name": "Content Writer",
      "role": "Technical Writer",
      "goal": "Create compelling content",
      "backstory": "Former journalist, expert in tech writing",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": [
        {"id": "tool-001", "name": "web_search"},
        {"id": "tool-004", "name": "email_sender"}
      ],
      "verbose": true
    },
    {
      "id": "agent-004",
      "name": "Quality Assurance",
      "role": "QA Engineer",
      "goal": "Ensure quality and accuracy",
      "backstory": "Expert in quality assurance and editing",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "tools": [
        {"id": "tool-003", "name": "calculator"}
      ],
      "verbose": false
    }
  ],
  "tasks": [
    {
      "id": "task-001",
      "title": "Research Competitors",
      "description": "Research top 5 competitors in the AI market",
      "expected_output": "List with market share, pricing, features",
      "agent_id": "agent-001",
      "order": 1
    },
    {
      "id": "task-002",
      "title": "Analyze Data",
      "description": "Analyze competitor data for trends",
      "expected_output": "Charts and insights",
      "agent_id": "agent-002",
      "order": 2,
      "context_tasks": ["task-001"]
    },
    {
      "id": "task-003",
      "title": "Write Report",
      "description": "Write comprehensive market report",
      "expected_output": "PDF report with executive summary",
      "agent_id": "agent-003",
      "order": 3,
      "context_tasks": ["task-002"]
    },
    {
      "id": "task-004",
      "title": "Review Quality",
      "description": "Review report for accuracy",
      "expected_output": "Reviewed report with notes",
      "agent_id": "agent-004",
      "order": 4,
      "context_tasks": ["task-003"]
    }
  ],
  "crews": [
    {
      "id": "crew-001",
      "name": "Market Analysis Team",
      "description": "Complete market analysis workflow",
      "process": "sequential",
      "agents": [
        {"id": "agent-001", "name": "Market Researcher"},
        {"id": "agent-002", "name": "Data Analyst"},
        {"id": "agent-003", "name": "Content Writer"},
        {"id": "agent-004", "name": "Quality Assurance"}
      ],
      "tasks": [
        {"id": "task-001", "title": "Research Competitors", "order": 1},
        {"id": "task-002", "title": "Analyze Data", "order": 2},
        {"id": "task-003", "title": "Write Report", "order": 3},
        {"id": "task-004", "title": "Review Quality", "order": 4}
      ],
      "can_trigger_other_crews": true,
      "trigger_chain_on": "completed",
      "is_active": true
    },
    {
      "id": "crew-002",
      "name": "Quick Analysis Team",
      "description": "Fast analysis with hierarchical control",
      "process": "hierarchical",
      "manager_agent_id": "agent-001",
      "agents": [
        {"id": "agent-001", "name": "Market Researcher"},
        {"id": "agent-002", "name": "Data Analyst"}
      ],
      "tasks": [
        {"id": "task-001"},
        {"id": "task-002"}
      ],
      "can_trigger_other_crews": false,
      "is_active": true
    }
  ],
  "triggers": [
    {
      "id": "trigger-001",
      "name": "New Lead",
      "description": "Trigger on new database lead",
      "type": "event",
      "event_source": "database",
      "crew_id": "crew-001",
      "enabled": true,
      "trigger_count": 12
    },
    {
      "id": "trigger-002",
      "name": "After Market Analysis",
      "description": "Run content pipeline after analysis",
      "type": "crew_completion",
      "event_source": "crew",
      "source_crew_id": "crew-001",
      "event_type": "completed",
      "crew_id": "crew-002",
      "enabled": true,
      "trigger_count": 8
    },
    {
      "id": "trigger-003",
      "name": "Daily Report",
      "description": "Generate daily report",
      "type": "schedule",
      "event_source": "cron",
      "cron_expression": "0 9 * * *",
      "crew_id": "crew-001",
      "enabled": true,
      "trigger_count": 30
    }
  ],
  "executions": [
    {
      "id": "exec-001",
      "crew_id": "crew-001",
      "crew_name": "Market Analysis Team",
      "status": "completed",
      "started_at": "2026-04-17T09:00:00Z",
      "completed_at": "2026-04-17T09:15:00Z",
      "duration_seconds": 900
    },
    {
      "id": "exec-002",
      "crew_id": "crew-001",
      "crew_name": "Market Analysis Team",
      "status": "running",
      "started_at": "2026-04-17T10:00:00Z",
      "duration_seconds": 120
    },
    {
      "id": "exec-003",
      "crew_id": "crew-001",
      "crew_name": "Market Analysis Team",
      "status": "failed",
      "started_at": "2026-04-17T08:00:00Z",
      "completed_at": "2026-04-17T08:05:00Z",
      "error_message": "API rate limit exceeded"
    }
  ]
}
```

---

# Part 3: Test UI (HTML Demo)

 `test-ui.html`  open in browser.

---

# Summary

This implementation package contains:

| Part | Contents |
|------|----------|
| **API Design** | Full REST API contracts with request/response examples |
| **Mock Data** | Complete JSON data for all entities (tools, agents, tasks, crews, triggers, executions) |
| **Test UI** | Interactive HTML demo showing canvas, nodes, drag-drop, and process toggle |

All three parts work together - the mock data can be used directly in the test UI for development.