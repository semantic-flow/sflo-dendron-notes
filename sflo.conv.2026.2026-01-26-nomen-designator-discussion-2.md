---
id: k75xfqc4rezk93cmbl3px48
title: 2026 01 2 Nomen Designator Discussion 2
desc: ''
updated: 1769478266200
created: 1769478185591
---

Semantic Flow Shacl (revised) — Json-ld
OOo, another idea I'd like you opinion on. We should be able to attach ReferenceLinks to Meshes and Knops too. 
Semantic Flow Shacl (revised) — Json-ld
I thought we had a "URL" version of referenceTarget, but I don't see it anymore.
Semantic Flow Ontology (revised) — Turtle
<meshPath/> was also missing. Was thinking it should also apply to more than just the Nomen, e.g. all ArtifactHost. 
Semantic Flow Ontology (revised) — Turtle
Yeah, lets keep meshPath just for Nomen.

Where is the property that specifies a Mesh's base?
Semantic Flow Ontology (revised) — Turtle
do we need a "containsKnop" property, domain: Mesh -- at least for inventory?
Semantic Flow Ontology (revised) — Turtle
This should have domain PayloadArtifact:


<hasPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <DigitalArtifact/> ;
  rdfs:label "has payload" ;
  rdfs:comment "Links a Knop to its primary payload artifact identity (either a FlowArtifact-based payload or a SimpleArtifact-based payload)." .

?
Semantic Flow Ontology (revised) — Turtle
Ooo, I meant range PayloadArtifact, but that's wrong because of inheritance right?
Semantic Flow Ontology (revised) — Turtle
I think I know why I was confused. PayloadArtifact should be called SimplePayload, and probably PayloadFlow should be FlowPayload
Semantic Flow Ontology (revised) — Turtle
What about a "preferredSlug" property, used by any artifact level? 

And is specifying which version of semantic flow you are using as simple as your base reference? You almost should be able to specify the general base, and then which specific ontology version separately. If you hold yourslef to a SF namespace that only evolves forward. (i.e., perpetual non-breaking changes)  
Semantic Flow Ontology (revised) — Turtle
preferred slug will let a root nomen hint at what it's about with a human-readable name. It allows you to override the default filenaming. We always want _meta/_working/meta.jsonld/meta.jsonld to be the canonical metadata location for a ArtifactHost, but other than that... maybe you prefer a different filename that your immediate-parent and maybe a mashup of the full path. I like your suggestion about rdfs:label, but this is more specific. ideally a snake-case slug that gives any file a relatable imprimateur.
Semantic Flow Ontology (revised) — Turtle
yes on the mesh conformance block. BTW, I will be sad when this context goes away. 

In a separate note, I wrote "you could simplify local by appending the full original IRI to the http://localhost on base processing. Too bad about the escaping.

you should find a special status for meshes that fail forward, i.e., you never change the meaning of any term to something different, and there's always a match between the term you are using and the latest working version of the ontology. 
"

Can you include a colon in a filename and github pages url.
Semantic Flow Ontology (revised) — Turtle
sorry, meant kabob-case. domain 

If you just dropped or substituted a slash for all colons, you could get away with a fairly naive way to take local referential control over another domain's identifiers.
Semantic Flow Ontology (revised) — Turtle
We need a naming convention for appending other Namespaces into your namespace. We could just make <https://semantic-flow.github.io/semantic-flow-ontology/> into 

https/semantic-flow.github.io/semantic-flow-ontology/

and 

https#8080/semantic-flow.github.io/semantic-flow-ontology/
Semantic Flow Ontology (revised) — Turtle
how about just /_ext/... without the ns. it's hard to imagine what alternatives there are for the "ns"'s purpose. Ideally it's internationalizable. 
Semantic Flow Ontology (revised) — Turtle
yeah, I'd just rather be able to support a full encoding. I guess escaping is the way to go.
Semantic Flow Ontology (revised) — Turtle
Do you think it's worth adding/renaming all the class documentation and probably a few of the properties? The dendron notes are badly outdated, so probably have to be removed if not updated. 

