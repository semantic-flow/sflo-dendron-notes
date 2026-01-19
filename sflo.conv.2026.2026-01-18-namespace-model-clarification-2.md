---
id: qspixtdcbxqci0la0emr9sx
title: 2026 01 18 Namespace Model Clarification 2
desc: ''
updated: 1768844806156
created: 1768760433082
---

MeshResource is good. MeshComponents include: Knops, MeshMetadataFlow, OpslogDatasetSeries, I suppose IdentityFlows and ???; A sublcass of MeshComponents is KnopComponents, which include PayloadFlow, KnopMetadataFlow, KnopInventoryFlow, PointersDataset, 

Should all MeshResources have an _identity-flow? there was some closely related data that we were thinking of separating into its own flow.


1) Taxonomy: keep it tight or you’ll explode the concept count

Your draft hierarchy is directionally right, but you’re mixing (a) kinds of resources with (b) specific flows/datasets you plan to implement. That’s a maintainability risk. Define a small, stable class taxonomy, and let the “which flows exist” be a profile/convention.

Suggested class skeleton

MeshResource

Nomen (user-facing named path; may denote world-things or artifacts)

MeshComponent (reserved _… subtrees; operational/system meaning)

Knop (UUID-named; anchor for flows/components)

MeshSystemComponent (everything else under /_mesh/ not a knop)

KnopComponent (components under a Knop; also MeshComponents)

This avoids needing “MeshMetadataFlow” etc. as classes. Those can be instances of KnopComponents or MeshSystemComponents.

Where your examples land

Knops → MeshComponent ∩ Knop

MeshMetadataFlow / OpslogDatasetSeries → MeshSystemComponent (not separate top-level classes)

PayloadFlow / KnopMetadataFlow / KnopInventoryFlow / PointersDataset → KnopComponent

You can still name them in docs as “standard components,” but avoid making each a first-class ontology class unless you need it for reasoning/validation.

2) Should all MeshResources have an _identity-flow?

Not all. If you require it for everything, you force:

empty boilerplate for every leaf

churn on filesystem operations

more “special cases” (what’s the identity-flow for a purely structural container Nomen?)

You should require _identity-flow only where it carries real, stable meaning.

Recommended rule

Only “identity-bearing” MeshResources require _identity-flow.

Define “identity-bearing” precisely:

Mesh root Nomen: yes, mandatory
This is where mesh-level identity + operational defaults live.

Nomina: optional, but encouraged when you need any of:

sameDenotationAs

hasReferenceData / canonicalReferenceSource

supersedes

publishedAs assertions

extraction overrides
If a Nomen has none of these, having _identity-flow is pure overhead.

Knops: yes, mandatory (but probably named differently under the Knop)
Knops need a stable UUID, maintainer/provenance, inventory pointers, etc. That’s identity-bearing.

KnopComponents: generally no, unless they are independently versioned/addressed and you want metadata about the component itself (rare; you can usually keep metadata in the KnopMetadataFlow or in the component’s own catalog/descriptor).

MeshSystemComponents (e.g., ops logs, global inventory): yes for the component(s) that are themselves versioned and need provenance; no for purely structural folders.

This keeps the system sane.

3) The “closely related data” you were separating: split Identity vs Operations

You’re probably referring to the separation between:

identity/denotation mapping (what a Nomen is about, equivalences, reference sources)
and

operational state/logs/inventories (what was found, verified, cached, last-published checks)

Those should not be the same flow, because operational data changes often and pollutes history.

Strong recommendation: two flows

For any resource where you care:

_identity-flow (low churn, human-authored, semantic claims)

sameDenotationAs

hasReferenceData / canonical subset

supersedes

declared published targets (optional)

_ops-flow (high churn, tool-authored, purely operational)

verification results (“URL existed at T”)

computed inventories

extraction caches (if you add them)

weave-run metadata / diagnostics

This is the “closely related but should be separate” split you’re gesturing at.

Where _ops-flow should exist

Mesh root: yes (ops log, publish verification, scan results)

Knops: yes (inventory, pointers verification, internal indexing)

Nomina: optional (only if you’re doing per-nomen verification/extraction caching)

If you don’t want another flow everywhere, make _ops-flow allowed but not required, and concentrate it at mesh root + knops.

4) Where the pieces you listed should live
Under <meshRoot>/_mesh/ (MeshSystemComponents)

_mesh/_identity-flow/ (mesh identity & declarative config)

_mesh/_ops-flow/ (ops logs, weave history, verification)

_mesh/_knops/ (knop store)

optional _mesh/_inventory-flow/ if you want inventories separate from ops logs

Under a Knop <.../_mesh/_knops/<uuid>/ (KnopComponents)

_identity-flow (knop UUID, maintainer/provenance, “is reference knop” vs “payload knob” if you still care)

_inventory-flow (or _ops-flow) (computed inventories)

_pointers dataset/series (if it’s declarative pointers)

payload flow(s) (if present)

You can keep “PointersDataset” as a named component without giving it its own ontology class if you don’t need it.

5) A concrete policy you can put in docs (normative and short)

Mesh root Nomen MUST have _mesh/_identity-flow.

Every Knop MUST have _identity-flow.

Nomina MAY have _identity-flow; if absent, they have no explicit identity assertions beyond their existence.

Operational/tool-written facts MUST NOT go in _identity-flow; they go in _ops-flow (mesh-level and/or knob-level).

This directly addresses your “should all have it?” question and preserves clarity.

If you want the next step: I can draft a single-page “MeshResource and MeshComponent taxonomy + required components” section, including an explicit table of what is MUST/SHOULD/MAY for mesh root, Nomen, Knop, and KnopComponent.

Knobs can be completely immutable. Nomen change all the time, but maintain history. This new approach gives us stronger immutability, what others 

You do get materially stronger immutability from the “Nomen ↔ Knop separation + immutable knops” move. Beyond “knops don’t change,” the main additional strengths are these:

1) You can make all edits become naming/mapping edits

If knops are immutable, then content evolution is always expressed as:

new knop(s) (new immutable artifacts)

updated Nomen identity-flow (sameDenotationAs / supersedes / reference sources)

That gives you a clean audit boundary:

knop history = append-only artifact graph

nomen history = narrative/history of naming, equivalence, and what sources to consult

This sharply reduces “what changed?” ambiguity compared to editing content in place.

2) You eliminate “filesystem mutation ambiguity”

Previously, you couldn’t tell:

rename vs copy vs move

whether identity should follow the folder or the data

With immutable knops:

copying a knob produces a new knob (or is detectable as a “reuse”)

renaming a Nomen is explicitly a nomen history event (supersedes/sameDenotationAs)

You stop relying on brittle filesystem heuristics.

3) Stronger composition: you can reuse knops safely across meshes

Immutability makes cross-mesh reuse non-scary:

two meshes can reference the same reference knop (shared ontology source)

you don’t worry about one mesh “updating” it and breaking another

This is especially valuable for:

ontology extraction

provenance-per-source knops

curated reference packs

4) Better caching and deterministic rebuilds

If knops never change, then:

any derived artifact (indexes, extracts, rendered pages) can be cached keyed by knop UUID + selection parameters

rebuild is deterministic and incremental:

only Nomina whose identity-flow changed need re-rendering

only new knops need scanning/indexing

This is a real operational win for weave-time performance.

5) Stronger security and trust properties

Immutability enables:

content-addressing or hashing (even if you don’t expose it)

signing/attestation of knops

tamper evidence (“this knob UUID corresponds to these exact bytes/quads”)

Even if you don’t implement signatures now, the architecture makes them feasible later.

6) Concurrency becomes tractable

Multiple contributors can:

add new knops independently

update Nomina mappings independently (with merge conflicts only in identity-flows)

You avoid concurrent edits to the same payload artifact.

Remaining conflicts are semantic (“which knob should this Nomen point to?”), not low-level file conflicts.

7) Cleaner provenance modeling

Because knops are immutable:

provenance claims can attach to the knob once and remain valid

Nomina can say “these are my reference sources” without rewriting those sources

This aligns well with “one reference knop per provenance source.”

8) You can enforce single-referent discipline more reliably

A Nomen can “change all the time” while maintaining history, but with immutable knops you can enforce:

at any given Nomen version, the set of reference sources is explicit

“sameDenotationAs” is explicit and auditable

accidental identity drift is easier to detect (e.g., mapping changes without supersedes)

In mutable systems, identity drift happens invisibly inside edited content.

9) The cost (important): you will mint more objects and need garbage strategy

Stronger immutability implies:

more knops over time

more stale/unreferenced knops

need for retention policies (“reachable from any Nomen history?”)

need for “bundle/export” tools

This isn’t a reason to avoid it, but it is the operational price.

If you want one sentence for docs:
“Immutable knops turn all evolution into explicit, versioned changes in nomen identity graphs, making composition, caching, provenance, and auditability deterministic.”

This k hater. Knops are exactly immutable, they have flows with slices. Slices are immutable. 

Your correction is right: “knops are immutable” is too coarse. The precise statement is:

A Knop is stable as an anchor, but it accumulates flows over time.

Flows are sequences of versions (slices).

Slices are immutable.

Therefore: immutability is at the slice level, not at the knop level.

That distinction matters because it changes what you can promise operationally and semantically.

What “stronger immutability” really becomes in this model
1) No in-place mutation of published states

If a slice is the unit of publication (or at least the unit of stable reference), then:

any change creates a new slice

old slices remain available and unchanged

So the stronger guarantee is: published referents are immutable (slice-identified resources), even though the knop continues to grow.

2) Clear separation between “current” and “historical”

You can allow a mutable working state (e.g., _next) while guaranteeing:

_vN slices never change

“current” is a pointer to a slice (or a computed view), not mutable content

This lets you say: only pointers move; slices do not.

3) Deterministic rebuild/caching keyed by slice IDs

All the caching/provenance advantages I listed still apply, but the stable key is:

(knop, flow, slice-id) (and format selection), not just knop UUID.

So the real benefit is append-only versioning of immutable snapshots.

4) Composition safety is “slice-safe,” not “knop-safe”

When you reuse something across meshes, the stable reused unit should be:

a specific slice (or a declared range), not “the knop,” because the knop can gain future slices.

So your reuse semantics become explicit:

“reuse this flow slice” vs “track current.”

5) Concurrency/merge shifts to pointer updates

Conflicts move to:

“which slice becomes current”

“which sources are canonicalReferenceSource”

“which slice range is included”

Not to editing historical data.

The doc wording you probably want

“Knops are stable anchors that host versioned flows.”

“Flows evolve by appending immutable slices.”

“Any published slice is immutable and permanently referencable.”

“Mutable working data may exist, but publication always produces a new slice.”

That’s the accurate immutability story.

A concrete pitfall to avoid

If you keep saying “knops are immutable,” people will infer:

“a knop is content-addressed / never changes”

“a knob is a snapshot”

That will contradict your flows/slices model and confuse the “current” pointer semantics.

So: reserve “immutable” for slice (and for any content-addressed artifact if you later add hashing/signing).

If you want, I can rewrite the immutability section in your architecture doc to use the correct hierarchy of invariants (knop stable identity, flow append-only, slice immutable, pointers movable).

I'm not sure about "Your draft hierarchy is directionally right, but you’re mixing (a) kinds of resources with (b) specific flows/datasets you plan to implement. That’s a maintainability risk. Define a small, stable class taxonomy, and let the “which flows exist” be a profile/convention." -- Things like PayloadFlows and IdentityFlow are primary constituents of the system. We need them for SHACL checking. 
---
Could you put this in a canvas:

- MeshResource - Anything contained in a semantic mesh. All meshes have contextualized identifiers and a place in the filesystem, and a 1-1 mapping between those things.
  - LocatedFile - the IRI directly corresponds to a concrete, downloadable representation. The IRI of a LocatedFile denotes the file it corresponds to; usually ends with an extension
  - AbstractResource - Everything else in a mesh; IRIs end with a slash (/) and de-referencing returns a resource page describing the abstract resource, (metadata), and potentially if the AbstractResource is a DigitalArtifact, where it can be accessed
    - Flow - a pseudo-immutable DigitalArtifact over time. 
    - Slice - a particular version of the DigitalArtifact represented by a Flow
    - AbstractFile - a specific representation of a slice (syntax, order, checksum, bundling)
    - LocatedFile - a place where an AbstractFile can be access.
    - 

Mesh Resource Taxonomy Draft
I was formerly thinking to have non-flow RdfDataset for things like OpsLogs that don't evolve over time. But it's definitely simpler to just give every DigitalArtifact a Flow. If it doesn't evolve, so be it... there's not that much extra overhead. Pointers is another one that I wanted to make easily discoverable, but I'm not even sure we need pointers... the "_working" slice is always the latest version. Sometime it'll be identical to the latest "woven" Slice, but that's okay. 
Thought for 17s

Making every DigitalArtifact have a Flow is a net simplification, and it aligns with your SHACL/checking instincts. The “overhead” is mostly conceptual naming and a couple extra folders; operationally it buys you uniform resolution rules.

Two things you’re currently papering over will cause real confusion unless you pin them down:

1) A “_working slice” is a contradiction unless you mean “_working dataset”

If slices are immutable, then “working slice is always the latest version” can only be true if every edit creates a new slice. That is possible, but it implies:

lots of micro-slices (noisy history)

unclear meaning of “woven slice” vs “working slice” if both are immutable snapshots

clients can’t know whether they’re looking at stable/published vs draft unless you add a status signal anyway

What you likely want is:

_working = mutable working dataset (not a slice)

weaving freezes _working into a new immutable slice vN

If you still want “_working is a slice,” then you need to explicitly accept “every change creates a new slice,” and treat “working” as a stage label (draft) on immutable snapshots.

2) Dropping “pointers” only works if you replace it with an equally-clear “current published” rule

You’re saying: “maybe we don’t need pointers; _working is always latest; sometimes identical to latest woven.”

That doesn’t answer the client question: what is the latest published slice?

If you remove an explicit pointer, you must specify one of these resolution rules:

Rule A (scan rule): “current published slice = highest published version number present under the flow.”
Downsides: requires listing, can’t represent “publish v7, but v8 exists as draft slice,” unless you separate draft/published folders.

Rule B (status rule): “slices carry a status (draft/published), and current published is the highest slice with status=published.”
Downside: still needs querying metadata; you’ve reinvented a pointer but distributed.

Rule C (pointer rule): keep a tiny, cheap pointer dataset/flow that says:

currentPublishedSlice

optionally currentWorkingState (either _working dataset or “draft head” slice)

This “pointer” doesn’t have to be a big concept; it’s the minimum needed to avoid ambiguous resolution. If you truly want zero pointers, you must accept a scan/metadata query requirement for clients.

OpsLogs as Flow: good, but be explicit about the expectation

If OpsLogs “don’t evolve,” they still do evolve in practice (appends). The only case they don’t evolve is “never written again,” which is rare. So treating OpsLogs as a Flow is actually more honest: it’s an append-only artifact where each weave can produce a new slice, or you keep it as mutable _working and periodically freeze.

Recommendation consistent with your direction

Every DigitalArtifact has a Flow.

Keep _working as mutable working dataset, not a slice.

On weave: freeze to immutable vN.

Keep a minimal “current published” pointer (call it whatever you want, but it’s functionally a pointer) unless you’re willing to force clients to scan/version-compare.

If you want, I can propose the minimal SHACL constraints for:

“Flow MUST have exactly one working dataset” (or draft head)

“Slices MUST be immutable and numbered”

“Flow MUST declare currentPublishedSlice” (if you keep the pointer)

Mesh Resource Taxonomy Draft
Maybe HistoricalSlices are immutable but WorkingSlices are not? Doesn't seem unreasonable. Publishing happens independently of weaving, you could git push a repo without weaving to publish the latest version of a DigitalArtifact. In fact, when working locally, applications might usually use the WorkingSlice and only check HistoricalSlices rarely.

The latest HistoricalSlice may be ascertained from the KnopMetadata Flow. 

For things like OpsLogs, I'd anticipated an RdfDatasetSeries. Like /_knops/<knop-uuid>/opslog/ could refer to the series, and depending on opslog settings you'd have a new opslog artifact every day, or at a certain size. 

But DatasetSeries can be accomplished with a Knop whose PayloadFlow is an RDF datasetseries description. 

Keep in mind that DigitalArtifacts are not necessarily RdfDatasets, so _working is a just the current version of the digitalartifact. Knops contain a PayloadFlow and SupportingFlows. Nomen contain only SupportingFlows, and probably just one, the _nomen-flow.  (formerly called the identity-flow, but Nomen self-identify, their metadata is more about what they denote (if a digital artifact), their co-denoters (if any), and any reference RdfDatasets (canonical or not).

If you allow WorkingSlices to be mutable and HistoricalSlices to be immutable, you get a coherent model that matches “publishing can happen without weaving.” The key is to stop pretending “slice” implies immutability and make the distinction explicit in SHACL and terminology.

Main benefits of this shift (beyond convenience):

Unblocks publish-without-weave: a git push can publish the current working state without forcing a new historical snapshot.

Keeps strong archival guarantees: historical slices remain stable reference points for audit, reproducibility, and caching.

Lets apps default to working state: local tooling can stay fast (operate on WorkingSlice) while only consulting historical slices when needed.

Makes “current” semantics unambiguous: “current working” is always defined; “latest historical” is separately defined (and can be declared in KnopMetadataFlow).

Two sharp edges you should lock down:

Vocabulary discipline
If “Slice” includes mutable working, then you need either:

WorkingSlice / HistoricalSlice as distinct classes, or

rename “WorkingSlice” to something like “WorkingState” (but you seem okay with the slice split).

Publish semantics
Define precisely what it means to publish:

publishing a working slice = exposing mutable state as of the push (not stable)

publishing a historical slice = exposing a stable version (stable)

Without this, clients won’t know whether they can cache forever.

On OpsLogs / series: your “DatasetSeries via Knop whose PayloadFlow describes the series” is consistent and avoids special-casing RdfDatasetSeries as non-flow. Rolling policies (daily/size) become operational behavior, while the series description remains stable RDF.

On your refined split:

Knops contain PayloadFlow + SupportingFlows.

Nomina contain SupportingFlows only, primarily _nomen-flow (renaming identity-flow), which is about denotation/co-denotation/reference sources, not ops facts.

I updated the canvas taxonomy to reflect: WorkingSlice vs HistoricalSlice, publish independent of weave, NomenFlow vs Knop payload/supporting flows, and series/logs as artifacts.

Mesh Resource Taxonomy Draft
Maybe we don't need identity-flow on Knops since their IRIs always denote themselves. 

One thing I settled on long ago, was that the inventory for the various flows contained in a Knop would be inventoried and described in the KnopMetadataFlow. This solved the problem of an infinite recursion of flows describing metadata of flows, and those metadata flows needing their own flows. But if we have a KnopMetadataFlow for that on the Knop side, we might need it on the Nomen side. Or else Nomen have to contain their own metadata, i.e., in the NomenFlow. 
Thought for 19s

You can drop a distinct “identity-flow” on Knops, but only if you’re willing to make KnopMetadataFlow carry the self-description + inventory role explicitly.

1) “Knop IRIs denote themselves” is not sufficient for SHACL/discovery

An IRI denoting itself gives you referent identity, but SHACL/checking typically needs asserted structure:

what flows exist in this knop (and their IRIs)

which one is the payload flow (if any)

which slices are historical vs working

what the “latest historical” is (your point)

optional constraints/expectations (e.g., “payload flow must exist for this knop kind”)

So you still need a place where the knop says what it contains. If you already settled on KnopMetadataFlow for that, you don’t need a separate identity-flow.

Conclusion: IdentityFlow-on-knop is redundant if KnopMetadataFlow is mandatory and includes knop self-description.

2) The “infinite recursion” problem is solved by making KnopMetadataFlow the base case

Your earlier solution works, but only if you treat it as normative:

KnopMetadataFlow inventories all flows in the knop, including itself, in a single graph/series.

You do not need “metadata about the KnopMetadataFlow” in another flow. You just include a record about that flow in the inventory it already publishes. That’s the recursion stopper.

If you don’t allow it to describe itself, you reintroduce the recursion.

3) Do you need the same pattern for Nomina?

Only if Nomina can have multiple supporting flows and you want SHACL to validate “what exists.”

Two clean options:

Option A (simplest): NomenFlow is the inventory

Nomen has exactly one required supporting flow: _nomen-flow (your rename).

If you later add more Nomen supporting flows, _nomen-flow also inventories them.

This avoids creating “NomenMetadataFlow” as a second required flow.

Option B: Mirror the knop pattern

Require NomenMetadataFlow that inventories Nomen supporting flows (including itself).

Keep _nomen-flow semantically “about denotation / sameDenotationAs / reference data,” not about structure.

This is more orthogonal, but it’s extra machinery unless you really expect many Nomen-side flows.

Given your current direction (“Nomen contain only SupportingFlows, probably just one”), Option A is the consistent move.

4) The “closely related data” you were separating

The right split isn’t “identity vs metadata”; it’s:

declarative semantics (denotation, sameDenotationAs, reference sources) → NomenFlow

structural inventory/pointers (what flows/slices exist, which is latest historical) → KnopMetadataFlow (and optionally a Nomen inventory section only if needed)

operational facts (verification, last-seen URLs, extraction warnings) → separate ops log / ops flow (mesh-level or knop-level)

If you mix ops facts into NomenFlow/KnopMetadataFlow, your histories will become noise.

5) Recommendation in one sentence

Make KnopMetadataFlow mandatory and let it self-describe + inventory everything in the knop (including itself); make NomenFlow mandatory for Nomina, and only add Nomen-side inventory if/when you actually have >1 Nomen supporting flow.

If you want, I can update the canvas taxonomy to (a) remove any implication that Knops need a separate identity-flow, and (b) state explicitly that KnopMetadataFlow is the recursion base case and inventories itself.

