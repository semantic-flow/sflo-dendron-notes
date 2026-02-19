---
id: up2ws57duvidloaknxlvr1i
title: 2026 02 01 Modular Datasets
desc: ''
updated: 1769953682549
created: 1769939201855
---
Skip to content
Chat history
You said:
I think I'm going to leave the general purpose classes and properties in a single "Semantic Flow Ontology". I'd love it if other people would use the general-purpose concepts (the Artifact stack, ReferenceLinks, the ResourcePage concept, and the RdfDocument class), but I don't want to do an Ontology for each of those and I honestly don't think re-use happens mostly with ontology imports, it's mostly in vocab re-use, and that can be as specific or wide as necessary.

I'm considering renaming <SemanticFlowResource> to <MeshResource> again because these are the primary constituents of a mesh: <Mesh> itself, <Nomen> for naming, <Knop> for artifact containment, and then the supporting artifacts. Meshes can contain other resources, but these are the main building blocks. 

Any thoughts? Here's the current ontology:

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix schema: <https://schema.org> .
@prefix vann: <http://purl.org/vocab/vann> .
@prefix prov: <http://www.w3.org/ns/prov#> .

<> a owl:Ontology ;
  dcterms:title "Semantic Flow Ontology" ;
  dcterms:description "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources." ;
  dcterms:creator <https://djradon.github.io/ns/djradon> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology> .

#################################################################
#################################################################
# General-purpose vocabulary
#################################################################
#################################################################

#################################################################
# DigitalArtifact
#################################################################


<DigitalArtifact> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A file-like digital object (documents, specifications, datasets, pages, binary assets), whether abstract or location-specific. A DigitalArtifact may exist inside or outside any Semantic Flow mesh." .

<ArtifactContainer> a rdfs:Class ;
  rdfs:label "Artifact container" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself required to be a DigitalArtifact." .

#################################################################
# Artifact facet lattice
#################################################################

<narrowerFacet> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "narrower facet" ;
  rdfs:comment "Relates a broader DigitalArtifact to a narrower DigitalArtifact facet within the same artifact structure. Intended to be closed (tree-shaped) within a mesh-managed Flow, but may be used more loosely for external artifacts." .

<broaderFacet> a owl:ObjectProperty ;
  owl:inverseOf <narrowerFacet> ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "broader facet" ;
  rdfs:comment "Inverse of narrowerFacet." .

#################################################################
# Core artifact model (AbstractArtifact / Flow / State / AbstractFile / LocatedFile)
#################################################################

<AbstractArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract artifact" ;
  rdfs:comment "The primary, whole-artifact identity. In a Semantic Mesh, each AbstractArtifact has exactly one Flow that supplies its working and historical states." .

<ArtifactFlow> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Flow" ;
  rdfs:comment "The over-time identity for an AbstractArtifact. An ArtifactFlow relates one WorkingArtifactState and zero or more HistoricalArtifactStates." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A single-state content identity within a Flow. ArtifactStates may be mutable (working) or immutable (historical)." .

<WorkingState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Working state" ;
  rdfs:comment "The mutable working state for a Flow. By convention in meshes, this is surfaced at a reserved '_working' path segment." .

<HistoricalState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published snapshot state for a Flow." .

<AbstractFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-level identity for a particular State: a specific file variant (e.g., TTL vs JSON-LD, zipped vs unzipped, or distinct build/canonicalization choices) whose bytes may be served from one or more LocatedFiles. Multiple AbstractFiles may exist for the same State, including multiple variants within the same media type or extension. In mesh file layouts that reserve direct State children for AbstractFiles, AbstractFile identifiers MUST NOT start with '_' and SHOULD NOT be 'index.html'." .

<LocatedFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes identity (typically with an extension). A LocatedFile may be used as a facet of broader artifacts (AbstractFile, State, Flow, AbstractArtifact) and may also be treated as an AbstractArtifact in external/collapsed cases." .

#################################################################
# Core structural relations (also participate in narrowerFacet)
#################################################################

<hasFlow> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <ArtifactFlow> ;
  rdfs:label "has flow" ;
  rdfs:comment "Relates an AbstractArtifact to its Flow. Mesh convention: exactly one Flow per AbstractArtifact." .

<hasWorkingState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <WorkingState> ;
  rdfs:label "has working state" ;
  rdfs:comment "Relates a Flow to its working (mutable) State. Mesh convention: exactly one working state per Flow." .

<hasHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "has historical state" ;
  rdfs:comment "Relates a Flow to a published historical State." .

<previousHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf prov:wasRevisionOf ;
  rdfs:domain <HistoricalState> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "previous historical state" ;
  rdfs:comment "Links a HistoricalState to the immediately preceding HistoricalState in its published revision lineage." .

<latestHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasHistoricalState> ;
  rdf:type owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "latest historical state" ;
  rdfs:comment "Optional convenience pointer from a Flow to its latest (most recently published) HistoricalState. This is a logical pointer in RDF and does not imply any filesystem aliasing." .

<hasAbstractFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <AbstractFile> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Relates a State to one of its AbstractFile variants." .

<hasLocatedFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractFile> ;
  rdfs:range <LocatedFile> ;
  owl:inverseOf <locatesAbstractFile> ;
  rdfs:label "has located file" ;
  rdfs:comment "Relates an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile> ;
  rdfs:range <AbstractFile> ;
  owl:inverseOf <hasLocatedFile> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .

#################################################################
# Optional skip-links (structural narrowing shortcuts)
#################################################################

<locatedFileForState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <LocatedFile> ;
  rdfs:label "located file for state" ;
  rdfs:comment "Optional shortcut relating a State directly to a LocatedFile. When present in mesh-managed data, it SHOULD be consistent with the chain: hasAbstractFile / hasLocatedFile." .

#################################################################
# Content kinds
#################################################################

<RdfDocument> a rdfs:Class ;
  rdfs:subClassOf dcat:Dataset ;
  rdfs:label "RDF document" ;
  rdfs:comment "A DigitalArtifact whose content is at least one RDF graph. Concrete bytes are an RDF document in an RDF serialization (e.g., Turtle, JSON-LD, TriG, N-Quads). May be used to type an AbstractArtifact, a Flow, specific State(s), an AbstractFile, or a LocatedFile, depending on modeling needs." .

#################################################################
# Web presentation
#################################################################

<ResourcePage> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "An HTML page that presents a resource. By convention, resource pages are located at a reserved 'index.html' located-file name. Mesh layout invariant: reserve 'index.html' for ResourcePages; other artifact facets SHOULD NOT use 'index.html' as their identifier token." .

<hasResourcePage> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource with a human-readable presentation page." .

#################################################################
# Reference data
#################################################################

<ReferenceLink> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a resource to a specific reference data source, including role metadata." .

<ReferenceRole> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical> a <ReferenceRole> ;
  rdfs:label "Canonical" .

<ReferenceRole/Supplemental> a <ReferenceRole> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated> a <ReferenceRole> ;
  rdfs:label "Deprecated" .

<hasReferenceLink> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLink> ;
  owl:inverseOf <referenceLinkFor> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <hasReferenceLink> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the resource that a ReferenceLink is about." .

<hasReferenceRole> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range <ReferenceRole> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a resource used as a reference-data identifier." .

<referenceUriLiteral> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

<lastVerifiedAt> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range dcterms:Agent ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .

#################################################################
#################################################################
# Semantic Flow resources
#################################################################
#################################################################

<SemanticFlowResource> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A resource modeled using Semantic Flow conventions and vocabulary. Semantic Flow resources may exist inside or outside a mesh." .

<containsSemanticFlowResource> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactContainer> ;
  rdfs:range <SemanticFlowResource> ;
  rdfs:label "contains Semantic Flow resource" ;
  rdfs:comment "Structural containment relation for Semantic Flow resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

#################################################################
# Semantic Meshes
#################################################################

<Mesh> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains Semantic Flow resources and other supporting resources." .

<meshBase> a owl:DatatypeProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "mesh base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

#################################################################
# Naming
#################################################################

<Nomen> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about." .

<containsNomen> a owl:ObjectProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range <Nomen> ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:label "contains nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<designatorPath> a owl:DatatypeProperty ;
  rdfs:domain <Nomen> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root Nomen." .

<designates> a owl:ObjectProperty ;
  rdfs:domain <Nomen> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for token management)." .

#################################################################
# Knops + payload artifacts
#################################################################

<Knop> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for primary payload artifacts and supporting artifacts." .

<containsKnop> a owl:ObjectProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range <Knop> ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<PayloadArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <SemanticFlowResource> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary AbstractArtifact hosted by a Knop." .

<containsPayloadArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <PayloadArtifact> ;
  rdfs:label "contains payload artifact" ;
  rdfs:comment "Links a Knop to its primary payload AbstractArtifact." .

<payloadSlug> a owl:DatatypeProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range xsd:string ;
  rdfs:label "payload slug" ;
  rdfs:comment "Stable, immutable slug used as an AbstractArtifact's identifier path segment in a mesh file layout (folder name). Changing it changes IRIs and should be treated as minting a new AbstractArtifact. Mesh invariant: payloadSlug MUST NOT be 'index.html' and SHOULD NOT start with '_' (reserved for structural items)." .

<preferredPayloadFileSlug> a owl:DatatypeProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload file slug" ;
  rdfs:comment "Mutable filename-stem preference for LocatedFiles (e.g., 'alice-bio'). Generators may ignore/sanitize it and must resolve collisions mechanically (e.g., suffix or hash). Mesh invariant: should avoid values that would result in a LocatedFile named 'index.html'." .

<preferredPayloadSlug> a owl:DatatypeProperty ;
  owl:deprecated true ;
  rdfs:domain <Knop> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload slug" ;
  rdfs:comment "DEPRECATED. Replaced by payloadSlug (immutable identifier segment) and preferredPayloadFileSlug (mutable filename preference)." .

#################################################################
# Metadata payload artifacts
#################################################################

<KnopMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Knop metadata artifact" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop. Modeled as an AbstractArtifact with a Flow and States." .

<MeshMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Mesh metadata artifact" ;
  rdfs:comment "Optional mesh-level operational metadata/config. Modeled as an AbstractArtifact with a Flow and States." .

<NomenMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Nomen metadata artifact" ;
  rdfs:comment "Metadata about a Nomen. Modeled as an AbstractArtifact with a Flow and States." .

<containsKnopMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <KnopMetadataArtifact> ;
  rdfs:label "contains knop metadata artifact" ;
  rdfs:comment "Links a Knop to its knop metadata artifact (if present)." .

<containsMeshMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Mesh> ;
  rdfs:range <MeshMetadataArtifact> ;
  rdfs:label "contains mesh metadata artifact" ;
  rdfs:comment "Links a Mesh to its mesh metadata artifact (if present)." .

<containsNomenMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Nomen> ;
  rdfs:range <NomenMetadataArtifact> ;
  rdfs:label "contains nomen metadata artifact" ;
  rdfs:comment "Links a Nomen to its nomen metadata artifact (if present)." .

#################################################################
# Inventory artifacts
#################################################################

<MeshInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Mesh inventory artifact" ;
  rdfs:comment "Index for explorability at the mesh level." .

<KnopInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Knop inventory artifact" ;
  rdfs:comment "Index for explorability at the knop level." .

<NomenInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Nomen inventory artifact" ;
  rdfs:comment "Index for explorability at the nomen level." .

<containsMeshInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Mesh> ;
  rdfs:range <MeshInventoryArtifact> ;
  rdfs:label "contains mesh inventory artifact" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryArtifacts (may be large and/or sharded)." .

<containsKnopInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <KnopInventoryArtifact> ;
  rdfs:label "contains knop inventory artifact" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryArtifacts (may be large and/or sharded)." .

<containsNomenInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Nomen> ;
  rdfs:range <NomenInventoryArtifact> ;
  rdfs:label "contains nomen inventory artifact" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryArtifacts (may be large and/or sharded)." .
ChatGPT said:
1) Biggest issue: several prefixes are currently wrong, which breaks many IRIs

In Turtle, a prefixed name expands by string concatenation of the prefix IRI + local part.
Because these three prefixes lack a trailing / or #, you are minting unintended IRIs like http://purl.org/dc/termstitle and https://schema.orgWebPage:

    @prefix dcterms: <http://purl.org/dc/terms> . should be .../terms/

    @prefix schema: <https://schema.org> . should be https://schema.org/

    @prefix vann: <http://purl.org/vocab/vann> . should be .../vann/

Minimal fix:

@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix schema:  <https://schema.org/> .
@prefix vann:    <http://purl.org/vocab/vann/> .

<> vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .

Related: vann:preferredNamespaceUri also really wants the trailing / for the same reason (even if it’s “just metadata”, it guides how others mint terms).

If you do nothing else, do this—because right now your ontology is internally inconsistent at the IRI level.
2) Single “Semantic Flow Ontology” file: your rationale is mostly correct, but there’s a trap

Empirically, term reuse (cherry-picking classes/properties) is common, while whole-ontology reuse via owl:imports is harder and less prevalent.
So keeping one ontology and expecting “vocab reuse” rather than “import reuse” is realistic.

The trap: if you want outsiders to reuse the “general-purpose” stack, the branding and top-level framing matter. A developer scanning vocabularies will often skip something that looks domain/project-specific, even if the core is generic. If you keep one ontology, compensate by:

    publishing a “General-purpose” entry section in docs (DigitalArtifact stack, ResourcePage, ReferenceLink, RdfDocument),

    clearly stating which terms are intended to be reusable outside Semantic Flow (you already have that section boundary—good),

    optionally publishing modules as separate files without making them separate ontology IRIs (same namespace, multiple distributions). That gives you modular consumption without implying owl:imports should be the reuse mechanism.

3) Renaming SemanticFlowResource → MeshResource: the name fights your own semantics

Right now your definition says:

    “modeled using Semantic Flow conventions … may exist inside or outside a mesh”

That is conformance-to-conventions, not “is part of a mesh”.

If you rename it to MeshResource, most readers will infer one of these (both problematic):

    “This resource is in a mesh.”

    “This resource is a constituent of a mesh (and probably contained by Mesh).”

But you also say meshes can contain other resources and that these are “primary constituents”. That implies there are mesh-contained resources that are not in this class, which makes “MeshResource” a leaky/ambiguous label.

A clean split (recommended):

    Keep SemanticFlowResource = “conforms to SF conventions (even outside a mesh)”.

    Introduce MeshConstituent (or MeshComponentResource) = “the structural building blocks you inventory as part of a mesh surface”.

        Make Mesh, Nomen, Knop, and your SF-managed artifacts subclasses of MeshConstituent.

        Rename containsSemanticFlowResource (if you want) to something like containsMeshConstituent, or keep it but tighten its range.

This avoids conflating (a) “SF-modeled” with (b) “mesh-inventoried”.

If you still want the rename: then you should rewrite the definition to match the new implication, e.g. “a resource that participates in the Semantic Flow mesh model (whether instantiated in a mesh or not)”. That can work, but it’s subtle and people will still misread it as “in a mesh”.
4) Versioning metadata: owl:versionIRI pointing at _working is self-contradictory

You have:

    owl:versionInfo "0.0.1"

    owl:versionIRI .../_working/...

Best practice discussions explicitly distinguish “development-time” vs “published” versions; owl:versionIRI is intended to identify a specific published version, not an in-progress working location.

If your system has _working as mutable by design, then owl:versionIRI should not point there. Put the working pointer somewhere else (your own property, or even dcterms:hasVersion/PAV-style metadata), and reserve owl:versionIRI for immutable releases.
5) Two more conceptual friction points if you want outside reuse

A) RdfDocument ⊑ dcat:Dataset is likely backwards for many consumers
In DCAT, a Dataset is conceptual; a Distribution is the downloadable/accessible file form.
If your intent is “this thing is an RDF file (serialization)”, dcat:Distribution will align better with common DCAT expectations. If your intent is “this thing is a dataset whose content is RDF graphs”, then keep dcat:Dataset—but then be explicit that it’s not file-level.

B) Labels “Flow” and “State” are too generic for cross-context reuse
You have ArtifactFlow rdfs:label "Flow" and ArtifactState rdfs:label "State". Inside your ecosystem it’s fine; outside, it’s ambiguous. If you want reuse, consider “Artifact flow” / “Artifact state” as the labels, even if you keep the compact local names.
Bottom line

    Fix the prefix IRIs first (they currently invalidate lots of your vocabulary by minting unintended terms).

    Keeping a single ontology is consistent with how reuse happens in practice (term reuse beats imports), but you’ll need documentation and packaging to make the “general-purpose” subset discoverable.

    Renaming SemanticFlowResource to MeshResource is likely a semantic regression unless you also refactor the model to separate SF-conformance from mesh-constituency.

You said:
Regarding modules, I don't believe there's good mechanisms for spreading an ontology across multiple files. void:inDataset is close. Maybe I will invent the mechanism, because part of the sflo general-purpose core is about RdfDocuments. Or it's a great use case for "collection Knops" which I touched on before, but got scared off of. Do you remember talking about DatasetSeries support, and then "sets"? Basically Knops with no payload of their own, just references to other Knops?
ChatGPT said:

No—I don’t have any prior “DatasetSeries → sets → collection Knops” decisions or naming from our earlier chats available right now (nothing relevant came back), so I’ll treat this as a fresh design pass.
1) “Modules across multiple files”: what’s actually missing in the ecosystem

You’re right that there isn’t a single, widely-adopted “this file is module A of ontology X” mechanism that:

    keeps one ontology IRI,

    cleanly declares partial graphs as “parts”,

    and gives you a predictable module closure semantics.

The closest existing pieces are:

    owl:imports (but that pushes you toward multiple ontologies, and imports semantics are heavy / tooling-dependent).

    DCAT (dcat:Dataset, dcat:Distribution) and VoID (void:Dataset, void:subset) to describe datasets and subsets; but neither standardizes “ontology module identity + assembly rules”.

    dcterms:hasPart / dcterms:isPartOf for mereology—but no normative “compose these parts to obtain the ontology graph”.

So if you want “one ontology IRI, many files, with explicit assembly semantics”, you either:

    accept “loose” part-whole metadata (good enough for humans and your generator), or

    invent a minimal SFLO mechanism.

Given SFLO already has RdfDocument as a first-class concept, you can justify a lightweight, explicit “module document” pattern without pretending it’s universal.
A minimal mechanism that stays honest

If you invent it, keep it brutally small:

    sflo:OntologyModuleDocument ⊑ sflo:RdfDocument

    sflo:moduleOfOntology (domain module document, range ontology IRI)

    sflo:moduleRole (optional: core / artifacts / naming / reference-links / etc.)

    sflo:moduleDependsOn (optional; purely editorial dependency graph)

Then your build rules can be explicit:

    “the published ontology graph = union of all module docs with moduleOfOntology = <> (plus optionally their dependency closure).”

The important bit: you’re not claiming OWL semantics—just assembly semantics for publication and tooling.
2) “Collection Knops”: this is actually a strong fit, but you need to nail the invariants

Your phrase “Knops with no payload of their own, just references to other Knops” is coherent, but only if you tighten what “no payload” means.

Right now your model implies:

    Knop is an ArtifactContainer.

    containsPayloadArtifact links a Knop to a PayloadArtifact.

If “collection knop” is still a Knop, then either:

    it simply has zero containsPayloadArtifact, or

    it has a payload, but it’s purely an index (which is not “no payload”).

You should decide which, because it affects everything downstream (UI expectations, build expectations, “what is the canonical thing?”).
Option A: True “payloadless” CollectionKnop (cleaner with your current statement)

Introduce:

    sflo:CollectionKnop ⊑ sflo:Knop

    SHACL / rules: containsPayloadArtifact maxCount 0 for CollectionKnop

Then add an explicit membership relation:

    sflo:collectsKnop (domain CollectionKnop, range Knop)

        or sflo:hasMemberKnop / sflo:memberOfCollectionKnop

This keeps “collection” as a purely structural thing—consistent with “Knops with no payload”.

What you gain: clear semantics; can be used for “sets”, curated bundles, module groupings, etc.
What you lose: no single “payload artifact” to hang metadata off of unless you use the existing metadata/inventory artifacts patterns (which you already have).

But note: you already have KnopInventoryArtifact and KnopMetadataArtifact patterns. A CollectionKnop can still have those, without having a primary payload.
Option B: “CollectionKnop has a payload, but it’s just a set manifest” (often more practical)

Here “collection” is an artifact, not just a container. You model a collection as a first-class payload artifact (a manifest dataset), and the knob is just its host.

This is how DCAT-ish thinking tends to go: the collection is a dataset with members.

Problem: it contradicts what you said you want (“Knops with no payload”), and it increases conceptual load.
My recommendation

If you’re “scared off” because collections feel like a can of worms: Option A is safer.

But you must define two invariants, otherwise “collection” devolves into ambiguous “tagging”:

    Membership is about Knops, not about designata.
    i.e., the set contains these Knops (and indirectly their payload artifacts / nomens), not “these people/things”. If you want designatum-level sets too, that’s a separate construct.

    Collection membership is not containment.
    Do not reuse containsSemanticFlowResource for membership. That property reads like filesystem/structural containment. Collection membership is a graph edge that should not imply hierarchy or hosting.

3) How this connects back to “ontology modules”

If you treat each module file as a Knop (or a payload artifact inside a Knop), then a CollectionKnop can be your “module set”:

    One CollectionKnop represents “sflo ontology release assembly set”

    It collectsKnop the module-knops (artifacts, naming, reference-links, etc.)

    Your generator builds the union graph from the collected knops’ RDF documents

That’s exactly the kind of SFLO-native “module mechanism” you were hinting at—without relying on VoID/DCAT to mean more than they do.
4) Critical naming/semantics check

If you do this, avoid calling it “set” in the ontology unless you want to invite set-theory expectations (extensionality, identity by members, etc.). “Collection” is softer and matches real-world usage.

Suggested terms:

    Class: sflo:CollectionKnop

    Property: sflo:collectsKnop

    Inverse: sflo:collectedByKnop (or sflo:memberOfCollectionKnop)

And explicitly state in comments:

    membership edges are editorial and don’t imply hosting, filesystem layout, or dereferenceability.

If you want, I can sketch the exact RDF/SHACL snippets for Option A (including min/maxCount constraints and a tiny “collection inventory” shape) in the same style as your current ontology.
You said:
Regarding #1, void seems close with its backlinks. It's amazing it doesn't have its own RdfDocument class. But if you consider an ontology a void:Dataset (which isn't hard; imo, it must be), void:inDataset is basically a more-generalized moduleOfOntology. Interesting that inDataset doesn't have an inverse. 

Regarding #2, Knops always contain a KnopMetadataArtifact and a KnopInventoryArtifact. So you could specify the collection in KnopInventoryArtifact. I don't know if we need a CollectionKnop subclass or not. It's kinda derived. But I think you're on the right track with Option B. It can just be a Knop that defines a Dataset as its payload. 

Option A, you hang metadata off the MetdataArtifact, no problem.

Anyhow, I don't want specifically a Knop with no payload. I want a way to split a dataset/ontology across multiple artifacts/RdfDocuments. I think I want membership to be about designata, in this case RDF (sub-)graphs. Knops can contain RdfDocuments, and RdfDocuments contain graphs, and I think we can accomplish graph composition by using a Knop with a payload that's a void:Dataset that's composed of other RdfDocument-Knops. 

Ontologies are a perfect use case. 

NO WONDER semantic web never took off, with no standardized, mentally-accessible way to compose datasets from RdfDocuments.
ChatGPT said:

Keeping the “general-purpose” artifact stack, reference links, and page/document concepts in one Semantic Flow Ontology is consistent with your stated goal: maximize vocab reuse without forcing consumers into an imports graph. That’s a real-world alignment with how people actually reuse RDF terms (selective term reuse > ontology-module imports).
1) Renaming SemanticFlowResource → MeshResource: the inconsistency you’d be introducing

Right now you explicitly say:

    “Semantic Flow resources may exist inside or outside a mesh.”

If you rename that class to MeshResource, you’ve created a semantic mismatch: either

    “mesh” means “participates in mesh conventions” (even if not physically inside one), in which case MeshResource is OK but your comments and some properties should stop implying physical containment; or

    “mesh” means “is in a mesh”, in which case your current definition is wrong and your current containsSemanticFlowResource range is too broad.

If your real intent is “primary constituents of a mesh” (Mesh, Nomen, Knop, and the artifact stack), then “SemanticFlowResource” is actually doing a different job (participates in SFLO conventions), and “MeshResource” sounds like a subset (“things that are mesh constituents / mesh-managed”).

A cleaner move than renaming is:

    Keep SemanticFlowResource = “uses SFLO conventions anywhere”

    Add MeshConstituent (or MeshManagedResource) = “is intended to be inventoried/contained by a Mesh”

    Make Mesh, Knop, Nomen, PayloadArtifact, etc. subclasses of MeshConstituent

    Change containsSemanticFlowResource (or introduce containsMeshResource) to range over MeshConstituent

That preserves your “outside a mesh” story while still naming the “mesh building blocks” explicitly.
2) Dataset/ontology “modules”: VoID already has the core pattern you want

What you describe as missing—“a standardized, mentally-accessible way to compose datasets from RDF documents”—is closer to present than it feels, but split across two ideas:
A. Dataset composition/partitioning: void:subset + void:dataDump

VoID explicitly uses void:subset to describe parts of a dataset and also to describe aggregates (“aggregate of A and B”). It also ties subsets to separately downloadable dumps via void:dataDump.

That is essentially “module membership,” without needing a dedicated moduleOfOntology predicate.
B. Multi-document datasets: backlinks via void:inDataset

VoID also explicitly covers “multi-document datasets,” recommending:

    publish one dataset description

    add a backlink from each RDF document to the dataset with void:inDataset

…and it warns against putting multiple void:inDataset on the same document; instead you define a joint dataset and relate datasets via void:subset.

So: you can model a composed ontology as a void:Dataset with void:subset module-datasets, and each module’s concrete file can backlink with void:inDataset.
3) Your current ontology has a blocking conflation: sflo:RdfDocument as dcat:Dataset

If you want “dataset/ontology split across multiple artifacts/RDF documents,” you need to keep three levels mentally separate:

    Conceptual dataset (set of triples): void:Dataset and/or dcat:Dataset

    Concrete document / dump (a retrievable file): what your LocatedFile models very well

    Container/versioning (Knop/Flow/State/AbstractFile/LocatedFile)

Right now, sflo:RdfDocument rdfs:subClassOf dcat:Dataset pulls (1) and (2) together. That will fight you constantly when you start using void:dataDump/dcat:distribution because those expect a dataset → distribution/file relationship.

Minimal fix (conceptually): treat sflo:RdfDocument as a file/distribution-like thing (probably a subtype of LocatedFile and optionally a dcat:Distribution), and use void:Dataset / dcat:Dataset for the conceptual dataset.

You can keep your current term name and just change its superclass later, but be aware: until you do, “composed dataset” modeling will feel inside-out.

Also: your prefixes for dcterms, schema, and vann are missing the trailing / in this snippet, which will mint the wrong IRIs (http://purl.org/dc/terms vs http://purl.org/dc/terms/, etc.). That’s not cosmetic; it will break downstream reuse.
4) A concrete “collection knop = composed ontology” pattern that stays SFLO-native

This matches what you’re converging on: “a Knop whose payload is a dataset-manifest composed of other Knops.”
Shape of the pattern

    A collection Knop has a payload that describes a void:Dataset (the ontology-as-dataset)

    That dataset points to module datasets via void:subset

    Each module dataset points to its file(s) via void:dataDump

    Each module file backlinks with void:inDataset

In Turtle (illustrative, not prescribing your exact IRI scheme):

@prefix void: <http://rdfs.org/ns/void#> .
@prefix sflo: <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix dcterms: <http://purl.org/dc/terms/> .

# The “collection knop” (normal Knop)
<knops/ontology/> a sflo:Knop .

# The conceptual dataset/ontology being assembled
<dataset/ontology/> a void:Dataset ;
  dcterms:title "Semantic Flow Ontology" ;
  void:subset <dataset/module-core/> ;
  void:subset <dataset/module-artifacts/> ;
  void:subset <dataset/module-nomen/> .

# Each module is itself a dataset (subset)
<dataset/module-core/> a void:Dataset ;
  dcterms:title "Core terms" ;
  void:dataDump <files/module-core.ttl> .

<dataset/module-artifacts/> a void:Dataset ;
  dcterms:title "Artifact stack" ;
  void:dataDump <files/module-artifacts.ttl> .

# Backlink from each RDF document to its dataset
<files/module-core.ttl> void:inDataset <dataset/module-core/> .
<files/module-artifacts.ttl> void:inDataset <dataset/module-artifacts/> .

This is exactly the “multi-document dataset + composition” story VoID intends: void:subset for partitions/aggregates and void:inDataset for backlinks.
If you truly mean “membership is about subgraphs, not files”

If you need explicit graph-level membership (e.g., “this module contributes named graph G inside a TriG”), VoID doesn’t give a first-class “graph member” term, but SPARQL Service Description does give a standard way to describe datasets in terms of default graph + named graphs (sd:Dataset, sd:namedGraph, sd:name).

You can use that to describe the assembled dataset as a graph collection, even in a static setting (you’re describing structure, not necessarily an endpoint).
5) The hard problems you still have to choose on purpose

This is where “mentally accessible” lives or dies:

    Union semantics: Is composition just RDF-graph union? Usually yes. But if you allow “overrides,” that’s no longer RDF semantics; it’s a build rule you must specify.

    Blank node scope across modules: If you ever expect cross-module identity through bnodes, it will fail. You’ll need skolem IRIs or a “no cross-module bnode identity” rule.

    Version pinning: does the composed dataset point to current working states of modules, or pinned historical releases? Your Flow/State model can express either, but your “assembly dataset” needs to say which.

    Collision policy: what happens if two modules assert contradictory ontology metadata (two owl:Ontology headers, two owl:versionIRI, etc.)? You should treat that as a validation error unless you define precedence.

Recommendation summary

    Don’t rename SemanticFlowResource to MeshResource unless you also tighten the semantics; better: add a subclass for “mesh constituents/mesh-managed” and keep SemanticFlowResource as the umbrella.

    Use VoID directly for “modules”: void:subset, void:dataDump, and void:inDataset already encode the backbone of what you want.

    Fix the conceptual conflation: sflo:RdfDocument should not be a dcat:Dataset if you want datasets composed of documents.

    If you need graph-level membership, layer in SPARQL Service Description’s dataset/named-graph terms (sd:Dataset, sd:namedGraph, sd:name).

