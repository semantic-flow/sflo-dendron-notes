---
id: gvwcl9de67qklcqh8qg9ere
title: 2026 01 30 Artifact Identity and Realizabilit
desc: ''
updated: 1769833101441
created: 1769833091079
---

Skip to content
Chat history
You said:
I can't find the session where we talked about making a <Slice/> a subClassOf <DigitalArtifact/>. I feel like you wanted a bunch of changes, but maybe just comment changes?

Here's the current ontology:

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
@prefix prov: <http://www.w3.org/ns/prov#> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .

#################################################################
#################################################################
# General-purpose Vocabulary
#################################################################
#################################################################

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A digital-artifact identity that may exist inside or outside any Semantic Flow mesh." .

<ArtifactContainer/> a rdfs:Class ;
  rdfs:label "Artifact container" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself an artifact identity." .

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A content-bearing resource that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Examples include SimpleArtifacts (where the identity is itself the content bearer) and Slices (revision states of a FlowArtifact). FlowArtifacts are identity anchors and are not themselves realizable revision states." .

<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A representation/encoding specification for a realizable content (e.g., an abstract file or stream), independent of any particular retrieval endpoint." .

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <Realization/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-form realization specification (format/syntax/canonicalization/bundling)." .

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
  rdfs:comment "A retrievable bytes endpoint (typically with an extension)." .

<LocatedStream/> a rdfs:Class ;
  rdfs:label "Located stream" ;
  rdfs:comment "A bytes endpoint that is not file-like (e.g., a stream or service-like packaging)" .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  owl:inverseOf <locatesAbstractFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<hasLocatedStream/> a owl:ObjectProperty ;
  rdfs:domain <AbstractStream/> ;
  rdfs:range <LocatedStream/> ;
  rdfs:label "has located stream" ;
  rdfs:comment "Links an AbstractStream to one or more LocatedStreams that provide streamable bytes." .

<locatesAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile/> ;
  rdfs:range <AbstractFile/> ;
  owl:inverseOf <hasLocatedFile/> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .

#################################################################
# Artifact kinds
#################################################################

<FlowArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Flow artifact" ;
  rdfs:comment "A DigitalArtifact whose content evolves over time through Slices. The FlowArtifact is an identity anchor, not itself a realizable revision state." .

<SimpleArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Simple artifact" ;
  rdfs:comment "A DigitalArtifact treated as a single content unit with no revision history. The SimpleArtifact resource itself is the realizable content bearer." .

<RdfDataset/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, dcat:Dataset ;
  rdfs:label "RDF dataset" ;
  rdfs:comment "A DigitalArtifact whose content is an RDF dataset (default graph + 0..n named graphs)." .


#################################################################
# Slices
#################################################################

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A particular revision member of a FlowArtifact. A Slice is a realizable DigitalArtifact with its own identity, i.e., each slice has its own identity, which is not the same identity as the FlowArtifact that it slices." .

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
  rdfs:subPropertyOf prov:wasRevisionOf ;
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
# Resource pages
#################################################################

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "An HTML page that presents a resource." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource with a human-readable presentation page." .

#################################################################
# Reference data
#################################################################

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a resource to a specific reference data source, including role metadata." .

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
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLink/> ;
  owl:inverseOf <referenceLinkFor/> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the resource that a ReferenceLink is about." .

<hasReferenceRole/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ReferenceRole/> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a resource used as a reference-data identifier." .

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
#################################################################
# Semantic Flow Resources
#################################################################
#################################################################

<SemanticFlowResource/> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A resource governed by Semantic Flow conventions." .

<containsSemanticFlowResource/> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactContainer/> ;
  rdfs:range <SemanticFlowResource/> ;
  rdfs:label "contains Semantic Flow resource" ;
  rdfs:comment "Structural containment relation for Semantic Flow resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

#################################################################
# Semantic Meshes
#################################################################

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource/>, <ArtifactContainer/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains Semantic Flow resources and other supporting resources." .

<meshBase/> a owl:DatatypeProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "mesh base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

#################################################################
# Naming
#################################################################

<Nomen/> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource/>, <ArtifactContainer/> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about." .

<containsNomen/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Nomen/> ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
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


#################################################################
# Knops + Payloads
#################################################################

<Knop/> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource/>, <ArtifactContainer/> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for payload artifacts and supporting artifacts." .

<containsKnop/> a owl:ObjectProperty ;
  rdfs:domain <Mesh/> ;
  rdfs:range <Knop/> ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<FlowPayload/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <SemanticFlowResource/> ;
  rdfs:label "Payload flow" ;
  rdfs:comment "The primary FlowArtifact hosted by a Knop." .

<SimplePayload/> a rdfs:Class ;
  rdfs:subClassOf <SimpleArtifact/>, <SemanticFlowResource/> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary (simple) artifact hosted directly by a Knop, for cases where the payload is not managed as a FlowArtifact." .

<containsFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "contains flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<containsSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "contains simple payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a SimpleArtifact (SimplePayload)." .

<preferredPayloadSlug/> a owl:DatatypeProperty ;
  rdfs:domain <Knop/> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload slug" ;
  rdfs:comment "Optional human-friendly file-stem hint for naming payload distribution files (e.g., 'alice-profile'). Not an identifier; does not affect IRIs or designator paths. Generators may sanitize/ignore it and must resolve collisions mechanically (e.g., suffix or hash)." .


#################################################################
# Metadata artifacts
#################################################################

<KnopMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/>, <SemanticFlowResource/> ;
  rdfs:label "Knop metadata flow" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop." .

<MeshMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/>, <SemanticFlowResource/> ;
  rdfs:label "Mesh metadata flow" ;
  rdfs:comment "Optional mesh-level operational metadata/config." .

<NomenMetadataFlow/> a rdfs:Class ;
  rdfs:subClassOf <FlowArtifact/>, <RdfDataset/>, <SemanticFlowResource/> ;
  rdfs:label "Nomen metadata flow" ;
  rdfs:comment "Metadata about a Nomen; may remain WorkingSlice-only until history is recorded." .

<containsKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "contains knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<containsMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "contains mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<containsNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "contains nomen metadata flow" ;
  rdfs:comment "Links a Nomen to its nomen metadata flow (if present)." .

#################################################################
# Inventory artifacts 
#################################################################

<MeshInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/>, <SemanticFlowResource/> ;
  rdfs:label "Mesh inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)." .

<KnopInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/>, <SemanticFlowResource/> ;
  rdfs:label "Knop inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)." .

<NomenInventoryDataset/> a rdfs:Class ;
  rdfs:subClassOf <RdfDataset/>, <SimpleArtifact/>, <SemanticFlowResource/> ;
  rdfs:label "Nomen inventory dataset" ;
  rdfs:comment "Optional derived index for explorability at the nomen level (non-flow; may be large/sharded)." .

<containsMeshInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshInventoryDataset/> ;
  rdfs:label "contains mesh inventory dataset" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryDatasets (may be large and/or sharded)." .

<containsKnopInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopInventoryDataset/> ;
  rdfs:label "contains knop inventory dataset" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryDatasets (may be large and/or sharded)." .

<containsNomenInventoryDataset/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenInventoryDataset/> ;
  rdfs:label "contains nomen inventory dataset" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryDatasets (may be large and/or sharded)." .
ChatGPT said:

You already have the axiom you’re looking for:

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  ...

So if the goal was “make <Slice/> a subclass of <DigitalArtifact/>”, it’s already done.
The real problem is your comments, not the subclassing

Right now, <Slice/> is described as “a particular revision member”, but you also define <WorkingSlice/> as mutable. A mutable “revision member” is a contradiction in plain English (and will confuse implementers).
Minimal, comment-only fix (what you suspected)

I’d change the Slice-family comments to stop over-committing to “revision” at the supertype:

<Slice/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Slice" ;
  rdfs:comment "A realizable state of a FlowArtifact (draft or published). Each Slice has its own DigitalArtifact identity distinct from the FlowArtifact identity." .

<WorkingSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Working slice" ;
  rdfs:comment "A mutable draft slice of a FlowArtifact. May later be published as (or replaced by) a HistoricalSlice." .

<HistoricalSlice/> a rdfs:Class ;
  rdfs:subClassOf <Slice/> ;
  rdfs:label "Historical slice" ;
  rdfs:comment "An immutable published slice of a FlowArtifact (a revision snapshot)." .

That resolves the “revision vs mutable” inconsistency cleanly.
If you remember “a bunch of changes,” here are the ones that actually fall out of your model
1) Your class lattice is internally inconsistent about whether “Realizable things are artifacts”

Your Realizable comment says it covers “content-bearing resources” including SimpleArtifacts and Slices—both of which you explicitly model as DigitalArtifact. But Realizable itself is not a DigitalArtifact.

If you mean “anything realizable is a digital artifact identity,” then do this:

<Realizable/> rdfs:subClassOf <DigitalArtifact/> .

…and then you can remove the redundant <DigitalArtifact/> superclass from <Slice/> and <SimpleArtifact/> (optional, but it makes the ontology cleaner and reduces drift).

If you don’t mean that, then your Realizable comment is misleading and should be tightened to clarify that Realizable is orthogonal to artifact identity.
2) previousSlice is underspecified relative to WorkingSlice

“Immediately-preceding Slice” is ambiguous when the current thing is mutable. You probably mean:

    for HistoricalSlice: previous historical revision

    for WorkingSlice: the most recent HistoricalSlice (or none)

That should be stated in the comment (at minimum), and ideally constrained in SHACL later.
3) Your ontology header has a real red flag

owl:versionIRI points into _working/.... A version IRI is supposed to identify a released version, not a working location. If this is intentional, it undermines the semantics of “version”. If it’s not intentional, it’s worth fixing sooner than later.
Recommendation

If you want the smallest safe change: do the comment-only update to the Slice hierarchy (first block). If you want the model to stop fighting itself: also make Realizable ⊑ DigitalArtifact (second block), or rewrite the Realizable comment so it no longer implies artifact identity.

If you paste your current intended semantics for “WorkingSlice lifecycle” (does it ever become historical vs is it replaced?), I can tighten previousSlice/currentSlice text so it stops being interpretive.
You said:
Following your thinking, I'm glad you're bringing up the DigitalArtifactness of Realizable. We use the term "artifact identity" as if artifacts had a single identity, but I don't think they do. DigitalArtifacts are multi-level. The FlowArtifact and SimpleArtifact identify the "most abstract" facet of their corresponding mutli-level DigitalArtifact kinds... but only a SimpleArtifact is realizable, i.e., has a single state. Slices are a single state so they are realizable too. Slices and SimpleArtifacts have a singular content-ness. But in the same way  that a "the-simpsons-superwork" Nomen could refer to an indistinct amalgamation of "the tv show", "the fictional family", "the video games", etc., the FlowArtifact refers to "the over-time identity" of an artifact.
ChatGPT said:

Two things in what you wrote don’t line up, and they matter for the ontology structure:

    You said “only a SimpleArtifact is realizable” and then immediately “Slices are a single state so they are realizable too.” You mean: Realizable = single-state content-bearing artifact identities, which includes SimpleArtifact and Slice, but excludes FlowArtifact.

    “Artifacts are multi-level” is fine, but don’t talk like they have a single identity. What you actually have is a family of related identities at different abstraction levels (over-time anchor vs state). That’s why the “DigitalArtifactness of Realizable” question keeps resurfacing: you’re mixing an identity taxonomy with a state/content taxonomy.

The clean axis (and the one your text implies)

    DigitalArtifact: any artifact identity at any abstraction level (over-time or single-state).

    FlowArtifact: the over-time identity (anchor). Not itself a single content state.

    Slice: a single-state identity (state) of a FlowArtifact.

    SimpleArtifact: a single-state identity where you choose not to model multiple states (and possibly where anchor==state).

Given that, yes: Realizable should be a subclass of DigitalArtifact if Realizable means “single-state artifact identity that can be realized.” Right now your comment for Realizable already implies that, but your axioms don’t.
Minimal ontology changes that make your story true (no new classes required)

    Make Realizable a DigitalArtifact and tighten its comment to “single-state”:

<Realizable/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Realizable" ;
  rdfs:comment "A single-state, content-bearing DigitalArtifact identity that can be realized in one or more representations (e.g., abstract files/streams and their retrieval endpoints). Slices and SimpleArtifacts are realizable; FlowArtifacts are not." .

    Then remove redundant DigitalArtifact superclasses if you want to reduce drift (optional):

<SimpleArtifact/> rdfs:subClassOf <Realizable/> .
<Slice/> rdfs:subClassOf <Realizable/> .

(Keeping them is harmless, but it increases the chance your comments/axioms diverge later.)

    Add the missing inverse link, because without it “multi-level identity” becomes hand-wavy:

<sliceOf/> a owl:ObjectProperty ;
  rdfs:domain <Slice/> ;
  rdfs:range <FlowArtifact/> ;
  owl:inverseOf <hasSlice/> ;
  rdfs:label "slice of" ;
  rdfs:comment "Links a Slice (single-state identity) to the FlowArtifact (over-time identity) it is a state of." .

Where your Simpsons analogy is dangerous (and how to keep it without breaking the model)

A Nomen can point at an “indistinct amalgamation” because that’s a naming surface and can be intentionally underdetermined.

But a FlowArtifact cannot safely be an amalgamation if you want slices to mean anything. If FlowArtifact is “TV show + fictional family + games + merch,” what is a Slice? An episode? A game release? A character bio? You lose the invariants that make “slice” useful.

If you need “the amalgam,” model it explicitly as something like a Collection/Concept (even just rdfs:Resource with a label), and let the Nomen designate that. Keep FlowArtifact for “one content-evolving thing with coherent states.”
One specific decision you should make (because it affects everything)

Do you want SimpleArtifact to be:

    (A) “single-state only; no over-time anchor exists,” or

    (B) “anchor and state coincide (same identity plays both roles)”?

(A) keeps the lattice clean.
(B) is pragmatically nice, but you should acknowledge the identity-collapse explicitly in the comment (and possibly via an axiom later).

If you pick (B), I would not try to model it as SimpleArtifact ⊑ FlowArtifact unless you also define what “hasSlice” means in that collapsed case (otherwise implementers will assume it has slices and be confused).

If you want, I can rewrite the FlowArtifact, Slice, and SimpleArtifact comments so they collectively enforce the “family of identities” idea without introducing any new classes.
You said:
"_working/" actually refers to the "current realizable state", not a draft, maybe something that will never be a slice.

So is a Realization also a digital artifact? and an AbstractFile? And a LocatedFile? And are they separate DigitalArtifacts? Or are they facets of a DigitalArtifact. 

For the specific decision, I think I've mostly been conceptualizing it as B. I would not try to model it as SimpleArtifact subclassOf FlowArtifact. But maybe we add another naming layer for the "anchor"? Because the state can vary over time, but there's still an idea of a singlular, abstract, time-varying thing.artifact. 
ChatGPT said:

1) _working/ cannot be owl:versionIRI if it’s not immutable

If _working/ means “the current realizable state” and it can move, then using it as owl:versionIRI is semantically wrong. In OWL, owl:versionIRI is for a specific version (i.e., stable, referencable, non-moving).

If you want _working/ to remain “head,” keep it, but do not hang it off owl:versionIRI. Do one of these instead:

Option A (recommended): versionIRI points at a historical slice; working is separate

<> a owl:Ontology ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <.../dss/_main/v1/> ;  # or whatever your immutable version IRI is
  sflo:workingCopy <.../_knops/_payload/_working/.../sflo.ttl> .

Option B: treat _working/ as “unversioned current” and never call it a version
Keep owl:versionIRI only when you publish; omit it while it’s moving.

Right now your header asserts “0.0.1” but points to _working/. That’s an integrity break: a stable label + a moving identifier.
2) Are Realization, AbstractFile, LocatedFile also DigitalArtifacts?

You’re mixing two uses of “artifact”:

    Artifact identity (the thing you care about citing, evolving, licensing, etc.)

    Facet/resources that support realization and access (representation specs and endpoints)

In RDF, they’re always separate resources (separate nodes). The question is whether they should be typed as DigitalArtifact.

Given your current story (“FlowArtifact is the over-time identity; Slice/SimpleArtifact are singular-state content identities”), the cleanest move is:
Treat Realization, AbstractFile, LocatedFile as facets, not DigitalArtifacts

Reason: if you make them DigitalArtifacts, you implicitly invite the question “do they have slices / over-time identity / content evolution?” and you will either (a) bloat the model or (b) lie by omission.

So I would not make them subclasses of DigitalArtifact unless you have a specific requirement like “located files themselves need revision tracking as first-class artifacts.”

What you can do (and what you’re already doing) is model them as separate nodes linked from a realizable identity:

    Realizable --hasAbstractFile--> AbstractFile --hasLocatedFile--> LocatedFile

That is already a facet structure.

If you want to make that explicit, add a facet superclass:

<ArtifactFacet/> a rdfs:Class ;
  rdfs:label "Artifact facet" ;
  rdfs:comment "A supporting resource that specifies or enables realization/access for a DigitalArtifact, without being the artifact identity itself." .

<Realization/> rdfs:subClassOf <ArtifactFacet/> .
<LocatedFile/> rdfs:subClassOf <ArtifactFacet/> .

This preserves your “DigitalArtifact = identity layer(s)” meaning.
3) Your Option B (SimpleArtifact as both anchor and state) needs an explicit anchor/state split

Right now you’re trying to express “multi-level identity” while only having one axis (DigitalArtifact) and one mixin (Realizable). That’s why you keep circling back.

Add two abstract classes:

<ArtifactAnchor/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact anchor" ;
  rdfs:comment "An over-time identity for an artifact. Not itself a single content state." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <Realizable/> ;  # and Realizable ⊑ DigitalArtifact (see below)
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state artifact identity that bears singular content." .

Then type your kinds:

<FlowArtifact/> rdfs:subClassOf <ArtifactAnchor/> .
<Slice/> rdfs:subClassOf <ArtifactState/> .
<SimpleArtifact/> rdfs:subClassOf <ArtifactAnchor/>, <ArtifactState/> .

This gives you B cleanly without claiming SimpleArtifact ⊑ FlowArtifact.

To make this consistent, do the one axiom you were already converging on:

<Realizable/> rdfs:subClassOf <DigitalArtifact/> .

and tighten the comment to “single-state content-bearing identity.”

Also add the inverse you still need for sanity:

<stateOf/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <ArtifactAnchor/> ;
  rdfs:label "state of" ;
  rdfs:comment "Links a single-state identity to the over-time anchor it is a state of." .

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> ;
  owl:inverseOf <stateOf/> ;
  rdfs:label "has state" .

Then keep hasSlice as a subproperty of hasState if you want.
4) Your “maybe never be a slice” claim conflicts with your current class tree

You currently have:

    WorkingSlice ⊑ Slice

    currentSlice range WorkingSlice

So either:

(1) Slice includes working states (draft/head), and “never be a slice” is just wrong
In that case: fix comments only.

(2) Slice means “recorded revision (immutable/published)” and working is not a slice
Then your current modeling is wrong and you should replace WorkingSlice with WorkingState (not a Slice), and adjust currentSlice.

Given you explicitly said _working/ might never become a slice, your semantics are actually leaning toward (2). But then you must stop calling it WorkingSlice and stop ranging currentSlice to it.

If you adopt the anchor/state split above, you can support both interpretations cleanly:

    ArtifactState = any state (including working)

    Slice = the subset of states you choose to record/publish (immutable), if you want that meaning

That avoids redefining half your ontology every time you think about _working/.
You said:
I want to figure out whether the anchor-state split is useful for SimpleArtifact. Whether WorkingSlice should be called something else and not considered a slice... then it could be shared, i.e., Both FlowArtifact and SimpleArtifact have a RealizableHead or ???. But I'm not sure that there is a split in FlowArtifact. The FlowArtifact IRI represents the DigitalArtifact over time. It's not so much an anchor as a different facet. 
ChatGPT said:

You’re using “facet” in two incompatible ways:

    Facet as supporting resource (Realization / AbstractFile / LocatedFile): representation/access facets.

    Facet as identity level (FlowArtifact vs Slice): diachronic vs synchronic identity.

If FlowArtifact is “a different facet” in the second sense, it is still an anchor-level identity in the only sense that matters for state tracking: it’s the identity that persists across state changes. The split exists whether you call it “anchor” or not—because FlowArtifact is not content-bearing, and the moment you need bytes you’re forced to talk about a state.
1) WorkingSlice: decide what “slice” means

Right now you want: “_working may never be a slice.” That implies:

    Slice = recorded/published revision (immutable, citeable).

    Working = head/live mutable state (may be ephemeral; may never be promoted).

If that’s your meaning, then WorkingSlice is misnamed and mis-typed. It should not be a subclass of Slice.

Minimal repair:

    Rename conceptually to WorkingState (or HeadState, LiveState).

    Make Slice mean published revisions only (keep HistoricalSlice or rename to PublishedSlice).

2) Anchor–state split: useful for SimpleArtifact only if you need uniform “head semantics”

You have three coherent options:
Option S1: SimpleArtifact is only a state (simplest)

    SimpleArtifact ⊑ Realizable

    No diachronic identity for simple things.

    Rule: “the artifact is its state.”

Cost: you lose a uniform “head state” interface across artifact kinds. If later you want revisions, you must migrate identity.
Option S2: SimpleArtifact is both diachronic and state (your B, but explicit)

    SimpleArtifact ⊑ DiachronicArtifact and SimpleArtifact ⊑ ArtifactState

    Conceptually: anchor and state coincide.

Cost: you will be tempted to assert headState points to itself, which is awkward to validate cleanly without SHACL-SPARQL. You can instead make it a rule (“for SimpleArtifact, head is itself”) and not assert triples.
Option S3: Always separate anchor and state (max uniformity)

    Introduce a distinct diachronic identity even for simple artifacts.

    SimpleArtifactAnchor + SimpleArtifactState.

Cost: more IRIs/nodes everywhere; heavier authoring.

Given your stated preference (B but not SimpleArtifact ⊑ FlowArtifact), S2 is the right conceptual stance, but you should probably implement it as: “SimpleArtifact is a state; treat its diachronic identity as implicit.” That gets you B without forcing self-links.
3) “Not sure there is a split in FlowArtifact” — there is, by your own semantics

If FlowArtifact denotes “the over-time identity,” then it cannot also be the thing with “singular content-ness.” So either:

    FlowArtifact is not Realizable (your current model), and therefore must have states to connect to bytes; or

    you collapse it into a state identity, which contradicts “over-time identity.”

So the split is mandatory; only the naming is optional. If “anchor” feels wrong, call it DiachronicArtifact and call states ArtifactState.
4) Concrete modeling that matches what you’re saying (and keeps FlowArtifact distinct from SimpleArtifact)

This is the smallest vocabulary addition that resolves the ambiguity:

<DiachronicArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Diachronic artifact" ;
  rdfs:comment "A DigitalArtifact identity that persists over time and may have one or more content states." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state, content-bearing DigitalArtifact identity." .

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "has state" ;
  rdfs:comment "Links a diachronic artifact identity to one of its single-state content identities." .

<headState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState/> ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "head state" ;
  rdfs:comment "Links to the current mutable or effective state (if any)." .

Then align your current classes:

    FlowArtifact ⊑ DiachronicArtifact

    Replace WorkingSlice with WorkingState ⊑ ArtifactState

    Make Slice mean published revisions: Slice ⊑ ArtifactState (or keep HistoricalSlice ⊑ Slice)

    hasSlice ⊑ hasState and range Slice (published only)

    currentSlice becomes headState (or currentState) and ranges WorkingState