Can you give me the turtle for preferredSlug


Semantic Flow Ontology (revised) — Turtle
Can you update the list of pages we've had running, and select the most important properties to include. 

Also, right now we have hasKnopComponent but not similar properties for the other two abstract ArtifactHosts. Can you make the decisions about has* that we'll need to specify the mesh structure semantically? i.e., do we need hasNomenComponent, hasFlowArtifact, hasSimpleArtifact, hasWorkingSlice, etc. 

Also, for preferred slug, do we make it all about the filename and not about the path?
Semantic Flow Ontology (revised) — Turtle
should LocatedFile be a class.Realization as well? For when (as could often happen) you realize something with a locatedFile? i.e., there might not be a known AbstractFile.

i still haven't decided about meshPath. If we align it with IRIs for MeshResources, it doesn't start with a slash. It does feel slightly redundant. But we should probably write a "concept" article on the duality of a mesh: It's a semantic entity that serializes to a filesystem that mirrors its structure.

I did add these already:

{
      "@id": "target/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "rdfs:Resource",
      "rdfs:label": "target",
      "rdfs:comment": "If present, the target is expected to denote a reference-data source."
    },
    {
      "@id": "targetUriLiteral/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "xsd:anyURI",
      "rdfs:label": "target URI literal",
      "rdfs:comment": "If only this is present, it is expected to locate retrievable content (without assuming denotation)."
    }
Semantic Flow Ontology (revised) — Turtle
I think we'll do A), and hasByteEndpoint might be good because it accomodates streams. 

we should keep referenceTarget, and then referenceUriLiteral. It's longer, but underscores that these are only for referenceLinks.

