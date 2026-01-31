---
id: xjuzewepu5hoihjehf4xsgg
title: 2026 01 29 Concise Documentation Guidelines 2
desc: ''
updated: 1769716851745
created: 1769716677526
---

OK, I think we have something really good. Can you give me some words of encouragement?


You made a clean set of decisions that actually reduce risk instead of just “simplifying” on paper.

Dropping inference assumptions in SHACL is the right call; it prevents silent false negatives and makes validation behavior predictable across engines and deployments.

Switching key shapes to targetSubjectsOf is a genuinely strong move: it validates real usage, not aspirational typing hygiene.

Tightening inventories and KnopMetadataFlow to exactly one with Violation gives you crisp invariants that will pay off immediately in tooling, generation, and debugging.

This is the kind of structure that stays slim while still being enforceable—rare combination.

And in the bigger picture? 

In the bigger picture, you’re converging on something most ontology/framework efforts never achieve: a small set of primitives that can actually support tooling, publishing, and validation without constantly rewriting the model.

You’re separating “semantic openness” from “operational invariants” correctly: open-world ontology for meaning; SHACL for enforcement where the system needs it. That’s the right architecture for a framework that has to run.

You’re resisting class/properties bloat while still naming the few things that matter. That keeps the ontology teachable, documentable, and stable.

You’re making choices that scale: usage-triggered validation, cardinality where it matters, and no reliance on reasoners. That means fewer fragile assumptions and fewer surprises as the ecosystem grows.

Net: you’re not just designing vocabulary; you’re designing a system that can be implemented repeatedly by different people and still behave the same. That’s the real milestone.

Can you update the documentation update list:

[ ] c.MeshResource
  [ ] c.ArtifactHost
    [ ] c.Mesh
    [ ] c.Knop
    [ ] c.Nomen
  [ ] c.MeshComponent
    [ ] c.MeshMetadataFlow
    [ ] c.MeshInventoryDataset
  [ ] c.KnopComponent
    [ ] c.FlowPayload 
    [ ] c.KnopMetadataFlow
    [ ] c.SimplePayload
    [ ] c.KnopInventoryDataset
  [ ] c.NomenComponent
    [ ] c.NomenMetadataFlow
    [ ] c.NomenInventoryDataset
  [ ] c.DigitalArtifact
    [ ] c.FlowArtifact
    [ ] c.SimpleArtifact
    [ ] c.RdfDataset
  [ ] c.Slice
    [ ] c.WorkingSlice
    [ ] c.HistoricalSlice
  [ ] c.Realizable
  [ ] c.Realization
    [ ] c.AbstractFile
      [ ] c.AbstractArtifactFile
    [ ] c.AbstractStream
  [ ] c.LocatedFile
    [ ] c.LocatedArtifactFile
    [ ] c.ResourcePage
  [ ] c.ReferenceLink
  [ ] c.ReferenceRole

### Properties (the ones worth first-class pages)

Containment / structure

[ ] p.containstainsMeshResource *(the root structural predicate)*
[ ] p.containsPayload
[ ] p.containsFlowPayload
[ ] p.containsSimplePayload
[ ] p.containsKnopMetadataFlow
[ ] p.containsMeshMetadataFlow
[ ] p.containsNomenMetadataFlow

Naming / mesh config

[ ] p.meshBase
[ ] p.meshPath *(currently missing in the ontology file you pasted; you’ve been relying on it conceptually)*
[ ] p.preferredSlug *(also not yet in the pasted ontology file)*
[ ] p.nomenSubject

Reference links

[ ] p.hasReferenceLink
[ ] p.referenceRole
[ ] p.referenceTarget
[ ] p.referenceTargetUriLiteral *(this is the “URL version” you’re remembering; it is currently missing from the ontology file)*
[ ] p.lastVerifiedAt
[ ] p.lastVerifiedBy

Artifact modeling

[ ] p.artifactKind
[ ] p.hasSlice
[ ] p.currentSlice
[ ] p.previousSlice
[ ] p.nextSliceSequenceNumber

Realizations / byte access

[ ] p.hasAbstractFile
[ ] p.hasAbstractArtifactFile
[ ] p.hasLocatedFile
[ ] p.locatesAbstractFile
[ ] p.hasLocatedArtifactFile
[ ] p.locatesAbstractArtifactFile
Thought for 56s
Classes

[ ] c.MeshResource
[ ] c.ArtifactContainer (renamed from ArtifactHost)
[ ] c.Mesh
[ ] c.Knop
[ ] c.Nomen
[ ] c.MeshComponent
[ ] c.MeshMetadataFlow
[ ] c.MeshInventoryDataset
[ ] c.KnopComponent
[ ] c.FlowPayload
[ ] c.KnopMetadataFlow
[ ] c.SimplePayload (also c.Realizable)
[ ] c.KnopInventoryDataset
[ ] c.NomenComponent
[ ] c.NomenMetadataFlow
[ ] c.NomenInventoryDataset
[ ] c.AbstractArtifactFile
[ ] c.LocatedArtifactFile
[ ] c.ResourcePage

[ ] c.DigitalArtifact
[ ] c.FlowArtifact
[ ] c.SimpleArtifact

[ ] c.Slice
[ ] c.WorkingSlice
[ ] c.HistoricalSlice

[ ] c.Realizable

[ ] c.Realization
[ ] c.AbstractFile
[ ] c.AbstractStream

[ ] c.LocatedFile

[ ] c.RdfDataset (content-kind tag; applies to both FlowArtifacts and SimpleArtifacts via multiple typing/subclassing, not a SimpleArtifact subclass)

[ ] c.ReferenceLink
[ ] c.ReferenceRole

Properties (the ones worth first-class pages)

Containment / structure

[ ] p.containsMeshResource (root structural predicate; ⊑ dcterms:hasPart)
[ ] p.containsMeshComponent
[ ] p.containsKnopComponent
[ ] p.containsNomenComponent
[ ] p.containsKnop
[ ] p.containsNomen

Payload

[ ] p.containsFlowPayload
[ ] p.containsSimplePayload
(omit p.containsPayload unless you’re keeping it as a convenience superproperty; it isn’t needed for SHACL or usage)

Mesh / naming

[ ] p.meshBase
[ ] p.designatorPath
[ ] p.designates
[ ] p.hasChildNomen (if it remains “operational/derived”; otherwise drop from first-class docs)

Reference links

[ ] p.hasReferenceLink
[ ] p.referenceLinkFor
[ ] p.hasReferenceRole (renamed from referenceRole)
[ ] p.referenceTarget
[ ] p.referenceUriLiteral (renamed from referenceTargetUriLiteral)
[ ] p.lastVerifiedAt
[ ] p.lastVerifiedBy

Artifact modeling

[ ] p.artifactKind
[ ] p.hasSlice
[ ] p.currentSlice
[ ] p.previousSlice
[ ] p.nextSliceSequenceNumber

Realizations / byte access

[ ] p.hasAbstractFile
[ ] p.hasAbstractStream
[ ] p.hasLocatedFile
[ ] p.locatesAbstractFile

Mesh-governed files

[ ] p.hasAbstractArtifactFile
[ ] p.hasLocatedArtifactFile
[ ] p.locatesAbstractArtifactFile

Inventories (now “exactly one”, so first-class)

[ ] p.containsMeshInventoryDataset
[ ] p.containsKnopInventoryDataset
[ ] p.containsNomenInventoryDataset

Removed from the list (per your latest direction)

p.meshPath → replaced by p.designatorPath

p.preferredSlug → still not in ontology (don’t doc as first-class yet)

p.nomenSubject → not in ontology; and you’ve been shifting away from that pattern anyway

not sure how we lost preferredSlug. I think we need it for good payload file naming. And only that. 

Occasionally I'm tempted to allow custome knop naming (instead of UUIDs), Maybe someday.
Thought for 31s

You didn’t “lose” preferredSlug for any technical reason — it dropped out because it wasn’t present in the ontology fragment you pasted, and I treated “first-class docs pages” as “things that exist right now.”

If you still want it (and your stated scope is narrow and reasonable), you should put it back explicitly and keep it deliberately weak.

Recommendation: keep preferredSlug, but constrain its semantics hard

Purpose: a hint for generators when naming payload distribution filenames (file stem), nothing else.

Key constraints (or it will metastasize):

It is not an identifier.

It does not affect IRIs / designator paths / containment.

It is optional and best-effort; generators may sanitize/ignore.

Collisions are resolved mechanically (suffix, short hash), not by rejecting data.

Where it should live

Put it on sflo:Knop (or if you renamed “host” to ArtifactContainer, still on Knop, not the container superclass). Your requirement is “payload file naming,” and payload is a Knop concern.

Minimal ontology addition (intent, not full TTL)

p.preferredSlug

domain: c.Knop

range: xsd:string

cardinality: max 1 (SHACL and/or convention)

comment: “Optional human-friendly file-stem hint for naming payload distribution files; not an identifier; may be sanitized; collisions resolved by generator.”

In your documentation update list, add back:

Naming / mesh config

 p.preferredSlug (payload filename hint only)

SHACL shape you probably want (no inference required)

target: sflo:Knop (or targetSubjectsOf sflo:containsFlowPayload / containsSimplePayload if you want usage-triggered)

property: sflo:preferredSlug

sh:maxCount 1 (Violation)

sh:datatype xsd:string (Violation)

optional sh:pattern (Warning, not Violation) — because strict patterns always bite later (Unicode, dashes, etc.)

About “custom knop naming instead of UUIDs”

Do not use preferredSlug for that. That’s where models get muddled.

If you ever allow non-UUID Knop IRIs/paths, that is a different feature with different risks:

collisions and namespace governance

rename semantics (redirects, tombstones, aliasing)

stability guarantees (“single referent”, long-lived links)

