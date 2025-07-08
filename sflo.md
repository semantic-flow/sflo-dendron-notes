---
id: ggw3ek46ptgebsgxgihr1dh
title: Semantic Flow framework
desc: 'ontology, tooling, and workflow for publishing semantic data'
updated: 1751957698024
created: 1716959415785
---

[[sflo.issue]] | [[sflo.todo]] | [[sflo.contributors.djradon.dev-log]]

A **dereferenceable, versioned semantic mesh** will be the foundation for a new
era of knowledge graphs.

**Semantic Flow** is a framework for managing and publishing resource indicators, knowledge graphs and other semantic data by leveraging GitHub, Gitlab, and other free static hosting services. It enables a **dereferenceable Semantic Web** where every HTTP IRI returns meaningful content.

[[sflo.concept.mesh-repo]] provide storage, [[Semantic Meshes|sflo.concept.mesh]] provide resource management and publishing, and [[semantic sites|sflo.concept.semantic-site]] support data discovery and explainability.

## Benefits

- own your own self-describing data and data schemas
- complete version history when you want it
- reliable persistence
- truly FAIR (Findable, Accessible, Interoperable, and Reusable)

## Features

- seamlessly integrate other data sources anywhere in your mesh
- generate and customize mini-sites or single-page applications for nodes in your mesh

## Mesh Organization

### Elements

with four types of weavable [[sflo.concept.mesh.resource.element]]:

- **Datasets**: bundles of data, with optional immutable, checkpointed history
- **Namespaces**: organize your data in familiar URL-based hierarchies
- **Identifiers**: Persistent URLs which refer to namespaces, datasets, and
  other things (like people, concepts, whatever). Good identifiers are useful
  for public data in general, and essential the Semantic Web

Files and folders that are present but aren't consistent with these elements are
ignored in the weaving process,

### Folders

- **Asset Bundles**: every identifier-folder can contain two file trees:
  - **Web Assets (the _assets folder)**: URL-addressable pictures, templates,
    audio, etc.

## Workflow

### Creation

- Semantic-flow is designed to work with git, and a good place to get started is
  with a new, empty git repository
- create one or more folders. In general, folders are considered identifier
  folders if they contain a subfolder named _id.
  - folders that do not contain an _id folder will be ignored.
- the "docs" folder name has a special meaning in some Github publishing
  workflows, so should be avoided as a identifier name in general. ^pnoqpr3ff4za

### Editing

- add new identifiers, namespaces, datasets or [[sflo.concepts.asset-bundle]],
  or modify existing ones

### Weaving

- creates support system files as needed
- checks mesh consistency and reports issues
- "bumps" versions of [[versioned datasets|sflo.concept.mesh.resource.element.node-component.versioned]]
- you can weave a part of your semantic mesh, or the whole thing

### Publishing

**Generator-free Workflow**: Semantic Meshes contain everything needed to function as sites "as-is", no build process required.  

**Aliasing**: Uses skos:closeMatch for node migration with backwards IRI compatibility.

**Content**: Every non-file URL gets an index.html for dereferencability. Only [[sflo.concept.mesh.resource.element.node-component.layer]] contain RDF files (index.trig, index.jsonld, etc.). 

## Key Constraints & Rules

### Mesh Constraints

- **Datasets**: Distribution files have intuitive, descriptive names
  (`people-data.trig` in `people-data/`)
- **Case sensitivity**: Preserved - file capitalization must match folder
- **Mult distribution**: only one "working" distribution per dataset, although
  the site can have other distribution generated

### Content & Format Handling

- **Reference pages**: Usually generated from templates, but could be overridden
  by putting an index.html in a resource folder
- **RDF formats**: Multiple distributions per version (`.trig`, `.jsonld`, etc.)
- **Assets**: relative paths (`./_assets/images/...`) for portability
- **Machine discovery**: [[sflo.concept.mesh.resource.element.node-component.metadata]] specify available formats and series
  info
- **Things vs Data**: Clear separation - things are presentation, DSSs are data

## Versioning & Immutability

- **Source simplicity**: No versioning in source - just current content
- **Output versioning**: SSG creates `v*` versions when content changes
- **Immutable assets**: Once published, assets never change (use new filenames)
- **Rollback capability**: Git commits after successful generation
- **Permanence checking**: Validates no historical IRIs lost, no immutable data
  changed

### Meta Dataset Content & Purpose

- **cataloging**: Directory of contained elements, including distributions, based on DCAT
- **discovery**: Discovery metadata - what dataset series reference this
  thing, basic identification
- **provenance**: based on PROV

## Import & External Data

- **Namespace-aware**: Import process assigns local IRIs within appropriate
  namespaces
- **File naming**: External data renamed to match target folder structure
- **Ontology import**: External ontologies get local IRIs with `schema:sameAs`
  to external IRIs
- **Validation**: Import process validates filename matches target location

## Aliasing vs Renaming

**True aliasing**:

- **Purpose**: Multiple names for the same resource
- **Implementation**: `schema:sameAs` relationships in alias distributions
- **Allowed for**: Things and DSSs
- **Not allowed for**: Namespaces (too complex), datasets (immutable)

**Renaming/moving**:

- **Purpose**: Change canonical location/name
- **Implementation**: `owl:deprecated` + `rdfs:seeAlso` or `dcterms:replaces`
- **Namespace moves**: Expensive operation copying all content, optional
  redirect preservation

## Publishing Workflow

1. Edit content in `src/` (things, datasets, assets)
2. Commit source changes to Git
3. Run SSG build process
4. SSG compares source with last published versions
5. Create new `v*` versions in `docs/` for changed datasets
6. Generate updated `_current/` folders and `index.html` files
7. Auto-generate all `_meta/` metadata from source data and templates
8. Copy immutable assets to `docs/assets/`
9. Commit generated output to Git (rollback capability)
10. Deploy `docs/` folder to GitHub Pages

## Technical Implementation

- **Platform**: GitHub Pages serving from `docs/` folder
- **SSG**: Lume (Deno-based) with custom plugins for RDF processing
- **Source vs Output**: Clean separation - source for authoring, output for
  publication
- **Data Sources**: Contained in repo (`src/` folder) or external retrieval via
  import tools
- **Content negotiation**: Via catalog metadata (GitHub Pages has no custom
  headers)
- **Provenance**: Built into RDF data as metadata triples
- **Asset management**: Immutable `assets/` namespace prevents breaking HTML
  history
- **Permanence checking**: Validates no historical IRIs lost, no immutable data
  changed

## Semantic Foundation

- **Primary ontology**: sflow (semantic flow ontology for dataset series and
  versioning)
- **Compatibility**: Works with any RDF ontology (Kosmion, schema.org, FOAF,
  etc.)
- **Philosophy**: "Thing" over "entity" (following `schema:Thing` as most
  general)

## Current Status

- **Active development**: Architecture design finalized
- **Next steps**: SSG tooling implementation, template system, import tools
- **Integration**: LLM-assisted import for ambiguous dataset situating


---

## Features

- an [[sflo.concept.mesh-repo]] could have files/distributions for more than one
  namespace, so it should have each of those namespaces segregated by a
  top-level folder

## Related Prodcuts / Possible Components

- [[prdct.comunica.query-sparql-file]]
