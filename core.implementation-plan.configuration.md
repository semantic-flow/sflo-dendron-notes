---
id: wjl3rmw6iyypzy13h4uci9q
title: Configuration
desc: ''
updated: 1752445355076
created: 1752434728498
---

## Overview

Implement a two-tier configuration system for Semantic Flow, adapting the proven weave config merge pattern with separate RDF ontologies and SHACL validation for service and node configurations.

## Architecture

### Two Configuration Types with Separate Ontologies

#### 1. Service Config
- **Ontology**: `service-ontology.ttl` - service-specific vocabulary
- **File**: `flow-service-config.jsonld` (optional, sparse)
- **Scope**: Service operation + node config defaults override
- **Content**:
  - Service ports, endpoints, logging settings
  - Node config platform default overrides (`svc:nodeDefaults`)

#### 2. Node Config
- **Ontology**: `node-ontology.ttl` - mesh operation vocabulary
- **File**: `_config-component` within each node (sparse)
- **Scope**: Node-specific operational settings
- **Content**:
  - Component versioning, templates, distribution formats
  - Only non-default/non-inherited settings

## Weave-Inspired Service Config Resolution

### Merge Priority (highest to lowest)
1. **Command-line options** (CLI args)
2. **Environment variables** (FLOW_*)
3. **Service configuration file** (sparse JSON-LD)
4. **Platform defaults** (in-memory, complete)

### Service Config Resolution Process
```typescript
// Weave-pattern merge for service config
async function resolveServiceConfig(cliOptions?: ServiceOptions): Promise<ServiceConfigContext> {
  // 1. Start with empty input options
  let inputOptions: Partial<ServiceConfig> = {};

  // 2. Merge environment variables (weave pattern)
  inputOptions = mergeConfigs(inputOptions, loadEnvConfig());

  // 3. Load and merge service config file (weave pattern)
  const serviceConfigPath = getServiceConfigPath(cliOptions?.configPath);
  if (serviceConfigPath) {
    const fileConfig = await loadServiceConfig(serviceConfigPath);
    inputOptions = mergeConfigs(inputOptions, fileConfig);
  }

  // 4. Apply CLI overrides (weave pattern)
  inputOptions = mergeConfigs(inputOptions, cliOptions);

  // 5. Validate sparse input options against SHACL
  await validateServiceConfigInput(inputOptions);

  // 6. Return side-by-side context (no merge)
  return {
    inputOptions,
    defaultOptions: PLATFORM_SERVICE_DEFAULTS
  };
}
```

## Node Config Resolution with Inheritance

## Side-by-Side Configuration Context

### Configuration Contexts (Not Merged Resolution)
```typescript
interface ServiceConfigContext {
  inputOptions: Partial<ServiceConfig>;    // From env + file + CLI (weave merge)
  defaultOptions: CompleteServiceConfig;   // Platform defaults
}

interface NodeConfigContext {
  inputOptions: Partial<NodeConfig>;       // From node + hierarchy walk
  defaultOptions: CompleteNodeConfig;      // Platform + service defaults
}

// Decision logic - check input first, fall back to defaults
function getServicePort(context: ServiceConfigContext): number {
  return context.inputOptions.port ?? context.defaultOptions.port;
}

function getVersioningEnabled(context: NodeConfigContext): boolean {
  return context.inputOptions.versioningEnabled ?? context.defaultOptions.versioningEnabled;
}
```

## Node Config Resolution with Hierarchy Walking

### Resolution Process (Different from Weave Pattern)
```typescript
async function resolveNodeConfig(nodePath: string): Promise<NodeConfigContext> {
  // 1. Start with node-specific config
  let inputOptions: Partial<NodeConfig> = {};
  const nodeConfig = await loadNodeConfig(nodePath);
  if (nodeConfig) {
    inputOptions = { ...nodeConfig };
  }

  // 2. Walk up hierarchy to fill missing values (if inheritance enabled)
  if (isConfigInheritanceEnabled(nodePath)) {
    const hierarchy = getNodeHierarchy(nodePath); // excluding current node
    for (const ancestorPath of hierarchy) {
      const ancestorConfig = await loadNodeConfig(ancestorPath);
      if (ancestorConfig) {
        // Fill in missing properties from ancestor
        for (const [key, value] of Object.entries(ancestorConfig)) {
          if (!(key in inputOptions)) {
            inputOptions[key] = value;
          }
        }
      }
    }
  }

  // 3. Validate sparse input options
  await validateNodeConfigInput(inputOptions);

  // 4. Build complete defaults (platform + service overrides)
  const serviceConfig = getServiceConfig();
  const defaultOptions = {
    ...PLATFORM_NODE_DEFAULTS,
    ...serviceConfig.inputOptions.nodeDefaults
  };

  // 5. Return side-by-side context
  return {
    inputOptions,
    defaultOptions
  };
}
```

## RDF/JSON-LD with Dual Validation

