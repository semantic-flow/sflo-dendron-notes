---
id: u0modyi4rs2kv024x7ef7dc
title: 2025 12 19 Ontology Suggestions and Fixe
desc: ''
updated: 1766164711267
created: 1766164707651
---
Skip to content
Chat history
You said:
Any last suggestions for the main ontology? I haven't defined an sflo:Dataset, but I don't see a need. 

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "prov": "http://www.w3.org/ns/prov#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
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
    }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": {
        "@id": "https://djradon.github.io/ns/djradon/"
      },
      "owl:versionInfo": "0.1.0",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/ontology/semantic-flow/_payload/_default/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
    },
    {
      "@id": "PlatformElement/",
      "@type": "rdfs:Class",
      "rdfs:label": "Platform Element",
      "rdfs:comment": "Abstract base class for all Semantic Flow resources. Provides fundamental identification and documentation affordances."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "Knop",
      "rdfs:comment": "Structural-conceptual units that provide identifiers for things, may act as namespace segments to organize/contain other Knops, may contain reference data about their referents, and may have a payload dataset. Knops form the superstructure of a semantic mesh."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "KnopComponent",
      "rdfs:comment": "Resources that provide specific mesh functionality and support knops."
    },
    {
      "@id": "hasKnopComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopComponent/",
      "rdfs:label": "has KnopComponent",
      "rdfs:comment": "Associates knops with their supporting components."
    },
    {
      "@id": "KnopNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "Knop node",
      "rdfs:comment": "A specific location of a Knop."
    },
    {
      "@id": "hasKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has knop node",
      "rdfs:comment": "Associates a Knop with a KnopNode.",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopNode/"
    },
    {
      "@id": "hasActiveKnopNode/",
      "@type": [
        "owl:ObjectProperty",
        "owl:FunctionalProperty"
      ],
      "rdfs:label": "has active knop node",
      "rdfs:comment": "Associates a Knop with its active KnopNode.",
      "rdfs:subPropertyOf": "hasKnopNode/"
    },
    {
      "@id": "KnopNodeComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "KnopNodeComponent",
      "rdfs:comment": "A KnopNodeComponent provides specific Node-level functionality."
    },
    {
      "@id": "hasKnopNodeComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNodeComponent/",
      "rdfs:label": "has KnopNodeComponent",
      "rdfs:comment": "Associates KnopNodes with their supporting components."
    },
    {
      "@id": "hasKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:range": "Knop/",
      "rdfs:label": "has Knop",
      "rdfs:comment": "Associates a referent (a thing or concept worth identifying) with its Knop."
    },
    {
      "@id": "ResourcePage/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "Resource page",
      "rdfs:comment": "Human-readable index.html pages that provide dereferenceability for non-file PlatformElements."
    },
    {
      "@id": "hasResourcePage/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "PlatformElement/",
      "rdfs:range": "ResourcePage/",
      "rdfs:label": "has ResourcePage",
      "rdfs:comment": "Associates a PlatformElement with its index.html resource page file."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "PlatformElement/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Flow",
      "rdfs:comment": "Evolvable containers for knop metadata, reference data, payload data."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Metadata Flow",
      "rdfs:comment": "Flow containing metadata about the mesh knop itself."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Reference Flow",
      "rdfs:comment": "Flow containing reference data about the thing or concept referred to by the mesh knop."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Payload flow",
      "rdfs:comment": "Flow containing version of the Dataset identified by a PayloadKnop."
    },
    {
      "@id": "hasFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "Flow/",
      "rdfs:label": "has Flow",
      "rdfs:comment": "Abstract property for linking mesh knops to their flows (metadata, reference, data)."
    },
    {
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has KnopMetadataFlow",
      "rdfs:comment": "Links a mesh knop to its metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "ReferenceFlow/",
      "rdfs:label": "has ReferenceFlow",
      "rdfs:comment": "Links a mesh knop to its reference flow."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "PayloadFlow/",
      "rdfs:label": "has PayloadFlow",
      "rdfs:comment": "Links a mesh knop to its payload flow."
    },
    {
      "@id": "FlowSlice/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "PlatformElement/",
        "dcat:Dataset"
      ],
      "rdfs:label": "Flow Slice",
      "rdfs:comment": "One concrete dataset instance belonging to a Flow. A FlowSlice represents the Flow's data as a whole at a particular stage, not a subset of it."
    },
    {
      "@id": "hasFlowSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "FlowSlice/",
      "rdfs:label": "has flow slice",
      "rdfs:comment": "Associates a Flow with one of its slices."
    },
    {
      "@id": "hasCurrentSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlowSlice/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "FlowSlice/",
      "rdfs:label": "has current slice",
      "rdfs:comment": "Associates a Flow with its current FlowSlice."
    },
    {
      "@id": "previousSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "dcat:previousVersion",
      "rdfs:label": "previous slice",
      "rdfs:comment": "Links to the immediately-preceeding slice of this FlowSlice.",
      "rdfs:domain": "FlowSlice/",
      "rdfs:range": "FlowSlice/"
    },
    {
      "@id": "sliceSequence/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "FlowSlice/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "slice sequence",
      "rdfs:comment": "Monotonic integer providing the sequence number of this slice within its Flow. Usually 1 for the first-woven slice of a flow, 2 for the next, etc. Used for ordering and for detecting gaps or compaction at the flow level."
    },
    {
      "@id": "anticipatedAbsoluteIri/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "anticipated absolute IRI",
      "rdfs:comment": "The anticipated absolute IRI of this knop within a published mesh, based on its Knop identifier, its path location, and the hosting platform."
    },
    {
      "@id": "peerKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "is peer KnopNode",
      "rdfs:comment": "Associates a KnopNode with another KnopNode that is part of the same conceptual knop.",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNode/"
    },
    {
      "@id": "activePeerKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "active peer KnopNode",
      "rdfs:comment": "Associates a KnopNode with the currently active KnopNode for a Knop.",
      "rdfs:subPropertyOf": "peerKnopNode/",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNode/"
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
      "rdfs:range": "xsd:boolean"
    },
    {
      "@id": "PointersDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "KnopComponent/",
        "dcat:Dataset"
      ],
      "rdfs:label": "Pointers dataset",
      "rdfs:comment": "A minimal dataset for bootstrapping."
    },
    {
      "@id": "currentMetaDownloadTTL/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:label": "current meta download Turtle",
      "rdfs:comment": "Relative IRI to fetch the current meta bytes for this knop. The representation is expected to be Turtle."
    },
    {
      "@id": "currentMetaDownloadJSONLD/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:label": "current meta download JSON-LD",
      "rdfs:comment": "Relative IRI to fetch the current meta bytes for this knop. The representation is expected to be JSON-LD."
    },
    {
      "@id": "KnopNodeDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "KnopNodeComponent/",
        "dcat:Dataset"
      ],
      "rdfs:label": "KnopNode dataset",
      "rdfs:comment": "Base class for KnopNode-level operational datasets (inventory, opslog, backlinks)."
    },
    {
      "@id": "InventoryDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "Inventory dataset",
      "rdfs:comment": "KnopNode-level dataset describing the locally-present artifacts/resources/components at that KnopNode."
    },
    {
      "@id": "hasInventoryDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "InventoryDataset/",
      "rdfs:label": "has inventory dataset",
      "rdfs:comment": "Links a KnopNode to its InventoryDataset component."
    },
    {
      "@id": "OpslogDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "opslog dataset",
      "rdfs:comment": "KnopNode-level append-only dataset recording operations/events relevant to that KnopNode."
    },
    {
      "@id": "hasOpslogDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links a KnopNode to its OpslogDataset component."
    },
    {
      "@id": "BacklinksDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "backlinks dataset",
      "rdfs:comment": "KnopNode-level dataset describing backlinks to a published IRI corresponding to this KnopNode."
    },
    {
      "@id": "hasBacklinksDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "BacklinksDataset/",
      "rdfs:label": "has backlinks dataset",
      "rdfs:comment": "Links a KnopNode to its BacklinksDataset component."
    },
    {
      "@id": "Distribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "dcat:Distribution",
      "rdfs:label": "Distribution",
      "rdfs:comment": "Semantic Flow abstract Distribution."
    },
    {
      "@id": "JsonldDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "JSON-LD Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for JSON-LD representations."
    },
    {
      "@id": "hasJsonldDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has JSON-LD Distribution",
      "rdfs:comment": "Links a FlowSlice to its JSON-LD Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "dcat:Dataset",
      "rdfs:range": "JsonldDistribution/"
    },
    {
      "@id": "TrigDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "Trig Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for TriG representations."
    },
    {
      "@id": "hasTrigDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has JSON-LD Distribution",
      "rdfs:comment": "Links a FlowSlice to its JSON-LD Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "dcat:Dataset",
      "rdfs:range": "JsonldDistribution/"
    },
    {
      "@id": "TurtleDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "Turtle Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for Turtle representations."
    },
    {
      "@id": "hasTurtleDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has Turtle Distribution",
      "rdfs:comment": "Links a FlowSlice to its Turtle Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "dcat:Dataset",
      "rdfs:range": "TurtleDistribution/"
    }
  ]
}
ChatGPT said:

Key issues and last-pass fixes (highest impact first).
1) You accidentally imply Flow ⊑ Knop (domain leakage via subPropertyOf)

You declared:

    hasKnopComponent/ has rdfs:domain Knop/.

    hasFlowSlice/ is a subPropertyOf hasKnopComponent/, but also has rdfs:domain Flow/.

In RDFS/OWL reasoning, subproperties inherit the domain/range constraints of their superproperties. So any subject using hasFlowSlice/ gets inferred to be a Knop/, which will effectively force Flow instances to also be Knop instances in your data.

Fix: remove this line:

    hasFlowSlice/ rdfs:subPropertyOf hasKnopComponent/

If you want a hierarchy, make hasFlowSlice/ a subproperty of something that does not have Knop/ as its domain (or leave it standalone). Similar caution for any future “component” superproperties with strong domains.
2) Two clear copy/paste bugs in your distribution properties
hasTrigDistribution/ is wrong in 3 places

It currently has:

    rdfs:label: “has JSON-LD Distribution” (wrong)

    rdfs:comment: “Links … to its JSON-LD Distribution.” (wrong)

    rdfs:range: JsonldDistribution/ (wrong)

Fix:

    label/comment should say TriG

    range should be TrigDistribution/

3) currentMetaDownloadTTL/ and currentMetaDownloadJSONLD/ are typed as ObjectProperties, but read like URL/IRI strings

