---
id: h5gp1pv7u8wte65pv3kzrdh
title: 2025 11 22 Shacl for 3rd Party Terminology
desc: ''
updated: 1761167887578
created: 1761167826087
---

Skip to content
Chat history
You said:
Prepare a research report on rdfjs: its ecosystem, its concepts, and best libraries for typescript-based RDF applications.
ChatGPT said:

Here’s a compact research report on RDF/JS for TypeScript apps.
What RDF/JS is

A set of low-level interfaces so JS libs interoperate on RDF terms, quads, datasets, and streams. Core specs:

    Data model: Term (NamedNode, BlankNode, Literal, Variable, DefaultGraph), Quad, DataFactory.
    rdf.js.org

    Dataset: DatasetCore for collections of quads.
    rdf.js.org

    Streams: Source, Sink, Stream for back-pressure and piping.
    rdf.js.org

    Authoritative TS types live in @rdfjs/types.
    GitHub

Ecosystem map (by task)

Parsing / serializing

    N3.js: fast Turtle/TriG/N-Triples/N-Quads parser/serializer, RDF* aware; provides an RDF/JS DataFactory.
    GitHub+1

    rdf-parse.js / rdf-serialize.js: content-type based meta-parsers/serializers that output/input RDF/JS quad streams.
    GitHub+1

    rdfxml-streaming-parser.js: streaming RDF/XML parser.
    GitHub

Query

    Comunica: modular SPARQL engine over RDF/JS sources (browser, Node, federated). Lite packages target in-process RDF/JS.
    GitHub+2rdfjs.dev+2

Traversal / graph API

    clownface: fluent graph traversal on RDF/JS datasets.
    GitHub+1

OO data model on top of RDF/JS

    RDFine: TS classes for RDF vocabularies; idiomatic objects backed by RDF/JS.
    GitHub

    LDO (Linked Data Objects): TS object view driven by ShEx shapes.
    Solid Community Forum

Validation

    rdf-validate-shacl: SHACL validation in JS on RDF/JS datasets.
    GitHub

Datasets / stores

    rdf-ext: modular RDF/JS toolkit and dataset implementation.
    rdf-ext.org+1

    quadstore: TypeScript triplestore over LevelDB/AbstractLevel; implements RDF/JS Store; SPARQL via Comunica.
    GitHub+2npm+2

Utilities

    @rdfjs/namespace (via Zazuko guides) for safe CURIE helpers; @rdfjs/types for TS.
    zazuko.com+1

Concepts you’ll use in TS code

    Factories create terms/quads: many libs expose DataFactory (e.g., N3).
    GitHub

    Datasets implement DatasetCore with add, delete, match; some add advanced ops.
    rdf.js.org

    Streams allow parse→transform→serialize pipelines with back-pressure.
    rdf.js.org

    Interoperability: any component that honors the specs plugs into the others.
    rdf.js.org

Recommended TypeScript stack (2025)

Baseline types

    @rdfjs/types for compile-time safety.
    GitHub

Read/write RDF

    n3 for Turtle/TriG/N-Triples/N-Quads in memory.

    rdf-parse + rdf-serialize for robust content-type handling and streaming.
    GitHub+1

Local store

    quadstore when you need persistence and SPARQL in-process; pairs with quadstore-comunica.
    GitHub+1

Query

    @comunica/query-sparql* packages for SPARQL over files, HTTP, or in-memory datasets.
    GitHub+1

Graph ergonomics

    clownface for traversal and graph building; @rdfjs/namespace for vocab helpers.
    GitHub+1

Domain modeling

    @tpluscode/rdfine to map vocabularies to TS classes and keep code idiomatic.
    GitHub

Validation

    rdf-validate-shacl for SHACL shapes in CI or at runtime.
    GitHub

JSON-LD

    jsonld alongside rdf-parse/rdf-serialize when consuming or producing JSON-LD. (See Ruben Taelman’s RDF/JS tool list.)
    Ruben Taelman

Minimal architectures

In-browser viewer
N3 parse → dataset (rdf-ext) → clownface traverse → render; optional Comunica for remote SPARQL.
rdfjs.dev+2GitHub+2

Node service with persistence
rdf-parse stream → quadstore .import() → validate with rdf-validate-shacl → query via Comunica → rdf-serialize to clients.
npm+4GitHub+4GitHub+4
Notes and signals

    Specs are stable and active; data-model updated Feb 1 2025; stream spec updated 2023.
    rdf.js.org+1

    The curated rdfjs.dev and rdf.js.org sites list and align libraries. Use them to discover compatible modules.
    rdf.js.org+1

    For SHACL how-to and patterns, see shacl.dev.
    Shacl

    For a practical overview of the JS tooling landscape, see Taelman’s guide.
    Ruben Taelman

If you want, I’ll draft a starter package.json and a tiny TS example wiring N3 + rdf-ext + clownface + Comunica.
You said:
what about ldkit? And how does comunica support json-ld but n3 doesn't?
ChatGPT said:

LDKit
TypeScript-first RDF framework by @zazuko.
Implements the RDF/JS interfaces but adds:

    Entity mapping: Define classes with decorators that bind RDF properties to TypeScript fields.

    Query integration: Uses Comunica under the hood for SPARQL queries.

    Fetch layer: Can dereference linked data documents and cache locally.

    JSON-LD: Built-in context handling, so you can read/write JSON-LD as plain JS objects.
    Essentially a higher-level replacement for RDFine when you want strong typing, reactive data access, and transparent serialization.

Why Comunica supports JSON-LD and N3 doesn’t
Library	Role	JSON-LD support	Reason
Comunica	Modular SPARQL engine and RDF orchestrator	✅ Yes (via @comunica/actor-init-sparql and JSON-LD parser actor)	Comunica plugs in actors for each RDF syntax, including a JSON-LD parser actor that uses the jsonld library.
N3.js	RDF* parser/serializer for line-based syntaxes	❌ No (only N-Triples, N-Quads, Turtle, TriG)	JSON-LD is not line-based and needs a full JSON processor, not an RDF triple grammar parser. N3 focuses on streaming text formats.

So:

    If you need JSON-LD I/O, use rdf-parse or Comunica’s built-in pipeline; both delegate to the jsonld package.

    Use N3.js for fast text parsing and in-memory datasets, not for JSON-LD.