Mesh Resource Taxonomy Draft
We could call the NomenFlow just NomenMetadataFlow for symmetry. I think we only need one Flow on Nomen. 

The flows we've considered:
- config: templates and css used to build ResourcePages; whether inheritable conflig should be accepted from a parent; BUT if we keep config in a separate knop (as Payload), we can just keep config as a property of a SemanticMesh, Knop, or Nomen.
- opslog: We can probably keep this at the mesh level. 
- pointers: might not be needed at all, but if we allow flexible naming of a Knop's LocatedFiles based on a "default token" or override, we'll need something (maybe in the KnopMetadataFlow) that points to the current in-mesh LocatedFiles. I think NomenMetadata should always the NomenToken (fixed), so e.g., <nomen>/_meta/_working/<nomen>_meta.jsonld
- backlinks: curated or computed backlinks -- could reasonably go in the MetadataFlow
- Inventory: which Flows, Slices, AbstractFiles, and LocatedFiles exist; optional external mirrors.



Thought for 12s

The canvas update failed because my regex patterns didn’t match the exact text currently in the document, so I couldn’t apply the rename/section replacement automatically. I attempted to (1) rename NomenFlow → NomenMetadataFlow, (2) rewrite the Nomen flow description to be “single required flow,” and (3) replace the MeshComponent/KnopComponent section with an expanded “standard supporting artifacts” section. The tool couldn’t find the targeted blocks.

Paste-ready replacements below (you can drop them into the canvas manually).

1) Replace the Nomen flow block with this (symmetry + single flow)

Replace the current Nomen subsection under Naming and anchoring with:

Nomen (AbstractResource) — A human-facing identifier in the mesh address space. A Nomen can denote a real-world thing or (optionally) a DigitalArtifact. Nomina do not contain payload flows.

NomenMetadataFlow (SupportingFlow; formerly “identity-flow”) — The single required flow attached to a Nomen (for symmetry with KnopMetadataFlow). It is about:

what the Nomen denotes (when the denotation is an IRI-identifiable thing or a DigitalArtifact),

sameDenotationAs links (co-denoters),

reference RDF datasets/sources (canonical or not),

publishing-context assertions/expectations (optional),

curated backlinks (optional),

not operational facts (verification, crawl results, computed indexes).

Hyper-critical point: “backlinks” here must be curated or extremely low-churn. Computed backlinks will thrash history and mix ops into semantics.

2) Your flow list: what belongs where (and the key traps)
Config

You’re oscillating between “config flow” and “config as property.” Pick one, or you’ll create partial duplication.

Best simplification: no dedicated config flow. Put config pointers/properties in:

mesh-level metadata (global defaults),

plus optional overrides in KnopMetadataFlow / NomenMetadataFlow.

If you want configs as artifacts (versioned bundles), represent them as a config Knop (payload) and reference it from the metadata flows.

Opslog

Mesh-level is right. Do not put opslog content into NomenMetadataFlow or KnopMetadataFlow.

If you want “roll daily/size,” model it as:

either a Flow with WorkingSlice/HistoricalSlices, or

a DatasetSeries described by a Knop payload (as you said).

But be consistent: if “everything is a Flow,” then opslog rollover is just “slice policy.”

Inventory

Your older decision is correct and still the cleanest recursion stopper:

KnopMetadataFlow inventories which Flows/Slices/AbstractFiles/LocatedFiles exist (including itself).

It also exposes “latest historical slice” so clients don’t need directory scanning.

You probably don’t need a separate KnopInventoryFlow unless you want to split “structural inventory” from “semantic metadata” (but you already have the recursion constraint pushing you toward one place).

Pointers

If pointers are only “current in-mesh located files,” they can be a subsection of KnopMetadataFlow inventory, not a separate dataset.

But your reason for pointers (“flexible naming based on a default token/override”) means you need at least one stable mapping primitive:

from (slice, abstractfile) → locatedfile(s) in this mesh context

If you don’t have that mapping, then discovery becomes “scan the filesystem and infer,” which breaks as soon as mirrors exist or naming is customized.

So: drop a separate “PointersDataset” only if KnopMetadataFlow inventory includes those mappings.

Backlinks

Split the concept:

Curated backlinks (author asserts “see also”) → metadata flows are fine.

Computed backlinks (index result) → belongs in ops/index artifacts, not metadata flows.

Otherwise you will (a) churn histories and (b) make Nomen semantics depend on which indexer ran last.

“NomenMetadata should always the NomenToken (fixed) …”

This is fine, but you’re mixing “token naming” with “resource identity.” The safe rule is:

_meta (or _nomen-flow) distributions may use the NomenToken purely as a filename convenience, but identity is the IRI, not the filename.

Also, _meta paths are MeshComponents, not regular Nomina—still consistent with your reserved underscore rule.

3) Replace the MeshComponent / KnopComponent section with this expanded one
Mesh components (reserved / underscore paths)

MeshComponent (AbstractResource) — Reserved operational/system resources under underscore paths (not regular Nomina), including:

Knops (UUID-named),

KnopComponents (payload + supporting flows and their components),

mesh-level operational containers (e.g., _mesh handle folders, ops logs, inventories).

KnopComponent (MeshComponent) — A component under a Knop, including:

PayloadFlow

KnopMetadataFlow

(optional) other supporting flows/components

Standard supporting artifacts (design notes)

These are primary constituents (useful for SHACL), but they don’t all need separate flows if their function can live inside KnopMetadataFlow / NomenMetadataFlow.

Configuration — Templates/CSS for ResourcePages; inheritance policy; presentation controls.

Prefer: declare config as properties of SemanticMesh, Knop, or Nomen (in metadata flows).

Alternative: a dedicated config Knop whose payload is the configuration artifact.

OpsLog — Tool-written operational log data (high churn).

Prefer: mesh-level artifact(s), possibly rolled.

Current LocatedFile mappings (optional “pointers”)

Prefer: include mappings in KnopMetadataFlow inventory if you support flexible naming or mirrors.

Backlinks

Curated → metadata flows.

Computed → ops/index artifacts.

Inventory

Authoritative at KnopMetadataFlow: flows/slices/abstractfiles/locatedfiles; optional external mirrors; latest historical slice.

4) On your earlier maintainability concern

You’re right that PayloadFlow / KnopMetadataFlow / NomenMetadataFlow are “primary constituents” and should be in SHACL. The maintainability risk isn’t “don’t model them”; it’s “don’t let every operational convention become a new ontology class.” The compromise is:

keep a small set of stable constituent classes (Flow, Slice, KnopMetadataFlow, NomenMetadataFlow, AbstractFile, LocatedFile, etc.)

model “opslog rollover daily” or “computed backlinks index” as instances/config + SHACL shapes, not as new top-level classes.

If you want, tell me whether you prefer the Nomen metadata flow path to be /_meta/ or /_nomen/ (or something else). I won’t ask follow-ups beyond that; I can provide a single canonical folder layout for Nomen + Knop that matches the decisions above.

Mesh Resource Taxonomy Draft
So only 3 types of flows? It's tempting to put most inventory (exhaustive list of every slice and every slice's files) in a separate flow as that could get somewhat large. And just have the pointer the current LocatedFile in metadata for ease of access. Especially if we attach provenance info the each slice, it'd be better to keep that separate?

I think only PayloadFlow distribution allow custom LocatedFile naming. MetadataFlows should definitely have standardized names so we don't need a pointer to find them, e.g. /_knops/<knop-uuid>/_meta/_working/meta.jsonld.

Should NomenMetadataFlows have customized distribution Filenames? Probably not, for the same reason: ease of discovery.
Thought for 17s

No, it does not have to be “only 3 flow types.” You have (at least) two orthogonal axes:

Role (payload vs metadata vs inventory vs opslog)

Artifact family (Flow/Slice/AbstractFile/LocatedFile)

What you can simplify to is: “for each resource, keep the small, stable stuff in metadata, and push the exhaustive, high-churn stuff into inventory/ops artifacts.”

Inventory bloat + per-slice provenance: separate it

Your instinct is right. If you attach provenance to each slice and also list every slice and every file, your “metadata” becomes the worst of all worlds (huge + frequently changing).

A clean split:

KnopMetadataFlow (mandatory, small, standardized naming)

inventories which flows exist in the knop (including itself)

points to current working slice, latest historical slice

contains lightweight “current located file” pointers for convenience

optionally includes a summary inventory, not exhaustive

KnopInventoryFlow (optional but likely necessary for any non-trivial artifact)

exhaustive list of slices + per-slice AbstractFiles + LocatedFiles

per-slice provenance

external mirrors

can be sharded/partitioned if it grows (per-slice inventory artifacts, rolling inventories, etc.)

This preserves your “no infinite recursion” principle because KnopMetadataFlow is the base case that can describe itself and point outward.

Standardized filenames for metadata flows: agree

Yes:

PayloadFlow distributions may have custom LocatedFile naming.

Metadata flows should not. They should have standardized names so discovery never needs pointers.

Your proposed pattern is good, just make it fully normative:

Knop metadata: /_knops/<uuid>/_meta/_working/meta.jsonld (or whatever you standardize)

Nomen metadata: similarly standardized under the Nomen, e.g. <nomen>/_meta/_working/meta.jsonld

And yes: NomenMetadataFlow should not have customized distribution filenames for the same reason—discovery and tooling determinism.

Pointers: keep only the minimum, inside metadata

If “pointers” means “map from abstract representation to the current in-mesh LocatedFile(s),” that can live inside KnopMetadataFlow as a small mapping. You only need a separate PointersDataset if you expect it to become large/high-churn, in which case it’s really an inventory/index and belongs with KnopInventoryFlow anyway.

I updated the canvas to reflect:

NomenFlow → NomenMetadataFlow as the single required Nomen flow

optional KnopInventoryFlow for exhaustive inventory + per-slice provenance

standardized filenames for metadata flows; custom naming restricted to payload distributions

Mesh Resource Taxonomy Draft
I think Knops now MUST have a PayloadFlow.

We talked last week about Datasets vs Named Graphs. How, in some sense, a named graph can't (or shouldn't) evolve. Let's re-examing that. It comes up because I'm thinking for Knops that contain RDF data, is the PayloadFlow a Dataset? Yes. Could it be leveraged to support NamedGraphs?
Thought for 19s

If Knops MUST have a PayloadFlow, you’ve effectively decided: “every knop is an anchor for some primary digital artifact.” That is consistent with reference-knops too: the payload can simply be “the reference dataset artifact” (or another non-RDF artifact). The danger is thinking “payload” implies “denotes the same as the Nomen”; it doesn’t. Payload is “what the knop primarily carries.”

Datasets vs Named Graphs: the crux

A named graph “shouldn’t evolve” only if you are using the graph name IRI as an identifier for a specific graph snapshot.

But in RDF practice, the same graph name is often used to mean “the graph over time,” i.e., an abstract graph identity whose content can vary.

So you need to pick (or support both) explicitly:

Model 1: Graph-name IRI denotes an abstract graph (can evolve)

Graph name IRI = identity of “the graph”

Versions are represented by your Flow/Slice mechanism

Across slices, you may reuse the same graph name in the dataset, but its triples can differ

This does not violate your slice immutability, because what changes is “which slice you’re looking at,” not the slice itself.

What you must not do: claim that the graph name IRI denotes an immutable snapshot. It doesn’t in this model.

Model 2: Graph-name IRI denotes an immutable graph snapshot

Then the graph name IRI must be slice-qualified (or otherwise version-qualified)

Each slice mints (or corresponds to) a distinct named graph IRI

A “current graph” is a pointer (or “working”) selection, not the same IRI

This aligns with “named graph shouldn’t evolve” and with your single-referent instinct.

Can the PayloadFlow (as Dataset) “support named graphs”?

Yes, in two non-confusing ways.

Pattern A: PayloadFlow is an RdfDataset Flow

Each Slice is a dataset snapshot (immutable historical; mutable working if you keep that)

Named graphs inside the dataset can be:

abstract identities (Model 1), or

snapshot identities (Model 2) depending on naming policy

Where your earlier “named graphs shouldn’t evolve” bites:
If you want Model 2, then your dataset slice should contain graphs whose names are slice IRIs (or otherwise versioned IRIs). Otherwise you’re implicitly doing Model 1.

Pattern B: PayloadFlow is a NamedGraph Flow (graph is the primary artifact)

The knop’s payload is the graph itself

A dataset is then a separate artifact that can “bundle” graphs (or you just publish multiple graphs independently)

This is cleaner when the thing you really want to treat as first-class is “the graph,” not “a bundle.”

The hard inconsistency you must resolve

You’re also saying: “Publishing happens independently of weaving; you can git push without weaving.”

If you simultaneously say “named graph IRIs denote immutable snapshots,” then publishing a mutable working state under a stable graph name IRI is a semantic violation.

So you need one of these guardrails:

Guardrail 1: only historical slices are publishable under stable, dereferenceable IRIs; working state is local/operational only.

Guardrail 2: allow publishing working state, but it must be under an explicitly “working/draft” IRI (not the same IRI as any immutable snapshot).

Guardrail 3: accept Model 1 for graph names (abstract evolving graph identity), and stop claiming “named graphs shouldn’t evolve.”

You can’t have “publish working state” + “graph-name denotes immutable snapshot” without splitting the IRIs.

Practical recommendation (fits your current architecture)

Treat Flow + Slice as the versioning mechanism for both Datasets and Named Graphs.

Decide per artifact which semantics you want:

Dataset payloads: usually fine as Flow of datasets.

Named graphs: if you care about immutability, use slice-qualified graph IRIs (Model 2).

Keep “latest historical slice” discoverable from KnopMetadataFlow (as you said), so clients can avoid scanning.

I updated the canvas taxonomy to (a) make PayloadFlow mandatory at Knop, and (b) introduce RdfDataset and NamedGraph as DigitalArtifact subtypes with the “abstract identity vs snapshot identity” distinction.

If you want to go further: state one normative rule in the spec—either “graph name IRIs are abstract identities” (Model 1) or “graph name IRIs for published graphs MUST be slice-qualified” (Model 2). Without that, your system will behave inconsistently across meshes.

Mesh Resource Taxonomy Draft
Actually “payload” does imply “denotes the same as the Nomen” (the Nomen that point to the Knop). Right?

Regarding named graphs, you can name them with a Nomen, and point them to a HistoricalSlice if you want immutability, or point them to a WorkingSlice if you just want the latest version. But there's have to be some kind of mechanism to ensure that the corresponding Knop's Payload was a dataset with only a single default graph? Maybe we have a separate NamedGraphPayload that allows only a single default graph and then an RdfDatasetPayload that allows an arbitrary RDF dataset.

Named Graphs are mostly useful in the context of dataset that can contain multiple graphs, but there's a danger of repeating yourself. Maybe in a RdfDatasetFlow's Knop's metadata, you can specify includedGraphs, and the weave process constructs the distribution file by pulling in the payloads for those named graphs. 
Thought for 42s
1) Does “payload” imply “denotes the same as the Nomen”?

Only if you mean the Nomen’s primary Knop, and only for artifact-denoting Nomina.

If you make the rule:

Nomen ─primaryKnop→ Knop

Knop ─payloadFlow→ PayloadFlow

then it is consistent to require:

payloadFlow is the DigitalArtifact that the Nomen denotes (or denotesArtifact / sameDenotationAs alignment).

But that rule cannot hold for “person Nomina described by RDF,” because a person Nomen does not denote a dataset artifact; it denotes a person. So those Nomina should not have primaryKnop at all; they only have NomenMetadataFlow that lists referenceSource datasets/knops.

If you instead let “Nomen points to reference knops,” your “payload implies same denotation” breaks immediately (because reference knops’ payloads denote datasets about the person). So: don’t model reference sources as “the Nomen points to them.” Keep them as referenceSource links inside NomenMetadataFlow.

Net: “payload implies same denotation” is a good invariant for primaryKnop only.

2) Named graphs: your proposed “includeGraphs + weave assembles distribution” is the right direction

Yes: for RDF payloads, you can support named graphs without repeating yourself by making the dataset distribution a composition of graph payloads.

However, your current “NamedGraphPayload = dataset with only a single default graph” is conceptually muddy:

a named graph is “a graph + a name within a dataset”

a “single default graph” is just a graph (unnamed)

So I would split payload types like this:

Payload types (cleaner)

GraphPayload: exactly one RDF graph (default graph only; no named graphs)

DatasetPayload: an RDF dataset (default graph + any number of named graphs)

Then “named graph” becomes a role in a DatasetPayload, defined by a mapping:

graphName IRI → GraphPayload slice

This avoids the “NamedGraph is mostly useful inside a dataset but also is its own artifact” duplication trap.

If you still want a Nomen for a “named graph,” fine—make that Nomen denote the graphName IRI (the role), and its metadata points to:

the dataset knop that includes it, and

the graph payload knop that supplies its content.

3) How includedGraphs should work (to preserve immutability and avoid ambiguity)

In the RdfDatasetFlow’s KnopMetadataFlow (or inventory), define something like:

includedGraph entries:

graphName (an IRI; can be a Nomen-derived IRI)

sourceGraphKnop

whichSlice selection rule:

for working dataset: usually “use working slice of each included graph”

for historical dataset slice vN: pin exact graph slices (immutability)

On weave (or on-demand build), you materialize dataset distributions by pulling the pinned graph slices and writing a TriG/N-Quads output.

Critical constraint: for historical dataset slices, you must record the exact included graph slice IDs, otherwise a rebuilt distribution could drift.

4) Standardized filenames for metadata flows: yes, make it normative

Agreed:

MetadataFlow distributions should have fixed, predictable filenames (zero pointers required for discovery).

Custom LocatedFile naming should be restricted to payload distributions.

One gotcha: if metadata filenames include the NomenToken, you’re making them sensitive to renames. If “Nomen changes all the time but keeps history,” you’ll either:

accept that metadata filenames change on rename (fine, if stable IRI stays stable), or

avoid embedding token in filenames and keep them purely conventional (meta.jsonld), letting the IRI path carry the identity.

Given your “ease of discovery” goal, I would not embed NomenToken in the filename; put it in the path only.

5) Reconcile “Knops MUST have a PayloadFlow” with the above

This is fine if you mean: every Knop is an artifact container (even “reference knops” have a payload dataset artifact). But then you must not equate “Knop payload denotation” with “Nomen denotation” except for primaryKnop.

So the crisp rule-set is:

Every Knop has exactly one PayloadFlow (artifact identity).

A Nomen optionally has a primaryKnop; if present, Nomen denotes the same artifact as that Knop’s payload.

A Nomen also has zero or more referenceSource artifacts (knops/datasets) that are about its denoted thing.

If you want, I can update the canvas taxonomy to reflect the GraphPayload vs DatasetPayload split and the “named graph is a role” rule; it will reduce the duplication you’re worried about.

We don't need "primary Knop", because a Nomen can only "sflo:denotes" one thing. But yes, that's only in the case of DigitalArtifact-denoting Nomen. 

I'm thinking that a Nomen can denote Denote any part of a Knop, but usually denotes its PayloadFlow. It could denote a Knop if you want the Nomen to denote a Knop, but the Knop already has an IRI and it's mostly applications that would want to talk about the Knop-as-knop. The usual target of "denotes" would be the PayloadFlow which represents a digital artifact over time. 

Nomen can denote reference Knops (Knops that hold reference data), but that that's very different from saying a Nomen hasReferenceSource to a reference Knop's PayloadFlow. 


You’re collapsing “primary knop” into “sflo:denotes is functional” (0..1). That’s fine, but you still need to prevent conflation between:

what the Nomen denotes (its denoted thing), and

what describes that denoted thing (reference sources).

Right now your last paragraph risks reintroducing that conflation.

1) Make sflo:denotes functional and typed by range

Define sflo:denotes as functional (0..1) with range something like MeshResource (or a narrower “Denotable”).

Then your statement becomes precise:

A Nomen that denotes a DigitalArtifact usually denotes the PayloadFlow (not the Knop).

It can denote other knob parts (a Slice, AbstractFile, LocatedFile), but that’s a conscious choice: you are choosing a finer-grained referent.

Hyper-critical point: if you allow “Nomen can denote any part of a Knop,” you must accept denotation drift risks when tooling or humans rename/reorganize internal components. You need explicit stability guarantees for the parts you allow as denotation targets (Flows/slices are stable; transient inventories probably are not).

2) Denoting a reference Knop is not the same as having it as a reference source

You’re correct and you should codify this as two different relationships:

sflo:denotes — identity-level: “this Nomen names/identifies that thing”

sflo:referenceSource (or hasReferenceData) — aboutness-level: “this artifact contains assertions about the thing denoted by this Nomen”

These are categorically different and must not be interchangeable.

When would a Nomen denote a “reference Knop”?

Only when the denoted thing is itself a digital artifact whose purpose is “reference dataset X” (e.g., “the ACS 2023 table set,” “this provenance corpus,” “the Wikipedia extract for Alice”). In that case:

the Nomen denotes the dataset artifact (probably its PayloadFlow)

the dataset may also describe other things (including people), but that does not make it their denotation target

When would a person Nomen point to a reference knob?

Not via denotes. Via referenceSource:

/people/alice/ denotes a person (world thing)

/people/alice/ hasReferenceSource /knops/<uuid-of-wikipedia-extract>/payloadFlow (or similar)

If you instead let /people/alice/ denote that reference Knop, you’ve changed the referent from “person” to “dataset artifact,” which is exactly the conflation you say you want to avoid.

So you need a rule:

A Nomen denotes a reference Knop only if the intended referent is the reference artifact itself, not the described world-thing.

3) You probably want sflo:denotes to be “denotesArtifact” not “denotes”

Because you also say many Nomina denote world-things with no IRI. A functional sflo:denotes whose object must be an IRI won’t apply there.

So choose one of these designs:

Design A (clean): sflo:denotesArtifact only

sflo:denotesArtifact (0..1) points to a MeshResource/DigitalArtifact when the Nomen is artifact-denoting

World-thing Nomina have no denotesArtifact; they just have reference sources and co-denoters.