Your comment says “Relative IRI to fetch the current meta bytes…”. That’s either:

    a literal string / anyURI value, or

    a resource denoting an access URL, distribution, or file.

Right now they’re owl:ObjectProperty with no range. That’s underspecified, and will cause inconsistent modeling later.

Pick one:

Option A (simple): make them datatype properties

    @type: owl:DatatypeProperty

    rdfs:range: xsd:anyURI (or xsd:string if you truly want relative-path strings)

Option B (more DCAT-correct): point to a Distribution

    @type: owl:ObjectProperty

    rdfs:range: Distribution/ (or a dedicated MetaDistribution/)

    then use dcat:downloadURL / dcat:accessURL on the Distribution.

Given your “pointers” bootstrapping goal, Option A is fine, but be explicit.
4) anticipatedAbsoluteIri/ is missing the basics

It has no rdfs:domain or rdfs:range, and it’s named “IRI” but typed as a datatype property.

At minimum:

    rdfs:domain: Knop/ (or KnopNode/ if it’s node-specific)

    rdfs:range: xsd:anyURI

Also: “anticipated” is operational/environment-derived. Consider moving it to an operational/config ontology rather than the “main” ontology, unless you truly want it to be first-class core data.
5) rootKnop/ should have a domain (and probably shouldn’t be core)

Same pattern as parentKnop/: you say it “Should only be used in operational KnopConfig.” If that’s true, it likely belongs in your config vocabulary, not your platform/core vocabulary.

If you keep it here anyway, add:

    rdfs:domain: Knop/

6) hasKnop/ is floating (no domain) and its semantics are muddy

You describe: “Associates a referent … with its Knop.”

But you have not introduced a Referent class (and you previously leaned toward not introducing it). That’s fine, but then be disciplined:

    If the subject can be literally anything (rdfs:Resource), that’s OK—but then you’re effectively making a reverse index from “thing” → knop. That’s dangerous if multiple knops can identify the same thing over time, or if you want identity to be “path = identity”.

At minimum:

    add rdfs:domain: rdfs:Resource (or omit domain deliberately, but do it consciously)

    keep rdfs:range: Knop/ (you already have range)

    consider whether you actually want the inverse instead: identifies/ from Knop/ → thing, with hasKnop/ as inverse.

Right now your property name reads like Knop -> Knop, but your comment reads like Thing -> Knop. That mismatch will confuse future-you.
7) Naming / consistency nits that will bite later

    KnopNode/ label is “Knop node” while KnopNodeComponent/ label is “KnopNodeComponent” (inconsistent casing/spacing). Pick one style.

    peerKnopNode/ label says “is peer KnopNode” but the property name is not “is…”. Either rename label or property.

    “immediately-preceeding” → “immediately-preceding”.

8) Ontology header: your owl:versionIRI looks wrong for your stated base

You set @base and vann:preferredNamespaceUri to:

    https://semantic-flow.github.io/ontology/semantic-flow-ontology/

…but owl:versionIRI points to:

    https://semantic-flow.github.io/ontology/semantic-flow/_payload/_default/sflo.ttl

That’s a different namespace path and looks like a payload artifact location, not an ontology version IRI.

If you intend to publish versions as ontology resources, align this with your ontology namespace (even if the bytes live elsewhere).
9) “I haven’t defined sflo:Dataset” is fine, but then be consistent

You already model “dataset-ness” via dcat:Dataset and dcat:DatasetSeries subclasses (FlowSlice, Flow, PointersDataset, KnopNodeDataset, etc.). Introducing sflo:Dataset now would mostly create redundancy unless you have a platform-specific meaning that DCAT doesn’t capture.

If you keep it out, I’d still recommend:

    Decide whether FlowSlice is always RDF, or can be any digital artifact.

        Your comments oscillate (“Dataset” vs “any digital file”). If payload can be arbitrary, then dcat:Dataset may be the wrong superclass for some payload slices, and you may want a parallel “ArtifactSlice/Distribution” track.

The ontology currently suggests everything slice-like is a DCAT Dataset, which conflicts with “payload can be any digital file”.

If you only change three things before calling it “done”:

    remove hasFlowSlice subPropertyOf hasKnopComponent (domain leakage)

    fix hasTrigDistribution (label/comment/range)

    decide whether meta-download properties are literals (xsd:anyURI) or DCAT Distributions, and encode that explicitly.

You said:
Should I add a superproperty "hasDistribution"?

Regarding #3, Option B is correct. A dcat:downloadURL isn't necessary in my mind (it's redundant), but you seem to like it for some reason.

For versionIRI, it will live somewhere like _payload/v1/semantic-flow-ontology.jsonld -- so should "owl:versionInfo" be "1" or "v1"

Also, it's conventional to use versionIRI to point to a downloadURL.

FlowSlice is always RDF. Payload can only be RDF.

download URLs are DCAT Distributions, and always also encode the access URL.
ChatGPT said:
1) Add hasDistribution/?