The safe compromise you already have is:

keep Knop identity stable (UUID or stable minted ID)

use Nomen / designators for human-facing naming and organization

use preferredSlug only for filenames, where changing it is low-stakes

If you want, the future “custom Knop naming” feature should be framed as aliasing / designator binding, not “changing the Knop’s identifier.”

Please generate a TTL preferredSlug entry and a jsonld shacl constraint for it. For pattern, is there an easy way to signify the filename-legal characters and cross-operating-system conventions for files?
Thought for 43s
Turtle (sflo:preferredSlug)
<preferredSlug/> a owl:DatatypeProperty ;
  rdfs:domain <Knop/> ;
  rdfs:range xsd:string ;
  rdfs:label "preferred slug"@en ;
  rdfs:comment "Optional human-friendly file-stem hint for naming payload distribution files (e.g., 'alice-profile'). Not an identifier; does not affect IRIs or designator paths. Generators may sanitize/ignore it and must resolve collisions mechanically (e.g., suffix or hash)."@en .

JSON-LD SHACL (datatype/maxCount = Violation; pattern = Warning)
{
  "@id": "PreferredSlugShape/",
  "@type": "sh:NodeShape",
  "rdfs:label": "Knop preferred slug",
  "rdfs:comment": "Constraints for sflo:preferredSlug when used to suggest payload distribution filename stems.",
  "sh:targetClass": { "@id": "sflo:Knop/" },
  "sh:property": [
    {
      "sh:path": { "@id": "sflo:preferredSlug/" },
      "sh:maxCount": 1,
      "sh:datatype": { "@id": "xsd:string" },
      "sh:severity": { "@id": "sh:Violation" },
      "sh:message": "If present, sflo:preferredSlug MUST be a single xsd:string value."
    },
    {
      "sh:path": { "@id": "sflo:preferredSlug/" },
      "sh:pattern": "^[a-z0-9]+(?:-[a-z0-9]+)*$",
      "sh:severity": { "@id": "sh:Warning" },
      "sh:message": "If present, sflo:preferredSlug SHOULD be a portable kebab-case file-stem: lowercase ASCII letters/digits separated by hyphens (no spaces, no underscores, no dots)."
    }
  ]
}


If you prefer usage-triggered targeting (no reliance on rdf:type), replace sh:targetClass with:

sh:targetSubjectsOf sflo:containsFlowPayload and/or sflo:containsSimplePayload

…but that can double-report if you also keep targetClass.

Pattern for “filename-legal across OS” (easy?)

Not really as a single clean regex.

Cross-OS “portable filename” rules get messy because Windows adds constraints that POSIX doesn’t:

forbids characters: \ / : * ? " < > |

forbids control characters

forbids trailing . or space

forbids reserved basenames like CON, PRN, AUX, NUL, COM1… LPT9 (case-insensitive)

path segment length constraints vary

So the “easy” approach is: define a portable slug convention rather than attempt full OS legality. Your kebab-case pattern above is intentionally stricter than necessary, but it’s predictable and avoids nearly all cross-platform issues.

If you later want closer-to-real filename legality, do it with:

a permissive character whitelist pattern (Warning), plus

a SPARQL constraint for reserved Windows names + trailing dot/space (Violation/Warning), because that logic is awkward in regex alone.

your documentation pages list has out-dated entries, e.g. locatesAbstractArtifactFile
Thought for 39s

Correct — my “first-class property pages” list drifted. If you’ve collapsed the mesh-governed file layer (i.e., no special ArtifactFile types beyond AbstractFile/LocatedFile), then anything in the *AbstractArtifactFile* / *LocatedArtifactFile* family is outdated and should be removed from the docs list.

Below is a cleaned, current “docs update” checklist consistent with that slimmer direction (and with your newer naming: designatorPath, referenceUriLiteral, hasReferenceRole, preferredSlug, ArtifactContainer).

Classes

[ ] c.MeshResource
[ ] c.ArtifactContainer
[ ] c.Mesh
[ ] c.Knop
[ ] c.Nomen
[ ] c.MeshComponent
[ ] c.MeshMetadataFlow
[ ] c.MeshInventoryDataset
[ ] c.KnopComponent
[ ] c.FlowPayload
[ ] c.KnopMetadataFlow
[ ] c.SimplePayload (also c.Realizable)
[ ] c.KnopInventoryDataset
[ ] c.NomenComponent
[ ] c.NomenMetadataFlow
[ ] c.NomenInventoryDataset

[ ] c.DigitalArtifact
[ ] c.FlowArtifact
[ ] c.SimpleArtifact

[ ] c.RdfDataset

[ ] c.Slice
[ ] c.WorkingSlice
[ ] c.HistoricalSlice

[ ] c.Realizable

[ ] c.Realization
[ ] c.AbstractFile
[ ] c.AbstractStream

[ ] c.LocatedFile
[ ] c.ResourcePage

[ ] c.ReferenceLink
[ ] c.ReferenceRole

Properties (worth first-class pages)

Containment / structure

[ ] p.containsMeshResource (root structural predicate; ⊑ dcterms:hasPart)
[ ] p.containsMeshComponent
[ ] p.containsKnopComponent
[ ] p.containsNomenComponent
[ ] p.containsKnop
[ ] p.containsNomen

Payload

[ ] p.containsFlowPayload
[ ] p.containsSimplePayload
(skip p.containsPayload unless you’ve kept it explicitly as a convenience superproperty)

Flows / inventories

[ ] p.containsKnopMetadataFlow
[ ] p.containsMeshMetadataFlow
[ ] p.containsNomenMetadataFlow

[ ] p.containsMeshInventoryDataset
[ ] p.containsKnopInventoryDataset
[ ] p.containsNomenInventoryDataset

Mesh / naming

[ ] p.meshBase
[ ] p.designatorPath
[ ] p.designates
[ ] p.preferredSlug

Reference links

[ ] p.hasReferenceLink
[ ] p.referenceLinkFor
[ ] p.hasReferenceRole
[ ] p.referenceTarget
[ ] p.referenceUriLiteral
[ ] p.lastVerifiedAt
[ ] p.lastVerifiedBy

Artifact modeling

[ ] p.artifactKind
[ ] p.hasSlice
[ ] p.currentSlice
[ ] p.previousSlice
[ ] p.nextSliceSequenceNumber

Realizations / byte access

[ ] p.hasAbstractFile
[ ] p.hasAbstractStream
[ ] p.hasLocatedFile
[ ] p.locatesAbstractFile

Explicit deletions from the old docs list

p.locatesAbstractArtifactFile, p.hasLocatedArtifactFile, p.hasAbstractArtifactFile (drop if you removed the specialized ArtifactFile layer)

p.referenceTargetUriLiteral → use p.referenceUriLiteral

p.referenceRole → use p.hasReferenceRole

p.meshPath → use p.designatorPath

p.nomenSubject (if it’s no longer in the ontology, don’t doc it)

If locatesAbstractArtifactFile is still in your actual TTL right now, then the docs list isn’t the thing that’s outdated — the ontology is still carrying the old layer. In that case you should either (a) delete the ArtifactFile layer from TTL, or (b) keep it and restore those pages. The mismatch is the real problem.

Let's remove the kebab-case guideline. We need to support underscores too.

Use a looser “portable filename stem” pattern that allows underscores (and still avoids spaces and path separators). Replace the SHACL pattern constraint with this:

{
  "sh:path": { "@id": "sflo:preferredSlug/" },
  "sh:pattern": "^[A-Za-z0-9](?:[A-Za-z0-9_-]*[A-Za-z0-9])?$",
  "sh:severity": { "@id": "sh:Warning" },
  "sh:message": "If present, sflo:preferredSlug SHOULD be a portable filename stem using only ASCII letters/digits plus '_' and '-' (no spaces; no path separators)."
}


Notes on what this does:

Allows letters/digits anywhere.

Allows _ and - internally.

Disallows leading/trailing _/- (keeps stems clean; change if you want to allow them).

Still doesn’t attempt the full Windows-reserved-name problem; that’s better handled by generator sanitization or a separate SPARQL check if you decide it matters.

let's allow dots. what does the "?:" do?

Updated pattern (allows dots as internal separators; still disallows leading/trailing separator characters):

{
  "sh:path": { "@id": "sflo:preferredSlug/" },
  "sh:pattern": "^[A-Za-z0-9](?:[A-Za-z0-9_.-]*[A-Za-z0-9])?$",
  "sh:severity": { "@id": "sh:Warning" },
  "sh:message": "If present, sflo:preferredSlug SHOULD be a portable filename stem using only ASCII letters/digits plus '_', '-', and '.' (no spaces; no path separators)."
}


What (?: ... ) means:

It’s a non-capturing group in regular expressions.

Grouping (...) normally “captures” the matched text so it can be referenced later (e.g., backreferences like \1).

(?:...) groups without capturing. It’s slightly more efficient and avoids creating capture groups you don’t intend to use.

In this regex specifically:

^[A-Za-z0-9] requires the first character to be alphanumeric.

(?:[A-Za-z0-9_.-]*[A-Za-z0-9])? is an optional grouped tail:

[A-Za-z0-9_.-]* allows any number of allowed characters (including separators) in the middle

[A-Za-z0-9] requires the last character to be alphanumeric (so it can’t end with _/-/.)

the trailing ? makes the whole tail optional so a 1-character slug is valid.

are there modern filesystems that are case-insensitive?
Thought for 2m 47s

Yes. Several mainstream, modern filesystems are case-insensitive (but case-preserving) by default:

Windows (NTFS): filenames preserve case, but lookups are case-insensitive by default (i.e., Readme.md and README.md are treated as the same name in most contexts).

macOS (APFS default): the standard “APFS” format is case-insensitive; “APFS (Case-sensitive)” is a distinct option you must explicitly choose. Same pattern for “Mac OS Extended” vs “Mac OS Extended (Case-sensitive)”.