### Ontology Structure
```turtle
# service-ontology.ttl
@prefix svc: <https://semantic-flow.org/service#> .
@prefix node: <https://semantic-flow.org/node#> .

svc:ServiceConfig a owl:Class ;
  rdfs:label "Service Configuration" .

svc:port a owl:DatatypeProperty ;
  rdfs:domain svc:ServiceConfig ;
  rdfs:range xsd:integer .

svc:nodeDefaults a owl:ObjectProperty ;
  rdfs:domain svc:ServiceConfig ;
  rdfs:range node:NodeConfig ;
  rdfs:comment "Override platform defaults for node configuration" .

# node-ontology.ttl
@prefix node: <https://semantic-flow.org/node#> .

node:NodeConfig a owl:Class ;
  rdfs:label "Node Configuration" .

node:versioningEnabled a owl:DatatypeProperty ;
  rdfs:domain node:NodeConfig ;
  rdfs:range xsd:boolean .
```

### Dual Validation: SHACL + Zod
```typescript
import { Validator } from "npm:shacl-engine";
import rdfDataModel from "@rdfjs/data-model";

// SHACL validation for RDF semantics
async function validateServiceConfigSHACL(jsonldConfig: any): Promise<void> {
  const validator = new Validator(serviceShapesDataset, { factory: rdfDataModel });
  const report = await validator.validate({ dataset: configAsDataset });

  if (!report.conforms) {
    throw new ConfigError(`SHACL validation failed: ${report.results.map(r => r.message).join(', ')}`);
  }
}

// Zod validation for TypeScript ergonomics and API safety
const ServiceConfigInputSchema = z.object({
  "@context": z.record(z.string()),
  "@type": z.literal("svc:ServiceConfig"),
  "svc:port": z.number().int().min(1000).max(65535).optional(),
  "svc:sentryEnabled": z.boolean().optional(),
  "svc:nodeDefaults": z.object({
    "node:versioningEnabled": z.boolean().optional(),
    "node:distributionFormats": z.array(z.string()).optional(),
    "node:templateMappings": z.record(z.string()).optional()
  }).optional()
});

// Combined validation
async function validateServiceConfigInput(jsonldInput: unknown): Promise<ServiceConfigInput> {
  // 1. Zod validation for structure and types
  const parsedConfig = ServiceConfigInputSchema.parse(jsonldInput);

  // 2. SHACL validation for semantic constraints
  await validateServiceConfigSHACL(parsedConfig);

  return parsedConfig;
}
```

## Sparse Configuration Examples

### Service Config (Optional File)
```jsonld
{
  "@context": {
    "svc": "https://semantic-flow.org/service#",
    "node": "https://semantic-flow.org/node#"
  },
  "@type": "svc:ServiceConfig",
  "svc:port": 8080,                         // Override default 3000
  "svc:sentryEnabled": true,                // Override default false
  "svc:nodeDefaults": {                     // Organization-wide node policies
    "node:versioningEnabled": false,        // Disable by default
    "node:distributionFormats": ["application/ld+json"],
    "node:templateMappings": {
      "node:resourcePage": "corporate-template.html"
    }
  }
  // Omitted settings use platform defaults
}
```

### Node Config (Sparse Per-Node)
```jsonld
{
  "@context": { "node": "https://semantic-flow.org/node#" },
  "@type": "node:NodeConfig",
  "node:versioningEnabled": true,           // Override service/platform default
  "node:templateMappings": {
    "node:resourcePage": "special-template.html"  // Node-specific template
  }
  // Omitted settings inherited from hierarchy or service defaults
}
```

## File Structure