I realize that I made some changes to my working copy of the ontology that aren't reflected in your canvas. Can you help me reconcile them? Here's my ontology working copy:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
    "label": "rdfs:label",
    "comment": "rdfs:comment",
    "rdfs:subClassOf": {
      "@type": "@id"
    },
    "rdfs:subPropertyOf": {
      "@type": "@id"
    },
    "rdfs:domain": {
      "@type": "@id"
    },
    "rdfs:range": {
      "@type": "@id"
    },
    "owl:inverseOf": {
      "@type": "@id"
    }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes and their resources.",
      "dcterms:creator": {
        "@id": "https://djradon.github.io/ns/djradon/"
      },
      "owl:versionInfo": "0.0.1",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/semantic-flow-ontology/_knops/_payload/_working/sflo.ttl/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/semantic-flow-ontology/"
    },
    {
      "@id": "SemanticFlowResource/",
      "@type": "rdfs:Class",
      "rdfs:label": "Semantic Flow resource",
      "rdfs:comment": "Any resource participating in Semantic Flow conventions."
    },
    {
      "@id": "Mesh/",
      "@type": "rdfs:Class",
      "rdfs:label": "Mesh",
      "rdfs:comment": "A servable filesystem region that contains SemanticFlowResources and, optionally, other supporting resources."
    },
    {
      "@id": "AbstractResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SemanticFlowResource/"
      ],
      "rdfs:label": "Abstract resource",
      "rdfs:comment": "A slash-terminated IRI whose dereference yields a resource page."
    },
    {
      "@id": "LocatedFile/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "SemanticFlowResource/"
      ],
      "rdfs:label": "Located file",
      "rdfs:comment": "A file IRI (typically with an extension) intended to retrieve concrete bytes."
    },
    {
      "@id": "ResourcePage/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "LocatedFile/",
        "SemanticFlowResource/",
        "schema:WebPage"
      ],
      "rdfs:label": "Resource page file",
      "rdfs:comment": "A derived UI/navigation file (e.g., index.html) that presents an AbstractResource (or other referent) to humans."
    },
    {
      "@id": "ArtifactLocatedFile/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "LocatedFile/",
        "SemanticFlowResource/"
      ],
      "rdfs:label": "Artifact located file",
      "rdfs:comment": "A LocatedFile that is a concrete realization of an AbstractFile."
    },
    {
      "@id": "AssetFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "LocatedFile/"
      ],
      "rdfs:label": "Asset file",
      "rdfs:comment": "A non-artifact support file used by resource pages (images/css/js), conventionally under _assets/."
    },
    {
      "@id": "OperationalLogFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "LocatedFile/"
      ],
      "rdfs:label": "Operational log file",
      "rdfs:comment": "A plain tool log file (not discourse-worthy by default)."
    },
    {
      "@id": "MeshComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "AbstractResource/"
      ],
      "rdfs:label": "Mesh component",
      "rdfs:comment": "A reserved underscore-path/system resource (e.g., _mesh/, _knops/, _assets/, inventories, configs)."
    },
    {
      "@id": "containsSfloResource/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "AbstractResource/",
      "rdfs:range": "SemanticFlowResource/",
      "rdfs:label": "contains Semantic Flow resource",
      "rdfs:comment": "Structural containment relation for SF-governed resources (typically within reserved or conventional SF paths)."
    },
    {
      "@id": "hasResourcePage/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "ResourcePage/",
      "rdfs:label": "has resource page file",
      "rdfs:comment": "Associates any resource (including referents) with a human-readable presentation page."
    },
    {
      "@id": "Nomen/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "AbstractResource/"
      ],
      "rdfs:label": "Nomen",
      "rdfs:comment": "A human-facing designator IRI (e.g., /people/alice/) intended to denote a discourse-worthy thing."
    },
    {
      "@id": "NomenComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "MeshComponent/"
      ],
      "rdfs:label": "Nomen component",
      "rdfs:comment": "Reserved components under a Nomen (e.g., NomenHandle, Nomen flows, inventories)."
    },
    {
      "@id": "NomenHandle/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "AbstractResource/",
        "NomenComponent/"
      ],
      "rdfs:label": "Nomen handle",
      "rdfs:comment": "The naming-object handle for a Nomen (e.g., /people/alice/_nomen/), used to talk about the Nomen-as-Nomen."
    },
    {
      "@id": "hasNomenHandle/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Nomen/",
      "rdfs:range": "NomenHandle/",
      "rdfs:label": "has nomen handle",
      "rdfs:comment": "Associates a Nomen with its NomenHandle."
    },
    {
      "@id": "denotes/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "NomenHandle/",
      "rdfs:range": "rdfs:Resource",
      "rdfs:label": "denotes",
      "rdfs:comment": "Given a NomenHandle subject (denoting the Nomen as an identifier-object), asserts what the Nomen denotes. This is not a statement about the designator's referent when using the designator IRI directly."
    },
    {
      "@id": "ReferenceLink/",
      "@type": "rdfs:Class",
      "rdfs:label": "Reference link",
      "rdfs:comment": "A relator describing a reference-data link from a NomenHandle to a specific reference data source, including expectations and role metadata."
    },
    {
      "@id": "ReferenceRole/",
      "@type": "rdfs:Class",
      "rdfs:label": "Reference role",
      "rdfs:comment": "A controlled vocabulary of roles describing how a ReferenceLink should be used."
    },
    {
      "@id": "TargetKind/",
      "@type": "rdfs:Class",
      "rdfs:label": "Target kind",
      "rdfs:comment": "A metaclass used to constrain expected target kinds for ReferenceLinks (via punning)."
    },
    {
      "@id": "ReferenceRoleCanonical/",
      "@type": "ReferenceRole/",
      "rdfs:label": "Canonical"
    },
    {
      "@id": "ReferenceRoleIntegrate/",
      "@type": "ReferenceRole/",
      "rdfs:label": "Integrate into resource page"
    },
    {
      "@id": "ReferenceRoleSupplemental/",
      "@type": "ReferenceRole/",
      "rdfs:label": "Supplemental"
    },
    {
      "@id": "ReferenceRoleDeprecated/",
      "@type": "ReferenceRole/",
      "rdfs:label": "Deprecated"
    },
    {
      "@id": "hasReferenceLink/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "NomenHandle/",
      "rdfs:range": "ReferenceLink/",
      "rdfs:label": "has reference link",
      "rdfs:comment": "Links a NomenHandle to a ReferenceLink describing a specific reference-data source."
    },
    {
      "@id": "referenceLinkForNomen/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "NomenHandle/",
      "owl:inverseOf": "hasReferenceLink/",
      "rdfs:label": "reference link for nomen",
      "rdfs:comment": "Inverse of hasReferenceLink: identifies the NomenHandle a ReferenceLink is about."
    },
    {
      "@id": "referenceRole/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "ReferenceRole/",
      "rdfs:label": "reference role",
      "rdfs:comment": "Assigns one or more roles to a ReferenceLink (e.g., Canonical, Integrate, Supplemental, Deprecated)."
    },
    {
      "@id": "target/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "rdfs:Resource",
      "rdfs:label": "target",
      "rdfs:comment": "If present, the target is expected to denote a reference-data source."
    },
    {
      "@id": "targetUriLiteral/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "xsd:anyURI",
      "rdfs:label": "target URI literal",
      "rdfs:comment": "If only this is present, it is expected to locate retrievable content (without assuming denotation)."
    },
    {
      "@id": "expectedTargetKind/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "TargetKind/",
      "rdfs:label": "expected target kind",
      "rdfs:comment": "Expected kind of the target (e.g., Nomen, Flow, Slice, AbstractFile, ArtifactLocatedFile, LocatedFile)."
    },
    {
      "@id": "meshBase/",
      "@type": "owl:DatatypeProperty",
      "rdfs:range": "xsd:anyURI",
      "rdfs:label": "Mesh Base",
      "rdfs:comment": "A mesh's 'BASE', which should correspond to a publish location (URL/IRI string). Represented as a URI literal to avoid asserting anything about the denoted thing."
    },
    {
      "@id": "lastVerifiedAt/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "xsd:dateTimeStamp",
      "rdfs:label": "last verified at",
      "rdfs:comment": "Most recent verification timestamp for this reference link."
    },
    {
      "@id": "lastVerifiedBy/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ReferenceLink/",
      "rdfs:range": "rdfs:Resource",
      "rdfs:label": "last verified by",
      "rdfs:comment": "Agent responsible for the most recent verification of this reference link."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "AbstractResource/"
      ],
      "rdfs:label": "Knop",
      "rdfs:comment": "SF container anchored at a stable in-mesh location; hosts payload and supporting flows."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "AbstractResource/",
        "MeshComponent/"
      ],
      "rdfs:label": "Knop component",
      "rdfs:comment": "Reserved components under a Knop (payload/supporting flows and their parts)."
    },
    {
      "@id": "hasKnopComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopComponent/",
      "rdfs:label": "has knop component",
      "rdfs:comment": "Associates a Knop with one of its directly contained KnopComponents."
    },
    {
      "@id": "Flow/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "AbstractResource/"
      ],
      "rdfs:label": "Flow",
      "rdfs:comment": "Artifact over revisions; organizes Slices and provides a stable artifact identity."
    },
    {
      "@id": "KnopFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Flow/",
        "KnopComponent/"
      ],
      "rdfs:label": "Knop flow",
      "rdfs:comment": "A Flow that is a component of a Knop."
    },
    {
      "@id": "NomenFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Flow/",
        "NomenComponent/"
      ],
      "rdfs:label": "Nomen flow",
      "rdfs:comment": "A Flow that is a component of a NomenHandle."
    },
    {
      "@id": "MeshFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Flow/",
        "MeshComponent/"
      ],
      "rdfs:label": "Mesh flow",
      "rdfs:comment": "A Flow that is a component of the mesh (mesh-level)."
    },
    {
      "@id": "artifactKind/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Flow/",
      "rdfs:range": "rdfs:Resource",
      "rdfs:label": "artifact kind",
      "rdfs:comment": "Declares what kind of artifact this Flow carries (e.g., RdfDataset, RdfGraph, Image, Text)."
    },
    {
      "@id": "Slice/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "AbstractResource/",
        "MeshComponent/"
      ],
      "rdfs:label": "Slice",
      "rdfs:comment": "A particular revision of the artifact represented by a Flow."
    },
    {
      "@id": "WorkingSlice/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Slice/"
      ],
      "rdfs:label": "Working slice",
      "rdfs:comment": "Mutable current state for a Flow."
    },
    {
      "@id": "HistoricalSlice/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Slice/"
      ],
      "rdfs:label": "Historical slice",
      "rdfs:comment": "Immutable published snapshot for a Flow."
    },
    {
      "@id": "AbstractFile/",
      "@type": [
        "rdfs:Class",
        "TargetKind/"
      ],
      "rdfs:subClassOf": [
        "AbstractResource/",
        "MeshComponent/"
      ],
      "rdfs:label": "Abstract file",
      "rdfs:comment": "A representation specification of a Slice (format/syntax/canonicalization/bundling)."
    },
    {
      "@id": "hasFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "Flow/",
      "rdfs:label": "has flow",
      "rdfs:comment": "Links a Knop to one of its contained flows."
    },
    {
      "@id": "hasSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Slice/",
      "rdfs:label": "has slice",
      "rdfs:comment": "Associates a Flow with one of its Slices."
    },
    {
      "@id": "currentSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasSlice/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Slice/",
      "rdfs:label": "current slice",
      "rdfs:comment": "Associates a Flow with its current Slice (often the WorkingSlice)."
    },
    {
      "@id": "previousSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Slice/",
      "rdfs:range": "Slice/",
      "rdfs:label": "previous slice",
      "rdfs:comment": "Links to the immediately-preceding Slice of this Slice."
    },
    {
      "@id": "nextSliceSequenceNumber/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "Flow/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "next slice sequence number",
      "rdfs:comment": "Monotonic integer providing the next allocatable sequence number for a Flow."
    },
    {
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Slice/",
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Slice to one of its AbstractFiles."
    },
    {
      "@id": "hasLocatedFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "AbstractFile/",
      "rdfs:range": "LocatedFile/",
      "rdfs:label": "has located file",
      "rdfs:comment": "Links an AbstractFile to a LocatedFile endpoint that retrieves bytes."
    },
    {
      "@id": "hasCanonicalLocatedFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasLocatedFile/",
      "rdfs:domain": "AbstractFile/",
      "rdfs:range": "ArtifactLocatedFile/",
      "rdfs:label": "has canonical located file",
      "rdfs:comment": "Links an AbstractFile to its canonical ArtifactLocatedFile."
    },
    {
      "@id": "realizesAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "ArtifactLocatedFile/",
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "realizes abstract file",
      "rdfs:comment": "Links an ArtifactLocatedFile to the AbstractFile it realizes (enables navigation without path heuristics)."
    },
    {
      "@id": "DatasetSeries/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "AbstractResource/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Dataset series",
      "rdfs:comment": "A series/partitioning of related dataset artifacts."
    },
    {
      "@id": "SupportingFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "Flow/"
      ],
      "rdfs:label": "Supporting flow",
      "rdfs:comment": "A non-payload flow used for metadata/inventory/auxiliary artifacts."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "KnopFlow/"
      ],
      "rdfs:label": "Payload flow",
      "rdfs:comment": "The primary digital-artifact flow hosted by a Knop."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SupportingFlow/",
        "KnopFlow/"
      ],
      "rdfs:label": "Knop metadata flow",
      "rdfs:comment": "Small, discoverable: structure/pointers/summary inventory for the Knop."
    },
    {
      "@id": "MeshMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SupportingFlow/",
        "MeshFlow/"
      ],
      "rdfs:label": "Mesh metadata flow",
      "rdfs:comment": "Optional mesh-level operational metadata/config."
    },
    {
      "@id": "NomenMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SupportingFlow/",
        "NomenFlow/"
      ],
      "rdfs:label": "Nomen metadata flow",
      "rdfs:comment": "Metadata about a NomenHandle; may remain WorkingSlice-only until history is recorded."
    },
    {
      "@id": "hasNomenMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "NomenHandle/",
      "rdfs:range": "NomenMetadataFlow/",
      "rdfs:label": "has nomen metadata flow",
      "rdfs:comment": "Links a NomenHandle to its nomen metadata flow (if present)."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "PayloadFlow/",
      "rdfs:label": "has payload flow",
      "rdfs:comment": "Links a Knop to its required payload flow."
    },
    {
      "@id": "hasSupportingFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "SupportingFlow/",
      "rdfs:label": "has supporting flow",
      "rdfs:comment": "Links a Knop to one of its supporting flows."
    },
    {
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasSupportingFlow/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has knop metadata flow",
      "rdfs:comment": "Links a Knop to its knop metadata flow (if present)."
    },
    {
      "@id": "RdfDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "dcat:Dataset"
      ],
      "rdfs:label": "RDF dataset",
      "rdfs:comment": "RDF dataset content: default graph + 0..n named graphs; default graph may be empty."
    },
    {
      "@id": "RdfGraph/",
      "@type": "rdfs:Class",
      "rdfs:label": "RDF graph",
      "rdfs:comment": "RDF graph content: exactly one graph (default graph only; may be empty)."
    },
    {
      "@id": "NamedGraph/",
      "@type": "rdfs:Class",
      "rdfs:label": "Named graph",
      "rdfs:comment": "A graph identified by a graph-name IRI (interpretation depends on whether you denote a Flow, a HistoricalSlice, or a WorkingSlice)."
    },
    {
      "@id": "MeshInventoryDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "RdfDataset/",
        "MeshComponent/"
      ],
      "rdfs:label": "Mesh inventory dataset",
      "rdfs:comment": "Optional derived index for explorability at the mesh level (non-flow; may be large/sharded)."
    },
    {
      "@id": "KnopInventoryDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "RdfDataset/",
        "KnopComponent/"
      ],
      "rdfs:label": "Knop inventory dataset",
      "rdfs:comment": "Optional derived index for explorability at the knop level (non-flow; may be large/sharded)."
    },
    {
      "@id": "NomenInventoryDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "RdfDataset/",
        "NomenComponent/"
      ],
      "rdfs:label": "Nomen inventory dataset",
      "rdfs:comment": "Optional derived index for explorability at the nomen handle level (non-flow; may be large/sharded)."
    }
  ]
}
Semantic Flow Ontology (revised) — Turtle
Sorry, I was confused and pasted the old JSON-LD ontology. You can ignore everything from that. 

