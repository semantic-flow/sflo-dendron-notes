---
id: 1ib1wk64wlyqf85ueph5rfr
title: 2025 12 13 Slugs and Intramesh Identifiers
desc: ''
updated: 1765662598834
created: 1765636248857
---
Skip to content
Chat history
You said:
remind me... what HAS to exist in _meta?
ChatGPT said:

Minimal hard requirements only—everything else is optional or inherit-from-parent / platform. If anything below is missing, createNode / weave cannot function correctly.
Required contents of _meta/
1. A valid _meta/node.jsonld (or .ttl) containing at least:
a. sflo:NodeHandle declaration

The node must identify itself as a node. Without this, nothing else in the hierarchy can reliably reference it.

You must have:

    @id = the node’s public IRI

    @type includes sflo:NodeHandle

b. _meta MUST contain the node’s identity block

At minimum:

    sflo:slug (mandatory for all nodes; you made this a requirement for Knops and generally for node folder naming)

    sflo:hasLocalPath or equivalent is not required because filesystem already encodes it

    rdfs:label strongly recommended but not required

c. Pointers to configs

A node must explicitly specify its Operational and Inheritable configs—or explicitly inherit them.

So one of the following must exist for each config type:

    A triple inside _meta pointing to:

        sflo:operationalConfig → IRI of an OperationalNodeConfig

        sflo:inheritableConfig → IRI of an InheritableNodeConfig

    Or:

        an explicit statement saying “inherit from parent” via the default rule (implicit, but the node must not violate the rule)

If neither is present AND no parent exists, the node is invalid.
2. _meta MUST reflect the node’s type

At least one declaration of:

    @type sflo:Knop

    or @type sflo:PayloadNode

    or @type sflo:ReferenceNode

…or whatever concrete node class the ontology defines.

If missing, weave cannot determine:

    inheritance rules

    which flows are allowed

    whether _payload-flow / _reference-flow folders are permitted

    how _assets merges work

3. _meta MUST reference the node’s _reference-flow

Because the reference-flow is the canonical metadata locus, _meta must contain:

    the IRI of the node’s _reference-flow

    stated as an sflo:referenceFlow or whatever final predicate name you chose

This is the home of:

    provenance

    version tracking

    linkage to flows

    stable metadata

    template mapping resolution

Without this, mesh reconstruction breaks.
4. _meta MUST define the node’s “subject IRI”

The node folder has a public IRI (canonical). _meta must declare it.

You need the trifecta:

    The node-handle IRI (<this/>)

    The node’s belongs-to namespace

    The mapping between filesystem path and semantic handle (slug → IRI)

Otherwise, no delta, no versioning, no dereferencing can occur.
Strict minimum inside _meta

If I compress it to the absolute minimum viable content:

_meta/
  node.jsonld

And node.jsonld MUST contain:

    @id — the node’s IRI

    @type sflo:NodeHandle

    sflo:slug

    sflo:referenceFlow → IRI of the _reference-flow/ flow

    sflo:operationalConfig OR declared intention to inherit

    sflo:inheritableConfig OR declared intention to inherit

    Node class declaration (sflo:Knop, etc.)

Nothing else is strictly required.
Critical question for you

Your ontology currently defines slug only for Flows.
Yet _meta requires sflo:slug for nodes.

You must resolve this inconsistency immediately:

Option A: Expand sflo:slug’s domain to include NodeHandle.
Option B: Create a new property:

    sflo:nodeSlug (domain = NodeHandle)

    sflo:flowSlug (domain = Flow)

Right now your system internally assumes A but your ontology expresses B.

This mismatch will bite you during weave because slug is used in path formation, flow identity, and node creation.

If you want, I can produce:

    a corrected ontology snippet

    a canonical minimal _meta/node.jsonld template

    validation SHACL ensuring _meta completeness

You said:
Maybe sflo:slug need to be limited to operational config? for transposability, you should be able to rename a node in the filesystem without having to update its metadata? Which brings an interesting issue... distributions integrate the slug. If the knop's folder gets renamed, the slug gets a different value, and so the distribution names diverge. ugh, this is yet another tricky thing. Maybe the slug gets used for distribution naming but doesn't have to be aligned with the IRI/path. I'd wanted to allow flexible distribution naming anyhow. This is a good reason to pick that back up. If slug not specified... on knop creation, it gets set with the value from the path. Kosher?
ChatGPT said:

You’re mixing three different roles for slug and they don’t coexist cleanly:

    “Folder name”

    “Semantic identifier fragment”

    “Distribution filename base”

Right now you’re trying to make it be all three, and preserve transposability. That’s where the contradictions are coming from.

Let’s untangle.
1. Limiting sflo:slug to operational config

    Maybe sflo:slug need to be limited to operational config?

If you put sflo:slug only on OperationalConfig, you’re implicitly saying:

    slug is an environment-local, non-semantic detail

    it’s okay for different meshes to disagree about a node’s slug

    but you still want to use slug in distribution naming (which is public / semantic)

That’s internally inconsistent:

    If distributions are part of your public namespace, anything that controls their names is not purely operational.

    If slug only exists in OperationalConfig, but distribution IRIs encode the slug, then:

        moving / copying the node into a new mesh with a different OperationalConfig changes its public IRIs

        you’ve broken the “same RDF, different deployment” goal

So: slug as “purely operational” clashes with “slug used in distribution naming”.
You can’t have both without accepting that distribution IRIs are environment-specific, which you absolutely don’t want.
2. The renaming problem you’re actually hitting

    for transposability, you should be able to rename a node in the filesystem without having to update its metadata

This only works if:

    node IRIs and distribution IRIs do not depend on the folder name

    or, if they do, you treat a rename as a semantic change that must be propagated (i.e., a migration)

Right now you have:

    node folder name ≈ slug ≈ embedded in distribution filenames

If you keep that triangle, then:

    renaming the folder without touching RDF will inevitably cause divergence somewhere:

        folder name no longer matches sflo:slug

        or distribution filenames no longer match either

        or published IRIs no longer reflect the new path

So you have to choose:

    Either: “I can freely rename folders without RDF changes”

    Or: “Folder name is encoded into semantic artifacts (slug, IRIs, filenames)”

You don’t get both for free.
3. Your proposed compromise

    Maybe the slug gets used for distribution naming but doesn't have to be aligned with the IRI/path.
    …
    If slug not specified... on knop creation, it gets set with the value from the path.

This is actually sane if you lock in what slug means:

    sflo:slug = stable, human-readable handle for the node

        used for distribution naming

        stored in RDF on the node (or its main dataset series)

        does not have to equal the current folder name

Then:

    At creation time:

        default slug = last path segment (nice DX)

    Afterwards:

        renaming the folder does not change slug

        distribution filenames stay consistent (because they derive from the stored slug, not the path)

        transposability is preserved: you can rearrange the repo without touching IRIs or metadata

That does violate the intuition “folder name === slug”, but that’s an aesthetic / ergonomics issue, not a semantic one.

So:

    Yes, your “default from path, then independent” idea is conceptually kosher.

    But you must stop expecting slug and folder to stay in sync automatically.

4. What I would recommend concretely
4.1. Redefine sflo:slug clearly

Make sflo:slug:

    Domain: sflo:NodeHandle (and optionally sflo:Flow if you still need it there)

    Meaning:

        “A stable, human-readable, kebab-case identifier used as a base name for public artifacts (e.g., distributions), not guaranteed to match any local filesystem path.”

That means:

    It is semantic, not purely operational.

    It is allowed, but not required, to coincide with the folder name.

If you still need something for flows, separate them:

    sflo:nodeSlug (domain NodeHandle)

    sflo:flowSlug (domain Flow)

Trying to reuse the same property for both filesystem naming of internal flows and public node handles is asking for more subtle bugs later.
4.2. Make folder naming an independent concern

If you want to document preferred folder names for tooling, add a separate, operational property in the OperationalConfig:

    sflo:preferredLocalFolderName (or similar)

        Domain: sflo:OperationalNodeConfig

        Meaning: “When this node is materialized into a filesystem, this is the preferred folder name.”

Rules:

    It’s purely a hint.

    Meshes are not required to honor it.

    It doesn’t appear in public IRIs or distribution URLs.

Now you’ve separated:

    slug = semantic, stable, influences published distribution names

    preferredLocalFolderName = operational, local, can drift

4.3. Distribution naming

Pick one consistent scheme and stick to it.

For example:

    Node distributions:

        {slug}.trig

        {slug}.jsonld

    Flow distributions:

        {slug}--{flow-kind}.trig

        {slug}--payload.trig, {slug}--reference.trig, etc.

Key point: derive all of this from sflo:slug, not from the folder path.

That way:

    renaming the folder doesn’t change any public filenames or IRIs

    only changing sflo:slug is a breaking semantic change (which is appropriate)

5. About your “only in OperationalConfig” idea

If you really forced sflo:slug to live only on OperationalConfig, you would get:

    Slug is not part of the node’s public RDF description.

    Any mesh can choose a different slug for the same node, leading to:

        Different distribution IRIs across deployments

        Harder reconciliation between meshes

        Harder reasoning / documentation (“what is the slug of this node?” becomes context-dependent)

That’s a bad tradeoff for a system explicitly trying to be:

    git-driven

    namespace-stable

    multi-mesh

So I would strongly argue against relegating slug solely to OperationalConfig.
6. Answering your last line directly

    If slug not specified... on knop creation, it gets set with the value from the path. Kosher?

