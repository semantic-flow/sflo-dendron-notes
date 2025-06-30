---
id: xebek3dtv2zgs9ah0vbv57g
title: Semantic Flow General Guidance
desc: ''
updated: 1751265773402
created: 1751259888479
---

**Semantic Flow** is a framework for managing knowledge graphs and other Semantic Web resources in publish-ready [[semantic meshes|sflow.concept.mesh]]

## Workspace Components

The workspace contains multiple interconnected components:

- **sf-api/**: API component for creating, augmenting and maintaining [[sflow.concept.mesh-repo]] (currently empty structure)
- **sf-cli/**: Command-line application that consumes the sf-api
- **sf-service/**: Service component (currently empty structure)  
- **sflo-ontology/**: Ontology definitions (currently empty)
- **sflow-dendron-notes/**: Complete documentation and specification in Dendron format
- **test-ns/**: Test mesh repo

## Key Concepts

### Semantic Mesh

A dereferenceable, versioned collection of semantic data and other resources, where every HTTP URI returns meaningful content.

#### Core Components

- **Mesh Resources**: 
  - **Nodes**: Semantic Atoms
    - **Datasets Nodes**: Bundles of data with optional quasi-immutable, versioned history
    - **Namespace Nodes**: basically empty folders for URL-based hierarchical organization
    - **Reference Nodes**: Refer to "things that exist" like people, or songs, or ideas
  - **Elements**: things that help define and systematize the nodes
  - **Handles**: things that let you refer to a node as a node instead of its referent
  - **Asset Trees**: elements that allow you to attach arbitrary collections of files and folders to a mesh; in a sense, these things are "outside" the mesh, and other than the top-level "_mesh" folder, they don't contain any other mesh resources

### Semantic Flow Workflow**: 

- In General: Mesh resource addition & editing → Weaving → Publishing using GitHub Pages 

### Semantic Site

- The repo IS the site:
  - can be served locally
  - SSG (Static Site Generator) not required
    - but static resource page generation should happen on every weave as necessary
  - after push, you should be able to see the changed mesh at the corresponding github pages URL

## RDF and Semantic Web

- meshes support multiple RDF formats (.trig, .jsonld, etc.)
- be mindful of RDF terminology and concepts
- Uses DCAT for dataset catalogs
- When referring to IRIs or URIs that are part of a semantic mesh, prefer the term URLs instead of IRI or URI
  - if you see a reference to IRI or URI, it might need updating, or it might mean a distinction should be drawn

## Documentation

- All specifications and design docs are in `sflow-dendron-notes/`
- Check conversation logs in `sflow.conv.*` for context on design decisions if necessary, but beware of superseded info 

### Documentation First

- unclear or anemic documentation should be called out
- when assisting with writing documentation, it should be kept concise and specific to the topic at hand
- whenever documentation is updated, any corresponding LLM conversation context should be updated too   
- to encourage documentation-driven software engineering, code comments should refer to corresponding documentation by filename, and the documentation and code should be cross-checked for consistency whenever possible 

### Documentation Architecture

Project documentation, specifications, journaling, and design choices are stored in `sflow-dendron-notes/` using Dendron's hierarchical note system. Key documentation includes:

- **Core concepts**: `sflow.concept.*` files define the semantic mesh architecture
- **Mesh docs**: `sflow.mesh.*` files define the semantic mesh architecture
- **Product specifications**: `sflow.product.*` files detail each component
- **Requirements**: `sflow.requirements.md`
- **Use cases**: `sflow.use-cases.*` files
- **Conversation logs**: `sflow.conv.*` files track design decisions and development history; BEWARE! These conversations contain information and decisions that have been superceded. Only reference conversations when necessary for historical context. Newer conversations are usually less misleading.

### Documentation Standards

- this project use wiki-style documentation. It's fine for articles to be extremely short. In general, articles should focus on their namesake topics.

### Component Development with Docs

- Each component (sf-api, sf-cli, sf-service) should follow the architecture defined in the documentation
- Refer to `sflow.products.*` files for component-specifc descriptions, requirements, etc

## Project Architecture

TBD
finalizing my design and concept documentation and my prompts
## Coding Standards

//TODO