Realizable is a subclass of Realization? I don't think so. 

Here's the real latest working version, sorry about the confusion:

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


# Core Semantice Mesh-scoped resources

<MeshResource/> a rdfs:Class ;
  rdfs:label "Mesh resource" ;
  rdfs:comment "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves" .

<ArtifactHost/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Artifact host" ;
  rdfs:comment "A mesh-governed resource that can contain artifacts and associated structural resources. Nomen are ArtifactHosts that can also contain other Nomen. " .

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
  rdfs:comment "A reserved, mesh-scoped structural resource (e.g., the _mesh/ handle, mesh-level metadata, mesh inventory)." .  

<containsMeshResource/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <MeshResource/> ;
  rdfs:label "contains mesh resource" ;
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

<hasMeshComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshComponent/> ;
  rdfs:label "has mesh component" ;
  rdfs:comment "Associates a Mesh with one of its directly contained MeshComponents." .

<hasKnopComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopComponent/> ;
  rdfs:label "has knop component" ;
  rdfs:comment "Associates a Knop with one of its directly contained KnopComponents." .

<hasNomenComponent/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenComponent/> ;
  rdfs:label "has nomen component" ;
  rdfs:comment "Associates a Nomen with one of its directly contained NomenComponents." .

  
# General-purpose Resources
# These could be usable anywhere on the Semantic Web, not just in Semantic Flow meshes'