If you want, I can propose the minimum new SFLO terms (if any) to make this ergonomic without “inventing a new module system”—e.g., a single convenience inverse for void:inDataset, and a single “assembly rule” property that stays explicitly non-semantic (build metadata, not RDF meaning).
You said:
"Semantic Flow resources may exist inside or outside a mesh." is wrong. Can you write me a new comment that reflects that they're the primary constituents of a Mesh?
ChatGPT said:

A resource that serves as a primary structural building block of a Mesh—used to represent and organize mesh-managed identity, naming, containment, and artifact structure (e.g., Meshes, Knops, Nomens, and related supporting artifacts).
You said:
Regarding your "blocking conflation"... Artifacts are a file/distribution-like thing, with multiple levels. I agree, subclassing dcat:Dataset is wrong, I've removed that. I think yes it should be a subclass of dcat:Distribution, and maybe it should also subclass DigitalArtifact? 

I want to avoid having to specify LocatedFiles for the dataDump. I'd rather specify DigitalArtifacts at the AbstractArtifact level, and let the artifact resolution functionality chose the current ArtifactState and allow application to choose their preferred packaging via AbstractFile to get an actual LocatedFile.

I might not truly mean  “membership is about subgraphs, not files”. And I'm not sure we need to talk about named graphs, although Nomen should be able to name graphs. (I got scared off of addressing named graphs because I can't tell whether they're mutable.) 

I'm fine with composition just being graph-union (not named graphs, just the "content" of RdfDocuments. No cross-module bnode identity is fine too.

Version pinning can be accomplished by specifying the HistoricalState IRI. The assembly RdfDocument can pick and choose. But let's clarify "Current vs Working". I've gone back and forth with whether they are different, and recently copped that latestHistoricalState could be considered current, or WorkingSlice could be considered current, it's up to the application. The problem with latestHistoricalState is it doesn't provide an IRI to name it. Earlier versions of Semantic Flow had separate CurrentState (then called Slice), and on weave Working overwrote Current and created a new HistoricalSlice. Maybe I should bring that back?

Regarding collision policy... Maybe only the "assembly RdfDocument" should specify the Ontology header. 

I've fixed the trailing slash on dcterms, schema, and vann. 

The problem I see with void:subset is that its about void:Datasets not RdfDocuments. 
ChatGPT said:
1) RdfDocument vs dcat:Distribution: your instinct is right, but don’t subclass the wrong layer

If you want SFLO’s “artifact resolution” story (AbstractArtifact → Flow → State → AbstractFile → LocatedFile), then dcat:Distribution aligns most naturally with AbstractFile, not with RdfDocument.

    A DCAT Distribution is “a particular representation / packaging of a dataset” (format, access method). That is exactly what your AbstractFile is: “TTL vs JSON-LD, zipped vs unzipped, canonicalized vs not…”.

    Your RdfDocument is more like a content-kind facet (“this artifact’s content is RDF graphs”) that you want to be able to apply at multiple levels (Flow, State, AbstractFile, LocatedFile).

So the clean alignment is:

    RdfDocument ⊑ DigitalArtifact (content-kind facet; can type any artifact level)

    AbstractFile ⊑ dcat:Distribution (packaging identity)

    (Optionally) LocatedFile can carry dcat:downloadURL/dcat:accessURL values, or you can generate those.

If you instead make RdfDocument ⊑ dcat:Distribution, you’re implying “RDF-ness” is a kind of distribution, which is backwards: RDF is a content form, not the distribution identity itself.

Answer to your question: yes, RdfDocument should also subclass DigitalArtifact if you keep it as a class (it’s clearly “artifact-typed”). But I would not make it subclass dcat:Distribution; I’d put that on AbstractFile.
2) “Avoid specifying LocatedFiles for void:dataDump”: you can, but not with pure VoID semantics

You want to point at AbstractArtifact-level identities and let resolution choose:

    “current/working/historical selection”

    “preferred packaging (AbstractFile)”

    “actual bytes (LocatedFile)”

That is not what void:dataDump means in practice: a VoID data dump is expected to be a retrievable document/URL for the dump.

So you have two honest options:
Option 2A (recommended): author in SFLO terms, publish derived VoID

    In authoring, you say “this dataset’s dump is this AbstractArtifact (or Flow/State/AbstractFile)”.

    In publication (inventory generation), you materialize standard VoID triples:

        choose the target State (working/current/historical pin)

        choose canonical AbstractFile(s)

        emit void:dataDump pointing to the resolved LocatedFile(s)

This keeps your authoring ergonomic and your published metadata interoperable.

You only need one SFLO property, e.g.:

    sflo:dataDumpArtifact (domain void:Dataset or your assembly dataset class, range DigitalArtifact)

…and then your generator emits void:dataDump for real.
Option 2B (less good): put AbstractArtifact IRIs directly in void:dataDump

This “works” syntactically (VoID doesn’t machine-enforce ranges), but semantically it will confuse any consumer that expects void:dataDump to be a concrete dump URL. You’re basically redefining VoID.
3) void:subset “is about datasets not documents”: yes—and that’s exactly why you want it

If your composition semantics is “graph union of module contributions”, you are composing datasets (graphs), not composing documents (files). Documents are just carriers.

The minimal mental model is:

    Assembly: a void:Dataset representing “the union”

    Modules: void:Dataset nodes representing “the parts”

    Documents: your artifact stack produces dump files/distributions for each module dataset

You do not need stable IRIs for module datasets if that feels heavy:

    module datasets can be blank nodes embedded in the assembly document.

    you still get clean “subset” semantics without exploding your IRI surface.

4) Membership “about subgraphs, not files”: you can ignore named graphs and still be coherent

If you mean “each module contributes some triples; union them”, you don’t need named graphs at all.

Your only hard rule should be explicit:

    No cross-module blank-node identity is assumed.

That’s good. It’s predictable. It should be stated as a normative constraint.

On your named-graph anxiety: named graphs are not “mutable” or “immutable” in themselves—the denotation can change just like any other IRI can. If you ever want immutable graphs, you already have the machinery: make “graph states” be your immutable identities. But you don’t have to go there for module union.
5) “Current vs Working”: you’re stuck because you want a nameable pointer, not just a pointer property

Right now you have:

    hasWorkingState (fine)

    latestHistoricalState (fine as a convenience pointer)

Your complaint is correct: latestHistoricalState gives you a pointer but not a stable IRI for “current”.

If you want something you can name (“current”) without naming a specific historical version, you need either:
Option 5A: reintroduce a CurrentState pointer resource (best fit to your past model)

    hasCurrentState (functional) → CurrentState

    currentResolvesTo → a HistoricalState (or possibly to WorkingState if you want to allow that)

    On weave/release: mint new HistoricalState, update currentResolvesTo

This gives you a stable IRI you can hang things off of and reference in other datasets.
Option 5B: declare that “current” is only the object of a functional property

    currentState (functional) → either WorkingState or latestHistoricalState

    No dedicated “current” IRI exists.

This is simpler, but it doesn’t solve your “name it” requirement.

Given your stated need (“latestHistoricalState doesn’t provide an IRI to name it”), Option 5A is the clean solution. It’s not bloat; it’s fixing a real expressivity gap.
6) Collision policy: yes—modules should not carry ontology headers

If your assembly RdfDocument is “the ontology”, then modules should be treated as partial graphs and must not:

    declare <> a owl:Ontology

    set owl:versionIRI, vann:*, etc.

Make that a rule. Otherwise, union becomes garbage.
7) One concrete pattern that matches everything you want

    An assembly Knop payload is an RDF document describing a void:Dataset (the composed ontology dataset).

    It uses void:subset to point to module datasets.

    Each module dataset uses sflo:dataDumpArtifact (author-time) to point to the AbstractArtifact (or pinned HistoricalState).

    Published inventory emits void:dataDump to the resolved LocatedFile(s).

This gives you:

    composition = union

    pinning by HistoricalState IRI

    ergonomic authoring

    interoperable VoID publication

    no named graphs required

If you want, I can write the exact SFLO triples you’d add (the minimal sflo:dataDumpArtifact property + the CurrentState pointer pattern) in your house style, including the comments and the “derived-to-VoID” rule text.
You said:
Regarding #1, maybe we don't subclass dcat:Distribution. Or maybe I need to re-examine my Artifact stack. ArtifactState is like the "content" of an artifact. In the RDF case, it's closest to a void:Dataset. I was mentally conflating the file-like part of the stack with the whole thing. 

The problem with AbstractFile ⊑ dcat:Distribution is that AbstractFile are not necessarily datasets. They could realize images. 

Do you remember why I dropped _current?
ChatGPT said:

I don’t have any prior note or retrieved discussion that records why you dropped _current (nothing came back), so I can’t truthfully “remember” the rationale.

What I can do is reconstruct the most likely reasons you would have dropped it, based on the design tensions you’ve been circling—then you can confirm/deny.
1) Your #1 point is correct: you were conflating “dataset-ness” with “file-ness”

If ArtifactState is the “content identity,” then:

    RDF case: ArtifactState is closest to a void:Dataset (a set of triples / graphs).

    Non-RDF case (image, PDF): ArtifactState is still “content identity,” but not a dataset in the RDF sense.

That implies dcat:* and void:* should be applied selectively (only for RDF-ish cases), and not baked into the core artifact stack classes as supertypes.
2) Why you probably dropped _current (reconstructed)
Reason A: _current felt redundant with “current = latest historical pointer”

Once you introduced something like latestHistoricalState (or a catalog pointer to “latest released”), _current looked like a second mechanism for the same idea:

    Either you dereference _current as an alias on the filesystem, or

    You look up latestHistoricalState in RDF and resolve to a pinned historical version.

If your philosophy shifted toward “no redirects, no aliases, let catalogs resolve,” _current becomes aesthetic clutter.
Reason B: _current blurs “stable published” vs “mutable working”

You’ve gone back and forth on whether “current” means:

    latest published (immutable historical), or

    whatever is working (mutable).

A path segment named _current invites people to assume it’s stable and citeable, but in many workflows it actually means “whatever’s latest right now,” which is citation-hostile.
Reason C: It creates a naming ontology problem

_current is not “a version.” It’s a pointer. Treating it like a versioned thing in the hierarchy creates confusion:

    Is _current part of the dataset series identity?

    Does it have provenance?

    Is it a state, a distribution, a redirect, or a dataset?

You likely cut it because it muddied the conceptual model.
Reason D: You wanted fewer reserved path segments

Your system already reserves _working, _catalog, _nomen, _mesh, etc. _current adds yet another reserved segment, and those accumulate complexity fast.
3) On AbstractFile ⊑ dcat:Distribution: your objection is decisive

You’re right: DCAT distributions are distributions of datasets. Images can be modeled as datasets in a very broad sense, but that’s stretching DCAT beyond how most consumers interpret it.

So either:

    Don’t use dcat:Distribution in the core artifact stack at all, or

    Use it only in the RDF-specific layer, e.g.:

        RdfDistribution ⊑ AbstractFile ⊓ dcat:Distribution

        and keep generic AbstractFile DCAT-free

That keeps your artifact model general-purpose without forcing DCAT semantics onto non-dataset content.
4) The actionable takeaway for “current” going forward

If you need a nameable “current” IRI (your complaint: “latestHistoricalState doesn’t provide an IRI to name it”), then you reintroduce something like _current—but as an explicit pointer resource, not as “a version”.

Concretely:

    Make a CurrentState resource (stable IRI) that resolves to a particular HistoricalState.

    _current can be the IRI token for that pointer, not a fake version.

That gives you the benefits of _current without the conceptual lie.

If you paste your older _current pattern (even a sketch), I can rewrite it into a pointer-resource pattern that’s consistent with your current stack and avoids DCAT/VoID misuse.
You said:
My current rejection is definitely not D. Maybe we have a new class CurrentStatePointer and  <_current> distributions could just have a single statement, <_current> pointsToHistoricalState <whatever-state>. And its ResourcePage could try to client-side redirect.

Regarding #1, is it okay that I mix the concept with the file? 
ChatGPT said:

Your _current idea is workable, but you should be explicit about what _current is (a pointer), and avoid accidentally treating it as a “state” or a “distribution of a dataset.”
_current as a pointer resource

Define a dedicated class and a single functional pointer property:

    sflo:CurrentStatePointer — a pointer resource, not a state

    sflo:pointsToState (functional) — range sflo:ArtifactState

        optionally sflo:pointsToHistoricalState ⊑ sflo:pointsToState with range sflo:HistoricalState

Example:

<_current/> a sflo:CurrentStatePointer ;
  sflo:pointsToHistoricalState <_history/v17/> .

Two important constraints (otherwise this becomes a citation hazard and a reasoning hazard):

    Never type <_current/> as ArtifactState (or HistoricalState). It is not content; it is a moving reference.

    Make pointsToState functional. If _current can point to multiple states, it’s no longer “current”.

About “client-side redirect”

A “resource page tries to client-side redirect” is still a redirect in practice. It also breaks bookmarking/reading workflows and can create confusing URL drift. Better: the resource page for <_current/> should present:

    “Current resolves to: <_history/v17/>” (prominent)

    a primary “Open resolved state” link

    optionally a tiny script button (“Open automatically”) if you really want convenience

That preserves your “no redirects” posture while still being ergonomic.
Is it OK to mix the concept with the file?

It’s OK only as an explicit, constrained modeling mode, not as your default.
When mixing is acceptable

Mixing “conceptual thing” + “file-like thing” on one IRI is acceptable when:

    you are intentionally using a collapsed identity (one IRI stands for both the abstract artifact and its canonical bytes), and

    you’re not relying on clean separation for reasoning, discovery, or multi-format packaging.

You already acknowledge this possibility in your own text for LocatedFile (“may also be treated as an AbstractArtifact in external/collapsed cases”). That’s the right place to allow it: at the edges.
When mixing will fight your architecture

Mixing becomes a problem when you want all of these simultaneously (you do):

    abstract identity (AbstractArtifact / Flow / State),

    multiple serializations and packages (AbstractFile variants),

    multiple locations (LocatedFile realizations),

    plus dataset composition (VoID-ish “this is a union of parts”).

If you mix “dataset identity” and “document identity” too early, you lose the ability to cleanly say:

    “same dataset, different distributions/formats/locations”

    “these distributions are just carriers”

    “state/content identity changes independently of file packaging”

So: allow mixing only for LocatedFile in collapsed/external cases. Keep your core model separated.
A sharp way to align with your “RDF state ~ dataset content” intuition

    Treat RDF-ness as a facet you can apply to ArtifactState (content identity):

        sflo:RdfState ⊑ sflo:ArtifactState (or just type some states as sflo:RdfContent)

    Treat RDF documents (bytes/serialization) as facets of AbstractFile / LocatedFile:

        sflo:RdfDocument ⊑ sflo:LocatedFile (and/or also applicable to AbstractFile)

    Let VoID “dataset composition” talk about the state-level dataset identity, not the file:

        modules are states/datasets

        dumps/distributions are resolved later (by your artifact resolution)

That avoids the “void:subset is about datasets not documents” objection: in your architecture, that’s a feature, because your composition should be defined at the content/state level anyway.

If you want, I can draft the exact SFLO terms (class + properties + comments) for CurrentStatePointer and the “RDF state/document” split in the same style as your ontology, including the functional-property axioms and the minimal SHACL constraints.
You said:
Please try to draft terms for a state/document split, but in general terms, not specific to RDF
ChatGPT said:

#################################################################
# Content / Document split (general-purpose)
#################################################################

<ContentArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Content artifact" ;
  rdfs:comment "A DigitalArtifact whose identity is determined primarily by information content, independent of any particular byte encoding, packaging, file naming, or retrieval location. Content artifacts are typically realized via one or more document-like artifacts, but may also be treated as 'collapsed' onto a document artifact in external or simplified modeling." .

<DocumentArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Document artifact" ;
  rdfs:comment "A DigitalArtifact whose identity is tied to a document-like carrier (an encoding/packaging/serialization and, optionally, retrievable bytes). Document artifacts may carry or realize content artifacts. In collapsed/external modeling, a document artifact MAY also be treated as a content artifact." .

<RepresentationArtifact> a rdfs:Class ;
  rdfs:subClassOf <DocumentArtifact> ;
  rdfs:label "Representation artifact" ;
  rdfs:comment "A document-level identity for a particular representation of some content artifact (e.g., format/packaging/serialization choices). A RepresentationArtifact may be realized by one or more retrievable byte artifacts." .

<ByteArtifact> a rdfs:Class ;
  rdfs:subClassOf <DocumentArtifact> ;
  rdfs:label "Byte artifact" ;
  rdfs:comment "A retrievable bytes identity (typically addressable at a concrete URL or file path). A ByteArtifact provides access to a particular representation of content." .

#################################################################
# Align existing stack to the split (no new semantics, just typing)
#################################################################

# Content identity
<ArtifactState> rdfs:subClassOf <ContentArtifact> .

# Representation identity (variant of a state/content)
<AbstractFile> rdfs:subClassOf <RepresentationArtifact> .

# Retrievable bytes identity
<LocatedFile> rdfs:subClassOf <ByteArtifact> .

#################################################################
# Generic relations (superproperties for existing specifics)
#################################################################

<hasRepresentation> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ContentArtifact> ;
  rdfs:range <RepresentationArtifact> ;
  owl:inverseOf <representsContent> ;
  rdfs:label "has representation" ;
  rdfs:comment "Relates a content artifact to a document-level representation artifact that carries that content (e.g., a format/packaging/serialization variant). This is a general superproperty; in the core artifact stack, hasAbstractFile SHOULD be treated as a subproperty of hasRepresentation." .

<representsContent> a owl:ObjectProperty ;
  rdfs:domain <RepresentationArtifact> ;
  rdfs:range <ContentArtifact> ;
  owl:inverseOf <hasRepresentation> ;
  rdfs:label "represents content" ;
  rdfs:comment "Inverse of hasRepresentation: links a representation artifact to the content artifact it carries." .

<hasByteArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <RepresentationArtifact> ;
  rdfs:range <ByteArtifact> ;
  owl:inverseOf <byteArtifactFor> ;
  rdfs:label "has byte artifact" ;
  rdfs:comment "Relates a representation artifact to one or more byte artifacts that provide retrievable bytes for that representation. This is a general superproperty; in the core artifact stack, hasLocatedFile SHOULD be treated as a subproperty of hasByteArtifact." .

<byteArtifactFor> a owl:ObjectProperty ;
  rdfs:domain <ByteArtifact> ;
  rdfs:range <RepresentationArtifact> ;
  owl:inverseOf <hasByteArtifact> ;
  rdfs:label "byte artifact for" ;
  rdfs:comment "Inverse of hasByteArtifact: links a retrievable byte artifact to the representation artifact whose bytes it provides." .

#################################################################
# Optional skip-link (mirrors locatedFileForState but stays general)
#################################################################

<byteArtifactForContent> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ContentArtifact> ;
  rdfs:range <ByteArtifact> ;
  rdfs:label "byte artifact for content" ;
  rdfs:comment "Optional shortcut relating a content artifact directly to a byte artifact that provides access to some representation of that content. When present in mesh-managed data, it SHOULD be consistent with the chain: hasRepresentation / hasByteArtifact." .

Key design choices (so you don’t paint yourself into a corner later):

    These classes are not disjoint. You explicitly want to allow “collapsed” modeling where the same IRI is treated as both content and document.

    The new properties are generic superproperties that let you keep your existing hasAbstractFile / hasLocatedFile semantics intact, while making the “content vs carrier” split explicit and reusable outside RDF-specific cases.

You said:
AbstractFile is supposed to be a set of bytes, just not located. 

If <AbstractFile> is the only rdfs:subClassOf <RepresentationArtifact>,why have both? same with ArtifactState/ContentArtifact. I don't think we're actually adding anything here. Maybe we just change the comment on DigitalArtifact from "A file-like digital object (documents, specifications, datasets, pages, binary assets), whether abstract or location-specific. " to "a multi-faceted entity that can have both content and representation. Its content can be diachronic, and its representations can have retrievable location." or something like that. 
ChatGPT said:

You’re right to call out the redundancy: if RepresentationArtifact only ever has one subclass (AbstractFile), it’s not buying you anything. Same with ContentArtifact if it only ever has ArtifactState.

The good news is: your existing stack already is a general state/document split, just not described that way:

    ArtifactState = content identity (what the artifact says/is, independent of packaging/location)

    AbstractFile = unlocated bytes / representation identity (a particular byte-sequence/encoding/packaging of that state)

    LocatedFile = retrievable bytes (a location-specific access identity for some AbstractFile)

So yes: you can get what you want largely by tightening comments (and optionally adding one or two generic superproperties, but you don’t have to).
Minimal change: revise comments to make the split explicit

Here are draft comment updates in your house style, without introducing new classes:

<DigitalArtifact> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A multi-faceted digital object that may be modeled at several identity levels, including (a) diachronic artifact identity over time (AbstractArtifact/Flow), (b) content identity for a particular state (ArtifactState), (c) an unlocated byte-sequence representation of that content (AbstractFile), and (d) one or more location-specific retrievable byte realizations (LocatedFile). Not every DigitalArtifact participates in every facet; collapsed modeling may treat a single resource as both content- and document-like when appropriate." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A single-state content identity within a Flow: what the artifact's information content is for that state, independent of any particular encoding, packaging, or retrieval location. A State may have one or more AbstractFile representations and those representations may have one or more LocatedFile realizations." .

<AbstractFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "An unlocated byte-sequence representation for a particular State: a specific encoding/packaging/serialization choice (e.g., different formats, compression, canonicalization). An AbstractFile is not itself a retrieval location; it may be realized by one or more LocatedFiles that provide access to its bytes." .

<LocatedFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Located file" ;
  rdfs:comment "A location-specific retrievable bytes identity (typically with an extension). A LocatedFile provides access to the bytes of some AbstractFile representation. In collapsed/external cases, a LocatedFile may also be treated as higher-level artifact identity when finer-grained modeling is unnecessary." .

This directly addresses your goal (“state/document split”) in general terms, without introducing new type layers that merely rename what you already have.
If you do want a general-purpose abstraction, make it non-redundant

If you later decide you want “general terms” beyond comments, then ContentArtifact/RepresentationArtifact only make sense if they classify multiple existing classes, e.g.:

    ContentArtifact as a superclass of both AbstractArtifact and ArtifactState (and maybe ArtifactFlow)

    RepresentationArtifact as a superclass of both AbstractFile and LocatedFile

Otherwise, you’re right: it’s just another name for what you already have.
Answer: “is it okay that I mix the concept with the file?”

It’s okay as an explicit collapsed modeling mode, not as your default.

    For “split ontology across multiple artifacts,” you actually benefit from not mixing: keep composition/membership at the ArtifactState level (content), and let resolution choose AbstractFile/LocatedFile (representation/access).

    Mixing becomes attractive only for external data or simple cases where you don’t care about packaging/location/version distinctions.

If you want, I can also draft two generic property comments that reframe your existing properties without adding new ones, e.g.:

    hasAbstractFile explicitly as “has representation”

    hasLocatedFile explicitly as “has retrievable realization”

You said:
If the "assembly artifact" references the AbstractArtifact and/or CurrenStatePointer, we might need some rewriting where, on weave, the current State gets rewritten to use a historical state. Maybe the user has to make a choice, or the weave process relies on a config property, like resolveDigitalArtifactReferencesToHistoricalStateOnWeave. 
ChatGPT said:

You’re identifying the real fault line: what kind of reference is an “assembly artifact” making? If it references AbstractArtifact or CurrentStatePointer, then it’s referencing something intentionally movable. If it references HistoricalState, it’s pinned. Mixing those without a rule will make “weave” semantics unpredictable.
First: avoid rewriting the user-authored assembly, if you can

Rewriting the assembly document on weave is technically doable, but it has nasty properties:

    It makes the user’s source file non-idempotent (two weaves can change it twice).

    It creates merge churn (git diffs in an author-controlled artifact caused by an automated process).

    It blurs “author intent” vs “generated output.”

A cleaner pattern is:

    Keep an authoring assembly that may reference AbstractArtifact / CurrentStatePointer.

    On weave, generate a resolved assembly snapshot that pins every member to a HistoricalState (or a chosen state), and treat that as the publishable/citable artifact.

That way, “resolution” is a derived product, not a mutation of authored intent.
You need to formalize reference modes

Give the assembly membership link an explicit semantics (even if only in comments + a config rule). There are three sane modes:

    Pinned (stable, citeable): member is a HistoricalState

    Floating (dynamic, not citeable): member is an AbstractArtifact or CurrentStatePointer

    Policy-resolved (author floats; weave materializes pin)

Your last option is what you’re describing, and it’s the only one that plays well with “author points at artifacts; build picks packaging/state.”
Recommended weave behavior
Inputs

    Assembly artifact contains membership references to things in one of these categories:

        HistoricalState (already pinned)

        WorkingState (explicitly unstable)

        CurrentStatePointer (resolves to some state)

        AbstractArtifact / ArtifactFlow (must be resolved via your artifact stack)

Outputs

    A resolved assembly artifact state whose membership list is only HistoricalState IRIs (or whatever “stable state class” you define).

Core rule

    “On weave, resolve every non-historical member reference to a HistoricalState, according to policy; emit a new HistoricalState for the resolved assembly itself.”

This gives you:

    stable publication of the assembly

    stable transitive closure of what it includes

    no rewriting of author intent

Where the policy should live

Yes: a config property is appropriate. But don’t make it a single boolean unless you want it to become a trap later. You want at least three choices.
Suggested config term (general)

sflo:artifactReferenceResolutionPolicy

Allowed values (as individuals in a controlled vocab):

    sflo:ResolutionPolicy/PinToHistoricalOnWeave

    sflo:ResolutionPolicy/PreserveFloatingReferences

    sflo:ResolutionPolicy/RequireExplicitPinning (fail weave if anything is floating)

If you insist on a boolean, you’ll end up reinventing this later.
Resolution precedence (must be specified)

If a reference is:

    HistoricalState: keep it

    CurrentStatePointer: resolve via its pointer (then apply “must be historical?” rule)

    AbstractArtifact: resolve by:

        choose flow

        choose target state (latestHistorical vs working) by policy

        if policy requires historical and chosen is working, then either:

            fail weave, or

            force “latestHistorical”, or

            publish working as a new historical (dangerous)

You need to pick one. My recommendation: fail weave if policy requires historical but can’t resolve to one.
Minimal RDF terms to support this cleanly

You can do this without introducing much vocabulary.
A. Assembly membership

sflo:hasAssemblyMember (domain: assembly content/state; range: rdfs:Resource)

Comment should say members MAY be floating references, but weave can resolve them.
B. Optional explicit per-reference override

So the user can override policy for one member without changing global config:

    sflo:AssemblyMemberLink (relator)

    sflo:memberTarget

    sflo:memberResolutionPolicy (optional; overrides global)

    sflo:resolvedToState (filled in only in resolved snapshot, not in authored input)

This avoids forcing the membership predicate itself to carry complicated semantics.
Your proposed “rewrite current to historical on weave”

This is fine only if you limit rewriting to generated artifacts. If you rewrite the user’s authored assembly, you’ll regret it.

So interpret your idea like this:

    The authored assembly can reference CurrentStatePointer.

    The weave process produces a resolved assembly where each pointer is replaced by (or annotated with) the resolved HistoricalState.

    The authored assembly is unchanged.

Edge case you must decide now: what does CurrentStatePointer point to?

If CurrentStatePointer can point to either working or historical, then it is not safe to use as a “stable-ish” reference. That’s okay, but then the weave policy must explicitly decide:

    CurrentMeansLatestHistorical

    CurrentMeansWorking

    CurrentMeansApplicationDefined (not weaveable unless overridden)

If you don’t pick one, your weave output becomes nondeterministic across implementations.
Practical rule-set that won’t paint you into a corner

    Authoring assembly members may be:

        HistoricalState, AbstractArtifact, or CurrentStatePointer

    Weave ALWAYS generates a resolved assembly historical state whose members are ONLY HistoricalState.

    Default policy: PinToHistoricalOnWeave

    If a reference cannot resolve to a HistoricalState, weave fails unless explicitly overridden.

This gives you “safe by default,” and still allows power users to float references intentionally.

If you want, paste the membership predicate(s) you’re currently imagining for assembly composition, and I’ll draft the exact SFLO class/property blocks (labels/comments + minimal axioms) consistent with the rest of your ontology style.
You said:
The WorkingState is always the source, and I wouldn't propose rewriting that. But on Weave, and we need a word for this, the WorkingState should be able to get crystallized, where intra-mesh AbstractArtifact references that aren't already Historical (because those get preserved as-is) are either:
- preserved as underspecified (stay as AbstractArtifact), so dependent on application interpretation
- transformed into contemporary HistoricalArtifact records (for "consistent at the time" snapshotting)
- transformed into CurrentStatePointer references, if applications/users want one thing to always refer to the latest version of something else. 

Maybe that's it, but conceivably there would be case for making the ArtifactFlow IRI useful.

Also, this makes me wonder about allowing whole-mesh weave only. 

Can you come up with use cases for the 3 crystallization possibilities? "transformed into contemporary HistoricalArtifact" is easy, that's basically git. 
ChatGPT said:
Term

“Crystallize” is good. It names the act of producing an immutable, citeable snapshot from working content without implying filesystem aliasing. If you want a more SFLO-ish paired verb for the event: “weave” is the process; “crystallize” is the result of that process for a specific scope.

A useful phrase you can reuse in comments/spec: reference crystallization policy.
Three crystallization outcomes: concrete use cases

Below, “assembly state” means the new immutable state you publish after weave/crystallization, and “intra-mesh reference” means a reference from one mesh-managed artifact to another.
A) Preserve as underspecified (keep AbstractArtifact references)

Meaning: the crystallized state is immutable as bytes/content, but its dependencies are intentionally late-bound. Consumers resolve those dependencies using their own policies (latest historical, current pointer, working, etc.).

