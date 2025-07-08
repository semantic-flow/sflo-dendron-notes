---
id: sif626tmt3mw3nir5gv1s1a
title: unified dataset
desc: ''
updated: 1751756078034
created: 1751599398246
---

## Overview

A **unified dataset** provides convenient aggregated access to a mesh node's current state by combining data from its [[sflo.concept.mesh.resource.element.node-component]] (meta; possibly ref, or ref and data) as named graphs into a single "current" [[sflo.concept.mesh.resource.element.node-component.layer]] with corresponding distribution files.

## Concept

For each mesh node (e.g., `ns/djradon/bio-dataset/`), generate combined distribution files:
- `_unified/bio-dataset_unified.trig` - Combined dataset in TriG format
- `_unified/bio-dataset_unified.jsonld` - Combined dataset in JSON-LD format
- Additional formats as needed
- unlike its sibling datasets, it is not a [[sflo.concept.mesh.component-facet.v-series]] and is not considered [[sflo.concept.mesh.resource.element.node-component]]

## Architecture

### IRI Structure & Single Referent Compliance
- **Node IRI** (`ns/djradon/bio-dataset/`) refers to its semantic entity (abstract dataset or thing)
- **Unified Dataset IRI** (`ns/djradon/bio-dataset/_unified/`) refers to the aggregated [[sflo.concept.mesh.resource.element.node-component.layer]]
- **Distribution files** are serializations of the unified dataset
- **No violation** of [[single referent principle|sflo.principle.single-referent]]

### Content Composition
```turtle
# Example unified dataset content
@prefix sflo: <https://semantic-flow.github.io/sflo-ontology/> .

# Unified dataset describes its aggregation
<ns/djradon/bio-dataset/_unified/> a dcat:Dataset ;
  dct:title "Unified view of bio-dataset node" ;
  sflo:aggregatedFrom [
    sflo:metaDataset <ns/djradon/bio-dataset/_meta/_current/> ;
    sflo:dataDataset <ns/djradon/bio-dataset/_data/_current/> ;
    sflo:aggregatedAt "2025-01-03T19:06:34Z"^^xsd:dateTime
  ] .

<ns/djradon/bio-dataset/_meta/_current/> {
  # Current meta dataset content
}

<ns/djradon/bio-dataset/_data/_current/> {
  # Current data dataset content
}
```

## Benefits

### Developer Experience
- **Single file access**: Complete node state in one distribution
- **Atomic consistency**: Guaranteed consistent snapshot of all components
- **Simplified client code**: No need to understand internal node structure

### Performance
- **Reduced HTTP requests**: One file instead of 3+ component requests
- **Bandwidth efficiency**: Optimized for common "give me everything about this node" use case

### Provenance 
- **Version tracking**: Metadata about which component versions were aggregated
- **Audit trail**: Clear record of what was combined and when
- **Debugging**: Easy to see complete node state at specific point in time

## Implementation Considerations

### Generation Strategy
- **Event-driven**: Regenerate when any 
- **Scheduled**: Periodic batch generation
- **Hybrid**: Combination based on node importance/access patterns

### Synchronization
- **Component changes**: Must trigger unified dataset updates
- **Consistency checks**: Validate component datasets before aggregation
- **Error handling**: Graceful degradation when components missing/invalid
- **Race conditions**: Handle concurrent updates to components

### Storage
- **Disk space**: Additional storage overhead for duplicated data
- **Cache management**: Balance between freshness and performance
- **Cleanup**: Remove stale unified datasets when nodes deleted

## Technical Requirements

### File Naming
- Pattern: `{node-name}_unified.{format}` (e.g., `bio-dataset_unified.trig`)
- Consistent across all node types
- Clear differentiation from component dataset files

### Content Validation
- **Component availability**: All required datasets must exist
- **RDF validity**: Generated content must be valid RDF
- **Namespace consistency**: Unified dataset must use consistent prefixes
- **Graph organization**: Clear separation of component data via named graphs

### API Integration
- **HTTP headers**: Appropriate content-type and caching headers
- **ETags**: Support for conditional requests
- **Last-Modified**: Cache validation support
- **Content negotiation**: Serve appropriate format based on Accept header

## Node Type Variations

### Namespace Nodes
- **Components**: name + meta + handle
- **Focus**: Organizational and containment information
- **Use case**: Directory listings, navigation, administrative metadata

### Reference Nodes  
- **Components**: name + meta + handle + ref
- **Focus**: External entity representation
- **Use case**: Entity profiles, reference resolution, relationship mapping

### Data Nodes
- **Components**: name + meta + handle + data
- **Focus**: Dataset abstraction and organization
- **Use case**: Dataset discovery, metadata aggregation, series information

## Future Enhancements

### Query Capabilities
- **SPARQL endpoint**: Query unified datasets directly
- **Filtering**: Request specific component subsets
- **Projection**: Return only requested properties

### Streaming
- **Large nodes**: Stream unified dataset for nodes with extensive data
- **Incremental updates**: Push component changes to subscribed clients

### Compression
- **Deduplication**: Optimize repeated triples across components
- **Format optimization**: Use most efficient serialization for each use case

## Architectural Implications

### Data Node Philosophy
This proposal raises questions about what data node IRIs actually refer to:

**Current assumption**: Data node IRI refers to abstract dataset concept
**Challenge**: Data nodes also contain name/meta components, making them composite

**Possible interpretations**:
1. **Composite referent**: Data node IRI refers to the entire node (name + meta + data)
2. **Primary dataset**: Data node IRI refers only to primary dataset, `_handle/` refers to node-as-container
3. **Symmetrical with reference nodes**: Data node works like reference node (IRI = primary entity, handle = node)

### Why Not Unified by Default?

- by keeping versioning out of a unified dataset, you can track evolution more granularly
- perhaps a single unified version would trigger an endless cascade as a metadata update (calculated after everything else is bumped) would cause a new unified version, which would trigger another metadata update, which would trigger another unified version

**Current separation benefits**:
- **Node type detection**: Folder existence (`_ref/` vs `_data/`) indicates node type without parsing
- **Granular access**: Clients can request specific components (meta-only, data-only)
- **Independent versioning**: Components can evolve at different rates
- **Clear semantics**: Each dataset has focused, well-defined purpose

**Unified approach benefits**:
- **Simplicity**: Single dataset per node reduces complexity
- **Atomic consistency**: All components always synchronized

## Related Concepts

- [[sflo.concept.mesh.resource.node]] - Base mesh node architecture
- [[sflo.concept.mesh.resource.element.node-component.metadata]] - Centralized metadata
- [[sflo.concept.mesh.resource.element]] - Dataset element concepts
- [[sflo.principle.single-referent]] - Architectural constraint this preserves
- [[sflo.concept.mesh.resource.element.node-component.layer.distribution]] - Distribution file concepts