Summary

    LDKit sits above RDF/JS, combining Comunica querying, JSON-LD support, and TS data models.

    Comunica’s modular design allows JSON-LD parsing through an adapter.

    N3.js intentionally omits JSON-LD because its scope is compact N-Triples/Turtle parsing, not JSON serialization.

You said:
compare rdfine and ldkit in depth.
ChatGPT said:

Here is a structured comparison of RDFine and LDKit, both TypeScript-based RDF frameworks built on top of the RDF/JS standard.
1. Purpose and Abstraction Level
Aspect	RDFine	LDKit
Primary goal	Create class-based APIs for vocabularies (OWL/RDFS/Schema.org)	Provide a high-level data access layer over RDF graphs and SPARQL endpoints
Paradigm	Object-oriented: extend RDF resources into rich TypeScript classes	Data-driven: define entities, queries, and shapes for typed Linked Data access
Typical use	Ontology modeling, data authoring, SHACL-like shape definition	Application-level graph access, remote SPARQL querying, JSON-LD integration
2. Core Concepts
Concept	RDFine	LDKit
Base unit	Resource class (TS class bound to RDF type)	Entity type (TS interface generated from schema or shape)
Binding	Decorators (@rdfine.namespace, @rdfine.property) attach RDF terms	Decorators or generated code map fields to predicates via JSON-LD context
Ontology support	Ships with curated vocab packages (schema.org, foaf, etc.)	Can import vocabularies as JSON-LD contexts or shapes
Data layer	Works directly with RDF/JS DatasetCore (no persistence)	Abstracts over local and remote sources; SPARQL and dereferencing built-in
3. Integration and I/O
Feature	RDFine	LDKit
RDF/JS integration	Tight: expects DatasetCore, Term, DataFactory	Also tight: interoperable via RDF/JS, but hides it behind entity abstractions
Querying	Manual SPARQL or dataset iteration	Built-in Comunica query engine (ldkit.query), returns typed entities
JSON-LD	None native (serialize via jsonld lib if needed)	First-class support (automatic context expansion/compaction)
SHACL / shapes	Separate library (rdf-validate-shacl)	Optional; can validate through shapes defined as part of entity model
Storage	In-memory datasets or external stores (e.g. quadstore)	Works with remote endpoints or local RDF/JS datasets; caching layer possible
4. TypeScript Experience
Feature	RDFine	LDKit
Typing model	Classes and inheritance; RdfResourceCore base class	Interfaces and schema-generated types; strong field typing
Runtime cost	Minimal—decorators and prototypes over RDF terms	Slightly higher—query execution, JSON-LD processing
IDE/DevX	Autocomplete on vocabulary classes and properties	Autocomplete on entity interfaces and query builders
Extensibility	Extend vocab classes to create custom ontologies	Extend entity definitions or plug new sources/adapters
5. Ecosystem
Area	RDFine	LDKit
Maintainers	Zazuko AG (Tomasz Pluskiewicz et al.)	Zazuko AG (also Pluskiewicz et al.)
Companion libs	clownface, @rdfjs/namespace, rdf-ext, rdfine-schema	@ldkit/query, @ldkit/core, integrates directly with Comunica
Documentation	Mature; extensive examples for vocab classes	Newer; detailed guides on entity mapping and querying
Community adoption	Used in Zazuko’s data platforms, RDF4J interop	Gaining traction for modern TS RDF apps
6. Design Trade-offs
Category	RDFine strengths	LDKit strengths
Modeling	Explicit class hierarchy matches OWL semantics; good for authoring data	Easier to consume existing JSON-LD APIs; less ontology boilerplate
Querying	Lightweight; leave query choice to you	Integrated SPARQL/query layer via Comunica
Serialization	Transparent RDF graph, but manual JSON-LD conversion	Native JSON-LD read/write and compaction
Complexity	Lower; just vocab classes	Higher; includes query engine and data management
Use case fit	Building ontologies, editors, validators, static RDF datasets	Building Linked Data clients, apps, or APIs consuming external RDF
7. Practical Recommendation
Use Case	Prefer
Ontology or data model definition (e.g., Semantic Flow, Kosmion)	RDFine — cleaner class extension and decorator syntax
Interactive Linked Data apps (fetch, query, update JSON-LD APIs)	LDKit — automatic Comunica integration and JSON-LD handling
Full-stack RDF app (TypeScript backend + SPARQL endpoint)	Combine: RDFine for ontology/vocab classes, LDKit for runtime data access
Example synthesis

You can mix them:

    Use RDFine to declare your schema (classes, properties).

    Use LDKit to query data conforming to those schemas and expose them as strongly-typed entities.

Both are 100 % RDF/JS compliant and maintained by the same ecosystem (Zazuko), so they interoperate cleanly.

Summary

    RDFine = vocabulary-centric, class-based, minimal runtime.

    LDKit = application-centric, query-driven, JSON-LD-first.
    Use RDFine when you own the ontology, LDKit when you consume or query data.

You said:
which is better (or both?) for an json-ld-based api that queries RDF data?
ChatGPT said:

For a JSON-LD–based API that queries RDF data, use LDKit as the primary layer, possibly supplemented by RDFine for vocabulary definition.
Why LDKit fits better
Criterion	Explanation
Native JSON-LD	LDKit treats JSON-LD as a first-class format: compaction/expansion, automatic contexts, and plain JS object I/O. RDFine does not handle JSON-LD directly.
Built-in querying	LDKit integrates Comunica. You can write SPARQL-like queries or entity fetches that run against remote endpoints or in-memory datasets.
API shape	It already abstracts entities as objects that serialize to JSON-LD. Perfect for REST or GraphQL-style endpoints that emit linked JSON.
Data access layer	It can dereference and cache linked documents, so clients can navigate or merge data from multiple sources.
TypeScript ergonomics	It generates entity types and decorators for compile-time safety; no need to hand-craft resource classes.
Where RDFine can still help
Use	Reason
Ontology maintenance	RDFine’s decorator syntax is concise for defining and re-using vocabularies.
Schema-driven validation	RDFine aligns neatly with SHACL and RDFS/OWL reasoning.
Integration layer	You can import RDFine vocab classes into LDKit to share term constants and IRIs.
Recommended architecture

    RDFine: define vocabularies and namespaces (e.g., schema:, foaf:).

    LDKit: use those terms to build entity models and queries.

    Comunica (via LDKit): execute SPARQL against RDF datasets or endpoints.

    JSON-LD serialization: LDKit handles expansion/compaction for API responses.

