---
id: hl2q7xntqg9mzjkxfle0p5g
title: 2026 01 19 Namespace Model Clarification 3
desc: ''
updated: 1768845082306
created: 1768845077884
---

Mesh Resource Taxonomy Draft
You didn't include some properties, like hasReferenceData. Anything else missing? Please open the ontology in a separate canvas.
Semantic Flow Ontology Draft
what's the difference between hasReferenceData and hasReferenceSource?
Semantic Flow Ontology Draft
It feels like hasReferenceSource is the more general thing. But I don't understand your example "You also extracted part of an external HR dataset." -- extracted it where? 

Maybe we should model ReferencesSources as relators. They could have their own fragment-identifier-based @ids and would use properties: sourceUrl (range anyURI), sourceIdentifier (range rdfs:Resource), referenceDate (could be re-used every time the reference is re-verified), referenceType (an enum of Nomen, Flow, Slice, AbstractFile, or LocatedFile), isCanonical, shouldBeIntegratedIntoResourcePage. Anything else?

aside: we talked about making a subclass of LocatedFile that specifically marks it as a child of an AbstractFile. That would help software realize "I can check the parent IRI for useful information". Maybe we call it ConcretizedFile or RealizedFile or ???
Semantic Flow Ontology Draft
Meaning A doesn't make sense to me. If you put the evidence/data in its own payload Knop (good), its KnopMetadataFlow can talk about where it came from. I want the Nomen references to all be actual sources of Reference Data, each of which may have its own provenance but that's not up to the Nomen to track. 

Regarding "Domain-side usage:" - do you see an advantage for doing that, vs having a property in the relator, e.g. <#ref1> sflo:forNomen <nomen-handle> ?

Regarding sflo:target's denotation, it's fine to use an identifier term, as long as the thing being denoted by the identifier term is a source of reference data. If your source is the webpage returned by an identifier (e.g. the target does not have a clean httprange-14 separation), best to use anyURI. 

For sflo:role, how do you mechanically intend to specify the enum? Presumably you can have multiple values simultaneously. What does provenanceOnly mean?

Maybe referenceDate is ambiguous, but if we just did verifiedAt, that could keep full history without a separate relator, but yes, it would lack assertedBy. So let's just do lastVerifiedBy.

Instead of "intendedTargetKind", let's do "expectedTargetKind". It won't be wrong constantly, IRIs aren't supposed to change their kind often, and ideally never. 

extractionQuery might be useful, but if not specified we can resort to "any statement that matches mesh-local or anticipatedPublishedIri dcterms:source is perfect for the "ontology extraction use case". 

regarding confidence, are we accomplishing co-denotation in these ReferenceLinks too? Pros and Cons please.
Semantic Flow Ontology Draft
Regarding 1) I like the "contained" option as it keeps everything bundled together, but sure let's call the properties hasReferenceLink and referenceLinkForNomen ? 

You keep reminding me about the drawbacks of AnyURI. But it's more important to me to honor the single referent principle. And in this case we are talking about a located web resource, so a URL is entirely appropriate. We'll offer both. Instead of what you have, let's say “If target is present, it is expected to denote a data source; if only targetUriLiteral is present, it is expected to locate some content.” For SF-compliant sources we can locate content from a denoting IRI. 

Regarding 3), yes. Maybe add a ReferenceRole for "Deprecated". 

And speaking of Enums, we're going to have one for  expectedTargetKind. We want to have Nomen, Flow, Slice, AbstractFile, ArtifactLocatedFile, LocatedFile, and maybe that's it. If it's none of those things, applications try to extract reference data will have to do best effort, and there's no sense having an "Other" or "Unknown"? So since those things are already defined, it'd be nice not to have to redefine them. Can we just add a @type of TargetKind (to the existing @type of rdfs:Class). That might not be Kosher with OWL DL, but I don't really care as long as it works with SHACL.
Semantic Flow Ontology Draft
Yes, please edit the canvas, I think we're agreed on all points. 
Semantic Flow Ontology Draft
Could you put together an up-to-date summary of documentation changes/suggestions you've made that are still relevant?
Semantic Flow Ontology Draft
Regarding "/<nomen>/ is the designator IRI that denotes something else (person, artifact, etc.)" -- we need a good word for the "parent IRI" that a Nomen (as identified by a _nomen handle) attaches to.
Semantic Flow Ontology Draft
What was our term for calculating which ArtifactLocatedFile to use given a Flow or Slice as a target? Not negotiation, but ??? 
Semantic Flow Ontology Draft
Should Knops be included in Artifact Resolution, ie. if a Knop is specified (directly or via a Nomen), we should assume we want the PayloadFlow's WorkingSlice, and choose a support AbstractFile mimetype?
Semantic Flow Ontology Draft
I'm working on a better definition for Nomen:

* **Nomen** *(AbstractResource; TargetKind)* — A human-facing Designator IRI (e.g., /people/alice/) intended to denote a discourse-worthy thing, with a corresponding system-facing NomenHandle (e.g., /people/alice/_nomen/) that denotes the IRI-as-an-identifier, and a NomenMetadataFlow...

But now I'm thinking there might be two flows. NomenMetadataFlow is fine for things about Nomen (e.g., most statements have a NomenHandle as the subject); But should we also have a DesignatorFlow where "denotes" lives, and probably ReferenceLinks live? Presumably Reference Data is about the thing itself, i.e., have the DesignatorIRI as the subject or object). It seems like it might be slightly clarifying, or at least consistent with Knops where the metadata has its own Flow.
Semantic Flow Ontology Draft
Give me a "use-case.semantic-flow-core-self-containment" wikipage that describes the functionality we'll need to extract the terms, create canonical links to the source, and generate self-documenting ResourcePages for all the Nomen. 

For the first time in forever, I'm excited by what this is going to be able to do. 
Semantic Flow Ontology Draft
Actually, the eventual goal is a general-purpose ontology browser. If that changes your doc, put it in a canvas.

What do you mean by " Not “perfect truth”; provenance and extraction must be explicit and reproducible, not magically inferred."
Use-case
I just realized the ontology doesn't have a "expectedPublishUrl" -- let's make that an anyURI too?
Use-case
Wait, where does the context entry go?
Use-case
for domain, could we use SemanticFlowResource? I added it to ResourcePage and ArtifactLocatedFile? I still don't understand the purpose of the @context item. Is that only if I want to support use of "expectedPublishUrl" without the slash?
Use-case
Did you ever generate a wikipage for artifact resolution? If not, please do, mention some examples, like with ReferenceLinks you can target a Flow, a Slice, an AbstractFile or a ArtifactLocatedFile, and end up getting the right URL for the data.
Concept
Can you answer these findings and answer the questions. I'm pretty sure the metonymy finding is wrong-headed:

Findings

Medium: Flow is constrained to KnopComponent, but you now need flow-shaped metadata under NomenHandle and mesh-level handles, so the class model blocks those placements. Consider generalizing Flow beyond KnopComponent or adding a host abstraction. semantic-flow-ontology.jsonld (line 321) semantic-flow-ontology.jsonld (line 498) semantic-flow-ontology.jsonld (line 508)
Medium: There’s no property linking a NomenHandle to its NomenMetadataFlow; only hasKnopMetadataFlow exists, so Nomen metadata can’t be expressed cleanly in the graph. semantic-flow-ontology.jsonld (line 508) semantic-flow-ontology.jsonld (line 535)
Low: Metonymy (handle stands in for the Designator/Nomen) isn’t captured explicitly; denotes has domain NomenHandle, which reads as the handle itself denoting the referent. Consider a clarifying property or stronger comment to avoid semantic confusion. semantic-flow-ontology.jsonld (line 157) semantic-flow-ontology.jsonld (line 175)
Low: RdfGraph and NamedGraph are standalone with no tie to TargetKind or RdfDataset; if you’re unsure about their near-term role, consider dropping them for now or formalizing their relation. semantic-flow-ontology.jsonld (line 553) semantic-flow-ontology.jsonld (line 559)
Questions / Suggestions

I’d add NomenInventoryDataset as a non-flow index: subclass RdfDataset + MeshComponent (and optionally TargetKind if you want it as an expectedTargetKind). Want me to add it?
“Designator” / “Designator IRI” works well; I’d treat it as a label on Nomen (docs + skos:altLabel) rather than a new class.
The new use-case doc has duplicate front matter and a blank heading; worth cleaning to keep Dendron happy. use-case.semantic-flow-core-self-containment.md (line 1) use-case.semantic-flow-core-self-containment.md (line 48)
Dendron hierarchy proposal: sflo.class.<PrimaryParent>.<ClassName> + sflo.property.<PropName> + sflo.use-case.*, with aliases in front matter for older mesh-resource.* names and a single “primary parent” per class when multiple inheritance exists.
Regarding Metonymy, "denotes has domain NomenHandle" is correct right? Because the NomenHandle denotes the Nomen. It is the Nomen's IRI. The Designator IRI is the referent's IRI, i.e. Alice. It's not Alice that denotes something, it's the Nomen. To talk about the Nomen, we use the NomenHandle.  I thought the finding was saying it's wrong. 
Where does dcterms:source fit in. Is it a superproperty somewhere?
I was imagining dcterms:source would specify that "we got the rdfs:label and rdfs:comment for the ontology term from the ontology." i.e., no derived datasets