Use cases:

    Living documentation / portal pages

        A published page or index should always show “the latest version of the linked things,” but you still want the page itself to be publishable and stable.

        Example: a mesh “home” artifact that links to “the Alice dataset” abstractly, not “Alice v17”.

    Reusable templates / blueprints

        A template artifact includes references like “the canonical shape library,” but you want the template to be reusable in other meshes or across time without rewriting its internals.

        Late binding makes templates portable.

    Policy experiments / A/B resolution

        You want to compare downstream behavior if “current=latest historical” vs “current=working”, without changing the upstream artifact. Keeping references underspecified allows multiple evaluation policies.

    Cross-environment builds

        Dev/staging/prod might resolve references differently. If crystallization pins dependencies, you lose the ability to produce environment-specific assemblies from the same authored state.

    Weakly governed ecosystems

        You’re publishing content into a mesh where “versions exist but aren’t curated.” You intentionally refuse to take responsibility for pinning dependencies; consumers decide what’s authoritative.

Risk (you should acknowledge explicitly): you can’t later reconstruct the exact dependency closure that a consumer used without recording resolution policy + timestamps + resolved results somewhere else.
B) Transform into “contemporary historical” dependencies (pin to specific HistoricalStates)

Meaning: on weave, every intra-mesh reference becomes a reference to the exact historical state that was “current at the time” by some deterministic rule. This is your “git-like” snapshot.

Use cases (beyond the obvious):

    Regulatory / audit / reproducible evidence

        You must be able to say “this report was generated against exactly these versions of the underlying datasets.”

    Scientific / research reproducibility

        A paper’s dataset references must be stable so results can be reproduced.

    Release engineering

        You publish “release bundles” where all dependencies are pinned so that downstream consumers can mirror/cache and get identical behavior.

    Offline durability

        You want a bundle that can be moved to cold storage; pinning is mandatory.

    Debugging provenance

        When something breaks, you need to know exactly what versions were used, without re-running resolution logic.

Design note: you need to define “contemporary” precisely:

    “latestHistoricalState at crystallization time”

    “the dependency’s current pointer target at crystallization time”

    “the dependency’s working state, first forced to become historical” (this is dangerous unless explicit)

C) Transform into CurrentStatePointer references (track-latest dependencies)

Meaning: you publish a stable crystallized artifact, but rewrite some references so they intentionally follow the evolving “current” of another artifact via a pointer resource.

This is not “underspecified” — it’s specified as dynamic.

Use cases:

    Always-latest integration points

        A “compatibility surface” artifact should always resolve to the latest stable of its dependencies.

        Example: a UI page or API description that should always reference the current schema, not a pinned one.

    Canonical indexes and navigation

        Your navigational artifacts should not be re-crystallized on every downstream change. Instead, they refer to current pointers so they “float” without churn.

    Long-lived identifiers for humans

        Humans want to cite “the current version of X” and accept that it changes, but they still want an IRI that denotes that intention clearly.

    Dependency tracking without repeated rewrites

        You want stable assembly artifacts, but you don’t want constant git diffs / new versions just because a dependency evolved.

    Federated meshes / “live imports”

        When a mesh depends on another mesh, and you want to “follow” the upstream’s current releases automatically, a pointer is a clean mechanism.

Risk: if “current” can ever point to working, you’ve made the published artifact semantically unstable in a way that looks stable. You should require that CurrentStatePointer resolves only to HistoricalState unless explicitly overridden.
Why not “ArtifactFlow” references?

You called this out, and you’re right: Flow IRIs can be useful, but only for a specific semantics:

    Flow reference means “the diachronic identity of this artifact, independent of state selection.”

    It is a stronger signal than AbstractArtifact if you want to talk explicitly about revision lineage.

Use cases for Flow references:

    Provenance tooling that wants to traverse the revision chain regardless of which state is selected.

    Subscription / notifications (“tell me when this artifact changes”) — subscribing to a Flow is natural.

    Governance rules (“this assembly must only depend on artifacts whose flows are approved by X”).

I would not use Flow as the default membership/reference target for content inclusion, but it’s useful for meta-level relationships.
Whole-mesh weave only: when it’s necessary

Whole-mesh weave becomes attractive when you choose policy B (“pin contemporaneous historical”), because partial crystallization can create inconsistencies:

    If you crystallize artifact A today and artifact B tomorrow, then “contemporary” across dependencies becomes ill-defined unless you record a mesh-wide weave boundary.

Use cases for whole-mesh weave:

    Release tags: “Mesh release v3” means every artifact’s latest historical is coherent at one boundary.

    Global reproducibility: you want a single timestamp/commit boundary that defines all “current” resolutions.

    Bulk migration: you’re making systematic changes and want one coherent before/after cut.

When whole-mesh weave is overkill:

    When most of your artifacts are independent and you’re not promising global consistency.

    When you prefer cheap incremental publishing.

A compromise that’s often best: allow partial weave, but introduce a mesh-level concept of a WeaveBoundary (or “MeshCrystallizationEvent”) that can be referenced, so “contemporary” always means “as of boundary X”.
Summary mapping: policy → intended semantics

    Keep AbstractArtifact: “stable artifact; dependencies intentionally late-bound”

    Pin to HistoricalState: “stable artifact; dependencies frozen”

    Rewrite to CurrentStatePointer: “stable artifact; dependencies explicitly dynamic but governed”

If you want, I can propose a minimal vocabulary to express this without exploding your ontology:

    one enum for crystallization policy

    one relator class for “artifact reference” with an optional per-link override and a “resolvedTo” slot populated only in crystallized outputs

You said:
Theoretically this could work between meshes as well.

Resolution policy, weave timestamps, and resolved results all get stored (by default), as metadata artifacts and configuration get woven contemporaneously. So if you can recreate the "point-in-time" maybe we don't need crystallization at all. Applications can reconstruct whichever view they want. 

Resource Pages for HistoricalStates should be drawn from the reference data available at the time and stay immutable. I think we can just leave it as "Always use the AbstractArtifact IRI for references, unless you want to talk about a specific HistoricalState."

ChatGPT said:

If you actually persist (a) the resolution policy, (b) the weave boundary (timestamp/identity), and (c) the resolved results (a mapping), then “no crystallization” is mostly a naming issue: you’re still crystallizing the dependency graph, you’re just not rewriting the authored content.

That can work well, but only if you accept a couple of invariants—otherwise your “immutable HistoricalState pages” claim collapses.
1) Your two statements currently conflict unless you add one missing artifact

You said:

    “Maybe we don’t need crystallization; apps can reconstruct whichever view they want.”

    “Resource Pages for HistoricalStates should be drawn from the reference data available at the time and stay immutable.”

    “Always use AbstractArtifact IRIs for references…”

If HistoricalState pages are immutable, they cannot rely on re-resolving AbstractArtifact references later (because later resolution might differ). Therefore, for every HistoricalState that gets a page, you need a captured resolution snapshot that pins how all referenced AbstractArtifacts were interpreted at that time.

So: you don’t need to rewrite references, but you do need to record a resolved closure per weave boundary (or per state) and make HistoricalState pages render against that closure.
2) The minimal model that makes this consistent

Keep authored references as AbstractArtifact (good default). On weave:

    Produce the new HistoricalState for the artifact being woven.

    Produce a Resolution Snapshot artifact (also woven, immutable) that records:

        weave boundary identity (timestamp + maybe a weave event IRI)

        the resolution policy used

        a mapping from each referenced AbstractArtifact (or pointer) to the resolved target (usually a HistoricalState)

        optionally the chosen representation preference if you want deterministic bytes selection

Then the rule becomes:

    HistoricalState ResourcePages MUST render using the Resolution Snapshot for that HistoricalState (not by live resolution).

That gives you:

    immutable pages,

    reproducibility,

    and still allows applications to compute alternate views if they want (they can ignore the snapshot).

3) Use cases for “no rewrite, only capture”

This is the strong part of your approach:

    Stable authored source: no weave-induced git churn in working content.

    Multiple views: “as woven,” “as of latest,” “dev (working),” etc., are all derivable.

    Cross-mesh possible: the snapshot can record remote resolved targets as IRIs.

But cross-mesh only works if remote meshes also preserve historical targets and don’t garbage-collect them, or if you archive/cache what you depend on.
4) What you must decide explicitly (or you’ll get silent nondeterminism)
A) What counts as “reference data available at the time”?

If your HistoricalState page includes content derived from ReferenceLinks that dereference external URLs, you must either:

    snapshot the external reference content into woven artifacts at weave time, or

    relax the immutability claim to “page layout is immutable, but embedded external reference data may drift.”

Your current claim implies the former.
B) What is “point-in-time” across the whole mesh?

If you allow partial weave, “point-in-time” is per-artifact unless you also mint a MeshWeaveEvent (a boundary) that multiple artifacts can attach to.

If you want coherent “mesh release” semantics, you need a mesh-level event. Otherwise, you’ll reconstruct “as of which weave?” ambiguously.
C) If you “always reference AbstractArtifact,” how does a consumer select a state?

You need a normative default for resolution (even if apps can override), otherwise two apps will disagree and you’ll have no way to say one is “correct.”

Common defaults:

    default resolution for AbstractArtifact = latestHistoricalState

    CurrentStatePointer (if used) resolves only to HistoricalState by default

5) Minimal vocabulary additions that avoid rewriting content

You can do this with one relator-like structure (names illustrative):

    sflo:WeaveEvent (or sflo:CrystallizationEvent) — has timestamp, policy identifiers, maybe scope (artifact-level vs mesh-level)

    sflo:ResolutionSnapshotArtifact ⊑ sflo:KnopMetadataArtifact (or a generic metadata artifact class)

    sflo:hasResolutionSnapshot (from HistoricalState to snapshot)

    sflo:ResolvedReference (relator node)

        sflo:referenceSource (the state/artifact whose outbound ref this is about)

        sflo:referencesAbstractArtifact (the original target)

        sflo:resolvedToState (usually a HistoricalState)

        sflo:resolvedAtWeaveEvent (link to weave event)

This keeps your authored graphs clean (AbstractArtifact references), while giving you a deterministic “as-woven” interpretation.
6) Your final rule, tightened so it’s actually implementable

Your version:

    Always use the AbstractArtifact IRI for references, unless you want to talk about a specific HistoricalState.

Works if you add one sentence:

    When publishing a HistoricalState, record a contemporaneous Resolution Snapshot that pins how its AbstractArtifact references were interpreted at weave time; HistoricalState ResourcePages MUST render against that snapshot.

Without that, “immutable historical pages” is not enforceable.

If you want, I can draft the exact RDF blocks (class/property declarations + comments) for WeaveEvent, ResolutionSnapshotArtifact, and ResolvedReference in the same style as your ontology, and show how a HistoricalState would link to its snapshot.
You said:
Let's flesh out the new CurrentStatePointer. It's essentially a reification/derivation of the latestHistoricalState relationship. Should we re-use that property somehow?
ChatGPT said:

You can’t directly “reuse” latestHistoricalState as the pointer relation for a CurrentStatePointer, because its domain/range are currently specific to ArtifactFlow → HistoricalState. If you make CurrentStatePointer latestHistoricalState <h>, then (given your current rdfs:domain on latestHistoricalState) you’ll infer the pointer is an ArtifactFlow, which is wrong.

That said, you can reuse latestHistoricalState in a clean way by making the pointer a reification whose target is derivable into latestHistoricalState.
Recommended pattern
1) New class and pointer property

<CurrentStatePointer> a rdfs:Class ;
  rdfs:label "Current state pointer" ;
  rdfs:comment "A stable, dereferenceable pointer resource whose purpose is to name the current published state of some artifact flow. The pointer may be updated over time; consumers that need a fixed snapshot SHOULD cite the target HistoricalState IRI instead." .

<pointsToHistoricalState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <CurrentStatePointer> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "points to historical state" ;
  rdfs:comment "Resolves this pointer to exactly one HistoricalState, representing the current published state under the mesh's chosen policy." .

2) Link it to the flow

<hasCurrentStatePointer> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <CurrentStatePointer> ;
  rdfs:label "has current state pointer" ;
  rdfs:comment "Associates a Flow with its CurrentStatePointer, which can be used as a stable name for the flow's current published state." .

3) Reuse latestHistoricalState as a derived convenience

Make latestHistoricalState derivable from the pointer via an OWL property chain:

<latestHistoricalState> owl:propertyChainAxiom ( <hasCurrentStatePointer> <pointsToHistoricalState> ) .

This gives you the “reuse” you want without forcing the pointer to masquerade as a flow.
4) Consistency rule (normative, even if enforced via SHACL)

You should state something like:

    If both latestHistoricalState and hasCurrentStatePointer/pointsToHistoricalState are present for the same flow, they MUST agree.

(You can enforce this in SHACL later; you don’t need to bake it into RDFS domains.)
Should you also “reuse” latestHistoricalState by subPropertyOf?

No. Any of these will backfire:

    pointsToHistoricalState rdfs:subPropertyOf latestHistoricalState
    ⟶ implies the pointer is an ArtifactFlow (because latestHistoricalState has domain ArtifactFlow)

    latestHistoricalState rdfs:subPropertyOf pointsToHistoricalState
    ⟶ implies the flow is a CurrentStatePointer (because pointsToHistoricalState has domain CurrentStatePointer)

So: property chain is the right “reuse mechanism,” not subproperty or equivalence.
One critical naming question you should lock down now

Is “current” always “latest historical”?

You’re describing CurrentStatePointer as “a reification/derivation of latestHistoricalState”, which means yes. If you ever allow CurrentStatePointer to resolve to working, you’ve turned “current” into a citation hazard and you’ll need a different class name (or a different property like pointsToState with weaker range).

If you want “always latest published,” keep the range only HistoricalState as above.
Minimal usage example

# Flow-level
<artifact/_flow/> a <ArtifactFlow> ;
  <hasCurrentStatePointer> <artifact/_flow/_current/> .

# Pointer-level (nameable)
<artifact/_flow/_current/> a <CurrentStatePointer> ;
  <pointsToHistoricalState> <artifact/_flow/_history/v17/> .

If you later decide the pointer resource should be mintable at the artifact root (<artifact/_current/>) rather than under the flow, that’s fine; the ontology doesn’t need to commit to the path layout.

If you want, I can draft the SHACL shape for “agreement between pointer and latestHistoricalState,” but the above is the core vocabulary that keeps your model consistent and avoids domain/range inference traps.
You said:
Sure, I'll take that SHACL shape. In JSONLD please, given this context: 

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
ChatGPT said:

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
      "@id": "ArtifactFlowCurrentStateAgreementShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "ArtifactFlow current-state agreement shape",
      "rdfs:comment": "Ensures that when both sflo:latestHistoricalState and sflo:hasCurrentStatePointer/sflo:pointsToHistoricalState are present on a Flow, they agree on the same HistoricalState.",
      "sh:targetClass": "sflo:ArtifactFlow",
      "sh:property": [
        {
          "sh:path": "sflo:hasCurrentStatePointer",
          "sh:class": "sflo:CurrentStatePointer",
          "sh:maxCount": 1
        }
      ],
      "sh:sparql": [
        {
          "@type": "sh:SPARQLConstraint",
          "sh:severity": "sh:Violation",
          "sh:message": "If both sflo:latestHistoricalState and the sflo:hasCurrentStatePointer target are present, they must be the same HistoricalState.",
          "sh:select": "PREFIX sflo: <https://semantic-flow.github.io/semantic-flow-ontology/>\nSELECT $this WHERE {\n  $this sflo:hasCurrentStatePointer ?ptr .\n  ?ptr sflo:pointsToHistoricalState ?ptrState .\n  $this sflo:latestHistoricalState ?latest .\n  FILTER(?latest != ?ptrState)\n}"
        }
      ]
    },
    {
      "@id": "CurrentStatePointerShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "CurrentStatePointer shape",
      "rdfs:comment": "Basic structural constraints for sflo:CurrentStatePointer: it must point to exactly one sflo:HistoricalState.",
      "sh:targetClass": "sflo:CurrentStatePointer",
      "sh:property": [
        {
          "sh:path": "sflo:pointsToHistoricalState",
          "sh:class": "sflo:HistoricalState",
          "sh:minCount": 1,
          "sh:maxCount": 1
        }
      ]
    }
  ]
}

You said:
How does this work: <latestHistoricalState> owl:propertyChainAxiom ( <hasCurrentStatePointer> <pointsToHistoricalState> ) .
ChatGPT said:

In OWL 2 RDF syntax, this:

<latestHistoricalState> owl:propertyChainAxiom
  ( <hasCurrentStatePointer> <pointsToHistoricalState> ) .

means: the two-step path hasCurrentStatePointer then pointsToHistoricalState is a subproperty of latestHistoricalState. In OWL 2 functional-style syntax, it corresponds to:

SubObjectPropertyOf(
  ObjectPropertyChain( hasCurrentStatePointer pointsToHistoricalState )
  latestHistoricalState
)

The operational entailment rule is:

    If X hasCurrentStatePointer Y and Y pointsToHistoricalState Z, then infer X latestHistoricalState Z.

What it does not do (critical)

    It does not make latestHistoricalState equivalent to that chain. It only adds the forward inference from the chain to latestHistoricalState. So if you assert X latestHistoricalState Z, OWL will not invent a Y pointer to satisfy the chain.

    It does not “reuse” latestHistoricalState in the RDFS sense (no subPropertyOf relationship is implied here); it’s a chain axiom, not a subproperty axiom.

Why this is the right reuse mechanism for your pointer design

You correctly avoided subPropertyOf, because latestHistoricalState has rdfs:domain <ArtifactFlow> and would incorrectly type your pointer as a flow if you tried to reuse it directly.

With the chain axiom, you get:

    authoring convenience: you can store the pointer triple(s) and let latestHistoricalState be inferred

    back-compat: any consumer that only knows latestHistoricalState still works

    no domain/range pollution: the pointer remains a CurrentStatePointer, not an ArtifactFlow

One OWL DL side-effect you should be aware of

A property that participates in a chain is considered non-simple (in OWL 2 DL). That matters if you ever want to use latestHistoricalState inside certain OWL constructs that require simple properties (notably some cardinality restrictions).
(Declaring it owl:FunctionalProperty is usually fine, but if you start using it in restrictions later, the “simple property” constraints can bite.)
Inference example in your terms

Given:

<flow1> <hasCurrentStatePointer> <ptr1> .
<ptr1>  <pointsToHistoricalState> <hist17> .

A reasoner can entail:

<flow1> <latestHistoricalState> <hist17> .

That’s exactly the “latestHistoricalState is derivable from the pointer” story you want.
You said:
OK, let's circle back to modularity. Can you summarize how it could work?
ChatGPT said:
What “modularity” means in your current SFLO direction

You want to split a conceptual dataset (an ontology is a perfect example) across multiple independently versioned artifacts, without forcing consumers to use owl:imports, and without forcing authors to point at concrete files (LocatedFile) when they mean “the thing.”

In SFLO terms, the modular unit is content, not bytes:

    Module content lives at ArtifactState (working or historical).

    Bytes are a selectable representation (AbstractFile) and access (LocatedFile) resolved later.

So “modularity” is: a composed view is defined as the union of the content of some set of artifact states.
The pattern in 6 steps
1) Each module is a normal SFLO artifact

For each module:

    AbstractArtifact → ArtifactFlow → WorkingState + HistoricalStates

    The module’s RDF content is the content of a state (your “state/document split” now makes this clean).

No special “module class” required.
2) Create an assembly artifact that defines the composition

Make another AbstractArtifact (“assembly artifact”) whose state contains:

    a membership list of module targets (usually AbstractArtifact IRIs)

    optionally explicit pins (a target HistoricalState IRI)

    optionally “follow current” links (via CurrentStatePointer)

Critical rule: modules should not carry the ontology header; only the assembly’s published view does (otherwise union creates collisions and nonsense).
3) Define composition semantics: union of module state content

Composition = graph union of module contents (not named graphs, unless you later decide otherwise).

Constraints you should state explicitly:

    No cross-module blank-node identity is assumed.

    If a consumer wants deterministic output, it must use the weave snapshot (below).

4) On weave, do NOT rewrite authored assembly membership; record a resolution snapshot

This is the key to making “Always reference AbstractArtifact” consistent with “HistoricalState pages are immutable.”

On weave/crystallization boundary:

    You mint a new HistoricalState for the assembly itself.

    You also mint an immutable Resolution Snapshot (as a metadata artifact/state) that records:

        which resolution policy was used

        which weave boundary/event it was

        for each referenced AbstractArtifact (or pointer), which HistoricalState it resolved to

That makes the assembly’s historical version reproducible without rewriting source.
5) ResourcePages render from the snapshot, not live resolution

For the assembly’s HistoricalState ResourcePage:

    render “members” and “resolved-to” from the contemporaneous snapshot

    the page stays immutable even if referenced artifacts later evolve

Consumers who want alternate views (latest, dev, etc.) can ignore the snapshot and re-resolve, but the published historical view stays stable.
6) Optional: generate conventional interchange metadata and combined dumps

You can optionally emit derived:

    a combined TTL/JSON-LD dump for the assembly historical state

    VoID/DCAT-ish descriptions for external tooling

But those should be generated from the snapshot + artifact resolution, not hand-authored.

This is where you can bridge to void:subset/void:dataDump if you care, without forcing authors to point at LocatedFile.
The three “reference modes” you’re effectively supporting

You don’t need to expose them as separate authoring patterns if you record the snapshot, but conceptually they exist:

    Floating: assembly references AbstractArtifact and is interpreted by application policy.

    Pinned: assembly references a specific HistoricalState.

    Follow-current: assembly references a CurrentStatePointer (which resolves to a HistoricalState).

Your snapshot captures which one happened at weave time.
The gotchas you must acknowledge in the spec

    Determinism requires a snapshot. Without storing resolved results, “always use AbstractArtifact” makes historical rendering and reproducibility impossible.

    External/other-mesh modules only work if the remote side keeps historical states stable and addressable (or you cache what you depend on).

    Ontology header collisions must be forbidden in modules if union is the composition model.

    BNodes: you must state “no cross-module blank-node identity,” or people will infer impossible expectations.

Minimal vocabulary you need (at most)

You can keep this minimal:

    one membership predicate (e.g., hasAssemblyMember or your preferred naming)

    CurrentStatePointer + pointsToHistoricalState + hasCurrentStatePointer (already discussed)

    a resolution snapshot structure (either a single mapping property or a small relator pattern)

Everything else is already in your stack.

If you want, I can draft the resolution snapshot terms in the same style as your ontology (and a SHACL shape that checks that an assembly historical state has a snapshot, and that snapshot entries resolve to HistoricalState).
You said:
Do we really need the resolution policy? Or even a snapshot? If authors want to pin, they can use a HistoricalState IRI. Otherwise, if applications want the a point-in-time, they can reconstruct it themselves. If they want latest, or some combination of latest, they can construct that too. Resource Pages get generated at the time and don't change? Or can be regenerated as well (if say the templates change.) 
ChatGPT said:

You can drop both “resolution policy” and “snapshot,” but only if you’re willing to accept (and document) one of two very different semantics for an “assembly”:

    Assembly-as-recipe (underspecified, resolver-dependent), or

    Assembly-as-dataset (fully determined at each historical state)

Right now you’re mixing expectations from both, which is where the contradictions come from.
Option 1: Assembly-as-recipe (no policy, no snapshot)

Authoring rule: assembly members are AbstractArtifact IRIs (or pointers), unless explicitly pinned to HistoricalState.

Interpretation: the assembly does not denote a fixed dataset unless pinned. It denotes a recipe like:

    “Union the content of these artifacts under whatever resolution rule you choose.”

What you gain

    Minimal metadata.

    Apps can compute “latest,” “as-of time,” or custom mixes.

What you lose (must be explicit)

    No shared meaning of “the assembly’s dataset” unless pinned.

    Two apps can produce different unions from the same assembly artifact and both are “correct.”

    You cannot later say “this assembly’s historical state corresponds to that exact dependency closure” unless the closure is captured somewhere else.

Resource Pages under this option

You can keep Resource Pages immutable as files, but they are then historical renderings, not re-derivable truth:

    If templates change and you regenerate, the page will likely change (because resolution may change), so it’s no longer “the same page.”

    If you want immutability, you don’t regenerate old pages; you publish new pages as new derived artifacts/versions.

Critical consistency requirement: treat ResourcePages as historical build artifacts, not as something you can “recreate” without also preserving the inputs.
Option 2: Assembly-as-dataset (still can omit “policy,” but you need something captured)

If you want each woven HistoricalState of the assembly to denote a specific dataset (a specific set of triples), then you need determinism at weave time. You can get that in one of two ways:
2A) Materialize the union into the assembly’s HistoricalState payload

On weave, compute the union and store it as the assembly’s historical content.

    No snapshot needed because the state is the result.

    This is effectively “crystallization B,” just framed as “publishing the assembled dataset.”

Downside: storage/duplication, and you’ve committed to a computed artifact as the canonical expression of the assembly.
2B) Don’t materialize, but store a resolved closure mapping (a snapshot)

This is the “snapshot” you’re debating.

    You can omit “policy” because the mapping is authoritative.

    You can re-render pages later with new templates while preserving the same closure.

Downside: extra metadata artifact, but it’s the minimal thing that makes “same historical state, re-renderable” coherent.
“Apps can reconstruct point-in-time themselves” is not enough for your stated immutability goal

If you don’t store either:

    the resolved closure, or

    the materialized union, or

    the original rendered page bytes as the canonical output,

then “point-in-time reconstruction” is inherently ambiguous unless you also standardize what “as-of” means:

    latest historical as-of timestamp?

    as-of weave event ordering?

    what if multiple artifacts were woven at different times?

    what if some artifacts had no historical state at that time?

Those are policy decisions. If you refuse to store policy and refuse to store resolved results, you’ve accepted that “point-in-time” is app-defined and non-portable.

That’s fine—just don’t also claim that historical pages are meaningfully immutable beyond “we never overwrite the HTML file.”
Resource Pages: immutable vs regenerable — pick one, or version them

These two statements are incompatible unless you version the pages:

    “Resource Pages get generated at the time and don’t change”

    “Or can be regenerated if templates change”

Regeneration is change unless you:

    overwrite (breaks immutability), or

    mint a new version/derived artifact (keeps immutability)

So the coherent rule is:

    HistoricalState pages are immutable (never overwritten).

    Template changes generate new ResourcePage artifacts/states, or only affect “current” pages.

You can still have “latest UI” without breaking history, but it becomes versioned UI.
Recommendation (minimal, consistent)

If you want the simplest model that still supports stable history and optional re-render:

    Default authoring: assemblies reference AbstractArtifact (floating). Authors pin with HistoricalState only when they care.

    Historical pages: never overwritten. If templates change, re-rendering produces new page versions (or only affects non-historical views).

    No policy stored is acceptable only if either:

        the assembly historical state materializes the union, or

        you accept that the assembly is a recipe, and historical pages are just frozen renderings, not reproducible/derivable.

If you want “recipe semantics” and reproducible re-rendering, then you do need at least a resolved closure snapshot (policy optional).
One sentence to put in the spec that will prevent future confusion

    An assembly that references only AbstractArtifact IRIs denotes a resolution-dependent recipe; only assemblies pinned to specific HistoricalState IRIs (or whose woven states materialize their union) denote a fixed dataset.

If you want, I can draft the exact normative text for:

    “assembly recipe vs dataset” semantics,

    the ResourcePage versioning rule,

    and a minimal “closure mapping” vocabulary that stores resolved results without introducing an explicit policy term.

You said:
Artifact Resource Pages are versioned along with their HistoricalArtifacts, at all 3 levels below Flow. I guess Container Resource Pages and AbstractArtifact and ArtifactFlow Resource Pages are all mutable. Maybe we should have a class for HistoricalResourcePages?

We need to store weave times (probably per Knop/Nomen, in their corresponding InventoryArtifacts). So you can pick your time, and pull the HistoricalState that corresponds. If an artifact had no historical state ever, you'd use its working copy. If it didn't have one at the time... hmm.

Which reminds me, I think we have to get rid of the CurrentStatePointer and just make it CurrentState. Usually you'll create a Payload Artifact from something, so it will have an initial state, that should be the CurrentState and the WorkingState. So everything always has a CurrentState and a WorkingState. If versioning is turned off from creation (i.e., createHistoricalStatesOnWeave false) CurrentState and WorkingState will be the only states. So point-in-time with "ahistorical" artifacts is not guaranteed to be accurate. 

IT's tempting to get rid of WorkingState altogether. But I think we have to keep it.



ChatGPT said:
1) “HistoricalResourcePage” class: yes, but only if you mean a page about a historical thing, not a page that is itself historical

You just introduced two different axes that need to stop being conflated:

    Subject historicity: the page is about a HistoricalState / HistoricalState-level artifact.

    Page artifact versioning: the page file itself can be versioned and immutable.

In your world, “resource pages for historical states are versioned along with their historical artifacts” implies both are true. So a class is useful, but define it precisely:

    sflo:HistoricalResourcePage = “a ResourcePage whose subject is a HistoricalState (or other historical identity) and whose bytes are immutable once published.”

You might also want sflo:MutableResourcePage just to make the editorial distinction explicit (mesh/knop overview pages, container pages, etc.), but you can avoid that if you don’t want taxonomy creep.
2) Storing weave times in InventoryArtifacts: necessary, but not sufficient for “as-of” without a boundary definition

If you store a weave timestamp per artifact, you still don’t have a consistent “as-of time” for a composed view unless you define what you mean by correspondence:

    “the latest historical state with weaveTime ≤ T”

    and if none exists, use WorkingState

That works as a deterministic rule, but you must admit what it implies:

    It is not a guarantee of semantic consistency across artifacts (because artifacts can be woven at different times).

    It is consistent enough to support “time travel” queries.

The tricky case you noticed—“if it didn’t have one at the time”—is handled by the rule above: if the last historical state is older than T, you use it; if there’s none, fall back to working. That yields a defined answer even when the world was “missing” at that time, but it’s not “accurate to the past,” it’s “best-effort as-of.”

You should explicitly name this semantics, e.g. “as-of projection”.
3) Dropping CurrentStatePointer in favor of CurrentState: you’re right, but only if you commit to “two states always exist”

Your new position:

    every Flow always has WorkingState and CurrentState

    initially, they are the same

    on weave:

        optionally mint a HistoricalState

        update CurrentState (pointer) to that historical

        working remains the mutable source

This eliminates the “IRI to name current” problem because CurrentState is the nameable thing.