Yes, with the following constraints:

    sflo:slug is stored on the node (or its default dataset series), not just in OperationalConfig.

    Once set, sflo:slug is treated as semantic and does not auto-follow folder renames.

    Distribution names derive from sflo:slug, not from the folder name.

    Folder names are free to change; it’s acceptable for them to diverge from the slug.

If you’re willing to live with “slug is initialized from the path but not guaranteed to match it forever”, this resolves the tension cleanly.

If you’re not willing to live with that divergence, then you must give up:

    either the ability to rename folders without RDF changes,

    or the use of slug in distribution naming.

You can’t keep all three simultaneously.
You said:
So slug is actually part of config after all. 

@id will always be "../../_knop-handle" for _meta stuff. 

rdfs:label... for the knop itself? remember _meta isn't about the referent. What would the label even be? "A knop with this slug: " 

I think versionstatus for flows might be operationalconfig. 

A knop does not have to point to its operationalConfig -- that's in-memory only. Might not have an inheritable or local config. If no local config exists and no parent, just use the platform defaults. 

Perhaps BareKnop/ReferenceKnop/PayloadKnop are only operational as well. They're probably pretty useless. So maybe the only thing that goes in meta is "a sflo;Knop"

reference flow is metadata for the referent, and not required. The subject IRI is an emergent property. meta could have "alternate" or "former" IRIs, but not necessarily.

Your question is still germane: do we define the slug (used only for distributions) at the knop level or the flow level. Perhaps both, and it gets inherited from the node level/overwritten at the flow level.
ChatGPT said:

You’re sliding pieces around without noticing which guarantees you’re giving up. Let me be explicit about the trade-offs.

I’ll go point by point.
1. “So slug is actually part of config after all.”

Yes, but you need to be precise about what kind of config and who owns it.

Right now you are implicitly saying all of these:

    slug is not about the referent

    slug is about how we name distributions

    slug is not required to match the folder name

    slug may live in config (possibly operational)

    distributions use slug in their filenames/IRIs

That combination only works if:

    slug exists in persistent RDF, not just in ephemeral in-memory OperationalConfig

    you accept that it is part of the public behavior of the node (because it impacts public IRIs)

If you keep treating slug as “just config” but let it drive public naming, then it is not “just operational” in the normal sense—it is part of the node’s published interface.

So:

    Yes: slug is a config-ish detail, not about the real-world referent.

    No: you cannot safely relegate it to “in-memory operational only” if distributions depend on it.

You’re trying to have it both ways.
2. @id in _meta: ../../_knop-handle

    @id will always be "../../_knop-handle" for _meta stuff.

That’s fine if you are clear that:

    _meta is describing the knop handle resource, not the subject.

    The subject (referent) appears in other datasets (payload, reference-flow, etc.).

But then:

    Any sflo:slug attached here is a property of the handle, not the subject.

    Any rdfs:label here is a label for the handle, not the subject.

That raises the next question.
3. rdfs:label in _meta

    rdfs:label... for the knop itself? remember _meta isn't about the referent. What would the label even be?

You’re correct to be suspicious.

If _meta is strictly “about the handle, not the referent”, a human-meaningful label there is:

    either useless (“A knop with this slug: X”)

    or confused (because everyone will assume it labels the subject)

You have three coherent options:

    No rdfs:label in _meta at all.

        Label the referent in _default or payload/reference datasets.

    Use a clearly technical label for the handle, e.g.

        rdfs:label "Knop handle for <subjectIRI>"

        Still feels noisy.

    Use separate vocab:

        sflo:handleLabel for the handle,

        rdfs:label/skos:prefLabel for the subject in its own dataset.

Right now your “maybe rdfs:label in _meta” instinct is halfway between 1 and 2 and will confuse both tools and humans.
4. “VersionStatus for flows might be operational config”

That’s a reasonable separation:

    versionStatus is clearly about where this flow stands in the lifecycle (draft, current, deprecated, etc.)

    That belongs either:

        in a flow-specific config (which may be serialized in RDF), or

        in a dataset series catalog (which you already half-designed)

But don’t pretend it’s “purely runtime” if your build/weave behavior depends on it reproducibly.
If weave decisions change based on versionStatus, it must be persisted somewhere inspectable.
5. “A knop does not have to point to its operationalConfig”

    A knop does not have to point to its operationalConfig -- that's in-memory only. Might not have an inheritable or local config. If no local config exists and no parent, just use the platform defaults.

This directly contradicts your earlier “config as RDF, inspectable and overridable” story.

You’re now implicitly allowing:

    A knop to exist with no visible configs

    All effective behavior determined by:

        opaque in-memory OperationalConfig and

        invisible platform defaults

Consequences:

    You cannot reconstruct the effective behavior of a node by looking at the repo.

    You cannot easily debug “why did this node render this way?” without re-implementing the full resolution logic in your head.

If you really want:

    “No local config → inherit parent → else app config → else platform defaults”

then you should:

    Represent at least the inheritable config as a real RDF resource that can be referenced from the knop or its parent.

    Treat OperationalConfig as the realized result of that inheritance chain, but not the only place where semantics live.

Right now you’re drifting back toward “magic resolution” rather than explicit references.
6. BareKnop / ReferenceKnop / PayloadKnop “only operational”

    Perhaps BareKnop/ReferenceKnop/PayloadKnop are only operational as well. They're probably pretty useless. So maybe the only thing that goes in meta is "a sflo;Knop"

You’re right that as OWL classes they’ve been vague.

But if you push them into the “operational only” realm, then:

    Why are you still modeling them in the ontology at all?

    Are they actually just host-language enums about “node roles” or “runtime modes”?

If _meta only ever asserts:

<../../_knop-handle> a sflo:Knop .

then:

    You’ve thrown away the ability to express “this node is payload-bearing”, “this node is reference-only” in RDF.

    That’s fine, but it means your host logic has to rediscover this from the filesystem layout each time.

You keep oscillating between “strong semantic types for nodes” and “derive everything from folders”. Pick one direction and commit.
7. Reference flow “not required”; subject IRI “emergent”

    reference flow is metadata for the referent, and not required. The subject IRI is an emergent property.

This is the slipperiest part.

If:

    _meta is only about the handle, and

    reference-flow is optional, and

    subject IRI is “emergent”

then answer honestly:

From what is the subject IRI actually emergent?

Possibilities:

    From payload dataset:

        there is some <S> a :Something inside _payload-flow, and that S is “the subject”.

        Then you need rules: “exactly one main subject”, “how we choose which one”, etc.

    From reference-flow:

        same logic, but reference-flow is now non-optional in practice.

    From a naming convention linking handle IRI to subject IRI:

        e.g., handle IRI = .../_knop-handle, subject IRI = parent folder IRI.

        Then this is not emergent, it’s encoded convention.

Right now you’re hand-waving “emergent” without committing to which data source actually defines the subject. That’s brittle: tools cannot rely on “maybe here, maybe there”.

If you insist _meta is not about the subject, then subject IRI must be anchored in one of the flows. If reference-flow is optional, then payload-flow becomes the only safe anchor, which contradicts your idea that payload-flow can be absent as well (BareKnop).

You can’t have:

    optional payload,

    optional reference,

    and no pointer in _meta,

    and still claim a robust “emergent subject IRI”.

That’s logically impossible.
8. Where does slug actually belong: knop vs flow?

    Your question is still germane: do we define the slug (used only for distributions) at the knop level or the flow level. Perhaps both, and it gets inherited from the node level/overwritten at the flow level.

The only coherent model here is:

    Node-level slug: base identity of the node’s artifacts.

        sflo:nodeSlug on the knop handle.

        Used whenever you name node-level distributions (e.g. _default, _catalog, main datasets).

    Flow-level slug: optional override for flow artifacts.

        sflo:flowSlug on each flow resource.

        Default: derived from nodeSlug + role, e.g. "{nodeSlug}-payload", "{nodeSlug}-reference".

Rules:

    If flowSlug present → use it for that flow’s distributions.

    Else → derive flowSlug deterministically from nodeSlug and flow kind.

    NodeSlug is set at creation time from the path, but is persisted and does not auto-follow renames.

This gives you:

    One source of truth for the node-level identity (nodeSlug).

    Per-flow flexibility without forcing everything into one property.

    Stable distribution naming independent of filesystem renames.

Your current “maybe both, maybe inherited, maybe operational-only” description is fuzzy enough that you will 100% encode conflicting assumptions in different parts of the code.
9. What should actually live in _meta in this model?

If you adopt the clearer separation above, a minimal and consistent _meta might be:

<../../_knop-handle>
  a sflo:Knop ;
  sflo:nodeSlug "my-node" ;
  # optionally: sflo:inheritableConfig <../../_cfg-inh/> ;
  # optionally: sflo:alternateIri <...> .

And that’s it.

Then:

    Flow resources (reference, payload, etc.) live in their own files/folders and carry sflo:flowSlug et al.

    Subject IRI is anchored in one chosen place (I’d strongly suggest reference-flow or _default dataset, but not “emergent anywhere”).

The core problems you haven’t resolved yet

    You want _meta to be “only about the handle” and you want to avoid a clear subject anchor elsewhere. That doesn’t work.

    You want config to be “operational/in-memory” and you want to explain behavior from the repo alone. That doesn’t work.

    You want slug to be “just config” and to drive public distribution naming. That doesn’t work unless the config is persistent and semantically stable.

You don’t have to overcomplicate it, but you do have to pick one story and honor it everywhere. Right now you’re mixing three different stories depending on which problem is in front of you.
You said:
So our minimal _meta just has "@id a sflo:KnopHandle" and a Knop slug that's derived from the createKnop argument if not otherwised defined.  