```
flow-core/
├── src/
│   ├── config/
│   │   ├── types.ts                    # Config-specific TypeScript interfaces
│   │   ├── external/                   # References to external ontologies
│   │   │   ├── service-ontology-ref.ts # Reference to flow-service-ontology
│   │   │   └── node-ontology-ref.ts    # Reference to node-config-ontology
│   │   ├── schemas/                    # Zod schemas for JSON-LD configs
│   │   │   ├── service-config.ts       # ServiceConfigInputSchema
│   │   │   └── node-config.ts          # NodeConfigInputSchema
│   │   ├── resolution/
│   │   │   ├── service-config-resolver.ts    # Weave-pattern merger
│   │   │   ├── node-config-resolver.ts       # Hierarchy walker
│   │   │   └── merge-utils.ts                # Config merging utilities
│   │   ├── defaults.ts                 # Platform default configurations
│   │   └── index.ts
│   ├── loaders/                        # General-purpose loaders
│   │   ├── jsonld-loader.ts           # JSON-LD file parsing
│   │   └── env-loader.ts              # Environment variable processing
│   ├── validation/                     # General validation utilities
│   │   ├── shacl-loader.ts           # Load shapes from external repos
│   │   ├── shacl-validator.ts        # shacl-engine integration
│   │   ├── zod-validator.ts          # Zod schema validation
│   │   └── combined-validator.ts     # SHACL + Zod pipeline
│   └── logging/                       # Uses resolved service config

flow-service/
├── src/
│   ├── storage/
│   │   ├── quadstore.ts               # RDF storage for service
│   │   └── config-manager.ts          # Config access + RDF population
│   ├── api/
│   └── app.ts
```
```

## Implementation Phases

### Phase 1: Core Service Config
1. **Service ontology setup**: Reference external flow-service-ontology repo (.9)
2. **Zod schemas**: Manual schemas for service config JSON-LD (.8)
3. **Weave-pattern resolution**: env + file + CLI merge for service config (.8)
4. **Single-stage loading**: Complete config before service start (.9)
5. **Logging integration**: Three-tier logging using resolved config (.8)

### Phase 2: Node Config + API
1. **Node ontology setup**: Reference external node-config-ontology repo (.8)
2. **Node config resolution**: Hierarchy walking with sparse configs (.8)
3. **RDF population**: Copy configs to Quadstore after service start (.7)
4. **JSON-LD APIs**: Config CRUD endpoints with validation (.9)
5. **Semantic queries**: SPARQL endpoint for config introspection (.7)

### Phase 3: Advanced Features
1. **Contained services**: Bootstrap control of SPARQL, static server, etc (.8)
2. **Per-mesh defaults**: Runtime configuration via API (.7)
3. **Config management UI**: Web interface using Comunica widget (.6)
4. **SHACL → Zod generation**: Auto-generate schemas from shapes (.6)

## JSON-LD API Design

### Service Config Endpoints
```typescript
// GET /api/service/config - Returns resolved service config
app.get('/api/service/config', async (c) => {
  const context = await resolveServiceConfig();
  const resolved = mergeConfigContext(context);
  return c.json(resolved); // JSON-LD response
});

// PUT /api/service/config - Update service config file
app.put('/api/service/config',
  zValidator('json', ServiceConfigInputSchema),
  async (c) => {
    const input = c.req.valid('json');
    await validateServiceConfigSHACL(input);
    await saveServiceConfig(input);
    return c.json({ status: "updated" });
  }
);
```

### Node Config Endpoints
```typescript
// GET /api/mesh/{meshPath}/config - Returns resolved node config
app.get('/api/mesh/:meshPath/config', async (c) => {
  const meshPath = c.req.param('meshPath');
  const context = await resolveNodeConfig(meshPath);
  const resolved = mergeConfigContext(context);
  return c.json(resolved); // JSON-LD response
});

// PUT /api/mesh/{meshPath}/config - Update node config
app.put('/api/mesh/:meshPath/config',
  zValidator('json', NodeConfigInputSchema),
  async (c) => {
    const input = c.req.valid('json');
    const meshPath = c.req.param('meshPath');
    await validateNodeConfigSHACL(input);
    await saveNodeConfig(meshPath, input);
    return c.json({ status: "updated" });
  }
);
```

## Environment Variable Mapping

```bash
# Service configuration (mapped to svc: properties)
FLOW_CONFIG_PATH=./flow-service-config.jsonld
FLOW_SERVICE_PORT=3000  # → svc:port
FLOW_LOG_LEVEL=info     # → svc:consoleLogLevel
FLOW_SENTRY_ENABLED=false # → svc:sentryEnabled

# Node defaults override (mapped to node: properties)
FLOW_DEFAULT_VERSIONING=true    # → svc:nodeDefaults/node:versioningEnabled
FLOW_DEFAULT_FORMATS=trig,jsonld # → svc:nodeDefaults/node:distributionFormats
```

## Key Design Decisions

### **Zod's Role Clarified**
- **File parsing validation**: JSON-LD config files need runtime validation (.9)
- **API request/response validation**: All config endpoints use JSON-LD (.9)
- **Environment variable mapping**: ENV → JSON-LD structure validation (.7)
- **TypeScript ergonomics**: Typed access to `"svc:port"` style properties (.8)

### **SHACL Library Choice**
- **Use shacl-engine**: 15-26x faster than alternatives, lighter weight (.8)
- **RDF/JS compatible**: Works well with Deno ecosystem (.8)
- **Performance critical**: Config validation happens frequently (.9)

### **In-Memory Representation**
- **Primary**: RDF datasets with Quadstore for semantic queries (.8)
- **Secondary**: Typed accessors for common properties (.8)
- **Benefits**: Native JSON-LD handling + SPARQL capability (.9)

### **Type Generation Strategy**
- **Phase 1**: Manual Zod schemas (faster to implement) (.8)
- **Phase 2**: Explore SHACL → Zod generation (JSON-LD makes it feasible) (.7)
- **JSON-LD insight**: Structure maps directly to TypeScript/Zod (.9)

## Recommended Implementation Order

**Yes, implement configuration first** (.9). Logging depends on resolved config, so the dependency order is:

1. **Config system** (this plan)
2. **Logging revamp** (uses resolved service config)
3. **Mesh operations** (uses resolved node configs)
4. **API layer** (uses both config types)

This ensures clean dependency flow and lets you validate the config patterns before building dependent systems.