Summary:
For an RDF-querying API that speaks JSON-LD, choose LDKit as the main framework. Use RDFine only if you want to reuse its vocabulary classes or define ontologies more formally.
You said:
I'm still having trouble conceptualizing how rdfine helps. maybe an example?
ChatGPT said:

Example clarifies it best.
Problem

You have RDF data:

@prefix schema: <http://schema.org/> .
<https://example.org/alice> a schema:Person ;
    schema:name "Alice" ;
    schema:email "alice@example.org" .

You want to manipulate this in TypeScript as if it were objects, not raw triples.
Using RDFine

import { RdfResourceCore, property, namespace } from '@tpluscode/rdfine';
import { DatasetCore } from '@rdfjs/types';
import { schema } from '@tpluscode/rdf-ns-builders';
import { dataset } from '@rdf-esm/dataset';

// 1. Define a class mapped to a vocabulary
@namespace(schema)
class Person extends RdfResourceCore {
  @property.literal()
  name!: string;

  @property.literal()
  email!: string;
}

// 2. Load dataset (from parser, N3, etc.)
const data: DatasetCore = dataset`
  <https://example.org/alice> a <http://schema.org/Person>;
    <http://schema.org/name> "Alice";
    <http://schema.org/email> "alice@example.org".
`;

// 3. Wrap an RDF node as a strongly typed object
const alice = new Person(data, data.factory.namedNode('https://example.org/alice'));

// 4. Read or mutate RDF via object properties
console.log(alice.name);  // "Alice"
alice.email = 'alice@newmail.org';

// 5. Changes reflect directly in the RDF dataset
console.log([...data]); // includes the updated triple

What just happened

    RDFine wraps RDF terms and quads in class instances.

    The decorators (@property.literal, @property.resource, etc.) define how RDF predicates map to class fields.

    When you assign a value, RDFine writes triples into the dataset.

    When you read a property, it resolves the RDF value from the dataset.

    The dataset remains your canonical source of truth.

Conceptual summary
Without RDFine	With RDFine
You handle quads manually: dataset.match(subject, predicate, object)	You treat resources as objects with fields
No typing or structure	Type-safe classes derived from vocabularies
You build SPARQL queries or loops	You manipulate RDF graphs directly in JS objects
Useful for data transport	Useful for data modeling and authoring
Key idea

RDFine turns ontology terms into TypeScript classes and triples into object fields.
It doesn’t fetch or query data (LDKit does that); it models it locally in memory or in an RDF store.

So:

    RDFine = object–graph mapper for RDF data

    LDKit = data access + query + JSON-LD layer

Use RDFine when you need to structure or modify RDF data in code.
You said:
can you give an example with ldkit
ChatGPT said:

Here’s a minimal LDKit example to contrast with the RDFine one.
Goal: query and expose schema:Person data in JSON-LD form.
1. Install

npm install @ldkit/core @ldkit/query @ldkit/jsonld @comunica/query-sparql

2. Define your entity

// person.ts
import { defineEntity, field } from "@ldkit/core";
import { schema } from "@tpluscode/rdf-ns-builders";

export const Person = defineEntity({
  iri: schema.Person.value,               // RDF class
  fields: {
    name: field(schema.name),             // literal
    email: field(schema.email),           // literal
  },
});

This defines a typed interface and helper methods for any resource of type schema:Person.
3. Connect to a data source

// source.ts
import { createSource } from "@ldkit/core";
import { createComunicaQueryEngine } from "@ldkit/query-comunica";

export const source = createSource({
  endpointUrl: "https://dbpedia.org/sparql",
  queryEngine: createComunicaQueryEngine(),
});

LDKit wraps a Comunica engine to talk to any SPARQL endpoint or local RDF/JS dataset.
4. Query and consume data

// main.ts
import { Person } from "./person.js";
import { source } from "./source.js";

async function main() {
  // Query all persons with the name "Alice"
  const results = await source.query(Person, {
    where: { name: "Alice" },
    limit: 1,
  });

  // Each result is a typed Person object
  const alice = results[0];
  console.log(alice.name);  // "Alice"
  console.log(alice.email); // maybe undefined

  // Serialize to JSON-LD
  const jsonld = alice.toJSONLD();
  console.log(JSON.stringify(jsonld, null, 2));
}

main();

5. What it does

    source.query builds a SPARQL query under the hood.

    Comunica executes it.

    LDKit maps the returned RDF bindings into strongly typed Person entities.

    .toJSONLD() compacts them with an automatically generated context.

Conceptual summary
Concept	LDKit	RDFine
Focus	Data access and querying	Data modeling and in-memory manipulation
Underlying engine	Comunica (SPARQL, deref, streaming)	rdf-ext or any DatasetCore
Output	JSON-LD objects	RDF/JS DatasetCore
Direction	Consume existing RDF data	Produce or maintain RDF data
Fit for API	Ideal for JSON-LD APIs that read/query RDF	Good for APIs that generate or edit RDF graphs

Result:
With LDKit you call SPARQL endpoints and get ready-to-serve JSON-LD.
With RDFine you shape and manage RDF datasets inside your application.
You said:
can you give an example of both working together?
ChatGPT said:

Use LDKit to fetch JSON-LD, then wrap the same resource with RDFine to edit triples before responding.

// npm i @ldkit/core @ldkit/query-comunica jsonld n3 @rdfjs/dataset
// npm i @tpluscode/rdfine @tpluscode/rdf-ns-builders

import { defineEntity, field, createSource } from "@ldkit/core";
import { createComunicaQueryEngine } from "@ldkit/query-comunica";
import { schema } from "@tpluscode/rdf-ns-builders";
import * as jsonld from "jsonld";
import { Parser } from "n3";
import { dataset as makeDataset } from "@rdfjs/dataset";
import { DataFactory } from "n3"; // RDF/JS factory
import { RdfResourceCore, property, namespace } from "@tpluscode/rdfine";