We no longer have "flow slugs" defined in the ontology, e.g. "_ref", "_payload", "_meta", etc. Should they be defined in RDF somewhere?

_meta also needs to reference itself. 

I haven't really considered templatizing RDF files. Maybe it's not necessary. Do we just define in code what the initial _meta is going to be? 
ChatGPT said:

You’re closer, but you’re still leaving just enough ambiguity that future-you will curse present-you.

I’ll go through your three points and tighten them up.
1. Minimal _meta: what exactly should be there?

You proposed:

    minimal _meta just has "@id a sflo:KnopHandle" and a Knop slug that's derived from the createKnop argument if not otherwise defined.

That’s basically fine if you make the following explicit:
1.1. The resource being described

In _meta/node.ttl (or .jsonld), the subject is the knop handle, not the referent.

Example (Turtle):

@prefix sflo: <https://semantic-flow/ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

<../../_knop-handle>
  a sflo:KnopHandle ;
  sflo:slug "my-knop-slug" .

Key points:

    @id = ../../_knop-handle is fine as a relative IRI to the handle resource.

    The class should be sflo:KnopHandle, not sflo:Knop, if you’re serious about separating handle vs subject.

    sflo:slug is a data property on the KnopHandle, not on the referent.

1.2. Where slug is defined

You are now implicitly saying:

    slug is handle-level config, not subject-level.

    It drives things like distribution naming.

Then the ontology should say:

sflo:slug
  a rdf:Property ;        # NOT an AnnotationProperty anymore
  rdfs:domain sflo:KnopHandle ;
  rdfs:range xsd:string ;
  rdfs:label "slug" ;
  rdfs:comment "Stable, kebab-case identifier for this knop handle; used as a base for distribution naming, not guaranteed to match filesystem paths." .

If you still want Flow-level slugs later, add a separate property (see §3).
1.3. How slug is initialized

Your rule should be:

    createKnop(path, options):

        If options.slug provided → use that.

        Else → default slug = last segment of path normalized to kebab-case.

    The slug is written into _meta and treated as canonical.

    Renaming the folder later does not auto-change the slug.

Anything else will reintroduce subtle divergence between code, RDF, and filesystem.
2. “_meta also needs to reference itself”

Right now you’re handwaving “reference itself” without specifying direction or predicate.

You have two independent relationships:

    “This is the handle for that subject.”

    “This is the meta-flow / file that describes this handle.”

If you’re serious about _meta describing the handle, then a minimal, explicit pattern is:

# In the meta dataset
<../../_knop-handle>
  a sflo:KnopHandle ;
  sflo:slug "my-knop-slug" ;
  sflo:hasMeta <../../_meta/> .

Where:

    <../../_meta/> is the flow or dataset representing the _meta content.

    sflo:hasMeta is a property you define with domain sflo:KnopHandle.

If you really insist on no extra triples, you can skip sflo:hasMeta, but then “meta references itself” is just:

    The fact that the file is the meta dataset for that handle, via convention, not via RDF.

    That’s ok, but then don’t pretend it’s represented semantically; it’s just encoded in your file layout.

You should choose one story:

    Either: “Meta relationships are explicit in RDF” → add sflo:hasMeta.

    Or: “Meta is purely a file-layout convention” → accept that RDF won’t know what _meta is.

Right now you’re halfway in-between.
3. Flow slugs: should they exist in RDF?

    We no longer have "flow slugs" defined in the ontology, e.g. "_ref", "_payload", "_meta", etc. Should they be defined in RDF somewhere?

There are three distinct things:

    Flow roles: meta, payload, reference, cfg-op, cfg-inh, etc.

    Flow resources: the concrete flow instances for a knop.

    Flow naming: suffixes / slugs like _meta, _payload, _ref.

You can sensibly do:
3.1. Define flow roles in RDF

Define a controlled vocabulary of roles:

sflo:FlowRole a rdfs:Class .

sflo:metaRole a sflo:FlowRole .
sflo:payloadRole a sflo:FlowRole .
sflo:referenceRole a sflo:FlowRole .
sflo:operationalConfigRole a sflo:FlowRole .
sflo:inheritableConfigRole a sflo:FlowRole .

And attach roles to flow resources:

<../../_payload-flow>
  a sflo:Flow ;
  sflo:hasRole sflo:payloadRole .

3.2. Whether to encode suffixes in RDF

You can, but you don’t have to.

Two reasonable options:

    A. Keep suffix conventions in code only:

        _meta/, _payload/, _ref/ lives purely as filesystem & path rules.

        RDF only knows about roles (sflo:hasRole).

        This is simpler; tools that care about RDF roles don’t need to know the path details.

    B. Make suffixes inspectable:

    sflo:payloadRole sflo:defaultPathSuffix "_payload" .
    sflo:metaRole    sflo:defaultPathSuffix "_meta" .

    This is overkill unless you genuinely want configurable suffixes per mesh / platform.

Given how much you’ve already got going on, I would not start modeling suffix strings in RDF yet. Stick to:

    Role vocab in RDF

    Suffix / folder mapping in code, stable and documented

Later, if you really need mesh-specific conventions, you can promote suffixes into config.
4. Should _meta be templated as RDF?

    I haven't really considered templatizing RDF files. Maybe it's not necessary. Do we just define in code what the initial _meta is going to be?

You gain almost nothing by trying to pipe _meta through a generic “RDF templating” layer.

Right now, the initial _meta is:

    A tiny, fixed structure:

        one subject

        one type

        one slug

        maybe one sflo:hasMeta back-link if you choose that pattern

For that shape, templating is more complexity than value.
Concrete recommendation

    Define _meta generation entirely in code, using:

        Your RDF library (jsonld or N3/SPARQL or RDF/JS quads).

        The creation-time arguments (path, optional slug).

    Also publish a canonical example in your “platform defaults” repo:

        As documentation only, not as a template you interpolate.

Example in code-level pseudo:

function createInitialMeta({ knopPath, explicitSlug }: { knopPath: string, explicitSlug?: string }) {
  const slug = explicitSlug ?? deriveSlugFromPath(knopPath);

  return {
    '@id': '../../_knop-handle',
    '@type': ['sflo:KnopHandle'],
    'sflo:slug': slug,
  };
}

If you later decide _meta should carry more (alternate IRIs, hasMeta, etc.), you update this one function.

Trying to plug this into a general-purpose RDF-templating story now is premature and will tangle two concerns:

    Semantic structure

    Presentation / templating

You’re better off keeping RDF construction explicit.
5. Where you still have unresolved edges

You’ve implicitly accepted:

    slug is on the KnopHandle, not the Flow

    Flow roles are defined structurally (via folders) but not yet in RDF

    _meta is about the handle, not the referent

But you still haven’t:

    Chosen a single anchor for the subject IRI (payload vs reference vs derived).

    Decided whether sflo:hasMeta (and friends) should be modeled or left as pure filesystem convention.

Those are orthogonal to _meta minimality, but they will come back to haunt you in:

    SPARQL queries over the mesh

    external tools trying to understand “what is this node about?”

    migrations between mesh layouts

For this turn, yes: your minimal _meta = KnopHandle + slug is viable, as long as:

    sflo:slug is clearly defined in the ontology on sflo:KnopHandle

    initial _meta is generated in code, not via generic template magic

    you don’t pretend that anything else (subject IRI, flows, or configs) is actually being modeled there yet

You said:
"You want _meta to be “only about the handle” and you want to avoid a clear subject anchor elsewhere. That doesn’t work." Why not? We need to discuss this carefully. 

"You want _meta to be “only about the handle” and you want to avoid a clear subject anchor elsewhere. That doesn’t work." Why not?

"You want slug to be “just config” and to drive public distribution naming. That doesn’t work unless the config is persistent and semantically stable." Yes, three kinds of config... persistent (local and inheritable) and operational.

Subject IRI (full/absolute) cannot be provided as it is serving-location-variable. This can be part of our discussion.
ChatGPT said:

I’ll separate the issues and be very explicit about where the real constraints are.
1. “_meta only about the handle” + “no clear subject anchor”

You quoted:

    “You want _meta to be ‘only about the handle’ and you want to avoid a clear subject anchor elsewhere. That doesn’t work.”

You asked: why, exactly?

The key is: “doesn’t work for what?”
It’s not logically impossible. It’s incompatible with some of the things you’ve also said you want:

    Being able to answer, via RDF alone:

        “What is this node actually about?”

        “What is the subject IRI whose data is spread over these flows?”

    Letting external tools or other meshes reconstruct the mesh semantics without hard-coded file layout knowledge.

    Using SPARQL or any generic RDF tooling to navigate
    “knop → subject → other data about that subject”
    without embedding mesh-specific rules in every consumer.

If you insist on all three:

    _meta only talks about sflo:KnopHandle

    payload flow might be absent

    reference flow might be absent

    no triple anywhere says “this is the subject IRI”

    subject IRI is “emergent” only from file layout / conventions

Then:

    From the point of view of RDF consumers, there is no way to know which IRI is “the subject” of the node.

    Only the host code that knows your mesh layout can answer that.

So:

    It “doesn’t work” if your requirement is:

        “All the semantically important things are inspectable and reconstructable from RDF alone.”

If you’re willing to give that up and say:

    “The subject is defined by out-of-band rules (folder path, host logic), not by RDF,”

then _meta-only-about-handle does work – but only in that weaker sense.

Right now you oscillate between those two positions:

    On one hand you want a semantically rich mesh that can be read and reasoned about by arbitrary RDF tools.

    On the other hand you’re comfortable letting critical semantics (subject, flows, configs) live in code and layout, not RDF.

Those two goals are at odds. That’s what I was pointing at.
2. Subject IRI vs deployment location