This matches your earlier correction: “If the thing isn’t a digital artifact, don’t use denotes.”

Design B (harder): sflo:denotes can point to “non-IRI world things”

That requires bnodes, proxies, or your “Literal Proxy”-style devices for world-things, which you’ve generally been avoiding for people. So I recommend Design A.

4) “Denote any part of a Knop” is OK if you constrain it

If you allow denoting “parts,” define the allowed targets as a closed set (SHACL):

PayloadFlow

Slice (historical or working)

AbstractFile

LocatedFile

(maybe) DatasetSeries

But not:

inventories

ops logs

“current” pointers (these are views, not stable referents)

5) How to express “Nomen denotes a reference Knop”

Do not say “Nomen denotes Knop” unless you really mean “the knob as a container.”

If your denotation target is almost always the artifact over time, you want:

denotesArtifact → PayloadFlow (even for reference knops)

and keep “knop-as-knop” addressability as an application/internal concern

That fits your “the knop already has an IRI; mostly apps care” note.

Minimal normative statements you should add to your docs

sflo:denotesArtifact is functional (0..1) and only used when a Nomen denotes a DigitalArtifact (usually a PayloadFlow).

sflo:hasReferenceSource / sflo:hasReferenceData expresses aboutness and may point to many artifacts/knops.

A Nomen that denotes a world-thing MUST NOT use denotesArtifact; it uses reference sources instead.

Denoting a reference artifact is allowed, but it changes what the Nomen is about (artifact vs world-thing).

If you want, I can turn those into SHACL shapes for:

NomenMetadataFlow constraints

allowed denotation targets

disallowing “world-thing Nomina” from having denotesArtifact (unless they’re explicitly typed as artifact-denoting).

Mesh Resource Taxonomy Draft
For includedGraphs, I think we can just specify the working slice IRI if we want mutable/latest, otherwise use a HistoricalSlice IRI. Those can be specified directly (if you want ugly graph names) or via Nomen that point to Slices if you want a readable name. Metadata filename don't care about Nomen, other than perhaps on first creation they leverage a Nomen if it's known. But you can create a Knop without creating a Nomen, right? You'll just have to choose a PayloadFilenameToken at creation, or specify one later in metadata. But it doesn't have to stay consistent, it can change. Inventory will handle that.

I love being able to support named graphs. A tricky part is, datasets have to have a default graph, it can be empty, so RdfGraph payloads must support empty graphs.

Regarding the range of sflo:denotes -- I want to make it able to denote any digital artifact anywhere, not just in a semantic mesh. And within a mesh, why constrain it? Maybe you want a humane name for your opslogs. (current pointers are now just statements in MetadataFlows) But yes, a Nomen that denotes a DigitalArtifact usually denotes the PayloadFlow if it is about "a DigitalArtifact over time", but it can also denote a specific Slice of a digital artifact or the WorkingSlice. It gives expressive power for Semantic Web discourse to be able to say "no particular version" vs "this version" vs "the latest version". Similarly, if there's a file you want to talk about on some other server somewhere, you can create a Nomen to talk about it directly, and add a ReferenceKnop to contain your discussion. So the denotesArtifact could point to something totally outside the mesh while the hasCanonicalReferenceSource can point to any URL anywhere, in any format, and weave can just do its best if the target isn't RDF. 

Separate from the metadata flow, we should have optional properties that can be included in graphs and datasets that reference their parent AbstractFile/Slice/Flow/Knop. 

Denoting other Knop parts with a Nomen is unusual, because they're not as discourse-worth as the payload. But it can be done.

It's very tempting to make the canonical distribution LocatedFiles use a standardized naming too. Then no Metadata lookups need to happen to get the actual data. But with UUIDs being so opaque, descriptive filenames is the only island of sanity in the world of Knop IRIs. 

Yes on denotesArtifact. or denotesDigitalArtifact might be better. 

We need to think about naming for the AbstractFile. It can default to the extension corresponding to its mime-type, but the mechanics are a little fuzzy.
Thought for 47s

Your direction is coherent, but there are two places you’re currently relying on “it’ll be fine, inventory will handle it” where you’ll end up with broken dereferencing unless you add a hard rule.

1) includedGraphs: yes, slice IRIs as selectors works — but don’t confuse “graph name” with “graph content selector”

What you proposed is correct if you keep these as two separate fields:

graphName: the IRI that appears in the dataset as the named-graph identifier

graphContent: which graph payload to materialize there (WorkingSlice IRI for mutable, HistoricalSlice IRI for immutable)

If you overload one IRI to do both (e.g., using the slice IRI itself as the graph name), you lose the ability to have a stable human name that still points at pinned content.

So, the best “no repetition” formulation is exactly what you said, but explicitly:

use a Nomen IRI as the graphName when you want readability

pin the graphContent selector to:

WorkingSlice IRI for “latest/mutable”

HistoricalSlice IRI for “frozen/immutable”

For historical dataset slices, the selector must be pinned to historical slices, or you will get rebuild drift.

I updated the canvas to make this split explicit.

2) “Knop without Nomen” is fine — but you must decide what happens when payload filenames change

You’re saying:

a Knop can exist without any Nomen

you pick a PayloadFilenameToken at creation (or later)

token can change; inventory maps old/new

That is only safe if you also decide whether old LocatedFiles persist.

Because in your own semantics:

LocatedFile IRIs denote concrete files.

Renaming/rewriting without keeping old files destroys old IRIs.

So you need one of these hard rules:

Rule P (preserve): once a LocatedFile IRI has been published, it stays on disk forever; new token → new LocatedFiles (old remain). Inventory maps “current” without breaking old.

Rule E (ephemeral): LocatedFiles are allowed to disappear; stability is only guaranteed at the AbstractFile/Slice level via metadata resolution.

If you don’t pick one, your spec will be internally inconsistent: you can’t promise dereferenceability while also allowing token churn that deletes old filenames.

3) RDF graph payloads must support empty graphs — agree

Yes. A dataset always has a default graph (possibly empty), so a “graph payload” should permit empty content.

This is why I would model:

RdfGraph (GraphPayload): exactly one graph (default graph only), possibly empty

RdfDataset: dataset with default graph + optional named graphs, default may be empty

Canvas now reflects that.

4) Range of denotesDigitalArtifact: don’t constrain to mesh — good

Agree. denotesDigitalArtifact should be able to point anywhere (external URLs, other servers), and Nomen can carry your discussion locally via reference knops/sources.

But you still want one constraint:

the predicate itself can point anywhere

the Nomen is still a MeshResource (so it still has your “filesystem ↔ contextual IRI” mapping)

No contradiction as long as you keep that separation.

5) Standardized metadata filenames: yes (and you can still bootstrap from token)

Agree:

Metadata flow filenames should be standardized and token-independent.

Bootstrapping from a known token is fine as a one-time convenience, but discovery must not depend on it.

Canvas now says this explicitly.

6) “Standardize canonical payload filenames” vs “island of sanity”

This is a real tradeoff:

Standardize → zero metadata lookup required to fetch “the data”

Descriptive payload filenames → human sanity in a UUID world

The only way to get both without symlinks is two LocatedFiles:

one standardized canonical name (for machines / no lookup)

one descriptive alias file (for humans), which can be an actual duplicate (bytes) or a small redirect/stub depending on medium

If you don’t want duplication, you must pick: machine-discoverable without lookup or human-friendly names. Inventory alone doesn’t solve this, because inventory requires lookup.

7) AbstractFile naming mechanics

Your instinct (“extension from MIME type, but fuzzy”) is correct. Make MIME type authoritative, extension optional.

A workable rule-set:

every AbstractFile declares:

mimeType (required)

fileExtension (optional, derived via registry/config)

LocatedFile names may use the extension when present, but clients must not rely on it.

Canvas now contains this as a design note.

I updated the canvas with:

denotation vs reference-source distinction + denotesDigitalArtifact unconstrained to mesh

GraphPayload class (empty graphs allowed)

includedGraphs composition with separate graphName and content selector

standardized metadata filenames independent of Nomen token

AbstractFile naming note

If you want the next pass to be productive: choose Rule P vs Rule E for LocatedFile persistence when payload filename tokens change. That single decision determines whether “inventory will handle it” is true or wishful.

Mesh Resource Taxonomy Draft
"If you overload one IRI to do both (e.g., using the slice IRI itself as the graph name), you lose the ability to have a stable human name that still points at pinned content." I'm imagining Nomen would normally be created for named graphs, and would denoteDigitalArtifact <SliceIRI> -- no need for separate graphContent selector.

Yes, old LocatedFiles must persist, in their Slices. They'll have to keep their names, but that's fine. If the PayloadFilenameToken changes it only affects future LocatedFiles. Weave could detect filename changes and automatically update the token. So Rule P. 

For RdfDataset, we get the slightly complicated situation of the payload being a single (unnamed internally) graph and the rest of the graphs get included from metadata? Or maybe the payload can have a default graph, optional internally-specified named graphs (maybe using external names, maybe using fragment-identifiers if they don't have full URLs) and optional "linked" graphs, referred to and built from statements in the metadata. It's not pretty. But it can accomodate "already built" datasets. And then ResourcePage generation for DatasetPayloads would scan for fragment identifiers and generate html anchors for them.
Thought for 1m 17s

Your “Nomen denotesSliceIRI for named graphs” idea works only if you accept a constraint you didn’t state yet:

If a historical dataset slice contains a named graph with name IRI :g, and :g later changes its denotesDigitalArtifact target (e.g., it always denotes the WorkingSlice), then the meaning of :g drifts while the dataset slice stays immutable. That’s a reproducibility/semantics footgun.

So you have two coherent styles:

Pinned-by-denotation (no selector)

Graph name IRI is a Nomen whose denotesDigitalArtifact points directly to a HistoricalSlice (for immutable) or WorkingSlice (for mutable/latest).

Hard rule: historical dataset slices must only use graph name IRIs that denote HistoricalSlice IRIs.

Stable-name + pinned-content (selector)

Graph name IRI stays stable across dataset versions; a separate selector pins which slice supplies the content.

This is the only way to get “same graph name across versions” and pin historical content without denotation drift.

I updated the canvas to document both styles, with the constraint made explicit.

Rule P accepted: good, but be explicit about consequences

Rule P (old LocatedFiles persist under their slice) is consistent with “LocatedFile IRIs denote bytes.” It also implies:

Storage growth is unbounded unless you add an explicit retention/GC policy (which can be optional/operational, but you should name it as such).

Token changes only affect future LocatedFiles; inventory/metadata maps “current” to the newest ones.

Canvas now states Rule P and the “token change only affects future outputs” point.

Dataset payload complexity: don’t hand-wave “not pretty” — you need a mode + conflict rule

Your “payload has default graph; other graphs included from metadata; but also allow already-built datasets” is exactly where systems become inconsistent unless you define modes.

The clean way is to recognize three DatasetPayload modes:

Native dataset: payload already includes default + named graphs.

Composed dataset: payload provides only default graph (possibly empty); named graphs come from includedGraphs.

Hybrid dataset: native dataset plus additional includedGraphs.

You must also define a conflict rule. Recommendation (most predictable):

If a named graph IRI appears both natively and via includedGraphs, treat as error unless an explicit override flag exists.

Canvas now includes these modes + the conflict rule.

“Datasets must have a default graph” and empty graphs

Yes: default graph may be empty, and your GraphPayload must support empty graphs. That’s now captured as “RdfGraph (GraphPayload) may be empty.”

Range of denotesDigitalArtifact

Agree: do not constrain it to “in-mesh.” It should point to any IRI-addressable digital artifact anywhere.

One caution: if you allow a Nomen to denote opslogs (or any internal mesh component), you’ve implicitly said “opslogs are discourse-worthy artifacts.” That’s fine, but then you need to avoid mixing opslog content into NomenMetadataFlow (high-churn) or you’ll make Nomen histories noisy.

Optional “parent pointers” inside payloads

Good idea, but keep them explicitly optional and non-authoritative, otherwise you create divergence between:

what the filesystem/metadata says

what the payload self-asserts

Model them as convenience hints like sflo:forSlice, sflo:forFlow, sflo:forKnop, with “may be stale” semantics.

Standardized canonical payload filenames vs “island of sanity”

You can’t have all three without symlinks:

no metadata lookup,

human-friendly names,

no duplicate bytes.

With Rule P + no symlinks, the only way to have both (1) and (2) is to write two files per distribution (canonical machine name + descriptive alias) and accept duplication. Otherwise you pick:

either canonical deterministic names (opaque), or

descriptive names that require metadata lookup for “current.”

AbstractFile naming mechanics

You’re right it’s fuzzy if you rely on extensions. Make MIME type authoritative and define a registry/config mapping to pick a default extension when needed (and accept “none” when ambiguous). Canvas already frames it that way.

If you want the next decision that will prevent rework: pick whether you actually need stable-name + pinned-content in includedGraphs. If yes, keep the selector. If no, adopt the hard constraint: historical dataset slices must only reference graph-name Nomina that denote HistoricalSlices.

Mesh Resource Taxonomy Draft
I'm trying to understand "If a historical dataset slice contains a named graph with name IRI :g, and :g later changes its denotesDigitalArtifact target (e.g., it always denotes the WorkingSlice), then the meaning of :g drifts while the dataset slice stays immutable. That’s a reproducibility/semantics footgun."

It's true in any case that denotesDigitalArtifact shouldn't usually change. But for immutable slices, the graph content gets written into the file at the time of weave, so if Nomen "g" changes its denotation, it only affects future Datasets. Maybe I'm not fully understanding your issue, but Nomen which denote graphs should always point to a Slice: Only Slices can represent graphs; Flows may represent graphs-over-time, a subtle difference.

---

As for Opslogging... let's keep it just as a series of regular log files. Logging can be turned on and off, it can be kept in the mesh or outside based on mesh config. But it's not a great fit for "worthy of discourse", doing it as a Flow (even at the mesh level) has a lot of overhead.

BUT that does remind me of our 4th kind of flow: MeshMetadataFlow
Thought for 52s
Why the “denotation drift” warning still matters (even if the bytes are frozen)

You’re right that the file bytes of a historical dataset slice won’t change: the weave writes the quads at time T, and that LocatedFile persists (Rule P). So “reproducibility of the file” is fine.

The footgun is semantic, not byte-level:

In RDF, the named graph is identified by an IRI (your :g).

That IRI is a global symbol. If, elsewhere in the system, :g denotes something that later changes (e.g., it denotes a WorkingSlice whose content evolves), then you now have a single IRI being used to identify different graph contents in different contexts over time.

Two concrete failure modes:

Cross-dataset joins / multi-dataset loading

You load historical dataset slice D@v1 and some later dataset D@v2 (or the current working dataset) into the same store.

Both use named graph :g, but the content differs.

Many systems treat the named graph IRI as “the graph identity,” so you’ve created an accidental collision. It’s not “illegal,” but it’s very easy for users/tools to misinterpret.

Referencing :g outside the dataset

Someone sees :g in the dataset and follows their nose: “:g is that graph.”

If :g now denotes a WorkingSlice (or a newer slice), that external meaning no longer matches what the historical dataset contains under the name :g.

Your proposed mitigation (“Nomina that denote graphs should always point to a Slice”) helps, but it forces a tradeoff you haven’t stated explicitly:

If :g always denotes a specific HistoricalSlice, then :g cannot remain the same name across versions. You’ll mint :g-v1, :g-v2, etc.

If you want :g to be a stable “graph identity” name across versions, then it cannot denote a specific slice; it should denote the Flow (graph-over-time), and you need a separate mechanism to pin which slice supplies content in a given dataset slice.

That’s exactly why “stable-name + pinned-content” exists. Your “pinned-by-denotation” style is fine, but it implies graph names are version-specific.

Also: you said “no need for separate selector” because :g denotes the slice. That works for pinned names, but it eliminates your ability to say “same graph name across time” without changing denotation (which you said you don’t want to do).

You also have a small inconsistency to resolve

You want expressive power for:

“no particular version” vs

“this version” vs

“latest version”

If graph Nomina must always denote a Slice, you can’t express “no particular version.” That’s a Flow-denotation. Either relax the rule (allow Flow targets) or accept losing that expression.

OpsLogs: agreed, don’t model them as flows

Your decision (“just regular log files, configurable, may live inside or outside the mesh”) contradicts the earlier “everything is a Flow,” but that’s fine: ops logs are operational exhaust, not discourse assets.

I updated the canvas to:

remove “opslog series” framing from the taxonomy section,

add OperationalLogFile as a plain LocatedFile mesh component,

add MeshMetadataFlow as an optional mesh-level supporting flow.

Dataset payload composition: your “native / composed / hybrid” framing is the right way to make it not ugly

What you described (“payload may already contain a dataset; may also include linked graphs from metadata; may use fragments internally”) is workable only if you define:

the mode (native/composed/hybrid), and

a conflict rule for graph-name collisions.

The canvas now records that recommended conflict rule: treat collisions as error unless explicitly overridden.

If you want the next tight spec step: decide whether “graph name Nomina are version-specific” is acceptable. If yes, keep your “Nomen denotes HistoricalSlice/WorkingSlice” approach and forbid reuse of the same graph-name IRI across pinned historical versions. If no, you need the stable-name + pinned-content selector (or make the graph-name denote the Flow).

Mesh Resource Taxonomy Draft
I was prepared to say that named graphs could only be HistoricalSlices. But you pointed out that in common RDF usage, named graphs can evolve over time. 

We're talking concretely about named graphs in datasets. For inclusion in a dataset via includesGraphs, we might constrain to only talk about Slices, and maybe only HistoricalSlices. The expressive power of "“no particular version”" is still there, but you can't include “no particular version” of a graph in a datset. 

If in common usage the named graph can evolve over time, there's already the common problem of "load the named graphs from two different time, you get semantic conflict." I'm happy to reduce that problem by pushing people towards using immutable named graphs. Question is: do we do it via SHACL warning, or Error. 
Thought for 13s

You can push people toward immutable named graphs without fighting common RDF practice by making the constraint context-sensitive:

HistoricalSlice dataset (immutable, publishable, cacheable): be strict.

WorkingSlice dataset (mutable, “latest”): allow evolving named graphs.

That maps cleanly to SHACL severity.

Why “but the bytes are frozen” doesn’t eliminate the issue

The file is reproducible, but consumers don’t reason only over bytes—they reason over graph names as IRIs and may:

load multiple datasets into one store,

query by graph IRI across datasets,

treat :g as “the graph,” not “the graph-as-frozen-in-this-file.”

If your system encourages graph-name IRIs to sometimes denote working/latest, you reintroduce the collision/drift patterns. You’re trying to reduce those. So enforce the reduction where it matters most: immutable dataset slices.

Recommendation: Error for HistoricalSlice, Warning (or allow) for WorkingSlice
HistoricalSlice dataset: SHACL Violation (Error)

Make it an error if includedGraphs points at anything that can drift:

graph content source is a WorkingSlice (or a Flow)

graph name IRI denotes a WorkingSlice (in the “pinned-by-denotation” style)

This protects the main promise: “HistoricalSlice is immutable and semantically stable.”

WorkingSlice dataset: Allowed; optionally warn

For working datasets, evolving graphs are the point. You can still optionally warn in cases that look accidental, e.g.:

someone uses a stable graph-name IRI but doesn’t pin content (if you support the selector style)

someone uses “pinned-by-denotation” but points the graph-name at WorkingSlice while also claiming the dataset is “publishable” (if you add such a flag)

If you don’t want flags, keep it simple: only key strictness off whether the dataset slice is HistoricalSlice vs WorkingSlice.

If you want one global rule, pick the strict one

If you force “includedGraphs may only reference HistoricalSlices” universally, you will block legitimate “latest dataset” use cases (unless you require weaving for every publication), which you explicitly do not want. So a global strict rule is misaligned with your “publish without weave” workflow.

Canvas updated

I added a “Validation policy (recommended)” section under includedGraphs stating:

HistoricalSlice datasets: WorkingSlice sources are SHACL errors

WorkingSlice datasets: allow working sources; warn if not pinned (optional)

optionally profile-driven, but historical immutability strict by default

Mesh Resource Taxonomy Draft
I think you got a bit confused in that last response. Try again.
Thought for 39s

The problem I was pointing at is not “your historical dataset file will change.” You’re right: the quads are frozen at weave time.

The problem is: the named-graph IRI is still a symbol that may be used outside that one frozen file, and your model gives it independent semantics via denotesDigitalArtifact.

Concrete example of the footgun

Historical dataset slice D@v1 contains a named graph with name IRI :g.

In your “pinned-by-denotation” style, :g is a Nomen, and :g denotesDigitalArtifact S1 where S1 is a Slice IRI.

Now two ways this goes wrong:

If :g ever changes its denotation (even “rarely”):

D@v1 still contains the old quads (fine).

But dereferencing/using :g as an IRI in discourse now points at S2, not S1.

Anyone treating :g as “the graph that appears in D@v1” is now misled.

If you reuse :g across time (which people will want for “stable graph name”):

You cannot keep :g stable and also keep :g → Slice without either:

changing its denotation (drift), or

minting a different :g-v2 per version.

If you don’t mint per-version names, you force denotation drift.

So: “no selector” is fine, but it implies graph-name IRIs are version-specific (each graph name denotes exactly one HistoricalSlice forever). That’s the reduction you said you want.

Your proposed constraint is coherent

For includedGraphs in a dataset:

Constrain the content source to be a Slice (not a Flow).
That forbids “no particular version” inside a dataset, which is good.

Then make strictness depend on the dataset slice type:

Recommended SHACL policy

For HistoricalSlice dataset:

includedGraphs.contentSource MUST be a HistoricalSlice → Violation (Error) if not.

If you use “pinned-by-denotation,” and the graph-name IRI is a Nomen, it MUST denote a HistoricalSlice → Violation (Error) if it denotes WorkingSlice.

For WorkingSlice dataset:

allow WorkingSlice sources (that’s the point of “latest/mutable”)

optionally emit a Warning if someone tries to publish/mark it as stable (if you have such a notion), but otherwise don’t block.

This does exactly what you asked: it pushes people toward immutable named graphs in the only place where immutability is actually a contract—HistoricalSlice datasets—while still allowing the common “named graphs evolve” usage in WorkingSlice datasets.

Bottom line decision: Warning vs Error

If your goal is to reduce the “load two times, conflict” problem, then for HistoricalSlice datasets it should be an Error, not a Warning. A warning still produces an immutable slice whose named-graph IRIs refer to moving targets or ambiguous identities, which defeats the point of “HistoricalSlice” being a high-integrity artifact.

I updated the canvas’s “Validation policy (recommended)” accordingly; it’s now explicitly “Error for HistoricalSlice datasets when WorkingSlice is included.”

Mesh Resource Taxonomy Draft
For any Nomen, if you change the denotation, it's going to break (or at least confuse) any extant references, so I don't think we need to worry about that specifically for named graphs. 

There's always the Knop IRIs that you can use if you're concerned about shifting Nomen denotation. 

If you want to refer to a named graph in a dataset with a Flow IRI to denote "graph over time", you can do it via a Native Dataset. But if you want to include via includesGraph you have to specify a slice, which can still be a WorkingSlice to accomplish something that changes over time. I think that is expressive enough, that is close enough to "stable graph name over time"

I do not like the idea of adding a selector to an IRI. But I haven't thought it through much.

I also don't like "HistoricalSlice dataset" vs "WorkingSlice dataset" -- We just have Datasets, and they may include graphs that are HistoricalSlice or WorkingSlice.


Thought for 1m 31s

You’re right to push back on “denotation drift” being a special problem for named graphs. It isn’t. Changing a Nomen’s denotation is globally confusing regardless of whether the Nomen is used as a graph name, a person name, or anything else. So the real question is narrower:

What constraints do you want on includedGraphs so that “dataset slices” have predictable semantics?

How hard do you enforce those constraints?

1) Your “Nomen for a graph denotes a Slice” approach is internally consistent