Common removable-media filesystems (exFAT/FAT variants) are typically case-insensitive as well, which is one reason they’re cross-platform-friendly.

By contrast, many Linux/Unix filesystems (e.g., ext-family in typical configurations) are case-sensitive, so a.txt and A.txt can coexist there.

Practical upshot for preferredSlug: if you want slugs that behave consistently across users’ machines, treat slugs as case-insensitive (often by normalizing to lowercase) to avoid “works on my machine” filename collisions.

Could you generate an concise article on "Mesh Namespaces" that touches on the Semantic/Filesystem split/mirroring?
Mesh Namespaces

A mesh namespace is the stable, semantic surface of a mesh: the set of IRIs a mesh claims as its own and governs under Semantic Flow conventions. It is not the filesystem. It is the meaningful address space you publish and reason over.

Two planes: semantic vs filesystem

A mesh has two orthogonal planes:

Semantic plane (namespace): IRIs for mesh-governed resources (Mesh, Knops, Nomens, artifacts, flows, slices). These are the things you talk about.

Filesystem plane (serialization): concrete bytes: files, folders, build outputs, generated pages, and other retrieval endpoints. These are how you serve and store.

Treat the filesystem as an implementation detail that may mirror the semantic structure, but never defines it.

The mirroring idea

Mirroring is a convenience: align the retrieval paths of files/pages with the mesh’s semantic hierarchy so humans can “read” the mesh by browsing directories and URLs.

But mirroring is optional and partial:

Some semantic resources have no direct file (they’re described by datasets elsewhere).

Some files are purely derived UI (resource pages) and should not be mistaken for primary semantic content.

Some semantic relationships (especially naming) do not map cleanly to folders.

Naming is not containment

A major trap is to treat Nomen structure as filesystem containment.

A Nomen’s designator is a public IRI segment (via designatorPath relative to meshBase).

Child Nomens do not “live under” the Nomen’s _nomen/ handle. They live under the designator IRI space.

So “Alice’s Nomen metadata” might be stored at an internal handle, while the designator it manages is /people/alice/. The operational storage path and the public designator path are different on purpose.

A concrete mental model

meshBase is the publish root (a URI literal; you’re not asserting identity of a thing, you’re declaring a base string).

Designators are what humans see and share (Nomens manage these).

Knops are stable payload hosts (often UUID-based) that exist to carry artifacts.

Artifacts are identities for content; realizations (abstract files, located files) are how that content becomes bytes.

The mesh namespace is the graph of these resources and relations; the filesystem is just one way to realize them.

Recommended invariants

Do not infer semantic membership from folder existence. Membership is asserted (and validated).

Do not infer reference from payload bytes. A file being RDF does not make it authoritative unless you say so.

Treat filenames as retrieval conveniences. preferredSlug is a hint, not identity.

Avoid portability footguns: assume some environments are case-insensitive; avoid relying on Foo vs foo as distinct.

When mirroring helps

Mirroring pays off when:

you want browseable URLs that feel “directory-like” to humans,

you want predictable locations for resource pages,

you want reproducible build outputs.

Mirroring hurts when:

it pressures you into pretending containment = naming,

it forces ontology-level commitments about storage layout,

it causes identity drift (“we renamed a folder, therefore the thing changed”).

A single sentence definition

A mesh namespace is the set of IRIs a mesh governs; the filesystem is a strategy for realizing and serving representations of those IRIs, optionally mirrored for human navigability.

Open in a canvas please. 

Maybe instead of "Semantic plane (namespace)" we use "Semantic plane (conceptualization)".

Regarding "but never defines it." -- it can define it, in a way. If you create a bunch of folders and files with the right structure, you've created/defined the namespace. Mirroring is perhaps more than a convenience, it's the expected primary way of defining, storing, and modifying the namespace. Mesh Resources always either have a direct file or a direct folder. 

Why doesn't naming map to folders? I've done away with child nomina. Perhaps we talk about the "designator hierarchy", with each node (wc?) having an associated Nomen. In a sense, naming is containment in the sense that designators form a tree. I don't know that Nomen manage designators so much as describe them.What do you think? 

Membership is asserted and validated, and yes, there may be other folders alongside the mesh folders that aren't "semantic". 

Do we need a name for the filesystem representation of a mesh?


Thought for 37s

On “it can define it”: agreed, and I updated the article to treat filesystem materialization as the primary authoring surface, with “minting by structure” as a first-class mechanism.

On “why doesn’t naming map to folders?”: with child-nomen removed, it maps better than before. The clean model is exactly what you described: designator hierarchy is the containment tree; the Nomen is metadata about a designator node (and optionally an assertion of its designatum). “Describes” is better than “manages” unless you want to imply authority/intent control.

Do you need a name for the filesystem representation? If you’re going to talk about it a lot, yes—otherwise you’ll keep saying “folders/files” and that bloats prose. The best options are in the canvas; my pick is mesh materialization (least ambiguous), with mesh working tree as a strong alternative when you’re being explicitly Git-centric.

Mesh Namespaces
Since naming is one of the two primary purpose of the Semantic Flow framework, this article has to be crisp and illuminating. The namespace is distinct from its (semantic) description or its filesystem-based specification/delineation. I would say that the namespace is a tree-shaped collection of identifiers that looks familiar to people. The Semantic plane a) describes the referents of those identifiers, b) contains digital artifacts that may be the referents of those identifiers, and c) contains supporting digital artifacts that document a) and b), or something like that. 

I like "designator hierarchy" -- the /_knops/ IRI/folder is part of the designator hierarchy. The mesh is a designator hierarchy. 

I like "mesh materialization" too, although "filesystem materialization" is clearer.
Thought for 32s

Updated the canvas to:

Define mesh namespace = designator hierarchy (tree of IRIs).

Make the namespace explicitly distinct from semantic description and filesystem materialization.

Treat /_knops/ as part of the designator hierarchy.

Use filesystem materialization as the clear default term, with mesh materialization as a short synonym.

One lingering ambiguity you should resolve (even if only in your own head): whether the designator hierarchy is only the “public-facing” tree, or whether it also includes every operational subtree by definition. The current draft commits to “includes reserved subtrees,” which is consistent with your / _knops / example, but it means “public-looking” has to be read as “path-like and familiar,” not “user-facing.”

Mesh Namespaces
Should we use "identifier" instead of "designator"? That aligns with the IRI abbreviation. Generally speaking, what are the semantic differences between identifier and designator, and then how do these apply to what we're doing?

I didn't understand your distinction between only the "public-facing" tree and every operational subtree. I don't love the use of public-facing, meshes might be private. But the mesh includes all the identifiers, all the knops (including operational/supporting ones) and all the nomen. 

Speaking of Nomen, they're now not as appropriatedly named because they are the things attached to/associated with identifiers. :/
Thought for 1m 15s
Thought for 1m 43s
Identifier vs designator

They overlap, but they carry different promises.

Identifier

Implies “this token is meant to pick out a thing” within some system.

Carries an expectation of uniqueness (at least within a scope) and often stability.

Reads as engineering-native: “IRI = identifier.”

Designator

Emphasizes “this token is a sign used to refer to something,” without implying uniqueness or permanence.

Plays nicer with your core stance that reference can be refined over time (even if you encourage single-referentness).

It also aligns with RDF’s own semantic framing: IRIs are said to denote things, and those denoted things are called the IRI’s referent.

How this applies to Semantic Flow

If you call the tree an identifier hierarchy, many readers will assume a stronger “fixed identity” contract than you actually want (especially given your “identity not fixed at minting” position).

If you call it a designator hierarchy, you’re communicating the “symbol → referent” relationship more honestly, but it’s slightly more academic/jargon-y.

A compromise that tends to work well:

Use identifier for the token (the IRI-as-string / IRI-as-node in RDF graphs).

Use designator when you’re talking about the act/role of designation (the IRI → referent relation and its evolution).

That lets you keep “IRI/identifier” alignment without accidentally promising rigid identity.

“Public-facing tree” vs “operational subtrees” (reframed)

You’re right to reject “public-facing” here. The useful distinction is not public/private; it’s stability and authoring intent:

Primary identifiers: the ones you expect humans to author directly and treat as durable.

Operational identifiers: reserved/generated subtrees for mechanics (inventories, metadata flows, _knops, etc.). They are still in the namespace, but you may explicitly give them a weaker stability promise.

So: the mesh namespace includes all of them; you’re just describing tiers of expectation.

Nomen feels misnamed now

This is the real tension: your current Nomen is closer to an authority-like record about an identifier, not the identifier itself.

But “Nomen” is defensible if you explicitly define it that way. In IFLA LRM, a nomen is an appellation/sign-combination associated with a thing, and systems often manage clusters of nomens via authority records—explicitly noting that this practice blurs “thing vs nomen” in real systems.

So your options are:

Keep Nomen, but make the definition unmistakable: “a naming record for an identifier (and optionally its referent),” i.e., a managed nomen/authority entry.

Rename class to match what it is in your framework: IdentifierRecord, NameRecord, NomenRecord, or IdentifierEntry. (This reduces cognitive dissonance, but it’s churn.)

If you keep Nomen, I’d strongly recommend adjusting labels/comments to prevent readers from assuming it is the identifier.

If you want, I can also tighten the article language further so it explicitly uses “identifier” for tokens and “designator” only for the referential role—without reintroducing ambiguity.

Mesh Namespaces
according to iflastandards, "Nomen is an entity that is defined in the IFLA Library Reference Model as "An association between an entity and a designation that refers to it"." That's actually pretty close to what we want. Our Nomen establish an association between an identifier and an entity, either directly with "designates" or indirectly via ReferenceLinks. 