This is stronger and simpler than CurrentStatePointer. The pointer class was only needed because you didn’t have a nameable “current.”

However, to avoid making CurrentState just a disguised pointer, you must decide what it is:

    If CurrentState is itself a subclass of ArtifactState (content identity), then it must denote content. But if it “moves” on weave, then it is not a stable content identity. That is philosophically inconsistent.

So the right modeling is:

    CurrentState is not an ArtifactState.

    It is a state handle (a pointer resource in the state namespace), and it resolves to a specific state identity.

Which sounds like… your pointer again, but with a better name and a stronger place in the model. So: you’re not really getting rid of the pointer concept; you’re rebranding it to “CurrentState” and embedding it into the state lattice as a first-class handle.

That’s fine. Just don’t pretend it’s the same kind of state as “HistoricalState” and “WorkingState.”
4) You cannot remove WorkingState without losing the ability to talk about “mutable source”

If WorkingState goes away, you have two bad options:

    Make CurrentState mutable (then “current” becomes citation-hostile and you lose stable publishing semantics), or

    Make “working” be just “latest state” (then you can’t refer to the stable published version without pinning).

Your intuition is correct: keep WorkingState.
5) What happens when createHistoricalStatesOnWeave = false

Your conclusions are correct:

    If no historical states are ever created, then “as-of projection” becomes partially meaningless: you can still compute it, but it degenerates to “working/current” for almost everything.

    That is acceptable as long as you state it: time travel is best-effort and only exact when historical states exist.

6) A coherent, minimal state model that matches what you just said

Here is the tight version that avoids the conceptual lie of “CurrentState is a real state” while still giving you the ergonomics you want:

State identities:

    WorkingState (mutable content state)

    HistoricalState (immutable content state(s))

    CurrentStateHandle (nameable handle; not content)

Relations:

    hasWorkingState: Flow → WorkingState (functional)

    hasHistoricalState: Flow → HistoricalState (0..n)

    hasCurrentState: Flow → CurrentStateHandle (functional)

    currentResolvesTo: CurrentStateHandle → ArtifactState (functional; normally to HistoricalState, optionally to WorkingState if you allow)

    Optional derived convenience:

        latestHistoricalState can be computed from currentResolvesTo if you enforce “current always points to latest historical when versioning is on.”

This gives you:

    nameable current

    stable historical states

    explicit mutable working source

    deterministic as-of projection

7) What you should do next (most important)

You need to pick and write down one deterministic rule for “as-of time resolution” for intra-mesh references:

Given time T and an AbstractArtifact A:

    pick the Flow for A

    find the newest HistoricalState with weaveTime ≤ T

    if found, use it

    else use WorkingState (and mark the result as “ahistorical fallback”)

This rule alone makes all your later modularity decisions much easier.

If you want, I can draft:

    the OWL/RDFS term blocks for CurrentStateHandle and currentResolvesTo,

    plus SHACL shapes enforcing:

        every Flow has exactly one WorkingState and one CurrentStateHandle,

        CurrentStateHandle resolves to exactly one ArtifactState,

        when versioning is enabled, current must resolve to a HistoricalState,

        and weaveTime constraints used for “as-of projection.”

You said:
We already had WorkingState as a "mutable state". In the same way that the content of named graphs can change without getting a new identifier, WorkingState and CurrentState can too. The difference is WorkingState can vary between weaves. 

There's a bit of a question of, whether the HistoricalState gets created from the CurrentState before or after the WorkingState overwrites the CurrentState, i.e., does the HistoricalState represent the previous state at Weave time, or the current state at weave time. 

Maybe when you create the Artifact, If createHistoricalStatesOnWeave is on, I'm guessing it's best to create a HistoricalState immediately (call it "first weave"). That's consistent with the HistoricalState getting created simultaneously and identically to the CurrentState on Weave.  

I don't think it make sense to create an Artifact if there is no initial artifact, but you could always create one with an empty-ish file I suppose.

If CurrentState is a full state, I guess you "fall back to current", not fall back to working.

So yes, at least two states always exist. But if createHistoricalStatesOnWeave is on at creation, at least three states always exist, and at least two of them are identical. 
ChatGPT said:

You’ve basically shifted “state” from “content identity” to “state handle” for WorkingState and CurrentState (mutable like a named graph IRI), while keeping HistoricalState as the immutable, citeable content identity. That can be coherent, but only if you lock down one thing:

    WorkingState can change any time.

    CurrentState only changes at weave boundaries (i.e., it’s system-written, not author-edited).

If CurrentState can also drift between weaves, it becomes a second WorkingState and you lose the semantic reason to have both.
What should a HistoricalState represent: “previous” or “current at weave time”?

It should represent the content being released at weave time (i.e., the post-weave “current”). That’s the only semantics that stays consistent with:

    “git-like” snapshotting,

    reproducible release bundles,

    and your previousHistoricalState chain.

If you instead make HistoricalState “the state before the weave,” you create an off-by-one history that confuses everyone: the thing you just released wouldn’t have a historical identity until the next weave.

So: HistoricalState = snapshot of the release content at that weave.
A deterministic weave algorithm that matches your model

Assume every Flow always has:

    exactly one WorkingState

    exactly one CurrentState

    zero or more HistoricalState (if versioning enabled)

On weave (when createHistoricalStatesOnWeave = true)

Let W be WorkingState, C be CurrentState, H_latest be latest HistoricalState (if any).

    Capture the release content = the content of W at weave time.

    Mint a new HistoricalState H_new whose content = that release content.

    If H_latest exists, assert: H_new previousHistoricalState H_latest.

    Update convenience pointers (if you keep them):

        latestHistoricalState = H_new

    Update CurrentState content to match H_new (or treat CurrentState as “the published mutable handle” that now contains the released content).

    Reset WorkingState to match CurrentState (optional but strongly recommended as the default), then it can diverge again during editing.

This makes “CurrentState at weave time” and “H_new” identical in content, but different identities (one mutable handle, one immutable snapshot).
On weave (when createHistoricalStatesOnWeave = false)

No HistoricalState is minted.

    CurrentState content becomes the content of WorkingState at weave time.

    Working may optionally be reset to current.

In this mode, “as-of” history is inherently lossy, and you should say so explicitly.
Creation semantics (“first weave”)

Your instinct is right: if createHistoricalStatesOnWeave is enabled at creation, do an initial crystallization immediately.
Recommended creation rule

On artifact creation:

    Create WorkingState and CurrentState.

    Populate them with initial content (empty is allowed, but you should treat “empty initial artifact” as a real state, not a special case).

    If versioning is enabled:

        mint HistoricalState v1 identical to CurrentState

        set latestHistoricalState = v1

So yes: at least three states exist, and initially two or three are content-identical. That’s fine; the important bit is that only HistoricalState is the citeable immutable identity.
“Fallback to current” vs “fallback to working”

If you adopt “CurrentState always exists,” then your default fallback for time-based resolution should be:

    Fall back to CurrentState, not WorkingState.

WorkingState is “editing workspace,” not “published view.” It should not be the default answer to “what is the artifact?” unless an application is explicitly asking for a working view.

So for “as-of time T” in the absence of historical states:

    return CurrentState (and note that historical accuracy is not guaranteed).

The one remaining ambiguity you should not leave open

You said:

    “WorkingState and CurrentState can change without getting a new identifier.”

That’s OK only if you enforce:

    No direct author edits to CurrentState (only weave writes CurrentState).

If you don’t enforce that, then:

    CurrentState and WorkingState are both mutable between weaves

    weave stops being a meaningful boundary

    “published vs working” collapses, and time-based reconstruction becomes mostly arbitrary

If you want to preserve the possibility of direct edits to current for some reason, then you need an explicit policy flag (“currentEditable”), and you should expect tooling and reasoning complexity.
Practical upshot for modularity

With this model, modularity is straightforward:

    Assemblies reference AbstractArtifact by default.

    “As-of T” resolution becomes:

        pick latest HistoricalState with weaveTime ≤ T if any

        else CurrentState

No CurrentStatePointer is necessary if CurrentState is a stable handle with a clear update rule.

If you want, I can draft the updated ontology term blocks (Turtle) for CurrentState and hasCurrentState (plus SHACL) consistent with this “CurrentState is mutable but only weave-updated” rule, including a shape that forbids edits to CurrentState outside weave (you can model that as “CurrentState must have weave timestamp equal to the Flow’s last weave” if you track events).
You said:
I don't get the "state handle" term. It's a full state, just mutable. 
ChatGPT said:

“State handle” was my attempt to name the semantic role of something whose IRI stays stable while the denoted content can change—i.e., like a named graph IRI. If you’re comfortable calling that a “full state,” that’s fine; the important part is to stop claiming that a State’s identity is determined by its content.

Right now your ontology text does claim that, implicitly:

    ArtifactState comment: “A single-state content identity within a Flow.”

    HistoricalState comment: “An immutable published snapshot state for a Flow.”

    WorkingState comment: “The mutable working state…”

Those three are not consistent unless you distinguish:

    Immutable state: content-identifying (HistoricalState)

    Mutable state: channel/view-identifying (WorkingState, CurrentState)

So the fix is mostly definitional.
What changes when “CurrentState is a full state, just mutable”

    A CurrentState IRI does not identify a specific snapshot.
    It identifies “whatever content is current right now (or at weave boundaries, if you constrain it).”

    Citations and reproducibility require HistoricalState.
    Your earlier rule “use AbstractArtifact unless you want a specific HistoricalState” still holds, but you should add: “use HistoricalState when you need a stable snapshot.”

    You cannot enforce mutability/immutability in OWL; it’s a process constraint.
    You can express it in comments and partially in SHACL (e.g., “current changes only at weave events” if you model weave events).

A clean terminology tweak (no “handle”)

If you dislike “handle,” introduce an explicit superclass:

    sflo:MutableState ⊑ sflo:ArtifactState

    sflo:ImmutableState ⊑ sflo:ArtifactState (or skip and just use HistoricalState)

Then:

    WorkingState ⊑ MutableState

    CurrentState ⊑ MutableState

    HistoricalState ⊑ ArtifactState (and treated as immutable by convention)

This lets you keep calling them “full states” while being honest about what the IRI means.
Minimal comment rewrites to make your model coherent

    ArtifactState
    “A single-state content view within a Flow. Some ArtifactStates are mutable (e.g., WorkingState, CurrentState) and may change their content over time without minting a new identifier; other ArtifactStates are immutable (e.g., HistoricalState) and represent published snapshots.”

    WorkingState
    “A mutable editing state whose content may change arbitrarily between weaves.”

    CurrentState (new)
    “A mutable published state whose content changes only at weave boundaries (by convention), representing the most recently woven content.”

    HistoricalState
    “An immutable published snapshot state for a Flow.”

That eliminates the need for “state handle” while preserving the conceptual distinction I was trying to protect.
The one constraint you should lock in

If you want CurrentState to mean anything, it must be governed differently from WorkingState:

    Working: changes between weaves

    Current: changes only on weave (or only via the weave process)

Otherwise Current and Working are just two mutable states with no semantic contract.

If you want, I can draft the SHACL shapes for:

    Flow has exactly one WorkingState and exactly one CurrentState

    WorkingState and CurrentState are distinct nodes

    If you model weave events/timestamps, CurrentState updates must correspond to a weave event (optional but doable)

You said:
I'm not sure what "content-identifying" or "a State’s identity is determined by its content". A state doesn't hold any content itself, it's more of a shell that's a "same content" marker. 

Can you think of examples of where "MutableState" superclass is useful, other than documentation/conceptually?
ChatGPT said:

Your description contains an internal contradiction that’s worth fixing explicitly:

    You said a State is a shell that’s a “same content” marker.

    You also want WorkingState / CurrentState to be mutable.

If a State is truly a “same content” marker, then mutating its content means it is no longer the same content, so the marker would be lying. The only way out is to admit that:

    HistoricalState is a “same content” marker (content-fixed).

    WorkingState / CurrentState are not “same content” markers; they are same role markers (content can vary).

You don’t have to say “the state holds content.” You can define “content” operationally as whatever your system treats as the state’s realized payload (e.g., the set of AbstractFiles / LocatedFiles that are considered the realization of that State at a moment).

So when I said “content-identifying,” I meant: the State’s identity corresponds to a fixed payload/realization set. That’s true for HistoricalState, false for mutable states.
When a MutableState superclass is useful beyond documentation

It’s only “extra taxonomy” if it never drives constraints, tooling behavior, or APIs. Here are concrete places it becomes mechanically useful.
1) Clean SHACL ranges and invariants without repeating yourself

You can write once:

    hasWorkingState range MutableState

    hasCurrentState range MutableState

    hasHistoricalState range ImmutableState (or just HistoricalState)

Then any future mutable state type (e.g., PreviewState, StagingState) automatically fits the same constraints.

Without MutableState, you end up with either:

    duplicated shapes for each mutable subtype, or

    looser constraints that allow historical states where you don’t want them.

2) Enforce “no edit metadata on immutable states” generically

If you add operational metadata like any of these (very likely):

    lockedBy, lastEditedAt, hasUncommittedChanges, editorSession, diffFromCurrent, etc.

You can enforce:

    only MutableState may have them

    HistoricalState must not have them

That’s not just conceptual; it prevents accidental pollution of immutable history.
3) Query and UI behavior without hard-coding specific subclasses

A generic client can do:

    “show editable states” = instances of MutableState

    “warn against citation” = instances of MutableState

    “safe to cache forever” = instances of not MutableState

This avoids clients needing to know every future concrete class name (WorkingState, CurrentState, PreviewState, …).
4) Cache and invalidation rules become type-driven

If you treat immutable things as cacheable forever, you can make build + CDN behavior deterministic:

    changes to MutableState invalidate derived artifacts

    HistoricalState never changes, so derived artifacts can be content-addressed / long-lived

That’s an implementation win, not just semantics.
5) Write-access control becomes declarative

Even in a file-based system, you’ll have rules like:

    “writes are only allowed to working/current”

    “historical is read-only”

Typing states as MutableState lets you express that as a simple rule instead of embedding it in procedural code paths.
6) It future-proofs “current” variants without ontology surgery

Today you have Working and Current. Tomorrow you’ll be tempted by:

    ReviewState

    StagingState

    PreviewState

    PinnedCurrentState (current-but-frozen for a release train)

MutableState prevents you from encoding “editability semantics” into a growing pile of special-case properties and shapes.
If you don’t add MutableState, you should still fix the definition

Even if you decide against the superclass, you should rewrite the comments so they don’t claim “same content marker” for mutable states.

A crisp way to say it:

    HistoricalState: “a content-fixed snapshot identity”

    WorkingState / CurrentState: “a role-fixed state whose realized payload may change over time”

That aligns with your “shell” model: the shell is stable, the shell’s realization changes.

If you want, I can propose exact revised comments for ArtifactState, WorkingState, CurrentState, and HistoricalState that are consistent with “states don’t contain content; they denote a realized payload that may vary (mutable) or not (historical).”
You said:
Sorry, by "same content marker", I mean "all the AbstractFile realizations of the state have essentially the same content". That's true of WorkingStates (which always have only a single file BTW for authoring consistency) and CurrentStates (which may have multiple Abstract-LocatedFile pairs). 

Why wouldn't we do:

hasWorkingState range WorkingState

hasCurrentState range CurrentState


Here's the releveant current ontology entries. Feel free to suggest changes, but don't add MutableState for now.

<AbstractArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract artifact" ;
  rdfs:comment "The primary, whole-artifact identity. In a Semantic Mesh, each AbstractArtifact has exactly one Flow that supplies its working and historical states." .

<ArtifactFlow> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Flow" ;
  rdfs:comment "The over-time identity for a DigitalArtifact. An ArtifactFlow relates one CurrentArtifactState, one WorkingArtifactState and zero or more HistoricalArtifactStates." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A single-state content identity within a DigitalArtifact. ArtifactStates may be mutable (working) or immutable (historical)." .

<WorkingState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Working state" ;
  rdfs:comment "The always-mutable working state for a DigitalArtifact. By convention in meshes, this is surfaced at a reserved '_working' path segment." .

<HistoricalState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable snapshot state for a DigitalArtifact." .

<CurrentState> a rdfs:Class ;
  rdfs:label "Current state" ;
  rdfs:comment "The mutable-at-weave current state for a DigitalArtifact. By convention in meshes, this is surfaced at a reserved '_current' path segment." .

<AbstractFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-level identity for a particular State: a specific file variant (e.g., TTL vs JSON-LD, zipped vs unzipped, or distinct build/canonicalization choices) whose bytes may be served from one or more LocatedFiles. Multiple AbstractFiles may exist for the same State, including multiple variants within the same media type or extension. In mesh file layouts that reserve direct State children for AbstractFiles, AbstractFile identifiers MUST NOT start with '_' and SHOULD NOT be 'index.html'." .


ChatGPT said:

Yes, you can (and probably should) make the ranges specific:

    hasWorkingState rdfs:range WorkingState

    hasCurrentState rdfs:range CurrentState

That buys you tighter validation and avoids accidental misuse (e.g., someone hanging a HistoricalState off hasWorkingState). The main reason not to do it would be extensibility (“maybe I’ll have other kinds of working-like states later”), but you can handle that cleanly without introducing MutableState by adding new subproperties later (e.g., hasDraftState) or by keeping a generic superproperty for querying.
Minimal property structure I’d recommend (no new state superclasses)

    Keep a generic hasState (or hasArtifactState) for “any state”

    Make hasWorkingState, hasCurrentState, hasHistoricalState subproperties with narrow ranges

That lets clients query “all states” without hardcoding every specialized property, while still keeping the strong ranges.
Specific issues in your current class blocks (suggested fixes)
1) CurrentState is missing rdfs:subClassOf <ArtifactState>

Right now it’s not a state at all in the model (it has no superclass), which will break SHACL/OWL expectations and your own narrative.
2) The ArtifactFlow comment uses names you don’t define

It says “CurrentArtifactState / WorkingArtifactState / HistoricalArtifactStates” but your classes are CurrentState, WorkingState, HistoricalState. Either rename those in the comment or rename the classes; don’t leave both.
3) ArtifactState comment still implies “content identity” in a way that will confuse readers (and future-you)

Given your clarified meaning (“all AbstractFiles under a state are intended to be equivalent in information content”), ArtifactState is better described as a content-equivalence bucket over representations, not “the content itself.”
Concrete revised Turtle (only edits, no new classes)

<ArtifactFlow> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Flow" ;
  rdfs:comment "The over-time identity for an AbstractArtifact. An ArtifactFlow relates exactly one WorkingState, exactly one CurrentState, and zero or more HistoricalStates." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A state groups one or more AbstractFile representations that are intended to be equivalent in information content for a given artifact at a point in its workflow. States may be mutable (WorkingState, CurrentState) or immutable (HistoricalState)." .

<WorkingState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Working state" ;
  rdfs:comment "The always-mutable authoring state for an artifact. WorkingState content may change between weaves. Mesh convention: surfaced at a reserved '_working' path segment. Mesh authoring convention: WorkingState SHOULD have exactly one AbstractFile." .

<CurrentState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Current state" ;
  rdfs:comment "The mutable-at-weave current state for an artifact. CurrentState content changes only at weave boundaries (by convention) and represents the most recently woven content. Mesh convention: surfaced at a reserved '_current' path segment." .

<HistoricalState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published snapshot state for an artifact, created at weave time when versioning is enabled." .

Optional: small precision tweak to AbstractArtifact comment

Your AbstractArtifact comment currently says “supplies its working and historical states,” but you now also have CurrentState. Update it to avoid drift.
Why specific ranges are fine (and what to do if you want flexibility later)

You asked: “Why wouldn’t we do range WorkingState / CurrentState?”
Answer: you should, unless you foresee multiple “current-like” or “working-like” state types.

If that happens, don’t loosen the range; instead do one of these:

    Add another property: hasDraftState rdfs:range DraftState

    Or make hasWorkingState a subproperty of hasState and keep hasState rdfs:range ArtifactState

This keeps the main model crisp and extensible without introducing MutableState.
One SHACL constraint you implicitly want (based on your “single file for WorkingState” rule)

You didn’t ask for it, but this is exactly the kind of rule that benefits from SHACL:

    WorkingState must have hasAbstractFile exactly 1

If you want, I can draft that shape in your JSON-LD context in the same style as the earlier constraint.
You said:
I guess I don't need to repeat NarrowFacet for the subProperty right?

<hasState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <ArtifactState> ;
  rdfs:label "has state" ;
  rdfs:comment "Relates a Flow to one of its States." .

<hasCurrentState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet>, <hasState> ;
  rdf:type owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <CurrentState> ;
  rdfs:label "has current state" ;
  rdfs:comment "Relates a Flow to its current (mutable-at-weave) State. Mesh convention: exactly one current state per Flow." .
  
---

I like ""The over-time identity for an DigitalArtifact..."" better than AbstractArtifact. Right?
---

It's like I'm re-inventing git, but way worse. My ontology refinements seem never-ending. Should I just give up on this project?

ChatGPT said:
1) Do you need to repeat narrowerFacet on hasCurrentState?

No, not for semantics.

If you have:

    hasCurrentState rdfs:subPropertyOf hasState .

    hasState rdfs:subPropertyOf narrowerFacet .

then a reasoner can already infer:

    hasCurrentState rdfs:subPropertyOf narrowerFacet .

So this is redundant:

rdfs:subPropertyOf <narrowerFacet>, <hasState> ;

That said, there are two legitimate reasons to keep the redundancy:

    Defensive modeling: if you later decide hasState should not be a facet relation, you won’t accidentally lose the facet connection.

    Readability: some humans (and some tools) read it more easily when it’s explicit.

If you want the cleanest structure, do this:

<hasCurrentState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <CurrentState> ;
  ...

and keep narrowerFacet only on the generic property.
2) “The over-time identity for a DigitalArtifact” vs “for an AbstractArtifact”

Your axioms currently say Flow is for AbstractArtifact, not arbitrary DigitalArtifact:

<hasFlow> rdfs:domain <AbstractArtifact> ; rdfs:range <ArtifactFlow> .

So if you prefer the broader phrasing (“DigitalArtifact”), you should either:

    keep the comment narrow (recommended for consistency), or

    widen hasFlow (domain DigitalArtifact) and accept the conceptual expansion.

Right now, “DigitalArtifact” in the comment is slightly misleading because the model doesn’t allow a Flow for a plain LocatedFile unless you treat it as/cast it into AbstractArtifact.

A good compromise comment that stays truthful without locking you in:

    “The over-time identity for an artifact modeled with a Flow (typically an AbstractArtifact).”

3) “Reinventing git, but worse” — and whether to give up

If you feel like you’re reinventing git, it’s because you are building a versioned artifact model. The difference is you’re also trying to make it:

    dereferenceable on the web,

    multi-representation (AbstractFile/LocatedFile selection),

    semantically queryable,

    and composable across artifacts/meshes.

That’s why it feels “worse”: it’s strictly more requirements than git’s object model.

The bigger issue I see is not that the ontology is doomed; it’s that you’re iterating without a “stop condition.” Ontology refinement becomes endless when you don’t freeze against concrete competency tests.

A pragmatic way to stop the churn:

    Write 5–10 competency questions you must support (e.g., “resolve artifact as-of weave time T”, “list representations available for CurrentState”, “compute assembly union as-of T”, etc.).

    Create 2–3 tiny example meshes and model them end-to-end in RDF.

    Freeze the ontology at v0.1 once those examples work. Add future ideas only if they unblock a competency question.

That’s the point where you start getting leverage from implementation feedback instead of conceptual self-critique.

So: no, “give up” is not the rational move. The rational move is to constrain scope and declare victory earlier. Your model is already converging; the remaining work is mostly deciding which invariants are normative vs “by convention,” and testing them on examples.
You said:
Yes, please draft the WorkingState SHACL constraint.
ChatGPT said:

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
      "@id": "WorkingStateSingleAbstractFileShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "WorkingState single AbstractFile shape",
      "rdfs:comment": "Enforces the mesh authoring convention that a sflo:WorkingState has exactly one sflo:AbstractFile representation for consistency of editing workflows.",
      "sh:targetClass": "sflo:WorkingState",
      "sh:property": [
        {
          "sh:path": "sflo:hasAbstractFile",
          "sh:class": "sflo:AbstractFile",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    }
  ]
}

You said:
“The over-time identity for a DigitalArtifact” vs “for an AbstractArtifact” -- The AbstractArtifact is a facet or view of the DigitalArtifact, same with Flow. Now I can't even remember the point of the AbstractArtifact. Doesn't the Flow hand better of the DigitalArtifact itself?
ChatGPT said:

If you can’t articulate what AbstractArtifact adds over ArtifactFlow, that’s a sign the model has drifted into redundancy. But you’re not wrong that a Flow can be the primary identity. The question is whether you want your “primary IRI people reference” to be:

    the thing (stable subject), or

    the thing’s revision stream (diachronic object).

Right now you keep saying “use the AbstractArtifact IRI for references,” which is exactly the strongest argument for keeping AbstractArtifact. If you drop it, you need a new answer to “what do I reference when I don’t want to pin a state?”
What AbstractArtifact is for (non-hand-wavy, actually useful)

ArtifactFlow is a history object. It exists to relate states and encode revision lineage.

AbstractArtifact is a subject anchor that stays meaningful even if your versioning mechanics change. It’s the “whole artifact” node that you hang facts on that are not inherently about the revision stream, e.g.:

    canonical identity / what humans mean by “this artifact”

    membership in Knops (“this is the payload artifact”)

    reference links, ownership, licensing intent, “what it’s about”

    stable inbound links from other artifacts (“depends on X”) without saying “depends on X’s revision stream”

The facet lattice you’ve already built (narrowerFacet) is implicitly saying:
AbstractArtifact is the broader facet; Flow is a narrower facet.

That’s coherent.
What ArtifactFlow does better than AbstractArtifact

If you want to talk about:

    revision order, predecessor/successor

    “latest historical”, “as-of time T”

    weave events, provenance of changes

…then ArtifactFlow is the right object. It’s “the versioned life” of the artifact.

So Flow doesn’t “handle better the DigitalArtifact itself.” It handles better the artifact’s evolution.
When you should drop AbstractArtifact (and just use Flow)

Drop AbstractArtifact if all of these are true:

    Every artifact you care about is always flow-managed.

    You’re comfortable with the “main IRI” being “the history object.”

    You don’t need a stable subject that could outlive/repoint to a different flow model later.

    You don’t expect to want multiple flows per artifact (even conceptually), ever.

If you meet those constraints, you can simplify: DigitalArtifact (or a renamed “Artifact”) has exactly one ArtifactFlow, and you reference the Flow as the canonical identifier.

But note the cost: your rule “Always use the AbstractArtifact IRI for references” becomes “Always use the Flow IRI,” which subtly changes what you’re saying (you’re now referencing a revision stream).
A sharper way to reduce the confusion without removing AbstractArtifact

Your current confusion is partly caused by comments and naming:

    DigitalArtifact is trying to mean “anything file-like” and “the whole artifact identity.” That’s too broad.

    AbstractArtifact sounds like “abstract file” and gets mentally merged with AbstractFile.

Two options that fix this without changing structure:
Option A: Rename AbstractArtifact

Rename it to something that reads as “the root subject”:

    Artifact (simple, but might collide with “DigitalArtifact”)

    ArtifactRoot

    ArtifactIdentity

    WholeArtifact

Then keep DigitalArtifact as the umbrella class for everything in the stack (states, flows, files).
Option B: Keep name but fix comments so it’s impossible to misread

Make it explicit that AbstractArtifact is the reference target and Flow is the revision stream.

Example comment edits:

    AbstractArtifact: “The stable subject identity for an artifact; use this IRI for general references and metadata. Its Flow captures revision history.”

    ArtifactFlow: “The revision stream facet for an artifact, relating Current/Working/Historical states.”

Recommendation (based on what you’ve said you want)

Keep AbstractArtifact because you keep re-deriving the requirement for an unpinned reference target (modularity, assemblies, “use AbstractArtifact unless pinning”). That’s exactly what it’s for.

What you should change is not the existence of AbstractArtifact, but the story:

    DigitalArtifact = umbrella type

    AbstractArtifact = canonical subject to reference

    ArtifactFlow = diachronic facet (revision stream)

    states/files/located files = progressively narrower facets

If you want, paste your current hasFlow / hasState property block, and I’ll suggest the minimal comment/label tweaks that make the “why does AbstractArtifact exist?” question stop coming up.
You said:
Oh, yeah, we need assembly property. Ideally just one. I think we decided a primary payload for assembly Knops. Or just put of assembly until later?
ChatGPT said:

You don’t need a special “assembly Knop” class if you keep one invariant: a Knop’s “primary payload” can itself be an assembly artifact (i.e., an artifact whose state describes how to assemble other artifacts). That keeps the Knop model stable and pushes “assembly semantics” into the artifact stack where it belongs.
Whether to put assembly off until later

You can safely defer most assembly semantics (resolution rules, as-of behavior, closure capture) as long as you reserve one predicate now so you don’t churn IRIs later. The predicate can start with a minimal, weak contract (“declares membership; interpretation is resolver-defined”) and you tighten it when you implement.

If you defer even the predicate, you’ll end up inventing it under deadline pressure later and likely regret the naming.
One property that works without forcing you into a whole assembly subsystem

Define a single predicate on the assembly’s state (not the Knop):

    Domain: ArtifactState (specifically the assembly artifact’s Working/Current/Historical)

    Range: keep it broad in OWL (to avoid incorrect inferences) and constrain in SHACL

Suggested term (pick one name; I’ll use hasAssemblyMember here):

<hasAssemblyMember> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "has assembly member" ;
  rdfs:comment "Declares that this State's content is an assembly composed from the content of one or more member artifacts or states. Members MAY be referenced as AbstractArtifacts (underspecified: resolved by application/policy) or as specific States (pinned, e.g., HistoricalState). Composition semantics are defined by the assembly profile in use (default: content union)." .

Why rdfs:range rdfs:Resource?

    Because if you try to make it a tight OWL range (AbstractArtifact), you lose pinning.

    If you make it a union range (AbstractArtifact OR ArtifactState), OWL 2 DL gets awkward in Turtle without blank nodes, and you’ll still want SHACL anyway.

The corresponding SHACL constraint (simple and enforceable)