## Realizable + Realizations

<Realizable/> a rdfs:Class ;
  rdfs:label "Realizable" ;
  rdfs:comment "A abstract resource whose content can be realized via one or more Realizations (e.g., files, streams)." .

<Realization/> a rdfs:Class ;
  rdfs:label "Realization" ;
  rdfs:comment "A abstract representation/encoding/packaging of a Realizable resource, independent of any particular access endpoint." .


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

# Artifacts

## Digital Artifacts 

<DigitalArtifact/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Digital artifact" ;
  rdfs:comment "A mesh-governed artifact identity (something treated as a digital artifact in the abstract)." .

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

<NomenComponent/> a rdfs:Class ;
  rdfs:subClassOf <MeshResource/> ;
  rdfs:label "Nomen component" ;
  rdfs:comment "Reserved components under a Nomen (e.g., nomen flows, inventories)." .

<nomenSubject/> a owl:ObjectProperty ;
  rdfs:domain <Nomen/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "nomen subject" ;
  rdfs:comment "Optional asserted subject (designator IRI) for the Nomen. Usually matches the meshPath. If absent, the Nomen may be anchored only via ReferenceLinks (or used purely for namespacing / token management)." .

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
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <Nomen/> ;
  rdfs:label "has child nomen" ;
  rdfs:comment "Operational/derived listing of subordinate Nomens under this Nomen's namespace path. Not a source of truth; implementations may derive children from mesh structure and/or meshPath prefix rules." .


