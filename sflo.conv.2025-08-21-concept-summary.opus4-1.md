---
id: kz73w03r0d1dh8crss0k1kx
title: Semantic Mesh - Comprehensive Summary for AI/LLMs
desc: Complete unified documentation of the Semantic Flow mesh architecture
updated: 1755843833587
created: 1755843750673
---

# Semantic Mesh: Comprehensive Documentation Summary

## Executive Summary

A **semantic mesh** is a dereferenceable, folder-structured, versioned corpus of semantic resources where every URL returns meaningful content. It provides a framework for:
1. **Minting IRIs** for referring to things on the semantic web
2. **Storing semantic data and datasets** that use those IRIs

The mesh maps directly from a Git repository's folder hierarchy to a published static site, enabling addressable, versioned, and transposable semantic data management without requiring server-side infrastructure.

## Part 1: Core Definition and Principles

### 1.1 Definition

A semantic mesh is a publish-ready collection of semantic resources with these characteristics:
- **Addressable**: Every resource has a unique URL-based identifier
- **Dereferenceable**: All URLs return meaningful content when accessed (HTML for humans, RDF for machines)
- **Versioned**: Data evolution is managed through flow snapshots with immutable historical versions
- **Transposable**: Meshes remain valid when moved between hosting locations
- **Composable**: Sub-meshes can be extracted and combined while maintaining integrity
- **Git-native**: Stored in ordinary Git repositories, publishable as static sites

### 1.2 Design Principles

1. **Dereferenceability for Humans**: Every folder URL returns a meaningful HTML page (index.html) for discovery and navigation
2. **Single Referent Principle**: Each URL identifies exactly one thing - concepts and content are clearly separated
3. **Pseudo-immutability**: Version snapshots are treated as immutable even though filesystems cannot enforce this
4. **Transposability**: Internal references use relative URIs; no hardcoded BASE URIs
5. **Composability**: Meshes can be combined or extracted; the weave process resolves broken references

## Part 2: Architecture - Nodes and Elements

### 2.1 Mesh Resources

The mesh has two fundamental resource types:

**Mesh Nodes** (extensible containers):
- Physically represented as folders
- Can contain other nodes and elements
- Extend the namespace with their folder name
- Always contain at least `_meta-flow/` and `_handle/`

**Mesh Elements** (terminal resources):
- Cannot contain other mesh resources
- Can be folders or files
- Support and define node structure and behavior

### 2.2 Node Types

#### Namespace Node
- **Purpose**: Organizational container
- **URL refers to**: The namespace itself
- **Required elements**: `_meta-flow/`, `_handle/`
- **Example**: `/ns/` organizing sub-namespaces

#### Reference Node  
- **Purpose**: Represents external entities (people, concepts, things)
- **URL refers to**: The external entity being referenced
- **Required elements**: `_meta-flow/`, `_handle/`, `_ref-flow/`
- **Example**: `/ns/djradon/` referring to the person Dave Jordan Radon

#### Data Node
- **Purpose**: Represents an abstract dataset concept
- **URL refers to**: The abstract dataset (not a specific version)
- **Required elements**: `_meta-flow/`, `_handle/`, `_data-flow/`
- **Optional**: `_ref-flow/` for metadata about the dataset concept
- **Example**: `/ns/census-2020/` referring to the concept of the 2020 census data

### 2.3 Element Types

#### Flows (Abstract DatasetSeries)
- **Metadata flow** (`_meta-flow/`): System metadata, versioning info, provenance
- **Reference flow** (`_ref-flow/`): Data about the entity a reference node refers to
- **Data flow** (`_data-flow/`): The actual payload data for data nodes
- **Config flow** (`_config-flow/`): Node configuration settings
- **Unified flow** (`_unified/`): System-generated aggregation of all node flows

#### Flow Snapshots (Concrete Datasets)
- **Current** (`_current/`): Stable published version, matches latest version snapshot
- **Next** (`_next/`): Mutable draft workspace for ongoing changes
- **Versioned** (`_v1/`, `_v2/`, etc.): Immutable historical snapshots

#### Other Elements
- **Handle** (`_handle/`): Allows referring to the node "as a mesh resource"
- **Asset tree** (`_assets/`): Arbitrary static files (images, CSS, templates)
- **Documentation**: README.md, CHANGELOG.md, index.html resource pages
- **Distributions**: RDF files in various formats (.trig, .jsonld, .ttl)

## Part 3: Addressing and URLs

### 3.1 URL Semantics

URLs in a semantic mesh follow strict patterns:

| URL Type    | Pattern            | Refers to                  | Example                             |
| ----------- | ------------------ | -------------------------- | ----------------------------------- |
| Concept URL | Ends with `/`      | Entity or abstract concept | `https://ex.org/ns/dave/`           |
| Content URL | Has file extension | Retrievable content        | `https://ex.org/ns/dave/index.html` |

### 3.2 URL Types and Referents

- **Namespace node**: `/ns/` → The namespace itself
- **Reference node**: `/ns/dave/` → Dave the person
- **Data node**: `/ns/census/` → The abstract census dataset concept
- **Abstract flow**: `/ns/census/_data-flow/` → The data flow DatasetSeries
- **Concrete snapshot**: `/ns/census/_data-flow/_current/` → Current dataset instance
- **Distribution**: `/ns/census/_data-flow/_v1/census.trig` → Actual RDF data file
- **Handle**: `/ns/dave/_handle/` → The dave node as a mesh resource
- **Resource page**: `/ns/dave/index.html` → Human-readable documentation

### 3.3 Relative Identifiers

Within distributions, references use relative paths:
- `../../` refers to parent node
- `../_meta-flow/_current/` refers to sibling flow snapshot
- Enables transposability and composition

## Part 4: Data Model and Versioning

### 4.1 Flow-Snapshot Architecture

**Flows** (abstract DatasetSeries):
- Represent evolving datasets over time
- Are versionable (controlled by config)
- Don't contain data directly

**Snapshots** (concrete Datasets):
- Temporal instances of flows
- Contain actual distributions
- Include current, next, and versioned snapshots

### 4.2 Versioning Rules

- **Only flows are versioned** - nodes themselves are never versioned
- **Version snapshots are immutable** - `_v1/`, `_v2/` should never change
- **_current equals latest version** - After weaving, current matches the newest version
- **_next is the working draft** - Mutable space for ongoing changes

### 4.3 Distribution Formats

Each snapshot can have multiple distributions in different RDF formats:
- **Primary**: TriG (.trig) - supports named graphs
- **Web-friendly**: JSON-LD (.jsonld) 
- **Human-readable**: Turtle (.ttl)
- **Others**: N-Quads, RDF/XML as needed

## Part 5: Lifecycle - The Weave Process

### 5.1 Weave Operations

The weave process maintains mesh integrity by:
1. **Creating missing system elements** (resource pages, handles)
2. **Version management**:
   - Creates new `_vN` snapshot from `_next`
   - Copies `_next` to `_current`
   - Updates version metadata
3. **Regenerating derived content**:
   - Resource pages (index.html)
   - Unified datasets
   - Aggregated distributions
4. **Resolving broken references** for transposability

### 5.2 Weave Scope

- **Flow weave**: Update one flow and its metadata
- **Node weave**: Update all flows in a node
- **Tree weave**: Recursive weave of node and descendants

### 5.3 Best Practices

- **Weave before push** to ensure published consistency
- **Use interactive mode** for important updates
- **Configure templates** in `_assets/_templates/`

## Part 6: Configuration and Inheritance

### 6.1 Configuration Types

**OperationalNodeConfig**: 
- Controls a node's actual behavior
- Doesn't participate in inheritance chain
- Settings: versioning, distribution formats, template usage

**InheritableNodeConfig**:
- Provides defaults for child nodes
- Can exist at platform, service, or node level
- Participates in inheritance chain

### 6.2 Inheritance Resolution

Precedence (most specific wins):
1. Node's own OperationalNodeConfig
2. Parent's InheritableNodeConfig
3. Grandparent's InheritableNodeConfig (walks up tree)
4. Service-level InheritableNodeConfig
5. Platform-level InheritableNodeConfig

### 6.3 Configuration Firewall

`inheritableConfigPropagationEnabled: false` creates a hard boundary blocking ALL inheritance from flowing to child nodes.

## Part 7: Provenance and Metadata

### 7.1 Provenance Architecture

- **Stored in meta-flow** for version snapshots only
- **Uses PROV standard** with semantic flow extensions
- **Delegation chains** track authorization hierarchy
- **Fragment URIs** provide stable references

### 7.2 Provenance Components

```turtle
:weaveActivity a prov:Activity ;
    prov:startedAtTime "2025-01-20T14:30:00Z" ;
    prov:used <../_data-flow/_v46/> ;
    prov:generated <../_data-flow/_v47/> ;
    prov:wasAssociatedWith <agent> .
```

### 7.3 Rights Management

- Copyright defaults to first agent in delegation chain
- Rights specified at snapshot level using Dublin Core terms
- Licensing tracked with dcterms:license