If you adopt:

:g (a Nomen used as graph name) denotesDigitalArtifact some Slice IRI

then you’ve implicitly decided:

Graph-name IRIs used via includedGraphs are versioned names (because they denote a particular slice).

“Stable graph name over time” becomes “stable pattern of names” (e.g., g/v17/, g/v18/), not literally the same :g reused forever.

That is compatible with your stated preference: “close enough to stable graph name over time.”

It also fits your dislike of selectors: the “selector” is effectively encoded in the denotation target.

2) You don’t need “HistoricalSlice dataset vs WorkingSlice dataset”

You only need “historical dataset slice vs working dataset slice,” because Slices are what have mutability semantics.

So constraints should be phrased as:

For dataset Flow WorkingSlice, you may include WorkingSlice graphs (mutable composition).

For dataset Flow HistoricalSlices, you should normally include only HistoricalSlice graphs (fully pinned composition).

That’s what I updated in the canvas: the policy is slice-scoped, not “dataset-typed.”

3) Warning vs Error for “HistoricalSlice includes WorkingSlice” in includedGraphs

Given your model, I recommend:

Default (strict): Error when a dataset HistoricalSlice includes a WorkingSlice graph source.

Rationale: if you want “mutable/latest composition,” you already have the dataset WorkingSlice. If you’re weaving a HistoricalSlice, you’re explicitly asking for a pinned artifact.

Permissive profile: Warning, but only if weave pins and records what was actually used (resolved HistoricalSlice IRI at weave time). Otherwise you’ve created an ambiguous historical artifact: the bytes are frozen, but the spec doesn’t tell you what version the inclusion meant.

This doesn’t require selectors; it’s purely about what IRIs you’re allowed to reference from a historical dataset slice.

4) Dataset payload composition options: accept “native + linked,” but define conflict handling

Your “native dataset may already contain named graphs, plus includedGraphs may add more” is fine if you enforce one rule:

If a named graph IRI appears both natively and via includedGraphs, treat as error unless an explicit override exists.

That prevents silent double-definition.

5) “Graph names as fragments” inside native datasets

If you allow fragment-only names (e.g., #g1) in a native dataset, you need a deterministic base for resolution, or transposition becomes a mess.

The simplest rule in Semantic Flow terms:

Relative/fragment graph names in RDF payloads resolve against the AbstractFile IRI (your implied base), not the filesystem path.

Then ResourcePage generation can safely render anchors for fragment graph names.

6) MeshMetadataFlow

Agree: this is your “fourth flow,” but it’s not a fourth flow type inside every Knop/Nomen. It’s a mesh-level supporting flow (optional) for operational facts/config that aren’t meaningfully attached to a single Nomen or Knop. Canvas now reflects this framing.

If you want the next decision to lock: choose whether strict mode makes “Historical dataset slice includes WorkingSlice graphs” an Error (my recommendation), and whether permissive mode requires pin-recording at weave time.

Mesh Resource Taxonomy Draft
we might want subproperties of denotesDigitalArtifact like "denotesDigitalArtifactFlow/denotesDigitalArtifactOverTime"; "denotesDigitalArtifactVersion"; "denotesDigitalArtifactCurrentVersion"; and "denotesAbstractFile" and "denotesLocatedFile" -- that way when the Nomen point to non-SemanticMesh IRIs, we have a sense of the level of abstraction, and can also check consistency internally and warn people if they try to change the denotation subject to something inconsistent.

----


If I understand correctly, this is still way off:

"For dataset Flow WorkingSlice, you may include WorkingSlice graphs (mutable composition).

For dataset Flow HistoricalSlices, you should normally include only HistoricalSlice graphs (fully pinned composition)."

HistoricalSlices are just WorkingSlices frozen in time. We're not going to rewrite the references on weave. 
Thought for 56s
1) Subproperties of denotesDigitalArtifact: yes, but treat them as expectations, not different denotation mechanics

Your idea is sound for two reasons:

External targets usually won’t have SF typing, so you need a way to say “I’m denoting a version vs a thing over time” without relying on rdf:type.

Internally, you can validate consistency and warn if someone “changes abstraction level” (e.g., a Nomen that used to denote a Slice now denotes a Flow).

The key design constraint: these subproperties should not change what denotation is; they should only add an intended abstraction level.

So denotesDigitalArtifactVersion, denotesDigitalArtifactOverTime, denotesAbstractFile, denotesLocatedFile, etc. are good as subproperties of denotesDigitalArtifact, with validation behavior:

If target is SF resource: mismatch can be Error/Warning (you choose).

If target is external: mismatch is usually “unverifiable” → Warning at most.

I updated the canvas with a “Denotation granularity” subsection and this exact framing.

2) The “historical dataset slice should include only historical graph sources” statement was about constraints, not rewriting references

You’re right that “HistoricalSlices are just WorkingSlices frozen in time” and that you won’t rewrite IRIs inside the data file at weave.

The reason the constraint still exists is this specific semantic situation:

You materialize a dataset HistoricalSlice at time T.

One included graph source is a WorkingSlice (mutable).

The resulting dataset file now contains a frozen snapshot of that graph as of time T.

But the graph’s name IRI (or a Nomen denoting that WorkingSlice) still denotes the current mutable thing afterward.

So you get: frozen bytes + “current” denotation symbol. That’s not a byte reproducibility issue; it’s a “meaning by dereference / user expectation” issue.

You said: “Denotation shouldn’t usually change; Knop IRIs exist if you’re worried.” That doesn’t remove this mismatch; it just gives an escape hatch for careful users. If your goal is to push people toward immutable named graphs, you should enforce it at the point where you mint immutable artifacts: when materializing a dataset HistoricalSlice.

So the corrected policy is:

When materializing a dataset HistoricalSlice, includedGraphs should resolve to HistoricalSlice sources, by default.

If it resolves to a WorkingSlice source, you either:

Error (strict profile), or

Allow (permissive profile) but require you to record a pinning map somewhere (metadata/inventory) that states which HistoricalSlice was actually used at weave time for each included graph.

This does not require rewriting graph names or adding selector IRIs. It’s purely a validation + provenance requirement.

I updated the canvas “Validation policy” section to reflect exactly this, and removed the confusing “HistoricalSlice dataset vs WorkingSlice dataset” framing—everything is slice-scoped.

3) On “no selector IRIs”

If you dislike selectors, your two non-selector options are:

Version-specific graph-name Nomina (graph name Nomen denotes a HistoricalSlice). Cleanest, most consistent.

Allow WorkingSlice sources, but then you must accept that historical dataset slices can embed “current-named” graphs; if you allow it, you should require a pinning map to avoid silent ambiguity.

Selectors are only needed if you insist on stable graph name across versions while also pinning historical content and keeping denotation stable. You’re explicitly de-prioritizing that, so you can omit selectors.

Mesh Resource Taxonomy Draft
The problem with "Allow (permissive profile) but require you to record a pinning map somewhere (metadata/inventory) that states which HistoricalSlice was actually used at weave time for each included graph." is that you don't necessarily know the HistoricalSlice at weave time. Maybe you haven't woven that thing yet. Maybe it's in another mesh. 

Which reminds me, it'll be hard to do includesGraphs for the latest HistoricalSlice of those graphs in the same mesh because they won't have been woven yet. :/ 

I think we just warn if the includeGraphs resolves to a WorkingSlice, it's no worse that all the other "common RDF usage" where named graphs change content over time.

Or we could just drop includesGraphs and make all datasets Native datasets. 
Thought for 56s

You’re right: “require a pinning map to an exact HistoricalSlice at weave time” is not generally feasible, because:

the included graph source may not have been woven yet (same mesh, dependency ordering),

the included graph source may be in another mesh you don’t control,

the included graph source may be external and never produces “HistoricalSlices.”

So that requirement was too strong.

What you can require (and what you can’t)
You can always freeze bytes

When you materialize a dataset slice, you necessarily materialize concrete quads for each included graph. That’s reproducible.

You cannot always know “the corresponding HistoricalSlice IRI”

Unless you control and pre-weave the source artifact, there may be no historical slice to point to.

So the only universally implementable “pinning” is content-based, not “slice-IRI-based.”

Practical validation rule for includedGraphs

Given your goals (“don’t add selectors; accept common RDF conflict; nudge toward immutability”):

If an includedGraphs entry points to a WorkingSlice, emit a SHACL Warning, not an Error.

Optionally, have a strict profile that upgrades it to Error, but don’t make strict the default if cross-mesh/external composition is a core use case.

This matches your statement: it’s no worse than typical named-graph evolution semantics.

If you still want “pinned semantics” without needing HistoricalSlice IRIs

Require that a dataset HistoricalSlice records materialization provenance per included graph:

source slice IRI actually used (even if it’s a WorkingSlice IRI),

materialization timestamp,

content digest/checksum of the materialized graph.

Then, later, tooling can optionally relate that digest to a HistoricalSlice if one appears (same bytes), but it doesn’t have to.

That gives you:

reproducible bytes now,

a path to “semantic pinning” later,

no selector IRIs,

no dependency on other meshes being woven.

I updated the canvas policy section accordingly.

“Latest historical slice” inclusion problem

You’ve identified a real sharp edge:

If you want to include “latest historical” of dependencies within the same weave, they might not exist yet.

You have three choices:

Weave dependency-first (topological order)
When weaving a dataset historical slice, weave all referenced graph sources first (if they’re in the same mesh and eligible). Then “latest historical” exists.

Fallback to working slice + warn
If dependency has no historical slice yet (or is external), include working slice, warn, and record digest.

Drop includedGraphs and force native datasets
Simplifies semantics but loses non-duplication and composition; you’ll end up repeating quads across artifacts.

Given your stated motivation (“I love being able to support named graphs” and avoid repetition), option (2) is the least painful baseline, with optional (1) when feasible.

Selector dislike

You don’t need selectors if you accept:

includedGraphs always points directly to a slice IRI (working or historical),

“latest historical” is not an addressable thing unless you reintroduce pointer IRIs.

Right now you’re choosing: no pointer IRIs → no “latest historical” reference → either dependency-first weave or fallback-to-working.

That’s a coherent trade, but it’s a trade.

Mesh Resource Taxonomy Draft
Regarding AbstractFile naming, the AbstractFile is a AbstractResource, so represented with a folder name. Its LocatedFiles will have an extension, it should have an extension, and the foldername should default to that extension. I feel like usually you will upload a file (.jpg, .txt., etc) to establish a PayloadFlow. Or if you want to start with a blank (but textual) one, you have to choose the file extension when the Knops created. Mime-type is optional. (Files may not have mime-types). So the AbstractFile's folder token is the filename of its (Canonical) file.

I'm intrigued by your idea about Composed/Hybrid dataset including some metadata. Probably we need to make it optional, but it could just be as simple as an inventory Flow pointer, as presumably inventory would include a materialization timestamp. 

BUT I'm tempted to just drop includedGraphs. Semantic Web needs better inclusion properties, but maybe we can leave that to application-specific ontologies. If you want to put a bunch of graphs into a single dataset, you can (we need to support that existing practice), just do a Native dataset
Thought for 1m 12s
AbstractFile naming: your proposal has an internal contradiction you should resolve explicitly

You stated both:

“The foldername should default to that extension.”

“The AbstractFile’s folder token is the filename of its (Canonical) file.”

Those are only compatible if your “canonical filename” is literally just the extension (e.g., ttl). That breaks as soon as you have more than one representation/variant per slice.

You will have collisions in at least these cases:

multiple syntaxes for the same slice (ttl, jsonld, trig, nq)

compression (ttl.gz vs ttl)

canonicalization variants (sorted.ttl, c14n.ttl)

bundling (zip, tar.gz)

So you need a stable uniqueness rule. The least painful one is:

AbstractFile folder token defaults to the canonical LocatedFile filename (including extension)
Example tokens: payload.ttl, payload.jsonld, payload.nq.gz.

That still respects your “extension-first” intuition (the extension is visible), without collapsing into “extension-only” collisions.

I updated the canvas’ AbstractFile naming section to reflect this.

MIME type “optional”: fine, but you’re implicitly making extensions normative

If “files may not have MIME types,” then:

content-type at serve time must be inferred from extension or server config, and

extension becomes your primary machine-readable hint.

That’s acceptable, but then don’t keep language like “MIME type is authoritative.” It isn’t, if it can be absent. The canvas now treats MIME type as optional and extension/canonical filename as the default discoverability anchor.

includedGraphs: dropping it is coherent, but you should be honest about what you lose

If you drop includedGraphs entirely and require Native datasets:

You gain:

simpler core semantics (no composition rules, no conflict rules, no “materialization provenance” concerns)

less platform commitment to dataset assembly

You lose:

a platform-level way to compose a dataset from separately-addressed graph artifacts without duplicating quads

a standardized mechanism to say “this dataset’s named graph X was sourced from artifact Y”

And you’ll still have to support “existing practice” inside native datasets (named graphs embedded in the payload). That means you still need to decide how you handle:

graph names that are fragments/relative IRIs (base rules)

the resource-page UX for browsing graphs inside a dataset

If your intent is “composition is an app-level concern,” that’s fine—just move includedGraphs out of the core profile and treat it as an optional convention/extension. I updated the canvas headings accordingly: includedGraphs is now labeled optional, and the DatasetPayload modes section is explicitly “only relevant if includedGraphs is supported.”

The inventory / materialization timestamp idea still works without includedGraphs

Even with only native datasets, inventory can still record:

materialization timestamp (weave time for historical slices),

digests/checksums for LocatedFiles,

provenance at the slice level.

So you don’t lose that capability by dropping includesGraphs; you only lose cross-artifact composition provenance.

MeshMetadataFlow

Yes: if you’re dropping opslogs-as-flow and maybe dropping includesGraphs, MeshMetadataFlow becomes the obvious “fourth kind of flow” at mesh level for operational facts/config. That fits your direction.

If you want to settle this cleanly: decide whether dataset composition is a core SF feature or an extension. If it’s an extension, your current text should stop implying it’s a fundamental mechanism and instead present it as “supported by some profiles/tooling.”

Mesh Resource Taxonomy Draft
Let's address inventory. Knop inventory only gets added to, so it's reasonable for it to just be a mutable RdfDataset. Making it a flow seems overblown. Do we need Mesh inventory? Seems like it would aid explorability. On the Nomen side, you could capture the Nomen hierarchy by just specifying child Nomina. But if the mesh had a complete inventory, it would be redundant.
Thought for 45s

If Knop inventory “only gets added to,” making it a Flow is usually overkill. But your “only gets added to” premise is fragile in RDF unless you pick an explicit correction model.

The core inconsistency

An “add-only RDF dataset” will inevitably accumulate contradictions unless you have one of:

Snapshot semantics: overwrite the inventory dataset to reflect the current index.

Event/validity semantics: keep old statements but qualify them with assertion time / validity / supersedes / tombstones so consumers can tell what’s current.

Without that, a single corrected checksum or renamed canonical filename creates two incompatible triples with no rule for which wins.

So: if you want “simple,” treat inventory as a mutable snapshot index.

Knop inventory as mutable dataset

This is consistent with your architecture if you frame inventory as:

operational / derived / non-authoritative, and

regenerable from the filesystem + slice contents.

Then it does not need versioning; it’s an index, not a discourse artifact.

If you later need auditability, you can introduce an optional append-only log (dataset series / flow) specifically for inventory events, but that’s not the default.

Do you need mesh inventory?

Not required, but useful for static-serving:

Without a server, clients otherwise have to crawl directories and parse lots of small metadata files.

A mesh-level inventory file (under _mesh/) gives a single entry point for exploration and tooling.

But it must be explicitly non-authoritative, because you already declared the filesystem is the source of truth.

So the clean stance is:

MeshInventoryDataset: optional, derived, regenerable
(think: “index.html for machines”)

Redundancy with “Nomen child Nomina”

Yes—mesh inventory makes explicit child links redundant if you only want structural containment.

There are still two reasons to keep optional child links on the Nomen side:

editorial grouping that isn’t purely filesystem (or is anticipatory),

human-friendly navigation hints without requiring a global index.

But structurally, filesystem scanning + optional mesh inventory is sufficient.

Canvas updated

I updated the taxonomy draft to:

rename KnopInventoryFlow → KnopInventoryDataset (optional operational index),

add MeshInventoryDataset as optional,

add the “add-only RDF contradictions” warning and the snapshot-vs-event semantics fork.

Mesh Resource Taxonomy Draft
So in this new model, MeshInventoryDataset and KnopInventoryDataset are the only non-flow RdfDatasets? 

Also, any complaints if I use DigitalArtifact to apply to all parts of a PayloadFlow including LocatedFile?
Thought for 32s

Yes—if you choose this model deliberately—it is coherent to have only these two non-flow RDF datasets:

KnopInventoryDataset

MeshInventoryDataset

…but only if you commit to the consequence that they are:

operational

derived / regenerable

non-authoritative

rewritable snapshot indices

If you later introduce any other RDF dataset that you expect people to cite, diff, or rely on historically (e.g., curated backlinks, provenance assertions you don’t want to lose), it should become a Flow. Otherwise you’ll end up quietly treating a mutable snapshot as a first-class artifact and the semantics will rot.

I added an explicit “Non-flow RDF datasets” note to the canvas to lock this down.

Using DigitalArtifact to cover everything down to LocatedFile

You can do it, but there’s a sharp modeling trap:

If a LocatedFile is a DigitalArtifact, then by your own definition it can have “one or more concrete representations,” which implies representations-of-representations (nonsense) unless you special-case it.

So you need one of these two clean options:

Preferred: LocatedFile is not a DigitalArtifact; it is a ConcreteRepresentation of a DigitalArtifact.

DigitalArtifact = thing that can have representations

LocatedFile = representation (terminal bytes)

Allowed but stricter: LocatedFile is a DigitalArtifact, but you enforce:

LocatedFile is terminal and MUST have zero further representations.

If you don’t enforce (2), you create infinite regress in your ontology and SHACL becomes messy.

Your canvas draft also had a small inconsistency (LocatedFile (LocatedFile; DigitalArtifact representation)), which I fixed by introducing ConcreteRepresentation and making LocatedFile a MeshResource; ConcreteRepresentation.

Mesh Resource Taxonomy Draft
What I'm trying to get at with DigitalArtifact is: an artifact-over-time (flow) is an abstract DigitalArtifact with temporal extension; a Slice is an abstract DigitalArtifact; an AbstractFile is a fairly-concrete but unlocated DigitalArtifact; and a ConcreteFile is a completely concrete, located DigitalArtifact. And then an RdfDataset is a subClass of DigitalArtifact. So we wouldn't need a RdfDatasetFlow, we could just say type Flow, DigitalArtifact.
Thought for 29s

What you’re proposing is coherent, but only if you make one thing explicit: “DigitalArtifact” is a supertype across multiple orthogonal axes (temporal versioning and representational realization). If you don’t model those axes explicitly, you’ll end up with ambiguous properties like “hasRepresentation” doing three different jobs.

Make DigitalArtifact span all levels (works)

You can treat all of these as DigitalArtifact:

Flow = a versioned DigitalArtifact (artifact-over-time)

Slice = a version of that DigitalArtifact

AbstractFile = an unlocated representation spec of a Slice (format/bundling/checksum/etc.)

LocatedFile = a located concrete byte realization of an AbstractFile

That’s fine, and it matches your mental model.

But you must separate the relations (critical)

Don’t let one generic “hasPart/hasRepresentation” try to cover everything. You want at least three distinct predicates:

hasVersion (Flow → Slice)

hasAbstractFile or hasRepresentationSpec (Slice → AbstractFile)

hasConcreteRepresentation or hasLocatedFile (AbstractFile → LocatedFile)

Otherwise, “representation” becomes semantically overloaded and you’ll regret it in SHACL and client code.

“No RdfDatasetFlow”: yes, but you still need an artifact-kind signal

You can avoid RdfDatasetFlow as a primary class by typing the Flow as:

rdf:type sflo:Flow, sflo:DigitalArtifact

plus a property that says what kind of artifact it is (for validation / tooling), e.g.:

sflo:artifactKind sflo:RdfDataset (or sflo:payloadKind, sflo:artifactClass, etc.)

Then your SHACL can say:

If Flow artifactKind RdfDataset then:

slices’ abstractfiles must be RDF serializations,

named-graph constraints apply, etc.

This is strictly better than proliferating XFlow classes unless you need them for query ergonomics.

Optional: keep derived convenience classes (best of both)

You can still define:

RdfDatasetFlow ⊑ Flow

with a rule/restriction: Flow artifactKind value RdfDataset

So you don’t author RdfDatasetFlow, but tools can infer it, and people can query it.

Concrete files as DigitalArtifacts

No objection if you enforce the terminal rule you already implied:

LocatedFile is a DigitalArtifact, but cannot itself have further representations.

Otherwise you get “representations of located files,” which is an infinite regress.

If you want, I can update the canvas taxonomy to add the three predicates (hasVersion, hasAbstractFile, hasLocatedFile) and an artifactKind property, and reframe RdfDatasetFlow as optional/derived.

Mesh Resource Taxonomy Draft
for "no RdfDatasetFlow" why not just rdf:type sflo:Flow, sflo:RdfDataset ? 

for the relations properties, they would be hasSlice, hasAbstractFile, and hasLocatedFile.

In my last ontology, I had FileBearingContainer:

{
      "@id": "FileBearingContainer/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SfloContainer/"
      ],
      "rdfs:label": "File-bearing container",
      "rdfs:comment": "A SfloContainer that should contain one or more AbstractFiles, i.e. it represents an abstract digital artifact."
    }

That would cover RdfDataset and Slices. Keep it?

For "with a rule/restriction: Flow artifactKind value RdfDataset", I think you meant RdfDatasetFlow artifactKind. But if RdfDatasetFlow exists, why not author it? It's simpler/clearer than Flow, RdfDataset.

I think we have to drop "“one or more concrete representations" because a Slice isn't a concrete representation. A better sentence might be "A digital artifact can exist through revisions, those revisions may have different specific representations, and those specific representations may each have multiple locations. At any of those levels, SemanticFlow considers it a a DigitalArtifact." But maybe it would be better to be explicit. Or maybe it's unnecessary altogether, since PayloadFlows are always DigitalArtifacts-over-revision, Slices are always a DigitalArtifact revision. 

Thought for 48s
1) Why rdf:type sflo:Flow, sflo:RdfDataset is a category error (in your model)

In your current taxonomy:

Flow is the artifact-over-revision container.

A RdfDataset is the dataset itself (a default graph + named graphs) at some semantic level.

If you type the same resource as both Flow and RdfDataset, you’ve implicitly asserted that:

the Flow is an RDF dataset (i.e., it is itself a set of quads), not merely something that has dataset versions.

That breaks multiple things:

What does hasSlice mean if the subject is already a dataset? Is the dataset “made of” datasets?

What does dereferencing the Flow IRI return? If it’s a dataset, clients may expect RDF data of that dataset, not a resource-page / container describing versions.

SHACL gets muddled: constraints for “dataset structure” and “version container structure” apply to the same node and will conflict (or force a mess of conditional shapes).

You can do this only if you redefine RdfDataset to mean “dataset-like artifact at any abstraction level” (including “dataset-over-time”). But then RdfDataset stops meaning “a dataset snapshot” and becomes “dataset-family,” which is exactly what you were trying to avoid.

Conclusion: Don’t use rdf:type RdfDataset on the Flow unless you deliberately redefine what RdfDataset means.

2) If you want to avoid RdfDatasetFlow, the clean alternative is not rdf:type, it’s a kind/class expectation

Two viable patterns:

Pattern A (recommended for your SHACL-first style): author RdfDatasetFlow and RdfDatasetSlice

This is the simplest and clearest:

RdfDatasetFlow ⊑ Flow

RdfDatasetSlice ⊑ Slice ⊓ RdfDataset

SHACL: RdfDatasetFlow must have slices of type RdfDatasetSlice, etc.

This avoids the category error and keeps your “primary constituents” typed for validation.

Pattern B (if you truly hate class proliferation): use an explicit kind property

Example:

:x rdf:type Flow

:x sflo:artifactKind sflo:RdfDataset

Then SHACL branches on artifactKind. This is workable, but it’s more complex than authoring RdfDatasetFlow once you have many kinds (Graph, Dataset, Image, ZipBundle, etc.). You were correct to be skeptical of this earlier.

3) Your relations (hasSlice, hasAbstractFile, hasLocatedFile) are right, but you must lock domain/range

Minimum crisp rules:

hasSlice: domain Flow, range Slice

hasAbstractFile: domain Slice, range AbstractFile

hasLocatedFile: domain AbstractFile, range LocatedFile

If you allow non-flow artifacts (like inventory datasets) to have AbstractFiles directly, you’ll be tempted to let hasAbstractFile apply to things other than Slice. Don’t do that unless you also introduce a “StaticArtifact” abstraction. Otherwise “Slice” stops being the universal “version” node.

4) FileBearingContainer: keep the concept, but your old placement is now off

Your old comment says:

“contains one or more AbstractFiles … represents an abstract digital artifact.”

In the new model, the thing that “should contain one or more AbstractFiles” is basically Slice (a revision) — not RdfDataset in general, and not Flow.

So if you keep it:

Rename to something like AbstractFileBearer or RepresentableRevision

Make it apply to Slice (and maybe other revision-like things), not to Flow.

If you keep FileBearingContainer and claim it covers “RdfDataset and Slices,” you’re back to conflating levels unless RdfDataset is explicitly the slice-level dataset class (RdfDatasetSlice). That’s another argument for authoring RdfDatasetSlice.

5) Fixing the definition of “DigitalArtifact” (you’re correct to drop “one or more concrete representations”)

Yes: “Slice isn’t a representation.” That sentence was encoding only the realization axis and ignoring the revision axis.

A definition that matches what you’re trying to do:

DigitalArtifact is any addressable information artifact in Semantic Flow. A DigitalArtifact may have revisions (Flow → Slice), and a revision may have representation specifications (Slice → AbstractFile) and locations (AbstractFile → LocatedFile). Semantic Flow treats the Flow, Slice, AbstractFile, and LocatedFile as DigitalArtifacts at different abstraction levels.

If you keep LocatedFile as a DigitalArtifact, enforce the terminal constraint you already accept:

LocatedFile MUST NOT have hasAbstractFile / hasLocatedFile / further realizations.

6) Your “if RdfDatasetFlow exists, why not author it?” instinct is correct

Given your stated needs:

“primary constituents”

“need them for SHACL checking”

“clearer than Flow + kind hacks”

Then author RdfDatasetFlow (and RdfDatasetSlice) and stop trying to make Flow also be a dataset via rdf:type. That will keep the ontology readable and keep validation sane.

If you want, I can update the canvas taxonomy to:

remove the implication that RdfDataset applies to Flow,

add RdfDatasetFlow and RdfDatasetSlice,

re-home/rename FileBearingContainer so it cleanly targets Slice-level representability.

Mesh Resource Taxonomy Draft
Well, I was going to assert that DigitalArtifact was a Flow, Slice, AbstractFile, or LocatedFile. And an RdfDataset was a subclass of DigitalArtifact, so also a F, S, AF, or LF. 

I don't hate class proliferation, I prefer Pattern A. 

Hopefully we don't need FileBearingContainer... RdfDatasets are either standalone/not-contained-in-flows/singular, like the InventoryDatasets; or Slices (either HistoricalSlice or WorkingSlice). 

Do we need DigitalArtifact as an ontology term? Might be unnecessary.
Thought for 56s

Your current plan (“DigitalArtifact = Flow ∪ Slice ∪ AbstractFile ∪ LocatedFile; RdfDataset ⊑ DigitalArtifact so it can be any of those”) collapses two axes that you actually need to keep separate:

Revision axis: Flow → Slice

Representation axis: Slice → AbstractFile → LocatedFile

If you don’t separate them, you end up asserting nonsense like “a file is an RDF dataset” (as opposed to serializing one), or “a Flow is a dataset snapshot.”

The biggest inconsistency you’ve introduced

You also want InventoryDatasets to be non-flow datasets.

But if DigitalArtifact is only (Flow | Slice | AbstractFile | LocatedFile), then a non-flow inventory dataset isn’t a DigitalArtifact at all.

So you can’t simultaneously have:

“inventory is a non-flow dataset” and

“digital artifacts are only F/S/AF/LF.”

One of those must change.

Clean fix with Pattern A (class proliferation is fine)
Step 1 — Keep DigitalArtifact, but define it as “artifact in the revision/representation pipeline”

Make DigitalArtifact the umbrella, but include a fifth case:

StaticArtifact (DigitalArtifact; AbstractResource) — a non-flow artifact (single mutable snapshot) that can still have AbstractFiles/LocatedFiles

Then:

Flow ⊑ DigitalArtifact

Slice ⊑ DigitalArtifact

AbstractFile ⊑ DigitalArtifact

LocatedFile ⊑ DigitalArtifact

StaticArtifact ⊑ DigitalArtifact ✅ covers InventoryDatasets

Step 2 — Do not make RdfDataset span AF/LF

If you type an AbstractFile or LocatedFile as RdfDataset, you’ve equated representation with content. That breaks basic questions like “how many named graphs are in this dataset?” (for a file, the answer depends on parsing bytes).

So split:

RdfDatasetArtifact (DigitalArtifact) — the dataset as an abstract RDF dataset at some lifecycle level
Subclasses you will actually want for SHACL:

RdfDatasetFlow ⊑ Flow ⊓ RdfDatasetArtifact

RdfDatasetSlice ⊑ Slice ⊓ RdfDatasetArtifact

RdfDatasetStatic ⊑ StaticArtifact ⊓ RdfDatasetArtifact (inventory snapshots)

RdfDatasetSerialization (AbstractFile/LocatedFile) — representations that serialize an RDF dataset
e.g.

RdfDatasetAbstractFile ⊑ AbstractFile ⊓ RdfDatasetSerialization

RdfDatasetLocatedFile ⊑ LocatedFile ⊓ RdfDatasetSerialization

This preserves your intended “RdfDataset is a subclass of DigitalArtifact” idea, but only at the content-bearing levels (Flow/Slice/Static), and keeps AF/LF in the serialization layer.

Step 3 — Your relations become well-typed without hacks

hasSlice: Flow → Slice

hasAbstractFile: Slice → AbstractFile

hasLocatedFile: AbstractFile → LocatedFile

Inventory datasets (Static) can either:

have their own hasAbstractFile directly (if you allow StaticArtifact → AbstractFile), or

be modeled as a degenerate Flow with only WorkingSlice (but you’ve said you don’t want that).

If you allow StaticArtifact → AbstractFile, don’t reuse hasAbstractFile (domain Slice) or you’ll weaken the model. Add a sibling property:

hasAbstractFile (Slice → AbstractFile)

hasStaticAbstractFile (StaticArtifact → AbstractFile)

or generalize:

hasAbstractFile domain = (Slice ⊔ StaticArtifact)

Pick one and lock it, or SHACL will get messy.

Do you “need” DigitalArtifact?

If you keep any non-flow artifacts (inventory datasets, ops logs if you ever RDF-ify them, etc.), then yes: DigitalArtifact is useful as the shared supertype across Flow/Slice/Static/AbstractFile/LocatedFile.

If you drop DigitalArtifact, you’ll end up repeatedly writing unions in SHACL and property ranges, and you’ll lose a clean range for denotesDigitalArtifact expectations.

So: you can drop it only if you accept either:

“everything is a Flow” (you’ve already rejected), or

lots of duplicated union typing and weaker constraints.

If you want, I can update the canvas taxonomy to:

add StaticArtifact,

replace RdfDataset with RdfDatasetArtifact + RdfDatasetSerialization,

and tighten the domain/range story for hasAbstractFile vs static inventory.

Mesh Resource Taxonomy Draft
If we keep DigitalArtifact, it should apply to non-flow artificats. But ultimately even ResourcePages are DigitalArtifact, ie. DigitalArtifact is the superclass of LocatedFile and AbstractResource. But it doesn't quite fit. MeshResource captures the same thing. 

I guess I'm searching for a name for a concept that excludes ResourcePages. 

Probably we get rid of ConcreteRepresentation.
Thought for 41s

You’re noticing a real modeling mismatch: in a static site, resource pages are files, but conceptually they’re renditions of abstract resources, not “payload artifacts” in the same sense as your flows/slices/data files.

1) “DigitalArtifact” vs “MeshResource”

MeshResource (as you’ve defined it) is fundamentally “anything in the mesh address space with a filesystem mapping.” That includes resource-page files, indexes, assets, logs, everything.

If you broaden DigitalArtifact to “everything including resource pages,” it becomes nearly synonymous with MeshResource, which makes it less useful for constraints.

So yes: you likely want a concept that excludes resource-page renditions.

2) The concept you actually want: “Rendition” vs “Artifact”

The clean split is:

RenditionFile: files whose primary job is rendering/navigation (resource pages like index.html, generated indexes, etc.)

Artifact files: files that are the concrete realizations of DigitalArtifacts (payload and supporting artifacts), i.e., they sit under the Flow/Slice/AbstractFile chain.

Then the “concept excluding ResourcePages” is simply:

NonRenditionArtifact (bad name), or better:

MeshArtifact / ArtifactResource / WeavableArtifact

I’d recommend MeshArtifact if you can tolerate the ambiguity that it still lives “in the mesh.” It reads as: “the artifacts the mesh is about, not the pages it uses to present them.”

Mechanically, you can define it as:

MeshArtifact ⊑ MeshResource

RenditionFile ⊑ LocatedFile

MeshArtifact excludes RenditionFile (and maybe excludes purely navigational assets if you want)

3) “Probably we get rid of ConcreteRepresentation”

Agreed. It’s not buying you anything if:

LocatedFile already means “bytes at a location”

you enforce the terminal rule (“a LocatedFile has no further realizations”)

So: drop ConcreteRepresentation, keep LocatedFile, and optionally add RenditionFile as a kind of LocatedFile.

I updated the canvas to:

remove ConcreteRepresentation,

strengthen DigitalArtifact to cover Flow/Slice/AbstractFile/LocatedFile with a terminal rule,

add RenditionFile (LocatedFile).

4) Naming the exclusion concept (pick one)

Options, with downsides:

MeshArtifact: best readability; might be confused with “anything in the mesh.”

WeavableArtifact: emphasizes “managed by weave/versioning”; excludes pure HTML pages naturally.

ContentArtifact: ambiguous with “resource page content.”

PayloadArtifact: too narrow (excludes supporting flows).

If your target is “everything that participates in Flow/Slice/AbstractFile/LocatedFile, excluding resource pages,” WeavableArtifact is the least ambiguous.

If you prefer less jargon, MeshArtifact is fine, but then define it precisely as “non-rendition artifacts.”

If you want, say which of those you prefer and I’ll adjust the canvas terminology consistently (and add the MeshArtifact/WeavableArtifact class and constraints).

Mesh Resource Taxonomy Draft
I wouldn't say renditions. ResourcePages describe AbstractResources. 

Anyhow, shifting slightly... Sometimes I imagine that SemanticFlow resources can be interspersed with other resources. A mesh-repo is "all mesh", but maybe you would like to turn a namespace that also has non-mesh files into something that can support SF constructs. Also, formerly I was just jamming in all the assets that a resource page might need into a Knop's _assets folder. Whether there, or scattered around the Knop, it's possible other files could be in a mesh that aren't SemanticFlow files. (Whether ResourcePages are or are not is an interesting question, but probably yes.) But yes, maybe a better name for MeshResource is SemanticFlowResource. 


Could you have a stand-alone Nomen? It does seem to be a nice capability to turn any URL into a Nomen, whether or not it is embedded in a Semantic Mesh. Alternately, maybe Knops would become a widely-used convention for making DigitalArtifacts referenceable. 

But OMG I'm also having a crisis here of "how much overthinking can I do here?" Anyhow, you want to decide for me? Is the mesh optional? 
Thought for 31s

Mesh is optional as a repository-level packaging, but not optional as the context that makes Semantic Flow identifiers work.

If you want a decision that minimizes future complexity, adopt these three rules:

Decision 1: A mesh is a declared region inside any repo

A repo may contain zero, one, or many meshes.

A mesh root is declared by a marker/handle folder (you already have the right shape): .../_mesh/.

Everything outside a mesh root is out of scope for Semantic Flow (servable, but not part of SF’s identifier system).

This directly supports “interspersed with other resources” and “non-mesh files in the same repo.”

Decision 2: Rename MeshResource → SemanticFlowResource

Your current MeshResource definition (“anything contained in a semantic mesh”) is too broad if meshes can coexist with arbitrary files. You don’t actually mean “any file under that folder”; you mean “anything SF recognizes and can speak about structurally.”

So:

SemanticFlowResource: a resource in a declared mesh region that participates in SF constructs (Nomina, Knops, Flows, Slices, AbstractFiles, LocatedFiles, mesh components, inventories, etc.).

Arbitrary files may exist under the mesh root, but they are only SF resources if they are claimed as LocatedFile (i.e., referenced by metadata/inventory), or if they are known reserved components (resource pages, _mesh/, _knops/, etc.).

This prevents SF from accidentally “trying to be a CMS for the entire repo.”

Decision 3: “Stand-alone Nomen” is not an SF resource; it’s just RDF about an IRI

The capability you want (“turn any URL into a Nomen”) is already achieved by standard RDF: you can describe any IRI in any dataset.

What SF adds is: mint a human-facing alias inside a mesh that can denote an external IRI and carry your editorial/reference machinery.

So:

Outside a mesh: you don’t have “Nomen” as an SF construct; you have an IRI + triples about it.

Inside a mesh: you can mint /<friendly>/ as a Nomen and set denotesDigitalArtifact (or your subproperties) to any external IRI.

If you keep calling both things “Nomen,” you will confuse yourself and your users. Treat “Nomen” as the mesh-local human alias resource.

One cleanup implied by this

Your current RenditionFile term is doing the right job (resource pages are files but not primary payload artifacts). If you dislike “rendition,” rename it to ResourcePageFile (and optionally “IndexPageFile” etc.). The concept is necessary; the label is negotiable.

Net: Yes, meshes are optional in a repo. No, a Nomen is not meaningful without a mesh context.

Mesh Resource Taxonomy Draft
Under Decision 3: The other thing that SF adds is a convention for declaring what the URL denotes (if a digital artifact), what its co-denoter IRIs might be, and what its (canonical or otherwise) reference data might include. If we dropped the NomenFlow concept, it could just be an index.jsonld file or nomen.jsonld/ttl that people could put anywhere without the Flow overhead. Then anyone couldn't mint a Nomen anywhere. 
Thought for 49s

You’re right to push back on Decision 3 as I phrased it. Semantic Flow can add value even when there is no “full mesh,” because the value is partly a standard metadata envelope for “this IRI, what it denotes, co-denoters, and reference sources.”

The clean decision that avoids the identity mess:

Make “mesh” optional, but only in a strict “Nomen-only” sense

There are two tiers:

Tier A — Stand-alone Nomen (no flows, no knops required)

A Nomen exists anywhere as a dereferenceable IRI.

Its SF contribution is a single RDF metadata dataset published at that IRI (or in a well-known adjacent file), containing:

optional denotesDigitalArtifact (0..1)

sameDenotationAs / co-denoters

canonical/other referenceSource links

This is not “Flow overhead.” It’s “publish nomen.jsonld (or content-negotiated RDF) next to the IRI.”

This works for “turn any URL into a Nomen anywhere,” because the URL is the Nomen, and the metadata file is just its RDF description.

Tier B — Full mesh (knops, flows/slices, inventories, etc.)

You still want this when you need:

revision semantics (Flow/Slice)

standardized representation/location machinery (AbstractFile/LocatedFile)

reserved _mesh/, _knops/ components

inventory indexing and validation

This is the only tier where “Nomen points to a Knop” or “Nomen has canonical reference knops” becomes meaningful.

What you should NOT do

Do not try to keep “Nomen is mesh-local only” and “anyone can mint a Nomen anywhere.” Those are incompatible unless you rename one of them.

The least confusing resolution is:

Nomen = the IRI itself (anywhere)

NomenMetadataDataset = the standard RDF description of that IRI

“Mesh Nomina” are just Nomina that happen to live inside a mesh region and participate in the Knop/Flow machinery.

NomenFlow vs single file

Your instinct is correct: for Nomen metadata, a Flow is usually gratuitous.

Nomen metadata is small.

Its “history” is typically in git anyway.

If you ever need temporal semantics, you can later “upgrade” to a Flow, but don’t force it.

I updated the canvas to replace NomenMetadataFlow with NomenMetadataDataset (non-flow, required), and to explicitly allow stand-alone Nomina and interspersed non-SF files.

Consequence: your “only non-flow RDF datasets are inventories” rule must relax

If you want stand-alone Nomen metadata without flows, then NomenMetadataDataset is also a first-class non-flow RDF dataset. That’s consistent because it’s edited, not derived.

That’s now reflected in the canvas as well.

If you want to lock down naming: pick one convention for Tier A:

content negotiation at the Nomen IRI, or

a standard file name (e.g., index.jsonld) colocated with index.html.

Mesh Resource Taxonomy Draft
Please stop editing the taxonomy draft until I give permission.

Nomen != the IRI itself. The IRI itself denotes some thing, like a DigitalArtifact or a Person. I think we were going to do a the "handle trick" where /<IRI>/_nomen/ refers to the Nomen. /<IRI>/_nomen-flow would seem to refer to a RdfDatasetFlow  which is a DigitalArtifact over time. So to be consistent, even in the old "everything in a mesh" model, Nomen would have both a _nomen-handle and a nomen-flow. Which makes a lot of sense now that I think about it... the subject of most of the NomenMetadataDataset needs to be something other than the other-denoting IRI or the NomenMetadataDataset IRI. 