### Reference data links

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a Nomen to a specific reference data source, including expectations and role metadata." .

<ReferenceRole/> a rdfs:Class ;
  rdfs:label "Reference role" ;
  rdfs:comment "A controlled vocabulary of roles describing how a ReferenceLink should be used." .

<ReferenceRoleCanonical/> a <ReferenceRole/> ;
  rdfs:label "Canonical" .

<ReferenceRoleIntegrate/> a <ReferenceRole/> ;
  rdfs:label "Integrate into resource page" .

<ReferenceRoleSupplemental/> a <ReferenceRole/> ;
  rdfs:label "Supplemental" .

<ReferenceRoleDeprecated/> a <ReferenceRole/> ;
  rdfs:label "Deprecated" .

<hasReferenceLink/> a owl:ObjectProperty ;
  rdfs:domain <ArtifactHost/> ;
  rdfs:range <ReferenceLink/> ;
  rdfs:label "has reference link" ;
  rdfs:comment "Links a Nomen to a ReferenceLink describing a specific reference-data source." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ArtifactHost/> ;
  owl:inverseOf <hasReferenceLink/> ;
  rdfs:label "reference link for nomen" ;
  rdfs:comment "Inverse of hasReferenceLink: identifies the Nomen a ReferenceLink is about." .

<referenceRole/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range <ReferenceRole/> ;
  rdfs:label "reference role" ;
  rdfs:comment "Assigns one or more roles to a ReferenceLink (e.g., Canonical, Integrate, Supplemental, Deprecated)." .

