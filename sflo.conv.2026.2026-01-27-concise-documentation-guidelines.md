---
id: 2bs5vxykg8j7fm315ilem0k
title: 2026 01 27 Concise Documentation Guidelines
desc: ''
updated: 1769583229633
created: 1769583227324
---
Skip to content
Chat history
You said:
I'm creating a task prompt for the documentation overhaul. Help me communicate the importance of not repeating yourself, and having extremely tight prose that is as succinct as possible.
ChatGPT said:

Use wording like this in your task prompt. It’s framed as non-negotiable acceptance criteria and gives concrete tests that prevent “tight prose” from turning into vague hand-waving.
Prose constraints (include verbatim)

Goal: Documentation must be ruthlessly concise and non-repetitive. Prefer one clear statement over layered restatements.

Non-repetition rules

    Do not restate the same idea in different words (no “echo paragraphs,” no “summary after the summary”).

    Never define a concept twice. If it must appear in multiple places: define once, then link/reference it.

    If two sentences could be merged without losing meaning, merge them.

    If a sentence can be deleted without losing essential information, delete it.

Tight prose rules

    Every paragraph must justify its existence: each paragraph should introduce exactly one new idea or decision.

    Prefer concrete nouns/verbs over qualifiers (“very,” “generally,” “typically,” “basically,” “in order to,” “it should be noted”).

    Avoid hedging unless there is real uncertainty and it matters.

    Use active voice unless passive is more precise.

Information placement

    State the rule/decision first. Put rationale only if it changes how someone uses the system.

    Put exceptions immediately after the rule, not later.

    Prefer bullets over prose when listing constraints, steps, or criteria.

Acceptance tests (hard gates)

    No paragraph may repeat information from any earlier paragraph in the same page.

    No section may re-explain a term already defined elsewhere; it must link to the canonical definition.

    Each sentence must pass a “delete test”: if removing it doesn’t change user decisions or actions, it must be removed.

    Each page must have a “Why this page exists” check: if the page doesn’t change behavior, remove or merge it.

Add this “editor mindset” note (short but sharp)

Write like a spec, not a lecture. Assume the reader is busy and competent. If you feel tempted to clarify by repeating, instead improve the original sentence.
Things to watch for (call these out explicitly)

    Repeating the same definition in “Overview,” then again in “Details,” then again in “FAQ.”

    Intro paragraphs that say what the section will say.

    “In other words…” and “to be clear…” that paper over fuzzy writing rather than fixing it.

    Multiple near-synonyms for the same concept (pick one term; enforce it).

If you paste your current task prompt, I can tighten it and remove duplication aggressively (including pointing out specific repeated ideas and where a single canonical definition should live).
You said:
I'm going to have "class" pages that shadow the ontology class structure. Should I have property pages too?
ChatGPT said:

Yes, unless your ontology is so small or so single-domain that properties are essentially “owned” by one class. In most real ontologies, property pages are the only way to stay consistent and avoid repetition.
When property pages are worth it

Create property pages if any of these are true:

    A property is used by more than one class (even if the formal rdfs:domain is broad/union/omitted).

    You have more than ~20–30 properties total.

    You have “framework” properties (your equivalent of identity, containment, versioning, provenance, naming, etc.) that show up everywhere.

    You care about tight prose: property pages let you define once and link everywhere.

When property pages are not worth it

Skip property pages (or only create a handful) if:

    Nearly every property is effectively class-specific and will never be reused.

    You’re not willing to enforce “define once, link elsewhere.” If you’ll end up re-explaining each property on each class page, you get the worst of both worlds.

The critical risk: duplication and inconsistency

If you add property pages, you must treat them as the canonical place where meaning lives. Otherwise you’ll drift:

    Class page says “X means …”

    Property page says “X means … (slightly different)”

    Examples contradict constraints

That will happen unless you set a hard rule:

Rule: Class pages never define properties. They only list properties and link to the property page.
A clean division of responsibility

    Class pages: what the thing is, how it fits the hierarchy, and which properties are typically used with it (as links).

    Property pages: what the relationship/attribute means, constraints, and usage patterns.

Minimal property page template (keeps prose tight)

Keep each property page short and structured:

    Definition (1 sentence)

    Type (ObjectProperty / DatatypeProperty / AnnotationProperty)

    Expected subject/object (domain/range if asserted; otherwise “typical usage”)

    Constraints (cardinality rules, inverse/subproperty notes)

    Usage note (1–3 bullets max)

    Example (one small snippet)

How class pages “shadow” properties without repeating

On each class page, include a compact “Common properties” table:

    Property (linked)

    Role (5–10 words)

    Cardinality (if relevant)

    Notes (optional, but don’t restate semantics; use “See property page”)

Recommendation

Implement property pages for all public/stable properties, generated from the ontology with a strict template. For internal/rare properties, either:

    keep them undocumented (if truly internal), or

    document them only on the property page (still canonical), but exclude them from class-page “Common properties.”

If you already know you’ll have “framework” properties reused across many classes (which Semantic Flow clearly does), not having property pages will force repetition or ambiguity.
You said:
Right now I have "facet" pages. 
Should I get rid of them?
ChatGPT said:

Do not keep them as prose pages.

They conflict directly with your “no repetition / tight prose” requirement, because they create a second place to explain the same concepts that will also appear on class pages (and eventually property pages). That guarantees drift.

Keep them only if you treat them as pure navigation indices.
Recommended rule

    Canonical meaning lives only on class pages and property pages.

    “Facet” pages, if they exist, contain:

        a one-sentence scope statement (optional), and

        a generated list of links (no definitions, no examples, no constraints).

If you can’t enforce that, delete the facet pages entirely.
What your current list suggests (problem)

Files like facet.flow.versioned.md, facet.flow.unversioned.md, facet.flow.v-series.md look like they’re actually class-level distinctions (or close enough) and will compete with the ontology shadow pages. That’s the worst duplication risk: same idea, different doc entity.
Minimal migration

    For each facet.* page, decide: is this an ontology term (class/property) or just a documentation grouping?

    If it’s an ontology term → it becomes a class/property page; the facet page is removed or becomes a redirect/stub link.

    If it’s just a grouping → keep it, but make it an index-only page (generated from tags/relations), with zero semantic prose.

Net: you either delete facets or demote them to link hubs. Anything in between will produce repetition and inconsistencies.
You said:
Facet pages are more like cross-cutting concerns. Some that tie directly to classes can definitely be eliminated. Some are like "naming-resource", ie., a non-File resource which isn't explicitly represented in the ontology, but is an important part of the conceptualization. I actually used to have a NonArtifact class (NonFile would be better now). Or the facet.system vs facet.user distinction, for "things that can be updated by users/clients". (That one was always a little fuzzy, but a updated juxtaposition could be Artifact vs Supporting). An important facet  is Filesystem vs Semantic. But those could all be concepts. 

Another question, originally my list of new files contained hierarchy in each filename, e.g. c.Realization.AbstractFile.AbstractArtifactFile. (due to multi-inheritance, you just choose the most salient parent class). It's useful contextual clues, but also clutters a bit.
ChatGPT said:

Do not delete “facet” pages wholesale. Delete them as canonical prose.

Right now you’re using “facet” to cover three different things, and that is why it feels fuzzy:

    Cross-cutting doc views (indexing/grouping)

    Missing-but-important concepts (your “naming-resource”)

    Policy/authority classifications (“system vs user”, “user-updatable”)

Those cannot share one documentation primitive without creating duplication and drift.
A workable split
A) Facet pages as generated “views” only

Keep facet.* pages only as link hubs that answer: “show me everything related to X”.
They should contain almost no prose. If they become explanatory, they will compete with class/property pages.

    Allowed content: 1-sentence scope, then links (classes, properties, examples)

    Not allowed: definitions, constraints, examples that aren’t elsewhere

B) Anything concept-like should become a concept

If “naming-resource”, “filesystem vs semantic”, “artifact vs supporting” are truly part of the conceptual model, treat them as first-class terms, not facets.

Options:

    Class (if instances exist)

    SKOS Concept (if this is primarily a classification axis)

    Annotation/property (if it’s metadata about other terms, e.g., isUserUpdatable)

If you refuse to model them, you’re implicitly accepting that they are editorial categories, not semantics—and then they must stay as “view-only” facets with minimal prose.
C) Policy dimensions need explicit criteria or they’re unusable

“facet.system vs facet.user” is currently not a concept, it’s an intuition. That will break the doc overhaul because nobody can apply it consistently.

If you keep it, you need a crisp test like:

    “User-updatable means: can be authored/modified by a client without violating mesh integrity” (and list the exception rules), or

    drop it in favor of a clearer axis (your “Artifact vs Supporting” is closer, but still needs a test: generated vs source? payload vs UI asset?).

If you can’t write a 2–3 bullet decision procedure, it’s not a facet; it’s a mood.
Filesystem vs Semantic

This is a good example of a real axis, but it should not be implicit:

    either model it as two top-level groupings (classes or SKOS concepts),

    or treat it as a doc taxonomy tag that you can compute (e.g., “isLocatedFile” implies filesystem-side).

Filename hierarchy in filenames (e.g., c.Realization.AbstractFile.AbstractArtifactFile)

This is useful as a human clue, but it’s also a duplication of ontology structure and it fails under multi-inheritance (your “choose the most salient parent” is subjective → drift).

Recommendation:

    Keep filenames stable and short: one file per term, named by localname/slug.

    Put hierarchy in generated navigation (breadcrumbs, “subclasses”, “superclasses”, “see also”) derived from the ontology.

    If you still want the “salient parent” signal, store it as frontmatter, not in the filename (so you can change it without renaming files):

        canonicalParent: Realization (or docParent:)

        then the site generator uses it for the sidebar, while the page still shows the actual multiple parents.

This preserves the contextual hint without encoding a fragile, opinionated tree into the file system.

If you want, paste one “facet” page you consider essential (e.g., facet.naming-resource.md) and I’ll tell you whether it should be (1) converted into a modeled concept, (2) reduced to a view-only index, or (3) deleted because it duplicates existing class/property pages.
You said:
Does the word denote have a connotation of indirection? Like a LocatedFile doesn't really denote itself? I'd guess no.

Here's an example facet file (needs updating):

---
id: external-facet
title: external facet
desc: ''
updated: 1756305833518
created: 1756305690271
---

[[concept.identifier]]s and [[concept.referent]] with the **external facet** denote things "outside the mesh", whether real or imaginary.
ChatGPT said:

“Denote” does not inherently mean “indirect.” It just names the symbol → referent relation.

Two important consequences:

    It presupposes two distinct relata.
    If you say “a LocatedFile denotes itself,” you’ve collapsed sign and referent. The IRI (or handle) denotes the LocatedFile resource; the LocatedFile resource does not “denote” anything by default.

    In RDF-land specifically, “denote” is close to the formal semantics: IRIs denote resources. That’s direct reference, not an extra indirection layer.

So: your instinct “no” is mostly right, but you’re brushing past the bigger issue: you’re currently tempted to use “denote” between things that are not denoters.
Your facet example has a category error

    “identifiers and referents with the external facet denote things…”

    Identifiers can denote.

    Referents do not denote (they are what’s denoted).
    Unless you mean “resources that stand in for referents,” but then they aren’t referents; they’re representations.

Also, “outside the mesh” is underspecified. Outside which mesh? Outside the filesystem? Outside the namespace? Outside the governance of Semantic Flow? Those are different, and you will get inconsistent tagging if you don’t pick one.
Tight replacement text (still one sentence, but coherent)

Pick one meaning for “external.” Here are two good options; choose one and make everything follow it.

Option A: external = non-SF referent (governance boundary)

    The external facet marks identifiers whose referent is not a Semantic Flow resource (it lies outside the mesh’s conventions/governance, whether real or fictional).

Option B: external = not contained in this mesh (containment boundary)

    The external facet marks identifiers whose referent is not contained in this mesh (it is only referenced, not hosted here).

Do not mix A and B; they diverge immediately in edge cases (e.g., a person with a full page in your mesh).
Structural recommendation for this facet

If “external/internal” is something you’ll rely on repeatedly, it shouldn’t live only as prose:

    Model it as a classification axis (e.g., sflo:referentScope with values sflo:External / sflo:Internal, or similar).

    Then your facet.external.md becomes a view page (index of everything tagged External) with at most the 1-sentence canonical definition (which itself ideally lives on the concept/property page).

If you keep it purely as a facet with prose, it will drift as soon as you add “Filesystem vs Semantic” and “Artifact vs Supporting.”
One more pitfall to fix now

“denote things outside the mesh” implies the mesh is the domain of existence. But your system explicitly allows minting IRIs for things not (yet) present in the mesh. So if you mean “not present,” say “not contained/hosted”; if you mean “not an SF resource,” say that. “Outside the mesh” is too mushy.

If you paste how you currently apply the “external” facet (what gets it, what doesn’t), I can tell you which option you’re actually using and where your existing usage contradicts itself.
You said:
O
You said:
These should have corresponding properties with domain and range:

# Inventories


<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <MeshComponent/>, <SimpleArtifact/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <KnopComponent/>, <SimpleArtifact/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <NomenComponent/>, <SimpleArtifact/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .
ChatGPT said:

# Inventory linkages (object properties)

<hasInventoryDataset/> a owl:ObjectProperty ;
  rdfs:label "has inventory dataset" ;
  rdfs:comment "Generic link from a host resource (mesh/knop/nomen) to one or more derived inventory datasets." .

<hasMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasInventoryDataset/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "has mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<meshInventoryDatasetFor/> a owl:ObjectProperty ;
  owl:inverseOf <hasMeshInventoryDataset/> ;
  rdfs:domain <MeshInventoryDataset/> ;
  rdfs:range <Mesh/> ;
  rdfs:label "mesh inventory dataset for" ;
  rdfs:comment "Inverse of hasMeshInventoryDataset." .

<hasKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasInventoryDataset/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "has knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<knopInventoryDatasetFor/> a owl:ObjectProperty ;
  owl:inverseOf <hasKnopInventoryDataset/> ;
  rdfs:domain <KnopInventoryDataset/> ;
  rdfs:range <Knop/> ;
  rdfs:label "knop inventory dataset for" ;
  rdfs:comment "Inverse of hasKnopInventoryDataset." .

<hasNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasInventoryDataset/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "has nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .

<nomenInventoryDatasetFor/> a owl:ObjectProperty ;
  owl:inverseOf <hasNomenInventoryDataset/> ;
  rdfs:domain <NomenInventoryDataset/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "nomen inventory dataset for" ;
  rdfs:comment "Inverse of hasNomenInventoryDataset." .

Critical notes you should resolve now (or this will drift):

    If you don’t already have <Mesh/>, <Knop/>, and <Nomen/> as the hosts, these domain axioms will be wrong. Don’t “guess-host” based on <MeshComponent/>—decide what the host class is and use that consistently.

    Don’t make these properties functional: your own comments say “may be sharded,” which implies multiple values are legitimate.

You said:
I think they should be, e.g. containsMeshInventoryDataset
ChatGPT said:

# Inventory containment properties (object properties)

<containsInventoryDataset/> a owl:ObjectProperty ;
  rdfs:label "contains inventory dataset" ;
  rdfs:comment "Generic containment link from a container resource (mesh/knop/nomen) to one or more derived inventory datasets." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsInventoryDataset/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Containment: links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<meshInventoryDatasetContainedIn/> a owl:ObjectProperty ;
  owl:inverseOf <containsMeshInventoryDataset/> ;
  rdfs:domain <MeshInventoryDataset/> ;
  rdfs:range <Mesh/> ;
  rdfs:label "mesh inventory dataset contained in" ;
  rdfs:comment "Inverse of containsMeshInventoryDataset." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsInventoryDataset/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Containment: links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<knopInventoryDatasetContainedIn/> a owl:ObjectProperty ;
  owl:inverseOf <containsKnopInventoryDataset/> ;
  rdfs:domain <KnopInventoryDataset/> ;
  rdfs:range <Knop/> ;
  rdfs:label "knop inventory dataset contained in" ;
  rdfs:comment "Inverse of containsKnopInventoryDataset." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsInventoryDataset/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Containment: links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .

<nomenInventoryDatasetContainedIn/> a owl:ObjectProperty ;
  owl:inverseOf <containsNomenInventoryDataset/> ;
  rdfs:domain <NomenInventoryDataset/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "nomen inventory dataset contained in" ;
  rdfs:comment "Inverse of containsNomenInventoryDataset." .

Two consistency traps to watch:

    If you already use containsMeshResource / containsMeshComponent, decide whether these inventory datasets are “resources” or “components” and make containsMeshInventoryDataset a subproperty of the correct one; otherwise your containment lattice will fragment.

    “contains” implies the dataset is in the container’s scope (hosted/served there). If you sometimes point to an inventory generated elsewhere, this property name will become incorrect—use a separate referencesInventoryDataset for that case rather than weakening “contains.”

You said:
Any feedback on the latest ontology?

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <https://schema.org/> .
@prefix vann: <http://purl.org/vocab/vann/> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .


# Core Semantic Mesh-scoped resources

<MeshResource/> a rdfs:Class ;
  rdfs:label "Mesh resource" ;
  rdfs:comment "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves" .

<ArtifactHost/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A mesh-governed resource that can contain artifacts and associated structural resources. Nomen are ArtifactHosts that can also contain other Nomen. " .

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains mesh-governed resources and, optionally, other supporting resources." .

<meshBase/> a owl:DatatypeProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "Mesh Base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

<MeshComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Mesh component" ;
  rdfs:comment "A reserved, mesh-scoped structural resource (e.g., mesh-level metadata, mesh inventory)." .  

<containsMeshComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshComponent/> ;
  rdfs:label "contains mesh component" ;
  rdfs:comment "Associates a Mesh with one of its directly contained MeshComponents." .

<containsKnopComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopComponent/> ;
  rdfs:label "contains knop component" ;
  rdfs:comment "Associates a Knop with one of its directly contained KnopComponents." .

<containsNomenComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenComponent/> ;
  rdfs:label "contains nomen component" ;
  rdfs:comment "Associates a Nomen with one of its directly contained NomenComponents." .

  
# General-purpose Resources
# These could be usable anywhere on the Semantic Web, not just in Semantic Flow meshes'

## Digital Artifact

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A mesh-governed artifact identity (something treated as a digital artifact in the abstract)." .

## Realizable + Realizations

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "The essential, particular (at a point in time) content of a DigitalArtifact (e.g., an image, an ontology, a text file) that can be realized via one or more Realizations (e.g., files, streams)." .

<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding/packaging of a Realizable DigitalArtifact, independent of any particular access endpoint." .


## Abstract files and streams

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-form realization specification (format/syntax/canonicalization/bundling). May be mesh-governed or external." .

<AbstractStream/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract stream" ;
  rdfs:comment "A non-file realization specification (e.g., a stream or service-like packaging), reserved for future use." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractFiles." .

<hasAbstractStream/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractStream/> ;
  rdfs:label "has abstract stream" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractStreams." .

## Located files (byte retrieval)

<LocatedFile/> a rdfs:Class ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes endpoint (typically with an extension), which may be in-mesh or external." .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  owl:inverseOf <locatesAbstractFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile/> ;
  rdfs:range <AbstractFile/> ;
  owl:inverseOf <hasLocatedFile/> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .


## Resource Pages

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "A derived UI/navigation file (e.g., index.html) that presents a mesh-governed resource (or other referent) to humans." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource (including referents) with a human-readable presentation page." .

# Semantic Flow Artifacts

## Digital Artifacts kinds

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A mesh-managed artifact identity that organizes a sequence of revisions (represented as Slices)." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A mesh-governed artifact identity treated as a single content unit (no revision history)." .

<artifactKind/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range rdfs:Class ;
  rdfs:label "artifact kind" ;
  rdfs:comment "Declares what kind of artifact this DigitalArtifact carries (e.g., RdfDataset)." .

## Artifact Files

<LocatedArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/> ;
  rdfs:label "Located artifact file" ;
  rdfs:comment "A mesh-governed LocatedFile that provides concrete bytes for a mesh-governed abstract artifact file." .

<AbstractArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <AbstractFile/>, <MeshResource/> ;
  rdfs:label "Abstract artifact file" ;
  rdfs:comment "A mesh-governed AbstractFile that participates in Semantic Flow's artifact modeling." .

<hasAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasAbstractFile/> ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractArtifactFile/> ;
  rdfs:label "has abstract artifact file" ;
  rdfs:comment "Links a Realizable resource to one of its mesh-governed AbstractArtifactFiles." .

<hasLocatedArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasLocatedFile/> ;
  rdfs:domain <AbstractArtifactFile/> ;
  rdfs:range <LocatedArtifactFile/> ;
  owl:inverseOf <locatesAbstractArtifactFile/> ;
  rdfs:label "has located artifact file" ;
  rdfs:comment "Links an AbstractArtifactFile to one or more mesh-governed LocatedArtifactFiles." .

<locatesAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <locatesAbstractFile/> ;
  rdfs:domain <LocatedArtifactFile/> ;
  rdfs:range <AbstractArtifactFile/> ;
  owl:inverseOf <hasLocatedArtifactFile/> ;
  rdfs:label "locates abstract artifact file" ;
  rdfs:comment "Navigation: links a LocatedArtifactFile to the AbstractArtifactFile whose bytes it provides." .


# Slices

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A particular revision member of a FlowArtifact. A Slice is content-bearing/realizable but is not a DigitalArtifact identity." .

<WorkingSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Working slice" ;
  rdfs:comment "Mutable current state for a FlowArtifact." .

<HistoricalSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Historical slice" ;
  rdfs:comment "Immutable published snapshot for a FlowArtifact." .

<hasSlice/> a owl:ObjectProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <Slice/> ;
  rdfs:label "has slice" ;
  rdfs:comment "Associates a FlowArtifact with one of its Slices." .

<currentSlice/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasSlice/> ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <WorkingSlice/> ;
  rdfs:label "current slice" ;
  rdfs:comment "Associates a FlowArtifact with its current Slice." .

<previousSlice/> a owl:ObjectProperty ;
  rdfs:domain <Slice/> ;
  rdfs:range <Slice/> ;
  rdfs:label "previous slice" ;
  rdfs:comment "Links to the immediately-preceding Slice of this Slice." .

<nextSliceSequenceNumber/> a owl:DatatypeProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range xsd:integer ;
  rdfs:label "next slice sequence number" ;
  rdfs:comment "Monotonic integer providing the next allocatable sequence number for a FlowArtifact." .


# Naming

<Nomen/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a meshPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about. A Nomen may also function as a namespace prefix for subordinate Nomens under its path." .

<hasNomen/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Nomen/> ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:label "has nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<NomenComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Nomen component" ;
  rdfs:comment "Reserved components under a Nomen (e.g., nomen flows, inventories)." .

<nomenFor/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "nomen for" ;
  rdfs:comment "Asserted designator IRI for the Nomen. Usually matches the meshPath." .

<meshPath/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:string ;
  rdfs:label "mesh path" ;
  rdfs:comment "Mesh-relative path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root (i.e., the base itself)." .

<designates/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for namespacing / token management)." .

<hasChildNomen/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "has child nomen" ;
  rdfs:comment "Operational/derived listing of subordinate Nomens under this Nomen's namespace path. Not a source of truth; implementations may derive children from mesh structure and/or meshPath prefix rules." .


### Reference data links

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a MeshResource to a specific reference data source, including role metadata." .

<ReferenceRole/> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical/> a <ReferenceRole/> ;
  rdfs:label "Canonical" .

<ReferenceRole/Integrate/> a <ReferenceRole/> ;
  rdfs:label "Integrate into resource page" .

<ReferenceRole/Supplemental/> a <ReferenceRole/> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated/> a <ReferenceRole/> ;
  rdfs:label "Deprecated" .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <ReferenceLink/> ;
  rdfs:label "has reference link" ;
  rdfs:comment "Links an ArtifactHost (Mesh, Nomen, Knop) to a ReferenceLink describing a specific reference-data source." .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <MeshResource/> ;
  rdfs:range <ReferenceLink/> ;
  owl:inverseOf <referenceLinkFor/> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a mesh-governed resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <MeshResource/> ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the mesh-governed resource that a ReferenceLink is about." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a node used as a reference-data identifier (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

<referenceUriLiteral/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

### Verification metadata

<lastVerifiedAt/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .


# Knops + flows

<Knop/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for DigitalArtifacts anchored at a stable in-mesh location; hosts payload and supporting FlowArtifacts and SimpleArtifacts." .

<hasKnop/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Knop/> ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:label "has knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<KnopComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Knop component" ;
  rdfs:comment "Reserved components under a Knop (payload/supporting flows and their parts)." .

<FlowPayload/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload flow" ;
  rdfs:comment "The primary FlowArtifact hosted by a Knop." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

<hasPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "contains payload" ;
  rdfs:comment "Links a Knop to its primary payload artifact identity (either a FlowArtifact-based payload or a SimpleArtifact-based payload)." .

<containsFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "contains flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<containsSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "contains simple payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a SimpleArtifact (SimplePayload)." .

# Metadata

<KnopMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/> ;
  rdfs:label "Knop metadata flow" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop." .

<MeshMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <MeshComponent/> ;
  rdfs:label "Mesh metadata flow" ;
  rdfs:comment "Optional mesh-level operational metadata/config." .

<NomenMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <NomenComponent/> ;
  rdfs:label "Nomen metadata flow" ;
  rdfs:comment "Metadata about a Nomen; may remain WorkingSlice-only until history is recorded." .

<containsKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "contains knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<containsMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "contains mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<containsNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "contains nomen metadata flow" ;
  rdfs:comment "Links a Nomen to its nomen metadata flow (if present)." .


# RDF datasets 

<RdfDataset/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, dcat:Dataset ;
  rdfs:label "RDF dataset" ;
  rdfs:comment "A SimpleArtifact whose content is an RDF dataset (default graph + 0..n named graphs)." .


# Inventories

<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <MeshComponent/>, <SimpleArtifact/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <KnopComponent/>, <SimpleArtifact/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <NomenComponent/>, <SimpleArtifact/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .
ChatGPT said:

High-value feedback, in descending order of “this will bite you later.”
1) You have at least one hard ontology bug: duplicated property IRI

<hasReferenceLink/> is declared twice with different domains (ArtifactHost vs MeshResource) and only one declaration wires up the inverse. In Turtle this collapses into one property with both domain axioms, which likely is not what you mean.

Fix: keep one declaration, then specialize if needed.

    If you want ReferenceLinks attachable to any mesh resource:

        hasReferenceLink domain <MeshResource/>

    If you want “stored under host” separate from “about target”:

        containsReferenceLink (domain <ArtifactHost/>) for storage/containment

        referenceLinkFor (range rdfs:Resource or <MeshResource/>) for “aboutness”

Right now you’re mixing those two roles.
2) “General-purpose resources” section contradicts your axioms

You say those resources “could be usable anywhere on the Semantic Web,” but:

    <DigitalArtifact/> rdfs:subClassOf <MeshResource/> makes it explicitly mesh-governed.

Either:

    drop the “general-purpose” claim and lean into “mesh-governed,” or

    remove the <MeshResource/> superclass and introduce a mesh-specific subclass (e.g., <MeshDigitalArtifact/>).

Same issue (softer) with other “may be external” concepts—your commentary and your class lattice need to agree.
3) Realizable vs DigitalArtifact semantics are currently unstable

<Realizable/> is described as “essential content,” while <DigitalArtifact/> is the abstract artifact identity.

But you then make:

    <SimpleArtifact/> rdfs:subClassOf <DigitalArtifact/>, <Realizable/>

That implies the identity is the content. That can be coherent, but only if you state it explicitly. Right now it reads like an accidental conflation.

Two clean options:

    Option A (identity/content split): SimpleArtifact is only a DigitalArtifact; it has a Realizable (or a Slice-like content node). Add hasRealizable / contentOf.

    Option B (identity = content for SimpleArtifact): keep the axiom, but change the Realizable/SimpleArtifact wording so it’s not contradicting itself.

4) “Avoid denotation” policy is inconsistently applied

You correctly use a URI literal for meshBase to avoid asserting a denoted resource. But then:

    <nomenFor/> “Asserted designator IRI” uses rdfs:range rdfs:Resource.

That does assert denotation (and also it’s not really what you want: “the designator IRI” is not the designatum).

If you want “designator IRI” without denotation, make it a datatype property:

    designatorUriLiteral (domain <Nomen/>, range xsd:anyURI)

…and reserve designates for the designatum.
5) You have parallel containment vocabularies that aren’t aligned

You use:

    containsMeshResource (ArtifactHost → MeshResource)

    hasKnop / hasNomen as dcterms:hasPart (Mesh → Knop/Nomen) with “semantic containment/inventory” wording

These are the same idea expressed twice, but they’re not connected. Either:

    make hasKnop and hasNomen subproperties of containsMeshResource, or

    drop them and use one containment spine everywhere.

Your current situation invites inconsistent querying and documentation duplication.
6) Missing property: ReferenceRole never gets used

You define ReferenceRole individuals, but there’s no property like:

    referenceRole (ReferenceLink → ReferenceRole)

Without that, the “controlled vocabulary” is dead weight.
7) artifactKind is meta-modeling: be explicit about OWL profile intent

artifactKind range rdfs:Class means you’re pointing to classes as objects. If you care about OWL DL behavior, this is a footgun (punning / metamodeling semantics get subtle fast).

Safer alternatives:

    Make ArtifactKind a SKOS concept / individual vocabulary (not classes).

    Or make artifactKind an annotation property if it’s editorial.

8) Smaller hygiene issues (worth fixing now)

    hasPayload IRI vs label “contains payload” — pick one naming scheme.

    Inventory classes redundantly subclass <SimpleArtifact/> even though <RdfDataset/> already does.

    Consider inverses for your containment properties if you expect navigation (you already do this for hasLocatedFile/locatesAbstractFile).