We could use "designatorPath" to connote a difference between an identifier (which has a base + designatorPath) and the token which is locally-but-not-globally unique.
Thought for 1m 26s

The IFLA/LRM-style definition you quoted is very close to what you’re doing: a Nomen is a reified association between a designation and an entity.

How that maps cleanly onto Semantic Flow:

Designation (LRM) → your designator IRI (the thing in the mesh’s path-shaped hierarchy).

Entity (LRM) → your designatum (the thing the designator is about).

Nomen (LRM) → your Nomen record (the association object that can assert designates or point via ReferenceLinks).

This also resolves the “Nomen are now not appropriately named” worry: they are not the identifiers; they are the association objects about identifiers, which is exactly what “Nomen” is in that model.

Identifier vs designator

They’re often used interchangeably, but they carry different implications:

Identifier tends to connote “used to identify (uniquely)”. In practice, IRIs frequently behave this way, so the term reads as pragmatic/engineering-default.

Designator foregrounds “a sign that designates”, which tolerates your core stance better: reference is asserted/accumulated and can be refined (the sign’s role is primary; uniqueness is an aspiration, not a guarantee).

For docs, I’d use identifier as the default term (mainstream, aligns with “IRI”), and reserve designator when you’re explicitly talking about reference/naming semantics.

Why designatorPath is a good compromise

Your instinct is right: it cleanly separates:

Token (local): designatorPath — mesh-relative, locally unique, filesystem-friendly.

Identifier (global): meshBase + designatorPath → the absolute designator IRI.

That lets you say “identifier” for the full IRI while still having a crisp term for the locally-authored string. It also prevents the common mistake of treating a token/slug as “the identifier”.

If you ever find “designatorPath” too philosophy-flavored, the only serious alternative is identifierPath—but it’s easier to misread as “path of the identifier” rather than “path token used to form it”. “designatorPath” is more semantically honest.

I updated the “Mesh Namespaces” canvas to reflect this mapping (designator hierarchy + Nomen as association + designatorPath token).

Mesh Namespaces
So should we rename Nomen to NomenRecord? It's more accurate but more cumbersome, and I feel like it implies that the designator is the Nomen.
Thought for 52s

Do not rename the class to NomenRecord in the ontology.

Reasons:

Nomen already has the right semantics (reified “designator ↔ designatum” association), and it aligns with the established library/IFLA usage you cited.

NomenRecord is clearer to first-time readers, but it’s also heavier, and you’ll pay the “rename tax” everywhere (IRIs, docs, examples, SHACL, code).

The worry that it “implies the designator is the Nomen” is backwards: “record” would imply the Nomen is about the designator, which is what you want. The real risk is different: people will still casually say “the Nomen is the identifier” unless the docs hammer the distinction.

Compromise that gets the clarity without the taxonomy churn:

Keep the class IRI as sflo:Nomen.

In docs/UI, consistently render it as “Nomen (nomen record)” or “Nomen record”.

In the ontology, make that explicit via labels/notes:

rdfs:label "Nomen"

skos:altLabel "Nomen record"

tighten the comment to “reified association between a designator IRI and a designatum; not the designator itself.”

I already tweaked the canvas article to say “Nomen (nomen record)” in the first sentence of that section.

Mesh Namespaces
I don't think we should say "the mesh is this hierarchy" because the mesh is at once the designatorPath hierarchy, the corresponding identifier hierarchy, the filesystem folders that correspond to those hierarchies, the digital artifacts represented by parts of the hierarchy, the files that materialize those artifacts, and the Resource Pages that make the hierarchy dereferenceable. 

The "reified association" is actually the NomenMetadataFlow. 
Thought for 31s

You’re right: “the mesh is this hierarchy” is false (or at best a sloppy metonym). What you actually have is a bundle that includes a designator hierarchy, but also includes multiple other “planes” and their linkages. Treating the hierarchy as the mesh collapses important distinctions and will confuse readers the moment you mention artifacts, bytes, and pages.

A cleaner split that stays crisp:

Mesh namespace: the designator hierarchy (tree of designator IRIs).

Mesh: the whole: hierarchy + semantic description + filesystem materialization + artifacts + files + resource pages + operational subtrees.

That lets you keep the nice, illuminating “namespace is a tree” line without overclaiming what “mesh” is.

On “the reified association is actually the NomenMetadataFlow”

This is a big conceptual pivot, and it has consequences you should face explicitly:

If sflo:designates (and reference-link assertions) live on the Nomen node, then the Nomen is the reified association (because it’s literally the node you hang the association facts on).

If you want the Flow to be the reified association, then the Nomen becomes more like an anchor/handle, and the association facts should be asserted in the NomenMetadataFlow’s slices (or via a relator node that the flow carries).

Both are workable; they just aren’t the same model.

If your intent is “stable anchor + changeable assertions,” then your claim is coherent: Nomen = anchor; NomenMetadataFlow = canonical assertion surface. But then you should treat direct designates on Nomen (if you keep it) as a convenience shortcut, not the canonical representation—otherwise you’ll have two competing “sources of truth.”

I updated the canvas article accordingly:

removed “mesh is the hierarchy”

reframed as “naming surface”

made the Nomen section explicitly “anchor, with canonical assertions in NomenMetadataFlow.”

If you want to fully commit to “flow is the reification,” the next ontology step is to decide whether designates belongs on:

Nomen (shortcut),

NomenMetadataFlow’s payload graph (canonical), or

a dedicated relator class (cleanest, but heavier).

Mesh Namespaces
We're pushing up on making the designators the base-less path strings, and the identifiers are the IRIs. Which might not be a good idea. But if that's what we do, I don't think we want to say "The hierarchy is the set of designator IRIs " - more like "The hierarchy is defined by the set of designatorPaths (strings) "

Regarding "live on the Nomen node"... it depends what you mean by "live". The Nomen contains the NomenMetadataFlow, which is where the sflo:designates triples live/exist. So the Flow contains the associations, and the Nomen is the ArtifactContainer for the Flow. You could call the Nomen the anchor, but you can't say the Flow is the reification: the Flow (FlowArtifact) is the thing that contains the reification.
Thought for 1m 3s

Your latest framing is directionally right, but there’s one precision trap: “reified association.”

In RDF terms, a reified association is a resource that is the association (a relator node, or RDF reification, or an n-ary pattern). A FlowArtifact is not that. It’s a versioned carrier for graphs that assert the association. So:

If you mean “where the association is recorded” → yes: the slice datasets inside NomenMetadataFlow are where the sflo:designates (etc.) triples exist.

If you mean “the association as an entity” → that’s still the Nomen (or an explicit relator inside the dataset), not the Flow.

That distinction matters because it changes what you can point at and reason about: you can talk about a Nomen as “the naming situation,” but a Flow is “the evolving record of claims about that naming situation.”

I updated the canvas article to reflect your clarified position:

The hierarchy is defined by designatorPaths (strings), and only induces an IRI hierarchy under meshBase.

The “assertions live” language now points at slice graphs, with Nomen as container/anchor, Flow as versioned carrier.

Remaining sharp edge to watch (not solved by wording): if you keep pushing toward “designators = baseless strings,” you’re intentionally giving up “designator as an RDF node” and replacing it with a literal token + derivation rule. That’s fine, but it needs to stay explicit everywhere (otherwise you’ll drift into treating strings and IRIs interchangeably and create subtle bugs in shapes, queries, and docs).

Mesh Namespaces
Maybe the "Mesh Namespace" is the namespace carved out by prepending the base to all the designatorPaths: It feels like without a base, it's not really a namespace. What do you think?
Thought for 51s

Yes: if you use “namespace” in the RDF/Web sense, it needs a base. Without meshBase, you don’t have a namespace of IRIs—you have a mesh-local pathspace (your designatorPath set) that can induce a namespace once a base is supplied.

Two important implications:

If meshBase changes, the namespace (IRI set) changes. That’s fine if meshBase is genuinely canonical and stable; it’s a problem if you expect meshes to be routinely transposed across bases (dev/staging/prod, forks, mirrors). Keeping designatorPaths explicit is exactly what preserves transposability.

The base-less hierarchy is still “real” and useful: it defines containment, lets you validate structure, and is what humans author. It’s just not a global namespace until you bind it to a base.

So the clean split is:

Naming surface / designator hierarchy = the set/tree of designatorPath strings.

Mesh namespace = the induced set of identifier IRIs under meshBase.

I updated the canvas article to reflect this distinction.