Yes, add it, but only if you tighten your model.

Right now you have syntax-specific properties (e.g., hasJsonldDistribution/) as subproperties of dcat:distribution. If you introduce sflo:hasDistribution/, you should avoid creating two parallel “distribution trees” that diverge.

Recommended pattern:

    Define sflo:hasDistribution/ as a subproperty of dcat:distribution.

    Make sflo:hasJsonldDistribution/, sflo:hasTrigDistribution/, sflo:hasTurtleDistribution/ subproperties of sflo:hasDistribution/ (not directly of dcat:distribution).

That gives you one “Semantic Flow” hook while remaining DCAT-native.

Minimal definition:

    sflo:hasDistribution/ (owl:ObjectProperty)

        rdfs:subPropertyOf dcat:distribution

        rdfs:domain dcat:Dataset (or sflo:FlowSlice/ if you want it narrower)

        rdfs:range sflo:Distribution/ (your subclass of dcat:Distribution)

2) About dcat:downloadURL

If your “download URL” is itself the Distribution IRI and that IRI is dereferenceable to bytes (or 303s to bytes), then you can omit dcat:downloadURL and still be internally consistent.

But be aware of the tradeoff: DCAT consumers generally expect the Distribution to describe access using dcat:downloadURL / dcat:accessURL. If you skip it, you’re leaning on a house rule: “Distribution IRI dereferences to the file.” That’s fine if you control the ecosystem; it’s less interoperable.

Given your statement: “download URLs are DCAT Distributions, and always also encode the access URL,” your clean approach is:

    Treat Distribution IRI as the access URL by convention.

    Optionally still include dcat:downloadURL with the same IRI for compatibility (yes, redundant; also pragmatic).

3) owl:versionInfo: “1” vs “v1”

Use semantic versioning if you have it: "0.1.0", "1.0.0", etc. That’s what owl:versionInfo is typically used for.

If you insist on the folder name convention _payload/v1/..., keep that for the IRI path, not for the version string.

So:

    owl:versionInfo: "0.1.0" (or "1.0.0")

    owl:versionIRI: the concrete versioned artifact IRI (which can include /v1/ in its path)

If you do not use SemVer and only have an integer release sequence, then "1" is better than "v1" for a value—reserve the “v” prefix for paths/filenames.
4) Is it conventional for owl:versionIRI to point to a download URL?

Not exactly. It’s conventional for owl:versionIRI to identify the versioned ontology resource. Many projects make that IRI dereference to the concrete file bytes (or content-negotiated variants), so it behaves like a download URL, but it’s conceptually the version identifier.

Your plan (“versionIRI lives at _payload/v1/semantic-flow-ontology.jsonld”) is compatible with that convention.
5) “FlowSlice is always RDF. Payload can only be RDF.”

Good—this resolves the earlier conceptual inconsistency. Keep FlowSlice ⊑ dcat:Dataset and keep payload slices as datasets.

Then tighten comments/names that still imply “any digital file”:

    PayloadFlow comment currently says “version of the Dataset identified by a PayloadKnop.” Good.

    Anywhere you still describe payload as arbitrary artifacts should be removed/updated.

6) “download URLs are DCAT Distributions, and always also encode the access URL.”

Then your ontology should reflect that explicitly to prevent drift:

Add either:

    a datatype property like sflo:distributionAccessIri/ (xsd:anyURI) only if the Distribution IRI is not always the access IRI; otherwise it’s pointless.

Or (better, given your conventions):

    document the convention in rdfs:comment on sflo:Distribution/: “The Distribution IRI is the access IRI; dereferencing yields the representation bytes.”

And if you want compatibility without changing your core rule:

    include dcat:downloadURL optionally, but treat it as a duplicated convenience assertion.

One more consistency trap to fix while you’re here

If you rewire distribution properties under sflo:hasDistribution/, make sure domains/ranges match:

    sflo:hasTrigDistribution/ must have rdfs:range sflo:TrigDistribution/

    label/comment corrected (you had the JSON-LD copy/paste error before)

This is the kind of tiny inconsistency that becomes permanent “schema folklore” later if not fixed now.
You said:
I'm very conflicted about defining my own dataset. This trailing slash justifies it so that my terms can be consistent, and it also might make it unnecessary to add DCaT to some contexts, but it doesn't hew to vocab reuse.
ChatGPT said:

If the real motivation is “terminology consistency” and “not having to include DCAT everywhere,” defining sflo:Dataset/ is a trap unless you nail down what it means beyond dcat:Dataset.
Why it’s a trap

    If sflo:Dataset/ owl:equivalentClass dcat:Dataset, you didn’t actually avoid DCAT. You just added an alias layer that will confuse consumers and future modeling (and you still need DCAT semantics somewhere).

    If sflo:Dataset/ rdfs:subClassOf dcat:Dataset, you’ve created a narrower class. Then you must be able to say what extra constraints/intent make something an “sflo dataset” vs a generic dataset. If you can’t, it’s arbitrary and you’ll regret it.

    If sflo:Dataset/ is not related to dcat:Dataset, you’ve forked meaning and lost reuse.

The clean compromise

Keep DCAT as your semantic foundation, but create a Semantic Flow façade that is explicitly an alias and optionally skippable.

