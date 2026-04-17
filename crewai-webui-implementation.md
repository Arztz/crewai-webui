# CrewAI WebUI - Implementation Package

> Version: 1.1 | Created: April 2026 | Status: Implementation Sync

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
  "type": "decorator"
}
```

**Response (201):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "agents": [],
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
      "agents": ["Research", "Write"]
    },
    {
      "id": "tool-002",
      "name": "pdf_reader",
      "description": "Read PDF files",
      "type": "basetool",
      "agents": ["Analyze"]
    },
    {
      "id": "tool-003",
      "name": "calculator",
      "description": "Math calculations",
      "type": "decorator",
      "agents": []
    }
  ],
  "total": 3
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
  "agents": ["Research", "Write"],
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
  "type": "decorator"
}
```

**Response (200):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "description": "Search the web for information",
  "type": "decorator",
  "agents": ["Research", "Write"],
  "updated_at": "2026-04-17T11:00:00Z"
}
```

### Link Tool to Agent
```
POST /tools/{tool_id}/agents
```

**Request:**
```json
{
  "agent_name": "Research"
}
```

**Response (200):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "agents": ["Research", "Write"]
}
```

### Unlink Tool from Agent
```
DELETE /tools/{tool_id}/agents/{agent_name}
```

**Response (200):**
```json
{
  "id": "tool-001",
  "name": "web_search",
  "agents": ["Write"]
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
  "name": "Research",
  "role": "Research Analyst",
  "goal": "Research market trends",
  "backstory": "Expert researcher"
}
```

**Response (201):**
```json
{
  "id": "agent-001",
  "name": "Research",
  "role": "Research Analyst",
  "goal": "Research market trends",
  "backstory": "Expert researcher",
  "tools": [],
  "verbose": false,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
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
      "name": "Research",
      "role": "Research Analyst",
      "goal": "Research market trends",
      "tools": ["web_search", "pdf_reader"]
    },
    {
      "id": "agent-002",
      "name": "Analyze",
      "role": "Data Analyst",
      "goal": "Analyze data",
      "tools": ["calculator"]
    },
    {
      "id": "agent-003",
      "name": "Write",
      "role": "Writer",
      "goal": "Write content",
      "tools": ["web_search"]
    },
    {
      "id": "agent-004",
      "name": "QA",
      "role": "QA Engineer",
      "goal": "Ensure quality",
      "tools": []
    }
  ],
  "total": 4
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
  "name": "Research",
  "role": "Research Analyst",
  "goal": "Research market trends",
  "backstory": "Expert researcher",
  "tools": [
    {"id": "tool-001", "name": "web_search"},
    {"id": "tool-002", "name": "pdf_reader"}
  ],
  "verbose": false,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
}
```

### Update Agent
```
PUT /agents/{id}
```

**Request:**
```json
{
  "name": "Research",
  "role": "Senior Research Analyst",
  "goal": "Research market trends and provide insights",
  "backstory": "Expert researcher with 10+ years experience"
}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "name": "Research",
  "role": "Senior Research Analyst",
  "goal": "Research market trends and provide insights",
  "backstory": "Expert researcher with 10+ years experience",
  "tools": ["web_search", "pdf_reader"],
  "updated_at": "2026-04-17T11:00:00Z"
}
```

### Add Tool to Agent
```
POST /agents/{agent_id}/tools
```

**Request:**
```json
{
  "tool_name": "calculator"
}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "name": "Research",
  "tools": ["web_search", "pdf_reader", "calculator"]
}
```

### Remove Tool from Agent
```
DELETE /agents/{agent_id}/tools/{tool_name}
```

**Response (200):**
```json
{
  "id": "agent-001",
  "name": "Research",
  "tools": ["web_search", "pdf_reader"]
}
```

### Delete Agent
```
DELETE /agents/{id}
```

**Response (204):** No content

---

## Tasks API

### Create Task
```
POST /tasks
```

**Request:**
```json
{
  "title": "Research",
  "description": "Research competitors"
}
```

**Response (201):**
```json
{
  "id": "task-001",
  "title": "Research",
  "description": "Research competitors",
  "agent": null,
  "context": [],
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
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
      "title": "Research",
      "description": "Research competitors",
      "agent": "Research",
      "context": []
    },
    {
      "id": "task-002",
      "title": "Analyze",
      "description": "Analyze data",
      "agent": "Analyze",
      "context": ["Research"]
    },
    {
      "id": "task-003",
      "title": "Write",
      "description": "Write report",
      "agent": "Write",
      "context": ["Analyze"]
    },
    {
      "id": "task-004",
      "title": "Review",
      "description": "Review work",
      "agent": null,
      "context": ["Write"]
    }
  ],
  "total": 4
}
```

### Get Task
```
GET /tasks/{id}
```

