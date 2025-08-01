---
id: rall4fbxm369okmy5383sf8
title: Weave Process
desc: ''
updated: 1753420415788
created: 1751128698638
---

- checks for required [[sflo.concept.mesh.resource-facet.system]] and creates them if missing
- optionally removes extraneous files, interactively if requested
- for changed [[sflo.concept.mesh.resource-facet.user]] datasets (i.e., need version bump)
  - if versioning is on:
    - creates a new [[sflo.concept.mesh.resource.element.flow.snapshot.version]] 
    - updates version metadata
  - regardless of whether versioning is on:
    - copies _next to _current
    - flags the unified dataset for regeneration
    - updates _meta-flow with new version information
- regenerates affected [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]]

```file
/repo-root/
├── _assets/
│   ├── _templates/
│   │   ├── default.html
│   │   ├── ontology.html
│   │   └── person.html
│   └── _css
│   │   ├── default.css
│   │   ├── ontology.css
│   │   └── person.css
└── my-ontology/
    ├── _config-flow              ← Node config (just syntax, etc.)
    ├── _assets/                  ← Optional node-specific assets
    │   ├── _templates/           ← Optional templates
    │   └── _css                  ← Optional css
    └── _data-flow
```

## Quirks

- if there's a pre-existing index.html under _assets, don't overwrite it?
  - or maybe use AI to update it.
  - (might need some marker html to indicate index.html was generated by weave process)

## Best Practices

### Weave Before Push

This ensures that in published meshes and sites:

- broken references are cleaned up
- [[sflo.concept.mesh.resource.element.flow.snapshot.current]] is identical to the latest version

## Features

### Interactive Mode

- to encourage data quality, the weave process can include an interactive mode, or modes. Perhaps one mode is naive, like "step through every bit of metadata" or "review every inferred metadata item", but you could have an AI-driven mode that optimizes for importance, identifying possible errors based on usage, etc.

### Tombstoning

If you know a sub-mesh is permanently moving to a new location (or even if a branch is being created somewhere else), you should be able to tell weave to insert references to the new location

### Resource Page Generation

- uses the [[sflo.product.service.design.in-memory]] to calculate template usage
- if no templates specified, and no "default template" exists in the root, it can generate its own
  - perhaps there's a default template and css distributed with the service in case its missing from the mesh root

## Scope

- **Single flow weave**: Update one (user) flow (data, ref, or config) + the corresponding meta-flow
- **Node weave**: Update all (user) flows in a node (each flow + meta gets woven)
- **Node tree weave**: Recursively weave nodes and their contained nodes

**Meta-flow co-weaving**: whenever any flow in a node gets woven, the meta-flow updates to reflect the new state. This ensures the meta-flow always accurately describes the current node state without requiring separate meta-weaving operations.

This gives you:
- **Local consistency**: Each node stays internally consistent
- **Flexible granularity**: Can weave at flow, node, or tree level as needed
- **Automatic meta updates**: No manual meta-flow management
- **Simple model**: No complex cross-node locking or coordination

For contained nodes mentioned in config flows, any references ideally point to the appropriate version snapshot. If there's inconsistency during concurrent operations, it's temporary and resolves when operations complete (.7).

This seems like the right balance between consistency guarantees and implementation complexity for the mesh architecture. We can always add more sophisticated coordination later if specific use cases demand it.