LOL, we managed to get rid of the _knop-handle by giving the Knop its own IRI, but the handle concept was needed after all... you can't escape the "IRI refers to the thing" vs "IRI refers to itself as a SemanticFlowResource" debacle.
Thought for 1m 6s

Acknowledged. I will not edit the taxonomy draft further until you explicitly ask.

Your “handle trick” is the correct way out of the denotation/self-reference collision, but only if you’re strict about what statements are allowed to target which IRI.

The collision you’re describing

You have two distinct things:

The designator IRI (e.g., /people/alice/) that, by your intention, denotes the referent (Person, Place, Artifact, etc.).

The Semantic Flow naming resource (“the Nomen as an object of discourse”: co-denoters, reference sources, publication assertions, etc.).

Trying to make (1) also serve as (2) is exactly the “debacle.”

Minimal, consistent resolution

Introduce a dedicated Nomen handle IRI:

Designator IRI: X (e.g., …/people/alice/) — denotes the world thing / denoted thing.

Nomen handle IRI: X/_nomen/ — denotes the Nomen resource (the naming object).

Then:

All “identifier talk” (co-denoters, publishing realizations, canonical reference sources, etc.) is asserted about X/_nomen/, not about X.

X/_nomen/ has a property that points back to the designator IRI X (call it sflo:designatorIri or sflo:hasDesignator).

This cleanly separates:

what X denotes from

what SF says about the naming/identifier itself.

Where the Nomen metadata lives (dataset vs flow)

You’re also right that the subject of the metadata dataset should not be X nor the dataset-file IRI; it should be X/_nomen/.

So you can have:

X/_nomen/_meta/_working/nomen.jsonld (or similar) as a LocatedFile whose RDF describes X/_nomen/.

Whether that metadata is a Flow (_nomen-flow) or a single dataset is a separate decision. The handle pattern works either way.

Why Knops “escaped” but Nomina don’t

Knops got their own IRI specifically as SF-native resources, i.e., “this IRI denotes this Knop.” No ambiguity.

Nomina are intentionally human-facing designators whose referents are often not SF resources. That’s why the handle is needed here and not for Knops.

Critical constraint you can’t ignore

X/_nomen/ only exists if you control the namespace under X.

So the “turn any URL into a Nomen” capability cannot literally mean “create https://other.site/foo/_nomen/”. For external IRIs, you must do one of:

Mint a local designator Y in your own namespace and say Y/_nomen/ sameDenotationAs <externalIRI> (or similar), or

Mint a local Nomen handle resource Y/_nomen/ whose sflo:hasDesignator <externalIRI> (i.e., your Nomen resource binds to an external designator).

Either way: the handle trick is still the mechanism; you just can’t host it under the external IRI.

Tight rule set (so this doesn’t sprawl)

If you adopt _nomen/, you should enforce:

“Naming metadata predicates” (co-denoters, publication assertions, canonical reference sources, etc.) MUST target X/_nomen/, not X.

X/_nomen/ sflo:hasDesignator X is required.

Resource pages for X MAY display Nomen metadata, but the RDF should still be about X/_nomen/.

That gives you the separation you want without resurrecting _knop-handle.

One naming critique

If you call it _nomen-flow, you are implicitly making Nomen metadata a DigitalArtifact-over-time. That may be fine, but it contradicts your earlier desire for “drop a nomen.jsonld anywhere without flow overhead.” You can keep both by policy:

default: X/_nomen/_meta/_working/nomen.jsonld is a mutable dataset

optional: X/_nomen/ has a metadata Flow if you need history

But pick one as the base profile; otherwise your docs will stay unstable.

If you want a decision: keep _nomen/ as the mandatory handle; make “Nomen metadata as a Flow” optional.

Mesh Resource Taxonomy Draft
I think for the co-denotation property, the object should be xsd:anyUri so that we're treating the object as a string, and not the thing it represents.

One of the ideas I played with for the handle was using a fragment identifier convention, like #_nomen-handle, that could be tacked on to external IRIs without breaking dereferencability, but unless you think that's brilliant, yeah, let's stick with "you can only mint Nomen for namespaces you control". 

For "just drop a" convention, X/_nomen/_meta/_working/nomen.jsonld/nomen.jsonld would be the symmetric location, an X/_nomen/ is defined to be the Nomen handle that refers to the nomen-as-nomen. You could just drop the distribution file at X/_nomen/_meta/_working/nomen.jsonld/nomen.jsonld  if you didn't want to set up weave functionality, and/or you could drop a turtle file at X/_nomen/_meta/_working/nomen.ttl/nomen.ttl to provide multiple formats. And then if you did ever decide to weave/keep history, everything is in place for typical flow behaviour. 

Here the AbstractFile name = LocatedFile name is a little weird, but I could see people getting use to it... "Oh, this convention means I can dereference the parent folder and get a ResourcePage with all kinds of goodies."
Thought for 1m 6s
1) Co-denotation object as xsd:anyURI is semantically wrong for “co-denotation”

If the predicate means “this other identifier denotes the same thing,” the object must be an IRI node, not a literal. Otherwise you are not asserting co-denotation; you are asserting “some string that looks like a URI.”

If you want both capabilities (strong vs weak), split it:

Strong: sameDenotationAs (range: rdf:Resource) — object is an IRI, and you are asserting it co-denotes.

Weak/operational: candidateCoDenoterString (range: xsd:anyURI or xsd:string) — object is a string you observed or want to remember without commitment.

The “treat it as a string, not the thing it represents” motivation is valid, but it applies to the weak property, not to the core co-denotation property.

If your deeper goal is “talk about the identifier as an identifier,” that is already handled by making the subject the handle (X/_nomen/). The object being an IRI is fine; you’re still asserting an identifier-relationship.

2) Fragment-handle (#_nomen-handle) is not brilliant here

It does not “preserve dereferenceability” in the way you need:

HTTP requests never send the fragment; dereferencing https://ex.org/foo#bar fetches https://ex.org/foo.

The meaning of fragments is supposed to be defined by the representation of the base resource. You can’t assume an external site defines your #_nomen-handle semantics.

So it’s okay as a purely internal RDF convention, but it’s not a robust handle mechanism for discovery or publishing.

Your “only mint under namespaces you control” constraint is the correct rule.

3) Your “drop-a-file” convention is workable, but it implies Flow structure even when you say “no weave”

This path:

X/_nomen/_meta/_working/nomen.jsonld/nomen.jsonld

is clean in one sense (it matches your Flow/Slice/AbstractFile/LocatedFile nesting), and it preserves a future upgrade path to “real weave history.”

But be explicit about what it means when there is no weaving:

You are still naming a WorkingSlice (_working) even if it’s the only slice that ever exists.

You are still committing to AbstractFile/LocatedFile layering for Nomen metadata.

That’s fine, but it’s not “no Flow overhead.” It’s “Flow-shaped layout without requiring tooling.”

4) AbstractFile token = LocatedFile filename (including extension) is consistent, just accept the weirdness

Yes, folders like nomen.jsonld/ are visually odd. They’re also mechanically very nice:

you can infer extension/MIME-ish intent without reading metadata

dereferencing the folder IRI naturally yields a resource page “about the AbstractFile”

Two gotchas you should preempt in docs:

If you ever have multiple LocatedFiles for one AbstractFile (mirrors, alternate locations), “canonical filename” must be stable, and the AbstractFile token must be anchored to that canonical one.

If you want multiple formats, you’ll have multiple AbstractFiles (nomen.jsonld/, nomen.ttl/, etc.). That’s correct and should be stated.

5) The handle trick is necessary — but don’t reintroduce it everywhere

You are correct: you cannot escape “IRI denotes referent” vs “IRI denotes the SF naming object” without some handle layer.

The key is containment: use handles only where the denotation collision actually exists:

X denotes the thing (person/artifact/etc.)

X/_nomen/ denotes the naming object

Knops don’t need this because they are already SF-native referents.

If you keep sameDenotationAs as an IRI-valued property on the Nomen handle, you also avoid the temptation to turn every external IRI into an “identifier handle resource” just to store a string.

If you want, I can propose a minimal predicate set for Nomen-handle metadata that keeps this small and avoids “handle proliferation,” without changing your directory convention.

Mesh Resource Taxonomy Draft
I think you're wrong about #1. using anyURI means "this URI as a URI". For external URIs that probably don't have a _nomen handle, this only makes sense. Otherwise you are saying "this nomen has a co-denoter, the person alice"
Thought for 40s

You’re mixing two different goals under one predicate, and that’s why this feels paradoxical.

What xsd:anyURI actually buys you

In RDF, an xsd:anyURI literal is just a literal. It’s not dereferenceable as a node, you can’t use it as an object in other triples, you can’t follow it, and you can’t do graph-joins against it without string-matching.

So yes: it treats the value “as a URI-shaped string.” But that also means you are not asserting co-denotation in a way that participates in RDF’s linking model.

Why using an IRI object is not “saying the co-denoter is the person”

If you write:

X/_nomen/ sflo:sameDenotationAs <https://ext.example/people/alice/> .


RDF semantics treats <https://ext.example/people/alice/> as a resource node, and that node denotes whatever it denotes (a person, a document, etc.). That’s not a bug—it’s literally what “co-denotation” means: you’re asserting that the denoted thing is the same.

If you are unwilling to assert that (because you want “identifier as identifier” without committing to denotation), then you don’t want sameDenotationAs. You want an “observed identifier string” property.

The actual design fork (you must pick one)
A) Strong co-denotation (semantic-web-native)

sflo:sameDenotationAs range is IRI (not literal).

This is the property you can align with owl:sameAs-like intent (even if you don’t actually equate them).

Use this when you mean it.

B) Weak “identifier mention” (no denotation commitment)

sflo:candidateCoDenoter range xsd:anyURI (or xsd:string).

This is explicitly not co-denotation; it’s “a URI string we believe might co-denote.”

Use this when you don’t mean it.

Trying to make (A) safe by turning it into (B) just destroys (A).

If you truly need “identifier-as-identifier” while still being linkable, you cannot avoid handles

You’re correct that external IRIs usually won’t have /_nomen/. If you want to talk about the identifier as an object (not its referent) and still keep graph connectivity, the only clean move is a handle you control, e.g.:

X/_id/https/ext.example/people/alice/ (encoded)

Then:

X/_nomen/ sflo:sameDenotationAsHandle X/_id/https/ext.example/people/alice/ .
X/_id/… sflo:designatorIri <https://ext.example/people/alice/> .


Now you can talk about the identifier without pretending the identifier node is the person.

But notice what this admits: your instinct is right—you can’t escape the handle layer if you want “IRI refers to referent” and “IRI refers to itself as a SF resource” simultaneously.

Bottom line

If the predicate is genuinely “co-denotes,” the object should be an IRI node, not an anyURI literal.

If you want “this is a URI-shaped string we observed,” use a different predicate with literal range.

If you want “identifier-as-identifier” and linkability for external IRIs, you need an identifier-handle IRI you control; literals won’t substitute for that.

Mesh Resource Taxonomy Draft
I still think you're wrong here. A co-denoter statement in the Nomen dataset takes the form of, e.g. "/person/alice/_nomen/ sflo:sameDenotationAs {target}". If {target} is <https://example.org/alice> you are saying that an identifier has the sameDenotationAs a person. anyURI is the only way I can think of to give the target to denotation of identifier. 
Thought for 50s

You’re using one predicate (sameDenotationAs) to do two incompatible jobs:

Strong claim: “these two designators denote the same referent.”

Mention-only: “this is another identifier string I want to record, without committing to what it denotes.”

In RDF, if {target} is an IRI node <https://example.org/alice>, then—per the semantics you’re otherwise relying on—that node denotes whatever it denotes (often a person). So yes: with a strong co-denotation predicate, you are asserting referent-level sameness. That is not an avoidable “type error”; it’s what co-denotation means.

xsd:anyURI does avoid that, but at a real cost: you no longer have an RDF link. You have a string that you can’t join against, can’t follow, and can’t validate structurally without string-parsing/casting hacks. So it’s not “the only way”; it’s “the weak, non-linking way.”

The clean resolution

Keep two properties (or two layers), and stop trying to make one property serve both semantics:

A) Strong co-denotation (linking; RDF-native)

Make sflo:sameDenotationAs point to an IRI node, but not directly to the external designator—I agree with you that you want “identifier as identifier.”

So: sameDenotationAs should relate identifier handles, not raw designator IRIs.

For mesh-controlled IRIs: the handle is X/_nomen/.

For external IRIs you don’t control: mint an ExternalIdentifierHandle in your namespace.

Example:

# your nomen handle
<.../person/alice/_nomen/> sflo:sameDenotationAs <.../_id/https/example.org/alice/> .

# external-id handle
<.../_id/https/example.org/alice/> a sflo:ExternalIdentifierHandle ;
  sflo:spellsIri "https://example.org/alice"^^xsd:anyURI .


Now sameDenotationAs relates identifier-resources (handles). You are no longer asserting “identifier has sameDenotationAs a person.” You’re asserting “identifier-handle A co-denotes with identifier-handle B.” That matches your intent.

B) Weak “mention only” (non-linking; operational)

If you still want the lightweight mode, add a separate property whose meaning is explicitly “string mention, no denotation commitment”:

<.../person/alice/_nomen/> sflo:candidateCoDenoter "https://example.org/alice"^^xsd:anyURI .


This is the place for xsd:anyURI.

Why your current proposal fails semantically

If you keep:

<.../person/alice/_nomen/> sflo:sameDenotationAs "https://example.org/alice"^^xsd:anyURI .


then either:

sameDenotationAs no longer means co-denotation (it’s just “string storage”), or

you are claiming a literal denotes the same thing as an identifier, which is nonsensical under your own denotation framing.

Minimal decision

sflo:sameDenotationAs MUST be handle-to-handle (never literal).

xsd:anyURI belongs on a property like sflo:spellsIri (on the external handle) or on sflo:candidateCoDenoter (weak mode).

This keeps your “identifier-as-identifier” stance without throwing away RDF linking.

Let's address the real costs. For joining, how would that work in SPARQL?  Could you give an example of something you couldn't do?  Why can't you follow that string? How would you validate structurally, why would you need to?

I'm open to two properties. equivalentNomen subject would be a Nomen, i.e., via the NomenHandle, but that only works for SF-compatible co-denotation.  Another property might be sameAs between the Nomen's "containing IRIs" (or whatever you want to call the parent folder of the NomenHandle). But I wary of sameAs. maybe skos:closeMatch is better.  

logically, for digital artifacts, equivalentNomen would basically have to sflo:denote the same thing, whether represented by the same IRI or different IRIs that were just different publishing contexts for the same Knop. 

I still think you're wrong, or maybe missing the point. It's the IRI-as-identifier that denotes something. You can't say that "a person" or whatever denotes something, so you can say that <https://example.com/alice> is a co-denoter. "https://example.com/alice" is a URI, therefore an identifier, therefore it can denote something when used in angle brackets.

Thought for 1m 2s

Using xsd:anyURI as the object is workable, but it has real, concrete costs in RDF/SPARQL that you don’t pay if the object is an IRI node (or an SF “external handle” IRI you mint). Here are the costs, with exact examples.

1) Joining in SPARQL: literals force casting and block graph traversal
If the object is an IRI node (easy join)

Data:

</person/alice/_nomen/> sflo:sameDenotationAs <https://example.com/alice> .
<https://example.com/alice> schema:name "Alice Example" .


Query:

SELECT ?name WHERE {
  </person/alice/_nomen/> sflo:sameDenotationAs ?id .
  ?id schema:name ?name .
}

If the object is xsd:anyURI literal (you must cast)

Data:

</person/alice/_nomen/> sflo:sameDenotationAs "https://example.com/alice"^^xsd:anyURI .
<https://example.com/alice> schema:name "Alice Example" .


Query becomes:

SELECT ?name WHERE {
  </person/alice/_nomen/> sflo:sameDenotationAs ?u .
  BIND( IRI(STR(?u)) AS ?id )
  ?id schema:name ?name .
}


This is already worse, but the bigger problem is:

Literals cannot be subjects, so you can’t traverse “identifier metadata”

If later you want to attach anything to the identifier itself (even something trivial like “seen on date”, “source”, “status”), you can do:

<https://example.com/alice> sflo:seenAt "2026-01-17"^^xsd:date .


…but only if you stored it as an IRI node. If you stored it as a literal, there is no node to attach to. You’re stuck with string-keyed “side tables” or more literals.

That is the single biggest loss: you can’t treat the external designator as a graph node. You can only treat it as a string.

2) “Why can’t you follow the string?”

You can follow it if your application adds custom logic: parse the string and make an HTTP request.

What you can’t do is “follow your nose” with ordinary RDF tooling that expects links to be RDF terms (IRIs) it can dereference, cache, and use in graph navigation. A literal is not a link in the RDF graph; it’s just a value.

So the cost is not “impossible,” it’s “every client must implement special-case behavior.”

3) Structural validation: why you might need it, and what you lose with literals

If you use an IRI node, SHACL can validate graph structure cleanly:

object is an IRI: sh:nodeKind sh:IRI

object is within a controlled namespace (if you want that as a warning)

object has an _nomen handle (if SF-controlled)

With an xsd:anyURI literal you can only validate lexical form (pattern/scheme). You cannot validate “this co-denoter has a corresponding handle node in this mesh” because it isn’t a node. That matters if you ever want “SF-compatible co-denotation” to be checkable.

4) The key semantic point you’re arguing about

You’re saying: “If {target} is <https://example.com/alice> then I’m saying the co-denoter is the person Alice.”

No: syntactically you are linking to an IRI term, which in RDF denotes some resource. If your predicate means “co-denotes,” you are intentionally making a referent-level claim (“these two IRIs pick out the same thing”). That is exactly what co-denotation is.

If you want “identifier mention without denotation commitment,” that is a different predicate.

Recommendation: stop forcing one predicate to mean two things

Use two properties with distinct meanings:

A) Strong / RDF-native co-denotation (object is IRI)

sflo:sameDenotationAs with sh:nodeKind sh:IRI

Use this when you are willing to assert co-denotation.

B) Weak “mention / observed identifier string” (object is literal)

sflo:candidateCoDenoterUri with sh:datatype xsd:anyURI

Use this when you are not committing to co-denotation and just want to record a URI-shaped string.

This gives you your “external IRIs probably don’t have handles” practicality without throwing away the graph.

If you want “identifier-as-identifier” and strong linking, you still need handles

If you insist that the object of “equivalent nomen” must be a Nomen (handle) rather than a designator IRI, then external IRIs need an SF-side handle resource you control. There is no third option that preserves both:

identifier-as-identifier semantics, and

graph-linkability, and

no minted handle.

Literals only satisfy the “no minted handle” constraint by giving up linkability.

On owl:sameAs / skos:closeMatch

Those don’t solve the literal-vs-IRI problem; they only change the strength/intent of the mapping. If you want a weaker assertion than co-denotation, use skos:closeMatch-like semantics, but still keep the object as an IRI node when you want graph connectivity.

If you answer one question: do you want sameDenotationAs to be a strong referent-level claim, or a weak “identifier mention”? Right now you’re trying to make it both, and that’s why the range debate won’t converge.

Mesh Resource Taxonomy Draft
You might be right, but the way it should be is that if you use an IRI term like <http://example.org/alice/> you're not talking about the term, you're talking about alice. alice can't co-denote. 

Anyhow, I agree, you could mint a new Nomen, and in the NomenFlow, assert that the Nomen's parent IRI sameAs the external IRI, and then say the original Nomen sameDenotationAs the new local Nomen. 
Thought for 57s

You are right about one thing and wrong about another, and the difference is precisely where RDF forces you to choose.

1) You’re right: RDF does not let you talk about the token <http://example.org/alice/>

In RDF, writing <http://example.org/alice/> introduces an RDF IRI node. RDF does not have a built-in way to distinguish:

“the identifier token” vs

“the thing denoted by that identifier”

So if your intended meaning of sameDenotationAs is meta-level (“these two identifiers co-denote”), RDF can’t express that about tokens unless you introduce a separate resource that stands for “the identifier-as-identifier” (your handle trick).

That part of your critique (“alice can’t co-denote”) is conceptually correct in philosophy-of-reference terms.

2) You’re wrong that using an IRI node necessarily asserts “the co-denoter is the person Alice”

This is the subtle RDF point:

If your dataset contains only:

</person/alice/_nomen/> sflo:sameDenotationAs <https://example.com/alice> .


you have not asserted <https://example.com/alice> is a schema:Person. You’ve only introduced it as a node.

The problem is not that it becomes “a person” by default; the problem is that RDF provides no way to guarantee it stays “just an identifier token” if you later merge in data that types it as a person, or if consumers assume it denotes a person (which many will). That’s exactly why your handle is a good idea.

So: using an IRI object isn’t inherently inconsistent, but it’s semantically underspecified relative to your “identifier-as-identifier” stance.

3) Your proposed compromise is coherent

Mint a new local Nomen for the external IRI; assert the parent IRI is “same as” the external IRI; then link local Nomina via sameDenotationAs.

Yes—this is essentially “internalize the external designator into your naming system.”

Concretely:

Let X = </person/alice/>

Local external-alias Nomen: Y = </_ext/example.com/alice/> (whatever scheme you pick)

Handles: Xn = X/_nomen/, Yn = Y/_nomen/

Then:

# co-denotation between *Nomina-as-naming-objects*
Xn sflo:sameDenotationAs Yn .

# the external alias Nomen points outward
Yn sflo:designatorIri <https://example.com/alice> .


Now your co-denotation predicate stays purely handle-to-handle (clean), and the external IRI only appears as a designator IRI (also clean).

Avoiding owl:sameAs

Do not use owl:sameAs for “designator equality across contexts.” It’s too strong and will be misused by reasoners.

If you want a weaker mapping, define an SF predicate, e.g.:

sflo:sameDesignatorAs (meaning: treat these designator IRIs as equivalent names)

or reuse SKOS mapping predicates (skos:exactMatch / skos:closeMatch) only if you accept their concept-scheme framing (often you won’t).

But don’t pretend SKOS solves the token-vs-referent issue; it doesn’t. It only changes strength.

4) What this implies for your property design

You end up with two layers (and this is stable):

sflo:sameDenotationAs

domain/range: NomenHandle → NomenHandle

meaning: co-denotation between naming objects

sflo:designatorIri (or sflo:spellsIri)

domain: NomenHandle

range: IRI (node) or xsd:anyURI literal (choose one)

meaning: the outward-facing designator this Nomen corresponds to (optional)

If you choose xsd:anyURI for designatorIri, you’re explicitly saying “this is just a string mention,” which is fine for outward pointers. You do not want xsd:anyURI for sameDenotationAs itself.

The decision point

If your priority is “identifier-as-identifier semantics stays clean even under graph merges,” then you must keep sameDenotationAs handle-to-handle, and “external IRIs” must be represented via a local alias Nomen (as you just proposed).

That gives you the model you want without relying on literals as a substitute for identity.

Mesh Resource Taxonomy Draft
Still problematic:

Yn sflo:designatorIri <https://example.com/alice> .

You're saying that alice is a designatorIri.

For my purposes I'm assuming that <https://example.com/alice> a Person somewhere. It doesn't matter whether theres specifically a Person statement out there. The fact is that <https://example.com/alice> almost always denotes something that isn't an identifier. Until everyone starts using Nomina handles, anyURI is the best we can do to differentiate "as identifier". 

My suggestion to point outward was Y sameAs <https://example.com/alice>
Thought for 55s

Your core point is valid: in RDF, an IRI term is “used” to talk about its referent, not “mentioned” as a token. If you want “mentioned-as-identifier,” you need either (a) a handle resource you control, or (b) a literal string. There is no third option.

Where you’re overreaching is here:

Yn sflo:designatorIri <https://example.com/alice> . means “alice is a designatorIri”

That triple does not assert “Alice is an identifier.” It asserts: the Nomen-handle Yn has a designator whose RDF node is <https://example.com/alice>. But yes: RDF still treats that node as denoting whatever it denotes (often a person), so if your intent is token-mention, you shouldn’t use an IRI object for that predicate.

What you lose with xsd:anyURI literals (real costs, concretely)
A) SPARQL joins become “cast-and-hope”

If sameDenotationAs is a literal:

</person/alice/_nomen/> sflo:sameDenotationAs "https://example.com/alice"^^xsd:anyURI .


To join to any triples about <https://example.com/alice> you must cast:

SELECT ?name WHERE {
  </person/alice/_nomen/> sflo:sameDenotationAs ?u .
  BIND( IRI(STR(?u)) AS ?id )
  ?id schema:name ?name .
}


That’s not terrible—but it’s boilerplate everywhere, and it fails if the lexical form isn’t a valid IRI after normalization.

B) You can’t attach identifier-level metadata without inventing another keying scheme

With an IRI object you can do:

<https://example.com/alice> sflo:seenIn <...source...> .


With a literal object, there is no node to hang metadata on. You end up either:

duplicating metadata as more literals (string-keyed), or

reifying it with an extra structure anyway (which is basically reinventing handles).

C) Equality is trickier than you think

Two literals can differ while denoting the “same” IRI after normalization (percent-encoding, trailing slashes, case rules in host, etc.). If you store IRIs as strings, you either accept duplicates or you must define canonicalization rules.

“Why can’t you follow the string?”

You can, but only with custom client logic. RDF tooling “follows links” by dereferencing IRI nodes; literals are inert. If you’re happy to define “following” as “parse string → fetch,” then this cost is acceptable.

Validation: why you might want it

If you ever want rules like:

“warn if co-denoter is not https”

“warn if host not in allowlist”

“warn if co-denoter is SF-controlled but you didn’t use a handle”
then literals force regex/pattern checks only. IRI nodes allow node-kind constraints plus graph structure checks.

The clean model that matches your philosophy-of-reference stance
1) Keep handle-to-handle for SF-compatible co-denotation

This is your equivalentNomen idea:

subject/range: NomenHandle -> NomenHandle

No literals here.

2) Add a second property specifically for external designator mention

Make it explicit that it’s mention-only:

sflo:externalCoDenoterUri (range: xsd:anyURI)

This is where xsd:anyURI belongs. It is intentionally non-linking.

This resolves your “alice can’t co-denote” objection: you are not asserting a referent-level relation; you’re recording an identifier string.

3) If you later want linkability, mint an external-handle (optional profile)

When you do need joins/metadata on external identifiers, you mint:

/_id/https/example.com/alice/ (an ExternalIdentifierHandle)

attach the string/IRI there

Then you can keep sameDenotationAs handle-to-handle universally.

About Y owl:sameAs <https://example.com/alice>

This is only coherent if Y is the designator IRI (the “containing IRI”), not the Nomen-handle.

If Y denotes Alice-the-person, then Y owl:sameAs <https://example.com/alice> is a referent-level identity claim. That may be too strong (reasoners treat it as full identity), which is why you’re wary.

If you want “co-denotes” but not full OWL identity, define your own predicate (or use a weaker mapping predicate with clear semantics). skos:closeMatch is usually the wrong fit unless you’re willing to treat these as concept mappings.

Recommendation

Make sflo:sameDenotationAs handle→handle only.

Add sflo:externalCoDenoterUri with xsd:anyURI for external co-denoters you don’t (or can’t) internalize.

Optionally support ExternalIdentifierHandle later when you need joins without casting and you want to annotate external identifiers.

This keeps your “identifier mention” purity without forcing everything into handles on day one.

Mesh Resource Taxonomy Draft
I think we can just say <nomen-handle> sflo:denotes <https://example.org/alice> i.e., re-use the property that identifies PayloadFlows. 
Thought for 1m 10s

Reusing sflo:denotes on the NomenHandle is coherent, but it does not solve the “IRI-as-token” problem you were trying to solve. It commits you to referent-level semantics.

What X/_nomen/ sflo:denotes <https://example.org/alice> means

It means: the Nomen (as a naming resource) denotes the same referent that <https://example.org/alice> denotes.

That is stronger than “this string is a co-denoter,” and it’s exactly the thing you previously objected to with designatorIri: you are still using an IRI term, which (in RDF) denotes its referent.

If you accept that, then you don’t need xsd:anyURI for co-denoters at all; just use IRI objects everywhere and live with referent-level assertions.

What you cannot do (cleanly) if you switch to xsd:anyURI for co-denoters

You can do everything, but you pay in friction and loss of graph structure:

Without literals (IRI object):

SELECT ?name WHERE {
  </person/alice/_nomen/> sflo:denotes ?x .
  ?x schema:name ?name .
}


With literals (anyURI):

SELECT ?name WHERE {
  </person/alice/_nomen/> sflo:coDenoterUri ?u .
  BIND( IRI(STR(?u)) AS ?x )
  ?x schema:name ?name .
}


The “real” cost is not the cast—it’s that you can’t attach any identifier-level metadata to the co-denoter without inventing a second structure. A literal can’t be a subject, so you can’t later say “this co-denoter string was observed from source S at time T” except by reifying it into another node (i.e., a handle).

The minimal model that preserves your intent

You want two different notions:

Denotation (referent-level claim): use IRI objects.

Identifier mention (token-level, non-committal): use xsd:anyURI literals.

So:

sflo:denotes (or your existing denotesDigitalArtifact subproperty)
Range: IRI node
Meaning: denotation claim (strong)

sflo:coDenoterUri (new)
Range: xsd:anyURI
Meaning: “I’m recording a URI-shaped identifier string as a possible co-denoter; no denotation claim.”

This matches your philosophical stance: “alice can’t co-denote; identifiers co-denote.” The literal property is the “identifier mention” channel.

Where sameDenotationAs fits

Keep sflo:sameDenotationAs handle→handle only, for SF-controlled co-denoters where both sides have Nomina handles.

External IRIs then have two supported representations:

Weak mode: record them only via sflo:coDenoterUri literal.

Strong/internalized mode: mint a local Nomen for the external IRI and relate handles with sflo:sameDenotationAs. (Avoid owl:sameAs unless you truly want OWL identity; if you need an outward link, use a weaker SF predicate like sflo:externalDesignatorUri as a literal.)

Direct answer to your proposal

Yes, you can say:

<nomen-handle> sflo:denotes <https://example.org/alice>

…but that is explicitly a denotation statement, not an “identifier-as-identifier” statement. If you want “as identifier,” you still need either:

a literal (xsd:anyURI), or

a minted external identifier handle.

If you pick one default, the least internally contradictory is:

sflo:denotes = referent-level (IRI object)

sflo:coDenoterUri = token-level mention (anyURI literal)

sflo:sameDenotationAs = handle-to-handle only

Mesh Resource Taxonomy Draft
Sorry, I meant <nomen-parent-IRI> sflo:denotes <https://example.org/alice>
Thought for 33s

<nomen-parent-IRI> sflo:denotes <https://example.org/alice> does not do what you intend, given the rest of your model.

If your position is:

X (the Nomen “parent IRI”, e.g. /person/alice/) denotes Alice-the-person, and

X/_nomen/ denotes “the Nomen-as-nomen (the naming object)”

then in RDF any triple with subject X is a statement about Alice, not about the identifier token and not about the Nomen object.

So this triple:

</person/alice/> sflo:denotes <https://example.org/alice> .


reads as:

Alice (the person) denotes https://example.org/alice.

That’s exactly the “alice can’t denote” problem you were complaining about earlier. You’ve just moved it from the object position to the subject position.

What does work (and is consistent with your handle trick)
A) Denotation belongs on the NomenHandle, not the parent IRI
</person/alice/_nomen/> sflo:denotes <https://example.org/alice> .


This says: the naming object denotes whatever the external IRI denotes. That’s the only place sflo:denotes can live without collapsing your “Nomen != IRI” stance.

You can (and should) enforce this in SHACL:

sflo:denotes domain: sflo:NomenHandle (or whatever you call it)

and/or sh:targetClass-based constraints.

B) If you need “identifier mention, not denotation,” keep it separate

If your goal is sometimes “record this co-denoter as a URI string without committing,” then keep a literal-valued property:

</person/alice/_nomen/> sflo:coDenoterUri "https://example.org/alice"^^xsd:anyURI .


But don’t try to make that be sflo:denotes or sflo:sameDenotationAs. It’s a different claim.

Your proposed alternative: Y owl:sameAs <https://example.org/alice>

This only makes sense if Y is itself an IRI that denotes Alice-the-person. In that case owl:sameAs is extremely strong (full identity). If you want a “co-denotes” assertion without OWL’s global substitution behavior, define an SF predicate for it (or use a weaker mapping predicate intentionally), but it still lives on the NomenHandle layer if you want token-level discipline.

Bottom line

If you keep “X denotes the person,” then sflo:denotes cannot be asserted with subject X without saying “the person denotes …”.

Put sflo:denotes on X/_nomen/ (handle), and optionally keep a separate xsd:anyURI property for “mention-only / uncommitted” external co-denoters.

Mesh Resource Taxonomy Draft
You're right.  I meant what I originall wrote, <nomen-handle> sflo:denotes <https://example.org/alice>

Yes—<nomen-handle> sflo:denotes <https://example.org/alice> is the consistent form.

It has two important consequences (good and bad):

It is a referent-level claim. You are asserting that your Nomen (as a naming object) denotes whatever <https://example.org/alice> denotes. That’s not “mentioning the identifier as a token”; it’s committing to co-denotation.

It avoids the “Alice denotes things” nonsense, because the subject is the handle (the naming resource), not the denoted person.

A minimal, internally consistent pattern looks like:

# the naming object (handle) denotes the referent via an external designator IRI
</person/alice/_nomen/> sflo:denotes <https://example.org/alice> .

# co-denotation inside SF should still be handle-to-handle
</person/alice/_nomen/> sflo:sameDenotationAs </people/alice/_nomen/> .


If you still want the “identifier mention without commitment” channel, keep it separate (literal-valued), but that is no longer denotes:

</person/alice/_nomen/> sflo:coDenoterUri "https://example.org/alice"^^xsd:anyURI .

Mesh Resource Taxonomy Draft
Can you please update the taxonomy draft, stripping away everything possible to make it extremely concise, with all the classes we've discussed.
Thought for 19s

Updated the taxonomy draft in the canvas to an extremely concise form, preserving the class set discussed and removing the design notes and elaborations.

Mesh Resource Taxonomy Draft
Excellent, big improvement. One thing that strikes me from the NomenMetadataDataset is that, with the flow-compatible pathing, it is indistinguishable from a Flow. Right? "History" could be turned on at any time. So I think all datasets should be Flows, i.e., our only pure datasets were inventory. If we do them as flow (e.g. KnopInventoryFlow, with slices as KnopInventoryDataset) we are uniform.

With RDF we are lucky that we have terms for the abstract thing vs the concrete thing (Dataset vs Distribution). We don't have that for every artifact type (although Image and ImageFile could be a decent pattern.) But given that there are lots of types of thing and to avoid a profusion of classes, maybe we introduce Flow/Slice "content roles" (needs a better name). But I think we keep specialty classes for RdfDatasetFlow and RdfDataset. 

Please don't make changes to the Taxonomy Draft without my permission.
Thought for 1m 5s

Acknowledged. I won’t modify the taxonomy draft unless you explicitly ask.

NomenMetadataDataset being indistinguishable from a Flow

Under your flow-compatible pathing, yes: a “dataset-only” thing stored at .../_meta/_working/<abstractfile>/<locatedfile> is structurally a degenerate Flow (Flow with only a WorkingSlice and no historical slices yet). If clients can infer “Flow-ness” from path conventions, then a non-flow dataset placed there is effectively lying to the filesystem semantics.

That pushes you toward a clean rule:

If it lives in Flow/Slice/AbstractFile/LocatedFile layout, treat it as a Flow, even if it never gets historical slices.

Otherwise you need an actually-distinct layout for true non-flow datasets (which breaks your “turn history on at any time without moving anything” benefit).

“All datasets should be Flows” vs “inventory is pure dataset”

You’re currently saying two incompatible things:

“Only pure datasets are inventory” (non-flow)

“If we do them as flow (KnopInventoryFlow …) we are uniform” (inventory as flow)

Pick one; otherwise you’ll keep re-litigating it.

If you want uniformity and flow-shaped pathing everywhere, the simplest consistent rule is:

Everything discourse-worthy is a Flow.

Inventory is either:

a Flow with a WorkingSlice only (and optionally occasional historical snapshots), or

not stored in flow layout at all.

My bias: if inventory is derived and potentially huge, keeping it non-versioned is defensible, but then don’t store it in flow layout. If you do store it in flow layout, make it a Flow.

Dataset/Distribution analogy and “content roles”

You’re right that RDF has a clean abstract/concrete split (Dataset vs Distribution). In your system, you already have an even cleaner general split:

Flow (artifact over revisions)

Slice (revision)

AbstractFile (representation spec)

LocatedFile (bytes)

So the missing piece isn’t “Dataset vs Distribution,” it’s “what kind of thing is this Flow?”

You have two viable approaches:

Specialized classes (your Pattern A preference)

RdfDatasetFlow ⊑ Flow

RdfDatasetSlice ⊑ Slice (optional but helps SHACL)

RdfGraphFlow, ImageFlow, etc. only where you truly need constraints

A single cross-cutting kind property (your “content roles”)

sflo:artifactKind (better name than “role”) whose values are stable kinds like sflo:RdfDataset, sflo:Image, sflo:Text, etc.

SHACL enforces: Flow’s kind must match all its slices/files.

You can combine them without bloat: keep RdfDatasetFlow as a class and require artifactKind=RdfDataset for that class. That gives you easy SHACL targeting and a uniform property for generic tooling.

Names that won’t confuse OWL people: artifactKind, payloadKind, or contentKind. Avoid “role.”

Keeping RdfDatasetFlow and RdfDataset

If you keep both, be strict about which level RdfDataset applies to. The least confusing is:

RdfDatasetFlow — says the evolving artifact is an RDF dataset over time

RdfDatasetSlice (or RdfDatasetSnapshot) — says this revision is a dataset instance

If you keep only RdfDataset (no RdfDatasetSlice), you’ll be tempted to type the Flow itself as RdfDataset, and then you’re back to category errors (“the dataset over time” vs “a dataset snapshot”). This is where a tiny bit of class proliferation pays off.

If you want, I can propose the minimal SHACL constraints for:

“everything in flow-layout must be typed as a Flow,” and

artifactKind invariance across Flow/Slice/AbstractFile/LocatedFile,
without changing the taxonomy draft.

Mesh Resource Taxonomy Draft
artifactKind is okay. payloadKind is not because some Flows are not PayloadFlows, and I presume we'd want to be consistent and specify artifactKind everywhere. If not, if only PayloadFlows get this property, payloadKind is better. Non-payload flows are all RdfDatasetFlows (except for maybe OpslogFlows? if we make those in some other format there's a distinction).

I like RdfDataset because it seems generally useful in other contexts, but  I could go either way. RdfDatasetSlice sounds like a slice of a dataset. We can have SHACL validate that no Flows are RdfDataset. 

Which also makes me think DigitalArtifact should be limited to Slices. As you mentioned earlier, a file isn't really a DigitalArtifact, and I think in an way neither is a DigitalArtifactFlow. Perhaps we can get rid of DigitalArtifact altogether? It's not clear whether it's just for Payloads or whether other types of Flows' Slices are DigitalArtifacts too. 
Thought for 56s

Two separate issues are getting conflated again:

Where do you attach “what kind of content is this?” (artifactKind / payloadKind)

What is the thing you want to call a “DigitalArtifact”? (series vs revision vs representation vs bytes)

1) artifactKind vs payloadKind

Your argument for payloadKind only holds if only PayloadFlows ever need a kind. That’s already false in your own model, because:

Supporting flows are “all RDF dataset flows”… until they aren’t. The second you add (say) a binary index, a compiled sitemap, a cached search corpus, an image derivative, etc., you either (a) proliferate special cases, or (b) wish you had a uniform kind mechanism.

So if you want stability: use sflo:artifactKind on every Flow.
Then you can have SHACL say:

PayloadFlow MUST have artifactKind.

SupportingFlow SHOULD have artifactKind (and in most cases it’ll be RdfDataset anyway).

If you instead use payloadKind, you will later regret the asymmetry because you’ll want generic tooling (“show me all flows and their kinds”) and you’ll end up inventing supportingKind or similar.

2) RdfDataset and the “RdfDatasetSlice” naming problem

You’re right that “RdfDatasetSlice” reads like “a slice of a dataset” (as if the dataset is the thing being sliced). It’s still technically accurate, but it’s ugly.

Two cleaner options:

Option A (minimal classes, best readability):

Keep sflo:RdfDataset as the class of Slice-level snapshots (the “dataset as a thing you can load”).

Do not create RdfDatasetSlice.

Use SHACL to enforce:

No Flow is a RdfDataset.

If a Flow has artifactKind = RdfDataset, then all its hasSlice objects MUST be rdf:type sflo:RdfDataset.

This gives you the “generally useful in other contexts” RdfDataset class without proliferating awkward names.

Option B (more explicit, more classes):

RdfDatasetFlow ⊑ Flow

RdfDatasetRevision ⊑ Slice (avoid “Slice” in the name)

RdfDataset ⊑ RdfDatasetRevision (if you want a shorter alias)

But you said you want to avoid profusion, so Option A is the better fit.

3) You’re not consistent about what “DigitalArtifact” is

You just said:

a file isn't really a DigitalArtifact, and neither is a DigitalArtifactFlow

…but earlier you were using “DigitalArtifact” specifically to unify Flow/Slice/AbstractFile/LocatedFile.

You need to pick which concept you mean:

Meaning 1: “Information artifact at any layer”
Then yes, the file (LocatedFile) is a digital artifact. It is literally the concrete artifact.

Meaning 2: “The discourse-worthy thing” (the content entity)
Then the best candidate is usually Slice (a revision snapshot), and Flow is an artifact-series.

If you want Meaning 2, then “DigitalArtifact” should not be the superclass of LocatedFile. It should be closer to “RevisionArtifact” (Slice). And you’d introduce separate terms for the other layers:

ArtifactSeries = Flow

ArtifactRevision = Slice

ArtifactRepresentation = AbstractFile

ArtifactRealization = LocatedFile

If you don’t want that much renaming, then yes: drop DigitalArtifact entirely and just talk in terms of Flow/Slice/AbstractFile/LocatedFile. The cost is you lose a single superclass for SHACL targeting and documentation grouping, and you’ll replace it with either:

a SHACL target union (“targetClass Flow OR Slice OR AbstractFile OR LocatedFile”), or

a lightweight umbrella class like ArtifactLayer that is explicitly “these four, nothing philosophical.”

If you keep any umbrella class, don’t call it DigitalArtifact unless you’re willing to accept that files are artifacts (because, in practice, they are).

4) The part you’re underestimating

If you eliminate DigitalArtifact and avoid adding an umbrella, you’ll quickly end up with repetitive SHACL shapes and repetitive prose (“Flow/Slice/AbstractFile/LocatedFile”) everywhere. That tends to rot.

So the practical middle ground is:

Keep an umbrella class, but name it ArtifactLayer (or ArtifactResource) rather than DigitalArtifact.

Put artifactKind on Flow, validate it down the chain.

Recommendation

sflo:artifactKind on all Flows (PayloadFlow required; SupportingFlow strongly encouraged).

Keep sflo:RdfDataset as the Slice-level class; forbid Flow∈RdfDataset via SHACL; enforce artifactKind=RdfDataset ⇒ all slices a RdfDataset.

Replace DigitalArtifact with a neutral umbrella like ArtifactLayer if you still want grouping; otherwise drop it and accept verbosity.

