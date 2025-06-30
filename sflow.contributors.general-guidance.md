---
id: xebek3dtv2zgs9ah0vbv57g
title: Semantic Flow General Guidance
desc: ''
updated: 1751261831335
created: 1751259888479
---

**Semantic Flow** is a framework for managing knowledge graphs and other Semantic Web resources in publish-ready [[semantic meshes|sflow.mesh]]

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

### Core Components

- **Mesh Resources**: 
  - **Nodes**: Semantic Atoms
    - **Datasets Nodes**: Bundles of data with optional quasi-immutable, versioned history
    - **Namespace Nodes**: basically empty folders for URL-based hierarchical organization
    - **Reference Nodes**: Refer to "things that exist" like people, or songs, or ideas
  - **Elements**: things that help define and systematize the nodes
  - **Handles**: things that let you refer to a node as a node instead of its referent
  - **Asset Trees**: elements that allow you to attach arbitrary collections of files and folders to a mesh; in a sense, these things are "outside" the mesh, and other than the top-level "_mesh" folder, they don't contain any other mesh resources

## Semantic Flow Workflow**: 

- In General: Mesh resource addition & editing → Weaving → Publishing using GitHub Pages 

## Semantic Site

- The repo IS the site:
  - can be served locally
  - SSG (Static Site Generator) not required
    - but static resource page generation should happen on every weave as necessary
  - after push, you should be able to see the changed mesh at the corresponding github pages URL



## Documentation First

- unclear or anemic documentation should be called out
- when assisting with writing documentation, it should be kept concise and specific to the topic at hand
- whenever documentation is updated, any corresponding LLM conversation context should be updated too   
- to encourage documentation-driven software engineering, code comments should refer to corresponding documentation by filename, and the documentation and code should be cross-checked for consistency whenever possible 

## Coding Standards