What’s working well

    The MeshResource / ArtifactHost / component pattern is becoming legible.

    The “located vs abstract file” split + mesh-governed specializations (AbstractArtifactFile, LocatedArtifactFile) is crisp.

    Slices as realizables (and not identities) is a good, clean separation.

If you want, I can rewrite the worst-conflict area (Naming + ReferenceLinks) into a consistent minimal set of properties/classes (with one containment spine, one “aboutness” relation, and the literal-vs-denotation policy applied uniformly).
You said:
Should DigitalArtifact subclass some well-known Class?
ChatGPT said:

Yes, but only pick a superclass if you actually want the semantic commitments that come with it.
Best default superclass: prov:Entity

If your “DigitalArtifact” is “a thing you can version, cite, derive, and attribute,” then prov:Entity is the lowest-commitment, highest-interop alignment. PROV-O is explicitly intended to be specialized for domain classes like yours.

@prefix prov: <http://www.w3.org/ns/prov#> .

<DigitalArtifact/> rdfs:subClassOf prov:Entity .

Optional, only if you want “web content” semantics: schema:CreativeWork

Schema’s CreativeWork is very broad (includes software, photos, etc.). If you want downstream compatibility with schema-centric tooling, it’s reasonable.

@prefix schema: <https://schema.org/> .

<DigitalArtifact/> rdfs:subClassOf schema:CreativeWork .

Be careful: “CreativeWork” may feel wrong if you intend DigitalArtifact to include purely operational/internal artifacts that aren’t “works” in any human-facing sense.
Avoid as a superclass: dcat:Resource

DCAT’s dcat:Resource is specifically “things described by a catalog record,” and the spec says it’s not intended to be used directly; it’s a parent of dcat:Dataset, dcat:DataService, dcat:Catalog (or profile-defined subclasses).
So: keep your RdfDataset ⊑ dcat:Dataset (good), but don’t make all DigitalArtifacts DCAT resources unless you’re committing to a DCAT-catalog worldview for everything.
Practical recommendation for your ontology

    Make DigitalArtifact ⊑ prov:Entity (strongly recommended).

    Keep RdfDataset ⊑ dcat:Dataset as you already do.

    Add schema:CreativeWork only if you want “web content” interoperability broadly.

If you share 2–3 concrete instances you expect to type as DigitalArtifact (e.g., “resource page HTML,” “knop metadata flow,” “inventory dataset”), I can sanity-check whether schema:CreativeWork would create weird implications for your actual set.
You said:
Regarding 3B, how would change the wording.

Regarding 4, it's okay because with meshBase, I don't want to say "This mesh has a mesh base of this mesh", but with nomenFor, it's okay to say "this Nomen is a Nomen for Alice the person", no?

For #5 should containsMeshResource subclass dcterms:hasPart?

Regarding RdfDataset, I removed its subClassOf <SimpleArtifact/> because I think I want it to be able to apply to FlowArtifacts as well. If that makes sense to you, should I declare the supporting Flows (i.e., non-PayloadFlow) to be RdfDataset?
ChatGPT said:
3B wording changes (identity = content for SimpleArtifact)

Right now your text implies Realizable is a separate thing from the artifact identity, but your axiom SimpleArtifact ⊑ DigitalArtifact ⊓ Realizable makes the same resource play both roles. Make that explicit in the comments so it doesn’t read like a mistake.

Suggested replacements:

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A resource that has intrinsic content that can be realized in one or more representations (e.g., files, streams). Some resources are both an artifact identity and a realizable content bearer (e.g., SimpleArtifact, Slice)." .

<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding specification for a realizable content (e.g., an abstract file or stream), independent of any particular retrieval endpoint." .

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A mesh-governed artifact identity. A DigitalArtifact may be treated as a single content unit (SimpleArtifact) or as a revisioned artifact organized via Slices (FlowArtifact)." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A DigitalArtifact treated as a single content unit with no revision history. The artifact resource itself is the realizable content bearer." .

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A DigitalArtifact whose content evolves over time through Slices. The FlowArtifact is an identity anchor, not itself a realizable revision state." .

That removes the implied contradiction without changing your axioms.
4 denotation: meshBase vs nomenFor / designates

Your instinct is correct: it’s totally fine to assert “this Nomen is about Alice” using an object property. The problem is not denotation; the problem is that your property semantics and comments are currently crossed.

    meshBase as xsd:anyURI avoids accidentally asserting that the mesh base URI denotes some mesh resource (or that it’s even dereferenceable). That’s a reasonable hygiene choice.

    For Nomen-aboutness, you already have the right property: designates.

The issue is nomenFor’s comment: it says “Asserted designator IRI,” but the range is rdfs:Resource, so it’s really “designatum/referent,” not “designator IRI.”

You should either:

    Delete nomenFor and use only designates, or

    Keep nomenFor but rewrite it to mean designatum and then decide why it’s different from designates (right now it isn’t).

If you want both:

    designatorUriLiteral (datatype) for “the designator IRI value”

    designates (object) for “the designatum (e.g., Alice)”

5 should containsMeshResource be a subproperty of dcterms:hasPart?

Yes, if you mean “part/contained member in the mesh structure” (even if it’s conceptual / inventory-based). dcterms:hasPart is broad enough that this is a safe alignment, and it helps interoperability and querying.

<containsMeshResource/> rdfs:subPropertyOf dcterms:hasPart .

Then either:

    make hasKnop / hasNomen subproperties of containsMeshResource (recommended), or

    drop hasKnop / hasNomen to avoid two parallel containment spines.

If you keep both spines, you will get drift.
RdfDataset and flows

Your change (“remove RdfDataset ⊑ SimpleArtifact so it can apply to flows”) makes sense if RdfDataset is an artifact-kind class (i.e., it types the artifact identity, not a particular revision state). But you need to be clear what you mean:

    If RdfDataset means “this artifact is an RDF dataset,” then it can apply to both SimpleArtifact and FlowArtifact.

    If RdfDataset means “this content state is an RDF dataset,” then it should apply to Slices, not to FlowArtifact identities.

Given your current DCAT alignment (dcat:Dataset), you’re implicitly on the artifact identity path. In that case:

    Yes: it is reasonable to declare supporting flows (Knop/Mesh/Nomen metadata flows) as RdfDataset if you intend those flows to always be RDF.

    No: do not declare FlowPayload as RdfDataset globally, because you explicitly support non-RDF payloads.

Concrete approach that stays consistent with your model:

<RdfDataset/> rdfs:subClassOf <DigitalArtifact/>, dcat:Dataset .

<KnopMetadataFlow/> rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> .
<MeshMetadataFlow/> rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> .
<NomenMetadataFlow/> rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> .

Critical check: if you ever want a metadata flow whose payload is not RDF, this becomes wrong. If that’s even a 10% possibility, don’t subclass; instead use artifactKind assertions per-instance.

If you want, paste your intended meaning of RdfDataset in one sentence (“identity-kind” vs “content-state-kind”). That single choice determines whether it should attach to FlowArtifacts or Slices.
You said:
Regarding the comment for Realizable, Slices aren't really an artifact identity... Flows are their artifact identity. 

I want DigitalArtifact NOT to subclass MeshResource, it can be anything out there. Is there a well-known vocabulary class we can subclass?

nomenFor is supposed to target the designator, not the designatum. Maybe we'll just remove it since it's redundant with meshPath. While we're at it, I think changing meshPath to designatorPath. 
ChatGPT said:
Realizable comment (Slices are not identities)

If FlowArtifact is the identity and Slice is the revision state, then the comment should say exactly that:

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A content-bearing resource that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Examples include SimpleArtifacts (where the identity is itself the content bearer) and Slices (revision states of a FlowArtifact). FlowArtifacts are identity anchors and are not themselves realizable revision states." .

That wording makes the intended separation explicit and prevents readers from inferring “FlowArtifact content == FlowArtifact”.
DigitalArtifact: don’t subclass MeshResource, but then you must split “mesh-governed” artifacts

If you remove DigitalArtifact ⊑ MeshResource, you cannot keep hasPayload ⊑ containsMeshResource with range DigitalArtifact, because containsMeshResource’s range will re-impose “everything contained is a MeshResource” via RDFS range inference.

So you need two layers:

    a global artifact identity class

    a mesh-governed artifact identity class used in containment

Recommended:

@prefix prov: <http://www.w3.org/ns/prov#> .

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf prov:Entity ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "An artifact identity that may exist inside or outside any particular mesh." .

<MeshDigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <MeshResource/> ;
  rdfs:label "Mesh digital artifact" ;
  rdfs:comment "A DigitalArtifact governed by a mesh and addressable within a mesh's namespace." .

<FlowArtifact/> rdfs:subClassOf <MeshDigitalArtifact/> .
<SimpleArtifact/> rdfs:subClassOf <MeshDigitalArtifact/>, <Realizable/> .

Superclass choice: prov:Entity is the cleanest “well-known” alignment for “physical, digital, conceptual… real or imaginary.”
(You can also align to schema:CreativeWork, but it carries a “published work” vibe that may not fit operational artifacts. )
nomenFor targeting the designator: remove it, but replace it with the right thing

If nomenFor is meant to point to the designator IRI, an object property is the wrong tool unless you introduce an explicit “designator resource” (e.g., a NomenHandle). If you just want the IRI value without asserting anything about what it denotes, make it a URI literal.

Given you already have a path token, the clean move is:

    remove nomenFor

    rename meshPath → designatorPath

    optionally add designatorUriLiteral for convenience/debugging (computed or asserted)

<designatorPath/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Mesh-relative path token for the Nomen's designator IRI. Interpreted relative to meshBase. No leading slash. Empty string denotes the mesh root." .

<designatorUriLiteral/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "designator URI literal" ;
  rdfs:comment "Optional convenience value for the full designator IRI (without asserting denotation). Typically derived from meshBase + designatorPath." .

Then keep designates strictly for the designatum (e.g., Alice).

The main inconsistency to avoid: “designator” (identifier) and “designatum” (thing identified) must never share a predicate family. Right now your nomenFor comment was pushing you toward that collision.
You said:
Regarding DigitalArtifact, I actually made Flow/SimpleArtifact not MeshResources but I know that's trouble because I made "Artifact" my signifier for "Mesh-managed". e.g. LocatedArtifactFile, AbstractArtifactFile, etc. I think we should rename those to LocatedMeshFile, etc. and then fix properties like locatesAbstractArtifactFile. See what you can do (maybe opening in a canvas?)

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <https://schema.org/> .
@prefix vann: <http://purl.org/vocab/vann/> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .


# Core Semantic Mesh-scoped resources

<MeshResource/> a rdfs:Class ;
  rdfs:label "Mesh resource" ;
  rdfs:comment "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves" .

<ArtifactHost/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A mesh-governed resource that can contain artifacts and associated structural resources. Nomen are ArtifactHosts that can also contain other Nomen. " .

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains mesh-governed resources and, optionally, other supporting resources." .

<meshBase/> a owl:DatatypeProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "Mesh Base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

<MeshComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Mesh component" ;
  rdfs:comment "A reserved, mesh-scoped structural resource (e.g., mesh-level metadata, mesh inventory)." .  

<containsMeshComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshComponent/> ;
  rdfs:label "contains mesh component" ;
  rdfs:comment "Associates a Mesh with one of its directly contained MeshComponents." .

<containsKnopComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopComponent/> ;
  rdfs:label "contains knop component" ;
  rdfs:comment "Associates a Knop with one of its directly contained KnopComponents." .

<containsNomenComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenComponent/> ;
  rdfs:label "contains nomen component" ;
  rdfs:comment "Associates a Nomen with one of its directly contained NomenComponents." .

  
# General-purpose Resources
# These could be usable anywhere on the Semantic Web, not just in Semantic Flow meshes'

## Digital Artifact

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A mesh-governed artifact identity (something treated as a digital artifact in the abstract)." .

## Realizable + Realizations

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A content-bearing resource that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Examples include SimpleArtifacts (where the identity is itself the content bearer) and Slices (revision states of a FlowArtifact). FlowArtifacts are identity anchors and are not themselves realizable revision states." . .


<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding specification for a realizable content (e.g., an abstract file or stream), independent of any particular retrieval endpoint." .


## Abstract files and streams

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-form realization specification (format/syntax/canonicalization/bundling). May be mesh-governed or external." .

<AbstractStream/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract stream" ;
  rdfs:comment "A non-file realization specification (e.g., a stream or service-like packaging), reserved for future use." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractFiles." .

<hasAbstractStream/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractStream/> ;
  rdfs:label "has abstract stream" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractStreams." .

## Located files (byte retrieval)

<LocatedFile/> a rdfs:Class ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes endpoint (typically with an extension), which may be in-mesh or external." .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  owl:inverseOf <locatesAbstractFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile/> ;
  rdfs:range <AbstractFile/> ;
  owl:inverseOf <hasLocatedFile/> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .


## Resource Pages

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "A derived UI/navigation file (e.g., index.html) that presents a mesh-governed resource (or other referent) to humans." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource (including referents) with a human-readable presentation page." .

# Semantic Flow Artifacts

## Digital Artifacts kinds

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A DigitalArtifact whose content evolves over time through Slices. The FlowArtifact is an identity anchor, not itself a realizable revision state." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A DigitalArtifact treated as a single content unit with no revision history. The SimpleArtifact resource itself is the realizable content bearer." .

## Artifact Files

<LocatedArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/> ;
  rdfs:label "Located artifact file" ;
  rdfs:comment "A mesh-governed LocatedFile that provides concrete bytes for a mesh-governed abstract artifact file." .

<AbstractArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <AbstractFile/>, <MeshResource/> ;
  rdfs:label "Abstract artifact file" ;
  rdfs:comment "A mesh-governed AbstractFile that participates in Semantic Flow's artifact modeling." .

<hasAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasAbstractFile/> ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractArtifactFile/> ;
  rdfs:label "has abstract artifact file" ;
  rdfs:comment "Links a Realizable resource to one of its mesh-governed AbstractArtifactFiles." .

<hasLocatedArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasLocatedFile/> ;
  rdfs:domain <AbstractArtifactFile/> ;
  rdfs:range <LocatedArtifactFile/> ;
  owl:inverseOf <locatesAbstractArtifactFile/> ;
  rdfs:label "has located artifact file" ;
  rdfs:comment "Links an AbstractArtifactFile to one or more mesh-governed LocatedArtifactFiles." .

<locatesAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <locatesAbstractFile/> ;
  rdfs:domain <LocatedArtifactFile/> ;
  rdfs:range <AbstractArtifactFile/> ;
  owl:inverseOf <hasLocatedArtifactFile/> ;
  rdfs:label "locates abstract artifact file" ;
  rdfs:comment "Navigation: links a LocatedArtifactFile to the AbstractArtifactFile whose bytes it provides." .


# Slices

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A particular revision member of a FlowArtifact. A Slice is content-bearing/realizable but is not a DigitalArtifact identity." .

<WorkingSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Working slice" ;
  rdfs:comment "Mutable current state for a FlowArtifact." .

<HistoricalSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Historical slice" ;
  rdfs:comment "Immutable published snapshot for a FlowArtifact." .

<hasSlice/> a owl:ObjectProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <Slice/> ;
  rdfs:label "has slice" ;
  rdfs:comment "Associates a FlowArtifact with one of its Slices." .

<currentSlice/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasSlice/> ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <WorkingSlice/> ;
  rdfs:label "current slice" ;
  rdfs:comment "Associates a FlowArtifact with its current Slice." .

<previousSlice/> a owl:ObjectProperty ;
  rdfs:domain <Slice/> ;
  rdfs:range <Slice/> ;
  rdfs:label "previous slice" ;
  rdfs:comment "Links to the immediately-preceding Slice of this Slice." .

<nextSliceSequenceNumber/> a owl:DatatypeProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range xsd:integer ;
  rdfs:label "next slice sequence number" ;
  rdfs:comment "Monotonic integer providing the next allocatable sequence number for a FlowArtifact." .


# Naming

<Nomen/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about. A Nomen may also function as a namespace prefix for subordinate Nomens under its path." .

<containsNomen/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Nomen/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<NomenComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Nomen component" ;
  rdfs:comment "Reserved components under a Nomen (e.g., nomen flows, inventories)." .

<designatorPath/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Mesh-relative path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root Nomen." .

<designates/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for namespacing / token management)." .

<hasChildNomen/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "has child nomen" ;
  rdfs:comment "Operational/derived listing of subordinate Nomens under this Nomen's namespace path. Not a source of truth; implementations may derive children from mesh structure and/or designatorPath prefix rules." .


### Reference data links

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a MeshResource to a specific reference data source, including role metadata." .

<ReferenceRole/> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical/> a <ReferenceRole/> ;
  rdfs:label "Canonical" .

<ReferenceRole/Integrate/> a <ReferenceRole/> ;
  rdfs:label "Integrate into resource page" .

<ReferenceRole/Supplemental/> a <ReferenceRole/> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated/> a <ReferenceRole/> ;
  rdfs:label "Deprecated" .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <MeshResource/> ;
  rdfs:range <ReferenceLink/> ;
  owl:inverseOf <referenceLinkFor/> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a mesh-governed resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <MeshResource/> ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the mesh-governed resource that a ReferenceLink is about." .

<hasReferenceRole/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ReferenceRole/> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a node used as a reference-data identifier (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

<referenceUriLiteral/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

### Verification metadata

<lastVerifiedAt/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .


# Knops + flows

<Knop/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for DigitalArtifacts anchored at a stable in-mesh location; hosts payload and supporting FlowArtifacts and SimpleArtifacts." .

<containsKnop/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Knop/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<KnopComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Knop component" ;
  rdfs:comment "Reserved components under a Knop (payload/supporting flows and their parts)." .

<FlowPayload/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload flow" ;
  rdfs:comment "The primary FlowArtifact hosted by a Knop." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

<containsPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "contains payload" ;
  rdfs:comment "Links a Knop to its primary payload artifact identity (either a FlowArtifact-based payload or a SimpleArtifact-based payload)." .

<containsFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "contains flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<containsSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "contains simple payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a SimpleArtifact (SimplePayload)." .

# Metadata

<KnopMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/>, <RdfDataset/> ;
  rdfs:label "Knop metadata flow" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop." .

<MeshMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <MeshComponent/>, <RdfDataset/> ;
  rdfs:label "Mesh metadata flow" ;
  rdfs:comment "Optional mesh-level operational metadata/config." .

<NomenMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <NomenComponent/>, <RdfDataset/> ;
  rdfs:label "Nomen metadata flow" ;
  rdfs:comment "Metadata about a Nomen; may remain WorkingSlice-only until history is recorded." .

<containsKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "contains knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<containsMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "contains mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<containsNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "contains nomen metadata flow" ;
  rdfs:comment "Links a Nomen to its nomen metadata flow (if present)." .


# RDF datasets 

<RdfDataset/> a rdfs:Class ;
  rdfs:subClassOf dcat:Dataset ;
  rdfs:label "RDF dataset" ;
  rdfs:comment "An Artifact whose content is an RDF dataset (default graph + 0..n named graphs)." .


# Inventories

<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <MeshComponent/>, <SimpleArtifact/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <KnopComponent/>, <SimpleArtifact/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <NomenComponent/>, <SimpleArtifact/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .
ChatGPT said:

The core problem is inference: as long as containsPayload ⊑ containsMeshResource, the payload must be a MeshResource. If you want “digital artifacts out in the world,” you need a mesh-governed subclass that is safe to use as the range of mesh-containment properties.

I opened a canvas with a concrete patch that:

    splits DigitalArtifact (general) vs MeshArtifact (mesh-governed)

    renames AbstractArtifactFile/LocatedArtifactFile → AbstractMeshFile/LocatedMeshFile

    renames the corresponding properties (hasLocatedMeshFile, locatesAbstractMeshFile, etc.)

    tightens containsPayload to range MeshArtifact

    optionally provides deprecated-IRI bridges so you can migrate without breaking everything

If you adopt that split, you can keep “artifact” as a general word, and make “Mesh*” your governance signifier (which is much less ambiguous than trying to make “Artifact” imply mesh-managed).
You said:
I think we could unify components and artifacts, because all the components are now artifacts (either Flows or RdfDataset ^ SimpleArtifact). Maybe we keep renamed classes KnopArtifact, MeshArtifact, and NomenArtifact as abstract classes but I'm thinking get rid of 'em. Additionally just use containsMeshResource as the Predicate? Well, might be hard because <MeshComponent> is now "mesh-specific supporting artifact". But I do want a name change because MeshResource includes KnopComponent/NomenComponents AND MeshComponent, i.e. the "Mesh" prefix should maybe only be used for specifically-mesh things.

If we keep components, ArtifactHost might make more sense as ComponentHost. 

I'm thinking instead of a "contains" property for each different class of thing, (e.g. containsPayload, containsNomenMetadataFlow,) we just use containsMeshResource or whatever we call it now... maybe containsSfloResource. 

No changes, just thoughts please. My goal is a slimmer ontology, but I think we should maintain the "general spine" and the "Sflo/Mesh spine" for artifacts. Maybe that distinction becomes Artifacts for general and Components for sflo/mesh.
ChatGPT said:

You’re circling the right simplification, but several of your instincts conflict with your own “slim + non-ambiguous” goal unless you tighten the semantics.
1) Unifying “components” and “artifacts” is feasible, but only if “component” stops meaning “special”

If every component is now a FlowArtifact or a (RdfDataset ∧ SimpleArtifact), then “Component” is no longer a fundamental ontological kind; it’s a role: “this thing is hosted under a particular host and has a reserved operational purpose.”

If you keep MeshComponent/KnopComponent/NomenComponent as classes, they will quickly become redundant shadows of:

    “is hosted under Mesh/Knop/Nomen”

    “is supporting vs payload”

    “is reserved/internal vs arbitrary”

That redundancy will re-inflate the ontology.

Slim option: drop *Component classes and represent “component-ness” as:

    containment (containsSfloResource)

        one classifier property (e.g., componentRole with a small controlled vocab: payload, metadata, inventory, etc.)

If you keep *Component, commit that they are purely structural (no extra meaning beyond “hosted under X and reserved namespace semantics”), or they will become meaning sinks.
2) The “Mesh” prefix really is overloaded right now

Agreed: MeshResource currently means “SF-governed resource in a mesh namespace,” not “resource about the Mesh itself.” That’s why “MeshComponent” reads wrong—it sounds “mesh-level,” but your model uses it as “mesh-scoped structural resource.”

If you want “Mesh” to mean “about the Mesh entity,” rename the spine:

    SfloResource (or MeshScopedResource) for “in-mesh governed”

    reserve Mesh* for “mesh-level” (things contained directly by the Mesh, not by a Knop/Nomen)

This is a genuine clarity win and helps documentation tightness.
3) Do not collapse all specialized containment predicates into only containsSfloResource unless you accept loss of meaningful constraints

If you use only one predicate, you lose:

    domain/range signal

    query ergonomics (“find payloads” becomes “find contained things of type X”)

    ability to express invariants (exactly-one payload per knop, 0..1 metadata flow, etc.) without SHACL

You can still slim the ontology by making the special properties optional conveniences rather than the core.

Good compromise:

    Keep one core containment predicate: containsSfloResource (or containsResource)

    Keep a small set of subproperties only where they encode real invariants:

        containsPayload (because it’s conceptually special and likely cardinality-constrained)

        maybe containsNomen / containsKnop if you care about index semantics

    Drop ultra-specific ones like containsNomenMetadataFlow if they don’t add constraints beyond type.

This keeps the ontology slim while preserving the “spine.”
4) Renaming ArtifactHost → ComponentHost only helps if you keep “components”

If you unify components into artifacts, renaming to ComponentHost becomes misleading. The host is hosting artifacts/resources, not “components.”

If you keep a host abstraction, name it for the structural role, not the hosted kind:

    ContainerResource / SfloContainer

    or keep ArtifactHost if “artifact hosting” remains the core function

Right now “ArtifactHost” is mostly “container in the mesh hierarchy.” If you rename anything, rename toward “Container.”
5) Your “general spine vs SF spine” is the right architecture—but pick one axis and stick to it

You proposed: “Artifacts general, Components SF.” That can work, but only if:

    “Component” means “SF-governed” (not “supporting”)

    and you never again let “Artifact” mean “mesh-managed”

However, you already use “artifact” for realizations (files), so conflating “component” with “mesh-governed” will collide conceptually with your file model unless you’re careful.

Cleaner:

    General artifact identity: DigitalArtifact (global)

    SF-governed resource spine: SfloResource (global “in mesh governance”)

    Mesh-governed artifact identity: SfloArtifact (intersection of both) if you need it; otherwise infer it by containment/placement.

That avoids reusing “component” as a synonym for “SF.”
6) A hard warning: if you drop *Component and rely only on containment + type, you must decide how you prevent “payload vs supporting” confusion

Right now KnopComponent provides a coarse “belongs under a knop” bucket. If you remove it, you need at least one of:

    SHACL shapes asserting which classes may be contained by which hosts, and/or

    one “role” property on contained resources (payload/metadata/inventory/etc.)

Otherwise, any SfloResource can be contained anywhere and you lose structural sanity.
Suggested direction (slim but not mushy)

    Rename MeshResource → SfloResource (or MeshScopedResource)

    Keep one containment predicate containsSfloResource (subPropertyOf dcterms:hasPart)

    Keep only these special subproperties:

        containsPayload (because it encodes a real conceptual role + cardinality expectations)

        optionally containsKnop, containsNomen if they support your “inventory not filesystem” story

    Either:

        keep *Component as purely structural buckets, or

        remove them and replace with a single controlled vocab like resourceRole or hostRole

The key inconsistency to avoid: don’t make “Components = SF” while also keeping “Components = supporting,” because you’ll immediately need a third axis.

If you want, I can identify which of your current containsX properties actually add unique semantics (vs just “contains + rdf:type”), so you can delete the rest without losing expressivity.
You said:
Does SPARQL do enough inferencing, for example, that querying ?s containsPayload ?o will return stuff specified with containsFlowPayload and containsSimplePayload as well?

If we only use one predicate, "find contained things of type X" isn't so bad. It's actually kinda nice because you don't have to worry about matching the predicate to the class. 

Right now all "contains" targets are cardinality-constrained... Exactly one component per ArtifactHost (which could be ComponentHost). We might introduce some optional ones later though, although they would also likely be max cardinality 1. Can SHACL express the cardinalities adequately with only a single predicate?

Regarding Artifacts vs Components... Yes, I'm thinking we get rid of Components. SfloContainer instead of ArtifactHost seems good. 

I think I want to re-introduce dcat:DatasetSeries as a new kind of Artifact: SeriesArtifact: Basically a set of SimpleArtifacts. It could be used for logging or sharding.
ChatGPT said:
SPARQL “inferencing” for subproperties

Not by default.

A plain triple pattern like:

    ?s sflo:containsPayload ?o

matches only triples asserted with that exact predicate, under SPARQL’s default “simple entailment”. Subproperty matches (e.g., sflo:containsFlowPayload rdfs:subPropertyOf sflo:containsPayload) only show up if your store/endpoint is configured with an entailment regime (e.g., RDFS) or has materialized those inferences.

If you want queries to be robust without relying on store reasoning, use an explicit “subproperty closure” pattern:

?p rdfs:subPropertyOf* sflo:containsPayload .
?s ?p ?o .

That will pull values asserted via containsPayload or any of its subproperties, regardless of entailment configuration.

One more sharp edge: SPARQL property paths are evaluated “as written” and can interact surprisingly with entailment regimes; don’t assume property-path evaluation itself is doing RDFS reasoning for you.
“One predicate for containment” + querying by type

Your instinct is right: if you standardize on one containment predicate (e.g., containsSfloResource), then “find contained things of type X” is straightforward and avoids predicate/class coupling:

    ?host sflo:containsSfloResource ?x . ?x a sflo:FlowPayload .

Two non-obvious requirements if you go this way:

    You must be disciplined about explicit typing. If you start relying on subclass inference (“it’s a FlowPayload because it’s a PayloadFlow because …”), your results will vary by store entailment settings.

    You need a clear story for “roles”. Today you encode role via class (FlowPayload, KnopMetadataFlow, etc.). That works, but it’s a vocabulary-growth pressure point: every “slot” becomes a class.

SHACL cardinalities with a single predicate

Yes—SHACL can express “exactly one contained thing of type X” even if everything uses one predicate, via qualified value shapes.

Conceptually:

    Path: sflo:containsSfloResource

    Constraint: among the values of that path, exactly 1 conforms to “is a FlowPayload (or SimplePayload)”

    Similarly: 0..1 conforms to “is a KnopMetadataFlow”

    Etc.

Critical consistency check: you said “all contains targets are cardinality-constrained… exactly one component per ArtifactHost”. That cannot be true if the same predicate is used for all containment and a host can contain multiple things (payload + metadata + inventories + child knops/nomens). With a single predicate you generally cannot use sh:maxCount 1 on the path; you must express cardinalities per category using qualified constraints.

So: single predicate is compatible with tight cardinalities, but only if you mean “max 1 per slot/type”, not “max 1 total contained thing”.
Re-introducing dcat:DatasetSeries as “SeriesArtifact”

DCAT 3’s dcat:DatasetSeries is explicitly “a collection of datasets…”, and DCAT’s recommended linkage is: the member dataset points to the series using dcat:inSeries.

Two design pitfalls to avoid:

    Scope mismatch. If your “SeriesArtifact” is “a set of SimpleArtifacts” (general-purpose), DCAT is too narrow: it’s about datasets. If you only need series for RDF datasets (shards, logs), then aligning to DCAT is clean.

    Concept collision with Flow/Slice. A FlowArtifact already models an ordered revision chain. A “series” used for sharding/logging is a different beast (collection/partition, not revision lineage). If you keep both, they need crisp separation in naming and comments.

