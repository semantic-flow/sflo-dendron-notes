---
id: 9j8lhoyb0xkhg926nafgkai
title: data component
desc: ''
updated: 1751955148508
created: 1751560483669
---

## Overview

Data components (_data/) are [[node components|sflo.concept.mesh.resource.element.node-component]] that effectively associate a versioned dataset with a mesh node. They contain the actual content payload of data nodes and provide the primary data storage functionality within the semantic flow mesh architecture.

## Purpose

Data components serve as the primary content containers for mesh nodes, providing:

- **Content Storage**: Hold the actual data payload that defines the node's content
- **Version Management**: Support multiple versions of the same conceptual dataset
- **Format Diversity**: Enable multiple data format distributions (TTL, JSON-LD, etc.)
- **State Management**: Track current, draft, and versioned states of data

## Structure

Data components organize content through [[component layers|sflo.concept.mesh.resource.element.node-component.layer]]:

- `_current/` - Current stable version of the dataset
- `_next/` - Draft/work-in-progress version
- `_v1/`, `_v2/`, etc. - Versioned snapshots for historical access

Like all [[sflo.concept.mesh.resource.folder]], they should contain an `index.html` [[sflo.concept.mesh.resource.element.resource-page]] -- a human-readable description for the component


## Relationship to Data Nodes

Data components create the fundamental association between nodes and their content:


## Content Types

Data components contain one or more 

## Versioning Strategy

Data components implement sophisticated version management:

- **Linear Versioning**: Sequential versions (v1, v2, v3...)
- **Branch Management**: Current stable vs. next draft
- **Snapshot Preservation**: Historical versions remain accessible
- **Aggregated Access**: Top-level files provide consolidated views

## Distribution Formats

Each component layer typically provides multiple format distributions:

- **Trig (.trig)**: Primary RDF serialization
- **JSON-LD (.jsonld)**: JSON-compatible linked data
- **RDF/XML (.xml or .trix)**: XML-based RDF serialization
- **N-Quads (.nq)**: Line-based RDF format

## Example

From the [[semantic mesh example|sflo.concept.semantic-mesh.example]]:

```
/test-ns/djradon-bio/_data/          # data component
├── _current/                        # current layer
│   ├── djradon-bio.ttl             # turtle distribution
│   ├── djradon-bio.jsonld          # json-ld distribution
│   └── index.html                  # layer interface
├── _next/                          # draft layer
│   ├── djradon-bio.ttl             # draft turtle
│   ├── djradon-bio.jsonld          # draft json-ld
│   └── index.html                  # layer interface
└── index.html                      # component interface
```

## Integration

Data components integrate with other mesh elements:

- **Metadata Components**: Provide provenance and management data
- **Reference Components**: Enable indirection and linking
- **Asset Trees**: Store associated files and media
- **Resource Pages**: Provide human-readable interfaces

Data components are the core content mechanism that enables the semantic flow mesh to function as a distributed, versioned knowledge management system.