For SimpleArtifact, pick S1 or S2:

    S1 (recommended for simplicity): SimpleArtifact ⊑ ArtifactState and you do not assert any diachronic identity.

    S2 (if you need uniformity): SimpleArtifact ⊑ DiachronicArtifact and SimpleArtifact ⊑ ArtifactState, but treat headState = self as a convention rather than asserted data.

5) The key litmus test

Do you need uniform operations across FlowArtifact and SimpleArtifact, like:

    “Give me the current bytes”

    “Tell me the current abstract file(s)”

    “Show me provenance and revision chain when it exists”

If yes, you want a DiachronicArtifact → headState pattern available everywhere (even if implicit for simple artifacts). If no, keep SimpleArtifact as just a state and don’t force the split.

If you answer that litmus test, the choice becomes mechanical.
You said:
Sorry, I meant "_working may never be a HistoricalSlice.”
ChatGPT said:

That correction removes the biggest inconsistency. With “_working may never be a HistoricalSlice,” your current typing of WorkingSlice ⊑ Slice is defensible, and you do not need to eject WorkingSlice from Slice just to make the language work.

The remaining question is purely: what does Slice mean?
1) If Slice = “any state of a FlowArtifact”

Then:

    Slice includes mutable and immutable states.

    WorkingSlice is a kind of Slice.

    HistoricalSlice is a kind of Slice.

In that case, rename/retune comments to stop implying “revision = immutable” at the supertype.

Suggested comment changes (minimal):

<Slice/> rdfs:comment "A single-state, realizable state of a FlowArtifact. Slices may be mutable (working) or immutable (historical/published). Each Slice has its own DigitalArtifact identity distinct from the FlowArtifact." .

<WorkingSlice/> rdfs:comment "The current mutable slice/state for a FlowArtifact. It may later be published as a HistoricalSlice (or replaced by another working slice)." .

<HistoricalSlice/> rdfs:comment "An immutable, published slice/state for a FlowArtifact." .

2) If Slice = “published revision only”

Then WorkingSlice should not be a Slice, and you’d need a separate class for head/draft state. But your correction indicates you’re not trying to say that.

Given your correction, (1) is the coherent reading, and it aligns with your existing class hierarchy.
3) Do you still need a diachronic vs state split?

Even with (1), you already have the split in practice:

    FlowArtifact: diachronic identity (over-time)

    Slice: synchronic identity (single-state, content-bearing)

You can call FlowArtifact “a different facet” if you like, but ontologically it is the diachronic identity class in the state model. The split is useful mainly if you want to generalize beyond flows (e.g., make headState work for SimpleArtifact too) without hard-coding Flow terms.
Minimal generalization (optional, low-risk)

Introduce just one abstract class and one property:

<DiachronicArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Diachronic artifact" ;
  rdfs:comment "A DigitalArtifact identity that persists over time and can have one or more single-state identities." .

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <Realizable/> ;
  rdfs:label "has state" ;
  rdfs:comment "Links a diachronic artifact identity to one of its single-state realizable identities." .

Then:

    FlowArtifact ⊑ DiachronicArtifact

    hasSlice ⊑ hasState

You can stop there.
4) SimpleArtifact and “head”

If you want “both FlowArtifact and SimpleArtifact have a head,” you have two clean options:

    Convention only (simplest): for SimpleArtifact, “the head is the artifact itself.” No triples needed.

    Explicit head property: define headState on DiachronicArtifact. For SimpleArtifact you either (a) don’t use it, or (b) allow self-reference as a permitted pattern.

I would not force self-reference into core unless you have a concrete consumer that needs it.

Bottom line: your corrected statement supports keeping WorkingSlice as a Slice; the main change is to scrub “revision member” language from Slice so it doesn’t imply immutability.
You said:
One thing I've had in the past, is everything is a Flow (FlowArtifact), and whether Slices get cut on weave is a configuration setting. Under that setup, the difference between FlowArtifact and SimpleArtifact is just whether any slice has ever been cut, you can always make a SimpleArtifact a FlowArtifact, and the FlowArtifact/SimpleArtifact  IRI is the diachronic identity anchor in the same way in both cases. Are Slices distinct DigitalArtifacts that never change? Is there a term for that? I'm definitely leaning to separate anchor and state, and Simple vs FlowArtifact has to be derived (i.e., not stored in data). But maybe we need to shift the paradigm a bit... Is it appropriate for the anchor IRI to also mean "the diachronic identity", 
ChatGPT said:

If “everything is a FlowArtifact and slicing-on-weave is a config,” then FlowArtifact vs SimpleArtifact is not a stable kind. It becomes a derived, history-dependent classification:

    “simple” = no historical slices have ever been cut (yet)

    “flow” = at least one historical slice exists

That pushes you toward an explicit anchor/state split, and it strongly argues against storing rdf:type SimpleArtifact vs rdf:type FlowArtifact in data.
1) Are slices distinct DigitalArtifacts that never change? What’s the term?

For the historical ones: yes—those are distinct identities for immutable states.

Common terms you can safely use (and that match your intent):

    snapshot

    revision

    versioned state

    immutable state token

In your vocabulary, HistoricalSlice is essentially “immutable snapshot/revision.” Your WorkingSlice is not that; it’s the mutable head.
Tight implication

If Slice includes both working + historical, then “slice” can’t mean “immutable snapshot” in general. Either:

    Slice = any state (mutable or immutable), and HistoricalSlice is the snapshot subtype; or

    Slice = immutable snapshot, and WorkingSlice must not be a Slice.

Given your clarified statement (“_working may never become a HistoricalSlice”), the first option is coherent and keeps your current hierarchy.
2) Your “everything is a flow” paradigm implies this minimal core model
Identity layer

    ArtifactAnchor: diachronic identity (the thing you cite over time)

    ArtifactState: synchronic identity (single-state content bearer)

State types

    WorkingState: mutable head

    HistoricalState: immutable snapshot (your HistoricalSlice)

This can be expressed with almost no new machinery:

<ArtifactAnchor/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact anchor" ;
  rdfs:comment "A diachronic DigitalArtifact identity that persists over time and may have one or more single-state content states." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/>, <Realizable/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state, realizable DigitalArtifact identity that bears singular content." .

<WorkingState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Working state" ;
  rdfs:comment "The current mutable state for an artifact anchor." .

<HistoricalState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published snapshot state." .

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> .

<headState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState/> ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <WorkingState/> .

Then you map your existing names:

    FlowArtifact ⊑ ArtifactAnchor

    Slice ⊑ ArtifactState

    WorkingSlice ⊑ WorkingState

    HistoricalSlice ⊑ HistoricalState

    hasSlice ⊑ hasState

    currentSlice is just headState (or keep currentSlice as Flow-specific alias)

3) Should the anchor IRI mean “the diachronic identity”?

Yes. That’s exactly what makes it an anchor. If you deny that, you lose the invariants that make “state” meaningful.

What you must avoid is accidentally making the anchor also look like it is the single-state content bearer. The easiest rule:

    Only states (ArtifactState / Slice) have hasAbstractFile / hasAbstractStream.

    The anchor only points to states (headState, hasState, etc.).

If you want dereferencing the anchor IRI to “give me the current bytes,” treat that as a view of the head state, not proof that the anchor is realizable. In other words: retrieval behavior can follow headState without asserting identity collapse.
4) Where does SimpleArtifact fit if you don’t store “simple vs flow”?

In the “everything is a flow” paradigm, SimpleArtifact becomes an interface illusion:

    It’s an ArtifactAnchor that currently has only a head state and no historical states.

    “Simple” is computed: NOT EXISTS { ?a sflo:hasState ?s . ?s a sflo:HistoricalState }

In OWL, that “absence” is not representable under open-world assumptions, so you should not try to encode “SimpleArtifact” as a negation-based class in the ontology. Treat it as derived in queries/SHACL/application logic.

If you still want the label “SimpleArtifact” for UI/ergonomics, make it a non-authoritative tag (or define it in SHACL as a target selection), not a truth-conditional type assertion.
5) The paradigm shift you’re circling

Drop “FlowArtifact vs SimpleArtifact” as core kinds. Make the core kinds:

    ArtifactAnchor (diachronic identity)

    ArtifactState (synchronic identity)

…and treat “flow-ness” as:

    policy/config (do we cut historical states on weave?)

    empirical history (has any historical state ever been cut?)

That fits your “always upgradeable” story without making “type” time-dependent.

If you want, I can rewrite just the Slice/WorkingSlice/HistoricalSlice comments and the Realizable comment so they enforce this model without introducing any new class names beyond ArtifactAnchor/ArtifactState (or reusing your existing FlowArtifact/Slice as the anchor/state names).
You said:
It seems like where we are headed is NOT FlowArtifact ⊑ ArtifactAnchor and Slice ⊑ ArtifactState, but getting rid of Flow/FlowArtifact and Slice altogether. Which is fine. I think we can then get rid of Realizable/Realization too. All ArtifactStates are realizable and nothing else is, and all AbstractFiles are realizations, and nothing else is. Streams aren't really Realizable or Realizations anyhow.
ChatGPT said:

Yes: if you introduce an explicit Anchor/State split, then FlowArtifact and Slice become redundant unless you want them as convenience aliases. And if ArtifactState is exactly “the realizable, content-bearing single state,” then Realizable collapses into ArtifactState. Same for Realization collapsing into AbstractFile (or a renamed “FileSpec”) if that’s the only realization type you actually mean to support.

There are two design hazards to avoid:

    Don’t delete vocabulary you still need conceptually. If you drop Realization but keep AbstractFile, you’re fine as long as you accept “realization” = “abstract file spec,” full stop.

    Streams: you’re right that your current AbstractStream/LocatedStream are not the same kind of thing as “realization of content.” They’re closer to access endpoints / services. If you keep them, model them as endpoints, not as realizations.

Minimal refactor that matches what you just said
Core identity model

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A digital artifact identity at any abstraction level (diachronic or single-state)." .

<ArtifactAnchor/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact anchor" ;
  rdfs:comment "A diachronic identity for an artifact over time. Not itself a single content state." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state, content-bearing identity for an artifact. All ArtifactStates are realizable." .

<WorkingState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Working state" ;
  rdfs:comment "The current mutable state for an artifact anchor." .

<HistoricalState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published snapshot state for an artifact anchor." .

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "has state" ;
  rdfs:comment "Associates an artifact anchor with one of its states." .

<headState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState/> ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <WorkingState/> ;
  rdfs:label "head state" ;
  rdfs:comment "Associates an artifact anchor with its current working state (if any)." .

<previousState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf prov:wasRevisionOf ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "previous state" ;
  rdfs:comment "Links a state to an immediately preceding state in a state lineage (typically used for HistoricalState)." .

This replaces FlowArtifact, Slice, WorkingSlice, HistoricalSlice, hasSlice, currentSlice, previousSlice cleanly.
Representation model (drop Realizable/Realization)

If “all AbstractFiles are realizations and nothing else is,” then you can simply do:

<AbstractFile/> a rdfs:Class ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-spec for realizing an ArtifactState (format/syntax/canonicalization/bundling), independent of any particular retrieval endpoint." .

<LocatedFile/> a rdfs:Class ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes endpoint (typically with an extension)." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Links an ArtifactState to one of its AbstractFiles." .

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

At this point, Realizable and Realization are genuinely unnecessary.
Streams: either delete or reclassify

If streams “aren’t realizations,” then AbstractStream as a subclass of “Realization” was doing the wrong conceptual job anyway. Options:

    Delete stream classes for now (simplest).

    Or keep them but rename/reframe as endpoints (not realizations), e.g. <LocatedEndpoint/> / <ServiceEndpoint/>, with properties like endpointUrl / acceptsMediaType—and link them from ArtifactState directly (or via a future “AccessMethod” relator).

The one inconsistency you should resolve while you’re here

<ResourcePage/> rdfs:subClassOf <LocatedFile/> currently makes a resource page a “retrievable bytes endpoint.” That’s fine mechanically, but it reinforces that LocatedFile is not a DigitalArtifact identity; it’s an access facet. If you ever felt tempted to make LocatedFile a DigitalArtifact, this is where it breaks.
Migration mapping (so you don’t lose meaning)

    FlowArtifact → ArtifactAnchor

    Slice → ArtifactState

    WorkingSlice → WorkingState

    HistoricalSlice → HistoricalState

    hasSlice → hasState

    currentSlice → headState

    previousSlice → previousState

    nextSliceSequenceNumber → (optional) nextStateSequenceNumber

If you want to keep old terms for readability/backward compatibility, you can keep them as subclasses or mark deprecated (comment-level), but the core semantics live in Anchor/State.

If you confirm “we are comfortable deleting streams for now,” the refactor becomes extremely small and internally consistent. If you want streams retained, you need to decide whether they’re (a) endpoint facets of a state, or (b) packaging specs like files. Right now your own statement implies (a).
You said:
Probably we should just be consistent with either WorkingState/workingState or HeadState/headState?

ResourcePage brings an important use case that includes external artifactual references: DigitalArtifacts may not have explicit anchors, state IRIs or even AbstractFile IRIs. So it'd be great to be able to say that, eg. <alice/index.html> a <DigitalArtifact>, <LocatedFile>. And it isn't an ArtifactState or AbstractFile?

And are ArtifactStates instances of <DigitalArtifact>? Are AbstractFile? I think so?
ChatGPT said:
1) Pick headState for the property; keep WorkingState for mutability

You don’t actually want HeadState as a class unless you’re willing to re-type resources every time “head” changes (or treat it as derived). “Head” is a role at a time, not an intrinsic kind.

So the consistent, low-regret pairing is:

    WorkingState (class): “mutable state”

    headState (property): “the anchor’s current effective state” (often a WorkingState)

If you really want perfectly mirrored naming, do it via a role relator later (e.g., HeadStateAssignment), not by retyping the state.
2) Your alice/index.html example exposes a definitional constraint

You said you want:

    <alice/index.html> a <DigitalArtifact>, <LocatedFile>. And it isn't an ArtifactState or AbstractFile?

If DigitalArtifact continues to mean “artifact identity” (your current comment: “digital-artifact identity”), then a thing that is only a bytes endpoint but not a state identity is not a DigitalArtifact. You can’t have it both ways without redefining DigitalArtifact.

You have two coherent options:
Option A (recommended): keep DigitalArtifact = identity, allow “collapsed identity” nodes

If you assert a located file IRI is the artifact identity, then it is an ArtifactState (a single-state identity). It just happens to also be a LocatedFile.

So your external case becomes:

<alice/index.html> a sflo:ArtifactState, sflo:LocatedFile .

No AbstractFile required. This matches your “DigitalArtifacts may not have anchors/states/AbstractFiles” requirement: it does have a state (itself), but no anchor or abstract file.

You do not make LocatedFile ⊑ DigitalArtifact. You allow overlap case-by-case.
Option B: broaden DigitalArtifact to include facets/endpoints

If you insist that a bytes endpoint can be a DigitalArtifact without being an identity/state, then DigitalArtifact can’t mean “artifact identity” anymore. You’d need:

    DigitalArtifact (broad umbrella)

    ArtifactIdentity ⊑ DigitalArtifact

    ArtifactFacet ⊑ DigitalArtifact

That’s a bigger semantic shift, and it will leak everywhere (resource pages, CSS, images, etc.).

Given your current intent, Option A is cleaner.
3) “Are ArtifactStates instances of DigitalArtifact? Are AbstractFile?”

If DigitalArtifact remains “artifact identity”:

    ArtifactState: yes, it should be ArtifactState ⊑ DigitalArtifact.

    AbstractFile: no, it should not be a DigitalArtifact identity. It’s a representation spec / facet.

If you make AbstractFile a DigitalArtifact, you implicitly invite “does it have states / head / history?” and you either bloat the model or create silent inconsistencies.

So keep it as a facet class.
4) Small TTL patch to support your external use case cleanly

Add a direct link from state → located file, because sometimes you don’t know/declare an AbstractFile:

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state, content-bearing artifact identity." .

<WorkingState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Working state" ;
  rdfs:comment "A mutable artifact state." .

<HistoricalState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState/> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published artifact state." .

<headState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "head state" ;
  rdfs:comment "The anchor's current effective state (often mutable)." .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Directly links a state identity to a retrievable bytes endpoint when no AbstractFile is modeled (or in addition to it)." .

You can keep your existing AbstractFile -> hasLocatedFile -> LocatedFile too; just rename that one to avoid collision (e.g., abstractFileHasLocatedFile) or keep it and name the new one stateHasLocatedFile. Right now you already have hasLocatedFile with domain AbstractFile, so you can’t also use it with domain ArtifactState without relaxing the domain.
5) ResourcePage implication

With Option A, you can still say:

    most generated pages: ResourcePage ⊑ LocatedFile only (not a DigitalArtifact identity)

    external artifact pages that are the identity: additionally type them as ArtifactState (and therefore DigitalArtifact)

That gives you the flexibility you want without turning every web asset into an “artifact identity.”

If you want, paste the current “Anchor/State” block you’ve drafted (or I can draft it from scratch against your existing ontology), and I’ll rewrite it so:

    domains/ranges don’t collide,

    “identity vs facet” is explicit in comments,

    and the external-located-file identity pattern is first-class without broadening DigitalArtifact.

You said:
Regarding #4, we could re-introduce Realizable and make AbstractFile and ArtifactState both Realizable. (Now Realizable means things that can have LocatedFiles). OR we keep hasLocatedFile with domain AbstractFile, and add a defaultLocatedFileForState property that could be used for ResourcePages to point to themselves AND help resolution from the ArtifactState level even if the ArtifactState does have AbstractFiles.

For #5, I kind of want to be able to refer to every web asset as a DigitalArtifact. It only make sense, after all. Same for AbstractFile. They are at once independent DigitalArtifacts and also potentially facets of more-broadly-conceptualized DigitalArtifacts
ChatGPT said:

You’re now choosing between two incompatible commitments:

    DigitalArtifact = “artifact identity layer(s)” (your earlier, tighter usage)

    DigitalArtifact = “any digital thing worth talking about, including endpoints, specs, pages, assets” (your new preference)

If you want (2), stop calling it “artifact identity” in the definition. Otherwise the ontology will keep contradicting itself.

Below is a coherent way to do what you want without losing the anchor/state model and without domain collisions.
A. Make DigitalArtifact broad; introduce ArtifactIdentity as the identity subset
Core change

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "Any digital thing that can be discussed, referenced, described, or linked to (identities, files, pages, specs, endpoints, assets)." .

<ArtifactIdentity/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact identity" ;
  rdfs:comment "A DigitalArtifact that serves as an identity anchor or a state identity for content." .

Then:

    ArtifactAnchor ⊑ ArtifactIdentity

    ArtifactState ⊑ ArtifactIdentity

    LocatedFile ⊑ DigitalArtifact (but not an identity by default)

    AbstractFile ⊑ DigitalArtifact (spec is a first-class artifact)

This matches your “everything is a DigitalArtifact” desire while preserving an identity subset for semantics like states/history.
B. Keep Realizable, but redefine it cleanly as “has retrievable bytes endpoints”

Your instinct is right: Realizable is useful, but only if it’s not muddled with “content-bearing” vs “identity” language.

Define:

<Realizable/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Realizable" ;
  rdfs:comment "A DigitalArtifact that can be realized as retrievable bytes via one or more LocatedFiles, either directly or through an AbstractFile specification." .

Then make both ArtifactState and AbstractFile realizable (as you proposed), because both can lead to bytes endpoints:

<ArtifactState/> rdfs:subClassOf <ArtifactIdentity/>, <Realizable/> .
<AbstractFile/> rdfs:subClassOf <Realizable/> .

Now Realizable is not redundant: it cleanly captures “can have bytes endpoints.”
C. Solve the domain collision with a relator or with two properties (pick one)
Option C1 (simplest): keep hasLocatedFile on AbstractFile and add state-level default

This is your defaultLocatedFileForState idea and it’s good because “default” is state-relative anyway.

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "has located file" ;
  rdfs:comment "Links an AbstractFile spec to one or more retrievable endpoints." .

<defaultLocatedFileForState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "default located file for state" ;
  rdfs:comment "Optional default retrievable endpoint for an ArtifactState. Useful for ResourcePages that point to themselves and for resolution when AbstractFiles are absent or multiple." .

This avoids reintroducing “stateHasLocatedFile” and keeps hasLocatedFile semantically about spec → endpoints.

If you want it to support “resolution even if the state does have AbstractFiles,” the comment should explicitly state that it can override/shortcut.
Option C2 (cleaner, more extensible): make a relator RealizationLink

If you want to capture why a located file is appropriate, media type, role, etc., you’ll want a relator anyway. But you didn’t ask for that right now, so C1 is probably the right minimal move.
D. What about ResourcePage pointing to itself?

Under C1:

<alice/index.html> a <ResourcePage/>, <LocatedFile/>, <DigitalArtifact/> ;
  <defaultLocatedFileForState/> <alice/index.html> .

But note: this triple only makes sense if <alice/index.html> is also an ArtifactState. If it’s merely a web page as an asset, “default located file for state” is the wrong predicate.

So you need a rule:

    If the resource page is the artifact state identity, type it as ArtifactState.

    If it’s merely a presentation of some other artifact, do not.

Example where the page presents Alice (a person), not itself:

<alice/index.html> a <ResourcePage/>, <LocatedFile/>, <DigitalArtifact/> .
<alice/> <hasResourcePage/> <alice/index.html> .

Example where the page is the artifact state (external web artifact whose identity is the page):

<alice/index.html> a <ResourcePage/>, <LocatedFile/>, <ArtifactState/> ;
  <defaultLocatedFileForState/> <alice/index.html> .

That distinction is unavoidable if you want the vocabulary to stay meaningful.
E. Consequences of “everything is a DigitalArtifact” (you should accept them explicitly)

If you make every web asset and AbstractFile a DigitalArtifact, then:

    DigitalArtifact is no longer a “kind” in the same sense as Anchor/State; it’s a top umbrella.

    You must avoid using DigitalArtifact in ways that imply content evolution, provenance, etc., unless you scope those to ArtifactIdentity or ArtifactState.

So: all your “over time / state / revision” predicates should have domain ArtifactIdentity or ArtifactAnchor, not DigitalArtifact.
F. Recommendation (most consistent with what you want now)

    Broaden DigitalArtifact and add ArtifactIdentity as the identity subset.

    Reintroduce Realizable as “can have bytes endpoints.”

    Keep hasLocatedFile domain AbstractFile.

    Add defaultLocatedFileForState for state-level shortcut and self-referential ResourcePages when the page is the state identity.

This gives you:

    “everything is a DigitalArtifact”

    anchor/state semantics remain tight

    no domain collisions

    direct resolution path for consumers

If you paste the exact block where you currently define DigitalArtifact, Realizable, Realization, AbstractFile, LocatedFile, and ResourcePage, I can rewrite just that section in-place (minimal diff) so the comments and domains/ranges stop fighting.
You said:
I don't think DigitalArtifact should include "any digital thing", only file-like things. Definitely not endpoints other than "file endpoints" (endpoint has the connotation of any device or point that connects to a network to exchange data); but yes, any document (spec, pages) or binary asset is a DigitalArtifact. Maybe we need a better term. ArtifactIdentity, just for identity. 

I'm still not sure we've captured the "A single DigitalArtifact can have diachronic identity, state identities, (abstract) file identities, and endpoint (don't like that overloaded term)/location identity"
ChatGPT said:

You’re still missing one layer in your vocabulary: you have kinds of nodes, but not a crisp account of the identity strata that can co-exist for what you informally call “one artifact.”

Right now you’re oscillating between:

    “DigitalArtifact = identity” (too narrow for your “every asset/spec/page is an artifact” requirement), and

    “DigitalArtifact = any digital thing” (too broad; pulls in non-file-ish network endpoints/devices).

1) Redefine DigitalArtifact as “file-like digital object,” not “any digital thing,” and stop saying “identity”

Use ArtifactIdentity for the identity layer.

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A file-like digital object (documents, specs, pages, datasets, binary assets), whether abstract or located." .

<ArtifactIdentity/> a rdfs:Class ;
  rdfs:label "Artifact identity" ;
  rdfs:comment "A DigitalArtifact that plays an identity role (diachronic or state-level) in an artifact's evolution." .

This matches your intent: specs/pages/assets are in DigitalArtifact, but you’re not committed to treating every file-ish thing as part of an evolution/history model unless you also type it as ArtifactIdentity.
2) Make the strata explicit as roles (and allow overlap)

This is the simplest way to capture “a single artifact can have diachronic identity, state identities, abstract file identities, and location identity” without inventing a mysterious “one true underlying thing.”

<ArtifactAnchor/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactIdentity/> ;
  rdfs:label "Artifact anchor" ;
  rdfs:comment "Diachronic identity for an artifact over time (the thing that persists across state changes)." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <ArtifactIdentity/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A single-state content identity (mutable or immutable)." .

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-spec artifact (format/syntax/canonicalization/bundling), independent of any particular location." .

<LocatedFile/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Located file" ;
  rdfs:comment "A file artifact identified as retrievable bytes at a particular location (avoid 'endpoint' language)." .

Key point: do not declare these disjoint. You want collapse cases to be legal:

    a resource can be both ArtifactState and LocatedFile (your alice/index.html case),

    a resource could even be ArtifactState and AbstractFile if you ever want “the spec is the state” (rare, but you said multi-level identities can coincide).

3) Use properties to connect strata, and add a state→location shortcut without changing domains

Keep your existing AbstractFile -> hasLocatedFile -> LocatedFile, and add a state-level shortcut that doesn’t collide:

<hasState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "has state" .

<headState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState/> ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "head state" ;
  rdfs:comment "Current effective state (often mutable)." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "has located file" .

<defaultLocatedFileForState/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "default located file for state" ;
  rdfs:comment "Optional shortcut/default retrieval location for a state, even when AbstractFiles exist." .

This gives you resolution from the state level without redefining hasLocatedFile and without reintroducing a broad Realizable unless you truly want it.
4) Your alice/index.html case becomes clean and non-hand-wavy

If the page is the artifact state identity:

<alice/index.html> a <ResourcePage/>, <LocatedFile/>, <ArtifactState/> ;
  <defaultLocatedFileForState/> <alice/index.html> .

If it’s merely a presentation of some other thing:

<alice/index.html> a <ResourcePage/>, <LocatedFile/> .
<alice/> <hasResourcePage/> <alice/index.html> .

That distinction matters. Otherwise every page becomes “the thing,” and your model stops meaning anything.
5) “Are ArtifactStates instances of DigitalArtifact? Are AbstractFile?”

Under this model:

    ArtifactState: yes, because ArtifactIdentity ⊑ DigitalArtifact (identity roles are still file-like artifacts in your worldview).

    AbstractFile: yes, because it’s a file-spec document/artifact.

    LocatedFile: yes, because it’s still a file-like artifact (just location-specific).

If you don’t like “identity roles are DigitalArtifacts,” then your earlier desire (“every asset/spec/page is a DigitalArtifact”) conflicts with “anchors aren’t file-like.” You need to pick. The above is the consistent pick.
6) Do you still need Realizable/Realization?

Not unless you want a third axis: “things that can have file locations.”

If you do want that axis, make it strictly about location-bearing:

    Realizable = “can have one or more LocatedFiles (directly or via AbstractFile)”

    ArtifactState ⊑ Realizable

    AbstractFile ⊑ Realizable

    LocatedFile is not Realizable (it is the location-specific file)

But you don’t need it if defaultLocatedFileForState exists and AbstractFile hasLocatedFile remains.
What you still haven’t named (but are implicitly using)

You keep saying “a single DigitalArtifact can have all these identities.” In the model above, the “single” thing is typically the Anchor (diachronic identity), and it links to other DigitalArtifacts (states, file-specs, located files). If you want a “whole family” that is not identical to the anchor (e.g., when there is no anchor), introduce an optional aggregator:

    ArtifactBundle / ArtifactConstellation with hasFacet to anchor/state/spec/location.

But don’t add it unless you can name a concrete use case where you have no anchor yet still want to assert “these belong together.”

If you want, I can rewrite just your current “Realizable/Realization/AbstractFile/LocatedFile/Slices” section into this structure with minimal diff, keeping your existing IRIs where possible.
You said:
Should've added "In addition to being part of a more-broadly-conceptualized DigitalArtifact, each state identity, file identity, and perhaps location identity is a distinct DigitalArtifacts of its own."

I like where you're going with #1, but ArtifactIdentity (and ArtifactAnchor) becomes an inadequate term name. It is specifically a DiachronicIdentity, not so much a singular identity or "the only possible anchor". Maybe the ArtifactIdentity becomes compositional and derived, depending on which facets exist.

We have four levels/facets of an Artifact: DiachronicIdentity, State, AbstractFile, LocalFile. If we want to say that a DiachronicIdentity must have States, maybe we want a fifth facet that is completely abstract, i.e., we know the artifact exists but we don't know if it has any States (so no DiachronicIdentity, AbstractFile, or LocatedFile, since those all depend on states.) That 5th state could be the ArtifactAnchor or AbstractArtifact or ???. 

We could have transitive+inverse properties like sflo:broaderIdentity and sflo:narrowerIdentity, with specialized subproperties for mapping between the levels. 

For things like ResourcePages, I think we accept violation of the single referent principle, and make their ArtifactAnchor IRI serve double-duty as the LocatedFile IRI. 

Somehow we want every level of an Artifact to be its own DigitalArtifact. e.g., a State is simultaneously a DigitalArtifact and also headState of a different DigitalArtifact.

ChatGPT said:

You can make this consistent, but you have to stop overloading DigitalArtifact with “identity” language.

Right now you want all of these to be true:

    DigitalArtifact applies to specs/pages/assets and also to diachronic/state identities.

    Each facet node is a distinct DigitalArtifact in its own right.

    Some DigitalArtifacts are “broader” (diachronic) and some “narrower” (state/file/location).

    Sometimes identities collapse (a resource page IRI is also the located-file IRI, maybe also the state IRI).

That is achievable if you model identity strata as roles/facets (classes), not as “the artifact itself.”
1) Redefine DigitalArtifact and introduce explicit facet classes

Change DigitalArtifact to “file-like digital object” (no “identity” wording):

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A file-like digital object (documents/specifications, pages, datasets, binary assets), whether abstract or location-specific." .

Then define the four facet levels as subclasses (not disjoint; overlap allowed):

<DiachronicArtifact/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Diachronic artifact" ;
  rdfs:comment "A DigitalArtifact viewed as an over-time identity that relates one or more state artifacts." .

<ArtifactState/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Artifact state" ;
  rdfs:comment "A DigitalArtifact viewed as a single-state content identity (mutable or immutable)." .

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A DigitalArtifact that is a file specification (format/syntax/canonicalization/bundling), independent of location." .

<LocatedFile/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Located file" ;
  rdfs:comment "A DigitalArtifact identified as retrievable bytes at a particular location (avoid 'endpoint')." .

This directly satisfies: “every level is its own DigitalArtifact.”
The “5th facet” you asked for is just DigitalArtifact with no facet typing

If you know “the artifact exists” but do not want to assert it has states yet, don’t type it as DiachronicArtifact. Leave it as just DigitalArtifact (or introduce a light tag like UnclassifiedArtifact, but it’s not necessary).

If you type it DiachronicArtifact, you are implicitly claiming it participates in the state relationship (even if SHACL, not OWL, enforces it).
2) Use broader/narrower as a generic refinement relation, then specialize per level

Your broaderIdentity / narrowerIdentity idea works, but name them artifact-wide (not “identity”) since files/locations are also artifacts:

<broaderArtifact/> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "broader artifact" ;
  rdfs:comment "Relates a narrower artifact facet to a broader artifact facet within the same artifact constellation." .

<narrowerArtifact/> a owl:ObjectProperty ;
  owl:inverseOf <broaderArtifact/> ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "narrower artifact" ;
  rdfs:comment "Inverse of broaderArtifact." .

Then add specialized subproperties that do the real work:

<hasState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerArtifact/> ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "has state" .

<headState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState/> ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <ArtifactState/> ;
  rdfs:label "head state" ;
  rdfs:comment "Current effective state (often mutable)." .

<hasAbstractFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerArtifact/> ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <AbstractFile/> ;
  rdfs:label "has abstract file" .

<hasLocatedFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerArtifact/> ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "has located file" .

This captures exactly what you described:

    A state is a DigitalArtifact and also a narrower facet of a diachronic DigitalArtifact.

    An abstract file is a DigitalArtifact and also narrower than a state.

    A located file is a DigitalArtifact and also narrower than an abstract file.

3) Add one state→location shortcut without breaking the AbstractFile domain

Keep hasLocatedFile as AbstractFile → LocatedFile and add your override/shortcut:

<defaultLocatedFileForState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerArtifact/> ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "default located file for state" ;
  rdfs:comment "Optional default retrieval location for a state, even when AbstractFiles exist (shortcut/override for resolution)." .

This avoids reintroducing Realizable purely to solve the domain collision.

If you still want Realizable, you can define it as a mixin meaning “can resolve to LocatedFiles,” and make both ArtifactState and AbstractFile subclasses of it. But it’s not necessary if you have defaultLocatedFileForState.
4) ResourcePages “double-duty” is intentional identity collapse, not a special case

If you want a resource page IRI to also be the located file IRI, you’re asserting they are the same resource. That’s fine if you accept collapse.

Example: page is also the state identity and points to itself as default location:

<alice/index.html> a <ResourcePage/>, <LocatedFile/>, <ArtifactState/> ;
  <defaultLocatedFileForState/> <alice/index.html> .

If it’s merely a presentation of some other artifact and not itself the artifact state identity, don’t type it as ArtifactState.
5) “DiachronicIdentity must have States” should be SHACL, not OWL

Because OWL won’t enforce existence constraints the way you want under open-world assumptions. Model the intent in SHACL:

    DiachronicArtifact must have sh:minCount 1 on hasState

    headState max 1

    HistoricalState / WorkingState split if you need immutability constraints

6) Naming issue: “ArtifactIdentity” is the wrong term for your model

Agreed. What you’re modeling is not “the identity” but facet strata.

If you want a term for “nodes that participate in this facet lattice,” use something like:

    ArtifactFacet (superclass of DiachronicArtifact / ArtifactState / AbstractFile / LocatedFile), or

    keep them directly under DigitalArtifact and rely on the broader/narrower relations.

The key is: don’t introduce ArtifactIdentity unless you mean “subset that is identity-bearing” (which would exclude LocatedFile if you sometimes want it to be “just a location facet”). But you also said located files may be independent artifacts; so “identity-bearing” won’t cleanly separate anything anyway.

This gets you the exact behavior you described:

    every level is a DigitalArtifact

    levels relate by broaderArtifact/narrowerArtifact plus typed subproperties

    diachronic/state/file/location can exist independently or as facets of a broader artifact

    identity collapse is allowed by multiple typing and optional self-referential defaults

If you want, I can rewrite your existing Slice/FlowArtifact/Realizable section into this lattice while keeping your existing IRIs wherever possible (e.g., Slice → ArtifactState, FlowArtifact → DiachronicArtifact, or deprecated aliases).
You said:
Would DigitalArtifact also be an ArtifactFacet? Do we want subproperties for skip-linking between any facet levels? maybe we keep defaultLocatedFileForState as "defaultLocatedFile" with domain DigitalArtifact, but it'd be a subproperty of narrowerArtifact, but I think those should be narrowerArtifactFacet. 
ChatGPT said:
1) Should DigitalArtifact also be an ArtifactFacet?

Only if ArtifactFacet adds real information. Otherwise it’s pure redundancy.

You have two coherent patterns:
Pattern A (recommended): No ArtifactFacet class

    Treat “facet-ness” as implicit: a DigitalArtifact is a facet when it participates in broaderArtifact / narrowerArtifact (or the level-specific subproperties).

    Keep broaderArtifact / narrowerArtifact domain+range as DigitalArtifact.

This avoids proliferating a “meta class” that you’ll forget to assert or maintain.
Pattern B: ArtifactFacet means “participates in facet lattice”

If you want an explicit marker, do not make DigitalArtifact ⊑ ArtifactFacet. Instead:

    ArtifactFacet ⊑ DigitalArtifact

    Intended meaning: “a DigitalArtifact that has at least one broader or narrower link”

You can even encode it as an equivalence (optional; not necessary if you don’t rely on OWL reasoning):

<ArtifactFacet/> a owl:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  owl:equivalentClass [
    a owl:Class ;
    owl:unionOf (
      [ a owl:Restriction ; owl:onProperty <broaderArtifact/> ; owl:someValuesFrom <DigitalArtifact/> ]
      [ a owl:Restriction ; owl:onProperty <narrowerArtifact/> ; owl:someValuesFrom <DigitalArtifact/> ]
    )
  ] .

This preserves your “every level is a DigitalArtifact” while making “facet-ness” derived.
2) Do you need skip-link subproperties between arbitrary facet levels?

You already get “skip-linking” for free if broaderArtifact is transitive:

    If State narrower AbstractFile and AbstractFile narrower LocatedFile, then State narrower LocatedFile is derivable via transitivity (as a generic relation).

So you don’t need additional generic properties.

Where you do want extra properties is when you need semantics beyond mere narrowing, e.g.:

    “this is the default retrieval location”

    “this is the current head state”

    “this state’s file-spec is canonical”

Those are not the same as “narrower,” so they deserve their own predicates.
3) Should defaultLocatedFile have domain DigitalArtifact?

Be careful: if you make it that general, the predicate becomes ambiguous across levels:

    For an AbstractFile, “default located file” means “preferred distribution endpoint”

    For a State, it means “preferred retrieval for this state, possibly overriding file specs”

    For a DiachronicArtifact, it means “preferred retrieval for the artifact as a whole” (usually “head state’s default”)

Those are related, but not identical. If you keep a single predicate, you’ll be forced into vague comments and consumers will misinterpret it.
Recommendation

Keep level-specific defaults, and (optionally) provide a generic superproperty for convenience:

<defaultLocatedFile/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "default located file" ;
  rdfs:comment "A preferred retrieval location. Subproperties should specify the facet level whose default is being declared." .

<defaultLocatedFileForAbstractFile/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <defaultLocatedFile/> ;
  rdfs:domain <AbstractFile/> ;
  rdfs:range <LocatedFile/> .

<defaultLocatedFileForState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <defaultLocatedFile/> ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> .

<defaultLocatedFileForDiachronicArtifact/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <defaultLocatedFile/> ;
  rdfs:domain <DiachronicArtifact/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:comment "Typically corresponds to the head state's default located file; asserted only when you want a direct shortcut." .

This preserves your desire for a single “default located file” handle, without collapsing three distinct meanings.
4) Should these be subproperties of narrowerArtifact?

Not automatically.

    defaultLocatedFileForState is not necessarily a “narrower facet” relation; it’s a preference/shortcut.

    If you make it a subproperty of narrowerArtifact, you are asserting a structural claim: “the located file is a narrower facet of the state.” That might be fine in your model, but it’s a stronger commitment than “this is the preferred retrieval.”

If you want structural narrowing to stay pure, keep narrowerArtifact for facet structure only, and don’t hang preference predicates under it.

If you do want preference to imply narrowing, then yes, make it a subproperty—but then you should also ensure every defaultLocatedFile* assertion is consistent with your facet lattice.
5) On “narrowerArtifactFacet”

If you keep ArtifactFacet, then it’s reasonable to restrict your lattice relations to facets:

<broaderFacet/> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <ArtifactFacet/> ;
  rdfs:range <ArtifactFacet/> ;
  owl:inverseOf <narrowerFacet/> .

<narrowerFacet/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactFacet/> ;
  rdfs:range <ArtifactFacet/> ;
  owl:inverseOf <broaderFacet/> .

But don’t do this unless you’re actually going to type things as ArtifactFacet (or define it equivalently as above). Otherwise you’ll have a taxonomy you can’t satisfy.

Net:

    Prefer Pattern A: drop ArtifactFacet entirely; keep the lattice on DigitalArtifact.

    Keep “skip-linking” via transitivity; add specialized properties only for semantics (head, default, canonical).

    Do not make defaultLocatedFileForState a subproperty of narrowerArtifact unless you explicitly want “default” to also imply a facet narrowing claim.

    If you want a generic defaultLocatedFile, keep it as a superproperty with level-specific subproperties to avoid semantic drift.

You said:
skip-linking with specific subproperties seems valuable.

Thinking about facets. DigtialArtifacts have facets. They aren't facets unless we overload the ArtifactAnchor IRI to also be "AbstractDigitalArtifactFacet". Ideally, any of an artifacts facet IRIs could also be used to anchor that artifact, but it's hard to use the narrower IRIs because they are facets of multiple digital artifacts. e.g. a LocatedFile is the anchor for itself-as-digital-artifact, but it is also potentially the locatedFileForAbstractFile for an AbstractFile DigitalArtifact, and a locatedFileForState for a State, and a locatedFileForDiachronicArtifact. It feels a bit like punning. 
ChatGPT said:

You are correctly diagnosing the core tension: you want each facet node to be its own DigitalArtifact, while also wanting “an artifact” as something that has facets. That is a classic “parts are also wholes” situation. It is not punning (in the OWL sense); it is a holonic / multi-membership modeling problem.

If you try to solve it by letting “any facet IRI anchor the whole artifact,” you will get ambiguity because a facet can belong to multiple “artifact wholes” (state→file→located file, but also reused specs, shared locations, etc.). That ambiguity is real, not a naming issue.
The missing construct is an explicit ArtifactWhole / Constellation node

Without an explicit “whole,” you can’t talk about “the artifact that has these facets” except by selecting one facet and pretending it’s privileged (your “overload the anchor IRI” move). That’s exactly why it feels like punning.

So the choice is:

    Choose a privileged facet as the whole (typically the diachronic IRI) and accept that some artifacts simply don’t have it; or

    Introduce an explicit whole that collects facets, and allow any facet to point to it.

Given your goal (“any facet IRI could be used to anchor that artifact”), you want (2).
1) Introduce ArtifactWhole (or ArtifactConstellation)

Keep DigitalArtifact for file-like things. Add a separate class for the “whole”:

<ArtifactWhole/> a rdfs:Class ;
  rdfs:label "Artifact whole" ;
  rdfs:comment "A conceptual whole that groups one or more DigitalArtifacts as facets/realizations/states of the same artifact. Used to avoid privileging any single facet IRI as the sole anchor." .

Then connect facets to the whole:

<facetOf/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <ArtifactWhole/> ;
  rdfs:label "facet of" ;
  rdfs:comment "Links a DigitalArtifact to an ArtifactWhole it participates in." .

<hasFacet/> a owl:ObjectProperty ;
  owl:inverseOf <facetOf/> ;
  rdfs:domain <ArtifactWhole/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "has facet" .

Now you can let any facet IRI “anchor” the whole via facetOf without conflating the facet with the whole.
Key implication

A LocatedFile can be:

    a whole of itself (facetOf some ArtifactWhole), and

    a facet of other wholes.

That is not punning. It is multi-membership. The whole is the disambiguator.
2) Keep your facet lattice, but make it structural among facets, not “the whole”

Your narrowerArtifact / broaderArtifact relations still make sense as facet-to-facet relations. They do not need to pretend to define “the artifact.”

<narrowerFacet/> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "narrower facet" .

<broaderFacet/> a owl:ObjectProperty ;
  owl:inverseOf <narrowerFacet/> ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "broader facet" .

Then your level-specific edges (hasState, hasAbstractFile, hasLocatedFile) can be subproperties of narrowerFacet if you want skip-linking derivations.
3) Why “any facet can anchor the artifact” is otherwise unsafe

You said it directly: narrower IRIs can be facets of multiple artifacts. Example:

    One LocatedFile could be used as a distribution for multiple AbstractFile specs (or multiple states).

    One AbstractFile spec could be shared across multiple states.

    One ResourcePage could present multiple things.

So if you treat “facet IRI” == “whole artifact IRI,” you cannot reliably answer “which artifact do you mean?” from that IRI alone. That’s why it feels wrong.

With ArtifactWhole, you can answer: “this facet participates in these wholes,” and applications can pick the intended one (or you can declare a default).
4) Add “preferred whole” to handle ambiguity when a facet participates in multiple wholes

To make usage practical:

<preferredWhole/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <ArtifactWhole/> ;
  rdfs:label "preferred whole" ;
  rdfs:comment "If a facet participates in multiple ArtifactWholes, indicates the intended/primary whole for default navigation." .

This is your “single referent principle” escape hatch, but explicit and controlled.
5) Skip-linking with specific subproperties

Yes—valuable. Do it by making the “narrower” relation transitive, and then introducing specialized shortcuts that are also subproperties.

Example: state → located file shortcut:

<locatedFileForState/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet/> ;
  rdfs:domain <ArtifactState/> ;
  rdfs:range <LocatedFile/> ;
  rdfs:label "located file for state" ;
  rdfs:comment "Skip-link from state to a located file (possibly derived via AbstractFile). May be asserted for convenience." .

Then you can also have “default located file for state” as a preference predicate that is not structural (unless you want it to imply structure).
6) What happens to DiachronicArtifact?

If you introduce ArtifactWhole, you can treat DiachronicArtifact as just one facet level that often serves as a convenient human-facing anchor, but no longer carries the burden of being the whole.

    DiachronicArtifact can be a DigitalArtifact and a facetOf an ArtifactWhole.

    When you have it, it’s the “stable public name.”

    When you don’t, the whole still exists.

This resolves your “fifth facet” concern without inventing an “AbstractArtifactFacet” that duplicates the function of ArtifactWhole.
Recommendation

If you want:

    every facet is its own DigitalArtifact, and

    you can talk about “the artifact” independent of which facet you have in hand,

then you need an explicit whole (ArtifactWhole/ArtifactConstellation) and a facetOf relation. Everything else (anchor/state/abstract-file/located-file) is then just a structured facet lattice attached to that whole.

If you refuse to add ArtifactWhole, you will be forced to privilege one facet as “the artifact,” and your ambiguity/punning feeling will keep coming back.
You said:
I like where we're going with ArtifactWhole. It's basically the same as ArtifactAnchor. We can give our mesh hierarchy enough folders to accomodate all 5 levels. But I think there's still an issue if we try to apply to non-mesh (external) Artifacts: There isn't necessarily an ArtifactWhole IRI available that's distinct from the LocatedFile (or AbstractFile, or State) IRIs.

There's also an issue for all artifacts, mesh or otherwise: If every facet has instances that are both facet of some bigger whole, and facets of their smaller wholes, we might want separate IRIs for every whole. 