A minimal, low-confusion option:

    Keep your general concept: something like ArtifactCollection / ArtifactSet (domain-neutral).

    Add a specific subclass for dataset series: RdfDatasetSeries subClassOf dcat:DatasetSeries (only when the members are dcat:Dataset / your RdfDataset), using dcat:inSeries for membership.

That lets you use DCAT where it actually fits, without forcing all “series” concepts into dataset semantics.

If you want, I can also propose how this interacts with your “FlowArtifact is identity anchor, Slice is realizable” rule—because typing FlowArtifact itself as RdfDataset is semantically inconsistent under your own model, whereas typing the slice content (or setting artifactKind) is consistent.
You said:
Yes, I meant "exactly one of each allowed component class per ArtifactHost".

I haven't thought through SeriesArtifact too much, but maybe it would just be an "abstract artifact" (not Realizable) like Flow, that pointed to other Payload SimpleArtifacts. i.e., a way to group artifacts together. 

A Flow/Slice is a revision chain, but a DatasetSeries is a series of distinct things, like log files, or annual summaries. 

Is ArtifactCollection really useful? I guess I was just assuming you would do groupings of artifacts with domain-specific language before. Maybe that's still the best way to go. 
ChatGPT said:

Your clarified constraint (“exactly one of each allowed component class per host”) is compatible with the “single containment predicate + SHACL qualified constraints” approach, and it strongly argues against keeping lots of containsX predicates. The predicates don’t buy you much if you’re going to enforce “slot cardinality by type” anyway.
1) SeriesArtifact as “abstract artifact” (not Realizable) is coherent

Model-wise it’s the same pattern as FlowArtifact: an identity anchor that organizes other artifacts but is not itself a content-bearing unit.

Key is to avoid accidental confusion with:

    revision lineage (Flow/Slice)

    membership/collection (Series)

So make the semantics crisp:

    FlowArtifact: orders Slices as revisions of “the same thing over time”

    SeriesArtifact: groups member artifacts as “a set/sequence of distinct things” (logs, shards, annual summaries)

Two hard questions you should answer before naming anything:

    Is a Series ordered or just a set? (Logs imply ordered; sharding often doesn’t.)

    Are members required to be homogeneous by kind? (All RdfDataset? All SimpleArtifact? Mixed?)

If you can’t answer those, you’ll end up with a mushy “collection” concept that doesn’t constrain anything and won’t be used consistently.
2) DCAT DatasetSeries: use it only for dataset-focused series

If your SeriesArtifact is genuinely “a series of datasets,” dcat:DatasetSeries is a good alignment. But if you want series for non-dataset artifacts too, DCAT won’t cover that cleanly, and forcing it will confuse your ontology.

Recommendation:

    Introduce a domain-neutral concept only if you truly need it.

    Otherwise, keep series domain-specific: RdfDatasetSeries ⊑ dcat:DatasetSeries.

You can still have “log file series” and “shard series” as subclasses of RdfDatasetSeries if they’re datasets.
3) Is ArtifactCollection useful?

It’s useful only if it buys you one of these:

    a shared membership pattern used across multiple series types

    shared constraints (homogeneity, ordering, cardinality, provenance)

    shared UI/discovery features (inventory generation, navigation, citations)

If none of those exist, then a generic ArtifactCollection becomes dead weight: it looks foundational but adds no predictive power. In a slim ontology, that’s exactly what you should avoid.

A good litmus test:

    If you can define one membership predicate + SHACL constraints that will be reused by at least two distinct “series-like” constructs, it’s worth a base class.

    If each grouping will immediately need its own semantics anyway (log cadence, shard key, season/year partitioning), do it domain-specifically and skip the generic base.

4) What I’d do given your current goals

Given “slimmer ontology” and your existing Flow/Slice complexity:

    Don’t add ArtifactCollection yet.

    Add a single grouping construct only if you already have at least two real use cases that share structure (e.g., “log series” and “shard series” both need ordering + members are datasets + you want a uniform UI pattern).

    If your immediate driver is “log files or annual summaries,” implement RdfDatasetSeries (dataset-only) and keep it tight.

If you later find you need series for non-datasets, you can generalize then (and you’ll know what semantics must be preserved).
5) One more consistency trap

You said “SeriesArtifact points to other payload SimpleArtifacts.” That’s already a semantic commitment: it excludes FlowArtifacts as members (and excludes “the slice content” which is where realizable states live). If you want series to group “published snapshots” (slices) or “current datasets,” you may regret limiting membership to SimpleArtifact.

Safer: members are DigitalArtifact (general) or MeshArtifact (mesh-governed), and homogeneity is enforced by SHACL per series subtype.

If you tell me whether “series membership” should include FlowArtifacts, SimpleArtifacts, or Slices, I can tell you which of those choices will create contradictions with your current Realizable/Slice story.
You said:
Removing containsX predicates is scary because it actually does add a little SPARQL complexity, but not too bad right?

Series membership would of course include FlowArtifacts as well as SimpleArtifacts. Slices is an interesting case. But I'm back to thinking no Sets/Series for now, it can be handled as Payloads using dcat or a domain-specific vocab.
ChatGPT said:

Removing containsX predicates does add a little SPARQL complexity, but it’s manageable if you standardize 1–2 query idioms and bake them into docs/examples.
1) How much extra SPARQL complexity?
With specialized predicates

?knop sflo:containsFlowPayload ?payload .

With one predicate + type filter

?knop sflo:containsSfloResource ?payload .
?payload a sflo:FlowPayload .

That’s not bad. The “scary” part is when you need “either FlowPayload or SimplePayload”:

?knop sflo:containsSfloResource ?payload .
?payload a ?t .
VALUES ?t { sflo:FlowPayload sflo:SimplePayload }

Or if you use subclassing heavily, you end up wanting entailment, but you can avoid that by asserting the direct types you query for.
Where it does get worse

If you remove specialized predicates and you remove the “slot classes” (FlowPayload, KnopMetadataFlow, etc.), then every query becomes “contains + role property” and you must remember role URIs everywhere. That’s when it becomes genuinely annoying.

So the key is: keep the slot-defining classes (or a small role vocabulary), even if you drop the slot predicates.
2) Mitigation strategy (practical)

If you drop containsX, keep:

    one containment predicate: containsSfloResource

    stable “slot classes” you query against:

        FlowPayload, SimplePayload, KnopMetadataFlow, MeshInventoryDataset, etc.

Then:

    SPARQL remains simple

    SHACL qualified cardinalities remain expressible

    your ontology gets slimmer (fewer predicates), but you don’t explode query patterns

3) Your series decision is consistent

If series membership would include FlowArtifacts and SimpleArtifacts, you’re already in “general grouping” territory, and DCAT’s DatasetSeries only fits the dataset subset. If you don’t need it immediately, deferring is the right move—otherwise you’ll either:

    over-generalize (ArtifactCollection) without real constraints, or

    shoehorn non-dataset membership into a dataset-only class and confuse users.

Handling “groupings” via payload-level domain vocabularies (including DCAT where applicable) is the low-risk path until the use cases force a common abstraction.
4) One last caution

If you remove containsPayload specifically, you lose the most important ergonomic query. Consider keeping only that one as a subproperty (for convenience + clarity), and delete the rest. That gives you a slim predicate set while preserving the “primary payload” shortcut.
You said:
OH, SPARQL does do subclass entailment either. e.g., we can't bring back all SfloResources with ?s ?p sflo:SfloResource  unless the data names them as SfloResource explicitly?
ChatGPT said:

Correct: under default SPARQL semantics you do not get rdfs:subClassOf (or rdfs:subPropertyOf) inference “for free”.

SPARQL basic graph patterns are defined over simple entailment; other entailment (RDFS/OWL/etc.) is optional and endpoint-specific.
Two separate issues in your example

    Your pattern is not a type query

?s ?p sflo:SfloResource matches triples whose object is the IRI sflo:SfloResource.

To ask for instances, you want:

?s a sflo:SfloResource .

    Subclass instances won’t be returned unless you do reasoning or rewrite the query

If your data says:

    :x a sflo:Knop .

    sflo:Knop rdfs:subClassOf sflo:SfloResource .

Then ?s a sflo:SfloResource will not return :x unless:

    your store is running an entailment regime (e.g., RDFS entailment), or it materialized inferred types ahead of time,
    or

    you explicitly traverse the class hierarchy in the query:

SELECT ?s WHERE {
  ?s a ?c .
  ?c rdfs:subClassOf* sflo:SfloResource .
}

That works even without entailment, as long as the queried dataset contains the rdfs:subClassOf triples. Property paths are standard SPARQL 1.1.
Practical takeaway for your ontology/docs

If you want “give me all SfloResources” to be a cheap, obvious query everywhere, you’ll either need:

    explicit rdf:type sflo:SfloResource assertions (materialized), or

    to standardize the property-path idiom above in your docs (and accept the extra join/path cost).

You said:
Man, I am going to miss MeshArtifact if we go to SfloArtifact. It does feel like there is some haziness because Mesh is "the whole thing" but also has its specific components/SfloArtifacts. 

Why don't you take a little time a suggest three different ontology approaches (doesn't have to be full turtle, just classnames): one that's a maximal take on where we were headed with the two "spines" of general and Sflo-specifc artifacts; one that's totally as minimal as possible; and one that "thinks outside the box"
ChatGPT said:
Approach A: Maximal two-spine, keep the “MeshArtifact” feel by splitting Mesh-scoped vs mesh-level

This keeps your two-spine architecture and resolves the haziness you’re feeling (“Mesh = whole thing” vs “Mesh has its own components”) by using two different Mesh-ish prefixes:

General spine (world-wide, not SF-specific)

    DigitalArtifact (identity)

    Realizable (content-bearing)

    Realization

    AbstractFile, AbstractStream

    LocatedFile

    ArtifactFlow (identity anchor for revision chains)

    ArtifactSlice (revision state; realizable)

SF spine (mesh-governed resources and structure)

    SfloResource

    SfloContainer (formerly ArtifactHost)

    Mesh, Knop, Nomen

Bridge classes (where the spines meet)

    SfloArtifact (intersection: DigitalArtifact + SfloResource)

    SfloFlowArtifact (subclass of ArtifactFlow + SfloArtifact)

    SfloSimpleArtifact (subclass of DigitalArtifact + Realizable + SfloArtifact)

    SfloSlice (subclass of ArtifactSlice + SfloResource)

Keep “MeshArtifact” without overloading “Mesh”

    MeshScopedArtifact (synonym-ish for SfloArtifact: governed by a mesh; lives anywhere under mesh namespace)

    MeshLevelArtifact (artifacts contained directly by the Mesh container: inventories, mesh metadata, etc.)

This is the cleanest way to keep the emotional clarity of “MeshArtifact” while preventing “Mesh = everything” ambiguity.
Approach B: Minimal as possible (slim ontology, accept more SHACL/SPARQL patterns)

This is the “single containment predicate + query-by-type + SHACL slots” world. Almost no intermediate abstractions.

    SfloResource

    SfloContainer

    Mesh, Knop, Nomen

    containsSfloResource (one containment predicate)

    DigitalArtifact

    FlowArtifact

    Slice

    SimpleArtifact

    AbstractFile, LocatedFile

    ReferenceLink

    designatorPath, designates

No *Component classes, no *MeshFile vs *ArtifactFile split, no “inventory dataset” subclasses unless you truly need them. Slot cardinalities are enforced entirely in SHACL via qualified constraints on containsSfloResource.

This will feel good to author, but it forces you to be disciplined about types as your primary API.
Approach C: Outside-the-box (stop making “containment” and “component slots” be classes; model them as relators/records)

This makes the ontology smaller and often clearer, but it’s a conceptual shift: instead of “a Knop contains a payload,” you create an explicit membership/slot record.

Core entities

    SfloResource, SfloContainer (Mesh, Knop, Nomen)

    DigitalArtifact (global)

    FlowArtifact, Slice, SimpleArtifact

    AbstractFile, LocatedFile

Relator layer (this is the shift)

    ContainmentLink (host ↔ member)

    SlotAssignment (host ↔ member with a role like payload/metadata/inventory)

    ReferenceLink (you already have this pattern)

Role vocabulary

    SlotRole with individuals: Payload, Metadata, Inventory, UIPage, etc.

Then you can have exactly one SlotAssignment per role per host (SHACL is trivial here), and your ontology stays slim because you don’t need containsPayload, containsKnopMetadataFlow, etc. The “slot” becomes data, not vocabulary bloat.

Tradeoff: more triples, more indirection, but far less class/property proliferation and far fewer naming collisions.
The inconsistency you should decide explicitly (it drives which approach wins)

Do you want “slot semantics” (payload/metadata/inventory) to live in:

    the vocabulary (lots of containsX / lots of XComponent classes), or

    the data (single contains + type queries, or explicit slot records)?

If you pick “vocabulary,” minimalism will keep fighting you. If you pick “data,” you must standardize 2–3 SPARQL idioms and accept that as part of the platform contract.
You said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
What I will miss about MeshResource is how concise and pronounceable it is. Brainstorm some ideas for something better than SfloResource. MeshScopedArtifact is ungainly, and it's not clearly different from MeshLevelArtifact. 

By the way, I definitely have moved away from "Nomen contain other Nomen". The Filesystem containment metaphor is broken because child Nomen don't live in under the Nomen IRI, they live under the designator IRI. 

For approach C, yes less class proliferation, but you get slot proliferation and loss of clarity. I do love relators in general, especially for solving n-ary relationships. But the simplicity of traditional class/property approach is nice here, we're a general framework.

I think the vocabulary is important here too. I want to name our few things. There's not a lot on the general side or the Sflo side. I think there might be a compromise between A & C where the general stuff doesn't need a mirroring sflo, but we do name the Sflo-specific stuff. i.e., no LocatedArtifactFile or equivalent, just LocatedFile; no hasAbstractMeshFile, just hasAbstractFile.

We need to brainstorm a replacement prefix for Sflo. Or else we just use Mesh and accept the ambiguity. If we get rid of MeshComponent (and KnopComponent and NomenComponent), and the use of Mesh in property names, it's only a little overloaded so MeshResource can be the marker superclass for things in a mesh. 

Wanna take a stab at a new "patch? Your containsPayload ⊑ containsMeshResource is actually not a big deal because only Knops have Payloads, i.e., isn't okay to force the payload to be a MeshResource?
ChatGPT said:
Names better than SfloResource (keeping the “concise + pronounceable” property)

If you want the class name itself to convey “in-mesh governed” (not “about the Mesh entity”), the best short candidates are:

    MeshResource (your current; still best overall)

    InMeshResource (explicit, still pronounceable)

    MeshLocalResource (clear, a bit longer)

    MeshMember / MeshItem (short, but “resource” semantics weaken)

    MeshedResource (cute, but reads a little informal)

    GovernedResource (too generic unless namespaced and heavily documented)

If you keep MeshResource, the way to reduce the “Mesh overload” is not renaming it; it’s removing other uses of “Mesh” that mean something different (especially MeshComponent).
Nomen containment: you’re right to back away

