---
id: ug4w9u17ynr2psj44yk9ych
title: Sf 6 Test Ns Dataset
desc: ''
updated: 1753028673462
created: 1752691832539
---

# SF-6: Create Test-NS Dataset for Flow-Service Development

## Summary

Create a comprehensive test namespace (test-ns) mesh repo that supports all three modalities of mesh operations and provides a foundation for developing flow-service functionality and API endpoints.

## Description

The test-ns [[sflo.concept.mesh-repo]] will serve as the primary development and testing data for the [[flow-service|sflo.product.service]], enabling systematic testing of semantic mesh operations across different interaction patterns. This dataset must support three distinct modalities of mesh operations: direct, api, and extranction.

It provides realistic data structures based on the [[DJ Radon radio show use case|sflo.use-case.djradons-radio-show]].

## Background

The flow-service needs a robust test environment that can demonstrate and validate all approaches to mesh construction and manipulation. The test-ns will bridge the gap between theoretical mesh architecture and practical implementation, providing concrete examples for API development.

## Requirements

### Multiple Initial States via Branching Strategy

**Branch Structure:**
- `main`: Fully populated mesh
- `minimal`: just a top-level README.md file
- `minimal-woven`: results of a first weave (all system resources created)
- `djradon`: a README.md file and a _ref-flow/_next/djradon.trig distribution
- `djradon-woven`: results of a first weave

### Realistic Data Using DJ Radon Radio Show Use Case

#### Core Entities

- DJ Radon (reference node)
- Musical picks dataset (versioned data node)
- Playlists namespace (data series node)
- Individual playlists (unversioned data nodes)
- Later: Albums and tracks (reference nodes from extraction)

#### Full Folder Structure

/                          # root (test-ns) namespace node
   djradon                 # ref node (refering to a human dj)
      bio                  # data node
      picks                # data node 
      underbrush           # ref node
         playlists         # data (series) node
            1996-11-10     # data node
            1996-11-17     # data node

### Technical Requirements


#### 5. Data Structure Specifications

**Namespace Structure:**
```
/test-ns/                          # Root namespace node
├── _config-flow/               # System metadata
├── _handle/                     # [[sflo.concept.mesh.resource.element.handle]]
├── _meta-flow/               # System metadata
├── _assets/                       # Shared assets
├── djradon/                       # Reference node (person)
│   ├── _config-flow/           # Reference data
│   ├── _handle/           # Reference data
│   ├── _meta-flow/          # Node metadata
│   ├── _ref-flow/           # Reference data
│   ├── bio/                      # Unversioned dataset
│   ├── picks/                    # Versioned dataset
│   └── playlists/                # Playlist namespace
│       ├── 1996-11-10/          # Individual playlist
│       └── 1996-11-17/          # Individual playlist
└── albums,artists,tracks, etc
```

**Flow Types:**
- `_meta-flow`: System metadata (provenance, versions)
- `_config-flow`: Node configuration (versioningEnabled, configInheritanceEnabled, distributionFormats, etc)
- `_ref-flow`: Reference data (descriptions, classifications)
- `_data-flow`: Payload data (actual content)
- `_unified-flow`: Aggregated views of multiple components

**Layer Structure:**
- `_current/`: Active/published layer
- `_next/`: Draft/staging layer
- `_v{n}/`: Historical version snapshots (for versioned datasets)

### Implementation Strategy

1. Create  single root-node mesh structure following semantic mesh example
2. Implement DJ Radon reference node with biographical data
3. Create picks dataset with sample music data
4. Establish playlist namespace with historical examples

#### Phase 2: API Integration
1. Develop flow-service endpoints for node operations
2. Implement component management API
3. Create dataset upload and processing endpoints
4. Add weave operation triggers

#### Phase 3: Extraction and Auto-Generation
1. Implement named entity extraction from RDF dataset uploads
2. Create reference nodes for extracted entities (metadata only)
3. Establish linking mechanisms between datasets and references
4. Develop batch processing for large RDF datasets
5. **Note**: Binary file resources (audio, images) must be handled via other modalities

#### Phase 4: Branch Strategy Implementation
1. Create specialized branches for different development scenarios
2. Implement branch-specific test data
3. Document switching procedures between branches
4. Establish CI/CD testing across branches

### Acceptance Criteria

- [ ] DJ Radon use case is fully implemented across all modalities
- [ ] Branch strategy supports different development scenarios
- [ ] Documentation covers branch strategy
- [ ] test-ns is browsable, i.e., [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]] for all [[sflo.concept.mesh.resource.folder]]

### Dependencies

- RDF processing libraries (for .trig and JSON-LD support)
- Semantic mesh architecture specifications
- DJ Radon radio show use case definition


## Related Documentation

- `sflo.concept.semantic-mesh.example.md` - Core mesh structure patterns
- `sflo.use-case.djradons-radio-show.md` - Primary use case specification
- `sflo.product.service.design.api.md` - API design patterns
- `sflo.contributors.general-guidance.md` - Development standards