// ---------- LDKit: entity + source ----------
export const Person = defineEntity({
  iri: schema.Person.value,
  fields: {
    name: field(schema.name),
    email: field(schema.email),
  },
});

const source = createSource({
  endpointUrl: "https://dbpedia.org/sparql",
  queryEngine: createComunicaQueryEngine(),
});

// ---------- RDFine: model class ----------
@namespace(schema)
class PersonR extends RdfResourceCore {
  @property.literal() name!: string;
  @property.literal() email?: string;
}

async function main() {
  // 1) Query RDF via LDKit (SPARQL through Comunica), get typed entities
  const people = await source.query(Person, { where: { name: "Alice" }, limit: 1 });
  if (!people.length) return;

  const alice = people[0];              // typed LDKit entity
  const idIri = alice.id.value;         // subject IRI (RDF/JS NamedNode-like)

  // 2) Get JSON-LD for API output if needed
  const jsonldDoc = alice.toJSONLD();

  // 3) Convert JSON-LD → quads → DatasetCore (to edit with RDFine)
  const nquads = await jsonld.toRDF(jsonldDoc, { format: "application/n-quads" }) as string;
  const quads = new Parser({ format: "N-Quads" }).parse(nquads);
  const ds = makeDataset();
  for (const q of quads) ds.add(q);

  // 4) Wrap the same resource with RDFine and mutate triples
  const personRes = new PersonR(ds, DataFactory.namedNode(idIri));
  if (!personRes.email) personRes.email = "alice@newmail.org"; // writes a triple

  // 5) Serialize back to JSON-LD for the API response
  //    Reuse the original context for stable keys
  const nquadsOut = ds.toArray().map(q => DataFactory.quad(q.subject, q.predicate, q.object, q.graph)).map(String).join("\n");
  const expanded = await jsonld.fromRDF(nquadsOut, { format: "application/n-quads" });
  const compact = await jsonld.compact(expanded, { "@vocab": schema().value });

  console.log(compact);                 // final JSON-LD with RDFine changes
}

main();

Flow:

    LDKit queries and yields typed entities and JSON-LD.

    JSON-LD is converted to an RDF/JS dataset.

    RDFine wraps the node so you edit graph data as class fields.

    Convert dataset back to JSON-LD for the API.

Rule of thumb: LDKit for querying and JSON-LD I/O. RDFine for object-style graph edits on the same data.
You said:
It seems horribly redundant to have to define the schema in LDKit and RDFind.
ChatGPT said:

You’re right: defining fields twice is waste. Pick one, or use a single source of truth.

Best path for a JSON-LD querying API

    Use LDKit only. It queries via Comunica and emits JSON-LD natively. Do mutations with LDKit + SPARQL INSERT/DELETE or RDF/JS ops. Skip RDFine.

If you still want OO editing

    Keep LDKit primary. For rare in-process edits, use clownface or direct DatasetCore writes. It avoids a second schema layer.

If you insist on LDKit + RDFine together
Use one schema source and generate the other:

    Single source = JSON-LD context + term constants

        Author a JSON-LD @context for your API.

        Import IRIs from @tpluscode/rdf-ns-builders (or @rdfjs/namespace).

        Define LDKit entities with those terms.

        Auto-generate thin RDFine mixins from the same field list.

    Tiny codegen sketch (conceptual)

        Input: an array { classIri, fields: [{key, iri, kind}] }.

        Emit LDKit defineEntity({...}).

        Emit RDFine mixin with @property.literal() / @property.resource() using the same IRIs.
        Result: zero hand-duplication, one place to change.

When to choose each

    LDKit-only: JSON-LD API, remote SPARQL, minimal moving parts.

    LDKit + RDFine: you also need rich OO manipulation of the same graph in memory.

    RDFine-only: you own the ontology and mainly author/transform RDF locally.

Bottom line: for your use case, LDKit alone is sufficient. Use a generator later if you truly need RDFine surfaces without redefining fields.
You said:
even if i only use ldkit, itd be nice to codegen the schema from the ontology
ChatGPT said:

Use SHACL (best), or RDFS/OWL (fallback) to codegen LDKit. Recommended pipeline:
Inputs → Outputs

    Input: SHACL shapes that target your classes (sh:NodeShape + sh:targetClass) and list properties (sh:property).

    Output:

        @context JSON-LD

        LDKit defineEntity(...) files

        Term constants (optional) via @tpluscode/rdf-ns-builders or @rdfjs/namespace

Why SHACL

It encodes multiplicity, datatype, and value class. That maps cleanly to TS types and LDKit fields. RDFS/OWL lacks cardinalities in practice.
Datatype map (core)

    xsd:string → string

    xsd:boolean → boolean

    xsd:integer, xsd:int, xsd:long → number

    xsd:decimal, xsd:double, xsd:float → number

    xsd:date, xsd:dateTime → string (or branded types if you enforce ISO)

    sh:class → IRI/resource (model as nested entity id or reference)

Minimal codegen sketch (TypeScript)

// npm i n3 @rdfjs/dataset
import { Parser } from 'n3';
import { dataset } from '@rdfjs/dataset';

// Config
const NS = {
  sh: 'http://www.w3.org/ns/shacl#',
  rdf: 'http://www.w3.org/1999/02/22-rdf-syntax-ns#',
  rdfs: 'http://www.w3.org/2000/01/rdf-schema#',
  xsd: 'http://www.w3.org/2001/XMLSchema#',
};

// Helpers
const q = (ds, s, p, o) =>
  [...ds.match(s, p, o)].map(q => q.object.value);

const first = a => a[0];

function tsTypeForDatatype(dtIri: string): string {
  if (!dtIri) return 'unknown';
  if (dtIri.startsWith(NS.xsd)) {
    const t = dtIri.slice(NS.xsd.length);
    if (['string'].includes(t)) return 'string';
    if (['boolean'].includes(t)) return 'boolean';
    if (['integer','int','long','decimal','double','float'].includes(t)) return 'number';
    if (['date','dateTime','time','gYear','gYearMonth'].includes(t)) return 'string';
  }
  return 'string';
}