If child Nomen live under the designator IRI surface, not under the Nomen resource IRI, then “Nomen contains Nomen” is a misleading metaphor. At most, you can keep something like hasChildNomen as an operational/derived relation based on designatorPath prefix rules, but it should not be presented as structural containment.
About containsPayload ⊑ containsMeshResource

Yes: if you accept “a Knop’s payload identity is always mesh-hosted,” then it’s fine (and arguably correct) that payloads are forced to be MeshResource.

The important consistency rule is:

    Hosting/containment implies governance.
    So if you ever want “payload is an external artifact not governed by the mesh,” it shouldn’t be modeled as a Knop payload; it should be referenced via ReferenceLink (or a separate “refersToPayload” style predicate).

A concrete compromise patch direction (between your A and your C)

I updated the canvas patch to reflect what you just described:

    keep MeshResource as the marker superclass (do not rename to SfloResource)

    keep DigitalArtifact as general (not mesh-scoped)

    keep MeshArtifact as the mesh-governed artifact identity (so you don’t lose it)

    drop mirrored mesh-specific file classes/properties (LocatedMeshFile, AbstractMeshFile, hasAbstractMeshFile, etc.)

        instead: use LocatedFile / AbstractFile everywhere, and type specific in-mesh files as both LocatedFile and MeshResource when needed (e.g., ResourcePage)

    recommend removing MeshComponent/KnopComponent/NomenComponent as redundant if “components are now just artifacts,” and move slot/cardinality enforcement to SHACL (qualified constraints on the single containment predicate)

    rename ArtifactHost → MeshContainer or SfloContainer (since “artifact” is no longer your governance signifier)

If you want one further slimming move without sacrificing clarity: keep containsPayload as the only convenience subproperty, and make everything else “contains + type.”
You said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
We could keep ArtifactHost (or is ArtifactContainer better?) as a general concept, something that contains Artifacts but isn't an artifact itself. It's better than MeshContainer (because a Mesh being a MeshContainer is weird, especially when Knops and Nomen are also MeshContainers). 

The problem is you don't get a clean domain for containsMeshResource. I guess we could define a new class SfloArtifactHost that's subclass ArtifactHost and MeshResource. It's mostly a convenience class for specifying the domain so it doesn't matter as much that the name is horrible.

I really have zero concern about payloads being an external artifact. The purpose of a Knop is to host a payload, so why would it be external?

I'd be okay keeping containsPayload, but I was actually going to get rid of it anyhow because we have containsFlowPayload and containsSimplePayload, i.e., it's an abstract class that would rarely get used. Right now containsPayload's 
  rdfs:range <DigitalArtifact/>  -- further evidence of flabbiness. What do you think?
Do we need MeshArtifact? We could just use <DigitalArtifact/>, <MeshResource/>
ChatGPT said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
Goal

Keep MeshResource as the short, pronounceable “in-mesh governed” marker superclass, while:

    keeping DigitalArtifact truly general (not mesh-scoped),

    slimming the ontology (fewer mirrored mesh-specific file/containment terms),

    avoiding awkward “Sflo…” prefixing,

    and reflecting the updated decision that Nomens do not structurally contain other Nomens.

Core stance changes (based on latest discussion)

    No mesh-specific file classes/properties: use AbstractFile, LocatedFile, hasAbstractFile, hasLocatedFile universally; type in-mesh files as MeshResource when needed.

    Components are removable if they add no semantics beyond “hosted in a slot”; prefer SHACL-qualified cardinalities.

    Containment properties should not be over-engineered for RDFS domain/range; SHACL is where constraints live.

Naming cleanup: containers/hosts

You want a general “thing that hosts artifacts but is not an artifact itself.” That is coherent.
Option 1 (slimmest): Keep ArtifactHost, but don’t over-constrain it

    ArtifactHost is a general structural role.

    In Semantic Flow meshes, Mesh, Knop, Nomen are all MeshResource and also ArtifactHost.

<ArtifactHost/> a rdfs:Class ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself an artifact identity." .

<Mesh/>  rdfs:subClassOf <MeshResource/>, <ArtifactHost/> .
<Knop/>  rdfs:subClassOf <MeshResource/>, <ArtifactHost/> .
<Nomen/> rdfs:subClassOf <MeshResource/>, <ArtifactHost/> .
Containment: one predicate, SHACL does the slot cardinalities

Keep a single containment predicate; name stays concise:
<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment for in-mesh governed resources." .

Notes:

    Domain can be omitted (recommended) to avoid forcing you into awkward host-class naming.

    SHACL can then state: “Knop must contain exactly one FlowPayload or exactly one SimplePayload, exactly one KnopMetadataFlow, etc.” using qualified value shapes.

About containsPayload

If you always assert only containsFlowPayload and containsSimplePayload, then containsPayload provides value only if:

    you run RDFS entailment, or

    you materialize containsPayload triples as a convenience view.

Otherwise, users still need property paths (rdfs:subPropertyOf*) to query “any payload,” so the property is mostly vocabulary flab.

Recommendation: **drop **containsPayload and keep only the two concrete properties.
Do you need MeshArtifact?

You can drop it if you are comfortable repeating two superclass axioms:
<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "An artifact identity that may exist inside or outside any mesh." .

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <MeshResource/> ;
  rdfs:label "Flow artifact" .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <MeshResource/>, <Realizable/> ;
  rdfs:label "Simple artifact" .

That makes “mesh-governed artifact” an implicit intersection (DigitalArtifact ∧ MeshResource) without naming it.

When MeshArtifact is worth keeping:

    you expect additional artifact-identity subclasses beyond Flow/Simple,

    you want a single range target for properties,

    you want a named concept for documentation/UI.

Given your “slimmer ontology” priority and small current set, dropping MeshArtifact is reasonable.
Nomen hierarchy

Remove “Nomen contains Nomen” as structural containment. If you keep any hierarchy link, make it explicitly derived/operational (prefix/path based), not containment.
What this version optimizes for

    Keeps the best short name: MeshResource.

    Avoids the “Sflo…” naming tax.

    Eliminates mirrored mesh file vocabulary.

    Lets SHACL do the real work for slot cardinalities.

    Keeps vocabulary surface area small: a few core classes + a few key containment predicates.

ArtifactHost vs MeshContainer / ArtifactContainer

    ArtifactHost is better than MeshContainer for exactly the reason you gave: it reads as a role (“hosts artifacts”) rather than “is a container for meshes.” A Mesh being an ArtifactHost is not weird; a Mesh being a MeshContainer sounds like “container of meshes” (even if you didn’t mean that).

    ArtifactContainer is fine too, but “Host” is slightly better because it implies “holds/serves/manages,” not just “contains.”

The underlying point: you’re trying to name a common supertype for Mesh, Knop, Nomen that is not itself an artifact identity. “Host” captures that.
“Clean domain” for containsMeshResource is not worth an ugly class name

You don’t actually need a named SfloArtifactHost (or similar) just to have a tidy rdfs:domain.

Two reasons:

    rdfs:domain is inference, not constraint. If someone asserts :x containsMeshResource :y, domain just types :x as whatever you put there. It doesn’t prevent misuse.

    SHACL is your constraint layer anyway. If the point is “only Mesh/Knop/Nomen can be subjects,” express that in SHACL shapes (or simply by convention + tests).

So: omit the domain (or set it loosely), and let SHACL enforce “allowed subjects + slot cardinalities.”
Payloads forced to be MeshResource: yes, that’s fine

Given your stance (“a Knop exists to host a payload”), it’s consistent that payload artifacts are mesh-governed. So containsFlowPayload ⊑ containsMeshResource and containsSimplePayload ⊑ containsMeshResource are fine.
Should you drop containsPayload?

Yes—unless you’re going to materialize it.

If you only ever assert containsFlowPayload and containsSimplePayload, then querying ?k containsPayload ?p won’t return anything unless:

    the store does RDFS entailment, or

    you add explicit containsPayload triples as a convenience view, or

    you use a subproperty-closure query pattern.

So as pure vocabulary, containsPayload is mostly flab, and the fact its range drifted to DigitalArtifact is another signal that it’s not anchored.

Recommendation:

    Drop containsPayload.

    Keep only containsFlowPayload and containsSimplePayload.

    In docs, show “payload” queries as either:

        two patterns, or

        one pattern with a VALUES type filter (if you unify predicates), or

        a property-path idiom (if you keep a payload superproperty but don’t materialize it).

Do you need MeshArtifact?

You can remove it.

If you only have two artifact identity subclasses (FlowArtifact, SimpleArtifact), it’s not crazy to repeat:

    FlowArtifact ⊑ DigitalArtifact ⊓ MeshResource

    SimpleArtifact ⊑ DigitalArtifact ⊓ MeshResource ⊓ Realizable

That saves a class and removes “MeshArtifact vs MeshResource” ambiguity in the reader’s head.

When MeshArtifact becomes worth reintroducing:

    you add more artifact identity kinds (Series, Bundle, Index, etc.)

    you want a single range target in properties/shapes/docs

Given your current “slim” goal: drop it now, re-add only if the taxonomy grows.

I updated the patch canvas to reflect this direction: keep MeshResource, keep ArtifactHost, omit the domain on containsMeshResource, drop containsPayload, and make “mesh-governed artifact” implicit via FlowArtifact/SimpleArtifact being both DigitalArtifact and MeshResource.
You said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
We'll take Option 1. Can you make all the changes to the ontology?

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <https://schema.org/> .
@prefix vann: <http://purl.org/vocab/vann/> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .


# Core Semantic Mesh-scoped resources

<MeshResource/> a rdfs:Class ;
  rdfs:label "Mesh resource" ;
  rdfs:comment "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves" .

<ArtifactHost/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A mesh-governed resource that can contain artifacts and associated structural resources. Nomen are ArtifactHosts that can also contain other Nomen. " .

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains mesh-governed resources and, optionally, other supporting resources." .

<meshBase/> a owl:DatatypeProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "Mesh Base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

<MeshComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Mesh component" ;
  rdfs:comment "A reserved, mesh-scoped structural resource (e.g., mesh-level metadata, mesh inventory)." .  

<containsMeshComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshComponent/> ;
  rdfs:label "contains mesh component" ;
  rdfs:comment "Associates a Mesh with one of its directly contained MeshComponents." .

<containsKnopComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopComponent/> ;
  rdfs:label "contains knop component" ;
  rdfs:comment "Associates a Knop with one of its directly contained KnopComponents." .

<containsNomenComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenComponent/> ;
  rdfs:label "contains nomen component" ;
  rdfs:comment "Associates a Nomen with one of its directly contained NomenComponents." .

  
# General-purpose Resources
# These could be usable anywhere on the Semantic Web, not just in Semantic Flow meshes'

## Digital Artifact

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A mesh-governed artifact identity (something treated as a digital artifact in the abstract)." .

## Realizable + Realizations

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A content-bearing resource that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Examples include SimpleArtifacts (where the identity is itself the content bearer) and Slices (revision states of a FlowArtifact). FlowArtifacts are identity anchors and are not themselves realizable revision states." . .


<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding specification for a realizable content (e.g., an abstract file or stream), independent of any particular retrieval endpoint." .


## Abstract files and streams

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-form realization specification (format/syntax/canonicalization/bundling). May be mesh-governed or external." .

<AbstractStream/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract stream" ;
  rdfs:comment "A non-file realization specification (e.g., a stream or service-like packaging), reserved for future use." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractFiles." .

<hasAbstractStream/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractStream/> ;
  rdfs:label "has abstract stream" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractStreams." .

## Located files (byte retrieval)

<LocatedFile/> a rdfs:Class ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes endpoint (typically with an extension), which may be in-mesh or external." .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  owl:inverseOf <locatesAbstractFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile/> ;
  rdfs:range <AbstractFile/> ;
  owl:inverseOf <hasLocatedFile/> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .


## Resource Pages

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "A derived UI/navigation file (e.g., index.html) that presents a mesh-governed resource (or other referent) to humans." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource (including referents) with a human-readable presentation page." .

# Semantic Flow Artifacts

## Digital Artifacts kinds

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A DigitalArtifact whose content evolves over time through Slices. The FlowArtifact is an identity anchor, not itself a realizable revision state." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A DigitalArtifact treated as a single content unit with no revision history. The SimpleArtifact resource itself is the realizable content bearer." .

## Artifact Files

<LocatedArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/> ;
  rdfs:label "Located artifact file" ;
  rdfs:comment "A mesh-governed LocatedFile that provides concrete bytes for a mesh-governed abstract artifact file." .

<AbstractArtifactFile/> a rdfs:Class ;
  rdfs:subClassOf <AbstractFile/>, <MeshResource/> ;
  rdfs:label "Abstract artifact file" ;
  rdfs:comment "A mesh-governed AbstractFile that participates in Semantic Flow's artifact modeling." .

<hasAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasAbstractFile/> ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractArtifactFile/> ;
  rdfs:label "has abstract artifact file" ;
  rdfs:comment "Links a Realizable resource to one of its mesh-governed AbstractArtifactFiles." .

<hasLocatedArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasLocatedFile/> ;
  rdfs:domain <AbstractArtifactFile/> ;
  rdfs:range <LocatedArtifactFile/> ;
  owl:inverseOf <locatesAbstractArtifactFile/> ;
  rdfs:label "has located artifact file" ;
  rdfs:comment "Links an AbstractArtifactFile to one or more mesh-governed LocatedArtifactFiles." .

<locatesAbstractArtifactFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <locatesAbstractFile/> ;
  rdfs:domain <LocatedArtifactFile/> ;
  rdfs:range <AbstractArtifactFile/> ;
  owl:inverseOf <hasLocatedArtifactFile/> ;
  rdfs:label "locates abstract artifact file" ;
  rdfs:comment "Navigation: links a LocatedArtifactFile to the AbstractArtifactFile whose bytes it provides." .


# Slices

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A particular revision member of a FlowArtifact. A Slice is content-bearing/realizable but is not a DigitalArtifact identity." .

<WorkingSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Working slice" ;
  rdfs:comment "Mutable current state for a FlowArtifact." .

<HistoricalSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Historical slice" ;
  rdfs:comment "Immutable published snapshot for a FlowArtifact." .

<hasSlice/> a owl:ObjectProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <Slice/> ;
  rdfs:label "has slice" ;
  rdfs:comment "Associates a FlowArtifact with one of its Slices." .

<currentSlice/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasSlice/> ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <WorkingSlice/> ;
  rdfs:label "current slice" ;
  rdfs:comment "Associates a FlowArtifact with its current Slice." .

<previousSlice/> a owl:ObjectProperty ;
  rdfs:domain <Slice/> ;
  rdfs:range <Slice/> ;
  rdfs:label "previous slice" ;
  rdfs:comment "Links to the immediately-preceding Slice of this Slice." .

<nextSliceSequenceNumber/> a owl:DatatypeProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range xsd:integer ;
  rdfs:label "next slice sequence number" ;
  rdfs:comment "Monotonic integer providing the next allocatable sequence number for a FlowArtifact." .