Constrain values to what you actually want today:

    AbstractArtifact (floating)

    HistoricalState (pinned)

    (Optionally) CurrentState if you decide it’s a valid member reference

This lets you keep “one property” without silently allowing AbstractFile/LocatedFile values.
Where to put membership triples

Put hasAssemblyMember triples in the assembly payload artifact’s state content (not InventoryArtifacts).

    InventoryArtifacts should index and summarize.

    The payload state is where the semantics live.

So: a Knop that “is an assembly” is just:

    a normal Knop

    whose primary PayloadArtifact is an “assembly manifest” artifact (no special Knop class required).

Minimal “stop here for now” semantics you can ship

If you want to avoid re-inventing git further right now, freeze these minimal rules:

    Assembly membership uses only hasAssemblyMember.

    Default composition = union of member contents (no named-graph semantics).

    If member is a HistoricalState, it’s pinned.

    If member is an AbstractArtifact, it’s floating (resolver chooses which state).

That’s enough to support ontology modularity without deciding “as-of reconstruction” or “closure snapshots” yet.
Hyper-critical note (the real trap)

Trying to make assembly “perfect” (time travel, reproducible closure, cross-mesh determinism) before you’ve built one end-to-end example will keep you spinning. Reserve the predicate + value-shape now, then force yourself to model one modular ontology and write the resolver. Your next round of ontology changes should be driven by concrete failures in that resolver, not hypothetical completeness.

If you want, I can draft the SHACL sh:or value constraint for hasAssemblyMember in your JSON-LD context (like your other shapes), targeting sflo:ArtifactState nodes that have that property.
You said:
Sound advice. Should we at least make the the property into a relator? our other relator allows and gentle inward inverse property that kinda lets you declare "the most salient part of the relator, usually the target" directly but still participate in the relator relationship via its inverse.
ChatGPT said:

Yes—if you expect to ever need per-member metadata (and you almost certainly will), making assembly membership a relator is the right move. Otherwise you’ll end up inventing a second structure later (order, optional vs required, pinning override, “header-only”, conflict policy, notes, provenance, etc.).

The trick is to do it without losing your “ideally just one property” ergonomics:

    Keep one simple authoring predicate (sflo:hasAssemblyMember) for the common case.

    Add an optional relator form (sflo:AssemblyMembership) that can carry metadata.

    Ensure both forms converge via a derived shortcut so queries don’t fragment.

Minimal relator pattern with “gentle inverse” and a single-query surface
1) The simple predicate (keep this)

Authors can write only this:

sflo:hasAssemblyMember a owl:ObjectProperty ;
  rdfs:domain sflo:ArtifactState ;
  rdfs:range rdfs:Resource ;
  rdfs:label "has assembly member" ;
  rdfs:comment "Declares a member of this assembly state. Members are typically sflo:AbstractArtifact or a specific sflo:ArtifactState (e.g., sflo:HistoricalState) for pinning." .

sflo:assemblyMemberOf a owl:ObjectProperty ;
  owl:inverseOf sflo:hasAssemblyMember ;
  rdfs:label "assembly member of" .

That inverse is the “gentle inward” convenience: you can assert membership from the member side when it reads better.
2) The relator (only when you need metadata)

sflo:AssemblyMembership a rdfs:Class ;
  rdfs:label "Assembly membership" ;
  rdfs:comment "Relator describing the inclusion of a member within an assembly state, optionally carrying metadata (order, role, resolution hints, etc.)." .

sflo:hasAssemblyMembership a owl:ObjectProperty ;
  rdfs:domain sflo:ArtifactState ;
  rdfs:range sflo:AssemblyMembership ;
  owl:inverseOf sflo:assemblyMembershipFor ;
  rdfs:label "has assembly membership" .

sflo:assemblyMembershipFor a owl:ObjectProperty ;
  rdfs:domain sflo:AssemblyMembership ;
  rdfs:range sflo:ArtifactState ;
  owl:inverseOf sflo:hasAssemblyMembership ;
  rdfs:label "assembly membership for" .

sflo:assemblyMember a owl:ObjectProperty ;
  rdfs:domain sflo:AssemblyMembership ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf sflo:assemblyMemberOfMembership ;
  rdfs:label "assembly member" .

sflo:assemblyMemberOfMembership a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range sflo:AssemblyMembership ;
  owl:inverseOf sflo:assemblyMember ;
  rdfs:label "assembly member of membership" .

Then you can hang metadata on the relator later:

    sflo:memberOrder (xsd:integer)

    sflo:memberRole (controlled vocab)

    sflo:resolutionMode (pinned/floating/current/as-of)

    sflo:includeAnnotationsOnly etc.

3) Make the relator imply the simple predicate (so query surface stays one property)

Use an OWL property chain so that any relator membership entails the simple hasAssemblyMember triple:

sflo:hasAssemblyMember owl:propertyChainAxiom
  ( sflo:hasAssemblyMembership sflo:assemblyMember ) .

Meaning: if

    S sflo:hasAssemblyMembership M and

    M sflo:assemblyMember X

then infer:

    S sflo:hasAssemblyMember X

That keeps “just query hasAssemblyMember” true even if the data is authored via relators.
Why this is the sweet spot

    You get your single simple predicate for 90% of use.

    You get a relator when you need any metadata at all (and you will).

    You don’t fork your ecosystem into two incompatible assembly representations, because the relator projects to the simple predicate.

One caution

Don’t try to make the relator the only way while also insisting on “one property.” You’ll either force everyone to write blank-node relators (pain) or you’ll later add a shortcut anyway. The hybrid pattern above avoids that churn.

If you want, I can provide the JSON-LD SHACL shapes for:

    sflo:ArtifactState values of sflo:hasAssemblyMember are only (sflo:AbstractArtifact OR sflo:ArtifactState)

    sflo:AssemblyMembership must have exactly one sflo:assemblyMember and exactly one sflo:assemblyMembershipFor

    If sflo:hasAssemblyMembership is used, require that sflo:hasAssemblyMember is either absent (derived) or consistent (no mismatches).

You said:
OK, I'm going to ask you to make 1) A summary of decisions we've made in this session (and previous ones if you want); 2) any obvious questions to research further. 3) A summary of the concept model via the core ontology, perhaps the config ontology, and the shacl. 4) updated ontology+shacl that reflects the latest decisions and conceptual model; 5) A plan for how to get this machine rolling.

Which order do you prefer?
ChatGPT said:

Preferred order: 1 → 3 → 4 → 5 → 2. It forces you to (a) freeze what changed, (b) state the conceptual model cleanly, (c) reflect it in artifacts, (d) move to execution, and only then (e) research anything you still genuinely need.

Below is the full content in that order.
1) Summary of decisions made
Scope and packaging

    Keep a single “Semantic Flow Ontology” rather than splitting “general-purpose” sub-ontologies. Reuse will happen at the vocabulary level, not via imports.

Core artifact stack and facets

    Keep the artifact stack: AbstractArtifact → ArtifactFlow → States → AbstractFile → LocatedFile.

    Treat these as facets/views of a broader “DigitalArtifact” universe, but keep AbstractArtifact as the stable “reference target” (the thing most links should point at), and ArtifactFlow as the revision-stream facet.

States (biggest refinement this session)

    A Flow relates exactly one WorkingState, exactly one CurrentState, and zero or more HistoricalStates.

    WorkingState is always mutable (changes between weaves). Authoring convention: WorkingState has exactly one AbstractFile.

    CurrentState is “mutable-at-weave”: conceptually mutable, but by convention only changes at weave boundaries (system-updated). It may have multiple AbstractFiles.

    HistoricalState is immutable snapshot created at weave time when versioning is enabled.

    On creation, if createHistoricalStatesOnWeave is enabled, you do an initial “first weave” so you have an initial HistoricalState identical to CurrentState.

    If versioning is disabled, “as-of time” accuracy is not guaranteed (because you lack HistoricalStates). In that case, fall back to CurrentState, not WorkingState.

Properties and redundancy

    Introduce hasState as the generic state relation (subPropertyOf narrowerFacet), then hasWorkingState, hasCurrentState, hasHistoricalState as subproperties.

    Do not repeat narrowerFacet on subproperties if they already subproperty hasState (redundant). Keep redundancy only if you want “defensive modeling.”

Mesh constituents

    You flagged: “Semantic Flow resources may exist inside or outside a mesh” is wrong. The class comment should reflect that these are primary constituents of a Mesh (mesh-managed objects).

Modularity / assembly

    Don’t invent a special “assembly Knop” class. Use a normal Knop whose primary payload artifact is an assembly manifest.

    You want “ideally one property” for membership, but also want per-member metadata later.

    Decision: hybrid pattern:

        Keep simple hasAssemblyMember / assemblyMemberOf for ergonomics.

        Add optional AssemblyMembership relator for metadata, with a property chain so relator usage still entails hasAssemblyMember.

    Defer deeper semantics (resolution snapshots, policy capture) until implementation pressure forces it; reserve the predicates now.

SHACL added

    Add WorkingState authoring constraint: WorkingState must have exactly one hasAbstractFile.

3) Concept model summary (core ontology + config + SHACL)
Core ontology (conceptual spine)

DigitalArtifact
Umbrella “artifact universe” containing: AbstractArtifact, ArtifactFlow, ArtifactState, AbstractFile, LocatedFile, and other facets. narrowerFacet relates broader → narrower facets.

AbstractArtifact
Stable “whole artifact” reference target. Normal references and memberships (Knop payload, assembly membership) should usually point here unless pinning a specific state.

ArtifactFlow
Diachronic/revision-stream facet. Holds the state relationships (working/current/historical) and revision links (previous historical, latest historical convenience).

States

    WorkingState: mutable authoring; exactly one AbstractFile (authoring consistency).

    CurrentState: mutable-at-weave; represents last woven content (by convention).

    HistoricalState: immutable snapshots (release history).

Files

    AbstractFile: representation identity for a State (variant/packaging/serialization)

    LocatedFile: retrievable bytes identity; can serve an AbstractFile

Assembly

    Assembly is expressed at the State level (assembly manifest is “content for a state” that declares members).

    Members can be referenced as AbstractArtifact (floating) or specific ArtifactState (pinned; typically HistoricalState).

Config ontology (minimal, implied)

You have at least one control knob that must exist somewhere (config ontology or operational metadata):

    createHistoricalStatesOnWeave (boolean)
    Optionally later:

    any default “as-of” resolution rule, template selection, output generation flags.

SHACL role

    Enforce authoring invariants that OWL cannot (e.g., “WorkingState has exactly one AbstractFile”).

    Validate assembly manifest constraints later (member type restrictions, required ontology header only in assembly, etc.).

4) Updated ontology + SHACL reflecting latest decisions
4A) Core ontology patch (Turtle)

This is not your full ontology—this is the updated “artifact core + states + assembly” slice that matches the latest decisions.

@base <https://semantic-flow.github.io/semantic-flow-ontology/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix prov: <http://www.w3.org/ns/prov#> .

#################################################################
# Artifact facet lattice (unchanged conceptually)
#################################################################

<narrowerFacet> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "narrower facet" ;
  rdfs:comment "Relates a broader DigitalArtifact to a narrower DigitalArtifact facet within the same artifact structure." .

<broaderFacet> a owl:ObjectProperty ;
  owl:inverseOf <narrowerFacet> ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "broader facet" ;
  rdfs:comment "Inverse of narrowerFacet." .

#################################################################
# Core artifact model
#################################################################

<AbstractArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract artifact" ;
  rdfs:comment "The primary, whole-artifact identity used as the default reference target. In a mesh, each AbstractArtifact has exactly one Flow that relates its WorkingState, CurrentState, and HistoricalStates." .

<ArtifactFlow> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Flow" ;
  rdfs:comment "The over-time identity (revision-stream facet) for an AbstractArtifact. A Flow relates exactly one WorkingState, exactly one CurrentState, and zero or more HistoricalStates." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A state groups one or more AbstractFile representations intended to be equivalent in information content for an artifact at a point in its workflow. States may be mutable (WorkingState, CurrentState) or immutable (HistoricalState)." .

<WorkingState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Working state" ;
  rdfs:comment "The always-mutable authoring state for an artifact. WorkingState content may change between weaves. Mesh convention: surfaced at a reserved '_working' path segment. Mesh authoring convention: WorkingState SHOULD have exactly one AbstractFile." .

<CurrentState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Current state" ;
  rdfs:comment "The mutable-at-weave current state for an artifact. By convention, CurrentState content changes only at weave boundaries and represents the most recently woven content. Mesh convention: surfaced at a reserved '_current' path segment." .

<HistoricalState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable published snapshot state for an artifact, created at weave time when versioning is enabled." .

<AbstractFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-level representation identity for a particular State. Multiple AbstractFiles may exist for the same State (variants/packaging/serialization). AbstractFiles are served by one or more LocatedFiles." .

<LocatedFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes identity (typically with an extension) that provides concrete bytes for an AbstractFile." .

#################################################################
# Core structural relations
#################################################################

<hasFlow> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <ArtifactFlow> ;
  rdfs:label "has flow" ;
  rdfs:comment "Relates an AbstractArtifact to its Flow. Mesh convention: exactly one Flow per AbstractArtifact." .

<hasState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <ArtifactState> ;
  rdfs:label "has state" ;
  rdfs:comment "Relates a Flow to one of its States." .

<hasWorkingState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <WorkingState> ;
  rdfs:label "has working state" ;
  rdfs:comment "Relates a Flow to its WorkingState. Mesh convention: exactly one WorkingState per Flow." .

<hasCurrentState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <CurrentState> ;
  rdfs:label "has current state" ;
  rdfs:comment "Relates a Flow to its CurrentState. Mesh convention: exactly one CurrentState per Flow." .

<hasHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "has historical state" ;
  rdfs:comment "Relates a Flow to a published HistoricalState snapshot." .

<previousHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf prov:wasRevisionOf ;
  rdfs:domain <HistoricalState> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "previous historical state" ;
  rdfs:comment "Links a HistoricalState to the immediately preceding HistoricalState in its published revision lineage." .

<latestHistoricalState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <hasHistoricalState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "latest historical state" ;
  rdfs:comment "Optional convenience pointer from a Flow to its latest (most recently published) HistoricalState." .

<hasAbstractFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <AbstractFile> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Relates a State to one of its AbstractFile representations." .

<hasLocatedFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractFile> ;
  rdfs:range <LocatedFile> ;
  rdfs:label "has located file" ;
  rdfs:comment "Relates an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

#################################################################
# Mesh constituents: comment correction
#################################################################

<SemanticFlowResource> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A primary constituent of a Mesh, modeled using Semantic Flow conventions (e.g., Mesh, Nomen, Knop, and their hosted artifacts and derived resources)." .

#################################################################
# Assembly (hybrid: simple predicate + relator)
#################################################################

<hasAssemblyMember> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "has assembly member" ;
  rdfs:comment "Declares a member of this assembly State. Members are typically AbstractArtifacts (floating) or specific ArtifactStates (pinned; usually HistoricalState). Default composition semantics are defined by the consuming application (typical: content union)." .

<assemblyMemberOf> a owl:ObjectProperty ;
  owl:inverseOf <hasAssemblyMember> ;
  rdfs:label "assembly member of" ;
  rdfs:comment "Inverse of hasAssemblyMember." .

<AssemblyMembership> a rdfs:Class ;
  rdfs:label "Assembly membership" ;
  rdfs:comment "Relator describing the inclusion of a member within an assembly State, optionally carrying metadata (order, role, resolution hints, notes, etc.)." .

<hasAssemblyMembership> a owl:ObjectProperty ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <AssemblyMembership> ;
  owl:inverseOf <assemblyMembershipFor> ;
  rdfs:label "has assembly membership" .

<assemblyMembershipFor> a owl:ObjectProperty ;
  rdfs:domain <AssemblyMembership> ;
  rdfs:range <ArtifactState> ;
  owl:inverseOf <hasAssemblyMembership> ;
  rdfs:label "assembly membership for" .

<assemblyMember> a owl:ObjectProperty ;
  rdfs:domain <AssemblyMembership> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <assemblyMemberOfMembership> ;
  rdfs:label "assembly member" .

<assemblyMemberOfMembership> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <AssemblyMembership> ;
  owl:inverseOf <assemblyMember> ;
  rdfs:label "assembly member of membership" .

# Property chain: relator implies simple predicate
<hasAssemblyMember> owl:propertyChainAxiom ( <hasAssemblyMembership> <assemblyMember> ) .

Critical notes you should not ignore:

    If you keep saying “CurrentState changes only on weave,” you should eventually model weave events/timestamps or you’ll have no way to validate that claim.

    latestHistoricalState is still optional convenience. If you depend on it for “as-of” logic later, you’ll regret not defining the actual selection rule.

4B) Updated SHACL JSON-LD: WorkingState has exactly one AbstractFile

This is the WorkingState constraint you requested, in your context.

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
      "@id": "WorkingStateSingleAbstractFileShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "WorkingState single AbstractFile shape",
      "rdfs:comment": "Enforces the mesh authoring convention that a sflo:WorkingState has exactly one sflo:AbstractFile representation for consistency of editing workflows.",
      "sh:targetClass": "sflo:WorkingState",
      "sh:property": [
        {
          "sh:path": "sflo:hasAbstractFile",
          "sh:class": "sflo:AbstractFile",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    }
  ]
}

5) Plan to “get the machine rolling” (stop the ontology spiral)
Phase 0 — Freeze a minimal competence suite (1–2 hours)

Write 8–10 competency queries before more ontology edits. Examples:

    Given an AbstractArtifact, get its Flow, WorkingState, CurrentState, latest HistoricalState.

    Resolve an AbstractArtifact “as-of T” (rule: latest HistoricalState ≤ T else CurrentState).

    For a WorkingState, list its single AbstractFile and its LocatedFile(s).

    For an assembly state, list members (simple + relator forms) with one query surface (hasAssemblyMember).

If you can’t express a competency question, you don’t need an ontology term for it yet.
Phase 1 — Build one canonical toy mesh in RDF (same day)

Create a minimal dataset with:

    1 Knop payload artifact with Working/Current/Historical

    1 assembly artifact that includes the first by AbstractArtifact reference, plus one pinned HistoricalState member

    1 inventory artifact where you record weave timestamps (even if just literals)

This will force your model to prove itself.
Phase 2 — Implement resolver + weaver with hard constraints (MVP)

Implement:

    resolveArtifact(abstractArtifact, mode) where mode ∈ {working, current, latestHistorical, asOfTime}

    weave(flow):

        update CurrentState from WorkingState

        optionally mint HistoricalState (and link previous)

        update inventory timestamp(s)

Do not implement “perfect modularity” or “perfect provenance” yet. Implement the deterministic basics.
Phase 3 — Add assembly union as a derived artifact

    Consume hasAssemblyMember (and relator form via chain)

    Compute union output for a state

    Emit derived files (AbstractFile + LocatedFile) for the assembly CurrentState and HistoricalStates (if enabled)

Phase 4 — Only then: expand SHACL and config

Add SHACL that protects you from regressions:

    Flow has exactly one WorkingState and CurrentState

    WorkingState/CurrentState are distinct IRIs

    CurrentState has at least one AbstractFile (optional, but likely)

    Assembly membership values constrained to AbstractArtifact OR ArtifactState

2) Obvious questions to research further

(Do these only if they block implementation; otherwise defer.)

    Existing dataset modularity vocabularies beyond VoID/DCAT (and how real tools treat them). You may decide to align terms later, but don’t let alignment stall the MVP.

    OWL/SHACL/tooling behavior around owl:propertyChainAxiom in your intended reasoners/validators (some validators ignore OWL entailment).

    “As-of time” semantics in bitemporal/versioned content systems (you’re inventing a deterministic projection rule; validate it against real expectations).

    Blank node and union semantics across modules (whether you want to skolemize on weave or treat bnodes as strictly local).

    UI versioning strategy for ResourcePages:

        Immutable historical pages vs regenerable “latest UI”

        What gets rebuilt when templates change (and where it’s stored)

If you want to make the next step maximally concrete: provide the competency questions you care about most, and I’ll translate them into (a) required ontology terms, (b) SHACL constraints, and (c) resolver pseudo-code. That’s the fastest way to stop the churn.
You said:
We need  a weave property on Knops. (I think all Knop components weave simultaneously. But you don't have to generate a new version just because you're woven. If you haven't changed, you can drop out of the weave.)

I meant to paste my latest ontology in case there were any small changes.  Oh yeah, I added some functionalproperties:

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
  dcterms:creator <https://djradon.github.io/ns/djradon> ;
  owl:versionInfo "0.0.1" ;
  owl:versionIRI <https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl> ;
  vann:preferredNamespacePrefix "sflo" ;
  vann:preferredNamespaceUri <https://semantic-flow.github.io/semantic-flow-ontology/> .

#################################################################
#################################################################
# General-purpose vocabulary
#################################################################
#################################################################

#################################################################
# DigitalArtifact
#################################################################


<DigitalArtifact> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A file-like digital object (documents, specifications, datasets, pages, binary assets), whether abstract or location-specific. A DigitalArtifact may exist inside or outside any Semantic Flow mesh." .

<ArtifactContainer> a rdfs:Class ;
  rdfs:label "Artifact container" ;
  rdfs:comment "A resource that hosts/contains artifacts and other structural resources. Not itself required to be a DigitalArtifact." .

#################################################################
# Artifact facet lattice
#################################################################

<narrowerFacet> a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "narrower facet" ;
  rdfs:comment "Relates a broader DigitalArtifact to a narrower DigitalArtifact facet within the same artifact structure. Intended to be closed (tree-shaped) within a mesh-managed Flow, but may be used more loosely for external artifacts." .

<broaderFacet> a owl:ObjectProperty ;
  owl:inverseOf <narrowerFacet> ;
  rdfs:domain <DigitalArtifact> ;
  rdfs:range <DigitalArtifact> ;
  rdfs:label "broader facet" ;
  rdfs:comment "Inverse of narrowerFacet." .

#################################################################
# Core artifact model (AbstractArtifact / Flow / State / AbstractFile / LocatedFile)
#################################################################

<AbstractArtifact> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract artifact" ;
  rdfs:comment "The primary, whole-artifact identity. In a Semantic Mesh, each AbstractArtifact has exactly one Flow that supplies its working and historical states." .

<ArtifactFlow> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Flow" ;
  rdfs:comment "The over-time identity for an DigitalArtifact. An ArtifactFlow relates exactly one WorkingState, exactly one CurrentState, and zero or more HistoricalStates." .

<ArtifactState> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "State" ;
  rdfs:comment "A single-state content identity within a DigitalArtifact. ArtifactStates may be mutable (working) or immutable (historical)." .

<CurrentState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Current state" ;
  rdfs:comment "The mutable-at-weave current state for a DigitalArtifact. By convention in meshes, this is surfaced at a reserved '_current' path segment." .

<WorkingState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Working state" ;
  rdfs:comment "The always-mutable working state for a DigitalArtifact. By convention in meshes, this is surfaced at a reserved '_working' path segment." .

<HistoricalState> a rdfs:Class ;
  rdfs:subClassOf <ArtifactState> ;
  rdfs:label "Historical state" ;
  rdfs:comment "An immutable snapshot state for a DigitalArtifact." .

<AbstractFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Abstract file" ;
  rdfs:comment "A file-level identity for a particular State: a specific file variant (e.g., TTL vs JSON-LD, zipped vs unzipped, or distinct build/canonicalization choices) whose bytes may be served from one or more LocatedFiles. Multiple AbstractFiles may exist for the same State, including multiple variants within the same media type or extension. In mesh file layouts that reserve direct State children for AbstractFiles, AbstractFile identifiers MUST NOT start with '_' and SHOULD NOT be 'index.html'." .

<LocatedFile> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "Located file" ;
  rdfs:comment "A retrievable bytes identity (typically with an extension). A LocatedFile may be used as a facet of broader artifacts (AbstractFile, State, Flow, AbstractArtifact) and may also be treated as an AbstractArtifact in external/collapsed cases." .

#################################################################
# Core structural relations (also participate in narrowerFacet)
#################################################################

<hasFlow> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <ArtifactFlow> ;
  rdfs:label "has flow" ;
  rdfs:comment "Relates an AbstractArtifact to its Flow. Mesh convention: exactly one Flow per AbstractArtifact." .

<hasState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <ArtifactState> ;
  rdfs:label "has state" ;
  rdfs:comment "Relates a Flow to one of its States." .

<hasCurrentState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <narrowerFacet>, <hasState> ;
  rdf:type owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <CurrentState> ;
  rdfs:label "has current state" ;
  rdfs:comment "Relates a Flow to its current (mutable-at-weave) State. Mesh convention: exactly one current state per Flow." .
  
<hasWorkingState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf <narrowerFacet>, <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <WorkingState> ;
  rdfs:label "has working state" ;
  rdfs:comment "Relates a Flow to its working (mutable) State. Mesh convention: exactly one working state per Flow." .

<hasHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet>, <hasState> ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "has historical state" ;
  rdfs:comment "Relates a Flow to a published historical State." .

<previousHistoricalState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf prov:wasRevisionOf ;
  rdfs:domain <HistoricalState> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "previous historical state" ;
  rdfs:comment "Links a HistoricalState to the immediately preceding HistoricalState in its published revision lineage." .

<latestHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasHistoricalState> ;
  rdf:type owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "latest historical state" ;
  rdfs:comment "Optional convenience pointer from a Flow to its latest (most recently published) HistoricalState. This is a logical pointer in RDF and does not imply any filesystem aliasing." .

<hasAbstractFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <AbstractFile> ;
  rdfs:label "has abstract file" ;
  rdfs:comment "Relates a State to one of its AbstractFile variants." .

<hasLocatedFile> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <AbstractFile> ;
  rdfs:range <LocatedFile> ;
  owl:inverseOf <locatesAbstractFile> ;
  rdfs:label "has located file" ;
  rdfs:comment "Relates an AbstractFile to one or more LocatedFiles that provide retrievable bytes." .

<locatesAbstractFile> a owl:ObjectProperty ;
  rdfs:domain <LocatedFile> ;
  rdfs:range <AbstractFile> ;
  owl:inverseOf <hasLocatedFile> ;
  rdfs:label "locates abstract file" ;
  rdfs:comment "Navigation: links a LocatedFile to an AbstractFile whose bytes it provides." .

#################################################################
# Optional skip-links (structural narrowing shortcuts)
#################################################################

<locatedFileForState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <narrowerFacet> ;
  rdfs:domain <ArtifactState> ;
  rdfs:range <LocatedFile> ;
  rdfs:label "located file for state" ;
  rdfs:comment "Optional shortcut relating a State directly to a LocatedFile. When present in mesh-managed data, it SHOULD be consistent with the chain: hasAbstractFile / hasLocatedFile." .

#################################################################
# Content kinds
#################################################################

<RdfDocument> a rdfs:Class ;
  rdfs:subClassOf <DigitalArtifact> ;
  rdfs:label "RDF document" ;
  rdfs:comment "A DigitalArtifact whose content is at least one RDF graph. Concrete bytes are an RDF document in an RDF serialization (e.g., Turtle, JSON-LD, TriG, N-Quads). May be used to type an AbstractArtifact, a Flow, specific State(s), an AbstractFile, or a LocatedFile, depending on modeling needs." .

#################################################################
# Web presentation
#################################################################

<ResourcePage> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "An HTML page that presents a resource. By convention, resource pages are located at a reserved 'index.html' located-file name. Mesh layout invariant: reserve 'index.html' for ResourcePages; other artifact facets SHOULD NOT use 'index.html' as their identifier token." .

<hasResourcePage> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource with a human-readable presentation page." .

#################################################################
# Reference data
#################################################################

<ReferenceLink> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a resource to a specific reference data source, including role metadata." .

<ReferenceRole> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRole/Canonical> a <ReferenceRole> ;
  rdfs:label "Canonical" .

<ReferenceRole/Supplemental> a <ReferenceRole> ;
  rdfs:label "Supplemental" .

<ReferenceRole/Deprecated> a <ReferenceRole> ;
  rdfs:label "Deprecated" .

<hasReferenceLink> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLink> ;
  owl:inverseOf <referenceLinkFor> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <hasReferenceLink> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the resource that a ReferenceLink is about." .

<hasReferenceRole> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range <ReferenceRole> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .

<referenceTarget> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a resource used as a reference-data identifier." .

<referenceUriLiteral> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "reference URI literal" ;
  rdfs:comment "If present, a retrievable location for reference data (without asserting denotation). May be used alongside referenceTarget or alone." .

<lastVerifiedAt> a owl:DatatypeProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "last verified at" ;
  rdfs:comment "Most recent verification timestamp for this reference link." .

<lastVerifiedBy> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range dcterms:Agent ;
  rdfs:label "last verified by" ;
  rdfs:comment "Agent responsible for the most recent verification of this reference link." .

#################################################################
#################################################################
# Semantic Flow resources
#################################################################
#################################################################

<SemanticFlowResource> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A resource modeled using Semantic Flow conventions and vocabulary. Semantic Flow resources may exist inside or outside a mesh." .

<containsSemanticFlowResource> a owl:ObjectProperty ;
  rdfs:subPropertyOf dcterms:hasPart ;
  rdfs:domain <ArtifactContainer> ;
  rdfs:range <SemanticFlowResource> ;
  rdfs:label "contains Semantic Flow resource" ;
  rdfs:comment "Structural containment relation for Semantic Flow resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

#################################################################
# Semantic Meshes
#################################################################

<Mesh> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains Semantic Flow resources and other supporting resources." .

<meshBase> a owl:DatatypeProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:anyURI ;
  rdfs:label "mesh base" ;
  rdfs:comment "A mesh's canonical BASE (publish location). Represented as a URI literal to avoid asserting anything about the denoted thing." .

#################################################################
# Naming
#################################################################

<Nomen> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Nomen" ;
  rdfs:comment "A naming-object resource (conventionally located at a reserved _nomen/ path) that records a designatorPath token (interpreted relative to the mesh base) and ties that token to what it is about. A Nomen may specify, via ReferenceLinks, additional sources of reference data about what it is about." .

<containsNomen> a owl:ObjectProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range <Nomen> ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:label "contains nomen"@en ;
  rdfs:comment "Asserts that a Nomen participates in (and is inventoried by) a Mesh's naming surface. This supports meshes whose naming vehicle exists independent of any filesystem serialization or dereferenceable publication."@en .

<designatorPath> a owl:DatatypeProperty ;
  rdfs:domain <Nomen> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Path token for the Nomen's designator. Interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root Nomen." .

<designates> a owl:ObjectProperty ;
  rdfs:domain <Nomen> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "designates" ;
  rdfs:comment "Optional asserted designatum for the Nomen's designator token. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for token management)." .

#################################################################
# Knops + payload artifacts
#################################################################

<Knop> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource>, <ArtifactContainer> ;
  rdfs:label "Knop" ;
  rdfs:comment "A container for primary payload artifacts and supporting artifacts." .

<containsKnop> a owl:ObjectProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range <Knop> ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:label "contains knop"@en ;
  rdfs:comment "Asserts that a Knop is a member of (and is inventoried by) a Mesh. This is a semantic containment/inventory assertion, not a filesystem directory listing, and can apply to meshes that exist only in memory."@en .

<PayloadArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <SemanticFlowResource> ;
  rdfs:label "Payload artifact" ;
  rdfs:comment "A primary AbstractArtifact hosted by a Knop." .

<containsPayloadArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <PayloadArtifact> ;
  rdfs:label "contains payload artifact" ;
  rdfs:comment "Links a Knop to its primary payload AbstractArtifact." .

<payloadSlug> a owl:DatatypeProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range xsd:string ;
  rdfs:label "payload slug" ;
  rdfs:comment "Stable, immutable slug used as an AbstractArtifact's identifier path segment in a mesh file layout (folder name). Changing it changes IRIs and should be treated as minting a new AbstractArtifact. Mesh invariant: payloadSlug MUST NOT be 'index.html' and SHOULD NOT start with '_' (reserved for structural items)." .

<preferredPayloadFileSlug> a owl:DatatypeProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload file slug" ;
  rdfs:comment "Mutable filename-stem preference for LocatedFiles (e.g., 'alice-bio'). Generators may ignore/sanitize it and must resolve collisions mechanically (e.g., suffix or hash). Mesh invariant: should avoid values that would result in a LocatedFile named 'index.html'." .

<preferredPayloadSlug> a owl:DatatypeProperty ;
  owl:deprecated true ;
  rdfs:domain <Knop> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred payload slug" ;
  rdfs:comment "DEPRECATED. Replaced by payloadSlug (immutable identifier segment) and preferredPayloadFileSlug (mutable filename preference)." .

#################################################################
# Metadata payload artifacts
#################################################################

<KnopMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Knop metadata artifact" ;
  rdfs:comment "Small, discoverable: structure/pointers/summary inventory for the Knop. Modeled as an AbstractArtifact with a Flow and States." .

<MeshMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Mesh metadata artifact" ;
  rdfs:comment "Optional mesh-level operational metadata/config. Modeled as an AbstractArtifact with a Flow and States." .

<NomenMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Nomen metadata artifact" ;
  rdfs:comment "Metadata about a Nomen. Modeled as an AbstractArtifact with a Flow and States." .

<containsKnopMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <KnopMetadataArtifact> ;
  rdfs:label "contains knop metadata artifact" ;
  rdfs:comment "Links a Knop to its knop metadata artifact (if present)." .

<containsMeshMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Mesh> ;
  rdfs:range <MeshMetadataArtifact> ;
  rdfs:label "contains mesh metadata artifact" ;
  rdfs:comment "Links a Mesh to its mesh metadata artifact (if present)." .

<containsNomenMetadataArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Nomen> ;
  rdfs:range <NomenMetadataArtifact> ;
  rdfs:label "contains nomen metadata artifact" ;
  rdfs:comment "Links a Nomen to its nomen metadata artifact (if present)." .

#################################################################
# Inventory artifacts
#################################################################

<MeshInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Mesh inventory artifact" ;
  rdfs:comment "Index for explorability at the mesh level." .

<KnopInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Knop inventory artifact" ;
  rdfs:comment "Index for explorability at the knop level." .

<NomenInventoryArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Nomen inventory artifact" ;
  rdfs:comment "Index for explorability at the nomen level." .

<containsMeshInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Mesh> ;
  rdfs:range <MeshInventoryArtifact> ;
  rdfs:label "contains mesh inventory artifact" ;
  rdfs:comment "Links a Mesh to zero or more derived MeshInventoryArtifacts (may be large and/or sharded)." .

<containsKnopInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Knop> ;
  rdfs:range <KnopInventoryArtifact> ;
  rdfs:label "contains knop inventory artifact" ;
  rdfs:comment "Links a Knop to zero or more derived KnopInventoryArtifacts (may be large and/or sharded)." .

<containsNomenInventoryArtifact> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsSemanticFlowResource> ;
  rdfs:domain <Nomen> ;
  rdfs:range <NomenInventoryArtifact> ;
  rdfs:label "contains nomen inventory artifact" ;
  rdfs:comment "Links a Nomen to zero or more derived NomenInventoryArtifacts (may be large and/or sharded)." .
---

latestHistoricalState really should never be written permanently to the filesystem? Or do we depend on people to realize that latestHistoricalState is not to be trusted in immutable artifacts. We should provide a way to filter out data when passing to immutability. The thing should have it and maybe the system. Later I want to explore the idea of independent artifacts committing to each other. Either that or we should make the artifact that stores "anti-immutable data" turned off by default. It seems like a huge improvement to the whole system. It only makes sense to have a companion artifact for the payload... the anti-payload of things that get scrubbed from the data.
ChatGPT said:

1) “Weave property on Knops” — yes, but make it explicitly weave-run metadata (not “new version” metadata)