If you want, I can propose the exact SHACL constraints for the above without changing the taxonomy draft.

Mesh Resource Taxonomy Draft
The parenthetical addition to SemanticFlowResource  "(has an in-mesh IRI and a filesystem location)" is misleading, as a mesh may contain assets helpful for supporting ResourcePages. 

Which reminds me, should assets live in well-defined places? You could require all assets to live in Knops, but I don't love that idea -- it leads to a profusion of Knops. But we should definitely support the use of Knopified assets. 

Before I've had an _assets folder in each Knop. 
Thought for 35s

The parenthetical is misleading, but the deeper problem is that your taxonomy currently conflates:

“things SF assigns semantics to” vs

“files that happen to exist in a mesh repo (including presentation assets, tooling files, arbitrary project files)”.

If you keep SemanticFlowResource as “anything participating in SF conventions,” then the parenthetical should not describe the whole mesh; it should describe the SF-governed subset.

Fix the definition (without expanding the taxonomy)

Instead of:

“(has an in-mesh IRI and a filesystem location)”

use something like:

“(SF-governed: identity/meaning is defined by SF conventions and mapped to a filesystem location)”

Or remove the parenthetical entirely; right now it reads like “everything on disk is a SF resource,” which you do not want if meshes can be interspersed with non-SF material.

You probably also want an explicit sentence (not necessarily a class) somewhere in the docs:

“A mesh repository may contain non-SF files; only reserved paths and declared resources are SF-governed.”

If you don’t say this, every later rule becomes ambiguous (“does this apply to all files or only SF-managed ones?”).

Should assets live in well-defined places?

Yes, because without conventions you get non-determinism in resource page rendering (“which CSS/logo did I mean?”) and you lose transposability (relative links break under composition).

But “well-defined places” does not mean “every asset is a Knop.”

You need three asset categories

You will otherwise keep oscillating between “profusion of Knops” and “no place to put things.”

Presentation assets (images/css/js used by resource pages, themes, docs)

Should be plain LocatedFiles.

Should live in predictable places.

Should be referenced by stable relative paths.

Payload assets (assets that are part of the domain artifact itself)

Might be payload files inside a Knop (if you want them versioned with the payload).

Or separate Knops if you need independent identity/provenance.

Operational assets (build caches, indexes, generated artifacts, logs)

Belong under reserved operational paths (_mesh/...) and should be ignorable/cleanable.

If you don’t draw these lines, you’ll keep trying to make one mechanism serve incompatible goals.

A minimal asset placement convention that avoids Knop explosion

Keep your per-Knop _assets/, but add a mesh-level place for shared assets, and an inheritance rule.

Proposed places

Per Knop: <knop>/_assets/…
Best for assets tightly coupled to that Knop’s resource page or payload.

Mesh shared: /_mesh/_assets/… (or a reserved root /_assets/… if you prefer shorter)
Best for shared logos, CSS, JS, icons, etc.

Template/theme assets: (wherever your template system already puts them, but they should resolve as a last resort or be explicitly imported)

Resolution rule (critical)

When rendering a resource page for a given Nomen/Knop, asset lookup should follow a predictable precedence, e.g.:

closest _assets/ at the current node

walk up parents to find inheritable _assets/ (optional)

mesh shared assets

template/theme defaults

If you don’t define precedence, composition will produce accidental shadowing.

When to “Knopify” an asset

Only when you need one or more of:

independent denotation (“this image is a thing worth naming”)

provenance/attribution/licensing metadata

versioning semantics beyond “the file changed”

reuse across meshes with identity preserved

Everything else should remain a plain file in _assets/.

One more sharp edge you should address now

If assets are allowed “anywhere,” users will put them next to content, and later you’ll want to reserve underscore paths. That becomes a collision nightmare.

So the rule should be:

Underscore directories are reserved for SF components and conventions.

Arbitrary non-SF files may be interspersed, but not under reserved underscore paths unless they are explicitly “asset folders” with known semantics (_assets/).

That keeps your namespace clean.

If you want, I can suggest exact one-paragraph wording for the taxonomy line + a short “Assets” section (no canvas edits unless you tell me to).

Mesh Resource Taxonomy Draft
 previous turn: 1) agree, artifactKind. 2) Option A, RdfDataset 

3) I don't understand the purpose of introducing ArtifactLayer ArtifactSeries/Revision/Representation/Realization because the have the same connotation as Flow/Slice/AbstractFile/LocatedFile. DigitalArtifact is potentially useful as a term for Flow+All its slices+All the slices' AbstractFiles+All the AbstractFiles' LocatedFiles, take all together. (I know I'm flip-flopping on this) We don't have a term yet for the 4-layer things that are contained in Knops and Nomen. Do we? Do we need it? Or is it enough to say a Flow, since the other things are contained in a Flow.  Is there a word for something that is abstract and concrete at the same time? 

most recent turn: just remove the parenthetical. I re-defined Mesh to "A servable filesystem region that contains SemanticFlowResources and, optionally, other supporting resources"

Do we need to differentiate LocatedFile into the things that have AbstractFile (i.e., part of the DigitalArtifact or whatever better name we find) vs ResourcePages and possibly other assets (which don't have AbstractFiles)? 

I agree _assets are per-knop and per-mesh. Templates are their own Knops so  template assets can just go in their knop/_assets. Sharp edge is fine. Please make all these agreed updates.
Thought for 1m 40s

Applied the agreed updates to the taxonomy draft:

Removed the misleading parenthetical from SemanticFlowResource.

Kept Mesh definition as you rewrote it.

Removed NomenMetadataDataset (so metadata can remain WorkingSlice-only and become historical later).

Added ArtifactLocatedFile vs other LocatedFile kinds, and added AssetFile.

Adopted sflo:artifactKind on Flow.

Updated RdfDataset per Option A (dataset snapshot; also used for standalone datasets like inventories).

Added explicit _assets conventions (mesh-level + per-knop; templates as knops).

Mesh Resource Taxonomy Draft
What did you think about this part of my last response, probing whether there's a better term than DigitalArtifact? "We don't have a term yet for the 4-layer things that are contained in Knops and Nomen. Do we? Do we need it? Or is it enough to say a Flow, since the other things are contained in a Flow. Is there a word for something that is abstract and concrete at the same time?"

Maybe FlowArtifact? FourLayerArtifact? 
Thought for 1m 4s

You currently have an internal inconsistency: “DigitalArtifact” is defined as a 4-layer family rooted at a Flow, but then you type Flow, Slice, AbstractFile, and ArtifactLocatedFile as DigitalArtifact. That uses “DigitalArtifact” as an umbrella class, not as “the family-as-a-whole.”

You need to decide which of these two meanings you want.

Option 1: Don’t introduce a new term (recommended if you want minimal ontology)

Treat Flow as the “thing” (the artifact identity over time). Then:

Slice / AbstractFile / ArtifactLocatedFile are parts / realizations / representations of that Flow.

You don’t need a term meaning “abstract and concrete at the same time,” because the “at the same time” thing is just the Flow (and the others are structural layers under it).

This is the cleanest and avoids a term that will get overloaded.

Option 2: Keep DigitalArtifact as an umbrella class (recommended if you want a grouping class)

Redefine DigitalArtifact to mean:

Any member of the artifact stack: Flow OR Slice OR AbstractFile OR ArtifactLocatedFile.

Then delete the “family rooted at a Flow” phrasing. Use a different phrase in prose like “artifact stack” or “artifact closure” when you mean the whole set.

If you do this, “DigitalArtifact” is fine, but don’t pretend it denotes the family object. It denotes members.

Option 3: Introduce an explicit “family” term (only if you truly need it)

If you really want a term for “Flow + all its layers taken together” as a single thing, that thing is basically the Flow again—unless you create a new resource that represents the family. If you still want a distinct label, use a name that signals “collection/closure,” e.g.:

ArtifactFamily (best: clear, flexible)

ArtifactStack (good; implies layered)

ArtifactClosure (accurate but a bit mathy)

Avoid FourLayerArtifact: it bakes in today’s architecture and will become wrong as soon as you add a layer (e.g., “DistributionGroup”, “Mirror”, “Bundle”, etc.).
Avoid FlowArtifact: it reads like a synonym for Flow, so it won’t reduce confusion.

“A word for something abstract and concrete at the same time?”

In practice, systems name this as a Work and then treat concrete forms as manifestations (library science FRBR: Work/Expression/Manifestation/Item). If you want that vibe without importing the whole FRBR world:

Flow ≈ Work

Slice ≈ Expression (revision)

AbstractFile ≈ Manifestation (format-specific)

ArtifactLocatedFile ≈ Item (bytes at a location)

But I would not adopt “Work” unless you want that conceptual framing.

My call

Use Option 2: keep DigitalArtifact as the umbrella class (members across the stack), and use “artifact stack” as the prose phrase for the 4-layer structure. Don’t create a second term unless you find yourself repeatedly needing to refer to the whole closure as a first-class subject distinct from the Flow (which is unlikely).

If you want, I can propose one-sentence replacement wording for the DigitalArtifact line in the taxonomy that removes the “family rooted at a Flow” ambiguity—only if you explicitly ask to edit the draft.

Mesh Resource Taxonomy Draft
I don't like Option 2, I'm not looking for an umbrella class, at least I don't think. But maybe we should rename flow. It doesn't really capture "artifact over time". 
Thought for 31s

If you don’t want an umbrella class, then “Flow” must carry almost all of the conceptual weight: it is the identity-bearing artifact-over-time; Slice/AbstractFile/LocatedFile are its revision/representation/realization structure.

The problem is that the word Flow strongly suggests continuous streaming, while your model is discrete snapshots with optional working mutability. So you’re right: the label is semantically leaky.

What you actually mean by Flow

You mean something like:

“the continuing artifact identity across revisions”

“a revision series with a current working state and historical snapshots”

“the thing you cite when you mean ‘no particular version’”

That is much closer to Series or Lineage than “Flow”.

Renaming “Flow” is expensive

If you truly rename the class, you implicitly rename a lot of your surface vocabulary:

PayloadFlow, SupportingFlow

KnopMetadataFlow, NomenMetadataFlow, MeshMetadataFlow

hasSlice, WorkingSlice/HistoricalSlice semantics in prose

So if your goal is documentation clarity, the cheaper move is: keep the ontology term sflo:Flow but change the label/definition in docs.

Best candidates for a new name (if you do rename)

These are the only ones I think survive contact with real usage:

ArtifactSeries

Pros: says what it is; works for binary/text/RDF; doesn’t imply append-only.

Cons: “Series” is generic, but that’s probably fine.

RevisionSeries

Pros: extremely explicit.

Cons: verbose; “revision” may bias toward editorial workflows.

Lineage

Pros: provenance-friendly; fits your “weave” story.

Cons: implies derivation graphs, not necessarily linear versions (even if yours is linear).

Ledger / Record

Pros: implies history.

Cons: implies append-only and formal accounting; might mislead.

What I would not use: Stream, Timeline, Track. They read like time-series data, not “artifact identity with versions and representations.”

A clean compromise

Keep the class IRI sflo:Flow.

In docs, call it ArtifactSeries everywhere, and define:

“Flow (ArtifactSeries): the continuing artifact identity across revisions; has Slices; has a WorkingSlice and optional HistoricalSlices.”

This gives you clarity without a breaking rename.

How this interacts with your “DigitalArtifact family” question

If you treat Flow as the identity-bearing artifact, then you don’t need a special term for “abstract and concrete at the same time.” You just say:

“the Flow (ArtifactSeries) and its realizations”

or “the Flow’s revision stack”

Trying to coin a single noun for “the four-layer closure” tends to become either (a) an umbrella class you said you don’t want, or (b) a second identity object you don’t actually need.

If you want one short phrase for docs, I’d use artifact stack (prose term, not ontology class).

Mesh Resource Taxonomy Draft
Definitely don't want to call the class Flow and refer to it as something else. I don't think the problem is so much that Flows are discrete. (We know from video that lots of discrete things make a flowing thing). The problem is it doesn't capture what is flowing: a  digital artifact/work. "Slices" came from the metaphor of spacetime as a loaf, but loaf is very confusing. From philosophy, these are Endurants. Could do EnduringArtifact. A river is always changing but it's the same river. Maybe FlowArtifact? Let's get creative here.
Thought for 40s

You’re right about what’s missing from “Flow”: it doesn’t say what is flowing. In your system it’s “the continuing identity of a work-like digital artifact across revisions.” If you keep “Flow” as the base term, you’ll forever be adding prose to disambiguate.

Two problems to watch for (because they’ll bite you later):

Philosophy mismatch: your “slice” metaphor is 4D/temporal-part flavored (perdurant-ish), but your actual mechanics are “revision snapshots + working state,” which is closer to versioning than spacetime slicing. Calling the thing an Endurant while also calling its revisions “Slices” is conceptually noisy. You can keep the terms, but it will confuse new readers unless you explicitly say “slice = revision, not temporal part.”

Rename blast radius: PayloadFlow/SupportingFlow/etc. magnify whatever base term you choose. So the base term must be short and composable.

Better base names than Flow (short, composable)
Best “minimal change” fix

ArtifactFlow
Keeps your existing metaphor (“flow”) but states what flows. Your derived terms become:

PayloadArtifactFlow

SupportingArtifactFlow

KnopMetadataArtifactFlow (maybe keep KnopMetadataFlow as a named subtype anyway)

If you want to keep the suffix Flow for tooling familiarity, this is the cleanest correction.

Best “weaving-consistent” alternative

Thread / ArtifactThread
This aligns with your mesh/weave language. But “thread” has concurrency baggage (“thread” ≠ “artifact thread”), and “thread” as a base class makes derived names weird:

PayloadThread, SupportingThread (fine)

KnopMetadataThread (fine)
It’s actually workable, but you’ll need to defend it once: “thread = artifact identity over revisions.”

Best “semantic clarity” (but more sterile)

Lineage / ArtifactLineage
Communicates “continuing thing with a history.” Downside: implies branching ancestry graphs; you currently have a linear sequence (even if you might branch later).

Best “versioning-native”

RevisionSeries / ArtifactSeries
This is the most accurate. Downside: it de-emphasizes your flowing/weaving metaphor and can feel dry.

Names you suggested (and why they’re risky)

FlowArtifact: reads like “an artifact that flows” (fine) but is a bit tongue-twisting; also ambiguous whether it’s the series or the member.

EnduringArtifact / EndurantArtifact: conceptually loaded and jargon-y. If you use it, you are committing to ontological baggage and will need to explain endurant/perdurant repeatedly. It will not age well in docs aimed at builders.

FourLayerArtifact: bakes architecture into the name. The moment you add “mirrors,” “bundles,” “packs,” etc., the name becomes misleading.

Do you actually need a term beyond Flow?

If you rename Flow to something that already means “artifact identity over revisions” (ArtifactFlow, ArtifactSeries, ArtifactLineage), then you don’t need a separate “abstract and concrete at the same time” word. That “both” thing is just:

the artifact identity (the series), and

its revisions/representations/realizations as structure beneath it.

Trying to name “Flow + all its slices + all files” as a single thing tends to reintroduce an umbrella class (which you said you don’t want) or a second identity object (which you also don’t want).

My recommendation

If you want maximum continuity with your current vocabulary and minimum future explanation burden:

Rename Flow → ArtifactFlow (base class name, not just docs wording).

It fixes the exact defect you identified (“what is flowing?”) without forcing you into philosophical jargon or a brittle architecture-name.

If you want something more on-brand with mesh/weave and you can tolerate one-time explanation:

Rename Flow → Thread (or ArtifactThread if you want explicitness, but it’s longer).

If you want, I can propose a consistent renaming set (Flow/Slice/AbstractFile/LocatedFile) that preserves your existing paths and derived term names as much as possible.

Mesh Resource Taxonomy Draft
Maybe ArtifactFlow as the superclass, and then just PayloadFlow, NomenMetadataFlow, etc. (i.e., don't repeate "Artifact", replace it when the Artifact type is known). 
Thought for 34s

That naming scheme is workable, but only if you handle the one predictable confusion point:

People will see PayloadFlow, NomenMetadataFlow, etc. and assume there is a base class named Flow.

If you rename the base class to ArtifactFlow and remove Flow, you should either:

Keep Flow as a deprecated alias (e.g., sflo:Flow owl:equivalentClass sflo:ArtifactFlow + owl:deprecated true), or

Be willing to answer “where is Flow?” forever.

Given your constraint (“don’t call the class Flow”), (1) is the clean compromise: you don’t use it, but you keep the ecosystem understandable and backward-compatible.

Recommended pattern

Superclass: ArtifactFlow

Subclasses (keep the short names): PayloadFlow, SupportingFlow, NomenMetadataFlow, KnopMetadataFlow, MeshMetadataFlow

Documentation convention: “Any class ending with …Flow is a subclass of ArtifactFlow and denotes an artifact-over-revisions.”

This matches your “don’t repeat Artifact when the artifact type is known” principle and keeps names readable.

What I would not do

Rename subclasses to PayloadArtifactFlow, etc. That redundancy adds noise and doesn’t actually remove ambiguity.

Small SHACL / modeling implication

If artifactKind is on ArtifactFlow, then every …Flow subclass inherits it. You can then enforce:

PayloadFlow MUST have exactly one artifactKind

SupportingFlow SHOULD have one (or MUST if you want uniformity)

One more consistency check

If you adopt ArtifactFlow, you should decide whether DigitalArtifact stays at all. Right now you’re using it as “4-layer family rooted at a Flow.” If the root term becomes ArtifactFlow, then DigitalArtifact becomes either:

unnecessary (just talk about ArtifactFlow and its slices/files), or

a family concept (“artifact closure”) which is not a class you can sensibly attach to individual nodes without reintroducing the umbrella semantics you said you don’t want.

So renaming Flow tends to push you toward dropping DigitalArtifact or renaming it to a prose-only phrase like “artifact stack.”

If you want, I can propose an exact rename/deprecation map (classes + properties) that minimizes churn, but I won’t edit the taxonomy draft.

Mesh Resource Taxonomy Draft
I removed DigitalArtifact.   Let's just keep Flow as the Superclass. That explains itself as a superclass, whereas ArtifactFlow get buried with the other types of Flows.   I've renamed Core to Foundations.  
Thought for 52s
Mesh Resource Taxonomy Draft
Semantic Flow resource taxonomy (concise)
Foundations

SemanticFlowResource — Any resource participating in SF conventions.

Mesh — A servable filesystem region that contains SemanticFlowResources and, optionally, other supporting resources.

AbstractResource (SemanticFlowResource) — Slash IRI (/); dereferencing yields a resource page.

LocatedFile (SemanticFlowResource) — File IRI (typically with extension); denotes concrete bytes.

ArtifactLocatedFile (LocatedFile) — A concrete realization of an AbstractFile.

AssetFile (LocatedFile) — A non-artifact support file used by ResourcePages (images/css/js), conventionally under _assets/.

ResourcePageFile (LocatedFile; aka RenditionFile) — A derived UI/navigation file (e.g., index.html); not a primary payload artifact.

Naming

Nomen (AbstractResource) — A human-facing designator IRI (e.g., /people/alice/) intended to denote a discourse-worthy thing.

NomenHandle (AbstractResource) — The naming-object handle for a Nomen (e.g., /people/alice/_nomen/).

NomenMetadataFlow (SupportingFlow) — Metadata about a NomenHandle; may remain WorkingSlice-only until history is recorded.

Containers

Knop (AbstractResource) — SF container anchored at a stable in-mesh location; hosts payload and supporting flows.

KnopComponent (AbstractResource; MeshComponent) — Reserved components under a Knop (payload/supporting flows and their parts).

Artifacts and versioning

Flow (AbstractResource) — Artifact over revisions; has sflo:artifactKind.

Slice (AbstractResource) — A particular revision of the artifact. 

WorkingSlice — Mutable current state.

HistoricalSlice — Immutable published snapshot.

AbstractFile (AbstractResource) — A representation specification of a Slice (format/syntax/canonicalization/bundling).

DatasetSeries (AbstractResource) — A series/partitioning of related dataset artifacts.

RDF payload types

RdfDataset — RDF dataset content: default graph + 0..n named graphs; default graph may be empty. Typically realized as a Slice; also used for standalone datasets such as inventories.

RdfGraph — RDF graph content: exactly one graph (default graph only; may be empty).

NamedGraph — A graph identified by a graph-name IRI (may be treated as identity-over-time or snapshot by which artifact you denote).

Supporting flows

SupportingFlow (Flow) — A non-payload flow used for metadata/inventory/auxiliary artifacts.

PayloadFlow (Flow) — The primary digital-artifact flow hosted by a Knop.

KnopMetadataFlow (SupportingFlow) — Small, discoverable: structure/pointers/summary inventory for the Knop.

MeshMetadataFlow (SupportingFlow; MeshComponent) — Optional mesh-level operational metadata/config.

Operational indices and logs

MeshInventoryDataset (RdfDataset; non-flow; MeshComponent) — Optional derived index for explorability.

KnopInventoryDataset (RdfDataset; non-flow; KnopComponent) — Optional derived index (may be large/sharded).

OperationalLogFile (LocatedFile; MeshComponent) — Plain tool logs (not discourse-worthy by default).

Reserved/system resources

MeshComponent (AbstractResource) — Reserved underscore-path/system resources (e.g., _mesh/, _knops/, _assets/, inventories, configs).

Asset folders (convention) — /_mesh/_assets/… (mesh shared) and <knop>/_assets/… (per Knop). Templates are Knops, so template assets live in their Knop _assets/.

Applied the updates you specified:

Removed all remaining DigitalArtifact references from the draft.

Kept Flow as the superclass (no ArtifactFlow).

Kept your Foundations rename.

Removed the stray marker on SemanticFlowResource.

Fixed the Mesh line punctuation and cleaned up the RdfDataset entry (extra ) and clarified wording).

Left ArtifactLocatedFile in place, but it no longer depends on DigitalArtifact.