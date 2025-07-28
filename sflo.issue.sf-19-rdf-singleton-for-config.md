---
id: z4bbjcfzfrrlftirc54242p
title: Sf 19 Rdf Singleton for Config
desc: ''
updated: 1753632430594
created: 1753625041811
---

# SF-19: Switch Service Config to In-Memory Singleton Based on Quadstore

## Background

The current service configuration system uses a cascading waterfall approach to resolve configuration values from CLI options, environment variables, JSON-LD config files, and platform defaults. The config is represented as JSON-LD objects and accessed via a side-by-side context pattern that preserves input and default layers.

## Objective

Refactor the service configuration loading and resolution to use an in-memory singleton backed by a Quadstore RDF store. This singleton will hold the service configuration as RDF data, enabling efficient querying, updates, and integration with the semantic data model.

## Current Approach Summary

- Configuration is resolved in this order: CLI > ENV > Config File > Defaults.
- Environment variables and CLI options are parsed and converted into partial JSON-LD config objects.
- The config file is loaded and parsed as JSON-LD.
- All input layers are merged into a single inputOptions object.
- Platform defaults are loaded separately.
- The resolved context contains both inputOptions and defaultOptions side-by-side.
- Accessors provide typed access to config values, preferring inputOptions over defaults.
- Validation ensures correctness of critical config values.

## Proposed Design for Quadstore Singleton Integration

1. **Singleton Quadstore Instance**
   - Create a generic singleton module (e.g., `quadstore.ts`) that initializes and exports a shared Quadstore instance backed by an in-memory database (MemoryLevel).
   - This singleton will serve as the central RDF store for all semantic data, including service configuration.

2. **Separate Config Graphs**
   - Maintain separate RDF graphs within the Quadstore for:
     - Platform defaults (service defaults and node defaults as distinct graphs)
     - Input configuration graphs (from CLI, ENV, config files, including separate service and node config files)
     - A merged or resolved graph representing the effective in-use configuration, blending input and defaults

3. **Initialization and Loading**
   - On service startup, initialize the Quadstore singleton.
   - Load configuration from environment variables, CLI options, and config files as currently done.
   - Convert the merged input configuration into RDF triples and load into the appropriate input config graphs in the Quadstore.
   - Load platform defaults into their respective graphs as fallback data.

4. **Configuration Access**
   - Refactor or augment the current `ServiceConfigAccessor` to query the Quadstore for configuration values.
   - Implement fallback logic within SPARQL queries or accessor methods to respect the waterfall precedence.
   - Provide reactive or event-driven updates if configuration changes in the Quadstore.

5. **Validation**
   - Perform validation on the configuration data loaded into the Quadstore.
   - Validation can be done before or after loading into the store.
   - Ensure errors are surfaced similarly to current ConfigError handling.

6. **Persistence and Updates**
   - Optionally support persisting changes back to config files or environment variables if needed.
   - Support runtime updates to configuration via the Quadstore singleton.

## Handling Node Defaults

- Rename `PLATFORM_NODE_DEFAULTS` to `PLATFORM_IMPLICIT_NODE_CONFIG` to reflect its role as implicit fallback defaults.
- Rename `SERVICE_NODE_DEFAULTS` to `ROOT_NODE_CONFIG_TEMPLATE` to reflect its role as a template for root node configs of meshes created by the service.
- Instead of embedding node defaults within the service config, maintain an optional separate `node-defaults-config.jsonld` file.
- Load node defaults into a dedicated graph in the Quadstore.
- This separation avoids duplication and clarifies the distinction between service and node defaults.
- Queries and accessors can combine service and node config graphs as needed to resolve effective configuration.

## Benefits

- Centralized RDF-based config store aligns with the semantic data model.
- Enables richer querying and integration with other RDF data.
- Supports dynamic updates and reactive config management.
- Maintains existing config waterfall semantics with enhanced flexibility.
- Modular separation of service and node defaults improves clarity and maintainability.

## Next Steps

- Implement the Quadstore singleton module (`quadstore.ts`) with in-memory backend.
- Refactor config loading to populate separate graphs in the Quadstore.
- Refactor config accessors to query the Quadstore with fallback logic.
- Add tests to verify equivalence with current config behavior.
- Gradually migrate service components to use the new Quadstore-backed config system.

---

This refined plan preserves the existing config resolution semantics while introducing an RDF-backed in-memory singleton with modular graph separation for enhanced flexibility and semantic integration.