You have two distinct facts you’re trying to represent:

    This Knop was processed in a weave run (even if nothing changed and no new HistoricalState was minted).

    This artifact produced a new immutable snapshot (a new HistoricalState) — that’s already represented by the existence of that state (plus its timestamp, if you add one).

So the Knop-level property should be about (1), not (2).
Minimal ontology terms (recommended)

Use a weave-event resource so “simultaneous” is representable without hand-waving.

#################################################################
# Weave runs
#################################################################

<WeaveEvent> a rdfs:Class ;
  rdfs:label "Weave event" ;
  rdfs:comment "A weave run over a mesh (or subset) at a particular time. A resource may participate in a weave without producing a new HistoricalState." .

<weaveAt> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <WeaveEvent> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "weave at" ;
  rdfs:comment "Timestamp for a weave event." .

<lastWeave> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <Knop> ;
  rdfs:range <WeaveEvent> ;
  rdfs:label "last weave" ;
  rdfs:comment "Most recent weave event in which this Knop participated." .

This gives you:

    “All Knop components weave simultaneously” → components inherit the Knop’s lastWeave (or you later add lastWeave to other component types if you truly need it).

    “Drop out of weave if unchanged” → you still may update lastWeave (it was processed), but you do not mint new HistoricalStates.

If you want an even cheaper authoring shortcut, you can add lastWovenAt too, but it’s redundant with lastWeave/weaveAt. I would skip it unless you have strong reasons.
Where it lives

You can still store the triple in KnopInventoryArtifact (or KnopMetadataArtifact), while the domain remains sflo:Knop. That matches your “inventory holds operational facts” approach.
2) Small changes you should make in the ontology you pasted (these are real foot-guns)

    You duplicate functional-property typing on hasCurrentState:

<hasCurrentState> a owl:ObjectProperty, owl:FunctionalProperty ;
  ...
  rdf:type owl:FunctionalProperty ;

Pick one. The a owl:ObjectProperty, owl:FunctionalProperty form is sufficient.

    Redundant rdfs:subPropertyOf <narrowerFacet> on hasCurrentState/hasWorkingState/hasHistoricalState.
    If they are subproperties of hasState, and hasState is already a subproperty of narrowerFacet, you don’t need to repeat it.

    SemanticFlowResource comment is still the wrong claim (you already noticed this). Fix it to reflect “mesh constituents,” not “inside or outside.”

Suggested replacement:

<SemanticFlowResource> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A first-class constituent of a Mesh (e.g., Mesh, Nomen, Knop, and the artifacts and derived resources they host), modeled using Semantic Flow conventions." .

    Your DigitalArtifact comment still says “may exist inside or outside any mesh.” That’s fine for DigitalArtifact (it’s general-purpose). Don’t conflate it with SemanticFlowResource.

3) latestHistoricalState persistence + immutability: do NOT rely on humans to “just know”

If you let latestHistoricalState be written into anything you call “immutable,” you will create silent contradictions over time.

Rule I recommend you make normative:

    sflo:latestHistoricalState is an operational derived pointer.

    It MAY appear in mutable/operational graphs.

    It MUST NOT appear in immutable HistoricalState snapshots/distributions.

You asked: “Should it never be written permanently to the filesystem?”
I’d say: It can be written, but only into an explicitly non-immutable place. That’s where your “anti-payload” idea is actually strong.
4) Your “anti-payload” / “scrubbed-from-immutability” companion artifact is a good idea — formalize it

This is one of the few “new concepts” here that genuinely reduces long-term confusion.
The pattern

    Keep “payload” artifacts clean: only statements intended to be preserved (or at least eligible).

    Put operational/derived/pointer-y facts in an overlay artifact (your “anti-payload”).

    Make overlays off by default and versioning-disabled by default (i.e., no HistoricalStates for the overlay flow).

You don’t need a new stack; reuse the same artifact stack and just set config defaults.
Minimal ontology terms

#################################################################
# Operational overlay ("anti-payload")
#################################################################

<OperationalOverlayArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Operational overlay artifact" ;
  rdfs:comment "Optional companion artifact that stores operational, derived, or otherwise non-immutable statements about another artifact or resource. Intended to be excluded from immutable HistoricalState snapshots." .

<operationalOverlayFor> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <OperationalOverlayArtifact> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "operational overlay for" ;
  rdfs:comment "Identifies the resource this operational overlay is about (often a payload AbstractArtifact or its Flow)." .

<hasOperationalOverlay> a owl:ObjectProperty ;
  owl:inverseOf <operationalOverlayFor> ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <OperationalOverlayArtifact> ;
  rdfs:label "has operational overlay" ;
  rdfs:comment "Inverse of operationalOverlayFor." .

Then latestHistoricalState can be stored as a triple about the Flow inside the overlay document, not inside any immutable snapshot.

This also sets you up for your “independent artifacts committing to each other” idea later: overlays are where you put cross-artifact commitments/pointers without polluting immutable content.
5) “We should provide a way to filter out data when passing to immutability”

You have two viable mechanisms; pick one and make it consistent:

Option A (cleaner, less magic): Don’t copy operational predicates into historical snapshots at all because they’re stored only in overlays.
Pros: simplest mental model.
Cons: you must consistently separate them during writing.

Option B (more general): add an annotation property that marks predicates as “exclude from historical.”
This is handy if you don’t want to force overlays for everything.

Minimal version:

<excludeFromHistoricalSnapshot> a owl:AnnotationProperty ;
  rdfs:label "exclude from historical snapshot" ;
  rdfs:comment "Weave processes SHOULD omit triples using this predicate from immutable HistoricalState snapshots/distributions." .

<latestHistoricalState> <excludeFromHistoricalSnapshot> true .
<hasCurrentState> <excludeFromHistoricalSnapshot> true .
<hasWorkingState> <excludeFromHistoricalSnapshot> true .
<lastWeave> <excludeFromHistoricalSnapshot> true .
<weaveAt> <excludeFromHistoricalSnapshot> true .

I prefer Option A as the default and Option B as an escape hatch.
6) SHACL additions you should make (in addition to WorkingState single AbstractFile)

If you add the weave terms above, add these two shapes:

    Knop has max 1 lastWeave

    WeaveEvent has exactly 1 weaveAt (dateTimeStamp)

If you want them in your JSON-LD SHACL context, say so and I’ll output them in the same style as the WorkingState shape you already requested.
One critical consistency check

If KnopInventoryArtifact is always present and can record weave participation, then lastWeave belongs there conceptually, not on the payload artifact. That separation is exactly what prevents “immutable data accidentally containing dynamic pointers.”

Your “anti-payload” concept is the right extension of that same separation discipline.
You said:
The other big surprise I have for you concerns our weave process. I'd formerly imagined a mostly-filesystem based, awkward process: declaring a soft lock; taking a checksum, copying working file; do processing, write files, check checksum; copy and rename . My new idea, which is at least a good alternate process: keep mesh in memory; run fs monitors to detect changes in covered files and update the mesh; on weave, stop the fsmonitor-backed change processing; write and rename files as needed; restart monitored-change-detection processing.
ChatGPT said:

This is a materially better mental model than “copy/rename/checksum gymnastics,” but only if you treat it as a transaction + reconciliation system, not as “watchers are truth.”
What your new approach gets right

    Single source of truth during operations: you operate on an in-memory mesh model, not an ad hoc pile of partially-written files.

    Cleaner weave semantics: weave becomes “commit a computed new mesh image to disk,” not “hope the filesystem is coherent while I mutate it.”

    Enables real incrementalism: “drop out if unchanged” becomes a first-class dirty-flag/hash check in memory.

The hard problems you’re about to inherit (and must design around)
1) You will miss changes during the pause unless you reconcile

If you “stop fsmonitor-backed change processing” during weave, two things can happen:

    changes still occur (user edits, editor autosaves, git checkout, etc.)

    watchers may buffer events, overflow, or drop events (depending on platform/tooling)

If you just pause and resume, you can silently diverge.

Non-negotiable fix: after weave completes and before resuming normal processing, do a reconciliation scan:

    compare (mtime, size) and/or content hash for every watched file against what your in-memory model thinks it last ingested

    ingest anything that differs

    optionally fail the weave if critical inputs changed mid-weave (see below)

2) Watchers are not reliable enough to be your only ingestion mechanism

Across OSes and filesystems, file watching has pitfalls:

    event coalescing (you get “changed” but not which changes)

    overflow/drop under high churn

    editors write temp files then rename; you see rename storms

    network filesystems behave differently

So: treat watchers as a hint stream, not as the authoritative log.
3) You need a “weave barrier” to prevent self-trigger loops

Weave writes will trigger watcher events. If you resume watchers immediately, you can:

    re-ingest outputs as if user edits

    trigger derived artifact regeneration loops

    thrash CPU

You need a mechanism to ignore or classify “writes produced by weave.”

Two workable approaches:

    Epoch tagging: increment a weaveEpoch counter; all write operations performed by the weave engine are recorded with that epoch; watcher events observed during the epoch are ignored or handled in a special “output update” path.

    Path partition: never watch generated output directories, only authoring inputs. (This is simpler and usually the right answer if your layout can support it.)

4) Mid-weave modifications: decide whether they invalidate the weave

If someone edits a working file while weave is running, you have to pick one:

    Fail-fast: capture input hashes at weave start; if any changed by weave end, abort or mark weave as invalid.

    Last-writer-wins: accept that weave output is based on the snapshot at start; later edits will be ingested after weave and will require another weave.

    Continuous integration: keep ingesting during weave (buffering events) and re-run affected steps; this is complex and not worth it yet.

For an MVP, I’d recommend Last-writer-wins + reconcile scan (and optionally warn if critical inputs changed).
A concrete transactional design that matches your intent
Components

    Mesh Model (in memory): authoritative graph-ish representation + file index + dirty flags

    Ingest Engine: takes “file changed” -> re-parse/re-canonicalize -> updates mesh model

    Watcher(s): produce events (create/modify/rename/delete)

    Weave Engine: snapshots the model and writes the outputs transactionally

Normal mode

    Watchers enqueue file events.

    Ingest Engine debounces and processes them, updating:

        parsed representation

        “dirty” status for impacted artifacts

        optional content hashes for working inputs

Weave mode (transaction)

    Enter weave epoch N

    Stop applying watcher events (but keep collecting them raw in a buffer if possible)

    Drain ingest queue to get a stable snapshot (or require “quiet period”)

    Snapshot the in-memory model (or lock it for read)

    Compute outputs:

        only for dirty artifacts

        generate to a staging area

    Atomic publish:

        write to temp files

        rename/replace atomically into place

    Update operational metadata (inventory/weave time) in the appropriate artifacts

    Reconciliation scan:

        detect edits that occurred during weave

        ingest them

    Resume normal processing (epoch N+1)

This gives you:

    determinism (snapshot-based)

    no awkward partial files

    robust recovery (reconciliation)

How this affects your ontology decisions (important, but small)

You’re correct that you need weave metadata on Knops, but with this architecture it becomes even clearer what it means:

    sflo:lastWeave on a Knop is: the last weave event in which it was processed, not “the last time it changed.”

    “Drop out if unchanged” means: the Knop still participates in the weave epoch, but its payload artifacts don’t mint new HistoricalStates.

Store lastWeave in the Knop’s operational/inventory artifact (or overlay), not inside immutable payload snapshots.
The “latestHistoricalState should not be persisted in immutable artifacts” issue becomes easier here

With a transactional weave engine, you can enforce a simple rule:

    Immutable snapshots contain only stable relations.

    Derived pointers like latestHistoricalState live in an overlay/operational artifact (or are computed on read).

This aligns perfectly with your “anti-payload” idea: operational overlays become the home for:

    latestHistoricalState

    lastWeave

    resolver caches / derived links

    cross-artifact commitments (later)

Minimal next step to validate the approach (before more ontology work)

Build a toy implementation with just these mechanics:

    in-memory file index + hash

    watcher events -> mark dirty

    weave transaction writes a single derived file with atomic replace

    reconciliation scan catches edits during weave

If that works, your ontology work will stop feeling hypothetical, because you’ll have concrete pressure:

    which facts must be persisted vs derived

    which artifacts hold operational metadata

    what must be immutable

If you want to pin down the weave property now: tell me whether you prefer a timestamp-only (lastWovenAt) or event object (lastWeave -> WeaveEvent -> weaveAt). The event-object choice will age better once you track “weave over subset,” “weave reason,” or “weave tool version,” but it’s slightly heavier.
You said:
Make overlays ON by default and put the latestHistoricalState. It should really go in an anti-metadata artifact. You kinda need an anti- for every Artifact
ChatGPT said:

Making overlays ON by default and treating sflo:latestHistoricalState as anti-metadata is the right direction—but you must address one structural trap immediately:
Critical trap: infinite regress

If “every Artifact has an anti-artifact,” then the anti-artifact is itself an sflo:AbstractArtifact, so it would also need an anti-artifact, and so on forever.

You need a stop rule. The cleanest is:

    Every Primary AbstractArtifact SHOULD have exactly one AntiMetadataArtifact.

    AntiMetadataArtifacts do not themselves get anti-artifacts.

You don’t need to coin “PrimaryAbstractArtifact” yet; you can enforce the stop rule by SHACL (“anti has no anti”) and by how the weaver creates things.
Proposed ontology additions
1) Class: sflo:AntiMetadataArtifact

A companion artifact storing volatile/derived/system-only statements about another artifact.

<AntiMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Anti-metadata artifact" ;
  rdfs:comment "A companion artifact that stores operational, derived, or otherwise non-immutable statements about another artifact (e.g., pointers like latestHistoricalState). Anti-metadata artifacts are excluded from immutable HistoricalState snapshots/distributions by convention." .

2) Link it to the artifact it is “anti-” for

<antiMetadataFor> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <AntiMetadataArtifact> ;
  rdfs:range <AbstractArtifact> ;
  rdfs:label "anti-metadata for" ;
  rdfs:comment "Identifies the AbstractArtifact that this anti-metadata artifact is about." .

<hasAntiMetadataArtifact> a owl:ObjectProperty, owl:FunctionalProperty ;
  owl:inverseOf <antiMetadataFor> ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <AntiMetadataArtifact> ;
  rdfs:label "has anti-metadata artifact" ;
  rdfs:comment "Inverse of antiMetadataFor: associates an AbstractArtifact with its companion anti-metadata artifact." .

3) Policy: where latestHistoricalState lives

    sflo:latestHistoricalState MAY be asserted, but only in the AntiMetadataArtifact content (or other explicitly operational graphs).

    It MUST NOT be copied into immutable HistoricalState snapshots/distributions.

You don’t need new properties for that, just a hard convention for the weaver and (optionally) an annotation to mark “operational-only” predicates later.
What “anti-metadata” contains

The AntiMetadataArtifact should contain operational statements about the target artifact and its sub-facets, especially:

    sflo:hasFlow link(s) (if you want the anti doc to be self-contained)

    sflo:latestHistoricalState on the Flow

    sflo:lastWeave / sflo:WeaveEvent pointers (if you go that route)

    derived caches (e.g., “resolved-as-of” pointers, assembly member expansions, index sharding pointers)

This matches your direction: payload stays clean; anti-metadata is where “not safe to freeze forever” facts go.
SHACL to enforce the stop rule and the linkage

In your JSON-LD SHACL style, you want three basic constraints:

    AntiMetadataArtifact must point to exactly one AbstractArtifact

    AbstractArtifact should have exactly one AntiMetadataArtifact (for mesh-managed validation)

    AntiMetadataArtifact must not itself have an anti-metadata artifact (prevents infinite regress)

Here’s the SHACL (JSON-LD) using your context conventions:

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
      "@id": "AntiMetadataArtifactForExactlyOneAbstractArtifactShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AntiMetadataArtifact antiMetadataFor exactly one AbstractArtifact",
      "sh:targetClass": "sflo:AntiMetadataArtifact",
      "sh:property": [
        {
          "sh:path": "sflo:antiMetadataFor",
          "sh:class": "sflo:AbstractArtifact",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "AbstractArtifactHasExactlyOneAntiMetadataArtifactShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractArtifact has exactly one AntiMetadataArtifact (mesh-managed convention)",
      "rdfs:comment": "Mesh-managed validation rule: every AbstractArtifact should have exactly one companion AntiMetadataArtifact.",
      "sh:targetClass": "sflo:AbstractArtifact",
      "sh:property": [
        {
          "sh:path": "sflo:hasAntiMetadataArtifact",
          "sh:class": "sflo:AntiMetadataArtifact",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "AntiMetadataArtifactMustNotHaveAntiMetadataArtifactShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AntiMetadataArtifact must not have its own AntiMetadataArtifact",
      "rdfs:comment": "Prevents infinite regress: anti-metadata artifacts do not themselves get anti-metadata companions.",
      "sh:targetClass": "sflo:AntiMetadataArtifact",
      "sh:property": [
        {
          "sh:path": "sflo:hasAntiMetadataArtifact",
          "sh:maxCount": 0,
          "sh:severity": "sh:Violation"
        }
      ]
    }
  ]
}

If you want the “exactly one anti per artifact” rule to apply only to mesh-managed artifacts (not external), we can scope the target with sh:targetSubjectsOf sflo:containsPayloadArtifact etc., but the above is the straightforward default.
Answering your two concrete points
“Overlays ON by default”

Yes. But treat it as a weaver default behavior, not an ontology axiom:

    On artifact creation, auto-create its AntiMetadataArtifact.

    On weave, update anti-metadata facts (including latestHistoricalState).

“latestHistoricalState goes in anti-metadata”

Yes. And you should make it a derived pointer your weaver can recompute, not a “truth claim” you preserve immutably.
One more consistency requirement you should adopt now

If AntiMetadataArtifact exists for every AbstractArtifact, you must define where it lives in the mesh layout. Otherwise it will sprawl.

A reserved segment is enough; pick one and stick to it:

    _anti/ or _ops/ or _overlay/

I would choose _anti/ given your naming.

If you want, paste (or point to) your current “Knop always contains KnopMetadataArtifact and KnopInventoryArtifact” rule, and I’ll show the cleanest place to link the payload artifact to its anti-metadata artifact (so UIs and tooling can find it without scanning).
You said:
1) "You will miss changes during the pause unless you reconcile" Changes are only ever allowed on WorkingStates, apps can continue to abuse it in any way they can. And you can't weave in the meantime, until the previous weave succeeds or is rolled back (or aborted/inconsistent) All mesh-level stuff? At least to start with? Or we get into -per-knop weave specification immediately. 

You can do a reconciliation scan, sure. As long as only the WorkingState's WorkingLocatedFile is the only thing changed, I think you are golden. Eventually we can check in more detail and crosscheck the consistency/checksumming/etc of the mesh.

the "watchers" should be going continuously, adding to a queue ideally of things that

I wrote a little definiton of "path segment":

A chunk of a path, i.e., usually between the slashes. Because IRI paths (below the base) reflect filesystem paths, there is a direct correspondence to filesystem directory (aka folder) names. But "path segment" is the recommended term (unless you are particularly talking about a folder), as it can apply equally to IRI segments and filesystem path segments.

Should every Knop have a Nomen? a system-supported way of listing reference data via ReferenceLinks, and it makes sense for them as participants in the mesh namespace.

Oh, yeah, we need a new type of "flexible path segment" that can include trailing (or leading?) slashes
ChatGPT said:
1) Weave + fs-monitors: your constraint (“only WorkingState WorkingLocatedFile is mutable”) makes the pause/resume model viable, but you still need a barrier + reconciliation

If only WorkingState’s WorkingLocatedFile can change, then a weave can safely treat everything else as read-only. That’s a strong simplification.

However, two failure modes remain even under that rule:

    Edits happen during weave (which you allow): the weave output may represent an earlier snapshot than what ends up in WorkingState by the time weave finishes.

    Watch events are not a complete log (buffer overflow, coalescing, rename-save patterns).

So the minimal correct protocol is:

    Watchers run continuously, enqueueing “working file changed” events (do not treat watcher events as canonical truth).

    At weave start: stop applying events, but keep collecting them (or accept they may be dropped).

    Weave operates on an in-memory snapshot of the mesh model.

    At weave end: do a reconciliation scan over the allowed-mutable set (the WorkingLocatedFiles):

        If any changed during weave, mark the corresponding artifacts dirty and (optionally) require another weave to capture them.

        If anything outside the allowed set changed, treat as invalid weave (abort or mark inconsistent).

This is consistent with your “can’t weave again until previous weave succeeds/aborts/rolls back.”
Mesh-level weave vs per-knop weave

Start mesh-level. It’s simpler and aligns with “one barrier, one epoch.” You still get “drop out if unchanged” by skipping generation per artifact whose working hash didn’t change.

Per-knop weave is a later optimization that adds real complexity (multiple barriers, partial ordering, assembly dependencies). Don’t start there unless you already need it.
2) latestHistoricalState: do not persist it in immutable artifacts; put it in anti-metadata (and make anti-metadata ON by default)

Your instinct is correct. latestHistoricalState is a derived pointer; if it leaks into immutable snapshots it becomes a time bomb.

You said: “It should really go in an anti-metadata artifact… you kinda need an anti- for every Artifact.”

You can do that, but you must prevent infinite regress:

    Every primary AbstractArtifact has exactly one AntiMetadataArtifact (ON by default).

    AntiMetadataArtifact does not itself get an anti-artifact.

Minimal ontology additions

<AntiMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Anti-metadata artifact" ;
  rdfs:comment "Companion artifact for operational/derived/non-immutable statements about another AbstractArtifact (e.g., latestHistoricalState). Excluded from immutable HistoricalState snapshots/distributions by convention." .

<hasAntiMetadataArtifact> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <AntiMetadataArtifact> ;
  owl:inverseOf <antiMetadataFor> ;
  rdfs:label "has anti-metadata artifact" .

<antiMetadataFor> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <AntiMetadataArtifact> ;
  rdfs:range <AbstractArtifact> ;
  rdfs:label "anti-metadata for" .

Then enforce “anti has no anti” in SHACL (otherwise you get infinite regress).

Operationally: write latestHistoricalState triples into the anti-metadata document, not into historical snapshots.
3) Watcher queue: what should be in it (and what must never be in it)

Given your mutability rule, the watcher queue should contain only:

    WorkingLocatedFileChanged(path) events (debounced/coalesced)

    optionally WorkingLocatedFileDeleted(path) / Renamed(old,new) if your editor does atomic saves

And it should explicitly ignore:

    generated outputs (_current/, historical folders, inventory artifacts, anti-metadata outputs) unless you intentionally watch them for verification.

If you ever allow other files to be edited, your entire “pause and still be golden” claim breaks, so keep the watcher scope narrow.
4) “Path segment” definition: good; the next step is to stop calling non-segments “segments”

Your definition is correct and matches standard usage: a path segment is between slashes.

Where you’re about to hurt yourself is here:

    “a new type of flexible path segment that can include trailing (or leading?) slashes”

A string that can include leading/trailing slashes is not a path segment; it’s a relative path token (or a path string). A segment cannot contain / by definition.

So split the concepts:

    PathSegmentToken: one segment, no slashes (used for payloadSlug, reserved-name checks, etc.)

    RelativePathToken: one-or-more segments separated by /, no leading slash; trailing slash allowed only if you mean “directory designator”

You can enforce both via SHACL regex rather than pretending they’re the same “datatype.”

Suggested normalization rules for RelativePathToken:

    trim whitespace

    remove all leading /

    collapse multiple / to single / (optional)

    preserve or normalize trailing / depending on whether it denotes a directory IRI in your system

5) Should every Knop have a Nomen?

Not as a hard invariant.

Reasons it should not be required:

    Many Knops are internal/operational (especially once you add anti-metadata artifacts everywhere).

    You may want multiple Nomens (aliases) designating the same Knop.

    You may want Nomens for things that aren’t Knops.

Recommendation:

    Every public/discoverable Knop SHOULD be designated by at least one Nomen.

    Internal/system Knops MAY have zero Nomens.

You already have the machinery: a Nomen can designates any resource. Add the inverse for convenience:

<designatedByNomen> a owl:ObjectProperty ;
  owl:inverseOf <designates> ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <Nomen> ;
  rdfs:label "designated by nomen" ;
  rdfs:comment "Inverse of designates: links a resource to a Nomen that designates it." .

Then SHACL can express “Knop must have ≥1 designatedByNomen” only in the contexts you choose (e.g., only for Knops included in a published inventory).
6) Weave property on Knops: model “participated in a weave” separately from “changed”

Your requirement is: Knop has a weave marker even if nothing versioned.

Use a WeaveEvent and a pointer:

<WeaveEvent> a rdfs:Class ;
  rdfs:label "Weave event" ;
  rdfs:comment "A weave run over a mesh at a particular time." .

<weaveAt> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <WeaveEvent> ;
  rdfs:range xsd:dateTimeStamp ;
  rdfs:label "weave at" .