# Naming

<Nomen/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about. A Nomen may also function as a namespace prefix for subordinate Nomens under its path." .

<containsNomen/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Nomen/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<NomenComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Nomen component" ;
  rdfs:comment "Reserved components under a Nomen (e.g., nomen flows, inventories)." .

<designatorPath/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Mesh-relative path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root Nomen." .

<designates/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for namespacing / token management)." .

<hasChildNomen/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "has child nomen" ;
  rdfs:comment "Operational/derived listing of subordinate Nomens under this Nomen's designator's namespace path. Not a source of truth; implementations may derive children from mesh structure and/or designatorPath prefix rules." .


### Reference data links

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a MeshResource to a specific reference data source, including role metadata." .

<ReferenceRole/> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical/> a <ReferenceRole/> ;
  rdfs:label "Canonical" .

<ReferenceRole/Integrate/> a <ReferenceRole/> ;
  rdfs:label "Integrate into resource page" .

<ReferenceRole/Supplemental/> a <ReferenceRole/> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated/> a <ReferenceRole/> ;
  rdfs:label "Deprecated" .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <MeshResource/> ;
  rdfs:range <ReferenceLink/> ;
  owl:inverseOf <referenceLinkFor/> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a mesh-governed resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <MeshResource/> ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the mesh-governed resource that a ReferenceLink is about." .

<hasReferenceRole/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ReferenceRole/> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a node used as a reference-data identifier (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

<referenceUriLiteral/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

### Verification metadata

<lastVerifiedAt/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .


# Knops + flows

<Knop/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactHost/> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for DigitalArtifacts anchored at a stable in-mesh location; hosts payload and supporting FlowArtifacts and SimpleArtifacts." .

<containsKnop/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Knop/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<KnopComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Knop component" ;
  rdfs:comment "Reserved components under a Knop (payload/supporting flows and their parts)." .

<FlowPayload/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload flow" ;
  rdfs:comment "The primary FlowArtifact hosted by a Knop." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, <KnopComponent/> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

<containsPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "contains payload" ;
  rdfs:comment "Links a Knop to its primary payload artifact identity (either a FlowArtifact-based payload or a SimpleArtifact-based payload)." .

<containsFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "contains flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<containsSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "contains simple payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a SimpleArtifact (SimplePayload)." .

# Metadata

<KnopMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <KnopComponent/>, <RdfDataset/> ;
  rdfs:label "Knop metadata flow" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop." .

<MeshMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <MeshComponent/>, <RdfDataset/> ;
  rdfs:label "Mesh metadata flow" ;
  rdfs:comment "Optional mesh-level operational metadata/config." .

<NomenMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <NomenComponent/>, <RdfDataset/> ;
  rdfs:label "Nomen metadata flow" ;
  rdfs:comment "Metadata about a Nomen; may remain WorkingSlice-only until history is recorded." .

<containsKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "contains knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<containsMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "contains mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<containsNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "contains nomen metadata flow" ;
  rdfs:comment "Links a Nomen to its nomen metadata flow (if present)." .


# RDF datasets 

<RdfDataset/> a rdfs:Class ;
  rdfs:subClassOf dcat:Dataset ;
  rdfs:label "RDF dataset" ;
  rdfs:comment "An Artifact whose content is an RDF dataset (default graph + 0..n named graphs)." .


# Inventories

<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <MeshComponent/>, <SimpleArtifact/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <KnopComponent/>, <SimpleArtifact/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <NomenComponent/>, <SimpleArtifact/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .
ChatGPT said:

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <https://schema.org/> .
@prefix vann: <http://purl.org/vocab/vann/> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .

#################################################################
# Core Semantic Mesh-scoped resources
#################################################################

<MeshResource/> a rdfs:Class ;
  rdfs:label "Mesh resource" ;
  rdfs:comment "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves." .

<ArtifactHost/> a rdfs:Class ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself an artifact identity." .

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <ArtifactHost/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains mesh-governed resources and, optionally, other supporting resources." .

<meshBase/> a owl:DatatypeProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "mesh base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

#################################################################
# General-purpose resources
#################################################################

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A digital-artifact identity that may exist inside or outside any Semantic Flow mesh." .

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A content-bearing resource that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Examples include SimpleArtifacts (where the identity is itself the content bearer) and Slices (revision states of a FlowArtifact). FlowArtifacts are identity anchors and are not themselves realizable revision states." .

<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding specification for a realizable content (e.g., an abstract file or stream), independent of any particular retrieval endpoint." .

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-form realization specification (format/syntax/canonicalization/bundling). May be mesh-governed or external." .

<AbstractStream/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract stream" ;
  rdfs:comment "A non-file realization specification (e.g., a stream or service-like packaging), reserved for future use." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractFiles." .

<hasAbstractStream/> a owl:ObjectProperty ;
  rdfs:domain <Realizable/> ;
  rdfs:range <AbstractStream/> ;
  rdfs:label "has abstract stream" ;
  rdfs:comment "Links a Realizable resource to one of its AbstractStreams." .

<LocatedFile/> a rdfs:Class ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes endpoint (typically with an extension), which may be in-mesh or external." .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  owl:inverseOf <locatesAbstractFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile/> ;
  rdfs:range <AbstractFile/> ;
  owl:inverseOf <hasLocatedFile/> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .

#################################################################
# Resource pages
#################################################################

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, <MeshResource/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "A derived UI/navigation file (e.g., index.html) that presents a mesh-governed resource (or other referent) to humans." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource (including referents) with a human-readable presentation page." .

#################################################################
# Semantic Flow artifacts (mesh-governed artifact identities)
#################################################################

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <MeshResource/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A DigitalArtifact whose content evolves over time through Slices. The FlowArtifact is an identity anchor, not itself a realizable revision state." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <MeshResource/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A DigitalArtifact treated as a single content unit with no revision history. The SimpleArtifact resource itself is the realizable content bearer." .

#################################################################
# Slices
#################################################################

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A particular revision member of a FlowArtifact. A Slice is content-bearing/realizable but is not a DigitalArtifact identity." .

<WorkingSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Working slice" ;
  rdfs:comment "Mutable current state for a FlowArtifact." .

<HistoricalSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Historical slice" ;
  rdfs:comment "Immutable published snapshot for a FlowArtifact." .

<hasSlice/> a owl:ObjectProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <Slice/> ;
  rdfs:label "has slice" ;
  rdfs:comment "Associates a FlowArtifact with one of its Slices." .

<currentSlice/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasSlice/> ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range <WorkingSlice/> ;
  rdfs:label "current slice" ;
  rdfs:comment "Associates a FlowArtifact with its current Slice." .

<previousSlice/> a owl:ObjectProperty ;
  rdfs:domain <Slice/> ;
  rdfs:range <Slice/> ;
  rdfs:label "previous slice" ;
  rdfs:comment "Links to the immediately-preceding Slice of this Slice." .

<nextSliceSequenceNumber/> a owl:DatatypeProperty ;
  rdfs:domain <FlowArtifact/> ;
  rdfs:range xsd:integer ;
  rdfs:label "next slice sequence number" ;
  rdfs:comment "Monotonic integer providing the next allocatable sequence number for a FlowArtifact." .

#################################################################
# Naming
#################################################################

<Nomen/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <ArtifactHost/> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about." .

<containsNomen/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Nomen/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<designatorPath/> a owl:DatatypeProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Mesh-relative path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root Nomen." .

<designates/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for token management)." .

<hasChildNomen/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "has child nomen" ;
  rdfs:comment "Operational/derived listing of Nomens whose designatorPath falls under this Nomen's designatorPath by prefix rules. Not a structural containment relation and not a source of truth." .

#################################################################
# Reference data links
#################################################################

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a MeshResource to a specific reference data source, including role metadata." .

<ReferenceRole/> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical/> a <ReferenceRole/> ;
  rdfs:label "Canonical" .

<ReferenceRole/Integrate/> a <ReferenceRole/> ;
  rdfs:label "Integrate into resource page" .

<ReferenceRole/Supplemental/> a <ReferenceRole/> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated/> a <ReferenceRole/> ;
  rdfs:label "Deprecated" .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <MeshResource/> ;
  rdfs:range <ReferenceLink/> ;
  owl:inverseOf <referenceLinkFor/> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a mesh-governed resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <MeshResource/> ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the mesh-governed resource that a ReferenceLink is about." .

<hasReferenceRole/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ReferenceRole/> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a node used as a reference-data identifier (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

<referenceUriLiteral/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

<lastVerifiedAt/> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .

#################################################################
# Knops + flows
#################################################################

<Knop/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/>, <ArtifactHost/> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for mesh-governed artifact identities anchored at a stable in-mesh location; hosts a primary payload and supporting artifacts." .

<containsKnop/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Knop/> ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<FlowPayload/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/> ;
  rdfs:label "Payload flow" ;
  rdfs:comment "The primary FlowArtifact hosted by a Knop." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

<containsFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "contains flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<containsSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "contains simple payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a SimpleArtifact (SimplePayload)." .

#################################################################
# RDF datasets (artifact kind)
#################################################################

<RdfDataset/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, dcat:Dataset ;
  rdfs:label "RDF dataset" ;
  rdfs:comment "A DigitalArtifact whose content is an RDF dataset (default graph + 0..n named graphs)." .

#################################################################
# Metadata artifacts
#################################################################

<KnopMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> ;
  rdfs:label "Knop metadata flow" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop." .

<MeshMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> ;
  rdfs:label "Mesh metadata flow" ;
  rdfs:comment "Optional mesh-level operational metadata/config." .

<NomenMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/> ;
  rdfs:label "Nomen metadata flow" ;
  rdfs:comment "Metadata about a Nomen; may remain WorkingSlice-only until history is recorded." .

<containsKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "contains knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<containsMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "contains mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<containsNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "contains nomen metadata flow" ;
  rdfs:comment "Links a Nomen to its nomen metadata flow (if present)." .

#################################################################
# Inventories (derived datasets)
#################################################################

<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .

You said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
You forgot that SimplePayload is also subclass Realizable. Anything else we missed? Any lingering questions?
ChatGPT said:

You’re right to call it out, but there are two different notions here:

    Semantically: in the version I returned, SimplePayload ⊑ SimpleArtifact and SimpleArtifact ⊑ Realizable, so SimplePayload is already a Realizable by entailment.

    Practically / maintainability: if you want that to remain true even if you later refactor SimpleArtifact, then you should state it explicitly on SimplePayload.

Below is the corrected excerpt with the explicit axiom, plus the other Option-1 fix I missed (dropping the domain on containsMeshResource, since we agreed not to over-constrain it just for “clean” domains).

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  # (Option 1) omit domain; SHACL constrains allowed subjects
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, <Realizable/> ;  # explicit (redundant but stable)
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

Lingering issues / questions worth deciding now (they’ll bite later if left fuzzy):

    Do you want disjointness axioms?
    Minimally: FlowArtifact ⊓ SimpleArtifact ⊑ ⊥ and maybe DigitalArtifact ⊓ Slice ⊑ ⊥ (since slices are revision states, not identities). If you don’t add these, mistakes won’t surface.

    Are you intentionally allowing a Knop to contain both payload kinds?
    Right now OWL doesn’t prevent it. If you mean “exactly one of (FlowPayload, SimplePayload)”, that’s SHACL territory; just confirm you’re committing to SHACL for that invariant.

    RdfDataset as a class vs “kind”
    You’re currently using class membership (“this artifact is an RdfDataset”) as the kind marker. That’s fine, but then avoid resurrecting artifactKind unless you really want both mechanisms.

    In-mesh files without mesh-specific file classes
    If you want to say “this LocatedFile is in-mesh governed”, you’ll need to type it as MeshResource (since LocatedFile is no longer mesh-scoped). That’s consistent with the “no mesh-specific file vocabulary” decision—just be aware it’s now a modeling convention you must follow.

If you want, I can re-emit the entire consolidated Turtle with these two fixes applied (domain removal + explicit SimplePayload ⊑ Realizable).
You said:
Semantic Flow Ontology Patch: Mesh-governed Artifacts And Mesh Files
I think having domain ArtifactHost on containsMeshResource is fine. Don't you? Only if we get rid of ArtifactHost do we need to remove it as the domain. And we should keep ArtifactHost, I think. Just as a concept, it underscores the 3 main ingredients in a Mesh have something in common. My preference for ArtifactContainer was due to all those "contains*" properties. Might still be worth going that route. They don't really serve, and "manages" implies a more active role than they have too. I've changed it to ArtifactContainer

No need to re-emit the TTL. We need to re-do the SHACL. Feel free to do it in a canvas if that's preferable to you.

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-shacl/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "vann": "http://purl.org/vocab/vann/",
    "sh": "http://www.w3.org/ns/shacl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "dcat": "http://www.w3.org/ns/dcat#",
    "dcterms": "http://purl.org/dc/terms/",
    "formats": "http://www.w3.org/ns/formats/",
    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/"
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow SHACL Shapes",
      "dcterms:description": "SHACL shapes for validating Semantic Flow data structures and operational invariants.",
      "vann:preferredNamespacePrefix": "sflo-shacl",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-shacl/"
    },
    {
      "@id": "MeshBaseShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Mesh base sanity",
      "rdfs:comment": "Ensures a Mesh typically declares a canonical meshBase URI literal.",
      "sh:targetClass": {
        "@id": "sflo:Mesh/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:meshBase/"
        },
        "sh:minCount": 1,
        "sh:maxCount": 1,
        "sh:datatype": {
          "@id": "xsd:anyURI"
        },
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A Mesh SHOULD declare exactly one sflo:meshBase (xsd:anyURI)."
      }
    },
    {
      "@id": "FlowArtifactStructureShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "FlowArtifact structure constraints",
      "rdfs:comment": "Basic structural constraints for FlowArtifact / slice membership.",
      "sh:targetClass": {
        "@id": "sflo:FlowArtifact/"
      },
      "sh:property": [
        {
          "sh:path": {
            "@id": "sflo:hasSlice/"
          },
          "sh:minCount": 1,
          "sh:severity": {
            "@id": "sh:Warning"
          },
          "sh:message": "A FlowArtifact SHOULD have at least one sflo:hasSlice."
        },
        {
          "sh:path": {
            "@id": "sflo:currentSlice/"
          },
          "sh:maxCount": 1,
          "sh:class": {
            "@id": "sflo:WorkingSlice/"
          },
          "sh:severity": {
            "@id": "sh:Violation"
          },
          "sh:message": "If present, a FlowArtifact's sflo:currentSlice MUST be a sflo:WorkingSlice (and at most one value)."
        }
      ],
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "If sflo:currentSlice is present, its value MUST also be one of the FlowArtifact's sflo:hasSlice values.",
        "sh:severity": {
          "@id": "sh:Violation"
        },
        "sh:select": "SELECT $this WHERE {\n  $this sflo:currentSlice ?cs .\n  FILTER NOT EXISTS { $this sflo:hasSlice ?cs }\n}"
      }
    },
    {
      "@id": "SliceRealizableShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Slice realization presence",
      "rdfs:comment": "Slices are content-bearing; they should usually have at least one AbstractFile realization.",
      "sh:targetClass": {
        "@id": "sflo:Slice/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:hasAbstractFile/"
        },
        "sh:minCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A Slice SHOULD have at least one sflo:hasAbstractFile (realization spec)."
      }
    },
    {
      "@id": "SimpleArtifactRealizableShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "SimpleArtifact realization presence",
      "rdfs:comment": "SimpleArtifacts are realizable; they should usually have at least one AbstractFile realization.",
      "sh:targetClass": {
        "@id": "sflo:SimpleArtifact/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:hasAbstractFile/"
        },
        "sh:minCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A SimpleArtifact SHOULD have at least one sflo:hasAbstractFile (realization spec)."
      }
    },
    {
      "@id": "AbstractFileLocatedFileShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractFile should be locatable (usually)",
      "rdfs:comment": "In many meshes, AbstractFiles are expected to have at least one LocatedFile. This is a warning because external/non-locatable realization specs may exist.",
      "sh:targetClass": {
        "@id": "sflo:AbstractFile/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:hasLocatedFile/"
        },
        "sh:minCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "An AbstractFile SHOULD have at least one sflo:hasLocatedFile."
      }
    },
    {
      "@id": "AbstractArtifactFileLocatedArtifactFileShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractArtifactFile locatability",
      "rdfs:comment": "Mesh-governed abstract artifact files should typically be backed by at least one mesh-governed located artifact file.",
      "sh:targetClass": {
        "@id": "sflo:AbstractArtifactFile/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:hasLocatedArtifactFile/"
        },
        "sh:minCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "An AbstractArtifactFile SHOULD have at least one sflo:hasLocatedArtifactFile."
      }
    },
    {
      "@id": "LocatedArtifactFileBacklinkShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "LocatedArtifactFile backlink",
      "rdfs:comment": "Located artifact files should usually indicate which abstract artifact file they locate.",
      "sh:targetClass": {
        "@id": "sflo:LocatedArtifactFile/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:locatesAbstractArtifactFile/"
        },
        "sh:minCount": 1,
        "sh:maxCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A LocatedArtifactFile SHOULD locate exactly one AbstractArtifactFile via sflo:locatesAbstractArtifactFile."
      }
    },
    {
      "@id": "KnopPayloadChoiceShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop payload choice",
      "rdfs:comment": "A Knop should have exactly one payload form: either a flow payload or a simple payload.",
      "sh:targetClass": {
        "@id": "sflo:Knop/"
      },
      "sh:xone": [
        {
          "@type": "sh:NodeShape",
          "sh:property": [
            {
              "sh:path": {
                "@id": "sflo:containsFlowPayload/"
              },
              "sh:minCount": 1,
              "sh:maxCount": 1
            },
            {
              "sh:path": {
                "@id": "sflo:containsSimplePayload/"
              },
              "sh:maxCount": 0
            }
          ]
        },
        {
          "@type": "sh:NodeShape",
          "sh:property": [
            {
              "sh:path": {
                "@id": "sflo:containsSimplePayload/"
              },
              "sh:minCount": 1,
              "sh:maxCount": 1
            },
            {
              "sh:path": {
                "@id": "sflo:containsFlowPayload/"
              },
              "sh:maxCount": 0
            }
          ]
        }
      ],
      "sh:message": "A Knop MUST have exactly one of: sflo:containsFlowPayload or sflo:containsSimplePayload (and not both)."
    },
    {
      "@id": "KnopMetadataFlowShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop metadata flow presence",
      "rdfs:comment": "Operationally recommended: a Knop should have exactly one metadata flow.",
      "sh:targetClass": {
        "@id": "sflo:Knop/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:containsKnopMetadataFlow/"
        },
        "sh:minCount": 1,
        "sh:maxCount": 1,
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A Knop SHOULD have exactly one sflo:containsKnopMetadataFlow."
      }
    },
    {
      "@id": "NomenReferenceLinksShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Nomen reference links",
      "rdfs:comment": "Basic expectations for Nomen reference-link usage.",
      "sh:targetClass": {
        "@id": "sflo:Nomen/"
      },
      "sh:property": [
        {
          "sh:path": {
            "@id": "sflo:hasReferenceLink/"
          },
          "sh:class": {
            "@id": "sflo:ReferenceLink/"
          },
          "sh:severity": {
            "@id": "sh:Violation"
          },
          "sh:message": "Values of sflo:hasReferenceLink must be typed sflo:ReferenceLink."
        },
        {
          "sh:path": {
            "@id": "sflo:hasChildNomen/"
          },
          "sh:severity": {
            "@id": "sh:Info"
          },
          "sh:message": "sflo:hasChildNomen is operational/derived; validate only if present."
        }
      ]
    },
    {
      "@id": "ReferenceLinkBackpointerShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "ReferenceLink backpointer",
      "rdfs:comment": "A ReferenceLink should identify the thing it is about.",
      "sh:targetClass": {
        "@id": "sflo:ReferenceLink/"
      },
      "sh:property": {
        "sh:path": {
          "@id": "sflo:referenceLinkFor/"
        },
        "sh:minCount": 1,
        "sh:maxCount": 1,
        "sh:class": {
          "@id": "sflo:MeshResource/"
        },
        "sh:severity": {
          "@id": "sh:Warning"
        },
        "sh:message": "A ReferenceLink SHOULD point back to exactly one sflo:MeshResource via sflo:referenceLinkFor."
      }
    }
  ]
}
ChatGPT said:

    Yes: adding SimplePayload ⊑ Realizable means it must be covered by your “realization presence” SHACL. The main thing you were missing is that SHACL sh:targetClass does not automatically apply to subclasses unless you run inference. So if your instances are typed only as sflo:SimplePayload (not also sflo:SimpleArtifact), your existing SimpleArtifactRealizableShape would silently skip them.

    I updated the SHACL in the canvas (“Semantic Flow SHACL Patch: ArtifactContainer + Containment Slots”) to address that directly:

        SimpleArtifactRealizableShape now explicitly targets SimplePayload (and the inventory dataset leaf classes) so it catches them without RDFS inference.

        FlowArtifactStructureShape is now targeted by property usage (sh:targetSubjectsOf sflo:hasSlice/) so it catches FlowPayload (and any future FlowArtifact-subclasses) without requiring you to also type them as sflo:FlowArtifact.

    I also removed a bad constraint I introduced: I had (incorrectly) constrained inventory datasets to ≤ 1. That contradicts your own “large/sharded” intent.

