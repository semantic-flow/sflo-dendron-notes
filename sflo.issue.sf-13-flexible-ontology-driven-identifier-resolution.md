---
id: z1bjxm5s2zwu07j0ndhlcpf
title: 'SF-13: Ontology-Driven API Specification'
desc: >-
  Design for a generalized system to resolve user-friendly strings into formal,
  ontology-backed URIs at the API layer.
updated: 1753377851417
created: 1753329521764
---

- different-from: [[sflo.issue.sf-14-lexicon-low-overhead-fragment-based-uris-for-strings]]

### 1. Epic

Ontology-Driven API Specification

### 2. The Problem: Hardcoded API Schemas

When an API needs to accept a value from a controlled vocabulary (e.g., a `nodeType` which can be "Namespace", "Reference", etc.), the simplest approach is to hardcode this list into the API's validation schema (e.g., `z.enum(['Namespace', 'Reference'])`).

This creates a maintenance problem:
- **Dual Maintenance:** The list of valid types exists in two places: the formal ontology and the API code. When the ontology changes, a developer must remember to update the code.
- **Brittleness:** The API contract can easily become out of sync with the data model it's supposed to represent.
- **Lack of Stability:** If the API uses human-facing `rdfs:label`s for its values, any change to a label for UI purposes will break API clients.

The core challenge is to create an API whose validation schema is derived directly and automatically from the ontology, ensuring it is always in sync and stable.

### 3. The Solution: An Ontology-Driven Schema Factory

The solution is to create a generic service that generates API validation schemas by reading the ontology at application startup.

#### 3.1. The `OntologyService`

- A singleton service (`ontology-service.ts`) will be responsible for loading all project ontologies (`.trig`, `.jsonld`) into an in-memory RDF graph.
- It will provide a method to query this graph, for example: `getTermNamesForSubclasses(classUri: string): string[]`. This method will find all subclasses of a given class (e.g., `node:Node`) and return a list of their stable **term names** (the local name of the URI, e.g., `Namespace`, `Reference`, `Data`).

#### 3.2. The `createOntologyEnumSchema` Factory

- A new utility function will take a target class URI (e.g., `node:Node`).
- It will use the `OntologyService` to get the list of valid term names.
- It will generate and return a `z.enum([...])` schema from this list.
- It will include a `.transform()` step to convert the accepted term name (e.g., `"Namespace"`) into its full, canonical URI (e.g., `https://.../ontology/node/Namespace`) for use in the backend.

#### 3.3. The API Implementation

- The `flow-service` API will replace its single hardcoded `z.enum` for `nodeType` with a call to this new factory.
- **Example:**
  ```typescript
  // Before
  nodeType: z.enum(['Namespace', 'Reference', 'Dataset'])

  // After
  nodeType: createOntologyEnumSchema(node.Node)
  ```

### 4. The Resulting API Contract

- **Stable:** The API contract is based on the stable term names from the ontology, not on mutable `rdfs:label`s.
- **Self-Documenting:** The OpenAPI specification will be automatically generated with the correct list of allowed values.
- **Maintainable:** When a new node type is added to the ontology, the API automatically supports it on the next server restart with no code changes required.
- **Consistent:** The backend logic always receives a full, unambiguous URI, ensuring clean data processing.
