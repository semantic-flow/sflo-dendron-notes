---
id: 960nifinmgi8yulj3nndffl
title: Architecture
desc: ''
updated: 1752347925054
created: 1752347754851
---
# Flow CLI & Service Implementation Discussion Summary

## Project Context
Building a **Flow CLI** in Deno using Cliffy to manage semantic meshes - versioned collections of semantic data where every URL returns meaningful content. The CLI will consume a **flow-service** API to handle weave processes, configuration management, and mesh operations.

## Key Decisions Made

### **1. Architecture Approach**
- **Service-first architecture**: CLI becomes client of flow-service API
- **Multi-mesh support**: Service can handle multiple meshes across filesystem
- **REST API**: Better alignment with hierarchical mesh structure and future web UI needs
- **Deno-based implementation**: Both CLI and service in Deno for consistency

### **2. API Design**
- **REST endpoints** mapping to mesh hierarchy: `/mesh/{meshPath}/config`, `/mesh/{meshPath}/weave`
- **OpenAPI specification** with Zod validation using Hono framework
- **SPARQL endpoint** for advanced mesh querying
- **HTMX fragments** for dynamic web interfaces

### **3. Configuration System**
- **`.flow-config.jsonld`** files distributed throughout mesh nodes (as mesh elements)
- **Recursive config inheritance** up the node hierarchy
- **JSON-LD format** with semantic context from flow-ontology
- **Zod schemas** for runtime validation bridging to RDF validation

### **4. Versioning Strategy**
- **Assets versioning**: `_assets` should live within component layers (`_current`, `_next`, `_v1`, etc.)
- **Unified datasets**: Node-level aggregation of all component versions at a point in time
- **System components**: Both `_meta-component` and `_unified-dataset` are system-generated with no `_next` layer
- **Atomic weave operations**: System components version together to avoid circular dependencies

### **5. Resource Pages & UI**
- **Root index.html as router**: Can coordinate version viewing across components
- **HTMX integration** for rich interactions with static backend compatibility
- **Multiple resource fragments** possible per node, stored in `_assets/_fragments/`
- **Fragment-based architecture**: Generate HTML fragments during weave for HTMX navigation

## Technology Stack

### **Core Stack**
```typescript
// deno.json imports
{
  "hono": "jsr:@hono/hono@^4.6.0",
  "@hono/zod-openapi": "jsr:@hono/zod-openapi@^0.16.0",
  "quadstore": "npm:quadstore@^15.1.0",
  "memory-level": "npm:memory-level@^1.0.0", 
  "n3 data factory": ,
  "jsonld-streaming-parser": ,
  "@cliffy/command": "jsr:@cliffy/command@^1.0.0-rc.8"
}
```

### **RDF/Storage Layer**
- **filesystem as source of truth**: the mesh data may be in-memory during service operation, but storage is on disk in plain RDF distributions
- **Quadstore with MemoryLevel backend**: Provides both in-memory RDF storage and native SPARQL support
- **n3.js**: DataFactory implementation and serialization for non-jsonld formats
- **JSON-LD processing**: "[JSON-LD Streaming parser](https://github.com/rubensworks/jsonld-streaming-parser.js)" and maybe https://github.com/rubensworks/jsonld-context-parser.js

### **API Layer**  
- **Hono framework**: Fast, TypeScript-first web framework with excellent Deno support
- **OpenAPI + Zod**: Type-safe API with auto-generated documentation 


### Contained Servies
- **Local webserver**: serves the mesh locally, either for preview before publishing or for local application use
- **SPARQL endpoint**: Quadstore's native SPARQL support
- **Comunica jQuery widget**: For web-based SPARQL query interface
- **Swagger UI**: for testing and direct mesh manipulation

### **CLI Layer**
- **Cliffy framework**: For command-line interface implementation
- **Commands**: `flow init`, `flow service`, `flow weave`, `flow validate`, `flow status`

## Architecture Patterns

### **Weave Process Flow**
- Triggered via CLI, web app, or via API
- **User component changes** (i.e., editing of data in `_next` layers) determines whether weave does anything
- **Versioning occurs** if policies require it
- **System components regenerate** atomically (`_meta-component` + `_unified-component`)
- Resource pages and **HTMX fragments generate** for updated nodes

### **Configuration Resolution**
1. **Local config** loaded from `.flow-config.jsonld`
2. **Parent configs** recursively inherited up mesh hierarchy  
3. **Resolved config** computed and cached in service
4. **RDF representation** stored with file:// URIs for querying

### **SPARQL Integration**
- **Quadstore** provides SPARQL endpoint functionality
- **Comunica widget** consumes `/sparql` endpoint for user interface
- **Mesh structure** queryable via SPARQL for advanced operations

## Next Implementation Steps

- **Set up basic Hono service** with Quadstore + MemoryLevel
- **Implement mesh scanning** 
  - RDF loading
  - config loading
- **Create SPARQL endpoint** and test with Comunica widget
- **Build CLI commands** consuming the service API
- **Add weave process** with component versioning and fragment generation
- **Implement config inheritance** and JSON-LD processing

## Key Technical Considerations

- **Deno compatibility**: Most RDF libraries work via npm: imports, but some complex dependencies may require workarounds
- **Performance**: In-memory storage suitable for development
- **Standards compliance**: Following RDF/JS interfaces ensures ecosystem compatibility
- **Semantic consistency**: All mesh operations maintain semantic web principles with proper IRIs and ontology alignment

## Open Questions for Next Phase

1. **Deployment strategy**: How will flow-service be distributed and run?
2. **Persistence layer**: When to add persistent storage beyond memory-level?
3. **Multi-user support**: Authentication and authorization for service?
4. **Performance optimization**: Caching strategies for large mesh operations?
5. **Web UI scope**: How extensive should the web interface become beyond SPARQL?