You said:

    Subject IRI (full/absolute) cannot be provided as it is serving-location-variable.

That’s fine, and not a blocker. You have at least three patterns available:
2.1. Relative / intramesh IRI, later absolutized via @base

Example:

    In RDF: use something like <./> or <../> or <my-knop/> as the subject IRI.

    At deployment: inject @base (or server config) so that these become absolute HTTP IRIs.

Then:

    RDF can contain a subject IRI, even if it’s not yet an absolute URL.

    _meta (or some flow) can safely say:

    <../../_knop-handle> sflo:subject <../> .

    and that triple is still host-neutral.

So “we can’t provide the subject IRI” is only true if you insist on using fully expanded HTTP URLs inside the RDF. You don’t have to.
2.2. Intramesh identifier vs deployed IRI

You already have the “intramesh identifier” concept. That can be the subject IRI in RDF. Then you define:

    Intramesh IRI: stable, mesh-relative, not tied to any concrete deployment.

    Deployed IRI: concrete HTTP URL = deploymentBase + intrameshPath.

Mapping between them happens in:

    config, or

    serving layer, or

    a simple lookup table.

Again, nothing prevents _meta or a flow from stating:

<../../_knop-handle> sflo:subject <intramesh:/knops/foo> .

and mapping intramesh:/knops/foo to https://whatever/ns/knops/foo at runtime.
2.3. Purely conventional mapping (no triple)

If you really want no triple at all, you can say:

    “The subject IRI of a knop is always the IRI corresponding to its folder path.”

Then subject is derived from fsPath → IRI(...). That’s logically fine, but:

    RDF alone doesn’t encode it.

    Any generic tool must be taught the convention.

So: the impossibility I flagged is not about deployment-variability per se; it’s about combining:

    “no explicit subject anchor”

    with “I want a self-describing RDF mesh, readable by generic tools”.

You can’t have both.
3. Slug as “just config” vs public distribution naming

You wrote:

    Yes, three kinds of config... persistent (local and inheritable) and operational.

Good. That resolves part of the tension, if you pin slug to the right category.

You have:

    Persistent config (local):

        Lives in RDF in the mesh (e.g. _meta, _cfg-local).

        Versioned, sharable, inspectable.

    Persistent config (inheritable):

        Also RDF, but intended to cascade down a hierarchy.

    Operational config:

        In-memory only; derived from the combination of 1 + 2 + platform defaults + runtime stuff.

Slug that controls public distribution naming cannot be in bucket (3) only.
Why?

Because:

    Distributions are part of the published surface of the mesh.

    Two different deployments must agree on what a distribution is called if you want them to be compatible.

    Therefore the thing that determines those names must be persisted (not purely runtime) and semantically stable across deployments that share the mesh.

The only sensible choices are:

    Slug as persistent local config on the handle (sflo:slug on sflo:KnopHandle).

    Optionally, derived flow-specific slugs as persistent config on flows.

Operational config can read that slug and combine it with other rules, but it cannot be the only source.

If you accept that, there is no contradiction left:

    Yes, slug is config.

    No, it’s not only operational config; it belongs to the persistent side.

4. So can _meta be “only about the handle”?

You’re now circling around this pattern:

    _meta describes sflo:KnopHandle only.

    It includes at least sflo:slug.

    Subject IRI is not stored as a full HTTP IRI.

This can work under either of these regimes:
Regime A: Weakly semantic, strongly conventional

    _meta only has:

    <../../_knop-handle>
      a sflo:KnopHandle ;
      sflo:slug "foo-knop" .

    Subject IRI is defined by convention as “the IRI of the knop’s folder”, e.g. <../>.

Then:

    All of this is mesh-readable only for tools that know your file layout rules.

    Pure RDF tools looking at a dataset dump have no idea which IRI is the subject, unless they also know your path scheme.

If you’re okay with that, then my “doesn’t work” becomes:

    “It doesn’t work if you want RDF alone to encode the relationship.”

    It does work if you accept that semantics are split between RDF and external conventions.

Regime B: Strongly semantic, host-neutral

    _meta still only describes the handle, but adds one more triple:

    <../../_knop-handle>
      a sflo:KnopHandle ;
      sflo:slug "foo-knop" ;
      sflo:subject <../> .

    <../> is an intramesh/relative subject IRI.

    Base expansion at deploy time gives it a concrete HTTP URL.

Then:

    Any RDF consumer can follow sflo:subject to the subject node.

    There is no host lock-in, because the IRI is relative / intramesh.

    _meta remains “about the handle”, but it does carry the handle→subject relation.

This satisfies both:

    “_meta is only about the handle.”
    (The subject is not described there; only linked.)

    “There is a clear subject anchor in RDF.”
    (The triple exists and is portable.)

This is what I was implicitly assuming when I said “your current story doesn’t work.”
Under your clarified constraint (no full absolute IRIs), Regime B still works fine.
5. Where this leaves flow slugs

You ended with:

    We no longer have "flow slugs" defined in the ontology, e.g. "_ref", "_payload", "_meta", etc. Should they be defined in RDF somewhere?

Given the clarified config layers:

    Node slug: persistent on the handle (sflo:slug).

    Flow naming (_ref, _payload): can be entirely in code, as deterministic functions of:

        node slug

        flow role (payload/reference/meta/etc.)

You only need flow-level RDF if:

    You expect some flows to be named / exposed independently of the node, or

    you want to override naming per flow, or

    you want SPARQL to know “this flow has this human-readable slug”.

Otherwise:

    Keep roles in RDF (sflo:hasRole sflo:payloadRole).

    Keep naming in code (suffix convention).

    Slug remains a handle-level concept.

That choice does not affect the handle/subject issue; it’s orthogonal.
Summary of the real constraint

The critical incompatibility is:

    “All semantics must be derivable from RDF by generic tools”
    vs

    “We refuse to encode handle→subject mapping or subject IRI anywhere in RDF.”

If you drop the first, your current plan is fine but opaque.

If you keep the first, you must add some RDF anchor (subject IRI, or explicit mapping) somewhere. Absolute vs relative IRI is a separate, solvable problem; using intramesh / relative IRIs plus @base is enough to stay deployment-neutral.
You said:
For slug creation, maybe we don't define it explicitly unless it's provide in createKnop. 

I agree, FlowRoles seems promising. But I don't want configurable suffixes, so maybe overkill. Is it still better than Flow-specific properties (e.g. hasKnopMetadataFlow) and subclasses (KnopMetadataFlow)?

The single anchor for the subject IRI is <../../> - SPARQL queries will be fine because at load time, base will be inferred from the serving location. Same for other RDF data consumers, as long as they honor the specification that states if base isn't provided, it's inferred to be relative to the serving location.

All this is to support transposability. 

_meta is not just about the handle, it's about the knop "as a knop" and includes info about the Knop's flows, slices, and distributions.

If we make the handle->subject mapping explicit, it will look like <../../_knop-handle> sflo:isHandleFor <../../> - It is unfortunate that you won't know what you're referring to just by looking at the RDF though. But the subject/referent IRI will mostly only be found in _ref, which would usually have some kind of identifying info.

For roles vs classes for different Flow types.... A flow will only ever have a singe role. Is it still better than classes and flow-specific properties? YOu could have flow-specific properties anyhow... I like them because you can pick out a flow without inspecting its role. 
ChatGPT said:

I’ll go in the same order you raised things and push on the edges.
1. Slug creation: “maybe we don’t define it unless provided”

You’re flirting with two incompatible behaviors:

    Slug omitted

        Distributions must still be named somehow.

        So you either:

            derive from the path every time (unstable under renames), or

            have some other naming rule (which becomes the de facto slug).

    Slug present

        Now you have a stable, explicit base name for distributions.

If you allow “no slug” + “derive from path” you get:

    Move/rename folder → new distribution names → new IRIs → old links break.

    You’ve sacrificed exactly the “semi-permanent IRIs” you cared about earlier.

If you care about transposability across serving locations but also semi-stable IRIs, the rules should be:

    Slug is persistent config on the knop, not operational.

    On createKnop:

        If user passed a slug: use that.

        If not: derive from path, and immediately persist it into _meta.

    From that moment on, distributions derive from the persisted slug, not the current path.

So I would not leave slug undefined. I’d allow it to be implicit at creation time but require that:

    as soon as a knop exists, it has a slug in _meta.

Otherwise you’re smuggling path-based identity back in through the side door.
2. Subject IRI anchored at <../../> and @base

You said:

    The single anchor for the subject IRI is <../../> – SPARQL queries will be fine because at load time, base will be inferred from the serving location.

This is consistent if you accept:

    Subject IRI is deployment-relative:

        serve the same files from /a/ vs /b/ → different effective subject IRIs.

    Identity across deployments is established by:

        “same intramesh relative IRI in the same dataset” rather than “same absolute HTTP IRI”.

That’s okay for transposability, but it has trade-offs:

    Dump the mesh as pure RDF and load it into a random triple store → the store will likely treat the file URL (or some other default) as base, not your HTTP hosting base.

        Result: subject IRIs differ depending on how you loaded the file.

    So “SPARQL queries will be fine” is true within a given deployment, but not automatically across all tooling contexts.

If you can live with:

    “Subject identity is deployment-contextual unless we add an explicit intramesh base,”

then <../../> as the anchor is fine. Just don’t overclaim portability of “the same subject IRI” outside the hosting context.

(If you ever want cross-deployment subject identity, you’ll eventually need an explicit intramesh scheme or explicit @base, not just “implicit from serving location”.)
3. _meta scope: not just handle, but “knop-as-knop”