Maybe this is solved (similar to punning), where the each of the facet instances is a sflo:ArtifactWhole. That begets lots of semi-useless part-whole statements, unless there's some advanced transivity where, e.g., if a LocatedFile is sflo:hasWhole to an AbstractFile, it is also hasWhole to the AbstractFile's state-as-an-artifact and its AbstractFile's State's DiachronicArtifact.

ChatGPT said:

Treating ArtifactWhole as “basically the same as ArtifactAnchor” reintroduces the ambiguity you were trying to eliminate. The whole and the anchor can often coincide in mesh cases, but they must stay conceptually distinct or you lose the ability to represent external artifacts cleanly.

You have two separate problems:

    External artifacts may not provide a distinct whole IRI.

    Facet nodes can be wholes of themselves and facets of larger wholes, which threatens to explode into pointless part-whole graphs.

Both are solvable if you adopt one rule: “Whole IRIs are optional, but when present they’re introduced only at ‘artifact boundaries’ you care about.” Do not auto-create a whole for every facet unless you have a boundary reason.
1) External artifacts without a distinct whole IRI

You don’t need a distinct whole IRI to use the model. You need a way to talk about the whole.

You have three workable strategies, and you can support all three:
Strategy E1: Whole = facet IRI (identity collapse allowed)

For external artifacts, the “whole” may just be the located file IRI. That is fine, but only if you state it as a modeling convention: “this IRI plays both roles.”

Mechanically: allow any DigitalArtifact to also be an ArtifactWhole.

<ArtifactWhole/> rdfs:subClassOf rdfs:Resource .  # not a DigitalArtifact necessarily

Do not force ArtifactWhole to be disjoint from DigitalArtifact. Then you can say:

<ext.pdf> a <LocatedFile/>, <DigitalArtifact/>, <ArtifactWhole/> .

No separate IRI needed.
Strategy E2: Mint a local whole IRI in your mesh for an external artifact

This is usually what you want in Semantic Flow: you have a local Nomen/Knop that “brings in” external artifacts. That local IRI is your whole.

Example (conceptually): <_external/<hash>/ > a ArtifactWhole ; hasFacet <https://example.com/ext.pdf> .

This avoids polluting external namespaces while giving you a stable whole identifier in your mesh.
Strategy E3: Use a blank node whole (for transient integration)

Sometimes you don’t want an IRI at all. You can still group facets with a blank node, especially in derived inventory datasets. That’s acceptable RDF.

Conclusion: “No distinct IRI exists” is not a blocker. It just means you use E1 or E3 until you decide you care enough to mint E2.
2) Avoiding the “every facet is its own whole” explosion

Your instinct is right: if every facet is a whole, you generate lots of low-value “whole-of” edges. Don’t do that.

Instead:

    Keep ArtifactWhole as the optional aggregation node.

    Keep facet-to-facet structure via your narrower/broader lattice.

    Only assert hasFacet/facetOf when you want to establish a boundary for navigation, ownership, or identity management.

Minimal relations set
Facet lattice (structural, transitive)

<narrowerFacet/> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> .

<broaderFacet/> a owl:ObjectProperty ;
  owl:inverseOf <narrowerFacet/> ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> .

Then level-specific edges as subproperties:

    hasState ⊑ narrowerFacet

    hasAbstractFile ⊑ narrowerFacet

    hasLocatedFile ⊑ narrowerFacet

    plus optional skip-links like locatedFileForState ⊑ narrowerFacet

Whole aggregation (non-transitive, non-structural)

<hasFacet/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactWhole/> ;
  rdfs:range <DigitalArtifact/> .

<facetOf/> a owl:ObjectProperty ;
  owl:inverseOf <hasFacet/> ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <ArtifactWhole/> .

Critical rule: hasFacet/facetOf is not automatically implied by narrowerFacet. Otherwise every chain becomes a whole relationship and you’re back to the explosion.
3) Your “advanced transitivity” desire is better handled as a derived view, not axioms

You asked for something like:

    if a LocatedFile hasWhole to an AbstractFile, it also hasWhole to the AbstractFile’s state… etc.

If you encode that in OWL/RDFS, you will over-generate statements and make reasoning unpredictable.

Instead:

    Use narrowerFacet transitivity to compute “all narrower facets of X.”

    Use application logic / SPARQL property paths to resolve from any level to any other level on demand.

Example queries you’ll want (conceptually):

    from a whole: all located files via hasFacet/(narrowerFacet*) or hasFacet/(hasState?/hasAbstractFile?/hasLocatedFile?)

    from a facet: its nearest whole via facetOf (if asserted), otherwise use collapse rule (E1)

This keeps the data lean and the semantics controllable.
4) Where ArtifactAnchor fits now

“Anchor” is a type of facet (diachronic-level facet) that is often the preferred whole identity in mesh cases. But don’t equate it with whole.

    DiachronicArtifact (facet level) is a DigitalArtifact.

    It may be a facet of an ArtifactWhole.

    In mesh, you might choose to have the same IRI be both (collapse), but that’s a convention, not an identity law.

If you want the mesh hierarchy to “accommodate all 5 levels,” those 5 levels are:

    ArtifactWhole (optional aggregation)

    DiachronicArtifact (facet)

    ArtifactState (facet)

    AbstractFile (facet)

    LocatedFile (facet)

But note: (1) is not “above” the others structurally; it’s an index node.
5) Recommendation: a crisp rule-set that avoids semi-useless statements

    Model facet structure with narrowerFacet and level-specific subproperties.

    Introduce ArtifactWhole only when you need a stable grouping boundary (mesh-first).

    Permit identity collapse: a DigitalArtifact may also be an ArtifactWhole (external-first).

    Do not make “every facet is a whole.” That’s the source of the explosion.

    Do not axiomatize closure up the chain. Compute it with property paths.

If you want, I can propose exact property names for:

    lattice (broaderFacet, narrowerFacet)

    level edges (hasState, hasAbstractFile, hasLocatedFile)

    skip-links (locatedFileForState, locatedFileForWhole)

    whole aggregation (hasFacet, facetOf, preferredWhole)
    with domain/range choices that let you collapse roles when needed without domain collisions.

You said:
Yes, I don't think we need an Anchor if we have ArtifactWhole. I was suggesting that we could call ArtifactWhole "ArtifactAnchor". 

Instead of "Diachronic" as a term-part, we could re-introduce Flow. It's friendlier and ties us back to the Semantic Flow brand. In meshes, every ArtifactAnchor would have a Flow, even if it only had the WorkingState.

I'm considering, instead of using a "_payload" folder naming convention, using or just allowing the preferredPayloadSlug to serve as the payload foldername/identifier part. So then, sample metadata would look (non-exhaustively) like:

<_knops/<uuid>/alice-bio/> a sflo:AritfactAnchor\/ ;
    sflo:hasFlow\/ <_knops/<uuid>/alice-bio/_flow/> .
<_knops/<uuid>/alice-bio/_flow/> sflo:hasHistoricalState  <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/>;
    sflo:hasCurrentState <_knops/<uuid>/alice-bio/_flow/_current/>
<_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/> a sflo:State\/ ;
    sflo:hasAbstractFile  <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/alice-bio.txt/> ;
<_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/alice-bio.txt/> hasLocatedFile <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/alice-bio.txt/alice-bio.txt> .
ChatGPT said:

Calling ArtifactWhole “ArtifactAnchor” is fine only if you define it as the whole/constellation node, not as the diachronic identity. Right now your example makes that distinction implicitly (Anchor → Flow → States), which is good. The main remaining problems are (a) terminology drift, (b) slug-as-identifier contradictions, and (c) the file-spec vs located-file IRI pattern.
1) Anchor vs Flow: don’t collapse them semantically

Your proposed structure is coherent if you treat:

    sflo:ArtifactAnchor = the whole / constellation (may have many facets)

    sflo:Flow = the diachronic facet (over-time identity in your mesh)

    sflo:State = state facet

    sflo:AbstractFile = file-spec facet

    sflo:LocatedFile = location facet

Then the key is: Flow is the diachronic identity, not the Anchor. If you blur that, you reintroduce the ambiguity you were trying to fix.

If you want “every Anchor has a Flow (even if only WorkingState)”, make it an explicit SHACL constraint later (minCount 1 on hasFlow).
2) Slug as folder name contradicts your current meaning of preferredPayloadSlug

Your ontology comment currently says the slug “does not affect IRIs.” If you start using it as a folder segment, it does affect IRIs. You need to decide which of these you want:
Option A: Slug is an identifier segment (strongly recommended if you do it)

    Rename the property to something like payloadSlug (drop “preferred”)

    Treat it as immutable once minted

    Allow aliases via Nomens if the human-facing name changes

Option B: Keep slug as hint only (and don’t put it in IRIs)

    Keep canonical folder as a stable ID (UUID/hash)

    Use Nomens/designator paths for friendly URLs

A compromise that avoids “slug rename breaks everything” is:

    Canonical: .../{payloadId}/

    Friendly alias: .../{slug}/ (a Nomen or redirect)

    Or: .../{payloadId}--{slug}/ so the stable part remains even if you later add new slug aliases

If you go with “slug in the path,” you are implicitly choosing Option A, so the ontology must stop claiming it’s a hint.
3) Your AbstractFile / LocatedFile IRIs are currently awkward

You have:

    Abstract file: .../v1/alice-bio.txt/

    Located file: .../v1/alice-bio.txt/alice-bio.txt

That reads like “a directory named alice-bio.txt containing a file named alice-bio.txt”, which is technically workable but semantically noisy.

You want the identity strata to be obvious from the path. A clean pattern is:

    State: .../_flow/v1/ or .../_flow/20260130T101315Z/

    AbstractFile: .../_flow/v1/_abstract/alice-bio.txt

    LocatedFile(s): .../_flow/v1/_dist/alice-bio.txt (and variants if multiple formats)

That keeps “spec vs distribution” visually distinct and avoids duplicating file names as path segments.
4) Keep the “skip-linking” properties, but don’t pretend they define wholes

You said skip-linking with specific subproperties is valuable. Agreed. But don’t use those to infer that every facet is a whole.

Use two layers:
Structural lattice (transitive)

    sflo:narrowerFacet (transitive)

    sflo:broaderFacet (inverse)

Then make these subproperties of narrowerFacet:

    hasState

    hasAbstractFile

    hasLocatedFile

Semantic skip-links (optional assertions)

    locatedFileForState (State → LocatedFile)

    locatedFileForFlow (Flow → LocatedFile)

    defaultLocatedFileForState (preference, not structural)

If you want “advanced transitivity,” do it with narrowerFacet and SPARQL property paths in consumers, not by auto-creating ArtifactAnchor nodes for every facet.
5) External artifacts: you don’t need a distinct Anchor IRI, but you need a rule

For non-mesh artifacts you won’t always have an Anchor IRI distinct from a LocatedFile IRI. That’s not a flaw; it’s a modeling choice. Pick one rule:

    Collapse allowed: a LocatedFile may also be an ArtifactAnchor when no separate whole IRI exists.

    Integration anchor: when you ingest an external artifact into a mesh, you mint a local Anchor IRI and connect it to the external facets.

Your “ResourcePages can violate single referent and do double-duty” is exactly the “collapse allowed” rule. Just make it an explicit convention.
6) Your sample rewritten in the same spirit but with clearer strata

Turtle sketch (using your structure, but cleaning the path semantics and property names):

<_knops/<uuid>/alice-bio/> a sflo:ArtifactAnchor ;
  sflo:hasFlow <_knops/<uuid>/alice-bio/_flow/> .

<_knops/<uuid>/alice-bio/_flow/> a sflo:Flow ;
  sflo:hasCurrentState <_knops/<uuid>/alice-bio/_flow/_current/> ;
  sflo:hasHistoricalState <_knops/<uuid>/alice-bio/_flow/_history/v1/> .

<_knops/<uuid>/alice-bio/_flow/_history/v1/> a sflo:HistoricalState ;
  sflo:hasAbstractFile <_knops/<uuid>/alice-bio/_flow/_history/v1/_abstract/alice-bio.txt> ;
  sflo:locatedFileForState <_knops/<uuid>/alice-bio/_flow/_history/v1/_dist/alice-bio.txt> .

<_knops/<uuid>/alice-bio/_flow/_history/v1/_abstract/alice-bio.txt> a sflo:AbstractFile ;
  sflo:hasLocatedFile <_knops/<uuid>/alice-bio/_flow/_history/v1/_dist/alice-bio.txt> .

Notes:

    locatedFileForState is the skip-link.

    hasLocatedFile stays AbstractFile → LocatedFile.

    _current can be typed WorkingState and can have abstract files too, same pattern.

7) The critical conceptual point

    “Every level of an Artifact is its own DigitalArtifact… a State is simultaneously a DigitalArtifact and also headState of a different DigitalArtifact.”

That is fine only if you accept that:

    “ArtifactAnchor/whole” is not “the only true artifact,” it’s an aggregation boundary you mint when useful.

    Facets can belong to multiple wholes (shared AbstractFiles, reused LocatedFiles), so “any facet can anchor the whole” must be mediated by either:

        explicit facetOf relations, or

        a collapse convention when no whole exists.

If you want, I can propose the minimal class/property set (with names aligned to your “Flow” branding) that supports:

    Anchor (whole), Flow (diachronic facet), State, AbstractFile, LocatedFile

    Transitive facet lattice

    Skip-link predicates

    Collapse convention for external artifacts and resource pages

    A consistent path scheme that doesn’t make preferredPayloadSlug lie about being “non-identifying”

You said:
Yeah, I think we want payloadSlug (fixed forever) to be different that preferredPayloadFileSlug which can change if you want to change the LocatedFile's filename. 

I want a friendlier slug for payload because otherwise humans rooting around in the filesystem won't have a chance.

I saw in your thinking you might be worried about using the filename also in the AbstractFile IRI. a State might have multiple AbstractFiles, so we can't use just one conventional folder name. To make them meaningful and navigable, I think it's okay to repeat the filename. That also emphasizes that each LocatedFile has directly-corresponding AbstractFile. 

For #5, I'm not thinking primarily about ingestion. (If you ingest, you put it in a Knop, no issues.) I'm thinking about the use case where you want to talk about external Artifacts. In that case, I think collapse can be allowed. For ResourcePages, yes, collapse LocatedFile, ArtifactAnchor. Since LocatedFiles are also their own DigitalArtifacts (even in a Mesh), without stating e.g. <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/alice-bio.txt/> hasArtifactAnchor <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/alice-bio.txt/> (and the domain of hasArtifactAnchor is DigitalArtifact, or its a subproperty of facetOf, which would have a range of DigitalArtifact)

Here I'm thinking that ArtifactAnchor is the "true artifact". 
ChatGPT said:

You’re converging on a coherent model, but the sentence “ArtifactAnchor is the true artifact” will force you into either (a) lots of tautological/self-links or (b) losing the ability to represent shared facets without ambiguity. The way out is to make “true” mean “the grouping boundary” (the canonical whole you mint when you care), not “an entity that must exist separately from every facet.”

Below is a design that matches everything you just said, including collapse for externals and file-name repetition.
1) Slugs: split exactly as you proposed

    payloadSlug (immutable; identifier segment; folder name)

    preferredPayloadFileSlug (mutable; filename stem hint)

Update comments accordingly; do not keep the old “does not affect IRIs” claim for payloadSlug.

<payloadSlug/> a owl:DatatypeProperty ;
  rdfs:domain <ArtifactAnchor/> ;         # or Knop if you insist, but anchor is cleaner
  rdfs:range xsd:string ;
  rdfs:label "payload slug" ;
  rdfs:comment "Stable, immutable slug used as the artifact anchor's path segment (folder name). Changing it changes IRIs and should be treated as renaming/minting a new anchor." .

<preferredPayloadFileSlug/> a owl:DatatypeProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload file slug" ;
  rdfs:comment "Mutable filename-stem preference for located files (e.g., 'alice-bio'). Generators may ignore/sanitize it and must handle collisions." .

2) Repeating filenames in AbstractFile IRIs is fine, but make the level markers unambiguous

Your instinct is right: if a state can have multiple AbstractFiles, you need a stable way to name them and repeating the filename emphasizes correspondence.

What’s risky is using the same lexical token without a structural marker, because it blurs “spec identity” vs “bytes identity.”

So keep repetition, but put each level under a reserved segment:

    State: .../_flow/<state-id>/

    AbstractFile: .../_flow/<state-id>/_abstract/<filename>

    LocatedFile: .../_flow/<state-id>/_dist/<filename>

That yields a 1:1 pairing while remaining navigable.
3) External artifacts: collapse is allowed, but don’t model it via hasArtifactAnchor self-links

You suggested something like:

    <locatedFile/> hasArtifactAnchor <locatedFile/>

That’s the tautology trap. It produces a sea of useless triples and still doesn’t help with the core ambiguity problem (a located file can belong to multiple anchors).

Instead, use one of these two patterns:
Pattern X1 (recommended): “anchor is the true artifact” but optional, and collapse means typing

Allow a resource to be both a LocatedFile and an ArtifactAnchor.

<https://example.com/foo.pdf> a sflo:LocatedFile, sflo:DigitalArtifact, sflo:ArtifactAnchor .

No self-links required. Collapse is expressed by role overlap, not by an identity relation.
Pattern X2: facetOf only when nontrivial

If you have an external located file that you want to say is a facet of some distinct anchor (maybe you minted it in your mesh, maybe the external source provides one), then use:

<foo.pdf> sflo:facetOf <foo/> .

But you do not assert facetOf when the facet and anchor are the same node. Collapse is not modeled as a relation; it’s modeled as a shared IRI.

This is the same “collapse by multi-typing” pattern you’re already accepting for ResourcePages.
4) If ArtifactAnchor is the “true artifact,” then anchors cannot be derivable from facets in general

Because you already accepted:

    A LocatedFile can simultaneously be:

        a located file facet of an AbstractFile,

        a located file facet of a State,

        a located file facet of a Flow,

        and itself a DigitalArtifact.

If the anchor is “true,” then the mapping from facet → true anchor is not functional unless you add extra structure (or prohibit sharing). You have three choices:

    Prohibit sharing (unrealistic; you don’t want this).

    Allow ambiguity (facet may facetOf multiple anchors).

    Provide a preferredAnchor (to select a default in ambiguous cases).

If you want “humans rooting around in filesystem have a chance,” you want (3), because tooling needs a default navigation path.

<preferredAnchor/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <ArtifactAnchor/> ;
  rdfs:label "preferred anchor" ;
  rdfs:comment "If a facet participates in multiple anchors, indicates the intended default anchor for navigation." .

Then you use facetOf for structural membership when needed:

<facetOf/> a owl:ObjectProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <ArtifactAnchor/> ;
  rdfs:label "facet of" .

And you do not claim every facet has exactly one anchor.
5) How “Flow” fits if anchors are “true artifacts”

Your earlier split (Anchor → Flow → States) is compatible with “anchor is true,” but only if:

    Flow is a facet that’s always present in mesh-managed anchors.

    A Flow may exist without historical states (only _current).

    The anchor’s semantics do not depend on Flow being unique in the universe; it is just the anchor’s diachronic facet.

So:

<hasFlow/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactAnchor/> ;
  rdfs:range <Flow/> .

and in SHACL:

    ArtifactAnchor in a mesh context must have exactly 1 flow (or min 1, depending on whether you allow multiple flows per anchor).

6) Skip-link properties: yes, define them, but keep them as derived/shortcut, not definitive structure

You can keep hasLocatedFile on AbstractFile. Then add:

    locatedFileForState (State → LocatedFile)

    locatedFileForFlow (Flow → LocatedFile)

    locatedFileForAnchor (Anchor → LocatedFile)

These are valuable for resolution and navigation, and can be asserted or derived.

They should be subproperties of a generic “narrowerFacet” only if you want them to also claim structural narrowing. Otherwise keep them separate as “resolution shortcuts.”

Given your intent (“each LocatedFile corresponds directly to an AbstractFile”), it’s reasonable to treat them as structural subproperties when asserted.
7) A revised version of your example that matches all of this

<_knops/<uuid>/alice-bio/> a sflo:ArtifactAnchor ;
  sflo:payloadSlug "alice-bio" ;
  sflo:preferredPayloadFileSlug "alice-bio" ;
  sflo:hasFlow <_knops/<uuid>/alice-bio/_flow/> .

<_knops/<uuid>/alice-bio/_flow/> a sflo:Flow ;
  sflo:hasCurrentState <_knops/<uuid>/alice-bio/_flow/_current/> ;
  sflo:hasHistoricalState <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/> .

<_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/> a sflo:HistoricalState ;
  sflo:hasAbstractFile <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/_abstract/alice-bio.txt> ;
  sflo:locatedFileForState <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/_dist/alice-bio.txt> .

<_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/_abstract/alice-bio.txt> a sflo:AbstractFile ;
  sflo:hasLocatedFile <_knops/<uuid>/alice-bio/_flow/2026-01-30_1013_15_v1/_dist/alice-bio.txt> .

No need for any “hasArtifactAnchor self-link.”
8) The one hard constraint you must accept if Anchor is “true”

A facet cannot, in general, imply a unique anchor. Therefore:

    You cannot make facetOf functional (maxCount 1) globally.

    If you need a default, introduce preferredAnchor.

If you try to avoid that, you’ll end up inventing implicit rules that are equivalent to “self-link means anchor,” which is the tautology trap again.

If you want the next step to be mechanical: say whether facetOf should be asserted for all facets in a mesh (dense graph, easy traversal), or only when sharing/ambiguity exists (sparser graph, requires derivation). The rest of the ontology falls out of that decision.
You said:
Regarding X1, DigitalArtifact could probably just serve as the ArtifactAnchor? And since every facet instance is also an ArtifactAnchor/DigitalArtifact, can we just make LocatedFile et al a subclass of DigitalArtifact and be done with it?

Regarding #4, in a mesh, every narrower facet will be a facetOf every broader DigitalArtifact, and there will always be a general IRI for DigitalArtifact/ArtifactAnchor/ArtifactWhole. We're back to "Everything is a Flow, even if it only has a WorkingState", so we don't have to accomodate the weird case of SimpleArtifact. That's INTERNALLY. To accomodate talking about external things... I dunno. Your "preferredAnchor" seems meaningless, because it depends which DigitalArtifact you are talking about.

What's better, _current or _working. I think probably _working because it depends on the application whether the "current" state should be considered "last woven" or "_working".

In a Semantic Mesh, ArtifactAnchors have exactly 1 flow.

I think skip-link properties can claim structural narrowing as that's always the case, and it might save specifying the structure explicitly.

I'm debating your ".../_abstract/alice-bio.txt" suggestion. I kinda like it, but FYI, it would be <.../_abstract/alice-bio.txt/> (with a trailing slash). I was formerly thinking that the trailing slash is enough indicator that it's not a LocatedFile. It seems a little unnecessary to have the _abstract and _dist folders (which don't refer to anything) when you can just nest the LocatedFile directly under its AbstractFile segment/folder.
ChatGPT said:

If DigitalArtifact “just serves as the ArtifactAnchor,” you lose the ability to say “this thing has a Flow” without accidentally saying it about every facet (states, abstract files, located files). That conflicts with your constraint “in a mesh, anchors have exactly 1 flow.”

So: do not collapse ArtifactAnchor into DigitalArtifact unless you also accept that every LocatedFile (etc.) is an anchor and therefore must have a flow—which you do not want.
1) Keep the split: DigitalArtifact is the universe; ArtifactAnchor is the “true artifact” subset

    DigitalArtifact: file-like things (states/specs/assets/pages/bytes-at-location)

    ArtifactAnchor: the “true artifact” (the one you manage as a Flow in a mesh)