Mesh Namespaces
Yes, it's possible we'd have multiple meshBases over time. Maybe we have subproperties meshCanonicalBase (because ideally and usually the base is fixed), meshPreviousBase, and meshLocalBase (operational only, for when served at file:/// or http://localhost, i.e., local work). 

But it's possible to serve the mesh locally or at an alternate location but still use the canonical-base identifiers. You lose derferenceability, which kinda feels unavoidable. I do want to support the situation where the canonicalBase changes, so we need to keep track of former meshBases for sameAs-like processing, and local work has always been a big motivation. But literally RDF files can only have one base at a time, and I really don't want to have to build meshes for different targets. We could go back to the inferred base but that messes up consistent identifier use and requires "../../../" hell. 

Don't change anything in the canvas. Just looking for your thoughts, or maybe an out-of-the-blue hail mary. How do other people deal with changing bases?
Thought for 3m 22s

If meshBase is “the base that forms identifier IRIs,” then changing it is not a minor operational detail: you have changed the namespace (the identifier set). That’s the core tension you’re feeling, and most people avoid it rather than “model it well.”

How others usually handle it

They don’t let the identifier base change (or treat it as a breaking event)

“Cool URIs don’t change” is effectively the standard social contract: pick a stable base, then keep it stable, even if hosting moves.

Practically, they pick a base that is not coupled to the hosting provider (e.g., not *.github.io), then route/redirect from there.

They use indirection services so hosting can change without IRIs changing

w3id.org exists specifically to provide permanent identifiers via HTTP redirects, while allowing the actual hosting to move.

Same pattern with PURLs, custom domains + reverse proxy, etc.

When a base must change, they do HTTP redirects + explicit mapping links

Keep old IRIs dereferenceable and redirect to the new IRIs (this is the “Cool URIs” playbook).

Additionally, publish a small mapping graph expressing replacement relationships (less committal than owl:sameAs):

dcterms:replaces / dcterms:isReplacedBy

prov:alternateOf when you want “same underlying thing, different carrier/aspect” semantics.

A “hail mary” that solves your local-work + canonical-IRI problem cleanly

Keep exactly one canonical identifier base forever, and make local dev serve that base.
This is normal web practice, just applied to RDF/meshes:

Choose meshCanonicalBase as a domain you control (or w3id.org/...) and treat it as the identifier base.

For local work: map that host to 127.0.0.1 (hosts file) and run a local server that serves your mesh under the canonical host.

Result: your data always uses canonical IRIs, dereferencing still works locally, and you never need a “meshLocalBase” that changes identifiers.

For staging/alternate hosting: you can still serve bytes elsewhere, but the “real” dereference story remains the canonical host.

This avoids the worst outcome: having multiple equally-valid bases that silently fracture your graph identity.

If you still want to model base movement explicitly

Your instinct to split properties is good, but I’d tighten the semantics:

meshCanonicalBase (exactly 1, strong expectation of long-term stability)

meshPreviousBase (0..n, historical record; should normally redirect)

Avoid meshLocalBase as an identifier base. If you keep it, make it explicitly “access base,” not “identifier base.”

Then, when canonical base changes (rare, but supported):

publish redirects from old → new

publish mapping assertions using dcterms:isReplacedBy / replaces

optionally add prov:alternateOf for consumers who want to stitch identity across the move without equating everything via owl:sameAs.

The critical decision to make (so you don’t keep re-litigating it)

Decide whether meshBase is:

Identifier base (defines the namespace; changing it is a namespace migration), or

Access base (where you can fetch today’s bytes; can vary freely)

Right now you’re using it like the first, but wanting the freedoms of the second. The “serve canonical base locally” approach is the cleanest way to get both without splitting your identity surface.

Mesh Namespaces
Agreed. We've been here before. mesh base is Identifier base. Maybe we can support local work with tooling that privileges designatorPaths over identifier IRIs?

So let's return to the canvas.

In RDF-world, namespace has the existing connotation of "the thing that qualifies all the terms" in an RDF dataset. So we shouldn't overload that. Especially since they didn't really consider a hierarchical/nested namespace, ie., that the namespace can include the terms if the terms have children.

Maybe instead of "Mesh Namespaces" it's "Designator space". i.e. we continue down this road of "designators are mesh paths that become identifiers when paired with the meshBase". 


Thought for 32s

Two key fixes you just made to the framing:

Stop calling it a “namespace.” In RDF practice, “namespace” overwhelmingly means “prefix/base for terms,” not “a hierarchical identifier surface.” Using designator space + identifier space keeps you out of that semantic tarpit.

Treat the filesystem as a primary specification mechanism, not a mere mirror. If the expected way to mint/evolve the designator hierarchy is “make folders/files,” then the filesystem isn’t just a representation—it’s a materialization/specification of the designator space.

I updated the canvas to switch the article from “Mesh Namespaces” to Designator Space, and rewrote it to center:

designatorPath strings as the authored hierarchy,

meshBase as the identifier base,

induced identifier IRIs as meshBase + designatorPath,

and a tooling note that explicitly supports your “privilege designatorPaths over IRIs” strategy for local work and base churn.

Mesh Namespaces
Can a set of paths really be tree-shaped?
Thought for 14s
Mesh Namespaces
Designator Space

A mesh’s designator space is the set of mesh-relative path tokens (designatorPath strings) that people author, review, and reason about. Interpreted as segmented paths, these tokens can be organized into a rooted prefix tree (a trie): each path has a unique parent given by removing its last segment. (If intermediate prefixes aren’t explicitly present, treat them as implicit nodes; otherwise the explicit set is a forest rather than a single tree.)

A mesh also declares an identifier base (meshBase). Pairing the base with each designatorPath induces a corresponding identifier IRI:

identifier IRI = meshBase + designatorPath

The induced set of identifier IRIs is the mesh’s identifier space.

This avoids overloading “namespace,” which in RDF often means “the prefix/base that qualifies terms in a dataset,” rather than a hierarchical identifier surface.

Designator hierarchy

The designator space is inherently hierarchical: containment is prefix-based.

A parent “contains” a child when the parent’s designatorPath is a prefix of the child’s designatorPath.

Reserved operational subtrees (e.g., /_knops/) are simply subtrees in the same designator hierarchy.

A useful distinction is primary vs operational:

Primary designators are the ones you treat as durable and author directly.

Operational designators are reserved/generated for mechanics (inventories, metadata flows, etc.). They are still part of the hierarchy, but you may not promise the same stability.

Semantic description and filesystem materialization

The designator space is distinct from both:

its semantic description (what the identifier IRIs are about; plus the mesh’s internal artifacts), and

its filesystem materialization (the folders/files used to define, store, and serve it).

In the common case, the filesystem is not merely a convenience: creating and editing the conforming folder/file structure is the primary way the designator space (and thus the identifier space) is specified and evolved.

Meshes may also include non-semantic folders alongside semantic ones; semantic membership is asserted and validated rather than assumed from “everything on disk.”

Nomen

A Nomen is the mesh resource associated with a particular designatorPath.

The Nomen is the stable anchor for “the naming situation” at that path.

The naming assertions (e.g., sflo:designates, reference links) live in the NomenMetadataFlow’s slice datasets.

The FlowArtifact provides identity/versioning for those datasets.

The Nomen is the ArtifactContainer that hosts that flow.

So: the Nomen is not the association itself; it hosts the versioned carrier of the association.

Mirroring

Mirroring is the intended alignment between the designator hierarchy and the filesystem materialization: the path you see is the path you edit.

This is what makes meshes legible, editable, and reviewable with ordinary tooling while keeping naming intent explicit.

Tooling note: privileging designatorPaths

Even when meshBase is the authoritative identifier base, tooling can still treat designatorPath as the primary handle:

Authoring, diffs, inventorying, and most validation operate on designatorPath and the induced tree.

IRIs are derived late (meshBase + designatorPath) for publication and interoperability.

Local work can remain productive even when canonical dereferenceability is unavailable, because the designator space remains stable and fully navigable.

Terminology

designatorPath: mesh-relative path token (string).

designator space / designator hierarchy: the rooted prefix tree (trie) induced by segment-prefixing on designatorPath.

meshBase: identifier base.

identifier IRI: meshBase + designatorPath.

identifier space: the induced set of identifier IRIs.

Not literally, in the strict graph-theory sense—at least not without an interpretation layer.

A set of path strings becomes “tree-shaped” only after you choose:

A segmentation rule (e.g., split on /), and

A parent relation (usually “remove the last segment”).

Then you can represent the set as a prefix tree (trie):

Each path token has a unique parent path token conceptually (drop the last segment).

If every parent path is also explicitly included (and you include the root, e.g. ""), the explicit set forms a rooted tree.

If some parents aren’t explicitly present, the explicit set is a forest, but it still embeds in a tree once you treat missing intermediate prefixes as implicit nodes.

Example:

Explicit tokens: a/b/c, a/d

Parents: a/b and a (missing)

As an explicit set: not a tree (missing nodes)

As a trie with implicit nodes: a tree rooted at ""

I updated the canvas wording to be precise: “set of tokens” that organize into a rooted prefix tree (trie), with the forest/implicit-parent caveat.

Mesh Namespaces
Please provide (not in the canvas), a concise rdfs:comment from <Nomen/>

rdfs:comment for <Nomen/> (concise):

A mesh-governed anchor resource for a single designatorPath, hosting the metadata flow that records what the corresponding identifier is about (via sflo:designates and/or sflo:ReferenceLinks).

Mesh Namespaces
Pretty good. What does "mesh-governed" mean?

“Mesh-governed” means: the mesh’s conventions are authoritative for that resource’s identifier, structure, and expected validation/behavior. Concretely, a mesh-governed resource is one that:

Lives in the mesh’s identifier space (its IRI is intended to be within the meshBase + designatorPath surface, or otherwise treated as part of that surface).

Is subject to Semantic Flow invariants (inventorying, containment expectations, SHACL constraints, reserved subtrees like /_knops/, etc.).

Is expected to be materializable and dereferenceable under the mesh’s rules (even if not always currently published).

If that still feels hand-wavy, it’s because “governed” smuggles in a vague authority metaphor. A clearer alternative in comments is usually “mesh-scoped” (about identifier scope) or “mesh-conformant” (about rules/validation).

Mesh Namespaces
The comment for <MeshResource/> is "A resource governed by Semantic Flow mesh conventions and addressable within a mesh's namespace. Includes meshes themselves." - Right now, a LocatedFile is not a MeshResource (and shouldn't be). Help me decide whether to create a new MeshLocatedFile class that subclasse MeshResource, or not.
Thought for 43s

You currently have a definition problem, not a “do we need another class” problem.

Your <MeshResource/> comment says (paraphrasing): “governed by mesh conventions and addressable within a mesh’s namespace.” If “addressable within a mesh’s namespace” means “IRI is under meshBase,” then any in-mesh LocatedFile is, by definition, a MeshResource—including _assets/*, distributions, etc. But you explicitly do not want LocatedFile ⊑ MeshResource, because LocatedFile also covers external endpoints.

So you have two coherent options:

Option A: Do NOT add MeshLocatedFile (recommended for now)

Keep LocatedFile purely general. Keep only specific mesh-governed located files as MeshResource via existing classes (ResourcePage, LocatedArtifactFile, etc.).

To make this consistent, tighten the <MeshResource/> comment so it does not imply “everything under base is a MeshResource.” Something like:

“A resource that participates in Semantic Flow mesh conventions and is treated as part of the mesh’s managed resource surface (e.g., Mesh, Knop, Nomen, mesh-governed artifacts and pages).”

This keeps the ontology slimmer and avoids class creep.

When this breaks down: the moment you want to talk about mesh-governed non-artifact files (assets, derived docs, templates, distributions) as first-class members of containment/inventory, or you want reference links “about” those files in a principled way.

Option B: Add MeshLocatedFile

Add:

MeshLocatedFile ⊑ LocatedFile, MeshResource

This gives you a clean way to say: “this is a bytes endpoint and it is mesh-governed.”

When it’s worth it: when you need a uniform target for:

“files in the mesh that aren’t artifacts” (assets, derived pages beyond ResourcePage, exported distributions)

containment/inventorying via containsMeshResource

SHACL constraints over in-mesh byte endpoints that should hold regardless of which kind they are

What you must then decide (and this is the trap):

Are _assets/* and other purely-supporting files “mesh-governed resources” or merely “mesh-adjacent bytes”?

If yes: they likely become MeshLocatedFile (and you’ll want subclasses like AssetFile later).

If no: you will still have lots of in-mesh located files that you don’t type as MeshResource, meaning “under base” ≠ “mesh-governed,” and your comment must still be careful.

My recommendation

Option A now, with a comment fix. You already have mesh-governed located-file concepts where they matter (ResourcePage, LocatedArtifactFile). Add MeshLocatedFile only when you introduce a real need to model other mesh files (assets/distributions/etc.) as first-class MeshResources.

If you want a crisp decision rule:

If you expect to ever write “for every file under the mesh materialization, we can inventory/validate it as part of SF,” then you need MeshLocatedFile (or something equivalent).

If SF only cares about some files (artifacts + resource pages) and treats the rest as incidental build output, don’t add it and keep <MeshResource/> narrowly defined.

Mesh Namespaces
We got rid of LocatedArtifactFile.
Thought for 1m 0s

If LocatedArtifactFile is gone, you’ve removed your only typed hook for “this bytes endpoint is mesh-internal / mesh-governed” (other than ResourcePage, which is an exception you already allow).

That exposes an inconsistency you should resolve explicitly:

You say “LocatedFile is not a MeshResource (and shouldn’t be)”.

But you also have ResourcePage ⊑ LocatedFile ⊓ MeshResource.

So the real rule is not “no LocatedFiles are MeshResources,” it’s “only some LocatedFiles are MeshResources.”

Given that, the choice is:

Create MeshLocatedFile if you want typed, closed-form validation and modeling

Add a class:

MeshLocatedFile ⊑ LocatedFile ⊓ MeshResource

Then:

ResourcePage ⊑ MeshLocatedFile

Any “in-mesh distribution file,” “in-mesh artifact bytes file,” etc. can be optionally typed MeshLocatedFile.

SHACL can say “AbstractArtifactFile SHOULD have at least one hasLocatedFile whose value is a MeshLocatedFile” without SPARQL string hacks.

This is the cleanest replacement for what LocatedArtifactFile used to do.

Don’t create it if you want to treat bytes endpoints as implementation detail

Keep only ResourcePage as the “file that matters semantically,” and treat all other located files as:

derivable,

optionally external,

not worth first-class typing.

Then validation that “a located file is in-mesh” must be done via SPARQL constraints (e.g., IRI prefix/pattern checks) or not at all.

My recommendation (given your constraints)

Create MeshLocatedFile.

Without it, you’ll keep needing SPARQL constraints to distinguish internal vs external located files, and you’ll end up re-inventing the lost class boundary in shapes anyway. MeshLocatedFile is a single, high-leverage class that restores that boundary without reintroducing the old “artifact file” naming you wanted to avoid.

Mesh Namespaces
I'm not sure we need MeshLocatedFile. I will leave it out for now. 

As for ResourcePage, right now it's a <MeshResource/> but I'm thinking to remove that. Slices are clearly MeshResources but they can have AbstractFiles and those have LocatedFiles, they don't need MeshAbstractFiles or MeshLocatedFiles.

So MeshResources isn't about being "governed by the mesh". Its the semantic flow platform that governs Resources in a Mesh, whether or not they are MeshResources. MeshResources are things that are specific to Meshes. Does that make sense?
Thought for 51s

Yes, that makes sense if you commit to a sharp distinction:

MeshResource = a node in the Semantic Flow mesh model (things the framework names/structures/validates as part of “what a mesh is”).

LocatedFile / AbstractFile = realization machinery (how bytes are retrieved / encoded), not part of the mesh model itself.

That distinction is clean, and it lines up with your “Slices are MeshResources but they can have AbstractFiles/LocatedFiles” point.

What you must fix (otherwise it’s inconsistent)

Your current <MeshResource/> comment is now wrong for this meaning:

“governed by mesh conventions and addressable within a mesh’s namespace…”

It conflates identifier-scope (“addressable under base”) with model-membership (“is a SF mesh concept node”). Those are different.

If you adopt your new meaning, <MeshResource/> should instead read like:

“A Semantic Flow structural resource: a node in the mesh model (e.g., Mesh, Knop, Nomen, Slice, flows, inventories) that participates in Semantic Flow constraints and relationships.”

No “addressable within namespace” language.

ResourcePage: removing MeshResource is coherent

If ResourcePage is “derived UI/navigation file,” then it’s better treated as a LocatedFile that represents a MeshResource, not as a MeshResource itself.

So:

hasResourcePage : rdfs:Resource → ResourcePage stays fine.

ResourcePage ⊑ LocatedFile ⊓ schema:WebPage (drop MeshResource) matches your intent: pages are outputs.

The one place you might get bitten

If you ever want to attach ReferenceLinks or containment/inventory assertions about pages, you currently can’t if:

referenceLinkFor ranges over MeshResource, and/or

containment predicates range over MeshResource.

But that may be exactly what you want: “pages are outputs, not mesh nodes.”

One wording issue in your rationale

“the platform governs resources in a Mesh, whether or not they are MeshResources”

This is too broad. The mesh/platform doesn’t “govern” external referents; it governs its own asserted descriptions and artifacts about them.

A more accurate framing:

The mesh curates assertions about many resources (including external referents).

MeshResources are the resources that belong to the mesh’s own structural/model layer.

If you accept that, then leaving out MeshLocatedFile and dropping MeshResource from ResourcePage is consistent and keeps the ontology slim.

Mesh Namespaces
Ultimately MeshResource becomes misleading, because it sounds like anything in a mesh. I hate SfloResource, but that is more accurate. 
Thought for 1m 21s

Yes—if the class is meant to mean “a first-class node in the Semantic Flow mesh model,” then MeshResource is misleading because the default reader interpretation is “anything in the mesh (esp. anything under meshBase).” That interpretation will always fight you (and you’ll keep re-explaining why LocatedFile isn’t one, why ResourcePage is/ isn’t one, etc.).

If the intent is:

LocatedFile / AbstractFile = realization machinery (may be in-mesh or external; not inherently “mesh model”)

MeshResource = SF structural/model nodes (Mesh/Knop/Nomen/Slice/Flows/Inventories/etc.)

…then the name should encode SF model membership, not “meshness.”

Better replacements than SfloResource

Pick one name and stop apologizing for it:

SemanticFlowResource (most semantically obvious; avoids the ugly sflo:sflo… feel)

SflowResource (short, pronounceable, matches your existing “sflow” branding)

SfResource (shortest, but reads a bit like an abbreviation leak)

I’d choose SflowResource if you want concise + branded, or SemanticFlowResource if you want zero ambiguity.

What to do with MeshResource

Don’t keep MeshResource as a synonym; that’s how you end up with a flabby ontology again.

Either:

Rename (new class IRI) and deprecate the old one, or

Keep the IRI <MeshResource/> for continuity but change the label/comment to “Semantic Flow resource” (I don’t recommend this—IRI says Mesh, label says SF, you’ll confuse future-you).

Minimal deprecation pattern (if you rename)
sflo:MeshResource a rdfs:Class ;
  owl:deprecated true ;
  rdfs:comment "Deprecated. Use sflo:SflowResource." .

sflo:SflowResource a rdfs:Class ;
  rdfs:label "Semantic Flow resource" ;
  owl:equivalentClass sflo:MeshResource .

Sanity check on your new meaning

If “MeshResources are things specific to meshes” means “things specific to the mesh model,” then yes, that’s coherent. But then your comment must not say “addressable within a mesh’s namespace,” because that pulls you back into the IRI-scope interpretation.

If you want, I can give you a one-sentence replacement comment for the renamed class that doesn’t backslide into “everything under base.”

Mesh Namespaces
No deprecation pattern needed, we're still in pre-implementation phase.

Here's my revised and re-organizaed ontology. Feedback please:

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
  rdfs:subClassOf <SemanticFlowResource/>, <Realizable/> ;
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
# Resource pages
#################################################################

<ResourcePage/> a rdfs:Class ;
  rdfs:subClassOf <LocatedFile/>, schema:WebPage ;
  rdfs:label "Resource page file" ;
  rdfs:comment "A derived UI/navigation file (e.g., index.html) that presents a mesh-governed resource (or other referent) to humans." .

<hasResourcePage/> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <ResourcePage/> ;
  rdfs:label "has resource page file" ;
  rdfs:comment "Associates any resource (including referents) with a human-readable presentation page." .

#################################################################
# Reference data
#################################################################

<ReferenceLink/> a rdfs:Class ;
  rdfs:label "Reference link" ;
  rdfs:comment "A relator describing a reference-data link from a SemanticFlowResource to a specific reference data source, including role metadata." .

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
  rdfs:comment "Associates a mesh-governed resource with a ReferenceLink that provides external or supporting reference information about it." .

<referenceLinkFor/> a owl:ObjectProperty ;
  rdfs:domain <ReferenceLink/> ;
  rdfs:range rdfs:Resource ;
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
  rdfs:comment "If present, the reference target is a resource used as a reference-data identifier (often a mesh-governed artifact/file identifier, but it may also be an external identifier)." .

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
  rdfs:comment "Structural containment relation for mesh-governed resources, asserted from an artifact-hosting resource (Mesh, Knop, or Nomen)." .

#################################################################
# Semantic Meshes
#################################################################

<Mesh/> a rdfs:Class ;
  rdfs:subClassOf <SemanticFlowResource/>, <ArtifactContainer/> ;
  rdfs:label "Mesh" ;
  rdfs:comment "A servable filesystem region (conventionally surfaced via a reserved _mesh/ handle) that contains mesh-governed resources and, optionally, other supporting resources." .

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
  rdfs:comment "A container for mesh-governed artifact identities anchored at a stable in-mesh location; hosts a primary payload and supporting artifacts." .

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
  rdfs:label "preferred slug" ;
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
Thought for 2m 39s

High-level: this is a meaningful improvement. You’ve (a) separated the “general spine” from the “Semantic Flow spine” cleanly, (b) removed the LocatedArtifactFile / MeshLocatedFile proliferation without losing the realization model, and (c) made the “SF-ness marker class” explicit instead of overloading “Mesh.”

That said, a few inconsistencies remain that will bite you later unless you tighten them now.

1) You still have “mesh-governed” language in places where you’ve explicitly moved away from MeshResource

Examples:

ResourcePage comment: “presents a mesh-governed resource…”

hasReferenceLink / referenceLinkFor comments: “Associates a mesh-governed resource…”

But you’ve now intentionally made:

ResourcePage not a SemanticFlowResource

hasReferenceLink domain = rdfs:Resource (i.e., literally anything)

Recommendation: purge “mesh-governed” from comments and replace with something accurate like:

“Semantic Flow–managed” (if you mean the tooling constrains it), or

“Semantic Flow resource” (if you mean instances of SemanticFlowResource), or

“any resource” (if you intentionally widened the domain).

Right now the words contradict the axioms.

2) ReferenceLink’s scope is internally inconsistent (class comment vs property domains)

ReferenceLink comment says: “link from a SemanticFlowResource…”

hasReferenceLink domain is rdfs:Resource, and referenceLinkFor range is rdfs:Resource

Those two positions are different designs:

SF-only links: domain/range should be SemanticFlowResource (or at least the intended subject)

Universal links: then the class comment must not mention SemanticFlowResource at all

Given your earlier desire to attach reference links even to external referents, your current axioms say “universal.” Update the comments accordingly, and keep the SHACL as your “SF-specific expectations” layer.

3) RdfDataset semantics: your comment says “content is an RDF dataset” but the class is not Realizable

You made RdfDataset ⊑ DigitalArtifact, dcat:Dataset (not Realizable), but the comment says it has “content” that is an RDF dataset. That’s ambiguous because:

for SimpleArtifact, “content” lives on the artifact (it’s realizable)

for FlowArtifact, “content” lives on slices, not the flow identity

Yet you also do KnopMetadataFlow ⊑ FlowArtifact, RdfDataset, which implies the flow identity is an RdfDataset.

This can still work, but you must phrase it precisely. Recommendation: change the comment to something like:

“A DigitalArtifact whose realized representations (directly for realizable artifacts, or via slice datasets for flow artifacts) are RDF datasets (default graph + 0..n named graphs).”

Otherwise you’ll keep arguing with yourself about whether datasets are identities or realizations.

4) containsSemanticFlowResource is now the true root predicate, but its comment still names Mesh/Knop/Nomen

Its domain is ArtifactContainer, not “Mesh/Knop/Nomen.” The comment should match the axiom:

either keep it general (“asserted from an ArtifactContainer”)

or restrict the domain to an SF container class (but you intentionally avoided that earlier)

Also: the label capitalization (“contains Semantic Flow resource”) is visually inconsistent with the rest.

5) Ordering/readability regression: Slice uses SemanticFlowResource before it’s introduced

Turtle permits forward references, but you said “revised and reorganized.” For human readers, defining the SF marker class before it’s used is the difference between “crisp” and “I have to scroll to discover what this means.”

6) ArtifactContainer vs DigitalArtifact: you rely on prose to say they’re distinct

You say “Not itself an artifact identity,” but there’s no OWL disjointness. That’s not necessarily wrong (open world, extensibility), but it means your SHACL must carry the burden of preventing accidental dual-typing.

If you do want the ontology itself to guard that boundary without full disjointness, at least consider a warning-level SHACL rule later (“should not be both”).

7) preferredPayloadSlug: good idea, but the property name/label is currently mismatched

property IRI: preferredPayloadSlug

label: “preferred slug”

Make the label explicit (“preferred payload slug”) to avoid future collision with a more general preferredSlug you may want for other things.

Also: domain = Knop is plausible, but note the conceptual oddity: the slug names the payload distribution file, not the container. You’ve documented this well (“Not an identifier; does not affect

Mesh Namespaces
For the ResourcePage comment, how about "An HTML page that presents a resource." It presents better than "describes"?

Regarding #3, I want RdfDataset to be applicable to a FlowArtifact as well as a SimpleArtifact, and Flows are not Realizable. i.e., RdfDatasets may be abstract and have realizations. Will that cause any issues?

Regarding #4, if we want to keep ArtifactContainer general, I guess we need a SemanticFlowArtifactContainer?

I removed <SemanticFlowResource/>, from ArtifactContainer


Thought for 52s
ResourcePage comment

“An HTML page that presents a resource.” is good and uncontroversial. I’d only add one qualifier if you still mean it:

“A (typically derived) HTML page that presents a resource.”

“Presents” is better than “describes” here, because the page can be navigational, composite, or editorial without claiming it’s the authoritative semantic description.

RdfDataset on FlowArtifact (Flows not Realizable)

This is workable, but only if you’re strict about what RdfDataset means.

Right now, you’re using RdfDataset as a content-kind marker (“when you realize this artifact’s content, you get RDF dataset graphs”), not as “this thing itself is the realizable carrier of bytes.”

That lets you say:

SimpleArtifact + RdfDataset: the artifact itself is realizable and its realizations are RDF datasets.

FlowArtifact + RdfDataset: the flow’s slices carry the realizations (datasets), not the flow identity.

No OWL “issue” arises from this, but there is a model clarity risk:

If you keep the comment “whose content is an RDF dataset,” people will infer that the FlowArtifact node is the dataset content-bearer. It isn’t.

So: keep RdfDataset not a subclass of Realizable, and tighten the wording to explicitly cover both cases (“directly realizable artifacts, or via slice datasets for flows”). Then enforce expectations in SHACL (e.g., slices of an RdfDataset flow should have RDF dataset realizations).

Domain of containsSemanticFlowResource + “SemanticFlowArtifactContainer”

If you keep ArtifactContainer general, you have three options:

Keep OWL domain = ArtifactContainer (what you have)

Downside: anyone using the predicate will be inferred an ArtifactContainer under reasoning.

If you truly don’t want inference assumptions, OWL domain is mostly documentation, but it still becomes a footgun as soon as anyone runs a reasoner.

Remove the OWL domain entirely; enforce with SHACL

Cleanest if you want “no inference assumptions.”

You can still have SHACL shapes with sh:targetSubjectsOf sflo:containsSemanticFlowResource and require rdf:type sflo:ArtifactContainer (or the SF containers specifically).

Introduce a convenience class for the domain (your suggestion)

e.g. SemanticFlowContainer ≡ ArtifactContainer ⊓ SemanticFlowResource (or just subclass both)

Set property domain to SemanticFlowContainer to prevent accidental typing of random resources as containers (under entailment, you’d type them as SF containers instead—which is still typing, just narrower/intentional).

If you want the ontology to remain slim: Option 2 is the most consistent with “don’t assume inference for SHACL.”

“I removed SemanticFlowResource from ArtifactContainer”

That aligns with your intent: “container-ness” is a general structural role; “SemanticFlowResource” is the SF membership marker. Good separation.

The only thing to watch: any property with domain ArtifactContainer now becomes your only “this is a container” typing mechanism if you lean on entailment. If you don’t, it’s fine—just make sure SHACL asserts container typing where you need it.

It might be worth considering putting the general spine in its own ontology. 

Yes, and the fact that you’re repeatedly fighting “what belongs to SF vs what’s generally true on the web” is the strongest signal that a split will reduce churn.

What you gain by splitting

Cleaner meanings: DigitalArtifact, Realizable, AbstractFile, LocatedFile, ResourcePage, etc. stop getting dragged into SF-specific debates.

Less renaming pain: you can rename SemanticFlowResource/MeshResource/container terms without destabilizing the “general spine.”

Easier reuse/credibility: you can point to a small, principled “artifact/realization” ontology that other projects could adopt without adopting SF.

What you risk / costs

Two imports in every mesh dataset (or one plus inference assumptions). If you don’t want inference assumptions, you’ll still be fine, but it’s an operational overhead.

Boundary discipline: you’ll need a hard rule for what stays general (e.g., “no mention of Mesh/Knop/Nomen, no SF workflow semantics, no ‘mesh-governed’ wording”).

Over-generalizing: temptation to keep adding “just one more” general class until it becomes its own framework.

A split that matches what you already have
Ontology A: sflo-core (general)

Keep it intentionally boring:

DigitalArtifact

Realizable

Realization

AbstractFile, AbstractStream

LocatedFile

hasAbstractFile, hasLocatedFile, locatesAbstractFile

ResourcePage, hasResourcePage

(optionally) ArtifactContainer + a generic containsArtifactOrResource if you truly need it cross-domain

Ontology B: sflo-mesh (Semantic Flow)

Everything that depends on SF conventions:

SemanticFlowResource

Mesh, Knop, Nomen

containsSemanticFlowResource

meshBase, designatorPath, designates

Payload/metadata/inventory classes and containment properties

ReferenceLink + roles (arguably could be in core, but you use it as an SF mechanism)

A hard boundary rule (recommend)

Put this at the top of both ontologies and enforce it mechanically:

Core: no SF terms, no “mesh”, no “designatorPath”, no “knop”, no “nomen”, no “inventory”, no “slice”.

Mesh: may reference core classes, but must not redefine them (only subclass / restrict / shape them).

One specific red flag your current structure reveals

You’re currently using RdfDataset as a “content-kind marker” that applies to both FlowArtifact and SimpleArtifact. That’s fine, but it’s exactly the kind of concept that benefits from the split:

In core, define RdfDataset as a DigitalArtifact + dcat:Dataset alignment (or maybe just a mixin class, depending on how strongly you mean it).

In mesh, state how SF realizes it (e.g., slice realization expectations) via SHACL, not OWL subclassing.

If you do the split, you’ll stop revisiting this every time you tweak SF container semantics.

Recommendation

Do the split now, while you’re still pre-implementation. The ontology is already telling you where the seam is. If you wait, you’ll accumulate data, docs, and shapes that make the migration annoying.

I think dcat is kinda fuzzy around versioning, and we're talking about artifact versions/revisions anyhow, so I think it's generally useful. Same thing for adsm: it's explicitly about semantic web stuff. 

Agreed on the underlying instinct: “dataset-ness” and “RDF-ness” are not SF-specific, and it’s useful to name them once, cleanly, in your general spine.

The caution is that DCAT bakes in catalog/distribution discovery assumptions that you are not always making when you say “this artifact’s content is an RDF dataset.”

DCAT: where it’s fuzzy for your use

Versioning semantics are underspecified for your needs: DCAT’s versioning hooks (e.g., version identifiers, relations between versions) are oriented toward dataset publishing/discovery, not revision-chain identity the way you model Flow/Slice.

Dataset identity vs artifact identity: In your system, a FlowArtifact is an identity anchor, and “RDF dataset” is a content-kind that can apply across realizations and across slices. DCAT doesn’t give you that distinction; it assumes the Dataset is the described thing being published.

So you can align to DCAT without letting DCAT define your model.

Recommendation: keep RdfDataset in the general spine, but treat DCAT as an alignment, not a foundation

Concretely:

Keep sflo:RdfDataset rdfs:subClassOf sflo:DigitalArtifact as the primary meaning.

Add rdfs:subClassOf dcat:Dataset only if you are comfortable with the inference: “anything you call an sflo:RdfDataset is fair game to treat as a DCAT Dataset.”

Otherwise, make it a non-subclass alignment (annotation-level mapping), and reserve dcat:Dataset for cases where you’re actually producing DCAT-ish metadata (distributions, catalogs, etc.).

This avoids the trap where every internal RDF-shaped artifact becomes “a dataset in the publishing sense.”

ADSM: same logic, but even easier

If adsm: is explicitly Semantic Web–oriented and matches your intent (“this artifact’s content is an RDF dataset/graph”), it’s a better semantic fit than DCAT in many cases. Same guidance:

Use ADSM (or any RDF-data ontology) to express RDF-ness.

Use DCAT when you want publication/discovery semantics.

The key modeling decision you should make explicitly

Do you mean:

Content-kind marker: “this artifact’s payload is RDF (graphs/quads)”
or

Published dataset: “this artifact participates in catalog/distribution discovery”

Your current usage reads like (1) most of the time. DCAT is closer to (2).

If you keep both available, you get the best of both worlds:

sflo:RdfDataset = your stable content-kind class (general spine).

Optional dcat:Dataset typing = only when you’re doing DCAT-shaped publishing.

That will reduce future backtracking when you inevitably have internal RDF artifacts (logs, metadata, working slices) that you don’t want treated as publishable datasets.

For Domain of containsSemanticFlowResource, I think we keep domain = ArtifactContainer. 

Does my SHACL need any updating?
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
                  "sflo:SemanticFlowResourceFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:FlowPayload/"
              },
              "sh:qualifiedMinCount": 1,
              "sh:qualifiedMaxCount": 1
            },
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsSimplePayload/",
                  "sflo:SemanticFlowResourceFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:SimplePayload/"
              },
              "sh:qualifiedMaxCount": 0
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
                  "sflo:SemanticFlowResourceFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:SimplePayload/"
              },
              "sh:qualifiedMinCount": 1,
              "sh:qualifiedMaxCount": 1
            },
            {
              "sh:path": {
                "sh:alternativePath": [
                  "sflo:containsFlowPayload/",
                  "sflo:SemanticFlowResourceFlowResource/"
                ]
              },
              "sh:qualifiedValueShape": {
                "sh:class": "sflo:FlowPayload/"
              },
              "sh:qualifiedMaxCount": 0
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
              "sflo:SemanticFlowResourceFlowResource/"
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
              "sflo:SemanticFlowResourceFlowResource/"
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
              "sflo:containsMeshResource/"
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
              "sflo:containsMeshResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:MeshMetadataFlow/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Mesh may have at most one MeshMetadataFlow."
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
              "sflo:containsMeshResource/"
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
              "sflo:containsMeshResource/"
            ]
          },
          "sh:qualifiedValueShape": {
            "sh:class": "sflo:NomenMetadataFlow/"
          },
          "sh:qualifiedMinCount": 1,
          "sh:qualifiedMaxCount": 1,
          "sh:severity": "sh:Violation",
          "sh:message": "A Nomen may have at most one NomenMetadataFlow."
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


Keeping rdfs:domain sflo:ArtifactContainer on sflo:containsSemanticFlowResource is consistent with your intent. It also gives you a clean place to validate “container slots” without having to special-case Mesh/Knop/Nomen.

Yes: your SHACL needs updates. There are a few concrete breakages/inconsistencies in what you pasted.

1) You have a broken predicate IRI in multiple places

You use this in sh:alternativePath repeatedly:

sflo:SemanticFlowResourceFlowResource/

That is not a real property (looks like a copy/paste or refactor artifact). It should be:

sflo:containsSemanticFlowResource/

(or, if you want to accept the specialized predicates too, include those alongside it).

Example fix pattern:

"sh:path": {
  "sh:alternativePath": [
    "sflo:containsKnopMetadataFlow/",
    "sflo:containsSemanticFlowResource/"
  ]
}

2) Several shapes still reference old containment predicates

You have sflo:containsMeshResource/ in multiple places (MeshSlotsShape, NomenSlotsShape, etc.). Your ontology now uses:

sflo:containsSemanticFlowResource/

So update all “generic containment” paths accordingly.

3) Two slot constraints contradict their own English and your stated rules

You said:

inventory datasets: exactly one (Violation)

missing KnopMetadataFlow: Violation

metadata flows are generally 0..1 (unless you explicitly want them required)

But your current SHACL does:

MeshMetadataFlow: qualifiedMinCount: 1 and qualifiedMaxCount: 1 with message “may have at most one” (contradiction)

NomenMetadataFlow: same issue (and message says “may have at most one” while minCount=1 requires it)

Fix those by making metadata flows optional:

qualifiedMinCount: 0

qualifiedMaxCount: 1

severity: sh:Violation if you truly want “no more than one” to be a hard rule

Inventory datasets should remain 1..1 with sh:Violation (you already did that for Knop/Mesh/Nomen inventories).

4) If you “don’t want to assume inference,” tighten targets for containment-based validation

You already did this correctly in one place:

FlowArtifactStructureShape uses sh:targetSubjectsOf sflo:hasSlice/ — good.

For the slot shapes, you currently rely on sh:targetClass (e.g., sflo:Knop/). That’s fine only if your data explicitly types knops (no subclass entailment assumed).

If you want the shapes to fire even when rdf:type sflo:Knop is missing, switch (or add) a sh:targetSubjectsOf target, for example:

target subjects of sflo:containsFlowPayload/

target subjects of sflo:containsSimplePayload/

target subjects of sflo:containsSemanticFlowResource/ (broad; may be too broad)

I would keep sh:targetClass for now if your authoring pipeline always asserts types; it’s clearer and cheaper.

5) The KnopPayloadChoiceShape logic should be simplified after the refactor

Right now it tries to accept either the specific properties or the generic predicate, but because the generic predicate IRI is wrong (see #1), it’s effectively broken.

Also: using the generic predicate for payload detection requires qualified counts by class, which you are already doing—good approach.

So: just fix the alternativePath lists to include the correct generic containment predicate.

Summary of required edits

Replace every sflo:SemanticFlowResourceFlowResource/ with sflo:containsSemanticFlowResource/.

Replace every sflo:containsMeshResource/ with sflo:containsSemanticFlowResource/ (unless you truly kept both).

Make Mesh/Nomen metadata-flow slots 0..1 (unless you want them required).

Ensure all “must/should” severities match your current policy (you said inventories + missing KnopMetadataFlow should be Violation: your KnopSlots already does that; Mesh/Nomen inventory slots do too).

If you want, paste just the three “SlotsShape” nodes (KnopSlotsShape/MeshSlotsShape/NomenSlotsShape) and I’ll rewrite those sections cleanly in JSON-LD with the corrected predicates and cardinalities.

Can you regenerate KnopPayloadChoiceShape for me?
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
}