**Response (200):**
```json
{
  "id": "task-001",
  "title": "Research",
  "description": "Research competitors",
  "agent": {
    "id": "agent-001",
    "name": "Research"
  },
  "context": [],
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
}
```

### Update Task
```
PUT /tasks/{id}
```

**Request:**
```json
{
  "title": "Market Research",
  "description": "Research market trends and competitors"
}
```

**Response (200):**
```json
{
  "id": "task-001",
  "title": "Market Research",
  "description": "Research market trends and competitors",
  "agent": "Research",
  "context": [],
  "updated_at": "2026-04-17T11:00:00Z"
}
```

### Assign Agent to Task
```
POST /tasks/{task_id}/agent
```

**Request:**
```json
{
  "agent_name": "Research"
}
```

**Response (200):**
```json
{
  "id": "task-001",
  "agent": "Research"
}
```

### Unassign Agent from Task
```
DELETE /tasks/{task_id}/agent
```

**Response (200):**
```json
{
  "id": "task-001",
  "agent": null
}
```

### Add Context Task (Dependency)
```
POST /tasks/{task_id}/context
```

**Request:**
```json
{
  "context_task": "Research"
}
```

**Response (200):**
```json
{
  "id": "task-002",
  "context": ["Research"]
}
```

### Remove Context Task
```
DELETE /tasks/{task_id}/context/{context_task_name}
```

**Response (200):**
```json
{
  "id": "task-002",
  "context": []
}
```

### Delete Task
```
DELETE /tasks/{id}
```

**Response (204):** No content

---

## Crews API

### Create Crew
```
POST /crews
```

**Request:**
```json
{
  "name": "Market Team",
  "process": "sequential"
}
```

**Response (201):**
```json
{
  "id": "crew-001",
  "name": "Market Team",
  "process": "sequential",
  "agents": [],
  "tasks": [],
  "x": 50,
  "y": 50,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
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
      "name": "Market Team",
      "process": "sequential",
      "agents": ["Research", "Analyze", "Write", "QA"],
      "tasks": ["Research", "Analyze", "Write", "Review"]
    },
    {
      "id": "crew-002",
      "name": "Quick Team",
      "process": "hierarchical",
      "agents": ["Research", "Analyze"],
      "tasks": ["Research", "Analyze"]
    }
  ],
  "total": 2
}
```

### Get Crew
```
GET /crews/{id}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "name": "Market Team",
  "process": "sequential",
  "agents": [
    {"id": "agent-001", "name": "Research"},
    {"id": "agent-002", "name": "Analyze"},
    {"id": "agent-003", "name": "Write"},
    {"id": "agent-004", "name": "QA"}
  ],
  "tasks": [
    {"id": "task-001", "title": "Research"},
    {"id": "task-002", "title": "Analyze"},
    {"id": "task-003", "title": "Write"},
    {"id": "task-004", "title": "Review"}
  ],
  "x": 50,
  "y": 50,
  "created_at": "2026-04-17T10:00:00Z",
  "updated_at": "2026-04-17T10:00:00Z"
}
```

### Update Crew
```
PUT /crews/{id}
```

**Request:**
```json
{
  "name": "Market Analysis Team",
  "process": "hierarchical"
}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "name": "Market Analysis Team",
  "process": "hierarchical",
  "agents": ["Research", "Analyze", "Write", "QA"],
  "tasks": ["Research", "Analyze", "Write", "Review"],
  "x": 50,
  "y": 50,
  "updated_at": "2026-04-17T11:00:00Z"
}
```

### Update Crew Position (Canvas)
```
PATCH /crews/{id}/position
```

**Request:**
```json
{
  "x": 200,
  "y": 150
}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "x": 200,
  "y": 150
}
```

### Add Agent to Crew
```
POST /crews/{crew_id}/agents
```

**Request:**
```json
{
  "agent_name": "Write"
}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "agents": ["Research", "Analyze", "Write", "QA"]
}
```

### Remove Agent from Crew
```
DELETE /crews/{crew_id}/agents/{agent_name}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "agents": ["Research", "Analyze", "QA"]
}
```

### Add Task to Crew
```
POST /crews/{crew_id}/tasks
```

**Request:**
```json
{
  "task_title": "Review"
}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "tasks": ["Research", "Analyze", "Write", "Review"]
}
```

### Remove Task from Crew
```
DELETE /crews/{crew_id}/tasks/{task_title}
```

**Response (200):**
```json
{
  "id": "crew-001",
  "tasks": ["Research", "Analyze", "Write"]
}
```

### Run Crew
```
POST /crews/{id}/run
```

