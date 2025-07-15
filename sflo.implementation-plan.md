---
id: fhzhdkejlgo8sfbbaii8bmh
title: Implementation Plan
desc: ''
updated: 1752350503179
created: 1752071938622
---

## **Flow Stack Implementation Plan**

### **Project Overview**
We're building the [[flow-service|sflo.product.service]], a REST-based backend for creating, manipulating, and using semantic meshes, and a Deno-based CLI using Cliffy that consumes it.

Meshes are derefenceable, versioned collections of semantic data where every URL returns meaningful content. The CLI will trigger the "weave process" that maintains mesh integrity, versioning, and generates resource pages.

## **Phase 1: Minimal Flow Stack Setup**

### **Directory Structure**
```
semantic-flow/
├── flow-service/
│   ├── deno.json
│   ├── src/
│   │   ├── main.ts
│   │   └── routes/
│   │       └── health.ts
└── flow-cli/
    ├── deno.json
    └── src/
        └── main.ts
```

### **1. Flow Service (Hono API with Heartbeat)**

#### **flow-service/deno.json**
```json
{
  "imports": {
    "@hono/hono": "jsr:@hono/hono@^4.6.0",
    "@hono/zod-openapi": "jsr:@hono/zod-openapi@^0.16.0",
    "@scalar/hono-api-reference": "npm:@scalar/hono-api-reference@^0.5.0",
    "zod": "npm:zod@^3.22.0"
  },
  "tasks": {
    "dev": "deno run --allow-net --allow-read --allow-write src/main.ts",
    "start": "deno run --allow-net --allow-read --allow-write src/main.ts"
  }
}
```

#### **flow-service/src/main.ts**
```typescript
import { OpenAPIHono } from '@hono/zod-openapi'
import { apiReference } from '@scalar/hono-api-reference'
import { serve } from '@hono/hono/deno'
import { logger } from '@hono/hono/logger'
import { cors } from '@hono/hono/cors'
import { healthApp } from './routes/health.ts'

const app = new OpenAPIHono()

// Middleware
app.use('*', logger())
app.use('*', cors())

// OpenAPI doc
app.doc('/openapi.json', {
  openapi: '3.0.0',
  info: {
    version: '1.0.0',
    title: 'Flow Service API',
    description: 'Basic API for Flow mesh management'
  }
})

// Scalar docs
app.get('/docs', apiReference({
  spec: { url: '/openapi.json' }
}))

// Routes
app.route('/api', healthApp)

console.log('Flow Service starting on http://localhost:8000')
console.log('API docs available at http://localhost:8000/docs')

serve(app, { port: 8000 })
```

#### **flow-service/src/routes/health.ts**
```typescript
import { OpenAPIHono } from '@hono/zod-openapi'
import { z } from 'zod'

const healthApp = new OpenAPIHono()

const HealthResponse = z.object({
  status: z.string(),
  timestamp: z.string(),
  service: z.string(),
  version: z.string()
})

healthApp.openapi({
  method: 'get',
  path: '/health',
  tags: ['System'],
  summary: 'Service heartbeat',
  responses: {
    200: {
      description: 'Service health status',
      content: {
        'application/json': {
          schema: HealthResponse
        }
      }
    }
  }
}, (c) => {
  return c.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    service: 'flow-service',
    version: '0.1.0'
  })
})

export { healthApp }
```

### **2. Flow CLI (Cliffy-based)**

#### **flow-cli/deno.json**
```json
{
  "imports": {
    "@cliffy/command": "jsr:@cliffy/command@^1.0.0-rc.8",
    "@std/http": "jsr:@std/http@^1.0.0"
  },
  "tasks": {
    "dev": "deno run --allow-net src/main.ts",
    "install": "deno install --allow-net -n flow src/main.ts"
  }
}
```

#### **flow-cli/src/main.ts**
```typescript
import { Command } from '@cliffy/command'

// Service client
class FlowServiceClient {
  constructor(private baseUrl = 'http://localhost:8000') {}

  async health(): Promise<any> {
    const response = await fetch(`${this.baseUrl}/api/health`)
    if (!response.ok) {
      throw new Error(`Health check failed: ${response.status}`)
    }
    return await response.json()
  }
}

const client = new FlowServiceClient()

await new Command()
  .name('flow')
  .version('1.0.0')
  .description('Flow CLI for semantic mesh management')
  
  .command('health', 'Check service health')
  .action(async () => {
    try {
      const health = await client.health()
      console.log('✅ Service Status:', health.status)
      console.log('📍 Service:', health.service)
      console.log('🕐 Timestamp:', health.timestamp)
      console.log('📦 Version:', health.version)
    } catch (error) {
      console.error('❌ Health check failed:', error.message)
      Deno.exit(1)
    }
  })

  .command('service')
  .description('Service management commands')
  .command('start', 'Start the flow service')
  .action(() => {
    console.log('🚀 To start the service, run:')
    console.log('   cd flow-service && deno task dev')
  })

  .parse(Deno.args)
```

### **3. Quick Start Commands**

#### **Start the Service**
```bash
cd semantic-flow/flow-service
deno task dev
```

#### **Test the CLI**
```bash
cd semantic-flow/flow-cli
deno task dev health
```

#### **Install CLI Globally**
```bash
cd semantic-flow/flow-cli
deno task install
flow health
```

### **4. Development Workflow**

1. **Terminal 1**: Start the service
   ```bash
   cd flow-service && deno task dev
   ```

2. **Terminal 2**: Test with CLI
   ```bash
   cd flow-cli && deno task dev health
   ```

3. **Browser**: Check API docs
   - Visit `http://localhost:8000/docs` for Scalar interface
   - Visit `http://localhost:8000/api/health` for direct endpoint

### **5. Features Included**

- ✅ **Hono API** with OpenAPI and Scalar documentation
- ✅ **Heartbeat endpoint** for service health checking
- ✅ **CLI app** that can communicate with the service
- ✅ **Modern DevX** with Scalar instead of Swagger UI
- ✅ **Type safety** with Zod schemas
- ✅ **CORS and logging** middleware
- ✅ **Global CLI installation** support

## **Future Phases (Not Implemented Yet)**

### **Phase 2: Basic Mesh Management**
- Add mesh scanning endpoints
- Implement basic configuration loading
- Add mesh listing and info endpoints

### **Phase 3: RDF Storage Integration**
- Add Quadstore with MemoryLevel backend
- Implement native SPARQL endpoint (via Quadstore)
- Add RDF data loading and querying

### **Phase 4: Weave Process**
- Implement component versioning
- Add weave trigger endpoints
- Generate resource pages and HTMX fragments

### **Phase 5: Advanced Features**
- Configuration inheritance system
- HTMX integration for web UI
- Comunica widget for SPARQL queries

## **Architecture Notes**

### **Technology Stack**
- **API Framework**: Hono with zod-openapi for type safety
- **Documentation**: Scalar (modern alternative to Swagger UI)
- **CLI Framework**: Cliffy for command-line interface
- **Runtime**: Deno for both service and CLI
- **Future RDF**: Quadstore + MemoryLevel for SPARQL support

### **Key Design Decisions**
- **Service-first architecture**: CLI consumes REST API
- **OpenAPI specification**: Full API documentation and validation
- **Native SPARQL**: Quadstore provides built-in SPARQL endpoint
- **Modern DevX**: Scalar provides superior API testing interface
- **Type safety**: End-to-end TypeScript with Zod schemas

