---
id: osqi7xn4mw3j9v0kvsoc5uo
title: Why Dont Data Nodes Contain Distributions Directly
desc: ''
updated: 1751435070654
created: 1751387201637
---

## Question

Why don't [[data nodes|sflo.concept.mesh.resource.node.data]] contain distribution files directly? Why do I need to go to `_current/` to find the actual data?

## Answer

Data nodes represent **abstract data concepts**, not concrete data instances. This separation provides several important benefits:

### Clear Semantic Distinction

- **Data node** (`/ns/monsters/`): "The concept of monster data"
- **Dataset element** (`/ns/monsters/_current/`): "The current monster data files"

This allows you to reference the stable concept separately from any specific data instance.

### Stable Identity

The data node provides a permanent, stable identifier for the data concept that persists even as the concrete data changes over time. You can always refer to "monster data as a concept" using `/ns/monsters/` regardless of how many versions exist.

### Temporal Organization

By separating the concept from concrete instances, data nodes can cleanly organize different temporal states:
- `_current/` - current data
- `_next/` - draft changes  
- `_v1/`, `_v2/` - historical versions

### Consistent Architecture

This mirrors how [[reference nodes|sflo.concept.mesh.resource.node.reference]] work:
- **Reference nodes**: Abstract entity concept + `_ref/` element with concrete data
- **Data nodes**: Abstract data concept + `_current/` element with concrete data

### Metadata Separation

The data node's [[meta dataset|sflo.concept.mesh.resource.element.node-component.metadata]] contains metadata about the data concept itself, while each temporal dataset element contains metadata about that specific data instance.

## Analogy

Think of it like a library:
- **Data node** = "The concept of the Encyclopedia Britannica"
- **Dataset elements** = Specific editions (1990 edition, 2020 edition, current edition)

You can refer to "Encyclopedia Britannica" as a concept without specifying which edition, or you can reference a specific edition when you need concrete data.