**Request (optional):**
```json
{
  "inputs": {
    "topic": "AI market trends"
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

### Delete Crew
```
DELETE /crews/{id}
```

**Response (204):** No content

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

### Get Trigger
```
GET /triggers/{id}
```

**Response (200):**
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
  "trigger_count": 3,
  "last_triggered_at": "2026-04-17T09:30:00Z",
  "created_at": "2026-04-17T08:00:00Z"
}
```

### Update Trigger
```
PUT /triggers/{id}
```

**Request:**
```json
{
  "name": "After Market Analysis",
  "enabled": true
}
```

**Response (200):**
```json
{
  "id": "trigger-002",
  "name": "After Market Analysis",
  "enabled": true,
  "updated_at": "2026-04-17T11:00:00Z"
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

### Delete Trigger
```
DELETE /triggers/{id}
```

**Response (204):** No content

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
      "crew_name": "Market Team",
      "status": "completed",
      "started_at": "2026-04-17T09:00:00Z",
      "completed_at": "2026-04-17T09:15:00Z",
      "duration_seconds": 900
    },
    {
      "id": "exec-002",
      "crew_id": "crew-001",
      "crew_name": "Market Team",
      "status": "running",
      "started_at": "2026-04-17T10:00:00Z",
      "duration_seconds": 120
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
  "crew_name": "Market Team",
  "trigger_id": "trigger-002",
  "status": "completed",
  "input_data": {
    "topic": "AI market trends"
  },
  "output_data": {
    "research_results": "Top competitors: OpenAI, Anthropic, Google...",
    "summary": "The AI market is valued at $150B...",
    "recommendations": "Invest in LLM infrastructure..."
  },
  "error_message": null,
  "started_at": "2026-04-17T09:00:00Z",
  "completed_at": "2026-04-17T09:15:00Z",
  "duration_seconds": 900,
  "token_usage": {
    "input": 5000,
    "output": 3000,
    "total": 8000
  },
  "task_results": [
    {
      "task_id": "task-001",
      "task_title": "Research",
      "agent_name": "Research",
      "status": "completed",
      "output": "Competitor analysis complete...",
      "duration_seconds": 300
    },
    {
      "task_id": "task-002",
      "task_title": "Analyze",
      "agent_name": "Analyze",
      "status": "completed",
      "output": "Data analysis complete...",
      "duration_seconds": 300
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

# Part 2: Mock Data (Frontend Store)

## Full Mock Data for Frontend

```javascript
const store = {
    tools: {
        'web_search': {
            name: 'web_search',
            description: 'Search the web',
            type: 'decorator',
            agents: ['Research', 'Write']
        },
        'pdf_reader': {
            name: 'pdf_reader',
            description: 'Read PDF files',
            type: 'basetool',
            agents: ['Research', 'Analyze']
        },
        'calculator': {
            name: 'calculator',
            description: 'Math calculations',
            type: 'decorator',
            agents: ['Analyze']
        }
    },
    agents: {
        'Research': {
            name: 'Research',
            role: 'Research Analyst',
            goal: 'Research market trends',
            backstory: 'Expert researcher',
            tools: ['web_search', 'pdf_reader']
        },
        'Analyze': {
            name: 'Analyze',
            role: 'Data Analyst',
            goal: 'Analyze data',
            backstory: 'Data expert',
            tools: ['pdf_reader', 'calculator']
        },
        'Write': {
            name: 'Write',
            role: 'Writer',
            goal: 'Write content',
            backstory: 'Professional writer',
            tools: ['web_search']
        },
        'QA': {
            name: 'QA',
            role: 'QA Engineer',
            goal: 'Ensure quality',
            backstory: 'Quality expert',
            tools: []
        }
    },
    tasks: {
        'Research': {
            title: 'Research',
            description: 'Research competitors',
            agent: 'Research',
            context: []
        },
        'Analyze': {
            title: 'Analyze',
            description: 'Analyze data',
            agent: 'Analyze',
            context: ['Research']
        },
        'Write': {
            title: 'Write',
            description: 'Write report',
            agent: 'Write',
            context: ['Analyze']
        },
        'Review': {
            title: 'Review',
            description: 'Review work',
            agent: null,
            context: ['Write']
        }
    },
    crews: {
        'Market Team': {
            name: 'Market Team',
            process: 'sequential',
            agents: ['Research', 'Analyze', 'Write', 'QA'],
            tasks: ['Research', 'Analyze', 'Write', 'Review'],
            x: 50,
            y: 50
        },
        'Quick Team': {
            name: 'Quick Team',
            process: 'hierarchical',
            agents: ['Research', 'Analyze'],
            tasks: ['Research', 'Analyze'],
            x: 50,
            y: 320
        }
    },
    canvas: {
        offsetX: 0,
        offsetY: 0,
        scale: 1
    }
};
```

---

# Part 3: Implementation Notes

## Drag & Drop Implementation

### Drag Sources
- Sidebar items: `draggable="true"`, `data-type`, `data-name`
- Events: `dragstart`, `dragend`

### Drop Targets
- **Crew node**: `dragover`, `drop` handlers
- **Agent badge**: `dragover` to highlight, drop to link tool

### Drag State Management
```javascript
let draggedData = null;