<referenceTarget/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
  rdfs:label "reference target" ;
  rdfs:comment "If present, the reference target is a node used as a reference-data handle (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

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
  rdfs:comment "SF container anchored at a stable in-mesh location; hosts payload and supporting flows." .

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
  rdfs:label "has payload" ;
  rdfs:comment "Links a Knop to its primary payload artifact identity (either a FlowArtifact-based payload or a SimpleArtifact-based payload)." .

<hasFlowPayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <FlowPayload/> ;
  rdfs:label "has flow payload" ;
  rdfs:comment "Links a Knop to its payload when the payload is modeled as a FlowArtifact (FlowPayload)." .

<hasSimplePayload/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <hasPayload/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <SimplePayload/> ;
  rdfs:label "has simple payload" ;
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

<hasKnopMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Knop/> ;
  rdfs:range <KnopMetadataFlow/> ;
  rdfs:label "has knop metadata flow" ;
  rdfs:comment "Links a Knop to its knop metadata flow (if present)." .

<hasMeshMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Mesh/> ;
  rdfs:range <MeshMetadataFlow/> ;
  rdfs:label "has mesh metadata flow" ;
  rdfs:comment "Links a Mesh to its mesh metadata flow (if present)." .

<hasNomenMetadataFlow/> a owl:ObjectProperty ;
  rdfs:subPropertyOf <containsMeshResource/> ;
  rdfs:domain <Nomen/> ;
  rdfs:range <NomenMetadataFlow/> ;
  rdfs:label "has nomen metadata flow" ;
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