// Core: SHACL -> LDKit entity
export async function generate(shaclTurtle: string) {
  const quads = new Parser().parse(shaclTurtle);
  const ds = dataset(quads);

  const SH_NodeShape = NS.sh + 'NodeShape';
  const SH_targetClass = NS.sh + 'targetClass';
  const RDF_type = NS.rdf + 'type';
  const SH_property = NS.sh + 'property';
  const SH_path = NS.sh + 'path';
  const SH_datatype = NS.sh + 'datatype';
  const SH_class = NS.sh + 'class';
  const SH_minCount = NS.sh + 'minCount';
  const SH_maxCount = NS.sh + 'maxCount';
  const RDFS_label = NS.rdfs + 'label';

  const shapes = [...ds.match(null, RDF_type, SH_NodeShape)].map(q => q.subject);

  const files = [];

  for (const shape of shapes) {
    const classIri = first(q(ds, shape, SH_targetClass, null));
    if (!classIri) continue;

    const props = [...ds.match(shape, SH_property, null)].map(q => q.object);
    const fields = [];

    for (const pNode of props) {
      const pathIri = first(q(ds, pNode, SH_path, null));
      if (!pathIri) continue;

      const label = first(q(ds, pNode, RDFS_label, null)) || pathIri.split(/[\/#]/).pop();
      const dt = first(q(ds, pNode, SH_datatype, null));
      const klass = first(q(ds, pNode, SH_class, null));
      const min = Number(first(q(ds, pNode, SH_minCount, null)) || '0');
      const max = Number(first(q(ds, pNode, SH_maxCount, null)) || 'NaN');

      const isArray = Number.isNaN(max) ? false : max > 1;
      const required = min > 0;

      const ldkitField =
        dt
          ? `field("${pathIri}", { type: "${tsTypeForDatatype(dt)}"${isArray ? ', array: true' : ''}${required ? ', required: true' : ''} })`
          : `field("${pathIri}", { resource: true${isArray ? ', array: true' : ''}${required ? ', required: true' : ''} })`;

      fields.push(`    ${label}: ${ldkitField},`);
    }

    const code = `import { defineEntity, field } from "@ldkit/core";
export const Entity_${safe(classIri)} = defineEntity({
  iri: "${classIri}",
  fields: {
${fields.join('\n')}
  },
});
`;
    files.push({ name: `${safe(classIri)}.ts`, code });
  }

  // Simple JSON-LD context
  const ctx = buildContextFromFields(files);
  return { files, context: ctx };

  function safe(iri: string) {
    return iri.replace(/[^A-Za-z0-9_]/g, '_');
  }
  function buildContextFromFields(files) {
    const terms = new Set<string>();
    for (const f of files) {
      for (const m of f.code.matchAll(/"([^"]+)"/g)) {
        const v = m[1];
        if (v.startsWith('http')) terms.add(v);
      }
    }
    // naive: compact by last segment
    const ctx: Record<string, string> = {};
    for (const iri of terms) {
      const key = iri.split(/[\/#]/).pop()!;
      if (!ctx[key]) ctx[key] = iri;
    }
    return { '@context': ctx };
  }
}

Example SHACL → generated LDKit

Input (excerpt):

@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <http://schema.org/> .

[] a sh:NodeShape ;
   sh:targetClass schema:Person ;
   sh:property [
     sh:path schema:name ;
     sh:datatype <http://www.w3.org/2001/XMLSchema#string> ;
     sh:minCount 1
   ] ;
   sh:property [
     sh:path schema:email ;
     sh:datatype <http://www.w3.org/2001/XMLSchema#string>
   ] .

Generated:

import { defineEntity, field } from "@ldkit/core";
export const Entity_http___schema_org_Person = defineEntity({
  iri: "http://schema.org/Person",
  fields: {
    name: field("http://schema.org/name", { type: "string", required: true }),
    email: field("http://schema.org/email", { type: "string" }),
  },
});

Context (naive compaction):

{ "@context": { "Person": "http://schema.org/Person", "name": "http://schema.org/name", "email": "http://schema.org/email" } }

If you only have RDFS/OWL

Heuristics:

    Fields = properties where rdfs:domain includes your class.

    Type = from rdfs:range (XSD → TS; class IRIs → resource: true).

    Optional if no OWL cardinality; arrays if owl:cardinality > 1 or owl:maxQualifiedCardinality > 1.

CLI outline

ldkit-codegen \
  --in shapes.ttl \
  --format turtle \
  --out ./entities \
  --emit-context ./context.jsonld \
  --prefix schema=http://schema.org/

Edge handling

    Multi-range: prefer literal over resource if any XSD present, else resource.

    Unions: emit union field or separate fields with suffixes.

    Language-tagged strings: emit { type: "string", lang: true } and decide per project.

Bottom line

Author your ontology constraints in SHACL. Run a small generator to emit LDKit entities and a JSON-LD context. Zero duplication, one source of truth.
You said:
what's the purpose of sh:property
ChatGPT said:

sh:property links a SHACL shape to one of its property constraints.
It tells validators (or generators like yours) what predicates a node of a given type is expected or allowed to have, and what rules each must follow.
Structure

Each sh:property points to a blank node (or IRI) that defines constraints on one RDF predicate.

Example:

ex:PersonShape
  a sh:NodeShape ;
  sh:targetClass schema:Person ;
  sh:property [
    sh:path schema:email ;
    sh:datatype xsd:string ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
  ] ;
  sh:property [
    sh:path schema:knows ;
    sh:class schema:Person ;
  ] .

Meaning of each part
Predicate	Role
sh:property	Declares one property constraint for the enclosing sh:NodeShape. There can be many.
sh:path	The RDF predicate (property IRI) the constraint applies to.
sh:datatype	Expected literal datatype (e.g., xsd:string, xsd:integer).
sh:class	Expected type of linked resources (for object properties).
sh:minCount / sh:maxCount	Cardinality: how many values are required or allowed.
sh:pattern / sh:in / sh:nodeKind / etc.	Additional rules: regexes, enumerations, term kinds, and so on.
Why it exists

Without sh:property, a sh:NodeShape would only say “this node must be a Person” but not what makes a Person valid.
Each sh:property adds one dimension of structural expectation:

    Validation use: check data graphs for constraint violations.

    Generation use: derive fields, datatypes, multiplicities (as in LDKit codegen).

So sh:property is the bridge between a node-level shape and the individual RDF predicates that define its structure.
You said:
maybe sh:property could be useful for specifying what 3rd-party ontology terms to use with my classes?
ChatGPT said:

Exactly. That’s a legitimate and powerful use.
Functionally

sh:property lets you declare which external predicates (from schema.org, FOAF, DCAT, etc.) apply to your own class, even if you don’t own those terms.
You’re saying, in machine-readable form:

    “Instances of my:Thing may or must use schema:name, foaf:depicts, dcat:distribution, … with these datatypes and multiplicities.”

This gives you vocabulary alignment without subclassing or ontology imports.
Example

my:DatasetShape
  a sh:NodeShape ;
  sh:targetClass my:Dataset ;
  sh:property [
    sh:path dct:title ;
    sh:datatype xsd:string ;
    sh:minCount 1
  ] ;
  sh:property [
    sh:path dcat:distribution ;
    sh:class dcat:Distribution ;
    sh:minCount 1
  ] ;
  sh:property [
    sh:path prov:wasDerivedFrom ;
    sh:class prov:Entity ;
  ] .

Interpretation:

    A my:Dataset uses DCAT and PROV predicates.

    Consumers can validate or auto-generate code knowing which external terms to expect.

Benefits
Goal	How sh:property helps
Interoperability	Explicitly lists which foreign terms your data honors.
Documentation	Machines (and humans) see which vocabularies you align with.
Codegen	Tools can pull IRIs and datatypes to build LDKit or RDFine classes automatically.
Validation	Data using other predicates can be flagged as non-conformant.
Pattern for modular ontologies

    Keep your ontology classes simple (rdfs:Class, minimal rdfs:subClassOf).

    Define SHACL shapes in a parallel namespace that declare how each class relates to external vocabularies via sh:property.

    Use shapes both for validation and for schema generation in tooling.

In short:
Yes — sh:property is ideal for recording what 3rd-party terms your class permits or expects.
It formalizes vocabulary alignment and keeps your ontology cleanly modular.
You said:
"Define SHACL shapes in a parallel namespace that declare how each class relates to external vocabularies via sh:property." -- can you give an example?
ChatGPT said:

Yes. Parallel namespaces: one for your classes, one for SHACL shapes that align third-party terms.

# ontology.ttl  (your core classes, minimal axioms)
@prefix my:   <https://example.org/ns/my/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .

my:Dataset a rdfs:Class ;
  rdfs:label "Dataset" ;
  rdfs:subClassOf dcat:Resource .   # optional alignment

my:Person a rdfs:Class ;
  rdfs:label "Person" .

# shapes.ttl  (parallel namespace for constraints + vocabulary alignment)
@prefix my:     <https://example.org/ns/my/> .
@prefix mysh:   <https://example.org/ns/my/shapes/> .
@prefix sh:     <http://www.w3.org/ns/shacl#> .
@prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
@prefix dct:    <http://purl.org/dc/terms/> .
@prefix dcat:   <http://www.w3.org/ns/dcat#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix schema: <http://schema.org/> .

# Optional: declare prefixes for tooling/UIs
mysh:Prefixes a sh:PrefixDeclaration ;
  sh:declare ( 
    [ sh:prefix "my" ;     sh:namespace "https://example.org/ns/my/" ]
    [ sh:prefix "dct" ;    sh:namespace "http://purl.org/dc/terms/" ]
    [ sh:prefix "dcat" ;   sh:namespace "http://www.w3.org/ns/dcat#" ]
    [ sh:prefix "prov" ;   sh:namespace "http://www.w3.org/ns/prov#" ]
    [ sh:prefix "schema" ; sh:namespace "http://schema.org/" ]
  ) .

# ---- Dataset shape: says which 3rd-party terms a my:Dataset uses
mysh:DatasetShape
  a sh:NodeShape ;
  sh:targetClass my:Dataset ;
  sh:closed true ;
  sh:ignoredProperties ( rdf:type ) ;

  sh:property [
    sh:path dct:title ;
    sh:datatype xsd:string ;
    sh:minCount 1 ; sh:maxCount 1
  ] ;

  sh:property [
    sh:path dct:description ;
    sh:datatype xsd:string
  ] ;

  sh:property [
    sh:path dcat:distribution ;
    sh:class dcat:Distribution ;
    sh:minCount 1
  ] ;

  sh:property [
    sh:path prov:wasDerivedFrom ;
    sh:class prov:Entity
  ] .

# ---- Person shape: aligns to schema.org terms
mysh:PersonShape
  a sh:NodeShape ;
  sh:targetClass my:Person ;
  sh:closed true ;
  sh:ignoredProperties ( rdf:type ) ;

  sh:property [
    sh:path schema:name ;
    sh:datatype xsd:string ; sh:minCount 1
  ] ;

  sh:property [
    sh:path schema:email ;
    sh:datatype xsd:string ;
    sh:pattern "^[^@\\s]+@[^@\\s]+\\.[^@\\s]+$"
  ] ;

  sh:property [
    sh:path schema:knows ;
    sh:class my:Person ;     # or schema:Person if you prefer external class
    sh:maxCount 10
  ] .

Usage:

    Data model stays in ontology.ttl.

    Alignment lives in shapes.ttl via sh:property to DCAT/DCT/PROV/Schema.org.

    Validators read shapes.ttl.

    Your LDKit codegen reads shapes.ttl to emit entities and a JSON-LD @context.

You said:
why not include the shapes in the ontology?
ChatGPT said:

You can include them, but separating them is usually cleaner.
Purpose dictates placement.
1. Different roles
Layer	Role	Typical consumer
Ontology (RDFS/OWL)	Semantic structure: classes, properties, hierarchies, domains, ranges. Declares what things mean.	Reasoners, ontology editors, inferencers
Shapes (SHACL)	Constraints and validation: what data should look like or what’s allowed.	Validators, UI forms, schema/code generators

They overlap in vocabulary but serve different kinds of logic: truth vs quality control.
2. Reasons to keep shapes separate

    Layer isolation
    Ontology remains logically consistent and open-world.
    SHACL is closed-world and prescriptive.
    Mixing them can confuse inference systems that don’t expect constraint axioms.

    Versioning and governance

        You might evolve validation rules faster than core semantics.

        External parties can use your ontology but define their own shapes.

    Multiple constraint profiles
    One ontology can have many shapesets:

        mysh:StrictShapeSet for publishing

        mysh:LooseShapeSet for prototyping
        Each reuses the same classes.

    Tool compatibility
    Some tools (e.g., TopBraid, pySHACL, LDKit codegen) treat shapes as standalone graphs.
    Keeping them in a separate file simplifies automation.

    Readability
    Ontologies stay compact. Shapes files focus on field-level alignment and UI hints.

3. When embedding shapes makes sense

    For self-documenting vocabularies that you fully control.

    When you only need one stable constraint set.

    For compact publication where simplicity outweighs modularity.

Then you’d simply put sh:NodeShape instances in the same graph, using your ontology namespace.

my:Dataset a rdfs:Class ;
  rdfs:label "Dataset" ;
  sh:property [ sh:path dct:title ; sh:datatype xsd:string ] .

But doing so means your ontology file now mixes open-world meaning and closed-world validation.
Reasoners will ignore sh: terms, but human readers and tools may misinterpret intent.
4. Best practice (modern RDF workflows)

    Keep ontologies and shapes in parallel namespaces:

    https://example.org/ns/my/
    https://example.org/ns/my/shapes/

    Link them with sh:targetClass or sh:targetNode.

    Publish both; import either as needed.

Summary
Embedding shapes is allowed but conflates logic (semantics vs constraints).
Separating them keeps reasoning pure, versioning flexible, and tooling easier.
You said:
I would argue that if you're using SHACL to express required and optional 3rd-party vocabulary, it makes sense to keep everything together.
ChatGPT said:

Agreed. If SHACL is your contract for allowed 3rd-party terms, co-locate it with the ontology. Do it cleanly:
Pattern

    One publication unit.

    Two named graphs in the same TriG (or two files in the same repo): ontology graph and shapes graph.

    Stable base IRI; shared versioning.

Minimal TriG layout

@prefix my:   <https://example.org/ns/my/> .
@prefix myv:  <https://example.org/ns/my/v1/> .
@prefix mysh: <https://example.org/ns/my/shapes/> .
@prefix dct:  <http://purl.org/dc/terms/> .
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix schema: <http://schema.org/> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

# === Ontology graph (semantics) ===
myv:ontology {
  my:Dataset a rdfs:Class ;
    rdfs:label "Dataset" ;
    rdfs:comment "Core class." ;
    dct:hasVersion "1.0.0" .
}

# === Shapes graph (contract to 3rd-party vocab) ===
myv:shapes {
  mysh:DatasetShape a sh:NodeShape ;
    sh:targetClass my:Dataset ;
    sh:closed true ;
    sh:ignoredProperties ( rdf:type ) ;

    # 3rd-party terms you allow/require:
    sh:property [ sh:path dct:title ; sh:datatype xsd:string ; sh:minCount 1 ; sh:maxCount 1 ] ;
    sh:property [ sh:path dct:description ; sh:datatype xsd:string ] ;
    sh:property [ sh:path dcat:distribution ; sh:class dcat:Distribution ; sh:minCount 1 ] ;
    sh:property [ sh:path schema:keywords ; sh:datatype xsd:string ] .

  # Helpful for tooling:
  mysh:Prefixes a sh:PrefixDeclaration ;
    sh:declare (
      [ sh:prefix "my" ;     sh:namespace "https://example.org/ns/my/" ]
      [ sh:prefix "dct" ;    sh:namespace "http://purl.org/dc/terms/" ]
      [ sh:prefix "dcat" ;   sh:namespace "http://www.w3.org/ns/dcat#" ]
      [ sh:prefix "schema" ; sh:namespace "http://schema.org/" ]
    ) .
}

Practical tips

    Single doc URL: serve this TriG; clients pick the graph they need.

    Profiles: if you need strict/loose variants later, add more shapes graphs (e.g., myv:shapesStrict, myv:shapesLoose) in the same file.

    Codegen: point your LDKit generator at the shapes graph; no duplication.

    Validation: ship a default “use this shapes graph” note in docs; validators can load the same file and select the shapes graph.

This keeps semantics and the SHACL contract together, yet logically separated, and preserves one source of truth for permitted external vocabulary.
You said:
Remember: I want to prefer json-ld everywhere because Turtle CURIES don't support trailing slashes, and a trailing slash is my solution to httprange14, i.e., non-information resources get trailing slashes.
ChatGPT said:

Understood. Prefer JSON-LD end-to-end. Use trailing slashes on non-information resources. Avoid Turtle/CURIEs.
Rules

    Mint IRIs with a trailing slash for non-info resources. No slash for documents.

    Publish ontology + shapes as a single JSON-LD doc with two graphs.

    Use @vocab or explicit terms that include the trailing slash.

    Keep LDKit entities generated from the SHACL graph. No prefixes. No CURIEs.

Single JSON-LD file (ontology + shapes)

{
  "@context": {
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "sh": "http://www.w3.org/ns/shacl#",
    "dct": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "schema": "http://schema.org/",
    "@vocab": "https://example.org/ns/my/"   // trailing slash namespace
  },

  "@graph": [
    {
      "@id": "https://example.org/ns/my/v1/ontology",   // info resource (no slash)
      "@graph": [
        {
          "@id": "Dataset/",                             // non-info resource (slash)
          "rdfs:label": "Dataset",
          "@type": "rdfs:Class"
        }
      ]
    },
    {
      "@id": "https://example.org/ns/my/v1/shapes",      // info resource (no slash)
      "@graph": [
        {
          "@id": "DatasetShape",                         // shape node
          "@type": "sh:NodeShape",
          "sh:targetClass": { "@id": "Dataset/" },
          "sh:closed": true,
          "sh:ignoredProperties": [{ "@id": "rdf:type" }],

          "sh:property": [
            {
              "sh:path": { "@id": "dct:title" },
              "sh:datatype": { "@id": "xsd:string" },
              "sh:minCount": 1,
              "sh:maxCount": 1
            },
            {
              "sh:path": { "@id": "dct:description" },
              "sh:datatype": { "@id": "xsd:string" }
            },
            {
              "sh:path": { "@id": "dcat:distribution" },
              "sh:class": { "@id": "http://www.w3.org/ns/dcat#Distribution" },
              "sh:minCount": 1
            },
            {
              "sh:path": { "@id": "schema:keywords" },
              "sh:datatype": { "@id": "xsd:string" }
            }
          ]
        }
      ]
    }
  ]
}

Data example (JSON-LD instance)

{
  "@context": {
    "dct": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "@vocab": "https://example.org/ns/my/"
  },
  "@id": "https://example.org/ns/my/datasets/abc/",   // non-info resource
  "@type": "Dataset/",
  "dct:title": "Demo",
  "dcat:distribution": { "@id": "https://example.org/ns/my/datasets/abc/dist.ttl" }  // info resource
}

LDKit codegen targets the shapes graph

    Read the JSON-LD file.

    Select @id: "…/v1/shapes" graph.

    For each sh:NodeShape:

        sh:targetClass → entity IRI (Dataset/ with slash).

        Each sh:property → field(path, { type or resource, required, array }).

    Emit entities and a @context that preserves trailing slashes in term IRIs.

Ops notes

    Use @vocab to avoid CURIEs and keep trailing slashes intact.

    Do not trim slashes in your generators or routers.

    Keep document IRIs slashless (e.g., /v1/ontology, /v1/shapes) to signal info resources.

This keeps your httprange-14 convention consistent while staying JSON-LD-native and LDKit-friendly.
You said:
https://example.org/ns/my/v1/ontology/ with a trailing slash, refers to the ontology. I equate info resource with "file", so it would usually have a file extension.
ChatGPT said:

Understood. Then use this mapping:

    Non-information resources (things): trailing slash.
    Example: https://example.org/ns/my/Dataset/

    Information resources (files/documents): no trailing slash, explicit extension.
    Examples:

        Ontology doc: https://example.org/ns/my/v1/ontology.jsonld

        Shapes doc: https://example.org/ns/my/v1/shapes.jsonld

        Instance distribution: https://example.org/ns/my/datasets/abc/dist.trig

Corrected JSON-LD layout

Single publishable file that embeds both graphs:
https://example.org/ns/my/v1/package.jsonld

{
  "@context": {
    "@vocab": "https://example.org/ns/my/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "sh": "http://www.w3.org/ns/shacl#",
    "dct": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "schema": "http://schema.org/"
  },
  "@graph": [
    {
      "@id": "https://example.org/ns/my/v1/ontology.jsonld",
      "@graph": [
        { "@id": "Dataset/", "@type": "rdfs:Class", "rdfs:label": "Dataset" }
      ]
    },
    {
      "@id": "https://example.org/ns/my/v1/shapes.jsonld",
      "@graph": [
        {
          "@id": "DatasetShape",
          "@type": "sh:NodeShape",
          "sh:targetClass": { "@id": "Dataset/" },
          "sh:closed": true,
          "sh:ignoredProperties": [{ "@id": "rdf:type" }],
          "sh:property": [
            { "sh:path": { "@id": "dct:title" }, "sh:datatype": { "@id": "xsd:string" }, "sh:minCount": 1, "sh:maxCount": 1 },
            { "sh:path": { "@id": "dcat:distribution" }, "sh:class": { "@id": "http://www.w3.org/ns/dcat#Distribution" }, "sh:minCount": 1 }
          ]
        }
      ]
    }
  ]
}

Instance data example

{
  "@context": {
    "@vocab": "https://example.org/ns/my/",
    "dct": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#"
  },
  "@id": "https://example.org/ns/my/datasets/abc/",
  "@type": "Dataset/",
  "dct:title": "Demo",
  "dcat:distribution": { "@id": "https://example.org/ns/my/datasets/abc/dist.trig" }
}

LDKit codegen targets

    Read package.jsonld.

    Select graph …/shapes.jsonld.

    Generate entities with class IRIs like Dataset/.

    Preserve slashes; never compact to CURIEs.

    Serve ontology at …/ontology.jsonld and shapes at …/shapes.jsonld if clients fetch separately.

You said:
Interesting that you've used a "v1/" folder for the ontology, but the Dataset is specified without a version. 

Is the rationale for keeping shapes in a separate graph but within the same file just that LDKit can target the named graph? Can the ontology be in the default graph? I can't tell if that's the way you specified it, or if that is even sensible in json-ld
ChatGPT said:

Short answers:

    Unversioned class IRIs, versioned documents: Keep resource IRIs stable (…/Dataset/). Version the documents that describe them (…/v1/ontology.jsonld, …/v1/shapes.jsonld). This avoids IRI churn while letting you publish new docs.

    Why a separate graph for shapes in one file: Makes selection trivial for tooling (e.g., LDKit/codegen or validators) and supports multiple profiles in the same file. You can point tools at the graph IRI and ignore everything else.

    Can the ontology live in the default graph? Yes. In JSON-LD, top-level nodes (or a top-level @graph) are the default graph. Named graphs are nodes that contain their own @graph and an @id. It’s sensible to put ontology in the default graph and shapes in a named graph.

Minimal pattern with ontology in default graph and shapes in a named graph, single file …/v1/package.jsonld:

{
  "@context": {
    "@vocab": "https://example.org/ns/my/",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "dct": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#"
  },

  /* Default graph = ontology */
  "@graph": [
    { "@id": "Dataset/", "@type": "rdfs:Class", "rdfs:label": "Dataset" },

    /* Named graph = shapes */
    {
      "@id": "https://example.org/ns/my/v1/shapes.jsonld",
      "@graph": [
        {
          "@id": "DatasetShape",
          "@type": "sh:NodeShape",
          "sh:targetClass": { "@id": "Dataset/" },
          "sh:closed": true,
          "sh:property": [
            { "sh:path": { "@id": "dct:title" }, "sh:datatype": { "@id": "xsd:string" }, "sh:minCount": 1, "sh:maxCount": 1 },
            { "sh:path": { "@id": "dcat:distribution" }, "sh:class": { "@id": "dcat:Distribution" }, "sh:minCount": 1 }
          ]
        }
      ]
    }
  ]
}

Tooling notes:

    Validators or generators can select by graph IRI (…/v1/shapes.jsonld) or simply scan for sh:NodeShape regardless of graph.

    If you later need multiple profiles, add more named graphs (…/v1/shapes-loose.jsonld, …/v1/shapes-strict.jsonld) in the same file.

    Keep non-information resources with trailing slash; keep documents versioned and with file extensions.


ChatGPT can make mistakes. Check important info.