Do this:

    Keep FlowSlice/ rdfs:subClassOf dcat:Dataset and Flow/ rdfs:subClassOf dcat:DatasetSeries. (You already did.)

    If you want “sflo terminology consistency,” define:

    sflo:Dataset/ as a local term that is transparently mapped:

        sflo:Dataset/ owl:equivalentClass dcat:Dataset

…and then be honest in comments that it’s an alias.

This gives you:

    consistent sflo:* naming in your docs/UI/templates,

    ability to omit DCAT prefixes in many authoring contexts,

    no semantic divergence (because it is explicitly equivalent).

The price you pay (and it’s real)

    People consuming your data still need DCAT (or at least need to understand that your Dataset is DCAT’s Dataset).

    You’ve created one more thing to explain (“Why not just use dcat:Dataset?”).

    If you later add constraints to sflo:Dataset/, you’ve invalidated earlier assumptions.

If you want to avoid importing DCAT in some contexts

Don’t solve that with a new Dataset class. Solve it with JSON-LD contexts.

    Keep the ontology DCAT-aligned.

    Provide a “minimal authoring context” that maps your sflo:Dataset/ alias to dcat:Dataset (or directly maps sflo:FlowSlice/ etc. to DCAT terms) so authors don’t have to mention DCAT prefixes.

    Your ontology stays reuse-friendly; your authoring UX stays clean.

Decision rule

Only define sflo:Dataset/ if you can answer, in one sentence, without handwaving:

    “sflo:Dataset is exactly dcat:Dataset but named for Semantic Flow authoring convenience.”

If you can’t commit to that (i.e., you think you might later want a different meaning), then don’t define it—use contexts and keep FlowSlice as your concrete dataset concept.

Given your current model (“FlowSlice is always RDF; payload is RDF”), you already have a perfectly good sflo-native dataset concept: sflo:FlowSlice/. That’s the term your users will touch most. The extra sflo:Dataset/ layer is only worth it if you want a generic hook for other dataset-like things later.
You said:
I have things other than FlowSlice that are datasets. One justification is I might require all Dataset/ to have at least one Distribution/, and maybe that it's abstract (instances must at least declare some other concrete type). 