That preserves your “every facet is its own DigitalArtifact” while still letting “true artifact has flow” mean something.

Minimal typing:

    ArtifactAnchor ⊑ DigitalArtifact

    Flow ⊑ DigitalArtifact

    State ⊑ DigitalArtifact

    AbstractFile ⊑ DigitalArtifact

    LocatedFile ⊑ DigitalArtifact

And in a mesh: ArtifactAnchor must have exactly 1 hasFlow.

For externals: allow collapse by typing the external located file as an ArtifactAnchor when you don’t have anything better.
2) You don’t need preferredAnchor if you enforce “no sharing” internally

You’re assuming an internal invariant:

    each LocatedFile belongs to exactly one AbstractFile

    each AbstractFile belongs to exactly one State

    each State belongs to exactly one Flow

    each Flow belongs to exactly one ArtifactAnchor

If you commit to that (internally), then facet→anchor is functional and preferredAnchor is indeed pointless.

But you should say it explicitly as a mesh constraint, because the moment you allow reuse (same abstract file spec used by multiple states, same located file mirrored, etc.), you lose uniqueness.
3) _working vs _current

Use _working. It is unambiguous (“mutable head”).

If you also need “current published / last woven,” create a separate pointer and do not call it “working.” Two common patterns:

    _working/ = mutable head

    _current/ = pointer to current published state (optional)

Property names should mirror this to avoid semantic drift:

    hasWorkingState (Flow → WorkingState)

    hasPublishedState (Flow → HistoricalState) (multi)

    optionally hasCurrentPublishedState (Flow → HistoricalState) (functional)

Do not use hasCurrentState unless you freeze what “current” means.
4) Skip-link properties: make them structural and infer them (don’t replace structure)

If you still want AbstractFile nodes (you do), then the clean rule is:

    Assert only the adjacent structure:

        hasAbstractFile (State → AbstractFile)

        hasLocatedFile (AbstractFile → LocatedFile)

    Infer skip-links:

        locatedFileForState (State → LocatedFile)

You can support both “asserted” and “derived” consumers by declaring an OWL property chain:

    hasAbstractFile o hasLocatedFile ⊑ locatedFileForState

Then you can choose not to materialize skip-links in data; generators/queries can still compute them.

Similarly for Flow and Anchor:

    hasState o locatedFileForState ⊑ locatedFileForFlow

    hasFlow o locatedFileForFlow ⊑ locatedFileForAnchor

If you don’t want to introduce hasState separately, you can hang these off whatever you call the Flow→State relation.
5) _abstract and _dist folders are optional; the real question is collision management

Your “trailing slash distinguishes AbstractFile from LocatedFile” works as a human convention, but it doesn’t solve namespace collisions inside a state folder unless you reserve that namespace.

Two workable path schemes:
Scheme A: explicit level markers (clearer, fewer collisions)

    State: .../_flow/<state-id>/

    AbstractFile: .../_flow/<state-id>/_abstract/alice-bio.txt/

    LocatedFile: .../_flow/<state-id>/_dist/alice-bio.txt

Pros: unambiguous, easy to add alternates, easy to add other facet subtrees later.
Cons: _abstract/_dist are “structural” segments.
Scheme B: minimal (your preference), but you must reserve names

    AbstractFile: .../_flow/<state-id>/alice-bio.txt/

    LocatedFile: .../_flow/<state-id>/alice-bio.txt/alice-bio.txt

Pros: compact, filename repetition signals correspondence.
Cons: you’ve now dedicated the state’s child namespace to abstract-file tokens. If you later want other state children (metadata, indexes, other facets), you must either:

    put them under reserved _.../ segments anyway, or

    guarantee they never collide with filenames.

If you’re comfortable reserving _.../ for “non-abstract-file children,” Scheme B is viable and consistent with your goals.
6) Slugs: your split is correct, but it forces an ontology comment update

    payloadSlug is now an identifier path segment (immutable)

    preferredPayloadFileSlug is just a naming preference (mutable)

Make sure your ontology comments reflect that payloadSlug absolutely affects IRIs.
7) External artifacts without a separate anchor IRI

Your collapse rule is enough:

    If all you have is a URL to bytes, that IRI can be typed as LocatedFile and also ArtifactAnchor (and optionally Flow if you’re willing to assert a trivial flow).

    You do not need facetOf self-links.

If you require “every mesh anchor has a flow,” keep that as a constraint on a mesh-specific subclass, e.g. MeshArtifactAnchor ⊑ ArtifactAnchor with SHACL requiring exactly 1 flow. External anchors can then exist without flows.

Key decision you need to lock in (because it controls everything else):
Are you forbidding facet sharing internally (unique parent at each level)? If yes, you can safely drop preferredAnchor, treat anchors as “true artifacts,” and rely on property chains/transitivity for derived links. If no, you must reintroduce either (a) explicit facetOf to anchors or (b) a disambiguation mechanism.
You said:
Maybe instead of "ArtifactAnchor" we do "AbstractArtifact"? 

r.e. #2 I've been a little bit uncomfortable describing an AbstractFile as a "spec". It's a realization of the content of a State in a particular format with a specific set of bytes. Mostly that's going to be expressible as different file extensions (e.g., .ttl, .jsonld), but theoretically you could have multiple instances in the same filetype/mimetype, like if you wanted to support two different ontology versions, you could have alice-bio_v1.ttl and alice-bio_v2.ttl. Or if you wanted to provide zipped versions, you could have alice-bio.ttl.zip and alice-bio.jsonld.zip. Just FYI. Can you help write a better comment there? Does that address you concern about "collision management"? It doesn't really make sense to me. 

No _current, we can't support pointers (no symlinks or redirects in github pages)

Regarding the key decision... each facet has a unique parent (no sharing outside a Flow), BUT they should be simultaneously facets for each of their parent DigitalArtifacts (up to the AbstractArtifact). I guess there's a distinction between DigitalArtifacts that are contained directly under a Knop (FlowArtifacts?) and the DigitalArtifacts that help define the FlowArtifact (States, AbstractFiles, and LocatedFiles). 
ChatGPT said:
1) Rename: AbstractArtifact is better than ArtifactAnchor

Yes. It matches what you mean: “the abstract, whole artifact” without implying “anchor” mechanics or network-ish connotations.

Use:

    sflo:AbstractArtifact = the “true artifact” whole in a mesh (the thing humans care about)

    sflo:Flow = the over-time facet for an AbstractArtifact (brand-friendly)

    sflo:State = single-state facet (working or historical)

    sflo:AbstractFile = state realization in a particular file identity

    sflo:LocatedFile = bytes-at-location identity

And constrain (in mesh context): AbstractArtifact has exactly 1 Flow.
2) Better comment for AbstractFile

Your discomfort with “spec” is justified: you are not describing an abstract format; you’re describing an abstract file identity for a particular state, potentially with multiple variants even within the same media type.

A comment that captures that and stays future-proof:

<AbstractFile/> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact/> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-level identity for a particular State: a specific file variant (e.g., TTL vs JSON-LD, zipped vs unzipped, or distinct build/canonicalization choices) whose bytes may be served from one or more LocatedFiles. Multiple AbstractFiles may exist for the same State, including multiple variants within the same media type or extension." .

If you want it even tighter and more operational:

    “file-level identity” + “variant for a state” + “may have multiple locations” + “multiple per state including same type”.

This addresses the earlier “spec” confusion without committing to canonicalization machinery you may or may not build.
3) Collision management: what it actually was (and why your variant examples matter)

Your variant examples increase the need for a naming strategy. The “collision” isn’t about RDF identity; it’s about filesystem namespace collisions under a state folder when you avoid explicit _abstract/ and _dist/ segments.

If you use the minimal scheme:

    AbstractFile: .../<state-id>/alice-bio.ttl/

    LocatedFile: .../<state-id>/alice-bio.ttl/alice-bio.ttl

Then you’re reserving every direct child name of the state folder for “abstract file IDs” (which may include alice-bio_v1.ttl, alice-bio_v2.ttl, alice-bio.ttl.zip, etc.). That’s fine, but it means:

    You can’t also place other state-scoped children there (e.g., _meta/, _checksums/, _diff/) unless you reserve a prefix (commonly _).

    If you ever want another facet type under state besides AbstractFile, you need reserved subfolders.

So: your variant story doesn’t break the model; it just means you must declare a convention like:

    “Within a State folder, non-AbstractFile children MUST start with _.”

That is the collision management point.
4) No _current: agree, but you still need a working-state convention

If you can’t do pointers/redirects/symlinks on GitHub Pages, then you can’t have a stable “current historical state” path that changes without rebuilding content at the same IRI.

But you can still have:

    _working/ as the mutable working state

    versioned historical states as immutable folders

So your flow has:

    exactly one working state IRI: .../_flow/_working/

    zero or more historical state IRIs: .../_flow/<timestamp>_vN/

And “current published” becomes derived (latest by date/sequence) in metadata, not by a pointer folder.
5) Your “unique parent” constraint + “facet of all ancestors” is consistent and can be expressed cleanly

You want:

    No sharing outside a flow (tree-shaped containment)

    But also: each facet is a facet of all broader artifacts up to AbstractArtifact

That can be done without asserting a huge number of redundant triples if you rely on transitive subproperties.

Define a single transitive structural relation and then specialize:

<narrowerFacet/> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "narrower facet" ;
  rdfs:comment "Relates a broader DigitalArtifact to a narrower DigitalArtifact facet in the same flow-defined artifact structure." .

Then:

    hasFlow ⊑ narrowerFacet (AbstractArtifact → Flow)

    hasState ⊑ narrowerFacet (Flow → State)

    hasAbstractFile ⊑ narrowerFacet (State → AbstractFile)

    hasLocatedFile ⊑ narrowerFacet (AbstractFile → LocatedFile)

Because narrowerFacet is transitive, you automatically get:

    AbstractArtifact narrowerFacet LocatedFile

    Flow narrowerFacet LocatedFile

    etc.

That gives you “facet of all ancestors” as an inferred view without materializing every skip-link.

If you still want explicit skip-link predicates for convenience, add them as subproperties and optionally define them via property chains:

    hasState o hasAbstractFile ⊑ hasAbstractFileForFlow (etc.)

    but you likely don’t need them if you accept property paths in SPARQL.

6) The “two kinds of artifacts” distinction you’re gesturing at

You’re describing a containment boundary:

    Primary artifacts: the things directly under a Knop (your AbstractArtifacts)

    Structural artifacts: the internal facet artifacts that define/realize the primary artifact (Flow, State, AbstractFile, LocatedFile)

Model this explicitly if it helps consumers:

    sflo:PrimaryArtifact ⊑ sflo:AbstractArtifact

    sflo:StructuralArtifact ⊑ sflo:DigitalArtifact with subclasses Flow, State, AbstractFile, LocatedFile

But you don’t need these extra classes unless you have actual constraints or UI behavior that depends on them. Otherwise it’s just taxonomy noise.

If you confirm you’re adopting the “reserve _ children under State for non-AbstractFile items” convention, then your preferred “no _abstract/_dist folders” layout is consistent and you can keep the filename repetition scheme.
You said:
Yes, I think we can require non-AbstractFile items to use underscores. EXCEPT the ResourcePage (index.html). So we need an invariant there (and also for LocatedFiles and for PayloadFlow and ???) that "index.html" is not allowed. 

I think we're agreed everywhere. Unless you have further questions, please rewrite the ontology based on everything we've discussed. Maybe best to open in a canvas.
ChatGPT said:

Two things you should sanity-check before you treat this as “done,” because they’re the only remaining places I can see you tripping later:

    SemanticFlowResource vs DigitalArtifact: I left DigitalArtifact independent and made your mesh-contained artifacts be SemanticFlowResource only where you explicitly typed them that way (e.g., PayloadArtifact, metadata/inventory artifacts). Your facet classes (Flow, State, AbstractFile, LocatedFile) are not declared SemanticFlowResource subclasses—by design—so you can still describe externals as sflo:LocatedFile without implying “mesh-governed.” If you want all facets inside a mesh to be “SF resources,” you’ll either need (a) a mesh-specific subclass layer, or (b) to relax what SemanticFlowResource means.

    The index.html invariant: it’s captured as a convention in comments, not enforced. If you want enforcement, it should be done in SHACL (either via literal tokens you record, or via IRI regex patterns), but I didn’t add those shapes because they’ll become brittle if you ever change path layouts.

Canvas now contains the rewritten ontology with AbstractArtifact/Flow/State, the slug split, transitive facet lattice, and deprecations for the old Slice model.
You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
The range for previousState should be a HistoricalState. Probably the domain should be too. Maybe we call it previousHistoricalState. And I think we'll want a latestHistoricalState property too. 
ChatGPT said:

Updated in the canvas:

    Replaced previousState with previousHistoricalState:

        domain = HistoricalState

        range = HistoricalState

        subPropertyOf = prov:wasRevisionOf

    Added latestHistoricalState:

        domain = Flow

        range = HistoricalState

        subPropertyOf = hasHistoricalState

        declared owl:FunctionalProperty

        comment clarifies it’s an RDF pointer (not a filesystem alias).

You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
Could you provide a commit message for this new approach? 

Maybe instead of RdfDataset class, we should use a "isRdfDataset" boolean property with Domain DigitalArtifact? Pros and cons please.
ChatGPT said:
Commit message

Subject

refactor(ontology): replace FlowArtifact/Slice with AbstractArtifact→Flow→State facet model

Body

- Introduce AbstractArtifact/Flow/State/AbstractFile/LocatedFile as the primary artifact stratification
- Add transitive narrowerFacet/broaderFacet lattice; core relations participate in narrowerFacet
- Rename revision relation to previousHistoricalState; add latestHistoricalState pointer on Flow
- Split payload slug semantics: add immutable payloadSlug + mutable preferredPayloadFileSlug; deprecate preferredPayloadSlug
- Keep ResourcePage/index.html reserved; document mesh layout invariants for '_' items and index.html avoidance
- Deprecate prior FlowArtifact/Slice/WorkingSlice/HistoricalSlice and slice-related properties

If you prefer a shorter commit, drop the last two bullets and keep only the model shift + slug split + deprecations.
Replace RdfDataset class with isRdfDataset boolean

This is almost always a bad trade in RDF/OWL, especially for your use case.
Pros of isRdfDataset: xsd:boolean

    Slightly simpler for non-RDF-native consumers (one boolean flag).

    Avoids importing/depending on dcat:Dataset for basic tagging.

    You can attach it to anything without “class modeling.”

Cons (major)

    It destroys compatibility with the ecosystem

        A thing being an RDF dataset is already naturally expressed as a dcat:Dataset (and/or a void:Dataset).

        Tools, validators, and queries will expect class membership, not a boolean.

    It’s semantically leaky and invites contradictions

        You can assert both isRdfDataset true and isRdfDataset false unless you add SHACL constraints.

        With class typing, contradictions are less common and patterns are clearer.

    You lose subclassing and composability

        Right now you can say: KnopMetadataArtifact ⊑ RdfDataset and it “just works.”

        With a boolean, you can’t subclass “things that are RDF datasets” in OWL without hacks.

    It’s the wrong level of abstraction for “content kind”

        “Is this an RDF dataset?” is a kind of artifact content, which is precisely what classes are for in RDF.

    You already need class-based typing elsewhere

        Your ontology is explicitly class-first (AbstractArtifact, Flow, etc.). Introducing booleans for kind-tags makes the model inconsistent.

Recommendation

Keep RdfDataset as a class, but tighten it:

    Keep: RdfDataset ⊑ DigitalArtifact and RdfDataset ⊑ dcat:Dataset (as you have).

    Optionally add a lightweight “content kind” pattern if you want more kinds later:

        hasContentKind (object property) → instances like ContentKind/RdfDataset

        This keeps “taggability” without booleans and stays extensible.

If you still want a machine-friendly flag, add it as a derived annotation, not primary truth:

    isRdfDataset as an annotation property (or leave it out entirely), computed from rdf:type.

One small correction to consider (related)

Your current RdfDataset comment says it “may be used to type a Flow.” That’s plausible but a little muddy: usually the dataset-ness applies to a State (and its AbstractFiles), not the diachronic Flow. If you keep it, you may want the comment to say “typically used to type State (and/or AbstractArtifact) depending on modeling.”
You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
I think any level can declare a sflo:RdfDataset, even AbstractArtifact. In fact, that's probably the best place to declare it. It's hard to imagine something not being completely RdfDataset from top to bottom. Ideally, it would be transitive, i.e., if AbstractArtifact a RdfDataset, so would all its narrowerFacet.
ChatGPT said:

This is workable, but only if you accept that “RDF-dataset-ness” is a facet-closed kind in your artifact lattice.

In OWL, class membership isn’t transitive by itself, so you need an axiom that forces the type to propagate along your facet relations. The clean way is exactly what you’re implying: if something is an sflo:RdfDataset, then everything connected to it via narrowerFacet (and optionally broaderFacet) is also an sflo:RdfDataset.
What I changed in the canvas

I updated sflo:RdfDataset so it is closed under the facet lattice:

    If X a sflo:RdfDataset then all X sflo:narrowerFacet Y imply Y a sflo:RdfDataset.

    If X a sflo:RdfDataset then all X sflo:broaderFacet Y imply Y a sflo:RdfDataset.

This means you can assert sflo:RdfDataset at any level (AbstractArtifact, Flow, State, AbstractFile, LocatedFile) and a reasoner will infer the rest “top-to-bottom” and “bottom-to-top,” as long as the facet links exist.
Why this is a strong commitment

Pros:

    Matches your intuition: “hard to imagine not being RDF dataset from top to bottom.”

    Lets you type it once (ideally at AbstractArtifact) and get all facet levels inferred.

    Avoids repeating a sflo:RdfDataset across every facet in data.

Cons / failure modes:

    If you ever want a mixed artifact (e.g., an RDF dataset artifact that also has a non-dataset located file facet in the facet chain), this will make that inconsistent (or force you not to connect it by narrowerFacet).

    If you use narrowerFacet “loosely” for external artifacts, dataset typing could propagate farther than you intend.

    You only get the propagation if you run an OWL reasoner (or a SHACL rule engine). Plain RDF consumers won’t infer it.

Minimal alternative if you want less force

If you only want “top → down” (your last message), keep only the narrowerFacet only RdfDataset restriction, and don’t include the broaderFacet restriction.

But given your statement “any level can declare it,” bidirectional closure is the consistent interpretation—so that’s what is now in the canvas.
You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
You're right, RdfDataset is intended to be generally useful so we should close it under the facet lattice.
ChatGPT said:

Agreed. With your “any level can declare it” requirement, facet-closure has to work in both directions:

    If you assert X a sflo:RdfDataset at the AbstractArtifact level, it must propagate down to Flow/State/AbstractFile/LocatedFile.

    If you assert it at a narrow level (e.g., a LocatedFile you discovered externally), it must propagate up to its broader facets.

That’s why the canvas version closes sflo:RdfDataset under both sflo:narrowerFacet and sflo:broaderFacet. If you only closed it under narrowerFacet, then “declare at any level” would silently fail whenever the declaration happens on a narrow facet (because it usually has no outgoing narrowerFacet triples).

One caution you should keep in mind (not a blocker): this makes sflo:RdfDataset a globally facet-closed kind. So if you ever model an artifact lattice that mixes dataset and non-dataset facets under the same narrowerFacet structure, you’ll be forced to either (a) not connect them by narrowerFacet, or (b) accept that they’re all datasets. That’s consistent with your stated intent.
You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
Sorry, I meant to say we don't want to globally close RdfDataset, so revert those changes.
ChatGPT said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
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
@prefix prov: <http://www.w3.org/ns/prov#> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon/> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .

#################################################################
# General-purpose vocabulary
#################################################################

<DigitalArtifact/> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A file-like digital object (documents, specifications, datasets, pages, binary assets), whether abstract or location-specific. A DigitalArtifact may exist inside or outside any Semantic Flow mesh." .

<ArtifactContainer/> a rdfs:Class ;
  rdfs:label "Artifact container" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself required to be a DigitalArtifact." .

#################################################################
# Artifact facet lattice
#################################################################

Reverted in the canvas: sflo:RdfDataset no longer includes the facet-closure restrictions over sflo:narrowerFacet / sflo:broaderFacet. It is back to:

    sflo:RdfDataset ⊑ sflo:DigitalArtifact

    sflo:RdfDataset ⊑ dcat:Dataset

    with the simpler comment.

One inconsistency to flag: you want “assert at AbstractArtifact and it flows down” sometimes, but you don’t want it globally. Those two desires conflict unless you scope the propagation.

If you want propagation only for mesh-managed artifacts, use one of these patterns (pick based on how strict you want to be):

    Mesh-scoped dataset class

    Add sflo:MeshRdfDataset ⊑ sflo:RdfDataset

    Put the closure restrictions on sflo:MeshRdfDataset, not on sflo:RdfDataset.

    Generators assert a sflo:MeshRdfDataset on the AbstractArtifact for mesh artifacts.

    SHACL rules (operational, not global OWL semantics)

    Keep ontology clean (no global closure).

    In the mesh validation/derivation pipeline, add SHACL rules that copy sflo:RdfDataset to narrower facets for in-mesh graphs only.

    Property chain–based inference for skip-links, not type closure

    Leave typing manual at the layer you care about (typically AbstractArtifact).

    Consumers infer “this state is part of an RDF dataset artifact” by traversing broaderFacet+ to a typed node.

