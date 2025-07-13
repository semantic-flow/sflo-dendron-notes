---
id: wjl3rmw6iyypzy13h4uci9q
title: Configuration
desc: ''
updated: 1752434880787
created: 1752434728498
---
# Flow Configuration System Implementation Plan

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

### SHACL + Zod Validation Strategy
```typescript
// Use both validation approaches
async function validateServiceConfig(config: any): Promise<ServiceConfig> {
  // 1. SHACL validation for RDF semantics
  const shaclResult = await validateWithSHACL(config, serviceConfigShapes);
  if (!shaclResult.conforms) {
    throw new ConfigError(`SHACL validation failed: ${shaclResult.text}`);
  }

  // 2. Zod validation for TypeScript types (generated from SHACL)
  return ServiceConfigSchema.parse(config);
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
│   │   ├── types.ts                    # TypeScript interfaces
│   │   ├── ontologies/                 # RDF ontology definitions
│   │   │   ├── service-ontology.ttl
│   │   │   ├── node-ontology.ttl
│   │   │   └── shared-ontology.ttl
│   │   ├── shapes/                     # SHACL validation shapes
│   │   │   ├── service-shapes.ttl
│   │   │   └── node-shapes.ttl
│   │   ├── schemas/                    # Generated Zod schemas
│   │   │   ├── service-config.ts       # From SHACL shapes
│   │   │   └── node-config.ts
│   │   ├── resolution/
│   │   │   ├── service-resolver.ts     # Weave-pattern merger
│   │   │   ├── node-resolver.ts        # Hierarchy walker
│   │   │   └── merge-utils.ts          # RDF-aware merging
│   │   ├── loaders/
│   │   │   ├── jsonld-loader.ts
│   │   │   ├── env-loader.ts
│   │   │   └── defaults.ts             # Platform defaults
│   │   ├── validation/
│   │   │   ├── shacl-validator.ts
│   │   │   └── zod-validator.ts
│   │   └── index.ts
│   └── logging/                        # Depends on resolved config
```

## Implementation Phases

### Phase 1: Service Config Foundation
1. **Basic ontology**: Service-specific vocabulary
2. **SHACL shapes**: Validation constraints
3. **Weave-pattern resolution**: env → file → CLI merge
4. **Platform defaults**: In-memory complete model
5. **Sparse file loading**: JSON-LD with missing properties

### Phase 2: Node Config System
1. **Node ontology**: Mesh operation vocabulary
2. **Hierarchy walking**: Up mesh tree for inheritance
3. **Service integration**: `svc:nodeDefaults` override
4. **Sparse node configs**: `_config-component` handling

### Phase 3: Validation & Tooling
1. **Dual validation**: SHACL + Zod integration
2. **Config migration**: Schema update tools
3. **Development tools**: Config watching, validation CLI
4. **Type generation**: Zod schemas from SHACL shapes

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

## Key Questions

1. **SHACL library**: shacl-engine

2. **Type generation**: Should we auto-generate Zod schemas from SHACL shapes, or maintain them separately?

3. **In-memory representation**: Store resolved configs as RDF graphs or plain objects?

4. **Config watching**: Should we implement weave-style config file watching for development?

5. **Migration strategy**: How do we handle ontology/shape evolution over time?