You clarified:

    _meta is not just about the handle, it's about the knop "as a knop" and includes info about the Knop's flows, slices, and distributions.

That resolves some earlier tension but also forces a structural decision:

You now effectively have three conceptual layers:

    Subject (referent): <../../>

        “The thing in the world.”

    Knop: the “mesh node” corresponding to the subject.

        It knows about flows, slices, distributions.

        It’s the level where we talk about _payload, _ref, _meta flows, etc.

    Handle: optional separate “naming entity” that identifies the knop.

Right now you’re merging 2 and 3 inside _meta. That’s fine, but then:

    Either the handle is a separate resource (e.g. <../../_knop-handle>) with sflo:isHandleFor <../../>.

    Or the knop “as knop” is the subject <../../>, and the handle is just a conceptual notion, not a separate IRI.

If _meta is “about the knop as knop” and also carries flows/slices/distributions, then I’d strongly suggest:

    Make the knop the subject of _meta:

<../../>
  a sflo:Knop ;
  sflo:slug "foo-knop" ;
  sflo:hasPayloadFlow <../../_payload-flow> ;
  sflo:hasReferenceFlow <../../_ref-flow> ;
  ...

If you insist on a separate handle resource, keep it very minimal:

    <../../_knop-handle>
      a sflo:KnopHandle ;
      sflo:isHandleFor <../../> .

Right now you’re trying to have _meta “only about the handle” while also stuffing genuine knop semantics (flows, slices, dists) into it. That is exactly the conflation causing confusion.

If _meta is the “knop description file,” let it describe the knop (<../../>) directly.
4. Explicit handle→subject mapping: sflo:isHandleFor

    If we make the handle->subject mapping explicit, it will look like <../../_knop-handle> sflo:isHandleFor <../../> – It is unfortunate that you won't know what you're referring to just by looking at the RDF though.

That “unfortunate” is inescapable for this pattern:

    Any indirection layer (handle vs subject) means you have to chase a property to see “what this thing is for”.

You can mitigate this by:

    Putting a clear rdfs:comment / rdfs:label on the subject in _ref / _default.

    Using consistent naming patterns so that human readers can still infer some meaning from relative paths.

But fundamentally, if you want:

    a separate handle entity,

        explicit handle→subject mapping,

then “need to follow a link to see what this is about” is inevitable. That’s not a design bug; it’s the nature of the indirection.

If that still bothers you, the simpler fix is:

    Drop the separate handle IRI, and let <../../> be the knop-as-knop, with _meta describing it.

5. Roles vs classes vs flow-specific properties

You asked:

    I agree, FlowRoles seems promising. But I don't want configurable suffixes, so maybe overkill. Is it still better than Flow-specific properties (e.g. hasKnopMetadataFlow) and subclasses (KnopMetadataFlow)?

    A flow will only ever have a single role. Is it still better than classes and flow-specific properties?

These aren’t mutually exclusive; they serve different purposes:

    Classes (sflo:KnopMetadataFlow, sflo:KnopPayloadFlow):

        Good for SHACL / OWL constraints.

        Expresses “this flow structurally behaves like X” (required properties, shapes, etc.).

    Roles (sflo:hasRole sflo:payloadRole):

        Good for parametrization and config.

        Lets you say “apply template T to all flows with role = payloadRole”.

        Works well when you eventually want a small enum-like set.

    Flow-specific properties (sflo:hasKnopMetadataFlow, sflo:hasPayloadFlow):

        Great for ergonomics: “pick out this flow without inspecting its role.”

        They can be defined as shortcuts / functional properties whose range is constrained to the correct class/role.

Given your constraints:

    “Each flow has exactly one role”

    “Suffixes are fixed in code, not configurable”

    “I want to pick out a flow without inspecting role”

The most coherent pattern is:

    Keep flow-specific properties for ergonomics:

sflo:hasPayloadFlow
  rdfs:domain sflo:Knop ;
  rdfs:range  sflo:PayloadFlow .

Optionally keep classes (sflo:PayloadFlow, sflo:MetadataFlow) if you plan to write SHACL.

Add a single sflo:flowRole property if you foresee any role-based behavior (templates, viewers, etc.):

sflo:flowRole a rdf:Property ;
  rdfs:domain sflo:Flow ;
  rdfs:range sflo:FlowRole .

And assert both:

    <../../_payload-flow>
      a sflo:Flow, sflo:PayloadFlow ;
      sflo:flowRole sflo:payloadRole .

This is not “overkill”; it’s a tiny bit of redundancy that buys you:

    Clean SHACL / OWL typing.

    Easy ergonomic properties (hasPayloadFlow).

    A single generic hook (flowRole) for config and UI logic.

You can still:

    Hard-code suffixes (_payload, _ref, _meta) in code only. No need to represent them in RDF.

6. Back to your core goals

You keep repeating two meta-requirements:

    Transposability:

        Same tree of RDF files can be served from different bases.

        Subject IRIs are relative and resolved at load/serve time.

    Reasonable self-descriptiveness:

        SPARQL and other RDF tools should mostly be able to understand the mesh without specialized logic.

The design that minimally satisfies both, without going in circles, looks like:

    Subject IRI: <../../> in _meta, resolved via @base.

    Knop: <../../> a sflo:Knop ; sflo:slug "foo" ; sflo:hasPayloadFlow <../../_payload-flow> ; ....

    Slug: defined persistently on the knop; defaulted from path at creation time, never left undefined once created.

    Handle: optional; if present, keep it small and clearly mapped: <../../_knop-handle> sflo:isHandleFor <../../>.

    Flows: classes + flow-specific properties (hasPayloadFlow, hasMetadataFlow) + optional flowRole.

    Suffixes: fixed, in code only, not in RDF.

Where you’re still trying to be too clever is:

    Allowing slugs to be half-implicit and half-config;

    Letting _meta be about “handle” sometimes and “full knop semantics” other times;

    Wanting RDF to be self-describing while refusing to encode minimal subject/flow structure there.

If you lock in the combination above, most of these tensions disappear.
You said:
deriving the "identifier" from the path is good because it supports transposability/composability. If you move/rename the folder, yese the distribution names change, but _meta still lists the old versions, so they are just as discoverable as if you define a non-standard slug anyhow. path-based identity is essential for transposability/composability. 

If you dump the mesh as pure RDF, you could specify the base.

Pre-deployment, you can use file:// IRIs for SPARQL / processing. Handle is definitely a separate resource. The knop can't be the subject of _meta because, by the most important convention in the platform, the Knop IRI  refers to the referent/thing in the real world, and the knop-handle refers to the Knop. It's a bit convoluted, but it's the only way to preserve the "single referent principle". Meta is about knop semantics, i.e. subject is always ../../_knop-handle. 

For defining the flow-specific folder names and distribution suffixes, we could use an annotationproeprty on the classes. Does that run afoul of class-instance stuff? 

_meta is only about the knop-as-knop. I'm not sure you're grokking the subtle invention here: the _knop-handle IRI refers to the knop as a node. That's essential for single reference principle.
ChatGPT said:

Good, this is clearer. I’ll stick to your premises and push on the consequences, not try to drag you back to a different design.
1. Path-based identity and transposability

Your position now is essentially:

    The path is the canonical identifier seed.

    If you move/rename a folder, you are intentionally changing the identity:

        distribution names (and thus IRIs) change,

        previous distributions remain listed in older _meta / history and are just as discoverable as if you had picked some hard-coded slug.

Under that regime:

    A rename is not “same knop, new location”; it is “new knop, created from this filesystem state”.

    The old knop’s distributions remain at the old path (or in the old version tree) if you don’t delete them.

    Transposability/composability =
    “you can take this folder tree and drop it under another base and everything still works,”
    not
    “this abstract subject keeps a single eternal HTTP IRI regardless of how you move things around.”

That is logically coherent, but you must accept:

    A move/rename that physically relocates the same files without leaving the old ones in place is a semantic change that breaks old IRIs. That’s not a bug; that’s the model.

    The only way “old versions are still discoverable” is if you:

        either keep their distributions in some versioned tree that doesn’t move when you rename the “working” folder, or

        treat the old path as “frozen history” and only add new versions under the new path.

If you’re okay treating path as part of the versioned identity, then yes: deriving the identifier from the path is consistent with your transposability story.
2. Subject anchor <../../> and @base

What you’re now saying:

    Subject IRI is always <../../> in the mesh.

    At load/serve time, @base is inferred from:

        HTTP serving location, or

        file:// URL, or

        an explicitly supplied base when you dump/import into a triple store.

That gives you:

    Deployment-relative subject identity:

        Same relative <../../> + same content → “same subject” within that deployment.

    For pure RDF dumps, you can indeed supply a base explicitly when loading.

So:

    Yes: anchoring the subject at <../../> is compatible with RDF semantics and transposability, as long as you are okay with identity being contextual to the chosen base.

There is no contradiction here once you accept that “the” subject IRI is not a single absolute URL baked into the data.
3. Handle vs knop vs subject: I do grok it

Let me restate your layering in your terms:

    Subject (referent) – The real-world thing.

        IRI: <../../>

        This must obey the “single referent principle”.

    Knop-as-knop – The mesh node representing the subject.

        This is not the subject; it’s an artifact in the mesh.

    Knop handle – The IRI that refers specifically to the knop-as-knop.

        IRI: <../../_knop-handle>

        _meta is about this resource:

            its flows,

            slices,

            distributions,

            path-derived identity.

This is exactly the subtle invention you’re talking about:
you refuse to overload the subject IRI with “node semantics.”
Instead:

    <../../> → the thing in the world.

    <../../_knop-handle> → “the node that organizes flows about that thing.”