If you want, I can implement option (1) in the ontology right now with minimal surface area.
You said:
Semantic Flow Ontology (rewritten: Abstract Artifact/flow/state Model)
Alrighty, can you update the SHACL?

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
    "schema": "https://schema.org/",
    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/",
    "sh:targetClass": {
      "@type": "@id"
    },
    "sh:targetSubjectsOf": {
      "@type": "@id"
    },
    "sh:path": {
      "@type": "@id"
    },
    "sh:class": {
      "@type": "@id"
    },
    "sh:nodeKind": {
      "@type": "@id"
    },
    "sh:severity": {
      "@type": "@id"
    },
    "sh:datatype": {
      "@type": "@id"
    }
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
      "rdfs:comment": "Basic structural constraints for slice membership (targeted by property use; does not require rdf:type or RDFS inference).",
      "sh:targetSubjectsOf": "sflo:hasSlice/",
      "sh:property": [
        {
          "sh:path": "sflo:hasSlice/",
          "sh:minCount": 1,
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Warning",
          "sh:message": "A FlowArtifact SHOULD have at least one sflo:hasSlice."
        }
      ]
    },
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
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractFile should be locatable (usually)",
      "rdfs:comment": "Many meshes expect AbstractFiles to have at least one LocatedFile. Warning because some realization specs may be non-locatable.",
      "sh:targetClass": "sflo:AbstractFile/",
      "sh:property": {
        "sh:path": "sflo:hasLocatedFile/",
        "sh:minCount": 1,
        "sh:severity": "sh:Warning",
        "sh:message": "An AbstractFile SHOULD have at least one sflo:hasLocatedFile."
      }
    },
    {
      "@id": "ReferenceLinkShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "ReferenceLink basics",
      "rdfs:comment": "A ReferenceLink must identify the thing it is about.",
      "sh:targetClass": "sflo:ReferenceLink/",
      "sh:property": [
        {
          "sh:path": "sflo:referenceLinkFor/",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Violation",
          "sh:message": "A ReferenceLink MUST have exactly one sflo:referenceLinkFor."
        },
        {
          "sh:path": "sflo:hasReferenceRole/",
          "sh:minCount": 1,
          "sh:severity": "sh:Warning",
          "sh:message": "A ReferenceLink SHOULD have at least one sflo:hasReferenceRole."
        },
        {
          "sh:path": "sflo:referenceUriLiteral/",
          "sh:datatype": "xsd:anyURI",
          "sh:severity": "sh:Info",
          "sh:message": "If present, sflo:referenceUriLiteral must be xsd:anyURI."
        }
      ]
    },
    {
      "@id": "KnopPayloadChoiceShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop payload choice",
      "rdfs:comment": "A Knop MUST have exactly one payload form: either a FlowPayload or a SimplePayload. Counts are qualified-by-class and accept either the specific containment predicates or the generic sflo:containsSemanticFlowResource.",
      "sh:targetClass": "sflo:Knop/",
      "sh:xone": [
        {
          "@type": "sh:NodeShape",
          "sh:property": [
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsFlowPayload/",
                  "sflo:containsSemanticFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:FlowPayload/"
              },
              "sh:qualifiedMinCount": 1,
              "sh:qualifiedMaxCount": 1,
              "sh:severity": "sh:Violation",
              "sh:message": "A Knop MUST contain exactly one FlowPayload (via containsFlowPayload or containsSemanticFlowResource)."
            },
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsSimplePayload/",
                  "sflo:containsSemanticFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:SimplePayload/"
              },
              "sh:qualifiedMaxCount": 0,
              "sh:severity": "sh:Violation",
              "sh:message": "A Knop MUST NOT contain any SimplePayload when it contains a FlowPayload."
            }
          ]
        },
        {
          "@type": "sh:NodeShape",
          "sh:property": [
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsSimplePayload/",
                  "sflo:containsSemanticFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:SimplePayload/"
              },
              "sh:qualifiedMinCount": 1,
              "sh:qualifiedMaxCount": 1,
              "sh:severity": "sh:Violation",
              "sh:message": "A Knop MUST contain exactly one SimplePayload (via containsSimplePayload or containsSemanticFlowResource)."
            },
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsFlowPayload/",
                  "sflo:containsSemanticFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:FlowPayload/"
              },
              "sh:qualifiedMaxCount": 0,
              "sh:severity": "sh:Violation",
              "sh:message": "A Knop MUST NOT contain any FlowPayload when it contains a SimplePayload."
            }
          ]
        }
      ],
      "sh:message": "A Knop MUST have exactly one of: FlowPayload or SimplePayload (and not both)."
    },
    {
      "@id": "KnopSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop slots",
      "rdfs:comment": "Enforces per-class slot cardinalities for common Knop-contained artifacts.",
      "sh:targetClass": "sflo:Knop/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsKnopMetadataFlow/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:KnopMetadataFlow/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Knop MUST have exactly one KnopMetadataFlow."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsKnopInventoryDataset/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:KnopInventoryDataset/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Knop MUST have exactly one KnopInventoryDataset."
        }
      ]
    },
    {
      "@id": "MeshSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Mesh slots",
      "rdfs:comment": "Enforces per-class slot cardinalities for common Mesh-contained artifacts.",
      "sh:targetClass": "sflo:Mesh/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsMeshInventoryDataset/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:MeshInventoryDataset/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Mesh MUST have exactly one MeshInventoryDataset."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsMeshMetadataFlow/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:MeshMetadataFlow/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Mesh must have a MeshMetadataFlow."
        }
      ]
    },
    {
      "@id": "NomenSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Nomen slots",
      "rdfs:comment": "Enforces per-class slot cardinalities for common Nomen-contained artifacts.",
      "sh:targetClass": "sflo:Nomen/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsNomenInventoryDataset/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:NomenInventoryDataset/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Nomen MUST have exactly one NomenInventoryDataset."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsNomenMetadataFlow/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:NomenMetadataFlow/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Nomen must have one NomenMetadataFlow."
        },
        {
          "sh:path": "sflo:hasReferenceLink/",
          "sh:class": "sflo:ReferenceLink/",
          "sh:severity": "sh:Violation",
          "sh:message": "Values of sflo:hasReferenceLink must be typed sflo:ReferenceLink."
        }
      ]
    },
    {
      "@id": "preferredPayloadSlugShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop preferred slug",
      "rdfs:comment": "Constraints for sflo:preferredPayloadSlug when used to suggest payload distribution filename stems.",
      "sh:targetClass": "sflo:Knop/",
      "sh:property": [
        {
          "sh:path": {
            "@id": "sflo:preferredPayloadSlug/"
          },
          "sh:maxCount": 1,
          "sh:datatype": {
            "@id": "xsd:string"
          },
          "sh:severity": {
            "@id": "sh:Violation"
          },
          "sh:message": "If present, sflo:preferredPayloadSlug MUST be a single xsd:string value."
        },
        {
          "sh:path": {
            "@id": "sflo:preferredPayloadSlug/"
          },
          "sh:pattern": "^[A-Za-z0-9](?:[A-Za-z0-9_.-]*[A-Za-z0-9])?$",
          "sh:severity": "sh:Warning",
          "sh:message": "If present, sflo:preferredPayloadSlug SHOULD be a portable file-stem:  ASCII letters/digits separated by hyphens or underscores (no spaces)."
        }
      ]
    }
  ]
}
ChatGPT said:

Your SHACL file is still validating the retired model (FlowArtifact/hasSlice/Slice/...) and the retired payload choice (FlowPayload vs SimplePayload), plus the old “Dataset” inventory classes. It needs a full sweep to match the new AbstractArtifact → Flow → State → AbstractFile → LocatedFile structure.

Below is an updated JSON-LD shapes graph that:

    Replaces slice/flowartifact shapes with hasFlow / hasWorkingState / hasHistoricalState / previousHistoricalState / latestHistoricalState.

    Updates Knop/Mesh/Nomen slot shapes to the new …MetadataArtifact / …InventoryArtifact classes and containment properties.

    Adds slug shapes for payloadSlug and preferredPayloadFileSlug, plus a warning shape for deprecated preferredPayloadSlug.

    Adds a consistency warning for locatedFileForState vs hasAbstractFile/hasLocatedFile.

    Leaves index.html enforcement as warning-level and token-level (slug), since full IRI-pattern enforcement tends to become brittle fast.

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
    "schema": "https://schema.org/",
    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/",
    "sh:targetClass": { "@type": "@id" },
    "sh:targetSubjectsOf": { "@type": "@id" },
    "sh:path": { "@type": "@id" },
    "sh:class": { "@type": "@id" },
    "sh:nodeKind": { "@type": "@id" },
    "sh:severity": { "@type": "@id" },
    "sh:datatype": { "@type": "@id" }
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
      "@id": "AbstractArtifactHasFlowShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractArtifact has exactly one Flow",
      "rdfs:comment": "In meshes, each AbstractArtifact has exactly one Flow. This shape assumes AbstractArtifacts are typed.",
      "sh:targetClass": "sflo:AbstractArtifact/",
      "sh:property": {
        "sh:path": "sflo:hasFlow/",
        "sh:minCount": 1,
        "sh:maxCount": 1,
        "sh:nodeKind": "sh:IRI",
        "sh:severity": "sh:Violation",
        "sh:message": "An AbstractArtifact MUST have exactly one sflo:hasFlow (IRI)."
      }
    },

    {
      "@id": "FlowWorkingStateShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Flow working state",
      "rdfs:comment": "A Flow must have exactly one working state.",
      "sh:targetClass": "sflo:Flow/",
      "sh:property": [
        {
          "sh:path": "sflo:hasWorkingState/",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Violation",
          "sh:message": "A Flow MUST have exactly one sflo:hasWorkingState (IRI)."
        },
        {
          "sh:path": "sflo:hasHistoricalState/",
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Info",
          "sh:message": "If present, values of sflo:hasHistoricalState must be IRIs."
        },
        {
          "sh:path": "sflo:latestHistoricalState/",
          "sh:maxCount": 1,
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Info",
          "sh:message": "If present, sflo:latestHistoricalState must be a single IRI."
        }
      ]
    },

    {
      "@id": "LatestHistoricalStateIsMemberShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "latestHistoricalState must be among hasHistoricalState",
      "rdfs:comment": "If latestHistoricalState is asserted, it must also be asserted as a hasHistoricalState of the same Flow.",
      "sh:targetClass": "sflo:Flow/",
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "If a Flow has sflo:latestHistoricalState, that value MUST also appear among sflo:hasHistoricalState values.",
        "sh:severity": "sh:Violation",
        "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:latestHistoricalState ?s .\n  FILTER NOT EXISTS { $this sflo:hasHistoricalState ?s }\n}"
      }
    },

    {
      "@id": "StateHasAbstractFileShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "State realization presence",
      "rdfs:comment": "States are content-bearing; they should usually have at least one AbstractFile realization.",
      "sh:targetClass": "sflo:State/",
      "sh:property": {
        "sh:path": "sflo:hasAbstractFile/",
        "sh:minCount": 1,
        "sh:nodeKind": "sh:IRI",
        "sh:severity": "sh:Warning",
        "sh:message": "A State SHOULD have at least one sflo:hasAbstractFile (IRI)."
      }
    },

    {
      "@id": "AbstractFileLocatedFileShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractFile should be locatable (usually)",
      "rdfs:comment": "Many meshes expect AbstractFiles to have at least one LocatedFile. Warning because some abstract file identities may be non-locatable.",
      "sh:targetClass": "sflo:AbstractFile/",
      "sh:property": {
        "sh:path": "sflo:hasLocatedFile/",
        "sh:minCount": 1,
        "sh:nodeKind": "sh:IRI",
        "sh:severity": "sh:Warning",
        "sh:message": "An AbstractFile SHOULD have at least one sflo:hasLocatedFile (IRI)."
      }
    },

    {
      "@id": "PreviousHistoricalStateShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "HistoricalState revision linkage",
      "rdfs:comment": "previousHistoricalState is optional but, if present, must point to exactly one HistoricalState (the immediately preceding published state).",
      "sh:targetSubjectsOf": "sflo:previousHistoricalState/",
      "sh:property": {
        "sh:path": "sflo:previousHistoricalState/",
        "sh:maxCount": 1,
        "sh:nodeKind": "sh:IRI",
        "sh:class": "sflo:HistoricalState/",
        "sh:severity": "sh:Violation",
        "sh:message": "sflo:previousHistoricalState must have at most one value, and it must be an IRI typed sflo:HistoricalState."
      }
    },

    {
      "@id": "PreviousHistoricalStateNotSelfShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "previousHistoricalState must not be self",
      "rdfs:comment": "Disallow self-links in the published revision chain.",
      "sh:targetSubjectsOf": "sflo:previousHistoricalState/",
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "A HistoricalState MUST NOT have itself as sflo:previousHistoricalState.",
        "sh:severity": "sh:Violation",
        "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:previousHistoricalState $this .\n}"
      }
    },

    {
      "@id": "LocatedFileForStateConsistencyShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "locatedFileForState consistency",
      "rdfs:comment": "If locatedFileForState is used as a shortcut, it should align with hasAbstractFile/hasLocatedFile.",
      "sh:targetSubjectsOf": "sflo:locatedFileForState/",
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "If a State uses sflo:locatedFileForState ?lf, there SHOULD exist an AbstractFile ?af such that ($this sflo:hasAbstractFile ?af) and (?af sflo:hasLocatedFile ?lf).",
        "sh:severity": "sh:Warning",
        "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:locatedFileForState ?lf .\n  FILTER NOT EXISTS {\n    $this sflo:hasAbstractFile ?af .\n    ?af sflo:hasLocatedFile ?lf .\n  }\n}"
      }
    },

    {
      "@id": "ReferenceLinkShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "ReferenceLink basics",
      "rdfs:comment": "A ReferenceLink must identify the thing it is about.",
      "sh:targetClass": "sflo:ReferenceLink/",
      "sh:property": [
        {
          "sh:path": "sflo:referenceLinkFor/",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:nodeKind": "sh:IRI",
          "sh:severity": "sh:Violation",
          "sh:message": "A ReferenceLink MUST have exactly one sflo:referenceLinkFor."
        },
        {
          "sh:path": "sflo:hasReferenceRole/",
          "sh:minCount": 1,
          "sh:severity": "sh:Warning",
          "sh:message": "A ReferenceLink SHOULD have at least one sflo:hasReferenceRole."
        },
        {
          "sh:path": "sflo:referenceUriLiteral/",
          "sh:datatype": "xsd:anyURI",
          "sh:severity": "sh:Info",
          "sh:message": "If present, sflo:referenceUriLiteral must be xsd:anyURI."
        }
      ]
    },

    {
      "@id": "KnopPayloadArtifactShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop payload artifact",
      "rdfs:comment": "A Knop MUST contain exactly one PayloadArtifact. Accepts either containsPayloadArtifact or the generic containsSemanticFlowResource.",
      "sh:targetClass": "sflo:Knop/",
      "sh:property": {
        "sh:path": {
          "sh:alternativePath": [
            "sflo:containsPayloadArtifact/",
            "sflo:containsSemanticFlowResource/"
          ]
        },
        "sh:qualifiedValueShape": {
          "sh:class": "sflo:PayloadArtifact/"
        },
        "sh:qualifiedMinCount": 1,
        "sh:qualifiedMaxCount": 1,
        "sh:severity": "sh:Violation",
        "sh:message": "A Knop MUST contain exactly one PayloadArtifact (via containsPayloadArtifact or containsSemanticFlowResource)."
      }
    },

    {
      "@id": "KnopSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Knop slots",
      "rdfs:comment": "Operational slot expectations for common Knop-contained artifacts.",
      "sh:targetClass": "sflo:Knop/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsKnopMetadataArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:KnopMetadataArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Knop MUST have exactly one KnopMetadataArtifact."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsKnopInventoryArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:KnopInventoryArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Knop MUST have exactly one KnopInventoryArtifact."
        }
      ]
    },

    {
      "@id": "MeshSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Mesh slots",
      "rdfs:comment": "Operational slot expectations for common Mesh-contained artifacts.",
      "sh:targetClass": "sflo:Mesh/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsMeshInventoryArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:MeshInventoryArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Mesh MUST have exactly one MeshInventoryArtifact."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsMeshMetadataArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:MeshMetadataArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Mesh MUST have exactly one MeshMetadataArtifact."
        }
      ]
    },

    {
      "@id": "NomenSlotsShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "Nomen slots",
      "rdfs:comment": "Operational slot expectations for common Nomen-contained artifacts.",
      "sh:targetClass": "sflo:Nomen/",
      "sh:property": [
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsNomenInventoryArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:NomenInventoryArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Nomen MUST have exactly one NomenInventoryArtifact."
        },
        {
          "sh:path": {
            "sh:alternativePath": [
              "sflo:containsNomenMetadataArtifact/",
              "sflo:containsSemanticFlowResource/"
            ]
          },
          "sh:qualifiedValueShape": { "sh:class": "sflo:NomenMetadataArtifact/" },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Nomen MUST have exactly one NomenMetadataArtifact."
        },
        {
          "sh:path": "sflo:hasReferenceLink/",
          "sh:class": "sflo:ReferenceLink/",
          "sh:severity": "sh:Violation",
          "sh:message": "Values of sflo:hasReferenceLink must be typed sflo:ReferenceLink."
        }
      ]
    },

    {
      "@id": "PayloadSlugShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractArtifact payloadSlug",
      "rdfs:comment": "Validates sflo:payloadSlug token constraints (folder-segment).",
      "sh:targetClass": "sflo:AbstractArtifact/",
      "sh:property": [
        {
          "sh:path": "sflo:payloadSlug/",
          "sh:maxCount": 1,
          "sh:datatype": "xsd:string",
          "sh:severity": "sh:Warning",
          "sh:message": "If present, sflo:payloadSlug MUST be a single xsd:string."
        },
        {
          "sh:path": "sflo:payloadSlug/",
          "sh:pattern": "^[A-Za-z0-9](?:[A-Za-z0-9._-]*[A-Za-z0-9])?$",
          "sh:severity": "sh:Warning",
          "sh:message": "If present, sflo:payloadSlug SHOULD be a portable folder token (ASCII letters/digits with . _ -). It SHOULD NOT start with '_' (reserved)."
        }
      ]
    },

    {
      "@id": "PayloadSlugIndexHtmlBanShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "payloadSlug must not be index.html",
      "rdfs:comment": "Reserves index.html for ResourcePage LocatedFiles.",
      "sh:targetSubjectsOf": "sflo:payloadSlug/",
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "sflo:payloadSlug MUST NOT be 'index.html' (reserved for ResourcePage files).",
        "sh:severity": "sh:Violation",
        "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:payloadSlug ?s .\n  FILTER(LCASE(STR(?s)) = \"index.html\")\n}"
      }
    },

    {
      "@id": "PreferredPayloadFileSlugShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "preferredPayloadFileSlug",
      "rdfs:comment": "Validates sflo:preferredPayloadFileSlug token constraints (filename stem preference).",
      "sh:targetClass": "sflo:AbstractArtifact/",
      "sh:property": [
        {
          "sh:path": "sflo:preferredPayloadFileSlug/",
          "sh:maxCount": 1,
          "sh:datatype": "xsd:string",
          "sh:severity": "sh:Warning",
          "sh:message": "If present, sflo:preferredPayloadFileSlug MUST be a single xsd:string."
        },
        {
          "sh:path": "sflo:preferredPayloadFileSlug/",
          "sh:pattern": "^[A-Za-z0-9](?:[A-Za-z0-9._-]*[A-Za-z0-9])?$",
          "sh:severity": "sh:Warning",
          "sh:message": "If present, sflo:preferredPayloadFileSlug SHOULD be a portable filename stem token (ASCII letters/digits with . _ -)."
        }
      ]
    },

    {
      "@id": "PreferredPayloadFileSlugIndexBanShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "preferredPayloadFileSlug should avoid index",
      "rdfs:comment": "Best-effort guardrail to prevent generating index.html unintentionally.",
      "sh:targetSubjectsOf": "sflo:preferredPayloadFileSlug/",
      "sh:sparql": {
        "@type": "sh:SPARQLConstraint",
        "sh:message": "sflo:preferredPayloadFileSlug SHOULD NOT be 'index' or 'index.html' (index.html is reserved for ResourcePage files).",
        "sh:severity": "sh:Warning",
        "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:preferredPayloadFileSlug ?s .\n  BIND(LCASE(STR(?s)) AS ?lc)\n  FILTER(?lc = \"index\" || ?lc = \"index.html\")\n}"
      }
    },

    {
      "@id": "DeprecatedPreferredPayloadSlugUseShape/",
      "@type": "sh:NodeShape",
      "rdfs:label": "preferredPayloadSlug is deprecated",
      "rdfs:comment": "Warn when deprecated sflo:preferredPayloadSlug is used.",
      "sh:targetSubjectsOf": "sflo:preferredPayloadSlug/",
      "sh:property": {
        "sh:path": "sflo:preferredPayloadSlug/",
        "sh:severity": "sh:Warning",
        "sh:message": "sflo:preferredPayloadSlug is deprecated. Use sflo:payloadSlug (immutable identifier segment) and sflo:preferredPayloadFileSlug (mutable filename preference)."
      }
    }
  ]
}

One decision you still need to be honest about

You’re enforcing “exactly one inventory artifact” for Mesh/Knop/Nomen as Violations, but your ontology comments describe these as optional derived indexes (“zero or more”). That is a real semantic mismatch. If “exactly one” is truly required operationally, either:

    update ontology comments to match (“MUST have exactly one …InventoryArtifact”), or

    downgrade those SHACL constraints to Warning, or

    introduce a new class like sflo:PublishedMesh/ that carries the “must have these slots” invariants.

If you tell me which of those three is the intended truth, I’ll align the shapes accordingly.
You said:
I updated the comments in my working copy.

Next task, here's the config ontology in jsonld. I'd like to do a complete sweep to update it for our new terminology and conceptualization. 

Now that Knops aren't hierarchical, inheritable config probably doesn't make sense. So we'll have a PlatformConfig (hosted somewhere), application can have ApplicationDefaultConfig (that overrides platform defaults), our first-class meshes can have configs that override PlatformConfig and ApplicationDefaultConfig, and I think we allow per-knop config overrides too.