<lastWeave> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <Knop> ;
  rdfs:range <WeaveEvent> ;
  rdfs:label "last weave" ;
  rdfs:comment "Most recent weave event in which this Knop participated. Does not imply any new HistoricalState was minted." .

Store it in Knop’s inventory/anti-metadata output (operational), not in immutable payload snapshots.
Immediate fixes to your pasted ontology (small but necessary)

    Remove the duplicate owl:FunctionalProperty typing on hasCurrentState.

    Remove redundant rdfs:subPropertyOf <narrowerFacet> from hasCurrentState/hasWorkingState/hasHistoricalState since they already subproperty hasState.

    Update SemanticFlowResource comment (you already said it’s wrong).

If you want the next concrete deliverable: I can output (a) SHACL shapes for WeaveEvent/lastWeave, (b) SHACL regex constraints for PathSegmentToken vs RelativePathToken, and (c) SHACL constraints preventing anti-metadata infinite regress.
You said:
I guess you can allow anti- use to keep operational memory that gets written into a separate mesh. eventually the service should be able to host multiple meshes. So we need an operational property for a mounted mesh name. It could be a variation on the mesh's preffered management-handle-name
ChatGPT said:

You’re mixing two concepts that should stay separate, even if one can default from the other:

    Management-handle path segment: how a mesh exposes its management surface inside its own IRI space (e.g., _mesh/). This is fundamentally a path-segment token.

    Mounted mesh name: how a hosting service identifies/routs to a mesh among multiple meshes (e.g., prod-west, dave-main, mesh-17). This is fundamentally a service-level identifier, not a path segment.

They can be related, but they are not the same thing.
Minimal ontology terms
A) Preferred management handle segment (stable preference)

This is the “variation on preferred management-handle-name” you referenced. Keep it on the mesh itself; it’s a preference about how the mesh is shaped.

<preferredManagementHandleSegment> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred management handle segment" ;
  rdfs:comment "Preferred reserved path segment token used to surface the mesh management handle within the mesh's base (e.g., '_mesh'). Must be a single path segment (no slashes)." .

B) Mounted mesh name (operational, host-specific)

This is what you asked for. Make it explicitly operational. Put the triples in your anti-metadata / operational overlay for the mesh, not in immutable snapshots.

<mountedMeshName> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:string ;
  rdfs:label "mounted mesh name" ;
  rdfs:comment "Operational mount identifier for this Mesh within a hosting service instance (used for routing when multiple meshes are hosted). This is host-specific operational state and should be stored in anti-metadata / operational overlay, not immutable snapshots." .

Defaulting rule (implementation, not ontology):

    If mountedMeshName is absent, default to preferredManagementHandleSegment (or to a repo slug / configured name). But do not pretend they’re the same property.

Future-proofing: you will eventually want a mount relator

If the service can host the same mesh mounted multiple times (or in multiple hosts), mountedMeshName on the Mesh becomes underspecified. The correct model is a mount object:

<MeshHost> a rdfs:Class ;
  rdfs:label "Mesh host" ;
  rdfs:comment "A service instance or artifact container that can host multiple meshes." .

<MeshMount> a rdfs:Class ;
  rdfs:label "Mesh mount" ;
  rdfs:comment "Relator describing a mesh mounted into a MeshHost under a particular name/route." .

<hasMeshMount> a owl:ObjectProperty ;
  rdfs:domain <MeshHost> ;
  rdfs:range <MeshMount> ;
  rdfs:label "has mesh mount" .

<mountsMesh> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <MeshMount> ;
  rdfs:range <Mesh> ;
  rdfs:label "mounts mesh" .

<mountName> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <MeshMount> ;
  rdfs:range xsd:string ;
  rdfs:label "mount name" ;
  rdfs:comment "Host-level identifier used to route to the mounted mesh." .

Then sflo:mountedMeshName becomes a convenience derived view (“primary mount name”) if you still want it. But you don’t need to introduce MeshHost/MeshMount immediately unless you are already implementing multi-mesh hosting.
Constraint hygiene (don’t repeat the earlier mistake with “segment that can include slashes”)

    preferredManagementHandleSegment should be validated as a single path segment (no /).

    mountedMeshName should be validated as an identifier, not as a segment. It can be stricter or looser than segment rules depending on what your router wants.

If you want one “flexible path segment” concept, name it correctly:

    PathSegmentToken: no slashes, no leading/trailing slashes (because they are separators).

    RelativePathToken: may contain slashes between segments; no leading /; trailing / allowed only if you intentionally use it to mean “directory designator.”

Trying to call a string that may include / a “segment” will produce downstream contradictions.
Where to store it (given your anti-metadata direction)

    mountedMeshName is exactly the kind of thing that should live in the mesh’s anti-metadata artifact (or in a separate “operational mesh” you mount alongside it).

    preferredManagementHandleSegment can live in “normal” mesh metadata because it’s a stable preference about the mesh’s public structure.

If you want, I can output SHACL JSON-LD shapes that enforce:

    preferredManagementHandleSegment has no /

    mountedMeshName matches a router-safe pattern (e.g., ^[a-z][a-z0-9-]{0,62}$), or a looser one if you need namespaces like prod/west (but then it’s not a name, it’s a path).

You said:
Instead of every Knop having a Nomen, I guess knops can hold their own ReferenceLinks in a metadata, or maybe a separate artifact. Oh goody, another supporting artifact type. Does it make sense for Nomen to keep the ReferenceLinks in a ReferenceLinkFlow. And probably Meshes should too.

Maybe we should by default not have a weave marker if nothing versioned (if no historical state got created).

In the management space, I think we can use named graphs to hold different meshes simultaneously; helps with sparql. I guess both the mesh and the application will have a management space. 

Related to the "two knops can pair each other", there's the idea of n-book accounting, where if you need to record a transaction (like e.g. a checksum verification, or an application event) between n attestors or watcher-subscribers, the each co-sign the event. Independent mesh operators can thereby attest to shared data.

It seems like a bad idea to allow the same mesh to be mounted multiple times. Maybe for testing?

Does this pattern below give you way to make make a "direct" (non relator) property work via inverse as a relator property?

<hasReferenceLink> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLink> ;
  owl:inverseOf <referenceLinkFor> ;
  rdfs:label "has reference link"@en ;
  rdfs:comment "Associates a resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <hasReferenceLink> ;
  rdfs:label "reference link for"@en ;
  rdfs:comment "Identifies the resource that a ReferenceLink is about." .

<hasReferenceRole> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink> ;
  rdfs:range <ReferenceRole> ;
  rdfs:label "has reference role" ;
  rdfs:comment "Assigns one or more ReferenceRoles to this ReferenceLink, indicating how the reference data should be used." .
ChatGPT said:
ReferenceLinks: don’t add a new per-resource artifact type unless you enjoy artifact explosion

You have three distinct kinds of “reference-ish” statements mixed together:

    Stable reference assertions (e.g., “this thing has a Wikidata ID”)

    Operational/ephemeral facts about references (e.g., lastVerifiedAt, watcher-derived hints)

    A storage strategy (where these triples live: per resource, per Knop, per Mesh, etc.)

If you treat (2) as first-class, you’ll end up needing an “anti-” surface anyway. So the clean split is:

    Stable ReferenceLinks live in normal metadata (and can be woven/versioned if you want).

    Operational fields (lastVerifiedAt, attestations, cache pointers like latestHistoricalState) live in anti-metadata (or operational mesh).

Where should the ReferenceLinks live?

You do not need “ReferenceLinkFlow per Nomen/Knop/Mesh” as a new artifact type. Instead:

    Add one artifact per Mesh: MeshReferenceLinksArtifact (an AbstractArtifact + RdfDocument) that contains ReferenceLink relators for many resources in the mesh.

This avoids “one more supporting artifact type per resource” and still gives you a proper place to put link relators.

If you still want convenience on Nomen: fine—allow Nomen to have hasReferenceLink, but treat it as a view of the mesh reference registry (“these ReferenceLinks are about the thing this Nomen designates”), not as the canonical storage.

Meshes should also be able to have ReferenceLinks (for the mesh itself), same for Knops. Domain rdfs:Resource already allows this.
Weave marker: you’re about to lose useful debugging signal if you drop it entirely

“If nothing versioned, no weave marker” is reasonable only if you still record weave runs somewhere else.

Better split:

    Mesh-level weave event: always recorded (even if nothing versioned), because “a weave happened” matters operationally.

    Per-Knop marker: update only when something meaningful happened for that Knop (new HistoricalState minted, or derived outputs regenerated).

Concretely:

    sflo:Mesh gets sflo:lastWeave (always advances).

    sflo:Knop gets sflo:lastVersioningWeave (advances only when a historical state is created for any payload/metadata artifact under the Knop).

This matches your “drop out if unchanged” behavior while keeping observability.
Management space via named graphs: good, but keep the boundaries explicit

Using named graphs to host multiple meshes simultaneously is strong for SPARQL, but only if you treat it as an operational dataset, not part of the immutable content model.

Two critical requirements:

    Graph naming convention
    Pick one:

    graph name = mesh IRI

    graph name = mounted mesh name IRI

    graph name = a stable management-handle IRI

Do not leave this fuzzy or queries will become unportable.

    Separation of concerns

    Mesh management space: facts about the mesh (inventory, pointers, operational overlay, watcher state, last weave, etc.)

    Application management space: app-level events, caches, user sessions, router state, etc.

They can live in the same quadstore, but do not mix them in the same named graph unless you want SPARQL queries that can’t be reasoned about.
“N-book accounting” / co-signed events: model it as an attestation relator, not as “each Knop pairs”

This is plausible and fits your relator style, but it can explode if you try to co-sign statements rather than digests.

Minimum viable model:

    AttestationEvent (relator)

        attestsResource (what is being attested to: a file, a state, an artifact, a dataset)

        attestsDigest (hash/digest of bytes or canonicalized RDF)

        attestedBy (one or more agents)

        attestedAt (timestamp)

        optional attestationMethod (watcher, manual, CI, external operator)

Store these in anti-metadata / operational mesh, not in immutable historical snapshots.

Also: you will eventually need a conflict story (two attestors disagree). Don’t pretend co-signing implies consensus; it’s just evidence.
“Same mesh mounted multiple times”: you can forbid it operationally, but keep the concept model capable

Even if you believe it’s a bad idea, it’s useful for:

    testing

    staging vs prod routes

    offline mirrors / replay environments

Recommendation:

    Support mount as a relator conceptually, but add a host-level SHACL/config constraint: one mount per mesh per host (unless explicitly enabled).

Your inverse-property ReferenceLink pattern: yes, it’s valid—but it does not make “non-relator direct properties” magically work

This pattern is fine:

    Resource --hasReferenceLink--> ReferenceLink

    ReferenceLink --referenceLinkFor--> Resource (inverse)

    plus role/target fields on ReferenceLink

What it gives you:

    You can assert from either side.

    You can navigate conveniently.

    The metadata stays attached to a relator node (good).

What it does not give you:

    A way to assert a “direct property” (like hasWikidataId) without creating a ReferenceLink node.

If you want a direct shortcut property, add one explicitly and (optionally) derive it:

    Shortcut: hasReferenceTarget (domain rdfs:Resource, range rdfs:Resource or xsd:anyURI)

    Derivation: property chain hasReferenceLink o referenceTarget -> hasReferenceTarget

Limitations:

    You cannot cleanly make role-specific shortcuts in OWL alone (e.g., “canonical target”) without extra machinery (SHACL rules, SPARQL CONSTRUCT, or a dedicated property like hasCanonicalReferenceLink).

Net recommendation on your “Nomen vs Knop ReferenceLinks” question

    Do not require every Knop to have a Nomen.

    Do allow Knops to have ReferenceLinks (domain already supports it).

    Make the canonical storage a mesh-level ReferenceLinksArtifact, so you don’t proliferate per-resource artifacts.

    Put operational aspects (verification timestamps, derived pointers like latestHistoricalState, attestation events) in anti-metadata / operational mesh by default.

If you want, paste your current “artifact roster per Knop” (payload + knop metadata + knop inventory + anti-metadata), and I’ll suggest exactly where the mesh-level ReferenceLinksArtifact fits without adding a new per-Knop mandatory artifact.
You said:
RelativePathToken is really about being able to specify the trailing slash. Which is hard to do in the filesystem. So TrailingSlashPathToken could be used to represent a terminal resource or just an IRI with a trailing Slash. I don't think we need it for our IRI assembly strategy via base, but conceivably you would want to specify the meshPath

WE should probably anti-immutable metadata. 

ReferenceLinks should be stored per Nomen/Knop/Mesh so they are transposable. ReferenceLinks are the whole semantic meaning for non-denoting knops. They are important and per-container (so also mesh).

Mesh operations should not inject NOOP weave event data into a Knop's metadata. Keep it in operational data if you must. 

I think graph name = mesh IRI is nice. guaranteed unique. Why would you ever mount one more than once? Seems like a bad idea.
ChatGPT said:
1) “TrailingSlashPathToken” — your instinct is right, but name it so it doesn’t contradict “segment”

If the core purpose is “I can represent an IRI that ends in / even when the filesystem can’t,” then it’s not a segment datatype; it’s a relative IRI path (directory-designator) token.

If you call it a “path token” you avoid the contradiction that a “segment” cannot contain /.
Suggested term(s)

    DirectoryDesignatorPathToken (preferred; says what it means)

    or your TrailingSlashPathToken (fine, but slightly lower semantic clarity)

Suggested lexical rule

    No leading slash.

    May contain internal slashes.

    May optionally end with a slash.

    Empty string allowed only if you want “mesh root designator.”

SHACL regex examples (pick strictness):

    allow empty: ^$|^[^/][^]*$ is too loose; better explicit:

        no leading slash, allow trailing: ^(?:[^/]+/)*[^/]+/?$|^$

But the bigger point: don’t call it “segment.” You’re encoding a path, not a segment.
Where it’s actually useful

    meshPath or “designatorPath” that might intentionally end with / to denote a terminal slash-IRI.

    any place you need a stable IRI token independent of filesystem conventions.

You’re correct that you probably don’t need it for your current base-assembly strategy, but it’s a good “escape hatch” datatype.
2) “anti-immutable metadata” — do it, but define the boundary precisely

You already converged on:

    immutable snapshots contain only stable content

    derived pointers / operational facts live elsewhere

Make that a first-class rule:

    AntiMetadataArtifact (per AbstractArtifact, ON by default) holds:

        latestHistoricalState

        verification timestamps, watcher state

        caches, derived “current pointers,” etc.

Additionally, consider:

    AntiMetadata for containers (Mesh/Knop/Nomen) as well, not only payload artifacts.

        otherwise you’ll end up shoving operational facts into their “normal” metadata artifacts by convenience.

This matches your “WE should probably anti-immutable metadata.” Yes: treat “anti-” as where non-immutable statements live.
3) ReferenceLinks per Nomen/Knop/Mesh for transposability — good, but you’re accepting duplication; make that explicit

Storing ReferenceLinks per container is consistent with your transposability goal if the ReferenceLinks are treated as part of the container’s semantic payload.

Two consequences you should accept up front:

    Duplication is normal
    The same external reference might be present:

    on the Nomen (naming surface)

    on the Knop (artifact container)

    on the Mesh (top-level)

So you need a dedup/precedence strategy for consumers (“which link wins?”).

    ReferenceLinks become part of container meaning
    This matches your claim: “ReferenceLinks are the whole semantic meaning for non-denoting knops.” That’s coherent.

Structural recommendation (without inventing new artifact types)

Keep your existing pattern:

    Nomen/Knop/Mesh (as rdfs:Resource) can hasReferenceLink directly.

Store them in:

    their normal metadata artifact (if you want them versioned), OR

    their anti-metadata artifact (if you want them operational-only), OR

    both, split stable vs operational fields as noted below.

Do not create a separate “ReferenceLinkFlow” unless you have a concrete lifecycle reason that differs from the container’s metadata lifecycle. Most of the time, it doesn’t.
Split stable vs operational fields

    Stable: referenceTarget, referenceUriLiteral, roles

    Operational: lastVerifiedAt, lastVerifiedBy

Put operational parts in anti-metadata by default. Otherwise you’ll freeze “last verified” values into historical snapshots and it will look ridiculous later.
4) NOOP weave events: you’re right to reject them in Knop metadata

Your statement is internally consistent:

    “Mesh operations should not inject NOOP weave event data into a Knop’s metadata.”

    “Keep it in operational data if you must.”

So the policy becomes:

    If a Knop produces no new HistoricalState for any of its artifacts, then:

        do not update Knop’s normal metadata/inventory artifacts

        optionally update Knop’s anti-metadata (operational) if you need observability

Mesh-level weave events can still be recorded (operationally) because “a weave happened” can matter for the system, but it doesn’t belong in content.

This also dovetails with your earlier “maybe by default not have a weave marker if nothing versioned.” Yes—at least for Knops.
5) Graph name = mesh IRI — agree; that’s the cleanest default

Your rationale is solid: uniqueness and portability.

Operationally, if you host multiple meshes in one quadstore:

    dataset default graph: application/global config (optional)

    named graph <meshIRI>: management/operational graph for that mesh (or the mesh’s assembled state)

    optionally additional named graphs for subspaces (inventory graph, anti-metadata graph) if you want query hygiene

But graph name = mesh IRI is a strong default.
6) “Why mount more than once?” — mostly you won’t; but don’t let that force an ontology lie

You can make “only mount once” a service constraint, not a conceptual impossibility.

Reasons you might mount more than once (even if you dislike it):

    staging vs prod route names pointing at the same underlying mesh snapshot

    testing a mesh under a different base/router path

    read-only mirror mount

    replay/debug (“mount mesh at weave epoch N”)

So: enforce “one mount per mesh per host by default,” but don’t bake “impossible” into the core model. You can simply not implement multiple mounts until needed.
One key correction on your path-token point

    “RelativePathToken is really about being able to specify the trailing slash.”

Right—and that means it should be treated as IRI-designator syntax, not filesystem syntax. That’s exactly why it should be a distinct token type.

The moment you tie it to filesystem paths, you’ll lose trailing-slash fidelity. So this is one place where you should explicitly decouple IRI token modeling from the filesystem.
Concrete next changes I would make (minimal edits)

    Add DirectoryDesignatorPathToken (or your TrailingSlashPathToken) as a datatype-like convention (SHACL pattern) used by meshPath / similar properties.

    Add AntiMetadataArtifact + hasAntiMetadataArtifact + antiMetadataFor and adopt the rule: latestHistoricalState is written only there.

    Adopt ReferenceLink storage rule:

        containers can store stable ReferenceLinks in normal metadata

        verification fields in anti-metadata

    Adopt NOOP policy:

        no Knop weave markers unless versioning occurred

        mesh-level weave can be recorded operationally

If you want, paste the specific properties you intend for meshPath (or whatever you call it), and I’ll give you the exact SHACL patterns for:

    path segment token

    directory-designator (optional trailing slash) token

You said:
I'm thinking Meshes only have MeshReferenceLinks for their own (as a mesh) references. "This mesh is great! -djradon" Nomen have their reference data "you can understand my non-DigitalArtifact denotation only indirectly, because I am Alice, the person, and my bio is over in that Knop. " No need for authoritativeness, people will have to judge their source and behave with judgement. But hey, it's an open world. I think we should keep reference links away from metadata. They are inherently curated, and could grow large. Let's keep metadata slim and factual. metatdata is really KnopMetadata, not denotee metadata.
ChatGPT said:

Your partitioning intuition is coherent, but you’re about to create a silent inconsistency unless you tighten one distinction:

    Knop metadata (structural, machine-slim, “factual”): pointers, invariants, slugs, flows, states, inventory scaffolding.

    Reference links (curated, potentially large, perspective-laden): sources, evidence, endorsements, identity bridges.

That split is good. The inconsistency risk is that your current model treats ReferenceLinks as attachable to any resource (rdfs:Resource), which invites them to leak into “metadata” artifacts unless you define an explicit storage rule.
Recommendation: keep ReferenceLinks as relators, but make them live in dedicated ReferenceLinks artifacts, not metadata artifacts

You can do this without inventing an explosion of artifact types if you use one generic pattern:
1) A single generic artifact class

<ReferenceLinksArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Reference links artifact" ;
  rdfs:comment "A curated RDF document that contains ReferenceLink relators about the host resource and/or the things it designates. Intended to be separate from slim structural metadata artifacts and may grow large." .

Then specialize only if you need distinct UI/paths:

    NomenReferenceLinksArtifact ⊑ ReferenceLinksArtifact

    KnopReferenceLinksArtifact ⊑ ReferenceLinksArtifact

    MeshReferenceLinksArtifact ⊑ ReferenceLinksArtifact

You can postpone specialization and just classify instances as ReferenceLinksArtifact and relate them to their host.
2) Link the artifact to its host container

<referenceLinksFor> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <ReferenceLinksArtifact> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference links for" ;
  rdfs:comment "Identifies the host resource whose reference links are curated in this artifact." .

<hasReferenceLinksArtifact> a owl:ObjectProperty, owl:InverseFunctionalProperty ;
  owl:inverseOf <referenceLinksFor> ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLinksArtifact> ;
  rdfs:label "has reference links artifact" ;
  rdfs:comment "Associates a resource with its curated reference links artifact." .

If you truly want “metadata slim,” you can still keep only one triple in the metadata artifact: the pointer to the reference-links artifact. The curated bulk lives elsewhere.
Apply your desired semantics cleanly
Mesh

You want:

    Mesh reference links about the mesh itself (“this mesh is great”).
    That’s fine: set referenceLinksFor to the Mesh. No implication that mesh reference links should cover everything in the mesh.

Nomen

You want:

    Nomen reference links are the semantic meaning when denotation is indirect (“I am Alice, see bio over there”).
    This is exactly what a dedicated ReferenceLinksArtifact gives you: Nomen stays a slim naming object, and the curated claim graph lives in its reference-links artifact.

Crucially: Nomen’s reference-links artifact can include:

    ReferenceLink about the designatum (Alice) even if designates is absent or underspecified.

    links from the designatum to Knop payload artifacts (“bio is over in that Knop”).

This preserves your open-world “judge the sources” stance.
Knop

You didn’t state it explicitly here, but the same mechanism works:

    Knop can have curated reference links (e.g., “this payload is derived from X”).
    Keep those out of KnopMetadata; put them in KnopReferenceLinksArtifact.

Why “keep ReferenceLinks away from metadata” is right (and the cost)

Benefits:

    metadata remains stable and small

    reference curation can be large, perspective-laden, and versioned separately

    you can shard reference-link artifacts without touching metadata

Costs you must accept:

    Consumers now must know: “to get meaning, follow hasReferenceLinksArtifact.”

    You’ll want indexing/discovery so reference links aren’t invisible.

The cost is manageable because you already have inventories.
One sharp critique: “metadata is really KnopMetadata, not denotee metadata”

This is correct for your system, but it implies a naming correction:

If KnopMetadataArtifact currently contains anything about the designatum (“Alice is a person”), that’s a category error under your stated philosophy. It should instead live in:

    the payload artifact (if it’s the primary data), or

    the Nomen reference-links artifact (if it’s curated/indirect).

So enforce this by convention:

    Metadata artifacts describe the container/artifact machinery.

    Payload artifacts describe the designatum domain.

    ReferenceLinks artifacts describe curated bridge claims.

That will reduce conceptual drift and prevent the “why are we calling this metadata?” confusion later.
Minimal concrete next step

    Add ReferenceLinksArtifact, hasReferenceLinksArtifact, referenceLinksFor to the ontology.

    Stop using hasReferenceLink directly on Mesh/Nomen/Knop in persisted data; instead, put ReferenceLink relators in the reference-links artifact and relate them with referenceLinkFor.

    Put only the pointer hasReferenceLinksArtifact in the slim metadata artifact (or in inventory).

If you paste your current artifact roster per Knop/Nomen/Mesh (which ones are “always present”), I can propose the default set that doesn’t bloat creation but keeps references discoverable.
You said:
A Knop could have multiple ReferenceLinkSet. Maybe they should be their own DigitalArtifacts in their own Knops. 

ReferenceLinks describe the denoted thing, the DigitalArtifact in the case of Knops and either DigitalArtifacts or things-in-world without direct denotation. 

I will ask you to update 1) A summary of decisions we've made in this session (and previous ones if you want); 2) A summary of the concept model via the core ontology, perhaps the config ontology, and the shacl. 3) updated ontology+shacl that reflects the latest decisions and conceptual model; Which order would you like to do it in?
ChatGPT said:

Preferred order: **(1) decisions → (2) concept model summary → (3) updated ontology + SHACL**. I’ll do it in that order below.

---

## 1) Decisions captured

### Ontology structure

* Keep a single **Semantic Flow Ontology** (no import-based “module ontologies”).
* Keep `SemanticFlowResource` (not yet renamed), but its comment must reflect **mesh constituency**, not “inside/outside a mesh.”

### Artifact stack and mutability

* Artifact stack is: `DigitalArtifact` → (`AbstractArtifact` → `ArtifactFlow` → States) → (`AbstractFile` → `LocatedFile`).
* `ArtifactFlow` relates **exactly one** `WorkingState` and **exactly one** `CurrentState`, plus **zero or more** `HistoricalState`.
* **Only WorkingState is mutable**; authoring changes occur only in the WorkingState’s authoring bytes (your “WorkingLocatedFile” constraint).
* `latestHistoricalState` is a **derived/operational pointer** and must not be treated as immutable content.

### Operational overlays and “anti-immutable”

* Overlays are **ON by default** via a companion **AntiMetadataArtifact** per `AbstractArtifact`.
* Anti-metadata exists to hold volatile/derived/system-maintained facts (e.g., `latestHistoricalState`, verification timestamps, operational caches).
* Stop-rule: Anti-metadata artifacts **do not** recursively require their own anti-artifacts (prevent infinite regress).

### ReferenceLinks and curation

* ReferenceLinks are **curated** and can grow large; keep them **out of slim structural metadata**.
* Reference links should be stored as one or more **ReferenceLinkSet** artifacts (RDF documents) associated with a container (Nomen/Knop/Mesh), rather than embedded into metadata artifacts.
* A Mesh’s ReferenceLinkSets are for the mesh itself (not “authoritative registry for everything under it”).
* A Nomen’s ReferenceLinkSets can carry the semantic “bridge” when the designatum is not directly denoted by a DigitalArtifact.

### Weave markers

* Do **not** inject NOOP weave markers into Knop metadata when nothing versioned.
* If weave-run bookkeeping is needed, keep it in **operational/overlay data**; the definitive “versioning happened” signal is minting a `HistoricalState`.

### Management space and multiple meshes

* Service can host multiple meshes; use **named graphs** to hold simultaneous mesh states/management info.
* Graph naming default: **graph name = mesh IRI**.
* Treat “mounting the same mesh more than once” as a **service policy** you generally forbid (even if test-mode might allow it later).

### Path token semantics

* “Path segment” definition stands.
* Anything that can encode an IRI with an optional trailing slash is a **relative path token**, not a segment datatype.
* `designatorPath` should explicitly allow optional trailing slash (directory-designator fidelity independent of filesystem).

---

## 2) Concept model summary (core ontology + SHACL scope)

### Core “mesh constituents”

* `Mesh` is an `ArtifactContainer` and a `SemanticFlowResource`.
* `Nomen` is an `ArtifactContainer` and a `SemanticFlowResource`:

  * carries `designatorPath` (relative token; optional trailing slash).
  * may optionally `designates` some resource, but can also be “meaningful only via reference links.”
* `Knop` is an `ArtifactContainer` and a `SemanticFlowResource`:

  * hosts one or more payload artifacts (primary artifacts).
  * also hosts supporting artifacts (metadata/inventory), but those remain **structural**, not curated.

### Artifact stack (versioning and bytes)

* `AbstractArtifact` (whole-identity) `hasFlow` `ArtifactFlow`.
* `ArtifactFlow` has:

  * `hasWorkingState` exactly 1
  * `hasCurrentState` exactly 1
  * `hasHistoricalState` 0..n
  * optional `latestHistoricalState` pointer (operational only).
* `ArtifactState` has 0..n `AbstractFile` variants.
* `AbstractFile` has 1..n `LocatedFile` realizations (bytes locations).
* SHACL should enforce:

  * working/current existence and disjointness (different nodes)
  * working authoring consistency (exactly one abstract file, exactly one located file).

### Overlays / anti-metadata

* Every `AbstractArtifact` has exactly one `AntiMetadataArtifact` by default (mesh-managed convention).
* Anti-metadata is where:

  * `latestHistoricalState`
  * verification timestamps (`lastVerifiedAt`)
  * operational caches / derived pointers
  * system-only bookkeeping
    live and are updated without polluting immutable historical snapshots.

### Reference links as curated sets (not slim metadata)

* `ReferenceLink` remains the relator:

  * `referenceLinkFor` (the thing the link is about)
  * `hasReferenceRole` (canonical/supplemental/deprecated)
  * `referenceTarget` and/or `referenceUriLiteral`
  * verification fields are operational (overlay).
* Reference links are stored in one or more **ReferenceLinkSet** artifacts:

  * a `ReferenceLinkSet` can be in its own Knop (your proposed “reference knops”).
  * containers (Nomen/Knop/Mesh) point to their ReferenceLinkSet(s), preserving transposability while keeping structural metadata slim.

---

## 3) Updated ontology + SHACL (targeted patch)

Below is a **patch-style** update: new/modified terms to merge into your current TTL, plus SHACL (JSON-LD) for key invariants.

### 3.1 Ontology patch (Turtle)

