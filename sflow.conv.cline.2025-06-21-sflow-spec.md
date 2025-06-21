---
id: 8ry3qldaqi2j7b0emoomqdz
title: 2025 06 21 Sflow Spec
desc: ''
updated: 1750531903451
created: 1750528125174
---

## 10:55 AM - Recording Setup

**User:** read 'sflow-dendron-notes/sflow.conv.cline.recording-prompt.md' (see below for file content) and record to sflow.conv.cline.2025-06-21-sflow-spec

[User provided the recording prompt file content with guidelines for conversation recording]

**Cline:** I read the recording prompt guidelines and checked the existing (empty) target file. Following the guidelines to capture exchanges faithfully, compress tool responses into readable narrative, and maintain chronological order, I'm now recording this conversation setup exchange to establish the recording session.

## 11:05 AM - Getting Up to Speed on Sflow Framework

**User:** Get up to speed on the sflow framework by reading 'public-notes/sflow.md' and some concepts files. Beware as there is outdated/incorrect information everywhere. Past conversations with LLMs are in sflow.conv.* If you can treat everything skeptically until it's confirmed, especially from old conversations.

**Cline:** I explored the sflow framework documentation and recent conversations. Here's what I learned about the current state:

### Core Sflow Architecture

**Semantic Flow** is a system for managing and publishing knowledge graphs and other semantic data by leveraging GitHub, Gitlab, and other free static hosting services. It enables a **dereferenceable Semantic Web** where every HTTP IRI returns meaningful content.

## Features

- own your own data
- seamlessly integrate other data sources
- complete version history
- reliable persistence

## Elements

"SFlow" solves graph data evolutionand publishing problems with four weavable elements:

- **Stable Identifiers**: Permanent IRIs for things (people, concepts, namespaces) are essential to public data in general, and especially the Semantic Web
- **Versioned Datasets**: Versioned datasets with immutable, checkpointed history
- **Autonomous Namespacing**: organize your data in familiar URL-based hierarchies
- **Web Assets**: pictures, templates, audio, etc.

###  URL Schema (from recent conversations)

The framework produces websites where the familiar tree structure is composed with nested namespace "branches" and three kinds of leaves:

1. **Thing identifiers** - simple terminal folders that refer to things in the world. Or fictional things. Or concepts, etc.
2. **Datasets**- terminal folders that contain the actual data and metadata
3. **Assets**

Only namespaces can contain (sub-)elements, but dataset directories do contain a few sub-directories for metadata and previous versions.


### Key Architectural Decisions

**URL Semantics**: 
- Trailing slash = "refers" (abstract entities, series, concepts)
- Filename = "is" (concrete files, distributions, pages)

**Example Structure**:
```
ns/
├── _catalog/                 # namespace catalog (DSS)
├── people/                   # sub-namespace  
│   ├── _catalog/            # people namespace catalog
│   ├── _series/             # dataset series container
│   │   └── all-people/      # specific series
│   │       ├── v1/, v2/     # versioned datasets
│   │       ├── _next/       # work-in-progress
│   │       └── _catalog/    # series catalog
│   └── dave-richardson/     # thing identifier
│       ├── index.html       # human-readable
│       └── _catalog/        # thing catalog (DSS)
```

### Technical Implementation

**SSG Workflow**: Lume-based with plugins for RDF processing, automatic provenance generation, and version management.

**Version Bumping**: On command (not on save/commit), with _next folders for collaborative editing that get cleaned and normalized during version bumps.

**Aliasing**: Uses schema:sameAs for IRI renaming while maintaining backwards compatibility.

**Content**: Only v* folders contain RDF files (index.trig, index.jsonld, etc.). Everything else gets index.html for GitHub Pages compatibility.

### Current Status

The architecture appears well-defined with recent refinements around catalog management, versioning workflows, and namespace organization. The system aims to provide "dereferenceable, versioned semantic mesh" as foundation for knowledge graphs.

I noted the warning about outdated information in older conversations and focused on the most recent architectural decisions from the June 2025 discussions.