Some stuff in the ontology doesn't make sense any more. Can you tease out what does and delete the rest? Put your results in a new canvas.
ChatGPT said:
Semantic Flow Config Ontology (rewrite For Platform/application/mesh/knop Config)
{
  "@context": {
    "@base": "https://semantic-flow.github.io/semantic-flow-config-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "dcterms": "http://purl.org/dc/terms/",
    "schema": "https://schema.org/",

    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/",

    "vann": "http://purl.org/vocab/vann/",

    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" },
    "owl:imports": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Config Ontology",
      "dcterms:description": "Vocabulary for describing layered configuration artifacts used by Semantic Flow platforms, applications, meshes, and knops.",
      "owl:imports": [
        "https://semantic-flow.github.io/semantic-flow-ontology/"
      ],
      "vann:preferredNamespacePrefix": "sflo-cfg",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/semantic-flow-config-ontology/",
      "owl:versionInfo": "0.0.1"
    },

    ////////////////////////////////////////////////////////////////////
    // Core configuration concepts
    ////////////////////////////////////////////////////////////////////

Created a new canvas with a clean rewritten config ontology centered on the layering model you described:

    PlatformConfig (lowest precedence)

    ApplicationDefaultConfig

    MeshConfig

    KnopConfig (highest precedence, no hierarchical inheritance)

What I kept (because it still makes sense in the new world):

    Config as a first-class artifact (ConfigArtifact ⊑ sflo:AbstractArtifact ⊓ sflo:RdfDataset)

    Explicit attachment points (hasPlatformConfig, hasApplicationDefaultConfig, hasMeshConfig, hasKnopConfig)

    A generic configFor plus specialized subproperties for clarity

    Optional overridesConfig (but called out as risky unless you constrain it)

What I intentionally did not carry forward (because it becomes incoherent with non-hierarchical Knops + deterministic layering):

    Any “inherits from parent knop / inherited config chain” machinery

    Any model where config resolution depends on Knop hierarchy or containment depth

Two places you should be strict (otherwise this config layer becomes non-deterministic fast):

    If you keep overridesConfig, you should eventually SHACL it into acyclic + at-most-one-parent (or explicitly define diamond merge semantics).

    Decide whether configs are always RDF datasets. I assumed yes (since you’re authoring the ontology in JSON-LD), but if you want non-RDF config payloads (YAML/JSON), ConfigArtifact ⊑ sflo:RdfDataset is wrong and should become optional typing instead.

If you paste your existing config JSON-LD, I can do an actual “tease out + delete” sweep as a diff-based rewrite rather than this clean-room replacement.
You said:
Semantic Flow Config Ontology (rewrite For Platform/application/mesh/knop Config)
I don't think I want different types of Configs. A single config can be used by any of these things, but there's only one type of config. Configs are always RdfDatasets. I think I forgot to paste my original config. Here it is:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/sflo-config/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "sh": "http://www.w3.org/ns/shacl#",
    "dcat": "http://www.w3.org/ns/dcat#",
    "dcterms": "http://purl.org/dc/terms/",
    "schema": "https://schema.org/",
    "formats": "http://www.w3.org/ns/formats/",
    "vann": "http://purl.org/vocab/vann/",
    "sflo": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow knop configuration Ontology",
      "dcterms:description": "Configuration properties for Semantic Flow mesh entities. Defines configurable properties that apply directly to mesh knops (operational config) and through inheritable config. Used by configuration files and sflo-host ontology for setting defaults.",
      "dcterms:creator": "Semantic Flow Project",
      "owl:versionInfo": "0.1.0",
      "vann:preferredNamespacePrefix": "conf",
      "vann:alternateNamespacePrefix": [
        "sflo-conf"
      ]
    },
    {
      "@id": "ConfigurationClass/",
      "@type": "rdfs:Class",
      "rdfs:label": "Configuration Class",
      "rdfs:comment": "Abstract parent class for classes that structure configuration information."
    },
    {
      "@id": "KnopNodeConfig/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        {
          "@id": "sflo:PlatformElement/"
        },
        {
          "@id": "dcat:Dataset"
        }
      ],
      "rdfs:label": "knop config dataset",
      "rdfs:comment": "Dataset containing KnopNode configuration data."
    },
    {
      "@id": "hasLocalConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": {
        "@id": "sflo:KnopNode/"
      },
      "rdfs:range": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:label": "local config",
      "rdfs:comment": "Links KnopNodes to a local config."
    },
    {
      "@id": "hasInheritableConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": {
        "@id": "sflo:KnopNode/"
      },
      "rdfs:range": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:label": "inheritable config",
      "rdfs:comment": "Links KnopNodes to an inheritable config."
    },
    {
      "@id": "hasEffectiveConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": {
        "@id": "sflo:KnopNode/"
      },
      "rdfs:range": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:label": "effective config",
      "rdfs:comment": "Links KnopNodes to their effective config. Shouldn't be persisted, only computed at runtime."
    },
    {
      "@id": "TemplateMappingSet/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": {
        "@id": "ConfigurationClass/"
      },
      "rdfs:label": "Template Mappings",
      "rdfs:comment": "Collection of template mappings."
    },
    {
      "@id": "TemplateMapping/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": {
        "@id": "ConfigurationClass/"
      },
      "rdfs:label": "Template Mapping",
      "rdfs:comment": "Template configuration mapping for resource page generation. A template mapping applies only if all specified target conditions (via mappingTargetClassRegex and mappingTargetSlugRegex) are satisfied. If a mapping specifies no class or slug conditions, it is applicable to all knops. Since mappings may come from multiple sources and may be multiple even from a single source, mappingPriority is used to resolve conflicts when multiple mappings apply to the same knop."
    },
    {
      "@id": "knopNodeConfigInheritanceEnabled/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "KnopNode config inheritance enabled",
      "rdfs:comment": "Whether this knop should accept inherited configuration from parent knops. Set in local KnopNodeConfig to disable inheritance resolution for this KnopNode. Set in InheritaleKnopNodeConfig to disable inheritance for child knops. Set in application KnopNodeConfig to disable inheritance (if not otherwise specified) for knops within that application.",
      "rdfs:domain": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:range": {
        "@id": "xsd:boolean"
      }
    },
    {
      "@id": "hasTemplateMappingSet/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has template mappings",
      "rdfs:comment": "Collection of template mappings for resource page generation",
      "rdfs:domain": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:range": {
        "@id": "TemplateMappingSet/"
      }
    },
    {
      "@id": "hasTemplateMapping/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has template mapping",
      "rdfs:comment": "Individual template mapping within a set of template mappings. ",
      "rdfs:domain": {
        "@id": "TemplateMappingSet/"
      },
      "rdfs:range": {
        "@id": "TemplateMapping/"
      }
    },

    {
      "@id": "InnerTemplate/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "ContainerResource/",
      "rdfs:label": "Inner template",
      "rdfs:comment": "A template resource that can evolve via Flow/Version and is rendered as one or more AbstractFiles."
    },
    {
      "@id": "OuterTemplate/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "ContainerResource/",
      "rdfs:label": "Outer template",
      "rdfs:comment": "A template resource that can evolve via Flow/Version and is rendered as one or more AbstractFiles."
    },
    {
      "@id": "Stylesheet/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "ContainerResource/",
      "rdfs:label": "Stylesheet",
      "rdfs:comment": "A stylesheet resource that can evolve via Flow/Version and is rendered as one or more AbstractFiles."
    },
    {
      "@id": "innerTemplate/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "inner template",
      "rdfs:comment": "Inner template for resource pages",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "InnerTemplate/"
      }
    },
    {
      "@id": "outerTemplate/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "outer template",
      "rdfs:comment": "Outer template for resource pages",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "OuterTemplate/"
      }
    },
    {
      "@id": "stylesheet/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "stylesheet path",
      "rdfs:comment": "Path to CSS stylesheet for generated pages",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "Stylesheet/"
      }
    },
    {
      "@id": "mappingPriority/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "mapping priority",
      "rdfs:comment": "Integer priority for resolving conflicts between template mappings. Higher values SHOULD take precedence when multiple mappings apply to the same knop.",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "xsd:integer"
      }
    },
    {
      "@id": "mappingTargetClassRegex/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "mapping target class regex",
      "rdfs:comment": "Regular expression for attaching mappings to target knops based on their class identifier.",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "xsd:string"
      }
    },
    {
      "@id": "mappingTargetSlugRegex/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "mapping target slug regex",
      "rdfs:comment": "Regular expression for attaching mappings to target knops based on their 'local name' slug",
      "rdfs:domain": {
        "@id": "TemplateMapping/"
      },
      "rdfs:range": {
        "@id": "xsd:string"
      }
    },
    {
      "@id": "generateResourcePages/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "generate resource pages",
      "rdfs:comment": "Whether to (re-)generate HTML resource pages during weave",
      "rdfs:domain": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:range": {
        "@id": "xsd:boolean"
      }
    },
    {
      "@id": "folderToken/",
      "@type": "owl:AnnotationProperty",
      "rdfs:label": "Folder token",
      "rdfs:comment": "Useful in folder naming.",
      "rdfs:domain": [
        {
          "@id": "sflo:PlatformElement/"
        }
      ],
      "rdfs:range": {
        "@id": "xsd:string"
      }
    },
    {
      "@id": "fileToken/",
      "@type": "owl:AnnotationProperty",
      "rdfs:label": "Flow token",
      "rdfs:comment": "Useful in distribution file naming.",
      "rdfs:domain": {
        "@id": "sflo:PlatformElement/"
      },
      "rdfs:range": {
        "@id": "xsd:string"
      }
    },
    {
      "@id": "workingToken/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "Flow token",
      "rdfs:comment": "Used in determining working folder naming.",
      "rdfs:domain": [
        {
          "@id": "KnopNodeConfig/"
        }
      ],
      "rdfs:range": {
        "@id": "xsd:string"
      }
    },
    {
      "@id": "retainNumSlices/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "retain number of slices",
      "rdfs:comment": "Number of previous FlowSlices to retain when weaving. Not defined means retain all.",
      "rdfs:domain": {
        "@id": "KnopNodeConfig/"
      },
      "rdfs:range": {
        "@id": "xsd:integer"
      }
    },
    {
      "@id": "anticipatedAbsoluteIri/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "anticipated absolute IRI",
      "rdfs:comment": "The anticipated absolute IRI of this knop within a published mesh, based on its Knop identifier, its path location, and the hosting platform.",
      "rdfs:domain": [
        {
          "@id": "sflo:PlatformElement/"
        }
      ]
    },
    {
      "@id": "parentKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "parent knop",
      "rdfs:comment": "indicates that this knop is contained in another knop. Should only be used in operational KnopConfig.",
      "rdfs:domain": "Knop/",
      "rdfs:range": "Knop/"
    },
    {
      "@id": "rootKnop/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "root knop",
      "rdfs:comment": "indicates that this is a root knop. Should only be used in operational KnopConfig.",
      "rdfs:domain": "Knop/",
      "rdfs:range": "xsd:boolean"
    }
  ]
}
ChatGPT said:

Created a new canvas with a rewritten sflo-config ontology that:

    Uses one class: conf:Config/ (always sflo:AbstractArtifact/ + sflo:RdfDataset/)

    Removes local/inheritable and all Knop hierarchy assumptions

    Defines layering via association properties (not config subtypes):

        conf:hasPlatformConfig/

        conf:hasApplicationDefaultConfig/

        conf:hasMeshConfig/

        conf:hasKnopConfig/

    Keeps the TemplateMapping model, but fixes the stylesheet/ property (it was incorrectly a DatatypeProperty)

If you want this to be tighter, the next obvious hardening step is adding SHACL that:

    enforces “at most one config per layer per target”

    prevents hasEffectiveConfig from appearing in persisted datasets (or at least flags it)

You said:
Sflo Config Ontology (single Config; Layered By Target)
Please remove:
- retainNumHistoricalStates
- Platform, Application, and any associated properties. We will have an sflo-host ontology for hasConfig for Applications. 
ChatGPT said:

I attempted to apply these removals directly in the existing canvas, but the automated edit failed because the exact JSON block patterns didn’t match (so the regex-based patch couldn’t be applied safely).

Here is the updated config ontology JSON-LD with your requested removals applied:

    Removed retainNumHistoricalStates/

    Removed Platform/, Application/

    Removed hasPlatformConfig/, hasApplicationDefaultConfig/

    Updated the ontology description to no longer mention platform/application layering

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/sflo-config/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "dcterms": "http://purl.org/dc/terms/",
    "vann": "http://purl.org/vocab/vann/",
    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/",

    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:subPropertyOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" },
    "owl:imports": { "@type": "@id" },
    "owl:inverseOf": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Configuration Ontology",
      "dcterms:description": "Vocabulary for describing configuration artifacts used by Semantic Flow resources. This ontology intentionally does not define key-level merge semantics; resolution is application-defined.",
      "dcterms:creator": "Semantic Flow Project",
      "owl:imports": [
        "https://semantic-flow.github.io/semantic-flow-ontology/"
      ],
      "owl:versionInfo": "0.2.1",
      "vann:preferredNamespacePrefix": "conf",
      "vann:alternateNamespacePrefix": ["sflo-conf"],
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/sflo-config/"
    },

    {
      "@id": "Config/",
      "@type": "owl:Class",
      "rdfs:subClassOf": ["sflo:AbstractArtifact/", "sflo:RdfDataset/"],
      "rdfs:label": "Config",
      "rdfs:comment": "An RDF configuration artifact (always an RDF dataset) used to control or parameterize Semantic Flow behavior. Configs are managed using the AbstractArtifact→Flow→State model from the core ontology. This ontology does not define how multiple configs are merged at the key level."
    },

    {
      "@id": "hasConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Config/",
      "owl:inverseOf": "configFor/",
      "rdfs:label": "has config",
      "rdfs:comment": "Associates any resource with one or more Config artifacts. Precedence is application-defined (and may be expressed via specialized subproperties in other ontologies such as sflo-host)."
    },

    {
      "@id": "configFor/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Config/",
      "rdfs:range": "rdfs:Resource",
      "owl:inverseOf": "hasConfig/",
      "rdfs:label": "config for",
      "rdfs:comment": "Inverse of hasConfig: identifies the resource a Config is intended to configure."
    },

    {
      "@id": "hasMeshConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasConfig/",
      "rdfs:domain": "sflo:Mesh/",
      "rdfs:range": "Config/",
      "rdfs:label": "has mesh config",
      "rdfs:comment": "Associates a Mesh with one or more mesh-level Config artifacts."
    },

    {
      "@id": "hasKnopConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasConfig/",
      "rdfs:domain": "sflo:Knop/",
      "rdfs:range": "Config/",
      "rdfs:label": "has knop config",
      "rdfs:comment": "Associates a Knop with one or more per-knop Config artifacts. Knops are not hierarchical, so there is no inheritable knop config chain."
    },

    {
      "@id": "hasEffectiveConfig/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasConfig/",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Config/",
      "rdfs:label": "has effective config",
      "rdfs:comment": "Associates a resource with a computed effective Config. This SHOULD NOT be persisted as authoritative data; it is intended for runtime views, caches, or debugging output."
    },

    {
      "@id": "TemplateMappingSet/",
      "@type": "owl:Class",
      "rdfs:label": "Template mapping set",
      "rdfs:comment": "A collection of TemplateMappings used for resource page generation."
    },

    {
      "@id": "TemplateMapping/",
      "@type": "owl:Class",
      "rdfs:label": "Template mapping",
      "rdfs:comment": "A mapping that selects templates/assets for generating resource pages. A mapping applies only if all specified target conditions are satisfied. When multiple mappings apply, mappingPriority may be used to resolve conflicts."
    },

    {
      "@id": "hasTemplateMappingSet/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Config/",
      "rdfs:range": "TemplateMappingSet/",
      "rdfs:label": "has template mapping set",
      "rdfs:comment": "Associates a Config with a TemplateMappingSet used during resource page generation."
    },

    {
      "@id": "hasTemplateMapping/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "TemplateMappingSet/",
      "rdfs:range": "TemplateMapping/",
      "rdfs:label": "has template mapping",
      "rdfs:comment": "Associates a TemplateMappingSet with one of its TemplateMappings."
    },

    {
      "@id": "InnerTemplate/",
      "@type": "owl:Class",
      "rdfs:subClassOf": ["sflo:AbstractArtifact/"],
      "rdfs:label": "Inner template",
      "rdfs:comment": "A template artifact used as an inner page template. As an AbstractArtifact, it may evolve via Flow/State and be realized as one or more AbstractFiles."
    },

    {
      "@id": "OuterTemplate/",
      "@type": "owl:Class",
      "rdfs:subClassOf": ["sflo:AbstractArtifact/"],
      "rdfs:label": "Outer template",
      "rdfs:comment": "A template artifact used as an outer page template. As an AbstractArtifact, it may evolve via Flow/State and be realized as one or more AbstractFiles."
    },

    {
      "@id": "Stylesheet/",
      "@type": "owl:Class",
      "rdfs:subClassOf": ["sflo:AbstractArtifact/"],
      "rdfs:label": "Stylesheet",
      "rdfs:comment": "A stylesheet artifact referenced by resource pages. As an AbstractArtifact, it may evolve via Flow/State and be realized as one or more AbstractFiles."
    },

    {
      "@id": "innerTemplate/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "InnerTemplate/",
      "rdfs:label": "inner template",
      "rdfs:comment": "Inner template for generated resource pages."
    },

    {
      "@id": "outerTemplate/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "OuterTemplate/",
      "rdfs:label": "outer template",
      "rdfs:comment": "Outer template for generated resource pages."
    },

    {
      "@id": "stylesheet/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "Stylesheet/",
      "rdfs:label": "stylesheet",
      "rdfs:comment": "Stylesheet artifact to include in generated resource pages."
    },

    {
      "@id": "mappingPriority/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "mapping priority",
      "rdfs:comment": "Integer priority for resolving conflicts between applicable template mappings. Higher values SHOULD take precedence."
    },

    {
      "@id": "mappingTargetClassRegex/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "xsd:string",
      "rdfs:label": "mapping target class regex",
      "rdfs:comment": "Regular expression for matching target resources by class identifier (e.g., an rdf:type IRI string). Interpretation is application-defined."
    },

    {
      "@id": "mappingTargetSlugRegex/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "TemplateMapping/",
      "rdfs:range": "xsd:string",
      "rdfs:label": "mapping target slug regex",
      "rdfs:comment": "Regular expression for matching target resources by a slug-like token (e.g., payloadSlug, designatorPath, or other application-defined token)."
    },

    {
      "@id": "generateResourcePages/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "Config/",
      "rdfs:range": "xsd:boolean",
      "rdfs:label": "generate resource pages",
      "rdfs:comment": "Whether to (re-)generate HTML resource pages during weave."
    }
  ]
}

Two inconsistencies you should resolve explicitly (otherwise you’ll silently break consumers):

    Your original config ontology used sflo as https://semantic-flow.github.io/ontology/semantic-flow-ontology/, while your core TTL earlier used https://semantic-flow.github.io/semantic-flow-ontology/. This rewrite uses the latter. If the former is actually canonical in your repo, switch it back here.

    If you truly want all platform/application attachment to live in sflo-host, then hasMeshConfig/ and hasKnopConfig/ may also belong there. If you keep them here, you’re implicitly saying “config ontology defines attachments for meshes and knops, but not for apps/platforms.” That asymmetry is fine, but it’s a deliberate design choice.

You said:
Sflo Config Ontology (single Config; Layered By Target)
I updated the canvas. 

Can you give me a config boolean property for createHistoricalStatesOnWeave ? I think we need to support assigning config to AbstractArtifact as well. 

I guess we can use blank nodes (in the metadata flows) when we don't want to link to a named ConfigArtifact. We'll need a new subclass for ConfigArtifact and remove the subclass restrictions on Config
ChatGPT said:

Applied the changes in the canvas:

    Added conf:createHistoricalStatesOnWeave/ (boolean, domain conf:Config/)

    Added conf:hasAbstractArtifactConfig/ (domain sflo:AbstractArtifact/, range conf:Config/, subproperty of conf:hasConfig/)

    Refactored config typing:

        conf:Config/ is now only ⊑ sflo:RdfDataset/ (so it can be used as a blank node)

        New conf:ConfigArtifact/ ⊑ conf:Config/ ⊓ sflo:AbstractArtifact/ for named, revisionable configs

    Bumped owl:versionInfo to 0.2.2

One consistency check you should decide soon (or you’ll get semantic drift):

    If Config is “always an RDF dataset,” then using it as a blank node is fine, but you’re implicitly allowing “dataset identity without an IRI.” That’s OK in RDF, but if your tooling assumes “every dataset has a stable handle,” you’ll want SHACL later to warn when a blank-node config contains more than trivial settings (i.e., encourage promoting it to ConfigArtifact).

You said:
Sflo Config Ontology (single Config; Layered By Target)
It sounds like you don't have any issues with having a blank node as a RdfDataset, that's great. Should we update this comment then? "A DigitalArtifact whose content is an RDF dataset (default graph + 0..n named graphs). May be used to type an AbstractArtifact, a Flow, or specific State(s), depending on modeling needs." Do we remove the subClassOf <DigitalArtifact/>?
ChatGPT said:

Yes, update that comment. Right now it subtly bakes in two assumptions that you don’t want:

    that being an sflo:RdfDataset implies “artifact-ness” in the narrow sense of “file-like thing” (you’re using it more as a payload/content kind tag), and

    that it should “live” at one fixed layer (you explicitly want it usable at multiple facet levels).

A better comment is:

    “A classification for artifacts whose payload content is an RDF dataset (default graph + 0..n named graphs). May be asserted at any artifact facet (e.g., AbstractArtifact, Flow, State, AbstractFile, or LocatedFile) depending on what is being characterized. This typing does not imply that broader/narrower facets are also RDF datasets.”

That makes it clear it’s not transitive across your facet lattice unless you explicitly assert it.
Do we remove rdfs:subClassOf sflo:DigitalArtifact?

I would keep it, unless you want to allow sflo:RdfDataset to type things that you do not consider artifacts (e.g., meshes, nomens, relators, agents).

Keeping the subclass does not interfere with blank nodes: a blank node can still be an instance of sflo:DigitalArtifact / sflo:RdfDataset. OWL/RDF does not require individuals to be IRIs.

The real risk is future-you adding constraints to sflo:DigitalArtifact (e.g., “must have a flow/state/located file”), which would accidentally apply to every sflo:RdfDataset-typed blank node. If you foresee that, then either:

    keep DigitalArtifact permanently lightweight (recommended), or

    split out a “payload kind” tag like sflo:RdfDatasetContent that is not a subclass of DigitalArtifact.

Given your current direction, keeping sflo:RdfDataset ⊑ sflo:DigitalArtifact is consistent and still compatible with blank-node configs.
You said:
Sflo Config Ontology (single Config; Layered By Target)
For the comment, I don't want to say "payload content" because PayloadArtifacts are a particular kind of Artifact. Also, I was wanting the comment to reflect that it could be applied to blank nodes as well as those other things. That's why I wanted to removing the subclassing, since blank nodes don't qualify as DigitalArtifacts.
ChatGPT said:

Blank nodes absolutely can be instances of sflo:DigitalArtifact (or any other class). In RDF/OWL, a blank node is still a resource; it just lacks a globally-stable name (IRI). So “blank nodes don’t qualify as DigitalArtifacts” is an application convention you could adopt, but it isn’t how the semantics work by default.

That means: you don’t need to remove sflo:RdfDataset ⊑ sflo:DigitalArtifact to support blank-node configs.
Recommended comment for sflo:RdfDataset

If you want to avoid “payload content” (since “Payload” is a specific SFLO thing) and explicitly acknowledge blank nodes, use something like:

    “A classification for resources whose represented content is an RDF dataset (default graph plus 0..n named graphs). Instances may be identified by an IRI or left unnamed (blank node). This typing may be asserted at any artifact facet (e.g., AbstractArtifact, Flow, State, AbstractFile, LocatedFile) depending on modeling needs.”

Two deliberate choices in that wording:

    “represented content” avoids “payload”

    explicitly states blank-node applicability without implying any lattice closure

Should sflo:RdfDataset stop subclassing sflo:DigitalArtifact?

Only remove the subclass if you want sflo:RdfDataset to become a pure content-kind tag usable on non-artifacts (Meshes, Nomens, Agents, etc.). That’s a real semantic broadening, and it will be hard to prevent drift.

If your intent is still “this is a kind of digital artifact (file-like thing), sometimes unnamed,” then keep:

    sflo:RdfDataset rdfs:subClassOf sflo:DigitalArtifact

and treat blank nodes as “artifact exists but isn’t given a stable IRI.”
If you do want the convention “DigitalArtifacts must be IRIs”

Don’t encode that by breaking the class lattice. Enforce it with SHACL (warning or violation), e.g., “node kind must be IRI” for targets you care about. That keeps your ontology semantics clean and your operational constraints explicit.
You said:
Sflo Config Ontology (single Config; Layered By Target)
RdfDataset have a default graph plus 0..n named graphs. Is that true of blank nodes? Which facet are they? I guess we don't have to choose one. My goal is to just be able to use the config predicates as one-offs, and I was imagining needing a separate property, but if blank nodes can fully be ConfigArtifacts, we don't. They're not really file-like though. I guess in our Artifact Resolution process ???
ChatGPT said:

An RDF dataset is an abstract structure: “default graph + 0..n named graphs.” That statement is true regardless of whether you identify the dataset with an IRI or with a blank node. The blank node is just an identifier for some resource that you’re saying “is an RDF dataset.”

Two clarifications that matter for your modeling:
1) “Default graph + named graphs” vs blank nodes

    A blank node can denote a dataset just fine.

    The dataset’s named graphs are part of the dataset structure; they aren’t “the blank node.”

    In RDF 1.1, graph names may be IRIs (and in many systems, also blank nodes). So blank nodes can show up inside datasets as graph names too, depending on your stack.

So: you don’t have to pick a “facet” for a blank-node dataset. It’s just “an RDF dataset resource,” optionally unnamed.
2) Your real issue is operational, not semantic

You’re conflating:

    semantic eligibility (“may be an instance of class X”), with

    artifact resolution / filesystem realization (“I can resolve this to a flow/state/files”).

Blank-node configs are perfect for “one-offs” precisely because they’re not meant to be resolved as standalone artifacts. They live inside some other dataset (e.g., a metadata flow/state) and are interpreted in that context.