```turtle
#################################################################
# Updated comments
#################################################################

<DigitalArtifact> a rdfs:Class ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A multi-faceted digital artifact that can have content facets (diachronic or synchronic) and representation facets (bytes that may be retrievable from one or more locations). Used for documents, datasets, pages, binary assets, and their abstract/located variants." .

<SemanticFlowResource> a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  rdfs:comment "A primary constituent of a Mesh modeled using Semantic Flow conventions (Mesh, Nomen, Knop, and mesh-hosted artifacts such as payload, metadata, inventory, and overlay artifacts)." .

<designatorPath> a owl:DatatypeProperty ;
  rdfs:domain <Nomen> ;
  rdfs:range xsd:string ;
  rdfs:label "designator path" ;
  rdfs:comment "Relative path token for the Nomen's designator, interpreted relative to the mesh base. Use no leading slash. The empty string denotes the mesh root designator. A trailing slash MAY be included to explicitly designate a slash-IRI." .

<latestHistoricalState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasHistoricalState> ;
  rdf:type owl:FunctionalProperty ;
  rdfs:domain <ArtifactFlow> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "latest historical state" ;
  rdfs:comment "Operational convenience pointer to the most recently published HistoricalState. SHOULD be maintained only in overlay/anti-metadata data and MUST NOT be asserted into immutable historical snapshots/distributions." .

#################################################################
# Anti-metadata overlay artifacts (ON by default)
#################################################################

<AntiMetadataArtifact> a rdfs:Class ;
  rdfs:subClassOf <AbstractArtifact>, <RdfDocument>, <SemanticFlowResource> ;
  rdfs:label "Anti-metadata artifact" ;
  rdfs:comment "Companion artifact for operational/derived/non-immutable statements about another AbstractArtifact (e.g., latestHistoricalState, verification timestamps, caches). Excluded from immutable HistoricalState snapshots/distributions by convention." .

<hasAntiMetadataArtifact> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <AbstractArtifact> ;
  rdfs:range <AntiMetadataArtifact> ;
  owl:inverseOf <antiMetadataFor> ;
  rdfs:label "has anti-metadata artifact" ;
  rdfs:comment "Associates an AbstractArtifact with its companion AntiMetadataArtifact (overlay). Mesh-managed convention: exactly one." .

<antiMetadataFor> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:domain <AntiMetadataArtifact> ;
  rdfs:range <AbstractArtifact> ;
  rdfs:label "anti-metadata for" ;
  rdfs:comment "Identifies the AbstractArtifact that this AntiMetadataArtifact is about." .

#################################################################
# Reference links as curated sets (kept out of slim metadata)
#################################################################

<ReferenceLinkSet> a rdfs:Class ;
  rdfs:subClassOf <RdfDocument>, <DigitalArtifact>, <SemanticFlowResource> ;
  rdfs:label "Reference link set" ;
  rdfs:comment "A curated RDF document that contains one or more ReferenceLink relators. ReferenceLinkSets may be large, may evolve independently, and are intended to be stored separately from slim structural metadata." .

<hasReferenceLinkSet> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ReferenceLinkSet> ;
  owl:inverseOf <referenceLinkSetFor> ;
  rdfs:label "has reference link set" ;
  rdfs:comment "Associates a resource (Mesh, Nomen, Knop, or any resource) with a curated ReferenceLinkSet that contains ReferenceLinks relevant to it." .

<referenceLinkSetFor> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLinkSet> ;
  rdfs:range rdfs:Resource ;
  owl:inverseOf <hasReferenceLinkSet> ;
  rdfs:label "reference link set for" ;
  rdfs:comment "Identifies the resource that a ReferenceLinkSet is curated for." .

#################################################################
# Service-level operational mounting (policy: usually one mount)
#################################################################

<mountedMeshName> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:string ;
  rdfs:label "mounted mesh name" ;
  rdfs:comment "Operational mount identifier for routing within a hosting service instance. Host-specific operational state; should be stored in overlay/anti-metadata management data, not immutable snapshots." .

<preferredManagementHandleSegment> a owl:DatatypeProperty, owl:FunctionalProperty ;
  rdfs:domain <Mesh> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred management handle segment" ;
  rdfs:comment "Preferred reserved path segment token used to surface the mesh management handle within the mesh base (e.g., '_mesh'). Must be a single path segment token (no slashes)." .
```

Notes on scope (so you don’t accidentally misinterpret your own model later):

* `ReferenceLinkSet` is defined as a content-kind. In practice you may implement each set as a payload artifact in its own Knop, but the ontology doesn’t force that.
* If you want to force “ReferenceLinkSet must be an AbstractArtifact with a Flow,” change `ReferenceLinkSet ⊑ AbstractArtifact` instead of just `DigitalArtifact`. Right now it’s deliberately flexible.

---

### 3.2 SHACL (JSON-LD) for the invariants you’ve committed to

```json
{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-shacl/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "sh": "http://www.w3.org/ns/shacl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
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
      "@id": "ArtifactFlowStateCardinalityShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "ArtifactFlow must have exactly one WorkingState and one CurrentState",
      "sh:targetClass": "sflo:ArtifactFlow",
      "sh:property": [
        {
          "sh:path": "sflo:hasWorkingState",
          "sh:class": "sflo:WorkingState",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        },
        {
          "sh:path": "sflo:hasCurrentState",
          "sh:class": "sflo:CurrentState",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        },
        {
          "sh:path": "sflo:hasHistoricalState",
          "sh:class": "sflo:HistoricalState",
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "ArtifactFlowWorkingAndCurrentMustBeDistinctShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "ArtifactFlow working and current states must be distinct nodes",
      "sh:targetClass": "sflo:ArtifactFlow",
      "sh:property": [
        {
          "sh:path": "sflo:hasWorkingState",
          "sh:disjoint": "sflo:hasCurrentState",
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "WorkingStateSingleAuthoringFileShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "WorkingState has exactly one AbstractFile and exactly one LocatedFile",
      "rdfs:comment": "Authoring consistency: WorkingState is constrained to exactly one AbstractFile; that AbstractFile is constrained to exactly one LocatedFile.",
      "sh:targetClass": "sflo:WorkingState",
      "sh:property": [
        {
          "sh:path": "sflo:hasAbstractFile",
          "sh:class": "sflo:AbstractFile",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "WorkingAbstractFileSingleLocatedFileShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractFile used for WorkingState must have exactly one LocatedFile",
      "sh:targetSubjectsOf": "sflo:hasLocatedFile",
      "sh:property": [
        {
          "sh:path": "sflo:hasLocatedFile",
          "sh:class": "sflo:LocatedFile",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "AntiMetadataArtifactLinkageShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AntiMetadataArtifact must point to exactly one AbstractArtifact",
      "sh:targetClass": "sflo:AntiMetadataArtifact",
      "sh:property": [
        {
          "sh:path": "sflo:antiMetadataFor",
          "sh:class": "sflo:AbstractArtifact",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        },
        {
          "sh:path": "sflo:hasAntiMetadataArtifact",
          "sh:maxCount": 0,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "AbstractArtifactHasAntiMetadataArtifactShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "AbstractArtifact has exactly one AntiMetadataArtifact (mesh-managed convention)",
      "sh:targetClass": "sflo:AbstractArtifact",
      "sh:property": [
        {
          "sh:path": "sflo:hasAntiMetadataArtifact",
          "sh:class": "sflo:AntiMetadataArtifact",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "PayloadSlugTokenShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "payloadSlug must be a single non-reserved path segment token",
      "sh:targetSubjectsOf": "sflo:payloadSlug",
      "sh:property": [
        {
          "sh:path": "sflo:payloadSlug",
          "sh:datatype": "xsd:string",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:pattern": "^(?!_)(?!index\\.html$)[^/]+$",
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "DesignatorPathTokenShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "designatorPath is a relative path token (no leading slash; trailing slash allowed)",
      "sh:targetSubjectsOf": "sflo:designatorPath",
      "sh:property": [
        {
          "sh:path": "sflo:designatorPath",
          "sh:datatype": "xsd:string",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:pattern": "^(?:[^/]+/)*[^/]+/?$|^$",
          "sh:severity": "sh:Violation"
        }
      ]
    },
    {
      "@id": "PreferredManagementHandleSegmentShape",
      "@type": "sh:NodeShape",
      "rdfs:label": "preferredManagementHandleSegment must be a single path segment token (no slashes)",
      "sh:targetSubjectsOf": "sflo:preferredManagementHandleSegment",
      "sh:property": [
        {
          "sh:path": "sflo:preferredManagementHandleSegment",
          "sh:datatype": "xsd:string",
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:pattern": "^[^/]+$",
          "sh:severity": "sh:Violation"
        }
      ]
    }
  ]
}
```

Two deliberately missing constraints (because they require SHACL-SPARQL / dataset-level graph awareness):

* “`latestHistoricalState` must appear only in anti-metadata graphs.”
* “ReferenceLinkSets must be stored outside metadata artifacts.”

Those are enforceable, but not with pure core SHACL if you need to inspect named-graph placement.

---

If you want the *next* increment after this: decide whether `ReferenceLinkSet` should be forced to be an `AbstractArtifact` (Flow/States/Files), or whether it can remain a “content kind” that can be attached at any artifact layer. That choice drives whether “ReferenceLinkSet in its own Knop” is a convention or a hard invariant.

Why the term "overlay artifacts".

This morning I"m thinking that ReferenceLink are transitive, so if you specify something that is itself a DigitalArtifact, any ReferenceLinks present in that Artifact can also be resolved and used. Probably a maxReferenceLinkDepth property would be useful to prevent runaway chains. The benefit is you wouldn't need ReferenceLinkSets. I do still think they are a separate artifact from metadata. 

Regarding AntiMetadataArtifact, it would probably be named Anti-immutableMetadataArtifact. But was considering just using OperationalMetadatArtifact. We could just keep latestHistoricalState for each DigitalArtifact in mesh (operational) metadata to avoid a separate Artifact for each Knop. We'll have to be careful to update previousHistoricalState during weave, and should probably change the domain to ArtifactState so we can use it in CurrentState and WorkingState.
We almost need a subclass of DigitalArtifact like MeshDigitalArtifact or (uglier but clearer) SFDigitalArtifact  for things that are SF convention-compliant. It's like a marker class so we can know whether the resolution process can be run on the thing. It would replace using <AbstractArtifact>  (e.g. as subclass of PayloadArtifact) which is justified because AbstractArtifact is a general purpose class AND its only a facet of a (Mesh)DigitalArtifact.

You can target any facet of a MeshDigitalArtifact, from its AbstractArtifact to its LocatedFiles and still resolve to the appropriate LocatedFile when you need to retrieve bytes. But what should be the default level when you just want the current bytes? I'm leaning towards specifying a LocatedFile under _current, to minimize resolution overhead.
The problem with the direct pointer is it still requires a metadata lookup. For systems that don't have the MeshManagedDigitalArtifact lookup logic, they can't resolve bytes. I'd love to not have a single-syntax default... people should be able to reference <PayloadFileSlug>.trig or metadata.jsonld or alice.jpg and it works just as well as specifying the AbstractArtifact IRI, retrieving current bytes for naive systems and allowing resolution to the client's preferred distribution format if they support the resolution logic. On the other hand, there's the issue of every LocatedFile being its own DigitalArtifact (separate from its AbstractFile DigitalArtifact, which is separate from its ArtifactState, etc.) We don't have a way to disambiguate if you start at the leaf, so maybe we have to say "use the most-general/broadest artifact for the meaning you intend". I think that's what we have to do. Also we probably need a facetFor property linking each layer, so we know that, e.g., a particular LocatedFile is a facet of its AbstractFile DigitalArtifact, its State DigitalArtifact, its Flow DigitalArtifact, and its AbstractFile DigitalArtifact. I probably is also a facet of itself-as-digital-Artifact. Any reason to specialize facetFor?

I think we should get rid of <SemanticFlowResource>. It's kinda vague anyhow, and it's only used for SF-native ArtifactContainers and SF-native Artifacts. We replace it with SfArtifactContainer and SfArtifact. And then have subclasses of SfArtifact: MeshArtifact, NomenArtifact, and KnopArtifact. Then, for example, PayloadArtifact would just be subClassOf KnopArtifact instead of rdfs:subClassOf <AbstractArtifact>, <SemanticFlowResource>. 


Yes, if a leaf IRI always means "bytes-at-this-location", there's no ambiguity. ie., we can use the leaf IRI to select a single DigitalArtifact. 

Talk to me about PayloadArtifact ⊑ AbstractArtifact, KnopHostedArtifact vs PayloadArtifact ⊑ DigitalArtifact. Do we keep the concept general? Even if we don't, I think 

One uneasiness I have is about the Knop/Nomen split with regard to ReferenceLinks. If Nomen can denote SfArtifacts, we effectively have two IRIs for that artifact. I think they would be owl:sameAs. e.g. if "http://example.com/alice/bio" denoted "http://example.com/_knops/<uuid>/alice-bio" they are basically two identifiers for the same thing. So when computing ReferenceLinks (and generating ResourcePages), you have to take sameAs into account?

For PayloadArtifacts, I was trying to preserve it for external (non-SF) use. Because we can get whether it's a Knop-hosted PayloadArtifact from SfArtifact/KnopHostedArtifact. i.e., no need to invent VersionedPayloadArtifact, because that's same as a KnopHostedArtifact + PayloadArtifact.

For Nomen/Knop split, the IRIs are denotationally identical. The reason to have two is: Nomen can denote any digital artifact, so why not SfDigitalArtifacts? And also, the Knop IRIs are ugly and long. We need humane IRIs, especially for knop-hosted payload AbstractArtifacts. I know sameAs is scary, but this is a legitimate use case. 

Keep in mind that a Nomen can denote any DigitalArtifact, so could denote a metadata LocatedFile as easily as a Payload AbstractArtifact. Probably we don't even need <designates> -- unless you can convince me sameAs really is dangerous. identity is a big part of what Nomens are for. 
I'm thinking about reintegrating Knots and Nomen. I wanted to put all the Knots in the same place, so that references wouldn't have to take account of the tree containment. But now that meshes are fixed and we've deprioritized transposability and composability, I'm not sure of the value there. I'm also unclear about why the UUID is needed for a Knob if we're going to use a humane identifier under the Knob that can't change. We might as well just drop the UUID. Then every mesh path would have a Nomen, and optionally, it could have a Knob.
Yes, I'm using voice transcription, so I always mean K-N-O-P, never knob or not, period. A mesh path has its denotation, if any, established in the nomen. Theoretically, the mesh path could denote any part of the knot, or, and this is just for nomen which denote digital artifacts, any part except the located file, a located file, but most likely would denote the abstract artifact, period. Aren't gnomon or knobs, they are just associated with a gnomon, and optionally, additionally, a knob. I'm fine committing to path as identity. A base plus a path is always unique. And I think renaming, minting a new identity is great. You shouldn't be able to rename something without connoting that it is different.
The UUID basically becomes a lineage marker. If it's preserved, it actually doesn't establish identity, it just establishes, at one point, I was this other thing. If it never gets updated, it's like the genesis marker.
Every consumer who cares about semantics will already have to do a nomen lookup to discover what the IRI means. 

alice/ denotes Alice (let's change our property from designates to denotes; denotes is more conventional and is exactly what the IRI does); 
alice/_nomen/ (or alice/_knop) refers to the Nomen/KnopArtifact attached to the alice/ designatorPath (or maybe we call it meshPath or ???) as an AbstractArtifact- so it refers to to a NomenArtifact, but also marks the alice/ IRI.
alice/bio/ denotes a DigitalArtifact (at the AbstractArtifact level) so can directly contain a _flow folder (and the flow's subfolder). 
perhaps alice/bio/alice-bio.ttl denotes the WorkingLocatedFile. We can get rid of the WorkingState and its AbstractFiles because there's only ever one. Maybe we merge Working/Current (or current just becomes the latestHistoricalState, requires a lookup, but it might be worth it)... and then all states are HistoricalStates, we don't have 3 duplicate copies.
alice/bio/2026-02-02_1423_08/ refers to a payload HistoricalState
alice/bio/_metadata, alice/bio/_ref, alice/bio/_inventory, etc. for supporting files, and they repeat the pattern: alice/bio/_metadata/metadata.trig is the WorkingLocatedFile, etc

This is closer to what I had earlier, but the Knop maybe doesn't have its own semantic representation. It's just what you call all the stuff attached to an IRI. 
IRIs do denote! They denote something, and meaning is either attached by convention/reference or (in the case where an IRI denotes a digital thing), by using sflo:denotes. Or the 3rd case is there's some other IRI out there that already denotes something, and then you can say alice/ denotes http://example.com/alice. The _nomen denotes itself as a DigitalArtifact that contains denotation statements. 
designatorPath is probably wrong now, better to use identifierPath or meshPath. 

Yes, we don't need denotes at all, unless you want to denote some OTHER DigitalArtifact than the one contained directly, OR if it it co-denotes something else denoted by an IRI somewhere. If it denotes something in the real world, no direct denotation. There is no self-denotation unless it has a payload. Having a default, non-semantically materialized referent seems weak to me. Maybe in the NomenArtifact we put stuff (all optional) like its type. If its type is a SfDigitalArtifact, there must be a flow. Which reminds me, my mistake earlier:

alice/bio/_flow/2026-02-02_1423_08/ would denote a HistoricalSlice. Or perhaps "_payload-flow". Maybe to keep things compartmentalized, it would all be under _knop, which gives you a handle to refer to the Knop, which as usual is a collection of support SupportingDigitalArtifacts and an optional PayloadArtifact.
#1 We might need a string path, because it's the only way to semantically encode "this specific identifier". If we encode it as a node, e.g. it's NomenArtifact nomenFor <alice>, it's a Nomen for the person alice. She might have more than one. 

#2 I kinda like denoting itself explicitly, otherwise (open-world assumption) we just might assume that the denotation statement is missing. But see #4, I think we have a non-redundant candidate. The only time there shouldn't be a denotes statement in the NomenArtifact is when there's no co-denotation and the denoted isn't a DigitalArtifact (or is an unknown/external DigitalArtifact.

#3 I think a Knop is not an Artifact. It's an ArtifactContainer. Nomen is no longer an ArtifactContainer with its own metadata artifact, it's a NomenArtifact, contained in the Knop. I guess Mesh is still an ArtifactContainer. It's handle is at the root meshPath, "_mesh". 

#4 Instead of "alice/bio/_flow/2026-02-02_1423_08" for the payload state, we would have "alice/bio/_knop/_payload/_flow/2026-02-02_1423_08" -- and for payload-bearing meshPaths, "alice/bio/_knop/_payload" denotes the same thing as "alice/bio" AHA! That's what the Nomen can say is the denotee!

We don't need the unnecessary "_history" segment, and _working is no longer part of the flow. 

#5 If we really need WorkingState, we could keep an IRI for it at ".../_flow/_working" -- but I still want to keep the LocatedFile (at least for payloads) above the _knop. We could maybe keep _meta.jsonld, _ref.jsonld, _inventory.jsonld and _nomen.jsonld there too?

#6. If you don't want to make a semantic commitment, don't make any claims. But the reason we can't directly denote something in the real world is because there's no way to directly specify a real-world thing with an IRI. (unless it's already been denoted with some other IRI somewhere else, i.e. the co-denotation use case). If we store type, name, label, comments, etc. directly in the NomenArtifact, it fills the same role we used to have with ReferenceFlows: a default/canonical way to make statements about the denoted. (We ended up externalizing Referencing, to standalone SfDigitalArtifacts. I have a guess why, do you?)

#8 The NomenArtifact is now contained in the Knop. So EVERY "meshPath" has a Knop, and I think  "alice/" -- we need a better term than designator or meshPath, and we don't refer to it as a node, only a string. We could use a relator to capture it perhaps. "alice/" (I miss those trailing slashes! Damn you SPARQL!) We need a way to exclude all the stuff in a mesh other than the things with Knops. meshPath sounds like any IRI in the mesh, including "system IRIs" like <alice/_knop/_meta>. Maybe designator is okay, but to me it has a connotation, of "you have to designate something for some role". We're just naming things. Maybe "Nomen" is the term for IRIs that have associated Knops, and we remove the "Nomen" term from the ontology, because we don't refer to Nomen as instances of classes. And we have a datatype property "nomen" (instead of designator)

Hah, stick all that in your pipe and smoke it.
"If <alice/> is the identifier IRI (the mesh entry), then it already encodes “this specific identifier” and you don’t need a string." Not when you're using <alice/> as an IRI in RDF data. Believe me, we've been over this a hundred times. My single-referent principle hinges on this. <alice/> only denotes one thing: Alice the person. It does not denote an identifier IRI. So if we want to talk about "alice" the identifier, we have to talk about "alice" with a string, not <alice>. We can get around this by using a handle, like <alice/_nomen> if somewhere we say (and now I'm thinking best to put it in KnopMetadataArtifact):

<alice/_nomen> nomen "alice"

We maybe don't need a NomenArtifact. The reference stuff (about the referent) can go in the revived ReferenceArtifact and the mesh-stuff (about the identifier and the knop and the supporting artifacts) can go in the KnopMetadataArtifact.
r.e.#5, maybe we keep alice/bio/alice-bio.ttl and then put all the other stuff (WorkingLocatedFiles and _nomen-handle) below _knop, e.g. alice/_knop/_nomen-handle. Nice and clean. Still lets naive systems fetch bytes. And it emphasizes that the Knop ties all this together, the name, the name handle, the payload, the references, the metadata. 

Yes, they are leaves and my nomen can't denote leaves. 

Regarding "Critique of your last line" -- I was intending _nomen-handle to just be an empty segment, not an Artifact or anything that contained RDF data. It would have a ResourcePage and nothing else. The single entrypoint would be the KnopMetadataArtifact (or perhaps KnopInventoryArtifact), whose working version only changes "on weave". 
Regarding F), I would love to make Nomen identifiers for LocatedFiles. You told me I couldn't because, under Github with no redirects, I can't return the same bytes for both. I suppose we could copy the working file to two places on weave. 

Definitely want Flexible mode. I'm not even sure what's in Metadata any more if we have KnopInventoryArtifacat, but I think the idea was to keep Metadata thin, maybe just repeating the latest HistoricalVersion, pointing to the WorkingArtifacts, and maybe some Knop (not referent) provenance. Although the provenance could live primarily in the inventory. 
What's a better propertyname, "handleFor" or "nomen"? I like Nomen because it underscores the concept. 

Having a separate, central location for all the Knops might be worth it if that allows file-locating nomen. It hampers composability/transposability. Also makes it more of a pain to cross-reference the payload historicalstates, metadata, etc, which now lives far away from the nomen.  I don't know if we need separate _nomen and _knops directory, remember now the NomenMetadataArtifact just lives under the Knop. 

So, anyhow, for a new path structure, maybe the mesh sits adjacent to the Nomina, a kind of shadow namespace. 

_mesh/ -- denotes the mesh
_mesh/_meta and _mesh/_inventory -- MeshMetadataArtifact/MeshInventoryArtifact -- denotes the Artifacts that contains info about the mesh
_mesh/_knops/_root/_nomen-handle denotes the nomen that's associated with the namespace root (if any) and the "sflo:nomen" statement
_mesh/_knops/alice --  denotes the knop that's associated with a "alice" Nomen (Nomen=base-relative identifier)
_mesh/_knops/alice/_ref, _meta, _inventory, supporting Artifacts that are used for, among other things, generating the alice/index.html ResourcePage. 
_mesh/_knops/alice/_nomen-handle would denote the nomen/IRI "as an identifier" 


<_knops/alice/bio/_payload/_flow/2026-02-03-1234_20> is a payload artifact HistoricalState. I guess we keep the WorkingLocatedFile for DigitalArtifacts in the Nomen namespace instead

 


"you will eventually need both a token and an IRI anyway, and the names should make the distinction obvious." -- the IRI will be the nomen-handle, we won't use the nomen IRI directly. What I'm angling for is that the thing we've called the meshPath or the designator, we should call the nomen. The nomen-handle is the IRI that refers to the nomen-as-nomen. 

I was thinking that Nomen-handles are involve in a NomenDefinition relator that ties the IRI to (optionally) the thing that it denotes. But maybe a simple binary relationship is fine. Nomen (identified with the handle) -> nomenFor/namespacePath -> "identifier-string"  

One thing I don't like about this new "mesh namespace" is that we have to keep it mirroring (below _mesh/_knop) the "identifier/source namespace". One thing that I love is that you can drop a mesh next to any arbitrary file tree (even if it mixes "things intended to name" with other things, like source code. You could even theoretically keep it in its own repo so as not to pollute the source namespace. Of course then there would be different bases between the two. I guess it just makes sense to keep them in the same repo.

_mesh/_knops/alice/_nomen-handle/ statements:

<_mesh/_knops/alice/_nomen-handle/> a sflo:Nomen ; 
    nomenPath "alice" . 

Any other benefits or "sharp edges" separating the artifact storage hierarchy from the identifiers/nomen hierarchy?


 


You're right, when we refer to paths as string, we should include the leading slash if it's root-relative, and perhaps the trailing slash if it's not a file? so:

    nomenPath "/alice" .

Reminder: we decided to drop the trailing  slash in node identifiers/object props, because they don't place nice with SPARQL. It's not a token, it's a path, usually root-relative, so we at least need the initial slash. 
I think nomenPath is authoritative canonical identifier path. 

For SPARQL, is it possible and common to use a BASE? Or really for instance/designator IRI you use full IRIs? 

One thing to think about: Does every payload WorkingLocatedFile live in the Nomen namespace? Or is it just an option? If an option, it seems like an alias. Either way, there's a possibility of collision with a filename-shaped nomen. 

I really don't love splitting into a Knop namespace and a Nomen namespace. I really want to hear your opinion on it (i.e., the shadow "artifact namespace"), because it's a huge change. Can't even remember why we unified again other than clarity/simplicity. 
What do you think about the idea of putting all semantic data (knops, artifacts, and the nomen-handles) in the semantic mesh, and making the mesh adjacent to the nomenspace? So other than (optional) ResourceFiles for the nomen, a semantic mesh would be a completely separate overlay... easily cleaner, layerable anywhere.
Re #1: we can have separate meshBase and a nomenBase. The ideal situation is that they are the same, so we can use <iri> references anywhere. So nomenPath "/alice" would refer to (or translate to?) ${nomenBase}/alice. 

Tell me more about sflo:nomenspaceRootPath "/" -- Wouldn't it always be that? Or you're thinking you can juggle off parts of the base into rootPath? Oh, I see, you're talking about compared to the repo root. Do we need that info? Publish IRI is the important thing, handled in the base, right? Can you some deep analysis of why we'd need to specify the filesystem/repo location of the Nomen space and the SemanticMesh relative to each other? I guess just for ResourcePage generation. Is there any reasone the _mesh folder containing the semantic mesh would always be contained in the (filesystem) nomenRoot?

If you have ResourcePages everywhere, the cleanliness benefit kinda evaporates. But there's no getting around it other than turning resourcepages off, which is fine. You want dereferenceable identifiers, turn 'em on. You want a pristine nomenspace, turn 'em off.

We'll call them "nomen tree" in general, and "identifier tree" when paired with a base? And then "SemanticMesh", which always lives at the nomen root, under "_sf-mesh"
Regarding #1, I was thinking that you would strip off the leading slash in nomenPath. Basically the leading slash is there just to accentuate "root relative". Maybe you would want to specify a relative path sometime. If not, I guess we can just leave off the leading slash.

#2 & #3: I don't see an obvious need for separate "published bases" for the nomen tree and the semantic mesh. Maybe you want to use someone else's identifier and attach your own reference data? Or you want to create a flow of historical states for an artifact published elsewhere? But it seems like you can have a mesh without any corresponding nomen... just use the mesh IRIs as identifiers. It seems a little dangerous to allow people to "take over" the meaning of someone else's identifiers. But nothing preventing that on the Semantic Web in general, and you can accomplish that with a mesh that has no nomen of its own.

Maybe "_s-mesh" is better than "_sf-mesh". 

"nomen tree" or "nomen space"? -- I think nomen tree captures leaf nodes better, and not just metaphorically. ;)

By dereferencability, I mean "if you but the IRI into a web browser you either get the bytes (if it denotes a DigitalArtifact) or a ResourcePage about the IRI (its denotation, reference material, associated inventory)" If ResourcePages don't live directly under the designator/nomen, you don't have the dereferencability I'm talking about. 


maybe we call it nomenAbsolutePath to make it clearer.

What makes something a Nomen? I think the defining characteristic is that it has a Knop (and a Nomen handle): you can get metadata about the Nomen, including denotation (if it exists); 

I'm now thinking "nomen space" or "nomenspace" is better, because it supports the idea of sparse nomen, scattered throughout an abstract namespace. nomen are realizations of an abstract namespace in a way: "this is something I want to use as an identifier" and the way you mark it as a Nomen is you give it a corresponding Knop in the same namespace. If you just want to say something about someone elses Nomen, you can say it arbitrary RDF data. You don't need a Knop. But you can have a Knop if you want -- it's just for the DigitalArtifact about someone else's Nomen. 

By "non-root" publish directories, (like docs?) you just have to have the nomen root and the corresponding mesh in the docs folder? Syntactically, the _s-mesh and the nomen space should share a base because they are bound to each other. You can't have a nomen space without a support mesh. 

Yes, for #4, I meant LocatedFile. #5, agree with what you've said. But what I was getting at was more "does every (payload?) DigitalArtifact keep its bytes under a Nomen." The answer is clearly no IMO. If the Nomen is created simultaneously with the Knop, it can start under the Nomen. If not, it has to start under the Knop (maybe at the top of the Knop, likely with other supporting WorkingDigitalArtifacts). Question is: can it start in the Knop  and move to the Nomen. It'd be nice, but it could break some clients expecting the IRI for a WorkingDigitalArtifacts to always return the bytes.

Regarding mirroring, it's unnecessary. You mint a Nomen by creating Knop. Was considering that to avoid collisions, you require nomenless knops to use UUIDs. Or we can just say, if the Knop already exists that corresponds to a nomen path, it must become the knop for that nomen.
Regarding your Important nuance. Actually, I'm trying to establish the designator IRI is the nomen (i.e., a unique name, aka identifier, that has a corresponding Knop), but it is represented in RDF with its handle. So "http://examples.com/alice" is a nomen in prose, but <http://examples.com/_sf-mesh/alice/_nomen-handle> denote the nomen in RDF. 

Remind me what <http://examples.com/_sf-mesh/alice/> denotes? If it's not spoken for, it could denote the nomen in RDF, although its similarity to the designator IRI might be confusing.
#5, For now, let's not support moving a WorkingFile location from Knop to Nomen. If you want a Nomen to denote a WorkingLocatedFile, you have to create it fresh. 

r.e. #6, nomenPath is way better.  I was imagining just _s-mesh/alice (no _knops or "by-path"). I don't think encoding is necessary. Oh, yeah, that's what <https://example.com/_s-mesh/alice> denotes: the knop. good. Mirroring is necessary for existant nomen. I'm just saying the mesh doesn't have to mirror the full designator namespace or whatever we call it. 