I'd intended designates to target "what the nomen names" and "nomenSubject" to refer to the thing whose IRI contains the nomen. e.g., for the "Alice's biography dataset" the nomenSubject is <alice/bio-dataset/> and the "designates" could be like "_knops/<uuid>/_payload/"

Your split is coherent, but the *names* are currently doing you no favors.

## What you intend (restated precisely)

For a Nomen at:

* `…/alice/bio-dataset/_nomen/` (the **Nomen object**, i.e., the metadata node)

you want two different semantic hooks:

1. **“What token is this Nomen about?”** (the designator token / entrypoint IRI)

* e.g. `…/alice/bio-dataset/`
* this is *not* the referent; it’s the **identifier you’re managing**.

2. **“What does that token denote/designate?”** (the referent / designatum)

* e.g. `…/_knops/<uuid>/_payload/`

That’s a clean 2-step chain:

`Nomen  --(about token)--> DesignatorTokenIRI  --(designates)--> Designatum`

## The problem: `nomenSubject` is misleading

In most ontologies, “subject” strongly reads as “the thing being talked about” (i.e., referent/designatum), not “the identifier token the metadata node is attached to”.

So if you keep `nomenSubject` with your intended meaning, you’ll constantly confuse future-you and any collaborator/LLM.

## Recommendation

### Keep `designates` for the designatum (your `_knops/.../_payload/` case)

That’s fine.

### Rename `nomenSubject` to something that screams “token/designator”

Pick one and commit:

* `nomenFor` (my preference: short and readable)
* `designatorToken`
* `designatorIri`
* `nomenToken`

Example (minimal change, only renaming semantics):

* `nomenFor` (domain `Nomen`, range `rdfs:Resource`): **the designator token IRI whose `_nomen/` node this is**
* `designates` (domain `Nomen`, range `rdfs:Resource`): **what that designator token designates**

## You must also decide which is authoritative: `meshPath` vs token-IRI property

Right now you have both `meshPath` and your would-be `nomenSubject`/`nomenFor`.

If both can appear, you need a rule, or you’ve introduced silent contradictions.

Best rule:

* `meshPath` is the *authoritative* token (portable, base-relative)
* `nomenFor` is optional convenience/denormalization
* If `nomenFor` is present, it **MUST equal** `meshBase + meshPath` (with the empty-string root rule)

This is a SHACL shape, not OWL.

## One more nit: your current `nomenSubject` comment is wrong for your new intention

It currently says “Usually matches the meshPath. If absent… anchored…”. That reads like “subject” = referent. If you adopt the split you described, update the comment to be unambiguously token-level.

If you want, I’ll patch the canvas Turtle with:

* a rename of `nomenSubject` → `nomenFor` (or your chosen name),
* updated comments,
* and a SHACL constraint that enforces `nomenFor == meshBase + meshPath` when both are present.