Here's what it looks like now:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "prov": "http://www.w3.org/ns/prov#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
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
    }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": {
        "@id": "https://djradon.github.io/ns/djradon/"
      },
      "owl:versionInfo": "1",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/_payload/_default/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
    },
    {
      "@id": "PlatformElement/",
      "@type": "rdfs:Class",
      "rdfs:label": "Platform Element",
      "rdfs:comment": "Abstract base class for all Semantic Flow resources. Provides fundamental identification and documentation affordances."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "Knop",
      "rdfs:comment": "Structural-conceptual units that provide identifiers for things, may act as namespace segments to organize/contain other Knops, may contain reference data about their referents, and may have a payload dataset. Knops form the superstructure of a semantic mesh."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "KnopComponent",
      "rdfs:comment": "Resources that provide specific mesh functionality and support knops."
    },
    {
      "@id": "hasKnopComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopComponent/",
      "rdfs:label": "has KnopComponent",
      "rdfs:comment": "Associates knops with their supporting components."
    },
    {
      "@id": "KnopNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "KnopNode",
      "rdfs:comment": "A specific location of a Knop."
    },
    {
      "@id": "hasKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has KnopNode",
      "rdfs:comment": "Associates a Knop with a KnopNode.",
      "rdfs:domain": "Knop/",
      "rdfs:range": "KnopNode/"
    },
    {
      "@id": "hasActiveKnopNode/",
      "@type": [
        "owl:ObjectProperty",
        "owl:FunctionalProperty"
      ],
      "rdfs:label": "has active knop node",
      "rdfs:comment": "Associates a Knop with its active KnopNode.",
      "rdfs:subPropertyOf": "hasKnopNode/"
    },
    {
      "@id": "KnopNodeComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "KnopNodeComponent",
      "rdfs:comment": "A KnopNodeComponent provides specific Node-level functionality."
    },
    {
      "@id": "hasKnopNodeComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNodeComponent/",
      "rdfs:label": "has KnopNodeComponent",
      "rdfs:comment": "Associates KnopNodes with their supporting components."
    },
    {
      "@id": "hasKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Knop/",
      "rdfs:label": "has Knop",
      "rdfs:comment": "Associates a referent (a thing or concept worth identifying) with its Knop."
    },
    {
      "@id": "ResourcePage/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PlatformElement/",
      "rdfs:label": "Resource page",
      "rdfs:comment": "Human-readable index.html pages that provide dereferenceability for non-file PlatformElements."
    },
    {
      "@id": "hasResourcePage/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "PlatformElement/",
      "rdfs:range": "ResourcePage/",
      "rdfs:label": "has ResourcePage",
      "rdfs:comment": "Associates a PlatformElement with its index.html resource page file."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "PlatformElement/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Flow",
      "rdfs:comment": "Evolvable containers for knop metadata, reference data, payload data."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Metadata Flow",
      "rdfs:comment": "Flow containing metadata about the mesh knop itself."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Reference Flow",
      "rdfs:comment": "Flow containing reference data about the thing or concept referred to by the mesh knop."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Payload flow",
      "rdfs:comment": "Flow containing version of the Dataset identified by a PayloadKnop."
    },
    {
      "@id": "hasFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "Flow/",
      "rdfs:label": "has Flow",
      "rdfs:comment": "Abstract property for linking mesh knops to their flows (metadata, reference, data)."
    },
    {
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has KnopMetadataFlow",
      "rdfs:comment": "Links a mesh knop to its metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "ReferenceFlow/",
      "rdfs:label": "has ReferenceFlow",
      "rdfs:comment": "Links a mesh knop to its reference flow."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "PayloadFlow/",
      "rdfs:label": "has PayloadFlow",
      "rdfs:comment": "Links a mesh knop to its payload flow."
    },
    {
      "@id": "Dataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "PlatformElement/",
        "dcat:Dataset"
      ],
      "rdfs:label": "Dataset",
      "rdfs:comment": "An abstract base class for all datasets in the Semantic Flow platform. sflo:Dataset is exactly dcat:Dataset but named for Semantic Flow authoring convenience."
    },
    {
      "@id": "FlowSlice/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Dataset/",
      "rdfs:label": "Flow Slice",
      "rdfs:comment": "One concrete dataset instance belonging to a Flow. A FlowSlice represents the Flow's data as a whole at a particular stage, not a subset of it."
    },
    {
      "@id": "hasFlowSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Flow/",
      "rdfs:range": "FlowSlice/",
      "rdfs:label": "has flow slice",
      "rdfs:comment": "Associates a Flow with one of its slices."
    },
    {
      "@id": "hasCurrentSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlowSlice/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "FlowSlice/",
      "rdfs:label": "has current slice",
      "rdfs:comment": "Associates a Flow with its current FlowSlice."
    },
    {
      "@id": "previousSlice/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "dcat:previousVersion",
      "rdfs:label": "previous slice",
      "rdfs:comment": "Links to the immediately-preceeding slice of this FlowSlice.",
      "rdfs:domain": "FlowSlice/",
      "rdfs:range": "FlowSlice/"
    },
    {
      "@id": "sliceSequence/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "FlowSlice/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "slice sequence",
      "rdfs:comment": "Monotonic integer providing the sequence number of this slice within its Flow. Usually 1 for the first-woven slice of a flow, 2 for the next, etc. Used for ordering and for detecting gaps or compaction at the flow level."
    },
    {
      "@id": "peerKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "Peer KnopNode",
      "rdfs:comment": "Associates a KnopNode with another KnopNode that is part of the same conceptual knop.",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNode/"
    },
    {
      "@id": "activePeerKnopNode/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "active peer KnopNode",
      "rdfs:comment": "Associates a KnopNode with the currently active KnopNode for a Knop.",
      "rdfs:subPropertyOf": "peerKnopNode/",
      "rdfs:domain": "KnopNode/",
      "rdfs:range": "KnopNode/"
    },
    {
      "@id": "PointersDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "KnopComponent/",
        "Dataset/"
      ],
      "rdfs:label": "Pointers dataset",
      "rdfs:comment": "A minimal dataset for bootstrapping."
    },
    {
      "@id": "currentMetaDownloadTTL/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "TurtleDistribution/",
      "rdfs:label": "current meta download Turtle",
      "rdfs:comment": "Relative IRI to fetch the current meta bytes for this knop. The representation is expected to be Turtle."
    },
    {
      "@id": "currentMetaDownloadJSONLD/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "JsonldDistribution/",
      "rdfs:label": "current meta download JSON-LD",
      "rdfs:comment": "Relative IRI to fetch the current meta bytes for this knop. The representation is expected to be JSON-LD."
    },
    {
      "@id": "KnopNodeDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "KnopNodeComponent/",
        "Dataset/"
      ],
      "rdfs:label": "KnopNode dataset",
      "rdfs:comment": "Base class for KnopNode-level operational datasets (inventory, opslog, backlinks)."
    },
    {
      "@id": "InventoryDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "Inventory dataset",
      "rdfs:comment": "KnopNode-level dataset describing the locally-present artifacts/resources/components at that KnopNode."
    },
    {
      "@id": "hasInventoryDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "InventoryDataset/",
      "rdfs:label": "has inventory dataset",
      "rdfs:comment": "Links a KnopNode to its InventoryDataset component."
    },
    {
      "@id": "OpslogDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "opslog dataset",
      "rdfs:comment": "KnopNode-level append-only dataset recording operations/events relevant to that KnopNode."
    },
    {
      "@id": "hasOpslogDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links a KnopNode to its OpslogDataset component."
    },
    {
      "@id": "BacklinksDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "KnopNodeDataset/",
      "rdfs:label": "backlinks dataset",
      "rdfs:comment": "KnopNode-level dataset describing backlinks to a published IRI corresponding to this KnopNode."
    },
    {
      "@id": "hasBacklinksDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopNodeComponent/",
      "rdfs:range": "BacklinksDataset/",
      "rdfs:label": "has backlinks dataset",
      "rdfs:comment": "Links a KnopNode to its BacklinksDataset component."
    },
    {
      "@id": "Distribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "dcat:Distribution",
      "rdfs:label": "Distribution",
      "rdfs:comment": "Semantic Flow abstract Distribution. The Distribution IRI is the access IRI; dereferencing yields the representation bytes. You can include dcat:downloadURL optionally, but treat it as a duplicated convenience assertion"
    },
    {
      "@id": "hasDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has Distribution",
      "rdfs:comment": "Links a Dataset to one of its Distributions.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "Dataset/",
      "rdfs:range": "Distribution/"
    }
  {
      "@id": "JsonldDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "JSON-LD Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for JSON-LD representations."
    },
    {
      "@id": "hasJsonldDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has JSON-LD Distribution",
      "rdfs:comment": "Links a FlowSlice to its JSON-LD Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "Dataset/",
      "rdfs:range": "JsonldDistribution/"
    },
    {
      "@id": "TrigDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "Trig Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for TriG representations."
    },
    {
      "@id": "hasTrigDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has TriG Distribution",
      "rdfs:comment": "Links a FlowSlice to its TriG Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "Dataset/",
      "rdfs:range": "TrigDistribution/"
    },
    {
      "@id": "TurtleDistribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Distribution/",
      "rdfs:label": "Turtle Distribution",
      "rdfs:comment": "A DCAT Distribution specifically for Turtle representations."
    },
    {
      "@id": "hasTurtleDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has Turtle Distribution",
      "rdfs:comment": "Links a FlowSlice to its Turtle Distribution.",
      "rdfs:subPropertyOf": "dcat:distribution",
      "rdfs:domain": "Dataset/",
      "rdfs:range": "TurtleDistribution/"
    }
  ]
}
ChatGPT said:

Your current sflo:Dataset/ definition is internally inconsistent with its own comment, and your distribution properties are still structurally wrong in a way that will keep biting you.
1) Your sflo:Dataset/ is not “exactly dcat:Dataset”

You wrote:

    Dataset/ rdfs:subClassOf [PlatformElement/, dcat:Dataset]

    comment: “sflo:Dataset is exactly dcat:Dataset but named for … convenience.”

Those conflict.

If it’s “exactly,” you need equivalence, not subclassing-plus-extra-superclass:

    If you keep PlatformElement/ as a supertype, then sflo:Dataset is strictly narrower than dcat:Dataset, because not all dcat:Dataset are sflo:PlatformElement.

Pick one and be explicit

A. Alias (recommended if you truly mean “exactly”):

    Dataset/ owl:equivalentClass dcat:Dataset

    Remove PlatformElement/ as a superclass (you can still make your concrete dataset classes be PlatformElement/).

B. Platform dataset (recommended if you want constraints like “must have a distribution”):

    Keep Dataset/ rdfs:subClassOf dcat:Dataset and PlatformElement/

    Update the comment to say it’s a Semantic Flow dataset, not “exactly DCAT”.

Given your stated desire (“I might require all Dataset/ to have ≥1 Distribution/”), you are squarely in B territory. Stop calling it “exactly dcat:Dataset”.
2) Don’t try to enforce “must have a distribution” in OWL here

You can express it with OWL restrictions, but it will be:

    open-world (non-asserted distributions don’t imply they don’t exist),

    reasoner-dependent,

    and you already have SHACL in context.

Use SHACL shapes for operational constraints like:

    every sflo:Dataset must have sflo:hasDistribution minCount 1

    every dataset must also have at least one of {hasJsonldDistribution, hasTrigDistribution, hasTurtleDistribution} etc.

So: keep ontology light; put enforcement in SHACL.
3) Your hasDistribution/ superproperty is good — but you didn’t actually use it

You defined:

    hasDistribution/ subPropertyOf dcat:distribution

    domain Dataset/

    range Distribution/

Good.

But then your specific properties (hasJsonldDistribution, hasTrigDistribution, hasTurtleDistribution) are still:

    subPropertyOf dcat:distribution (not of hasDistribution/)

This defeats the point of introducing hasDistribution/.
Fix

Make all of these:

    hasJsonldDistribution/ rdfs:subPropertyOf hasDistribution/

    hasTrigDistribution/ rdfs:subPropertyOf hasDistribution/

    hasTurtleDistribution/ rdfs:subPropertyOf hasDistribution/

…and then you can keep hasDistribution/ as your single SF hook that still maps to DCAT.
4) Your domains are still inconsistent with your own labels/comments

Your specific distribution properties say “Links a FlowSlice to its … Distribution.” but their domain is Dataset/. That’s fine if you mean “any dataset,” but then the comment should not say “FlowSlice”.

Either:

    change comments to “Links a Dataset…”, or

    narrow domains to FlowSlice/ if that’s actually your intent.

Right now it’s a documentation lie.
5) Your “abstract dataset” goal is not achieved

    “maybe that it's abstract (instances must at least declare some other concrete type)”

RDFS/OWL doesn’t have “abstract class” enforcement in the way you mean. You can approximate with SHACL:

    For sflo:Dataset nodes: require either (a) membership in one of your known concrete subclasses, or (b) rdf:type count ≥ 2 (Dataset plus something else)

But the “type ≥ 2” rule is brittle and will force weird modeling.

Better: define the actual concrete dataset classes you expect (you already have several), and treat sflo:Dataset as a grouping superclass, not something you expect to be asserted alone.
6) “Distribution IRI is the access IRI”

