---
id: z1bjxm5s2zwu07j0ndhlcpf
title: 'SF-13: Flexible, Ontology-Driven Identifier Resolution'
desc: 'Design for a generalized system to resolve user-friendly strings into formal, ontology-backed URIs at the API layer.'
updated: 1753331046536
created: 1753329521764
---

### 1. Epic

Flexible, Ontology-Driven Identifier Resolution

### 2. The Problem: Bridging Human Input and Machine-Readable Data

In a semantic system like Semantic Flow, there is a natural tension between what is easy for humans and what is best for machines.

- **Humans prefer simplicity:** When specifying something like an author, a node type, or a license, users want to type a simple, memorable string (e.g., `"DJ Meeks"`, `"Namespace"`, `"CC-BY-4.0"`).
- **Machines require precision:** To build a true, queryable knowledge graph, the system needs unambiguous, unique identifiers (URIs) for these entities. A simple string like "DJ Meeks" is ambiguous; a relative URI like `../_agents/dj-meeks/` is not.

The core challenge is to create a system that provides a simple, user-friendly interface while producing a clean, structured, and standards-compliant data graph on the backend.

### 3. Use Cases

This problem manifests in several key areas of the `flow-service` API:

- **Attribution:** A user creating a new node wants to specify the author. They might have a full URI or a relative path for a known author, but they might also just want to type a name for a new one.
  - *Input:* `"prov:wasAttributedTo": "DJ Meeks"`
  - *Desired Output:* The system resolves "DJ Meeks" to a stable, document-local URI (`./#DJ%20Meeks`) and uses that in the final data.
- **Node Type Creation:** A user wants to create a new node of a specific type.
  - *Input:* `"nodeType": "Namespace"`
  - *Desired Output:* The system validates "Namespace" against the ontology and resolves it to the canonical URI `https://semantic-flow.github.io/ontology/node/Namespace`.
- **Enumerations:** Any API property that functions like an `enum` (e.g., specifying a license, a role, or a status) should be derivable from the ontology, not from hardcoded lists in the code.

### 4. Design Journey & Key Decisions

We explored several potential solutions, ultimately rejecting ideas like parallel `_str` properties in favor of a more robust solution that keeps the final data model clean and standards-compliant. The key decision is to handle all complexity in the server's business logic, which itself is guided by the ontology.

### 5. The Final Architecture: A Generic, Context-Aware URI Resolver

The chosen solution is to create a generic, ontology-driven URI resolution service.

#### 5.1. The API Interface

The API/configuration layer will expose standard properties (e.g., `prov:wasAttributedTo`, `nodeType`) that accept a simple `string`. This provides the simplest possible user experience.

#### 5.2. The Resolver Logic

A central `UriResolver` service will process the input string. Its behavior will be determined by the format of the string itself and the context of the request.

The resolution logic is as follows:

1.  **Check if it's a full URI:** If the string is a full, absolute URI (e.g., `http://...`), use it as is.
2.  **Check if it's a known ontology term:** For properties like `nodeType`, the resolver will be given a target class (e.g., `node:Node`). It will check if the input string matches the `rdfs:label` or local name of a valid subclass. If so, it returns the canonical URI of that class.
3.  **Check if it's a relative path:** If the string is a valid relative path within the mesh (e.g., `../_agents/dj-meeks/`), it is treated as a reference to a "Mesh Entity". The resolver returns this path.
4.  **Default to "Shadow Entity":** If none of the above conditions are met, the string is treated as a "Shadow Entity". The resolver creates a document-local, relative URI by URL-encoding the string and making it a fragment identifier: `./#DJ%20Meeks`.

#### 5.3. The Data Model

The final, stored RDF data is always clean and consistent.

- The property (e.g., `prov:wasAttributedTo`) **always** contains a resolved URI (be it absolute, relative, or a fragment).
- When a "Shadow Entity" is created from a string, a new block is added to the **current snapshot's `@graph`** to define it. This ensures the definition is co-located with its use.

**Example "Shadow Entity" Definition in a Snapshot:**
```json
{
  "@id": "./#DJ%20Meeks",
  "@type": "prov:Agent",
  "rdfs:label": "DJ Meeks"
}
```

This architecture is generalized and context-aware. It correctly places "shadow" definitions within the snapshot they are relevant to, whether it's a meta-flow, data-flow, or any other flow type. It provides a powerful, predictable, and user-friendly system for identifier management.