That’s a clean way to preserve the single referent principle.

Given that, it is correct that:

    _meta’s subject is always <../../_knop-handle>.

    _meta is “about the knop-as-knop,” not the real-world referent.

You still need one explicit mapping somewhere:

<../../_knop-handle> sflo:isHandleFor <../../> .

but that can live in _meta without violating your principle, because it’s a statement about the handle, not about the subject.
4. Slug and path-based identity under this scheme

With your clarified stance:

    You do not want a stable, path-independent slug to keep IRIs constant across renames.

    You explicitly want path-based identity to be first-class.

Then the consistent rule is:

    The “identifier” (whatever you call it: slug, nodeId, etc.) is derived from the path and used for:

        naming flows,

        naming distributions,

        maybe even forming fragments of IRIs.

    It only needs to be persisted if you want:

        traceability of former identifiers, or

        to detect accidental inconsistencies.

So your earlier idea now fits better:

    You don’t define sflo:slug unless createKnop is given an override.

    Default behavior = fully path-derived identifiers.

    If you do define a slug in _meta, it is an explicit divergence from the current path and therefore a conscious deviation from the normal transposable/composable story.

That’s coherent given your acceptance that renames = identity changes.

The only caution:
if you eventually introduce any mechanism that treats a folder rename as “same knop, new path”, you will have to revisit this and introduce some non-path key. For now, you’re intentionally not doing that.
5. _meta as “knop-as-knop” container

Given all of the above, _meta can legitimately carry:

    Properties of the handle-as-knop, subject = <../../_knop-handle>:

        path-derived identifier or slug,

        which flows exist (payload, ref, meta, configs),

        which slices and distributions are present / current,

        maybe versioning information relevant to the node.

None of that touches the subject <../../> directly. It only ever shows up as the object of sflo:isHandleFor.

That is consistent with the single referent principle and with your file layout.
6. Roles vs classes vs flow-specific properties

You asked:

    For roles vs classes for different Flow types.... A flow will only ever have a singe role. Is it still better than classes and flow-specific properties? You could have flow-specific properties anyhow... I like them because you can pick out a flow without inspecting its role.

With your constraints:

    each flow has exactly one role,

    suffixes are fixed and not configurable,

    you care about ergonomic queries like “give me the metadata flow for this knop”

the sensible pattern is:

    Flow-specific properties from handle to flows:

        sflo:metadataFlow, sflo:payloadFlow, sflo:referenceFlow, etc.

        Domain: sflo:KnopHandle

        Range: sflo:Flow (or sub-classes).

These give you the “pick out a flow without inspecting role” ergonomics you care about.

    Flow classes for structural constraints:

        sflo:MetadataFlow, sflo:PayloadFlow, etc.

        Useful for SHACL and for type-checking flows generally.

    Roles are optional syntactic sugar at this point:

        If you foresee logic like:

            “apply template T to all flows with role X for any knop”

        then a sflo:flowRole + sflo:PayloadRole enum remains useful.

        If you don’t see that need, classes + properties are enough.

Given your desire to avoid configurable suffixes and your preference for direct “hasXFlow” properties, it’s absolutely legitimate to skip roles in v1 and just use:

<../../_knop-handle>
  sflo:metadataFlow  <../../_meta-flow> ;
  sflo:payloadFlow   <../../_payload-flow> ;
  sflo:referenceFlow <../../_ref-flow> .

and type the flows with classes.

Roles become valuable later if you want to generalize behavior across flows independent of their specific handle properties.
7. Annotation properties for folder names and suffixes

    For defining the flow-specific folder names and distribution suffixes, we could use an annotation property on the classes. Does that run afoul of class-instance stuff?

You could do, for example:

sflo:folderSuffix a owl:AnnotationProperty .

sflo:MetadataFlow
  a owl:Class ;
  sflo:folderSuffix "_meta" .

sflo:PayloadFlow
  a owl:Class ;
  sflo:folderSuffix "_payload" .

This does not cause any class-instance punning problems:

    You’re attaching an annotation to a class resource, which is allowed in OWL 2 DL.

    You’re not treating the class as an individual of another class; you’re just adding metadata.

What you are doing is baking an implementation detail (folder/suffix) into the ontology TBox. That’s a judgment call:

    Pros:

        Tooling in any language can discover the convention directly from the ontology.

        No need to duplicate suffix constants across hosts if they all read the ontology.

    Cons:

        You’re entangling filesystem layout conventions with the conceptual model.

        If you ever want alternative layouts (even just for a test rig), you either:

            break the convention, or

            need another layer of override (platform config) that shadows these annotations.

Given that you explicitly don’t want suffixes to be configurable, putting them as annotation properties on classes is defensible. It doesn’t violate class–instance separation; it just couples ontology and platform tightly.

If you want a very strict separation between “platform defaults” and the conceptual model, you might instead:

    Put these annotations into a platform config graph in an sflo-platform ontology, not in the core ontology of flows themselves.

But that’s an organizational choice, not a semantic constraint.
Short version

    Yes, I now understand and agree that _knop-handle is the node-as-node, and _meta is about that resource; the subject <../../> remains the real-world referent and is only pointed to via something like sflo:isHandleFor.

    Path-based identity + renames-as-new-identity is coherent with your transposability story; in that world, leaving slug undefined unless overridden is acceptable, because the path is the identity.

    Using annotation properties on Flow classes to record folder/suffix conventions is technically fine and doesn’t break class/instance semantics; it just couples implementation and ontology.

    Given your preferences, flow-specific properties (metadataFlow, payloadFlow, etc.) plus classes are enough; FlowRole is optional sugar you can add later if you need more generic flow behavior.

You said:
“same knop, new location” -- it's not so easy. Is an identical knop served from a different location really a new knop? Luckily, I don't think we need to answer that question. If it's moved, it's distributions are moved too, but I do want to support a "redirect" syntax, where if you change a knop's path, the old path stays but the _current has some kind of "redirection" statement, ie. "frozen history". Maybe we can provide an optional "publishedIRI" or even "anticipatedAbsoluteIri" where, if the publishing target is provided, the absolute IRI is calculated. 

So do we abandon slug altogether? It might be useful if you want to override the distribution name. But that does impede transposability a bit... when you duplicate a knop, if the slug is defined, your distribution naming won't adapt to the new path. 

The "identifier" I want to call the "intramesh identifier".  

"if you eventually introduce any mechanism that treats a folder rename as “same knop, new path”, you will have to revisit this and introduce some non-path key." Why? 

We should put  sflo:folderSuffix details in platform config! That way it could be overridden easily, say, if you wanted a different language. 

folderSuffix isn't quite the right name. It's the actual folder name, and also by convention the distribution suffix. 
ChatGPT said:

I’ll stick with your premises and show where they force decisions.
1. Identity and “same knop, new location”

You’re right: “same knop, new location” is not trivial.

You’re currently saying:

    Path is your intramesh identifier seed.

    Moving a knop moves its distributions too.

    Old path can remain as “frozen history” with some redirection.

In terms of identity:

    If you move a folder and delete the old one → that’s definitely “new knop, new IRIs”.

    If you keep the old one and put a redirect in _current → you’ve effectively created a linking record saying:

        “Old IRI and new IRI refer to the same conceptual knop.”

That redirect is your “non-path key,” just expressed relationally rather than as a direct ID:

    E.g. <old/_knop-handle> sflo:isSameKnopAs <new/_knop-handle>
    or

    <old/_knop-handle> dct:isReplacedBy <new/_knop-handle>.

So when I said “you’ll need a non-path key if you support same-knop-new-path,” what I really meant was:

    You need some stable notion of sameness that is not just “same path string”.

    That can be an explicit ID (intrameshId) or an explicit relation between old/new handles (redirect/equivalence).

    Either way, it’s not “purely path-based” anymore.

Your redirect idea is compatible with that: it is the extra structure that encodes same-knop across paths.

“AnticipatedAbsoluteIri” is fine as extra metadata, but:

    It can’t be your canonical identity if deployment can change.

    It’s just a hint for server config or link generation.

2. Intramesh identifier vs slug vs path

You now want:

    “identifier” = intramesh identifier (good name).

    Path-based identity is “essential for transposability/composability”.

You need to be crystal clear:

    Is the intramesh identifier:

        literally the path string?

        or a value derived from the path at creation that can later diverge?

You just argued against slug as a long-lived override because:

    If you duplicate a knop to a new path but keep the slug, distribution names won’t auto-adapt.

That only matters if you intend “duplicate knop to new path = new, independent knop.”
If that’s the case, then:

    It is correct that slug must be regenerated or changed at duplication time.

    That’s true whether you call it “slug” or “intramesh identifier.”

So you have two coherent options:
Option A – Path = intramesh ID (strict)

    No separate slug.

    Intramesh identifier is the canonical path-based stem.

    Move/rename = new knop unless you explicitly maintain redirects/equivalence.

    Duplication to new path must produce a distinct intramesh ID (because the path changed).

This is the cleanest match to your transposability story.
Option B – Intramesh ID as a stored value

    At creation: intrameshId is derived from the path.

    It is persisted (in _meta).

    It does not auto-update on rename.

    Distributions are named from intrameshId, not directly from path.

Then:

    A rename where you intend “same knop” = change path but keep intrameshId and add redirect (old handle → new handle).

    A duplication where you intend “new knop” = same files but new path → the tool must generate a new intrameshId and adjust distributions.

That’s more work, but gives you “same knop, new path” as a first-class operation rather than an accident.