That implies a clean rule for your Artifact Resolution process:

    Resolution operates on IRIs (things you can locate/dereference/weave/publish).

    Blank-node configs are inline configuration descriptions, not resolvable artifacts.

3) Don’t make blank nodes “full ConfigArtifacts” unless you like nonsense

In the current config canvas you have:

    conf:Config ⊑ sflo:RdfDataset

    conf:ConfigArtifact ⊑ conf:Config ⊓ sflo:AbstractArtifact

A blank node can be typed sflo:AbstractArtifact in pure RDF terms, but operationally it’s weird because “AbstractArtifact” in your system implies it can have a Flow/States/AbstractFiles/LocatedFiles. If the config is a blank node, you’ve intentionally declined to give it an identity that supports that lifecycle.

So the distinction you already introduced is good:

    Blank node: a conf:Config (inline)

    Named + revisionable: a conf:ConfigArtifact (has its own identity/lifecycle)

If you want to enforce that “ConfigArtifact is actually named,” don’t try to encode it in OWL. Add SHACL later:

    conf:ConfigArtifact nodeKind sh:IRI (Violation)

That matches your intention precisely.
4) The sflo:RdfDataset ⊑ sflo:DigitalArtifact question

This is the dependency you’re bumping into:

If sflo:RdfDataset is a subclass of sflo:DigitalArtifact, and you type a blank node config as conf:Config ⊑ sflo:RdfDataset, then your blank node becomes a sflo:DigitalArtifact by inference.

If your definition of DigitalArtifact is “file-like / artifact-y in a resolution sense,” you probably don’t want that.

Two coherent options:
Option A (recommended given your stated intent)

Treat sflo:RdfDataset as a content-kind classifier, not an artifact-kind.

    Remove rdfs:subClassOf sflo:DigitalArtifact from sflo:RdfDataset.

    Keep sflo:DigitalArtifact for things you want to treat as artifacts (often named/resolvable).

    When you need both, assert both types (or define an intersection class if you insist).

This makes inline blank-node configs totally natural: they can be sflo:RdfDataset without implying “artifact resolution.”
Option B

Keep sflo:RdfDataset ⊑ sflo:DigitalArtifact and accept that:

    blank nodes can be digital artifacts semantically, and

    “file-like / resolvable” is a separate constraint.

If you choose this, then you should stop using OWL subclassing to encode “file-like,” and encode “resolvable” with SHACL (nodeKind IRI) or with a separate class like sflo:ResolvableArtifact.

Given what you said (“They’re not really file-like though”), Option A is the cleaner fit.
Proposed updated comment for sflo:RdfDataset

To reflect blank nodes and avoid “payload”:

    “A classification for resources whose represented content is an RDF dataset (default graph plus 0..n named graphs). Instances may be identified by an IRI or left unnamed (blank node). This typing may be asserted at any modeling layer where it is useful; it does not, by itself, imply artifact resolution or realization behavior.”

If you adopt Option A, that comment becomes strictly true and doesn’t fight your “file-like” boundary.

If you want, I can update the core ontology definition of sflo:RdfDataset (and any dependent classes like conf:Config) in the canvas next, but the key decision is: is RdfDataset content-kind (A) or artifact-kind (B)?
You said:
Sflo Config Ontology (single Config; Layered By Target)
I just want to reconfirm that there's no existing well-known external vocabulary with a class for RdfDataset. dcat:Dataset doesn't mandate RDF, is my understanding.
ChatGPT said:

There are well-known external vocabularies that already provide a class you can use for “RDF dataset” (in different senses), so it’s not accurate that nothing exists.
Option 1: VoID — void:Dataset

If what you mean is “a published/curated RDF dataset” (social/documentary notion of a dataset, often with dumps, SPARQL endpoint links, statistics, linksets, etc.), VoID is the canonical choice: it explicitly models an RDF dataset as an instance of void:Dataset.
Option 2: SPARQL 1.1 Service Description — sd:Dataset

If what you mean is specifically the SPARQL/RDF-1.1 dataset structure (“default graph + zero or more named graphs”), the Service Description vocabulary defines sd:Dataset exactly that way.
(And VoID even discusses aligning sd:Dataset vs void:Dataset and notes the definitional difference.)
Why DCAT isn’t enough by itself

dcat:Dataset is intentionally format-agnostic: it’s “a collection of data … available … in one or more formats,” including APIs/services, not “RDF dataset” specifically.
So dcat:Dataset is good for cataloging, but it does not assert “RDF-ness.”
Practical guidance for your sflo:RdfDataset

    If your intent is structural (“default graph + named graphs”), align sflo:RdfDataset rdfs:subClassOf sd:Dataset (or even equivalent, if you’re confident).

    If your intent is published dataset as a thing people cite/discover, align with void:Dataset.

    If you want both aspects, you can safely do both rdfs:subClassOf axioms; just be aware you’re asserting both “social dataset” and “SPARQL dataset structure,” which might be stronger than you want in some contexts.

So: there is existing vocabulary support; the real decision is which meaning(s) of “dataset” you want your class to commit to.
You said:
Sflo Config Ontology (single Config; Layered By Target)
I think I definitely would includ Option 1. Maybe Option 2 too? "Any collection of triples can be a void:Dataset, while sd:Dataset also requires that the triples are associated with the default graph or named graphs." -- My Artifacts are file-like things. Aren't triples associated with the default graph automatically in TTL/JSONLD files? How could Option 2 NOT apply to my artifacts?
ChatGPT said:

Option 2 can fail to apply for two different reasons: (a) vocabulary intent and (b) what your artifact actually serializes.
1) Option 1 vs Option 2 are talking about different “dataset” ideas

Option 1: void:Dataset (VoID Vocabulary)
VoID defines a dataset as a meaningful collection of RDF triples—it explicitly emphasizes the “social dimension” (published/maintained/aggregated by someone). It does not require talking about default graph vs named graphs.

Option 2: sd:Dataset (SPARQL 1.1 Service Description / SPARQL 1.2 Service Description)
Service Description’s sd:Dataset is a SPARQL dataset: “default graph + zero or more named graphs”, and (in SPARQL 1.1 SD) the default graph MUST be linked with sd:defaultGraph.

VoID itself calls this out: sd:Dataset is narrower than void:Dataset because it requires triples to be associated with the default graph and/or named graphs.

So: if you only mean “this is a published collection of triples”, Option 2 is extra structure you may not want to assert.
2) “But aren’t triples automatically in the default graph in TTL/JSON-LD?”

This is where your intuition is mostly right, but it’s also where the spec-level nuance matters.
Turtle

Turtle is defined as a syntax for an RDF graph, not a dataset.
You can embed that graph as the default graph of a dataset in your application, but that move is outside what the Turtle document itself asserts.

So Option 2 “doesn’t apply automatically” if you’re being strict about what the artifact intrinsically is: a graph serialization (Turtle), not an explicit dataset-with-graphs partition.
TriG

TriG is explicitly a syntax for an RDF dataset.
So Option 2 does naturally fit TriG artifacts.
JSON-LD

The JSON-LD → RDF algorithm is defined as producing an RDF dataset.
However, the JSON-LD spec also explicitly says JSON-LD can be treated as an RDF graph source by ignoring named graphs and using only the default graph.

So even for JSON-LD, whether Option 2 “applies” depends on whether your consumers treat the artifact as:

    an RDF dataset (default + named graphs), or

    just the default graph.

3) The real practical answer for your artifacts

If your “RDF dataset artifacts” are:

    Turtle-only (single graph, no named graphs in the syntax), and you don’t care about graph naming semantics → Option 1 (void:Dataset) is the clean semantic match; Option 2 is an additional modeling commitment.

    TriG / N-Quads / JSON-LD with meaningful named graphs, or you explicitly want “SPARQL dataset” semantics (FROM/FROM NAMED, dataset descriptions, graph naming) → Option 2 (sd:Dataset) fits.

If you still want to type them as sd:Dataset anyway, you’re basically standardizing this convention:

    “Every RDF artifact is interpreted as a dataset whose default graph is the graph parsed from the file (and named graphs exist only when the syntax supplies them).”

That’s coherent—but it’s not forced by Turtle, and it’s slightly misleading if you’re not actually participating in SPARQL dataset semantics (because sd:* is primarily for describing SPARQL datasets/services).
4) About blank nodes and “dataset-ness”

A blank node can absolutely be an instance of void:Dataset or sd:Dataset (blank nodes can be subjects and have rdf:type). Your earlier move—allowing embedded config as a blank node dataset—is consistent RDF practice.

The only “gotcha” is conceptual: a blank node config is a dataset description with no global identifier; that’s fine for “one-off config predicates”.
5) Is there a well-known external class for “RDF dataset”?

Yes, there are at least two, but they mean different things:

    void:Dataset = published/meaningful triple collection

    sd:Dataset = SPARQL dataset (default + named graphs)

And your understanding of dcat:Dataset is aligned: it’s a general cataloging notion and does not mandate RDF.

If you want the least-confusing alignment for sflo:RdfDataset given your “file-like artifacts” constraint, the most defensible stance is:

    keep sflo:RdfDataset as your “this artifact’s content is RDF (graph or dataset)” marker,

    optionally declare it rdfs:subClassOf void:Dataset (social/curation sense),

    only also align with sd:Dataset if/when you explicitly want SPARQL dataset semantics (or for artifacts whose syntax actually carries dataset structure, like TriG).

You said:
Sflo Config Ontology (single Config; Layered By Target)
Crazy complexity here. Are you saying if my data is in TTL, I need to explicitly include named graph statement for it to be considered part of a graph? 

I like how void talks about a dataset spanning multiple RDF documents. Artifacts are really documents, and if not all RDF document (e.g. TTL) imply Dataset-ness, I don't think we should even use the term dataset. Although void does, and I like the void dataset concept's generality. 

So we either change RdfDataset to sflo:RdfDocument OR if void or sd or something else has a class for "RDF Document", we can just re-use that.
ChatGPT said:

No — a Turtle (.ttl) file does not need any “named graph statement” to be “part of a graph.” Turtle is a concrete syntax for serializing an RDF graph (a set of triples).
What Turtle / JSON-LD “are” in RDF terms

    Turtle encodes an RDF graph.

    An RDF dataset (in the RDF 1.1 data model) is: exactly one default graph + 0..n named graphs.

    If you only have Turtle, you have a graph. You can still treat that as an RDF dataset with only a default graph, but Turtle itself doesn’t carry named-graph structure. (For named graphs you’d use TriG / N-Quads, etc.)

So your intuition is right: “dataset” is not automatically implied by “a Turtle document,” unless you’re using “dataset” in the loose sense “this is the unit of RDF I publish.”
Is there a well-known external class for “RDF dataset”?

There are two big, actually-used ones:

    void:Dataset from the W3C VoID vocabulary. It explicitly defines “dataset” as a meaningful collection of triples with a social/publishing dimension.

    sd:Dataset from SPARQL Service Description (used for describing queryable datasets/endpoints). It’s tied to the SPARQL notion “collection of graphs (default + named).”

And yes: dcat:Dataset does not mandate RDF; it’s intentionally format-agnostic.
Is there a well-known external class for “RDF document”?

There’s a key nuance:

    RDF 1.1 Concepts defines the term “RDF document”: a document that encodes an RDF graph or RDF dataset in a concrete syntax such as Turtle, JSON-LD, TriG…
    But that’s terminology in a spec, not a widely-adopted RDF class IRI you can just reuse.

The common reusable class people reach for is foaf:Document (generic “document”), optionally combined with dcterms:format / media type to say “this particular document is Turtle/JSON-LD/etc.” (I’m not citing FOAF here because the searches above didn’t return the FOAF spec text directly; but it’s the standard pattern you’ll see in practice.)
The inconsistency you’re running into

You said:

    DigitalArtifact = file-like things only, and

    you want to type blank-node configs as “RDF dataset-ish.”

Those goals conflict if sflo:RdfDataset is modeled as a subclass of your file-like DigitalArtifact, because blank nodes are not file-like artifacts in your sense.

So you need to decide what sflo:RdfDataset means:
If you want it to apply to blank nodes

Then sflo:RdfDataset cannot mean “file-like artifact.” It has to mean something like “RDF content structure/kind” (a marker that “the described content is an RDF dataset/graph-collection”). In that case:

    Remove sflo:RdfDataset ⊑ sflo:DigitalArtifact.

    Consider aligning it to void:Dataset (social/published dataset) or keeping it purely-internal and avoiding the overloaded word “dataset” (see below).

If you want it to mean “RDF file/document”

Then it should not apply to blank nodes at all. It should apply to AbstractFile / LocatedFile (or whatever level you deem “document-ish”). In that case:

    Rename it to sflo:RdfDocument (or sflo:RdfArtifactDocument if you want to be explicit).

    Use foaf:Document and/or media-type properties for interoperability.

    Your blank-node config can still be conf:Config without needing to also be an “RDF document.”

Why Option 2 (“sd:Dataset requires default/named graphs”) doesn’t exclude Turtle artifacts

Your Turtle file encodes an RDF graph.
An RDF dataset always has a default graph (which can be that graph) and may have zero named graphs.

So there’s no extra ceremony like “assert graph membership” needed. The only distinction is: Turtle doesn’t express multiple graphs; it expresses one graph. The “dataset-ness” is how you package/interpret it at the application/query level.
Recommendation that stays consistent with your goals

If you want to keep your “DigitalArtifact = file-like” boundary and keep “config can be a blank node”:

    Introduce (or rename to) a content marker class, e.g.

    sflo:RdfContent or sflo:RdfGraphContent (applies to blank nodes and to artifacts)

        comment: “Content is representable as an RDF graph (and optionally an RDF dataset if multiple graphs are used).”

    Separately (optional but clean) introduce sflo:RdfDocument as a DigitalArtifact/document class:

    sflo:RdfDocument ⊑ sflo:DigitalArtifact

    and optionally relate it to foaf:Document, plus dcterms:format / media type.

This avoids forcing one class to do two incompatible jobs.
If you insist on one class only

Then you have to pick which job wins:

    Job A: applies to blank nodes ⇒ it’s a content marker ⇒ drop the DigitalArtifact subclassing.

    Job B: means “RDF document” ⇒ it’s file-like ⇒ don’t apply it to blank nodes.

Given your stated use case (“one-off config predicates via blank nodes”), Job A seems the non-negotiable one. That strongly suggests: keep something like sflo:RdfDataset as a marker, but rename it away from “dataset” unless you’re comfortable with the VoID/SPARQL ambiguity.

If you want, I can propose the exact replacement names + comments that preserve your earlier intent (“default graph + 0..n named graphs”) without accidentally asserting “this is a file-like artifact.”
You said:
Sflo Config Ontology (single Config; Layered By Target)
Sorry, I mean "named graph statement" to be consider the default graph in a dataset. 

For "content marker" we can clearly just use void:Dataset (although I hate to privilege an overloaded term. But not sure we need a content marker.

It seems like there isn't a good RdfDocument term to re-use though. That's really what I was meaning with RdfDataset anyhow. Should we call it sflo:RdfArtifact or sflo:RdfDocument ? 

It's always tempting to make Semantic Flow about more than file-like Artifacts. But non-file-like stuff like abstract works or RDF data in general can be handled with domain-specifc vocab. Let's not boil the ocean yet.
ChatGPT said:
Sflo Config Ontology (single Config; Layered By Target)
{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/sflo-config/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "dcterms": "http://purl.org/dc/terms/",
    "vann": "http://purl.org/vocab/vann/",
    "sflo": "https://semantic-flow.github.io/semantic-flow-ontology/",
    "void": "http://rdfs.org/ns/void#",

    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:subPropertyOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" },
    "owl:imports": { "@type": "@id" },
    "owl:inverseOf": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Configuration Ontology",
      "dcterms:description": "Vocabulary for describing configuration artifacts used by Semantic Flow resources. This ontology intentionally does not define key-level merge semantics; resolution is application-defined.",
      "dcterms:creator": "Semantic Flow Project",
      "owl:imports": [
        "https://semantic-flow.github.io/semantic-flow-ontology/"
      ],
      "owl:versionInfo": "0.2.2",
      "vann:preferredNamespacePrefix": "conf",
      "vann:alternateNamespacePrefix": ["sflo-conf"],
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/sflo-config/"
    },

    {

You’re mixing (at least) two different things under the label “RdfDataset,” and that’s what’s creating the apparent complexity:

    A file-like RDF document (bytes on disk; Turtle/JSON-LD/TriG/etc.)

    An RDF dataset structure (default graph + 0..n named graphs) that a system treats as a dataset

Those are related, but not identical.
Turtle/JSON-LD vs “default graph”

    An RDF dataset is defined as “exactly one default graph and zero or more named graphs.”

    A Turtle document (and commonly JSON-LD) is fundamentally a graph serialization; when you load it into a dataset context (e.g., a SPARQL processor), that graph is typically taken as the default graph. There is no special triple/statement inside Turtle that “declares default graph membership”; the dataset-ness is the container/context, not an RDF assertion.

    By contrast, TriG / N-Quads can directly serialize multiple graphs (dataset-y syntaxes).

So: No, you do not need a “named graph statement” to make Turtle “count” as a default graph. The system associates the parsed graph with the dataset default graph when it chooses to treat the input as a dataset.
Existing external vocabularies

You do have well-known external classes—just not for “RDF document” specifically:

    void:Dataset (VoID) is explicitly broad: “Any collection of triples can be a void:Dataset.”
    It also explicitly discusses datasets published as sets of RDF documents.

    sd:Dataset (SPARQL Service Description) is narrower: it’s a dataset in the SPARQL sense (default + named graphs).

    dcat:Dataset is not RDF-specific; it’s a general dataset concept in data cataloging.

What you likely don’t get “off the shelf” is a widely-used class meaning “this particular file is an RDF document” (as opposed to “this resource is a dataset”). That’s why you’re feeling the gap.
Recommendation: split the concepts (minimal “don’t boil the ocean” version)
A) For file-like artifacts: use “RDF document” semantics

If your intent is “this artifact is an RDF document (bytes) in an RDF serialization”, name it something like:

    sflo:RdfDocument (or sflo:RdfArtifact, but “Document” is clearer about the file-like constraint)

Then the comment should be about bytes + syntax, not about abstract dataset structure. For example:

    “A file-like digital artifact whose concrete bytes are an RDF document in an RDF serialization (e.g., Turtle, JSON-LD, TriG, N-Quads), representing at least one RDF graph and—when the syntax supports it—optionally multiple named graphs.”

This stays squarely in your “Semantic Flow is about file-like artifacts” scope.
B) For content markers / blank-node configs: reuse void:Dataset (or skip a marker entirely)

If you want your embedded config blank nodes to be “RDF dataset-y” without pretending they’re artifacts in your facet lattice, void:Dataset is a solid, standard marker and it explicitly tolerates the generality you want.

Trying to make a blank node simultaneously behave as a facet-lattice artifact is the conceptual mismatch:

    If DigitalArtifact implies “file-like + has an IRI + resolvable identity,” then blank-node configs should not be DigitalArtifacts.

    If you later decide a config needs identity/history, you skolemize / mint an IRI and promote it to your ConfigArtifact pattern.

This is the cleanest way to keep “one-off config predicates” simple without collapsing your artifact model.
What this implies for your current sflo:RdfDataset naming

If sflo:RdfDataset was intended to mean “RDF document artifact,” then rename it to sflo:RdfDocument (or deprecate the old name) and keep it firmly on the artifact side.

If you keep a class meaning “any RDF triples collection / dataset-ish thing,” don’t make it an artifact-facet concept—just reuse void:Dataset (or keep it as a very lightweight marker that does not imply file-like identity).

This is also why dcat:Dataset doesn’t solve your need: it’s intentionally domain-general and doesn’t mean “RDF document.”
You said:
Sflo Config Ontology (single Config; Layered By Target)
OK, I renamed to RdfDocument. 

What's the best-known "Agent" class term out there? i.e., human/computer that can do things.

Next debate: Should I really split the general-purpose vocabulary off into a separate ontology?  
ChatGPT said:
Best-known RDF “Agent” class

If you want “human/computer/organization that can do things” with maximum interoperability, the most defensible choice is prov:Agent from W3C PROV-O. PROV defines an Agent as something that bears responsibility for an activity/entity/another agent.

Two other very widely-used “Agent” classes line up cleanly with PROV:

    dcterms:Agent (DCMI): “A resource that acts or has the power to act.” (examples include person, organization, software agent).

    foaf:Agent (FOAF): “An agent (eg. person, group, software or physical artifact).” and FOAF explicitly states it’s equivalent to dcterms:Agent.

    There’s also a W3C note mapping dcterms:Agent owl:equivalentClass prov:Agent (i.e., they’re intended to denote the same concept).

Recommendation for Semantic Flow

    Use prov:Agent as your canonical range/type when you mean “actor” in operational/provenance-ish properties (e.g., your lastVerifiedBy). It matches your existing use of PROV (prov:wasRevisionOf) and keeps the semantics coherent.

    If you want finer typing later, PROV also gives you prov:SoftwareAgent, prov:Person, prov:Organization.

    I would not center Schema.org for this (unless you’re optimizing for JSON-LD “webby” data exchange), because Schema tends to model “agent” via properties and concrete classes (schema:Person, schema:Organization, schema:SoftwareApplication) rather than a single, neutral upper class.

One caution: PROV’s “responsibility” framing can be slightly opinionated if you sometimes mean “process that happened to run” rather than “responsible actor.” If you foresee that ambiguity, keep your range as rdfs:Resource but document “SHOULD be a prov:Agent” and enforce it in SHACL, not RDFS/OWL.
Should you split the general-purpose vocabulary into a separate ontology?

You’re not wrong to want this, but it’s easy to do it in a way that creates busywork without real payoff. The decision hinges on release cadence and reuse boundaries, not aesthetics.
Reasons to split

    Stability boundary: your “general-purpose digital artifact” terms (artifact/state/file/location, etc.) want to stabilize; SF-specific terms (Mesh/Knop/Nomen, conventions) will churn.

    Reuse: if you expect other projects (or other people) to reuse just the artifact model without buying into the whole mesh system, splitting reduces cognitive load and import surface.

    Governance: a smaller “artifact spine” ontology is easier to version, review, and keep backwards compatible.

Reasons not to split (yet)

    Premature modularization: until you have at least two real consumers with different dependency needs, you’ll pay ongoing cost (imports, documentation duplication, release coordination) for a theoretical benefit.

    Namespace/tooling friction: you’ll need to decide whether the split introduces a new namespace (new term IRIs) or just a new ontology document (same term IRIs, different ontology IRI). The latter is doable, but some tooling and humans get confused when “where the term is defined” moves.

The least-regret move

Split by ontology document, not by term namespace:

    Keep your term IRIs the same (so you don’t churn downstream data).

    Create a second ontology document whose ontology IRI is something like “artifact spine”, and have the main SF ontology owl:imports it.

    Move only the truly general concepts (DigitalArtifact, AbstractArtifact/State/AbstractFile/LocatedFile lattice, plus generic relations) into the spine. Keep Mesh/Knop/Nomen and SF operational conventions in the SF ontology.

This gives you the modularity win without forcing a migration.
A consistency check you should apply

If the “general-purpose” ontology still contains assumptions that are only true because of Semantic Flow conventions (e.g., reserved folder names, _mesh/, _nomen/, weave semantics), it’s not actually general-purpose yet—splitting at that point will just hide coupling rather than remove it.

If you want a crisp criterion: split only when you can explain the spine without mentioning Mesh/Knop/Nomen at all.


ChatGPT can make mistakes. Check important info.
