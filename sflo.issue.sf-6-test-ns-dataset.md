---
id: ug4w9u17ynr2psj44yk9ych
title: Sf 6 Test Ns Dataset
desc: ''
updated: 1752730629408
created: 1752691832539
---

# SF-6: Create Test-NS Dataset for Flow-Service Development

## Summary

Create a comprehensive test namespace (test-ns) dataset that supports all three modalities of mesh operations and provides a foundation for developing flow-service functionality and API endpoints.

## Description

The test-ns dataset will serve as the primary development and testing environment for the flow-service, enabling systematic testing of semantic mesh operations across different interaction patterns. This dataset must support three distinct modalities of mesh operations while providing realistic data structures based on the DJ Radon radio show use case.

## Background

The flow-service needs a robust test environment that can demonstrate and validate all approaches to mesh construction and manipulation. The test-ns will bridge the gap between theoretical mesh architecture and practical implementation, providing concrete examples for API development.

## Requirements

### Core Functionality Requirements

#### 1. Support for Three Mesh Operation Modalities

**A. Direct Manual Construction Modality**
- Pre-built node folder structures with user-editable layers
- Manual mesh resource creation (nodes; flows, snapshots, distributions and other elements)
- File-system based editing workflows
- Validation of hand-crafted mesh structures

**B. API-Driven Node Creation Modality**
- Flow-service API endpoints for programmatic node creation
- Support for root node initialization
- Component and element management via API
- RESTful mesh resource manipulation

**C. Dataset Distribution Upload + Extraction Modality**
- Upload mechanisms for RDF dataset distributions (.trig, .jsonld, etc.)
- Automatic named entity extraction from semantic data
- System-generated reference and data nodes
- Batch processing of semantic data
- **Limitation**: Cannot handle binary file resources (audio, images, etc.) - only RDF data
- File resources must be handled via Direct Manual Construction or API-Driven modalities

#### 2. Multiple Initial States via Branching Strategy

**Branch Structure:**
- `main`: Fully populated mesh with all modalities demonstrated
- `minimal`: Basic namespace structure only
- `api-ready`: Structure optimized for API-driven development
- `upload-ready`: Structure prepared for dataset distribution uploads
- `empty`: Clean slate for testing mesh initialization

#### 3. Realistic Data Using DJ Radon Radio Show Use Case

**Core Entities:**
- DJ Radon (reference node)
- Musical picks dataset (versioned data node)
- Playlists namespace (container for playlist series)
- Individual playlists (data nodes)
- Albums and tracks (reference nodes from extraction)

### Technical Requirements

#### 4. Flow-Service API Endpoints to Support

**Mesh Management:**
- `POST /api/mesh/init` - Initialize new mesh
- `POST /api/mesh/node` - Create new node
- `GET /api/mesh/node/{path}` - Retrieve node
- `PUT /api/mesh/node/{path}` - Update node
- `DELETE /api/mesh/node/{path}` - Delete node

**Component Management:**
- `POST /api/mesh/node/{path}/component` - Add component
- `GET /api/mesh/node/{path}/component/{type}` - Get component
- `PUT /api/mesh/node/{path}/component/{type}` - Update component

**Dataset Operations:**
- `POST /api/mesh/dataset/upload` - Upload RDF dataset distribution
- `POST /api/mesh/dataset/extract` - Extract entities from RDF dataset
- `GET /api/mesh/dataset/{path}/versions` - List dataset versions
- `POST /api/mesh/dataset/{path}/weave` - Trigger weave operation
- `POST /api/mesh/assets/upload` - Upload binary file resources (separate from dataset extraction)

#### 5. Data Structure Specifications

**Namespace Structure:**
```
/test-ns/                          # Root namespace node
├── _meta-component/               # System metadata
├── _assets/                       # Shared assets
├── djradon/                       # Reference node (person)
│   ├── _ref-component/           # Reference data
│   ├── _meta-component/          # Node metadata
│   ├── bio/                      # Unversioned dataset
│   ├── picks/                    # Versioned dataset
│   └── playlists/                # Playlist namespace
│       ├── 1996-11-10/          # Individual playlist
│       └── 1996-11-17/          # Individual playlist
└── albums/                       # Auto-generated from extraction
    ├── nevermind/               # Album reference node
    └── ok-computer/             # Album reference node
```

**Component Types:**
- `_meta-component`: System metadata (provenance, versioning)
- `_ref-component`: Reference data (descriptions, classifications)
- `_data-component`: Payload data (actual content)
- `_unified-component`: Aggregated views of multiple components

**Layer Structure:**
- `_current/`: Active/published layer
- `_next/`: Draft/staging layer
- `_v{n}/`: Historical version snapshots (for versioned datasets)

### Implementation Strategy

#### Phase 1: Manual Construction Foundation
1. Create basic mesh structure following semantic mesh example
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

### Expected Outcomes

1. **Complete Test Environment**: Comprehensive test-ns supporting all mesh operations
2. **API Foundation**: Working flow-service endpoints for core mesh operations
3. **Development Workflow**: Clear procedures for mesh development and testing
4. **Documentation**: Examples and patterns for mesh construction
5. **Validation Framework**: Automated testing of mesh structures and operations

### Acceptance Criteria

- [ ] All three modalities can be demonstrated with test-ns
- [ ] Flow-service API endpoints are functional and tested
- [ ] Branch strategy supports different development scenarios
- [ ] DJ Radon use case is fully implemented across all modalities
- [ ] Automated extraction creates valid reference nodes
- [ ] Weave operations work correctly on test data
- [ ] Documentation covers all usage patterns
- [ ] CI/CD pipeline validates mesh integrity

### Dependencies

- Flow-service basic infrastructure (logging, configuration)
- RDF processing libraries (for .trig and JSON-LD support)
- Semantic mesh architecture specifications
- DJ Radon radio show use case definition

### Definition of Done

The test-ns dataset is considered complete when:
1. All three modalities are functional and demonstrated
2. Flow-service can successfully perform CRUD operations on test-ns
3. Dataset distribution uploads trigger correct entity extraction
4. Branch switching works seamlessly for different development scenarios
5. All mesh resources validate against semantic mesh specifications
6. Performance benchmarks are established for typical operations
7. Documentation enables new developers to understand and extend the system

## Related Documentation

- `sflo.concept.semantic-mesh.example.md` - Core mesh structure patterns
- `sflo.use-case.djradons-radio-show.md` - Primary use case specification
- `sflo.product.service.design.api.md` - API design patterns
- `sflo.contributors.general-guidance.md` - Development standards