## Part 8: Publishing and Deployment

### 8.1 Git to Web Mapping

Repository structure maps directly to URL structure:
```
Git: /repo/ns/djradon/bio/
Web: https://site.org/ns/djradon/bio/
```

### 8.2 Static Hosting

Meshes work with any static file server:
- GitHub Pages (automatic from repos)
- GitLab Pages
- Netlify, Vercel
- Apache/Nginx
- Local file servers

### 8.3 Transposition

Moving meshes between locations:
- All internal references remain valid (relative URIs)
- Publication history tracked in metadata
- No build step required

## Part 9: Advanced Features

### 9.1 Unified Datasets

System-generated aggregation providing:
- Single-file access to all node data
- Atomic consistency across flows
- Independent versioning
- Reduced HTTP requests

### 9.2 Aggregated Distributions

Roll-up of contained nodes' data:
- Useful for modular ontologies
- Generated during weave
- Multiple format outputs
- Automatic deduplication

### 9.3 Composability

Combining and extracting meshes:
- **Import**: Copy external mesh content permanently
- **Embed**: Maintain connection for updates (git subtree)
- **Extract**: Post-weave, any subtree is complete
- **Cross-mesh references**: Use absolute URIs

## Part 10: Standardization Notes

### 10.1 Recommended Conventions

**Folder Naming** (choose one and be consistent):
- Option A: `_meta-flow/`, `_ref-flow/`, `_data-flow/`, `_config-flow/`
- Option B: `_meta-component/`, `_ref-component/`, `_data-component/`, `_config-component/`
- *Current inconsistency needs resolution*

**File Naming**:
- Metadata: `{node-name}_meta.{ext}`
- Reference: `{node-name}_ref.{ext}`
- Data: `{node-name}.{ext}` (no suffix for payload data)
- Config: `config.jsonld`

### 10.2 Known Issues to Resolve

1. **Terminology**: Standardize on "flow" vs "component"
2. **Snapshot vs Layer**: Use "snapshot" consistently
3. **Distribution support**: Clarify TriG as primary format
4. **Documentation gaps**: Complete empty concept files
5. **Current semantics**: Confirm _current as stable/published

## Part 11: Quick Reference

### File Structure Template
```
/mesh-root/
├── _assets/                    # Root assets and templates
│   ├── _templates/
│   └── _css/
├── ns/                         # Namespace node
│   ├── _meta-flow/            # Metadata flow
│   │   ├── _current/
│   │   ├── _next/
│   │   └── _v1/
│   ├── _handle/               # Node handle
│   ├── person/                # Reference node
│   │   ├── _meta-flow/
│   │   ├── _ref-flow/        # Reference data
│   │   ├── _handle/
│   │   └── index.html
│   └── dataset/              # Data node
│       ├── _meta-flow/
│       ├── _data-flow/       # Payload data
│       │   ├── _current/
│       │   ├── _next/
│       │   └── _v1/
│       ├── _handle/
│       └── dataset.trig      # Aggregated distribution
└── index.html                # Root resource page
```

### Essential Commands (Conceptual)
```bash
# Initialize mesh
sflo init

# Create node
sflo create node --type reference ns/person

# Weave changes
sflo weave --node ns/dataset

# Publish
git push origin main  # GitHub Pages auto-publishes
```

## Part 12: For AI/LLM Context

When working with semantic meshes, remember:

1. **Every URL must resolve** - Either to content (files) or concepts (folders with index.html)
2. **Respect the Single Referent Principle** - Node URLs refer to their semantic referent, not the node itself
3. **Use relative references** internally for transposability
4. **Version only flows**, never nodes
5. **_current = latest version** after weaving
6. **Handles provide mesh resource identity** when needed
7. **Weave before publishing** to ensure consistency
8. **Configuration inheritance** flows from platform → service → parent nodes → node

This architecture enables distributed, versioned semantic data management using only static files and Git, making it accessible, permanent, and infrastructure-independent.

## Appendix: Vocabulary Quick Reference

- **Mesh**: Complete addressable semantic structure in a repository
- **Node**: Extensible namespace container (folder)
- **Element**: Terminal resource supporting nodes
- **Flow**: Abstract DatasetSeries evolving over time
- **Snapshot**: Concrete Dataset instance at a point in time
- **Distribution**: Serialized RDF file (.trig, .jsonld, etc.)
- **Handle**: Reference to node as mesh resource
- **Weave**: Process maintaining mesh integrity and versioning
- **Transposability**: Ability to move mesh while preserving structure
- **Composability**: Ability to combine/extract mesh components