This is coherent as a platform convention, but then your naming is misleading:

    “currentMetaDownloadTTL” is not a “download” URL; it’s a pointer to a TurtleDistribution/ resource (which acts as the access IRI).

Rename to something like:

    currentMetaTurtleDistribution/

    currentMetaJsonldDistribution/

Otherwise you’ll have a permanent conceptual mismatch (“download URL” vs “distribution resource”).
7) Syntax/structure: your JSON is currently invalid

There’s a missing comma / brace in the pasted snippet around hasDistribution/ and JsonldDistribution/ (you have } { "@id": "JsonldDistribution/" ...).

Fixing this now matters because invalid JSON-LD will lead to silent tooling failures and you’ll misdiagnose it as ontology issues.
8) owl:versionInfo: "1" is okay, but you’re mixing schemes

Earlier you used "0.1.0". Now you’re using "1". Either is fine, but choose a scheme:

    If you will publish _payload/v1/..., then "1" is consistent.

    If you will publish _payload/v1/... but want semver, then "1.0.0" is the stable move.

What you should not do is bounce between "0.1.0" and "1" without a declared policy.
Minimal changes I would make to your current file

    Update sflo:Dataset comment to reflect it’s a platform dataset, not “exactly dcat:Dataset” (unless you switch to owl:equivalentClass and drop PlatformElement/ as superclass).

    Make hasJsonldDistribution/, hasTrigDistribution/, hasTurtleDistribution/ subproperties of hasDistribution/.

    Rename the “currentMetaDownload*” properties to “currentMeta*Distribution” (or change range back to xsd:anyURI, but you explicitly rejected that).

    Fix the invalid JSON-LD structure.

If you want, I can provide the exact patched JSON-LD fragments for those four edits (only the affected nodes), without rewriting the whole file.
You said:
I don't put much faith in semver for ontologies other than to indicate totally minor changes separately from everything else, which could potentially be considered breaking but it's very fuzzy because there's no API contract.. I've switched back to 0.0.1 but I'm hesitant because of the "v1" sequence convention. maybe I should go with "_s1" for sequence number to avoid versioning language even more.
ChatGPT said:

Your instinct (“SemVer is fuzzy for ontologies”) is correct. The mistake would be to let that fuzziness leak into IRI policy and create irreversible churn.
1) Don’t conflate three different things

You currently have (or are drifting toward) one string doing three jobs:

    Release identifier for humans (owl:versionInfo)

    Stable identity for a specific published artifact (owl:versionIRI)

    Filesystem/layout sequencing convention (/v1/… folders)

Those should be allowed to diverge.
2) Keep /vN/ in paths, but stop treating it as “SemVer”

If /v1/ is already part of your platform’s general publishing convention (datasets/flows/slices/etc.), keep it. It is a sequence bucket, not a claim about compatibility.

Then:

    owl:versionInfo: can be whatever human label you want ("0.0.1", "2025-12-19", "s1").

    owl:versionIRI: points at the concrete artifact under /v1/… regardless of what owl:versionInfo says.

This avoids the false promise of SemVer while preserving a simple monotonic namespace layout.
3) If you want to avoid “version language”, use it only in the label, not the path

Switching /v1/ to /_s1/ buys you very little and costs you a lot:

    It’s nonstandard and will confuse every reader.

    You’ll still have the same underlying concept (a monotonically increasing series).

    You’ll now have to explain two conventions: “sequence but not version.”

So: don’t rename the folder unless you’re willing to standardize _sN across the entire platform (flows, slices, catalogs, etc.). If you only do it for the ontology, it becomes a one-off.
4) A workable policy that matches what you actually believe

Policy: “Monotonic artifact sequence + optional human label.”

    Artifact path: /.../_payload/v{N}/semantic-flow-ontology.{ext}

        where N increments on each published artifact.

    owl:versionIRI: that exact artifact IRI.

    owl:versionInfo: a human-oriented label that can be:

        "0.0.1" if you like “early stage”

        or "s1" if you want to be honest about “sequence”

        or a date ("2025-12-19") if you want “what changed when” without compatibility claims.

If you do this, you can keep /v1/ while setting owl:versionInfo to "s1" and you’ve achieved the “no SemVer promise” goal without inventing a new path convention.
5) One critical consistency rule (don’t violate this)

Once a version IRI is minted and published, never change what bytes it serves. If you need to correct something, publish a new v{N+1} artifact and link it (e.g., with dcat:previousVersion/owl:priorVersion/your own pointer).

That rule matters far more than whether the label is SemVer-like.
Recommendation

    Keep /v1/ (treat it as monotonic release bucket, not SemVer).

    Set owl:versionInfo to "s1" (or a date) if you want to avoid SemVer signaling.

    Keep 0.0.1 only if you want a separate human notion of maturity; otherwise it will constantly fight your /vN/ sequence.

If you tell me which of these you want owl:versionInfo to communicate—(a) maturity, (b) ordering, or (c) timestamp—I’ll give a single consistent mapping for versionInfo, versionIRI, and any “latest/current” pointers without introducing more terms.


ChatGPT can make mistakes. Check important info.