function handleDragStart(e) {
    const type = e.target.dataset.type;
    const name = e.target.dataset.name;
    draggedData = { type, name, targetAgent: null };
}

// Clean up on dragend (anywhere)
document.addEventListener('dragend', (e) => {
    draggedData = null;
    document.querySelectorAll('.crew-badge.agent').forEach(badge => {
        badge.style.background = '';
    });
});
```

### Tool → Agent Linking
```javascript
node.addEventListener('drop', (e) => {
    const agentBadge = e.target.closest('.crew-badge.agent');

    if (draggedData.type === 'tool' && agentBadge) {
        const agentName = extractAgentName(agentBadge);
        if (!store.agents[agentName].tools.includes(draggedData.name)) {
            store.agents[agentName].tools.push(draggedData.name);
            store.tools[draggedData.name].agents.push(agentName);
        }
    }
});
```

## Canvas Panning

```javascript
const canvas = document.getElementById('canvas');
const canvasInner = document.getElementById('canvasInner');
let isPanning = false;
let panStart = { x: 0, y: 0 };

canvas.addEventListener('mousedown', (e) => {
    if (e.target === canvas || e.target === canvasInner) {
        isPanning = true;
        panStart = { x: e.clientX - store.canvas.offsetX, y: e.clientY - store.canvas.offsetY };
    }
});

document.addEventListener('mousemove', (e) => {
    if (isPanning) {
        store.canvas.offsetX = e.clientX - panStart.x;
        store.canvas.offsetY = e.clientY - panStart.y;
        canvasInner.style.transform = `translate(${store.canvas.offsetX}px, ${store.canvas.offsetY}px)`;
    }
});

document.addEventListener('mouseup', () => {
    isPanning = false;
});
```

## Crew Node Dragging (Header Only)

```javascript
let draggedCrew = null;
let crewOffset = { x: 0, y: 0 };

document.addEventListener('mousedown', (e) => {
    const crewHeader = e.target.closest('.crew-header');
    if (crewHeader) {
        const crewNode = crewHeader.closest('.crew-node');
        draggedCrew = crewNode.dataset.crew;
        const rect = crewNode.getBoundingClientRect();
        crewOffset = {
            x: e.clientX - rect.left,
            y: e.clientY - rect.top
        };
    }
});

document.addEventListener('mousemove', (e) => {
    if (draggedCrew) {
        const rect = canvas.getBoundingClientRect();
        const x = e.clientX - rect.left - store.canvas.offsetX - crewOffset.x;
        const y = e.clientY - rect.top - store.canvas.offsetY - crewOffset.y;
        store.crews[draggedCrew].x = Math.max(0, x);
        store.crews[draggedCrew].y = Math.max(0, y);
        renderCanvas();
    }
});

document.addEventListener('mouseup', () => {
    draggedCrew = null;
});
```

## Modal Event Delegation

```javascript
document.getElementById('modalBody').addEventListener('click', (e) => {
    const removeToolBtn = e.target.closest('[data-action="remove-tool"]');
    if (removeToolBtn) {
        const agentName = removeToolBtn.dataset.agent;
        const toolName = removeToolBtn.dataset.tool;
        unlinkAgentTool(agentName, toolName);
        openEditModal(currentModal.type, currentModal.entity);
        return;
    }
    // ... handle other actions
});
```

---

# Part 4: Test UI Reference

`test-ui.html` - Full implementation (~1643 lines)

**Features implemented:**
- Sidebar with collapsible sections, search, entity counts
- Canvas with grid background and panning
- Crew nodes with header-only dragging
- Drag-drop from sidebar to crew
- Tool→Agent badge linking
- Create/Edit modals for all entities
- Link modals for adding connections
- Agent badges showing name + linked tools
- Task badges numbered by position
- Process mode (Sequential/Hierarchical) per crew
- Help panel overlay
- Keyboard shortcuts (Escape to close)

**File location:** `/home/arztz/Projects/CrewAI/test-ui.html`

---

# Summary

This implementation package contains:

| Part | Contents |
|------|----------|
| **API Design** | Full REST API contracts with request/response examples |
| **Mock Data** | Frontend store structure matching test-ui.html |
| **Implementation Notes** | Key code patterns for drag-drop, panning, modals |
| **Test UI** | Full HTML implementation (~1643 lines) |

All parts work together - the mock data structure matches the test-ui.html store, and the API design supports all frontend operations.

*Document Version: 1.1 - Updated to reflect actual test-ui.html implementation*