Right now, you’re half in A (path as identity) and half in B (redirects and future “same-knop” semantics). You should pick which world you’re in and design around it.
3. Do you abandon slug?

If “slug” is just “the intramesh identifier but with a more human-oriented name,” then it’s redundant.

Given what you just said, the realistic choices are:

    Drop sflo:slug entirely and:

        Use sflo:intrameshId (or similar) as the canonical identifier.

        Define conventions that this ID is usually path-shaped (foo/bar/baz), but it’s just data.

        Derive distribution names from intrameshId.

    Keep slug as a rare override, with hard rules:

        Default: slug = path-derived-stem at creation.

        On duplication-as-new-knop, your tooling must require or auto-generate a new slug.

        If you move and intend “same knop, new path”, you don’t change the slug, you just add redirect triples.

If you go with Option A (path is identity, no redirects), slug buys you very little; I’d drop it.

If you go with Option B (identity independent of current path, redirect supported), then slug == intrameshId and you should:

    Treat it as required, persistent config, not something that might be absent.

    Stop thinking of it as “just a filename override”.

Right now you’re using “slug” both as “identity seed” and “optional override”—that’s what’s muddy.
4. Why a non-path key for “same knop, new path”?

Your direct question:

    "if you eventually introduce any mechanism that treats a folder rename as “same knop, new path”, you will have to revisit this and introduce some non-path key." Why?

Because:

    If “knop identity” = “its path”, then by definition:

        new path → new identity.

    If you want to say “no, actually, this is the same knop despite changing the path”, then:

        sameness can no longer be “same path string”.

        you must define sameness based on something else:

            an explicit intramesh ID that doesn’t change, or

            a relation like isSameKnopAs / isReplacedBy that ties old and new handles together.

That “something else” is what I mean by a “non-path key.”
You’re already halfway there with your redirect idea; that’s exactly a non-path-based notion of sameness (a relation connecting different path-based IRIs).

So:

    Path-based alone ⇒ no “same knop, new path” without changing identity.

    “Same knop, new path” ⇒ you must encode sameness somewhere beyond “same string”.

5. Platform config for folder/distribution names

    We should put sflo:folderSuffix details in platform config!
    folderSuffix isn't quite the right name. It's the actual folder name, and also by convention the distribution suffix.

Agreed on platform config:

    These are platform conventions, not conceptual truths about flows.

    Putting them in a “platform config graph” means:

        core ontology stays cleaner,

        different platforms/localizations can override names,

        your code reads from a central, declarative place.

On naming:

    folderSuffix is indeed misleading if the folder name is exactly that token, not “suffix over something else”.

You probably want something like:

    sflo:canonicalFolderName

    sflo:defaultFolderName

    sflo:flowFolderName

and a separate property if you really want to distinguish file suffix:

    sflo:distributionNameStem or sflo:distributionSuffix.

Given your “same token used for folder name and distribution suffix” constraint, I’d pick:

sflo:artifactToken
  a owl:AnnotationProperty ;
  rdfs:comment "Canonical token used as folder name for this flow type and as the conventional stem/suffix for its distributions."

and define in platform-config graph:

sflo:MetadataFlow sflo:artifactToken "_meta" .
sflo:PayloadFlow  sflo:artifactToken "_payload" .
sflo:RefFlow      sflo:artifactToken "_ref" .

That matches your behavior:

    Code uses artifactToken for:

        folder name,

        distribution stem/suffix.

You can override it (e.g. to _metadonnees for a French-localized mesh) by swapping just the platform config.
6. The _knop-handle invention

I do understand it, and I think it’s internally consistent:

    Subject <../../>: the real-world thing.

    <../../_knop-handle>: the node-as-node, the organizing artifact in the mesh.

    _meta describes <../../_knop-handle>:

        its flows,

        slices,

        distributions,

        intramesh identifier / artifact token,

        and possibly its redirect relations.

That respects:

    the single referent principle, and

    your separation of “about the thing” vs “about the mesh artifact that organizes knowledge about the thing.”

Where you still need to make a hard choice is:

    Is the intramesh identifier just “whatever the current path is” (Option A)?

        Then you don’t store it separately (or only as a checksum / trace).

        Moves = new knop, unless you explicitly wire redirects.

    Or is the intramesh identifier a stable value stored in _meta (Option B)?

        Then path becomes a convenience; id is the semantic anchor.

        Moves = update path, keep intrameshId, possibly record redirect IRIs.

Your redirect idea implicitly leans toward Option B. If that’s what you want, stop treating path as the true identity and promote intrameshId to first-class, path-derived-at-creation-but-independent-thereafter.
You said:
Actually, the way I currently have intramesh identifier defined, it's different than the slug/knop identifier:


An **intramesh identifier** is a esentially a relative [[concept.iri]] (i.e., without the scheme, e.g., https:// or file://) except that they should correspond to an existing [[mesh-resource]], i.e. [[principle.dereferencability-for-humans]]

There are two types of intramesh identifiers: [[concept.identifier.intramesh.relative]] and [[concept.identifier.intramesh.absolute]]. Absolute identifiers are not recommended because they break composability/transposability.

## Syntax

Like relative URLs:

  * Written without a scheme (e.g., no https:// or file://).
  * May use path segments (../, /foo/bar/) 
  * May use fragment identifiers (../#, /foo/bar/#inner-default-template)

Unlike URLs:

  * **Must not** contain queries (?).


## Semantics

- Same as IRIs in RDF: they [[denote|concept.denotation]] things (aka [[concept.referent]]) which may be mesh resources or “things in the world”.
  - Fragments behave the same as in IRIs: they refine the primary denotation.

## Purpose

- locate [[mesh-resource]]s (when used in a filesystem or web site context)
- denote a [[concept.referent]], either [[internal|facet.internal]] or [[external|facet.external]]

## Identifier Name Limitations for Users

- initial underscores prefix all [[facet.system]] identifiers and should be avoided in general for [[facet.user]] identifiers

## Distribution Relativity



### Examples

- Knop self-reference: ../../
- sibling knop: ../../../my-dataset  
- sibling components: "../../_cfg-local/_current/config.trig", "../../_cfg-inh/_current/config.trig", "../../_payload/_current/data.jsonld"  
- Components in other flows: Same pattern, just different flow names

---

I guess we can just call these (former slugs) "knop identifiers", and they are just the "last segment" in the on-disk path, or the URL.  Can you compose a concise wiki article for them? Their relationship to intramesh identifiers seems delicate, so feel free to ask me questions
ChatGPT said:

Before I write the concise wiki article, I need two clarifications, because they determine the correct definition of “knop identifier” and its relationship to intramesh identifiers.

You’ve already sensed this is delicate; if I assume wrong, I’ll encode contradictions into your doctrine.
I need clarification on exactly these two points:
1. Is a knop identifier always exactly the last path segment?

You said:

    "we can just call these (former slugs) 'knop identifiers', and they are just the 'last segment' in the on-disk path, or the URL."

This raises a key question:
A. Does the system ever treat the knop identifier as stable across path changes?

Or is it strictly:

    “identifier = path segment”

    changing folder name → new identifier

    no persistence in _meta needed

If the identifier never stabilizes beyond the path, then:

    It is not metadata.

    It does not belong in _meta.

    It is purely a file-system projection: the symbolic name of the node.

But earlier you mentioned:

    “redirect syntax”
    “frozen history”
    “same knop new path”

Which only makes sense if there exists a conceptual entity whose identity is not simply the path.

So I need to know whether:
Which of these is your intended model?
Option A — Path = identity (strict)

    Rename = new knop.

    Duplicate = new knop.

    Redirects encode conceptual continuity but do not unify identity.

    Knop identifier is always derived from path, never stored.

Option B — Identity is path-derived-at-creation but may diverge

    A knop has a persistent identifier (knopId).

    Default knobId = last path segment at creation.

    Path may change without changing identity.

    Redirects encode movement; identity is stable.

Which one is correct?

This determines whether the “wiki article” describes:

    a purely syntactic name (Option A), or

    a semantic identity key that happens to default to the last segment (Option B).

2. Relationship to intramesh identifiers

You said:

    “intramesh identifiers are relative IRIs… they denote things in the world… or mesh resources.”

So here is the key distinction I must confirm:
Does every knop have an intramesh identifier?

Example possibilities:

    Knop identifier = "my-knop"

    Full intramesh identifier = "../my-knop/" or similar.

So I need to know:
Is the “knop identifier” simply the last path segment used to construct a full intramesh identifier?

Meaning:

knopId = "my-knop"
intrameshId = "../../"     # the subject
knop-handle = "../../_knop-handle"
knop-folder = "../../my-knop/"

Or:
Is the knop identifier itself an intramesh identifier (or intramesh-identifier fragment)?

e.g.:

knopId = "../my-knop/"

These reflect two different conceptual models:
Model 1 — Knop identifier is a name, intramesh identifier is the actual locator/IRI.

    For humans: "my-knop"

    For RDF/mesh: "../../"

Model 2 — Knop identifier is the intramesh identifier

    No duality

    The “identifier” is the thing you put in triples / distribution naming.

Please tell me explicitly which model you intend.
Why I need these clarifications before writing the wiki article

A concise, authoritative article about “knop identifiers” must state:

    Are they metadata or just path projections?

    Are they stable or ephemeral?

    Are they semantic identities or syntactic convenience names?

    Do they unify anything across renames?

    Are they part of intramesh addressing? Or only part of filesystem layout?

    Are flows allowed to use them for naming?

    Should users treat them as “public-facing IRIs” or internal symbolic handles?

Right now, these questions are still ambiguous because your design has evolved quickly over the last few messages.
Please answer these two clarifications:
✔ Q1: Is a knop identifier strictly equal to the current path segment, or can a knop keep its identifier across path changes?

(Option A vs Option B above.)
✔ Q2: Is a knop identifier:

    (1) merely the base name used to derive an intramesh identifier,

    (2) or itself a valid intramesh identifier (or identifier fragment)?

Pick explicitly.

Once you answer these two, I will produce a tight, elegant wiki article defining:

    knop identifier

    how it differs from intramesh identifier

    rules for path-based derivation

    rules for transposability/composability

    how publication and distribution naming interact with it

    when/if it becomes persistent metadata.

You said:
Path = identity (strict). That's consistent with the primary goal -- minting IRIs. redirect syntax, etc., can just point to the new path. So maybe rename/move is a new knop, but it's more like a single conceptual knop with multiple parts and identifiers, leveraging isSameKnopAs.

I'm now thinking that "moving" a knop doesn't move its previous distributions. At least by default. You have to use the sflo-cli (or API) to do a move, and it leaves the existing files in place, adds redirect info to the old "_current" slices, adds a banner to relevant old resource pages, copies the _working folders and their distribution files (and _assets) to the new location, and adds post-move metadata to the new _meta/_working/ so that the previous version in the old location can be referenced on weave.

the “knop identifier” is simply the last path segment
ChatGPT said:
Knop Identifier

A knop identifier is the final path segment of a knop’s on-disk (or URL) location. It is a purely path-based name, used as the local symbolic identity of a knop within a mesh.

It is intentionally simple and syntactic: the platform treats path = identity (strict) for knops.
Definition

    A knop identifier is:

        The last segment in the knop’s directory path (filesystem or URL).

        For example, given a knop root at:

            Filesystem:
            .../mesh/people/alice/ → knop identifier: alice

            URL:
            /ns/people/alice/ → knop identifier: alice

    Knop identifiers:

        Do not include scheme (https://, file://).

        Do not encode hierarchy by themselves (they are just the tail segment).

        Are conceptually separate from intramesh identifiers.

Relationship to Intramesh Identifiers

Recall:

    An intramesh identifier is a relative-IRI-like string (no scheme, no ?query) used to:

        Denote mesh resources or real-world referents.

        Locate resources in a filesystem or web context (using ../, /foo/bar/, #fragment, etc.).

Typical relationship:

    A knop’s intramesh identifier for its subject (the referent) is something like:

        ../../ (for “this knop’s subject”)

    The knop identifier is only the last path segment in that hierarchy, e.g.:

        If the knop folder is /ns/people/alice/, then:

            Subject intramesh IRI: ../../ (resolved with the appropriate base).

            Knop identifier: alice.

So:

    Intramesh identifiers are full locators (can contain multiple segments + #fragment).

    Knop identifiers are local names (single path segment) that appear at the end of those locators.

They are related, but not interchangeable:

    intramesh ID ≈ full relative IRI

    knop identifier ≈ the leaf name in that path

Identity Semantics

    For knops, path = identity (strict):

        The combination of mesh base + path uniquely identifies the knop.

        The knop identifier is part of that identity because it is the final path segment.

    Renaming or moving a knop folder changes its identity:

        New path → new knop identifier.

        Old path/identifier, if left in place, continues to identify the “old” knop.

Conceptually, you may still treat old/new knops as “the same conceptual thing” by linking them (e.g., via sflo:isSameKnopAs or redirect metadata), but technically they are distinct knops with distinct identifiers.
Moves, Redirects, and “Frozen History”

“Moving” a knop is not a raw filesystem rename; it is a higher-level operation performed via sflo-cli / API:

    Old location (pre-move):

        Existing distributions and _current slices stay in place.

        _current at the old path gains redirect metadata:

            e.g., “this knop has moved to /new/path/”.

            Optionally isSameKnopAs / isReplacedBy relationships.

    New location (post-move):

        _working folders and their distributions (and _assets) are copied to the new path.

        _meta/_working at the new path records post-move metadata so the old knop can be referenced on future weaves.

Effectively:

    The old knop becomes frozen history + redirect.

    The new knop is the current working instance.

    A chain of redirect/equivalence links lets you treat them as “facets of the same conceptual knop,” even though their knop identifiers (and paths) differ.

Usage and Constraints

    Knop identifiers:

        Are used to form folder names and, by convention, often appear in distribution filenames.

        Must avoid leading underscores for user-defined names; leading underscores are reserved for system facets (_meta, _payload, _cfg-local, etc.).

        Are not required to appear as a dedicated RDF property; they can be inferred from the path. (You can mirror them into RDF in _meta if you want, but that’s an implementation choice.)

    They are not:

        A global, path-independent identifier.

        A substitute for intramesh identifiers.

        A guarantee of conceptual sameness across moves; that job is done by explicit redirect / equivalence links.

Summary

    Knop identifier = last path segment naming a knop’s folder.

    Intramesh identifier = full relative-IRI-like locator (possibly multi-segment, with fragments) used to denote subjects and resources within a mesh.

    For knops, path (and thus knop identifier) is the identity; moves and copies create new knops, and any “same concept” relationship between them is encoded explicitly via redirects / equivalence, not via a shared intrinsic ID.

You said:
since intramesh identifiers might not contain the knop identifier (e.g. ../../) we can't say "the leaf name in that path"

ALso, "For knops, path (and thus knop identifier) is the identity" -- hints at something... we should call it "Knop name" or maybe "knopLocalName" because it is specifically not a unique identifier. 
ChatGPT said:

Understood. Those two corrections actually clarify the whole conceptual structure:

    Intramesh identifiers do not necessarily contain the knop name
    (../../ is the canonical example),
    therefore a knop’s name is not the “leaf segment of its intramesh identifier.”
    Good catch.

    The thing you were calling “identifier” is explicitly not an identifier
    —it is a local name, derived from path, used for naming artifacts and forming a readable folder structure, but not guaranteeing uniqueness and not determining subject identity.

So yes: “knopLocalName” or “knop name” is the correct category.

With those clarifications, here is the corrected wiki article.
Knop Local Name

A knop local name is the final path segment used to name a knop’s folder in the mesh filesystem or URL layout. It is a syntactic, non-unique, path-derived label used for organization and discoverability—not an identifier in the semantic sense.

Knop local names do not determine the subject’s identity. They are simply the human-readable names that appear in the mesh structure.
Definition

    A knop local name is:

        the last path component in the knop’s directory, e.g.
        …/people/alice/ → knopLocalName = "alice"

    It exists for:

        human readability,

        predictable organization of flows and distributions,

        forming convenient filenames for distributions.

    It is not:

        globally unique,

        semantically meaningful,

        necessarily present in intramesh identifiers.

Relationship to Intramesh Identifiers

Intramesh identifiers are:

    relative-IRI-like locators: ../../, ../characters/hero/#stats, etc.

    used by the mesh to denote:

        referents (“things in the world”),

        flows,

        distributions,

        other mesh resources.

Crucially:

    Intramesh identifiers do not need to (and often do not) contain the knop local name.
    Example:

        The knop’s subject intramesh IRI is always <../../>
        —a structural locator, not the string "alice" or "monster".

    Therefore, knopLocalName is not the leaf of any intramesh IRI.

The two concepts serve different layers:
Concept	Purpose	May contain knopLocalName?
Knop local name	Naming the folder representing the knop	yes
Intramesh identifier	Denoting resources & referents in RDF	not required; usually no
Subject intramesh id (../../)	Canonical IRI for the referent	never contains it
Path = Identity (for knops)

For knops as mesh constructs:

    The path itself is the identity.

    Changing the path creates a new knop, unless explicitly linked via redirects.

Thus:

    …/people/alice/ and …/members/alice/ are distinct knops, even if their contents are identical.

    The knopLocalName "alice" is part of the path, but does not itself guarantee identity.

Knop identity is path-based, not name-based.
Moves, Redirects, and Conceptual Continuity

A filesystem rename is not a semantic rename.
Instead, the system provides a knop move operation via sflo-cli or API:

    Old location:

        Distributions remain.

        _current slices acquire a redirect statement.

        Resource pages show a “moved” banner.

    New location:

        Working folders and distributions are copied.

        _meta/_working includes post-move metadata for weave.

The system can then express:

<old/_knop-handle> sflo:isSameKnopAs <new/_knop-handle> .

—without reusing identifiers.

These are distinct knops with conceptual continuity provided by explicit relations.
Why Not Call It an “Identifier”?

Because:

    It does not guarantee uniqueness across the mesh.

    It is not stable across moves.

    It is not used in intramesh identifiers (which determine referent identity).

    RDF does not use it to denote any resource.

Calling it an “identifier” would strongly imply semantics it does not have.

“KnopLocalName” or simply “Knop Name” is accurate and honest.
Usage Guidelines

    Users choose meaningful names for knops (folders).

    Avoid leading underscores (_) —reserved for system facets.

    Changing a knop name changes its path → new knop.

    If continuity is desired, use the official move operation to create redirect metadata.

Summary

A knop local name is:

    the last path segment of a knop’s folder,

    a human-facing naming element,

    neither unique nor semantically authoritative,

    not part of intramesh identifiers,

    not an identity mechanism,

    but central to the filesystem structure and distribution naming conventions.

It is syntax, not semantics, and must be understood as such.

If you'd like, I can now produce:

    A matching glossary entry for intramesh identifier referencing this distinction.

    A diagram showing: knopLocalName ↔ path ↔ knop-handle ↔ subject-intramesh-ID.

    SHACL shapes reflecting the fact that knopLocalName is not required in RDF.


ChatGPT can make mistakes. Check important info.