Lingering questions / risk points (worth deciding explicitly):

    Are you running pySHACL with RDFS/OWL inference? If “no,” you need to keep doing what the patch does: target leaf classes and/or targetSubjectsOf rather than relying on superclass targeting.

    Do you actually want inventories to be 0..n (shardable) now, or do you want a transitional “recommend 1” posture? (Right now the patch does not cap them.)

    Severity choices: should “missing KnopMetadataFlow” be Warning (ops recommendation) or Violation (structural invariant)? Your earlier language oscillates between “exactly one component” vs “optional later,” so SHACL severity is where that ambiguity will bite you first.

    Domain on containsMeshResource: I agree it’s fine if you accept the inference side-effect: any subject using that predicate becomes an ArtifactContainer under RDFS semantics. If that’s acceptable (it sounds like it is), keep it.

You said:
Semantic Flow Shacl Patch: Artifact Container + Containment Slots
I don't want to assume inference for the SHACL. 

Can you explain with examples targetSubjectsOf

Can you also explain "Slot constraints are qualified-by-class rather than “only these are allowed.” This avoids turning the ontology into a closed-world registry."

I think inventory datasets should be exactly one. Both it and missing KnopMetadataFlow should be violation.

You can take the notes and pre-amble out of the canvas so that it's only SHACL.
ChatGPT said:
Semantic Flow Shacl Patch: Artifact Container + Containment Slots

{ "@context": { "@base": "https://semantic-flow.github.io/ontology/semantic-flow-shacl/", "owl": "http://www.w3.org/2002/07/owl#", "vann": "http://purl.org/vocab/vann/", "sh": "http://www.w3.org/ns/shacl#", "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#", "rdfs": "http://www.w3.org/2000/01/rdf-schema#", "xsd": "http://www.w3.org/2001/XMLSchema#", "dcat": "http://www.w3.org/ns/dcat#", "dcterms": "http://purl.org/dc/terms/", "schema": "https://schema.org/", "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/" }, "@graph": [ { "@id": "", "@type": "owl:Ontology", "dcterms:title": "Semantic Flow SHACL Shapes", "dcterms:description": "SHACL shapes for validating Semantic Flow data structures and operational invariants.", "vann:preferredNamespacePrefix": "sflo-shacl", "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-shacl/" },
{
  "@id": "MeshBaseShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Mesh base sanity",
  "rdfs:comment": "Ensures a Mesh typically declares a canonical meshBase URI literal.",
  "sh:targetClass": "sflo:Mesh/",
  "sh:property": {
    "sh:path": "sflo:meshBase/",
    "sh:minCount": 1,
    "sh:maxCount": 1,
    "sh:datatype": "xsd:anyURI",
    "sh:severity": "sh:Warning",
    "sh:message": "A Mesh SHOULD declare exactly one sflo:meshBase (xsd:anyURI)."
  }
},

{
  "@id": "NomenDesignatorPathShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Nomen designatorPath",
  "rdfs:comment": "Nomens must have exactly one mesh-relative designatorPath token.",
  "sh:targetClass": "sflo:Nomen/",
  "sh:property": {
    "sh:path": "sflo:designatorPath/",
    "sh:minCount": 1,
    "sh:maxCount": 1,
    "sh:datatype": "xsd:string",
    "sh:severity": "sh:Violation",
    "sh:message": "A Nomen MUST declare exactly one sflo:designatorPath (xsd:string)."
  }
},

{
  "@id": "FlowArtifactStructureShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "FlowArtifact structure",

SELECT $this WHERE { $this sflo:currentSlice ?cs . FILTER NOT EXISTS { $this sflo:hasSlice ?cs } }" } },
{
  "@id": "SliceRealizableShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Slice realization presence",
  "rdfs:comment": "Slices are content-bearing; they should usually have at least one AbstractFile realization.",
  "sh:targetClass": "sflo:Slice/",
  "sh:property": {
    "sh:path": "sflo:hasAbstractFile/",
    "sh:minCount": 1,
    "sh:severity": "sh:Warning",
    "sh:message": "A Slice SHOULD have at least one sflo:hasAbstractFile (realization spec)."
  }
},

{
  "@id": "SimpleRealizableShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Simple realizable realization presence",
  "rdfs:comment": "Simple realizables should usually have at least one AbstractFile realization. Explicitly targets leaf classes to avoid requiring RDFS inference.",
  "sh:targetClass": [
    "sflo:SimpleArtifact/",
    "sflo:SimplePayload/",
    "sflo:MeshInventoryDataset/",
    "sflo:KnopInventoryDataset/",
    "sflo:NomenInventoryDataset/"
  ],
  "sh:property": {
    "sh:path": "sflo:hasAbstractFile/",
    "sh:minCount": 1,
    "sh:severity": "sh:Warning",
    "sh:message": "A realizable SHOULD have at least one sflo:hasAbstractFile (realization spec)."
  }
},

{
  "@id": "AbstractFileLocatedFileShape/",

SELECT $this WHERE { $this sflo:hasReferenceLink ?rl . FILTER NOT EXISTS { ?rl sflo:referenceLinkFor $this } }" } },
{
  "@id": "KnopPayloadChoiceShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Knop payload choice",
  "rdfs:comment": "A Knop must have exactly one payload form: either FlowPayload or SimplePayload (accepts either specific properties or the generic containment predicate).",
  "sh:targetClass": "sflo:Knop/",
  "sh:xone": [
    {
      "@type": "sh:NodeShape",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsFlowPayload/",
              "sflo:containsMeshResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:FlowPayload/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsSimplePayload/",
              "sflo:containsMeshResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:SimplePayload/" },
          "sh:qualifiedMaxCount": 0
        }
      ]
    },
    {
      "@type": "sh:NodeShape",
      "sh:property": [

] }
sh:targetSubjectsOf (with examples)

sh:targetSubjectsOf <P> targets every node that appears as a subject of predicate P in the data, regardless of its rdf:type.

Example data:

ex:flow1 sflo:hasSlice ex:slice1 .
ex:slice1 a sflo:WorkingSlice .

If you have a shape:

    sh:targetSubjectsOf sflo:hasSlice

then the shape targets:

    ex:flow1 (because it is a subject of sflo:hasSlice)

This is useful when you don’t want to assume inference or even that the instance is explicitly typed as sflo:FlowArtifact. It’s a “trigger by usage” pattern: “if you use the slot, you are in-scope for the constraint.”

Contrast with sh:targetClass sflo:FlowArtifact: that only targets nodes explicitly typed as that class (unless you turn on inferencing).

(There’s a symmetric sh:targetObjectsOf, which targets nodes that appear as objects of a given predicate.)
“Slot constraints are qualified-by-class rather than ‘only these are allowed.’”

This is about what you’re validating:
A) Qualified-by-class (what you want here)

You say: “Among the values of containment, there must be exactly one value of class X.”

Example (Knop contains lots of things):

ex:knop1 sflo:containsMeshResource ex:payload1, ex:meta1, ex:thumb1 .

ex:meta1 a sflo:KnopMetadataFlow .
ex:thumb1 a ex:ThumbnailImage .

A qualified constraint like:

    “Exactly 1 contained thing that is a sflo:KnopMetadataFlow”

passes, even though ex:thumb1 is some new/unknown future type. This keeps the model extensible.
B) “Only these are allowed” (closed-world registry)

You instead try to enforce: “A Knop may contain only these types and nothing else.”

That means any future addition (e.g., ThumbnailImage, AuditLog, DiagnosticsDataset) becomes a validation failure until you update SHACL. That turns your framework into a closed list of allowed contained types, which is usually a bad fit for your “general framework” goal.

So: qualified-by-class checks your “required slots exist with correct cardinality” without forbidding additional contained things.
Shape changes applied (and why)

    No inference assumed: the FlowArtifact constraints are targeted with sh:targetSubjectsOf sflo:hasSlice (so they trigger on usage, not type entailment).

    Inventory datasets: now exactly one each for Mesh/Knop/Nomen, severity = Violation.

    Missing KnopMetadataFlow: now Violation (exactly one).

    Canvas now contains only the SHACL JSON-LD, with the preamble/notes removed.

    I also added explicit PREFIX sflo: lines in SPARQL constraints so you’re not relying on engine-specific prefix injection from JSON-LD context.

Lingering question (because it affects SHACL severity policy): you made inventory exactly one and KnopMetadataFlow required. Do you also want MeshMetadataFlow and NomenMetadataFlow to be required (Violation) rather than optional (currently “at most one”, Info)?


ChatGPT can make mistakes. Check important info.
