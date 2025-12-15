---
id: 6p14zpyrundnjkbvrzszlf6
title: 2025 12 14 Template Iri Vs URL
desc: ''
updated: 1765734291296
created: 1765734286826
---

Skip to content
Chat history
You said:
Should templates and stylesheet have their own (referring) IRIs (i.e., give them knops), or are they just "informational resource" IRIs. i.e., is our property "innerTemplateUrl" with a range of anyURI?  or is our property innerTemplate, with range Template/, and then a separate property for a template's IRI. 
ChatGPT said:

You’re really asking two separate questions:

    Are templates/stylesheet assets part of the mesh or just opaque files?

    Do configs point at “a file URL” or “a thing that is a template”?

Given everything you’ve been building (transposability, versioning, provenance, overrides), you get into trouble if you treat these as dumb anyURIs only.
1. Why “just a URL” is too weak here

If you do:

sfconf:innerTemplateUrl
    rdfs:domain sfconf:RenderingConfig ;
    rdfs:range  xsd:anyURI .

you immediately hit several problems:

    No version semantics: You can’t cleanly distinguish “current template” from “v2”, “v3”, etc., except by encoding that in the path.

    No provenance / license: You can’t readily attach author, license, last-modified, etc., to the template as an entity.

    Bad for transposability: Raw URLs (especially absolute) collide with your “mesh can move between hosts” story. You’d end up having to rewrite config URLs, which you’ve been trying to avoid.

    No clean override story: Platform defaults vs app defaults vs local overrides are much easier to reason about if they reference the same conceptual template, not a copy-pasted URL string.

So “URL-only” is convenient for trivial cases but doesn’t fit the rest of your design pressures.
2. Treat templates and stylesheets as first-class knops

The more consistent move with everything you’ve said about knops is:

    Yes, give templates and stylesheets their own knops.

    Use those knops as the conceptual identity.

    Let the host/CLI resolve from “template knop” → “actual file(s)” as an implementation detail.

Concretely:

### Conceptual template entity
Template/ a owl:Class .

Stylesheet/ a owl:Class .

sfconf:innerTemplate
    rdfs:domain sfconf:RenderingConfig ;  # or FolderKnop/ etc.
    rdfs:range  Template/ .

sfconf:stylesheet
    rdfs:domain sfconf:RenderingConfig ;
    rdfs:range  Stylesheet/ .

Then, on the template side:

sfconf:templateFile
    rdfs:domain Template/ ;
    rdfs:range  xsd:anyURI .   # usually relative path inside _assets or similar

sfconf:stylesheetFile
    rdfs:domain Stylesheet/ ;
    rdfs:range  xsd:anyURI .

You can later evolve templateFile/stylesheetFile into “distribution” relationships (DCAT-style) without changing the config side. The config always points to the conceptual Template/ or Stylesheet/ knop; the mechanism for finding the compiled .vto or .css is pluggable.

That gives you:

    Versioning per template/stylesheet (via your dataset series story).

    Proper attribution and licensing.

    A clean place for tags like “platform default”, “recommended for docs”, “dark-mode variant”.

    A stable identity across repos and hosts.

3. Where to put innerTemplateUrl (if you still want it)

You may still want a bare-URL escape hatch for:

    Referencing external templates you don’t control.

    Early/quick setups where you don’t want to define Template/ knops.

In that case, I’d model it explicitly as a fallback or alternative, not the primary property:

sfconf:innerTemplateUrl
    rdfs:domain sfconf:RenderingConfig ;
    rdfs:range  xsd:anyURI ;
    rdfs:comment "Direct URL to a template file, used when no Template/ knob is available." .

And define the resolution semantics in prose:

    If innerTemplate is present → use that (resolve via Template/ knop).

    Else if innerTemplateUrl is present → use that.

    Else → fall back to platform defaults.

Same pattern for stylesheets (stylesheet vs stylesheetUrl).
4. Answering your exact either/or

    is our property innerTemplateUrl with a range of anyURI?
    or is our property innerTemplate, with range Template/, and then a separate property for a template's IRI?

You should prefer:

    innerTemplate with range Template/ (which is itself a knop).

    Plus a separate property like templateFile (or eventually a distribution relationship) on Template/.

innerTemplateUrl can exist as an optional, secondary convenience, but if you elevate it to the primary mechanism you’re undermining:

    your “intramesh identifier vs deployment URL” separation,

    your versioning story,

    and your override/config-inheritance story.

If you don’t make templates and stylesheets knops, they become a weird special case in a system whose whole premise is that structured, versioned, referencable things live as knops.
You said:
should I have InnerTemplate/ and OuterTemplate/ that subclass Template/
ChatGPT said:

Short answer: probably not. Model inner vs outer as a role in a relationship, not as intrinsic subclasses of Template/, unless you have a very specific need for hard separation.
1. What “InnerTemplate/” and “OuterTemplate/” would imply

If you define:

InnerTemplate/ rdfs:subClassOf Template/ .
OuterTemplate/ rdfs:subClassOf Template/ .

you are saying:

    Some templates are intrinsically “inner” and can never be used as outer.

    Some are intrinsically “outer”.

    If you ever want the same file to be usable in both roles, you now either:

        violate your own modeling, or

        duplicate the entity as :T1-inner and :T1-outer with the same file path.

That’s a pretty rigid commitment for something that is really “this template happens to be used as an outer shell in this mapping”.

It also spreads the concept of “inner vs outer” into the ontology core, instead of keeping it where it actually lives: in the mapping/config layer.
2. Recommended pattern: one Template/, role expressed in mappings

Stick with a single template class:

Template/ a owl:Class .

Use inner/outer on the config or mapping side, not as subclasses:

sfconf:innerTemplate
    rdfs:domain sfconf:TemplateMapping ;
    rdfs:range  Template/ .

sfconf:outerTemplate
    rdfs:domain sfconf:TemplateMapping ;
    rdfs:range  Template/ .

Or if you go with your earlier idea of mapping subclasses:

TemplateMapping/ a owl:Class .

InnerTemplateMapping/
    rdfs:subClassOf TemplateMapping/ .

OuterTemplateMapping/
    rdfs:subClassOf TemplateMapping/ .

sfconf:appliesTemplate
    rdfs:domain TemplateMapping/ ;
    rdfs:range  Template/ .

Then:

    Inner vs outer is purely a role in the mapping.

    A single Template/semantic-flow/default-shell can act as outer in one mapping set and inner in another, without ontology gymnastics.

    You can still query “all outer templates used in this mesh” by following sfconf:outerTemplate or all OuterTemplateMapping/ instances.

This matches your general “knop + relators” worldview much better: inner/outer is a role in a relator-like node (the mapping), not a type of the template itself.
3. When would InnerTemplate/ and OuterTemplate/ make sense?

Only if all of the following are true:

    You know you will never sensibly reuse the same template in both roles.

    You want different SHACL constraints on “inner templates” vs “outer templates” (e.g., outer must inject <head> and <body>, inner must not).

    You care about being able to say purely at the class level “this thing is always an outer template, regardless of how anyone uses it”.

Even then, I would consider annotations or a SKOS role concept over hard subclasses, e.g.:

sfconf:templateRole
    rdfs:domain Template/ ;
    rdfs:range sfconf:TemplateRoleConcept .

sfconf:InnerTemplateRole a sfconf:TemplateRoleConcept .
sfconf:OuterTemplateRole a sfconf:TemplateRoleConcept .

But that’s still weaker and more flexible than rigid subclasses.
4. Concrete recommendation

For now:

    Do not introduce InnerTemplate/ and OuterTemplate/ subclasses.

    Keep:

        Template/ (and Stylesheet/ if you want).

        innerTemplate and outerTemplate (or separate mapping subclasses) pointing to Template/.

    Let the mapping/config layer carry the inner/outer semantics; keep Template/ generic and reusable.

If you later discover a real need for class-level separation, you can still layer it on without breaking the innerTemplate / outerTemplate properties.
You said:
They are fundamentally different in that an OuterTemplate must have a {{{InnerTemplate}}} block and an InnerTemplate can never have one. All of this is defined in config. A TemplateMapping can specify inner, outer, or both.
ChatGPT said:

Given that rule, you’re now encoding a real semantic distinction, not just a usage pattern. The important part is to be honest about the trade-offs.

You’ve said:

    OuterTemplate must contain an {{{InnerTemplate}}} block.

    InnerTemplate must not contain one.

    A TemplateMapping can reference an inner, an outer, or both.

That implies:

    No single template instance can ever be validly used as both inner and outer.

    The distinction is intrinsic to how the template is written, not just how it’s wired in.

Under that assumption, it is coherent to give them distinct classes.
1. Reasonable modeling with subclasses

I would structure it like this:

Template/ a owl:Class .

InnerTemplate/
    a owl:Class ;
    rdfs:subClassOf Template/ .

OuterTemplate/
    a owl:Class ;
    rdfs:subClassOf Template/ .

# They really are mutually exclusive:
InnerTemplate/ owl:disjointWith OuterTemplate/ .

Config side:

sfconf:innerTemplate
    rdfs:domain sfconf:TemplateMapping ;
    rdfs:range  InnerTemplate/ .

sfconf:outerTemplate
    rdfs:domain sfconf:TemplateMapping ;
    rdfs:range  OuterTemplate/ .

And then attach SHACL (or other validation) to enforce your file-content rule:

    OuterTemplateShape → sh:targetClass OuterTemplate/, checks that the template file contains {{{InnerTemplate}}} (probably via a custom function).

    InnerTemplateShape → sh:targetClass InnerTemplate/, checks that it does not.

So:

    Class membership expresses the conceptual difference.

    Shapes / build-time validation enforce the “must contain / must not contain” constraints.

    TemplateMapping can safely assume an outer really is outer-compatible.

2. The cost you’re implicitly accepting

You should be aware you’re also choosing:

    You cannot have a single “base” template that is sometimes outer and sometimes inner without cloning it as two separate Template/ individuals (one a InnerTemplate/, one a OuterTemplate/), even if they point to the same file.

    Any time you want to evolve a template from inner → outer or vice versa, you are changing its class, which may invalidate existing mappings unless you migrate them.

If that rigidity matches your mental model (“these are distinct species of template, not roles”), the subclass approach is fine.

If you later realize you do want polymorphism (same file used as inner in some contexts, outer in others), this design will fight you.
3. Alternative if you want more flexibility

If you suspect you might want that flexibility, you can push the distinction into config instead:

Template/ a owl:Class .

sfconf:templateRole
    rdfs:domain Template/ ;
    rdfs:range sfconf:TemplateRoleConcept .

sfconf:InnerTemplateRole a sfconf:TemplateRoleConcept .
sfconf:OuterTemplateRole a sfconf:TemplateRoleConcept .

Then:

    sfconf:innerTemplate / sfconf:outerTemplate still point to Template/.

    Shapes use templateRole (and/or context) to decide which constraints to apply.

    In theory, a template could have both roles (if you ever relax your “never” rule).

But this weakens the guarantee compared to disjoint subclasses.
4. Direct answer

Given your explicit rule (“Outer must have a slot, Inner must never have one”) and that you’re comfortable with that being a hard distinction:

    Yes, it is defensible to introduce InnerTemplate/ and OuterTemplate/ as disjoint subclasses of Template/.

    Back that up with SHACL to actually enforce the {{{InnerTemplate}}} presence/absence.

    Keep TemplateMapping pointing to those subclasses via innerTemplate and outerTemplate properties, not via raw URLs.

If you later find that reusing the same template as both inner and outer is desirable, you’ll have to relax either the rule or the class split.
You said:
innerTemplate or hasInnerTemplate ?
ChatGPT said:

Use innerTemplate, not hasInnerTemplate.

Here’s why, and where the hidden landmines are.
1. The core question: is this a relationship or an attribute of the mapping?

This property expresses:

    “This TemplateMapping uses this Template as its inner template.”

That is a predicate naming the role, not a generic attribute like name/label/value.
In RDF/OWL design, role-bearing predicates should be bare role names, not hasX wrappers.

Compare:

    rdfs:subClassOf — not hasSuperClass

    schema:parent — not hasParent

    foaf:knows — not hasAcquaintance

    prov:used — not hasUsedEntity

Your property is of this type: an outgoing directional relation from a mapping → a template.

Therefore innerTemplate is the correct idiom.
2. Technical reasons to avoid the “hasX” prefix here
A. You’re not modeling a containment relationship

has… semantically signals an ownership/part-whole pattern, which is incorrect here: a TemplateMapping does not “own” or “contain” the inner template. It refers to it.
B. hasX invites unnecessary inverse-property assumptions

Users of the ontology will guess there is an inverse like isInnerTemplateOf.
Unless you actually want this, don’t create that expectation.
C. It conflicts with your emerging “knop + relator” vocabulary

You’re standardizing around strong role names.
InnerTemplateMapping/ is a relator; innerTemplate is the role.
Adding has… introduces a class of predicate names that do not match the way knops and flows work.
D. hasX makes SPARQL more verbose

People will try to guess:

?map sfconf:hasInnerTemplate ?tpl .

instead of:

?map sfconf:innerTemplate ?tpl .

The latter is clearer and idiomatic.
3. When should you use hasX?

Only when:

    The relationship is structurally part-whole (e.g., schema:hasPart), or

    You have a family of symmetric containment-like predicates.

But here:

    The role is not structural.

    It's not symmetric.

    It's not part-whole.

    It’s an assignment of a role inside a configuration object.

So hasX is semantically misleading.
4. Best practice recommendation

Define:

sfconf:innerTemplate
    a rdf:Property ;
    rdfs:domain sfconf:TemplateMapping ;
    rdfs:range  sfconf:InnerTemplate ;
    rdfs:label "inner template" ;
    rdfs:comment "Specifies the inner template applied by this mapping." .

and similarly:

sfconf:outerTemplate
    rdfs:range sfconf:OuterTemplate .

This aligns cleanly with:

    sfconf:stylesheet

    sfconf:outerWrapper

    sfconf:defaultDataset

    sfconf:templateFile

and so on.
5. Strictness test

Ask yourself:

    Would naming it hasInnerTemplate add any meaning?

If the answer is no — and here it is clearly no — then the prefix is noise.
Final position

Use innerTemplate.
hasInnerTemplate is the wrong semantic signal for this kind of role-bearing property.
You said:
To allow using an older version of a template, I think we need to have a "templateUrl" property defined on Template/ - it can point to a specific distribution. Maybe we were premature in making templates  and stylesheets into Knops... because their content isn't RDF (it's a Vento or CSS file, which would live in _assets), it's not an automatic win. 
ChatGPT said:

You’re mixing two separate issues again:

    “I want to pin a mapping to an older version of a template.”

    “Does non-RDF payload mean it shouldn’t be a knop?”

Those are orthogonal. Non-RDF payload does not argue against knops; how you want to handle identity and versioning does.
1. If Template/ gets a templateUrl, what does Template/ mean?

If you put:

sfconf:templateUrl
  rdfs:domain Template/ ;
  rdfs:range  xsd:anyURI .

on a Template/ individual, you’ve implicitly done one of two things:

    Either Template/ means “this particular version of the template file”, in which case:

        Changing templateUrl is actually changing which version the IRI denotes → semantic instability.

        You have no clean way to talk about “the abstract template” vs “v1/v2/v3”.

    Or Template/ is “the abstract template concept”, but then:

        A single templateUrl doesn’t model multiple versions.

        You’re smuggling versioning into the URL string (e.g., /v1/, /v2/), which undermines your whole dataset-series story.

Right now you seem to be trying to have it both ways.
2. Non-RDF payload is not a reason to abandon knops

Your own mesh ontology already distinguishes:

    “Identifier node” (knop / payload node / dataset node).

    “Payload” (RDF dataset, or something else produced by a flow).

    “Distributions” (files in _assets, HTTP URLs, etc.).

Nothing says the payload has to be RDF. A Template/ knop can be:

    The public identity of “the Semantic Flow default outer template”.

    With a flow/dataset that describes where to find the Vento files.

    With distributions that are the actual .vto files in _assets.

If you back off from knops here just because the content is Vento/CSS, you’ll end up with a parallel, second-class system for “asset things” that doesn’t integrate with your existing versioning/provenance machinery.

That’s a much bigger conceptual smell than “this file isn’t RDF”.
3. How to support “use an older version of a template” cleanly

You have two sane patterns; what you suggested (templateUrl directly on Template/) is the bad hybrid.
Pattern A (recommended): Template + TemplateVersion

Treat Template/ as the abstract template. Introduce explicit versioned entities:

Template/ a owl:Class .
TemplateVersion/ a owl:Class .

sfconf:templateFor
  rdfs:domain TemplateVersion/ ;
  rdfs:range  Template/ .

sfconf:templateVersionUrl
  rdfs:domain TemplateVersion/ ;
  rdfs:range xsd:anyURI .  # path into _assets

# Optionally:
sfconf:currentTemplateVersion
  rdfs:domain Template/ ;
  rdfs:range  TemplateVersion/ .

Then on the mapping:

# “Follow current version”:
sfconf:innerTemplate
  rdfs:range Template/ .

# “Pin to specific version”:
sfconf:innerTemplateVersion
  rdfs:range TemplateVersion/ .

Semantics:

    If innerTemplateVersion is set → use that specific version.

    Else if innerTemplate is set → resolve via sfconf:currentTemplateVersion.

    Else → fall back to defaults.

This gives you:

    A clean place for the URL (templateVersionUrl).

    True versioning & provenance on TemplateVersion/.

    A stable concept IRI for “this template” (Template/), which is what your platform defaults and docs can talk about.

The fact that the payload is Vento or CSS is irrelevant; the metadata is still RDF.
Pattern B (minimalist): no Template/, just URLs

If you really don’t care about any of:

    version series tracking,

    provenance,

    reuse across meshes/repos,

    or querying “which mappings use which conceptual template”,

then yes, simplify brutally:

sfconf:innerTemplateUrl
  rdfs:domain sfconf:TemplateMapping ;
  rdfs:range  xsd:anyURI .

sfconf:outerTemplateUrl
  rdfs:domain sfconf:TemplateMapping ;
  rdfs:range  xsd:anyURI .

To pin to an older version, you just point to an older file path.

But then:

    No shared identity for “that template” across mappings.

    No catalog of template versions.

    No consistent story with the rest of the mesh ontology.

Given how much effort you’re putting into everything else, this “URL only” approach is inconsistent with the rest of your design, even if it’s simpler.
4. Was making templates/stylesheets knops “premature”?

Not inherently. The real mistake would be:

    Making them knops but then abusing their IRIs to point at mutable file versions (templateUrl on Template/ directly), instead of:

        Template-knop → versioned TemplateVersion-knops → asset URLs.

If you want:

    platform defaults,

    app overrides,

    the ability to query “what is the current platform outer template?”,

    the ability to pin to “that older version we used for Mesh X”,

then keeping templates and stylesheets as knops is actually the more coherent move.

What you probably need to adjust is:

    Introduce an explicit “TemplateVersion/” (or reuse your DatasetSeries/Dataset/Distribution pattern for assets).

    Hang …Url properties off the version/distribution, not off the conceptual Template/ itself.

    Let mappings either follow “current” or explicitly pin a version.

5. Concrete recommendation

If you want a crisp rule set that fits your existing architecture:

    Keep Template/ and Stylesheet/ as knops (NamedIndividuals).

    Model versions explicitly (TemplateVersion/) and hang URLs off those.

    On mappings, provide both:

        innerTemplate / outerTemplate → Template/ (follow current).

        innerTemplateVersion / outerTemplateVersion → TemplateVersion/ (pin).

    Do not put a single mutable templateUrl directly on Template/ if Template/ is meant to be conceptual and semi-stable.

If you truly don’t care about any of that machinery for templates, then admit that and demote them back to URLs entirely. The in-between “Template/ with a single templateUrl” is precisely where you’ll get the worst of both worlds.
You said:
I do love the idea of accomodating arbitrary files. But I have been assuming that Payload is always an RDF payload. "Distributions" are defined as datasets, although I guess they don't have to be. If we make payloads accomodate arbitrary (text?) files, we don't need versions, that's handled as slices via weave. 
ChatGPT said:

You’re collapsing at least three different concerns here:

    What “payload” means.

    How non-RDF files fit into the mesh.

    Where versioning lives (per-node vs per-slice vs Git).

Right now you’re trying to fix (3) by weakening (1), and that’s going to bite you.
1. Be precise about “payload”

You’ve previously been using “payload node” as:

    The canonical public identifier for an RDF dataset whose content is the data about this thing.

If you now say:

    Payload can be arbitrary text files (e.g., Vento, CSS).

you’ve silently changed the meaning of “payload” from “RDF dataset” to “opaque bytestream.”

That breaks a bunch of assumptions:

    You can no longer rely on:

        SHACL validation on payload.

        SPARQL over payload.

        “payload” = “RDF graph about the node.”

    Any code that assumes “payload = RDF dataset” now has to handle “payload = random text.”

If you really want arbitrary files, the clean way is:

    Keep Payload = RDF dataset.

    Introduce asset/distribution metadata that points to non-RDF files.

For example:

Template/ a owl:Class .

# Payload is RDF about the template (author, role, etc.)
# This stays a dataset.

TemplateDistribution/ a owl:Class .

sflo:distribution
  rdfs:domain Template/ ;
  rdfs:range  TemplateDistribution/ .

sflo:contentUrl
  rdfs:domain TemplateDistribution/ ;
  rdfs:range  xsd:anyURI .

sflo:mediaType
  rdfs:domain TemplateDistribution/ ;
  rdfs:range  xsd:string .   # "text/x-vento", "text/css"

Payload stays RDF. The Vento/CSS lives in _assets and is pointed to from the RDF.

You still “accommodate arbitrary files” without corrupting the meaning of payload.
2. “Distributions are defined as datasets” – do they have to be?

You’re mixing two senses of “dataset”:

    Semantic: a DCAT/RDF dataset (a graph of triples).

    Pragmatic: “some file we ship” (could be JSON, CSS, whatever).

You don’t have to force “every distribution is itself an RDF dataset.” You can instead:

    Say: “Distributions are described in RDF by a small dataset.”

    That RDF dataset has properties like contentUrl, mediaType, byteSize, etc.

    The thing at contentUrl may or may not be RDF.

That’s enough to:

    Keep your RDF tooling intact.

    Still treat templates/stylesheets as first-class citizens.

You don’t need to pretend the CSS file itself is an RDF dataset; you just need RDF that describes it.
3. “If payloads accommodate arbitrary files, we don’t need versions; that’s handled by slices via weave”

This is the big conceptual mistake.

You’re conflating mesh-level temporal slices with per-asset versions.

If you rely solely on mesh slices (weave) for “versions of templates,” then:

    Every template change is only identifiable via:

        “the template as of slice N”, “the template as of slice N+1”, etc.

    There is no stable “Template v1” or “Template v2” identity; there is just “whatever the template looked like at mesh-slice T.”

That breaks several use cases you already care about:

    Pinning to a specific template version
    You can’t say “use Template v1” without binding to a whole mesh slice.
    If other things change in that slice, you’re stuck.

    Cross-mesh reuse
    If another mesh wants to say “use the same template version that Mesh A used for its docs,” there’s nothing local to point at:

        You’d have to say “use Mesh A’s slice 2025-12-13T…” which couples meshes and is brittle.

    Independent evolution
    You may want to change a template without bumping other nodes.
    Slices are coarse-grained; node-level versioning is fine-grained.

So:

    Slices are good for: “state of the entire mesh at time T”.

    They are not a substitute for: “this particular template has a meaningful history of versions.”

You can, of course, choose not to have per-template versions, but then you must accept:

    No stable references to “older template versions” except by anchoring in time/Git.

    Configs can’t semantically say “this is pinned to v1”; they can only say “this is whatever the template looked like when we wove slice X”.

That’s a real capability loss, not a free simplification.
4. A consistent model that gives you all three

If you want:

    Payloads = RDF datasets (keeps your stack coherent),

    Arbitrary file types (Vento/CSS/etc.),

    And meaningful per-template versioning,

then the clean story is:

    Knop: Template (conceptual).

    Payload (RDF): metadata about the template and its distributions.

    Distributions: RDF-described resources whose contentUrl is the actual file (RDF, CSS, Vento, etc.).

    Versions: either:

        TemplateVersion knops, or

        DatasetSeries/Dataset nodes in your existing versioning scheme.

And on top of that, you still have mesh slices via weave, which give you:

    “The state of all these TemplateVersion and Distribution nodes at time T.”

So you get:

    Local, semantic versioning on templates.

    Global, temporal snapshots on the mesh.

    RDF payloads everywhere.

    Support for arbitrary non-RDF files via distributions.

5. If you really want to drop versioning for templates

Then be explicit and accept the consequences:

    Keep payload strictly RDF.

    Templates are knops with a single distribution pointing at “current” file in _assets.

    No TemplateVersion / DatasetSeries for templates.

    If you need an older version, you either:

        Check out an older Git commit / weave slice, or

        Copy the file, give it a new path, and mint a new Template.

In that world, do not pretend that slices “are” versions of templates. They are versions of the whole mesh.
Bottom line

    Don’t overload Payload to mean “arbitrary text file.”
    Keep it “RDF dataset describing the thing.”

    Support arbitrary files via distributions described in RDF.

    Slices via weave do not remove the need for per-template versions if you want stable, addressable “template vN” semantics.

    Decide:

        Either templates get real versions (TemplateVersion / DatasetSeries) and can be pinned,

        Or templates are always “whatever is current in this mesh,” and you accept that you cannot cleanly refer to old versions except through Git/slices.

You said:
Let me clear up your misunderstandings. The payload node has been a node that refers to a dataset, and that dataset is contained in a flow. The content of a payload dataset is the dataset's data. _ref containes data about the dataset. Payloads don't need validations, they can be invalid. You can still sparql query if it is a dataset. 

One of the problems with using "_assets" for non-RDF files, is they are external to weave-based versioning (via slices). If we allow HTML as payload data, then semantic flow becomes a full-blown CMS... 

I am leaning now to per-flow weave semantics. You might want to update _ref without updating _payload. 

There's no mesh-slices. There's only Flow slices. 

So if we can decouple flow slicing per weave, you get independently evolvable flows, and you can pin a specific template version by just referring to a particular payload slice. 
ChatGPT said:

Understood. With the corrected model, the picture is drastically cleaner:

    Payload node → dataset (RDF or not, content = payload)

    _ref = metadata about the dataset

    Flow = thing that evolves via slices

    Weave creates slices per-flow, not across the whole mesh

    Payloads do not need to be RDF; they can be arbitrary content

    Slicing already is the versioning mechanism

Given this architecture, your emerging conclusion is correct:

    Templates and stylesheets can be modeled as flows with payload slices, and you can refer to an older template by pointing to the slice you want.

Let’s examine this precisely and critically.
1. If payloads can be arbitrary content, why keep _assets at all?

You’re right that _assets become second-class citizens:

    They are not sliced

    They do not participate in version history

    They cannot be pinned

    They live outside the flow machinery

This breaks:

    reproducibility of template behavior

    determinism in weave

    the ability to reference “Template version X” as a stable thing

Thus, if a template is important enough to reference, it should not live in _assets.
It should live as the payload of a Flow.

That gives you:

Template/
  _payload/
    slice1/
      template.vto
    slice2/
      template.vto
    ...
  _ref/
    sliceN/
      metadata.ttl

Everything is now versioned via slices.
No separate versioning layer.
No DatasetSeries.
No TemplateVersions.

And everything (template content + metadata) is truly part of the mesh.
2. CMS question: does allowing arbitrary payloads make Semantic Flow a CMS?

Only if you start rendering arbitrary user HTML as your public pages.
If you keep:

    payload = internal data (template text, CSS text, markup fragments, etc.)

    resource pages = still generated by the engine

then you have not become a CMS.

You’ve simply made flows capable of carrying their own configuration artifacts.

This is no different from:

    Hugo putting templates inside the repo

    Jekyll putting layouts inside the repo

    Docusaurus putting MDX inside the repo

Except you have better formal semantics.

So: no, not a CMS, unless you choose to treat arbitrary payloads as literal publication content.
3. Per-flow slicing gives you exactly what you need

The core win:

    Each Flow evolves independently.
    You can update _ref without touching _payload.
    You can update _payload without touching _ref.

That yields:
Use case A: Update metadata only

_flow/my-template/_ref/slice4/metadata.ttl

Use case B: Fix template text, preserving metadata

_flow/my-template/_payload/slice5/template.vto

Use case C: Pin an older template version

Your TemplateMapping simply points to:

<.../my-template/_payload/slice3>

or the generated convenience IRI:

my-template.payload[slice=3]

This gives you:

    reproducibility

    rollback capability

    multi-template workflows

    ref-updating without payload-updating

All with no new versioning machinery.
4. Should Templates and Stylesheets still be “knops”?

Yes—strongly yes.

Because:

    They have identity

    They have provenance

    They have flows

    They have payload slices

    They have _ref slices

    They can be referenced from mappings

    They participate in weave semantics

The fact that their payload is not RDF is irrelevant.

A knop is an identity-bearing node, not a dataset.
5. SHARP distinction to keep:
Payload slice = the template content itself (Vento/CSS/etc.)
Ref slice = metadata (author, role, encoding, media type, license, etc.)

DO NOT mix them.

Why?
Because you want to update _ref (e.g., change the template’s role from Inner to Outer) without creating a useless new content slice.
6. Modeling: what does a TemplateMapping refer to?

You now have two options, you must choose explicitly:
Option A — Mapping points to the flow itself (and uses its current payload slice)

sfconf:innerTemplate → Template/

Semantics:

    Mapping always uses the current slice of that flow’s payload.

Good for “always use latest”.

Bad for reproducibility.
Option B — Mapping points to a specific payload slice

sfconf:innerTemplateSlice → TemplatePayloadSlice/

or even:

sfconf:innerTemplate → <template-flow/_payload/slice5>

Semantics:

    Fully pinned, totally reproducible.

You can support both:

    innerTemplate (unversioned → use latest)

    innerTemplateSlice (version-pinned)

Exact parallel to “dependency vs dependency-lockfile”.
7. Critical question: Do you actually want DatasetSeries anymore?

Given your clarified model:
No. You don’t.

Flow slicing gives you:

    per-artifact versioning

    immutable historical snapshots

    natural branching

    independent evolution

    no need for dataset duplication

DatasetSeries was a patch to express “versioning” before you refined the flow-slice semantics.

It now looks redundant for Templates, Stylesheets, and similar things.

It may still have a role for RDF datasets that are themselves primary artifacts (like ontology files), but you could even unify that if you want.
FINAL: The cleanest, most coherent architecture
1. Every significant conceptual thing = a knop

(including Template, Stylesheet, OntologyFile, etc.)
2. Every knop has a Flow

(two flows if needed: _payload and _ref)
3. Payload flow slices contain the raw file content, RDF or not

This fully replaces _assets.
4. Ref flow slices contain metadata about the content

Role = Inner, Outer, mediaType, encoding, notes, etc.
5. TemplateMappings choose between:

    following the latest slice of the Template’s payload flow

    or pinning a specific slice

6. No mesh-level slicing; slicing is flow-local

Flows evolve independently, making config-driven pinning easy and stable.
This is much more elegant than anything involving _assets, DatasetSeries, or external version folders.

It gives you:

    true reproducibility

    no separate versioning system

    correct semantics for updating ref vs payload

    arbitrary content payloads without CMS creep

    easy pinning of template versions

You have converged toward the minimal and most logically consistent model.
You said:
_assets could be for binary files. Although I guess binary files could be distributions.

Just FYI, versioned (and deversioned) flows, i.e. FlowWithVersions/ are subclass DatasetSeries. It's not incompatible with the flow-slice semantics.

Yes, I am saying the payloads, whether markdown files or raw HTML or just text, could be woven via specific templates (or in the case of raw HTML, no templates at all), into resource pages. Presto: CMS.

Distributions are the actual content. A payload slice IRI refers to that distribution "in the abstract". 
ChatGPT said:

Good, this clarifies a lot, but you’ve also re-introduced a couple of contradictions. I’ll point them out and then give you a coherent model that matches what you just said.
1. You just changed what a “payload slice” is (again)

You said:

    Distributions are the actual content. A payload slice IRI refers to that distribution "in the abstract".

Earlier in this same thread you had:

    payload slice = possibly non-RDF content (Markdown, HTML, etc.)

    _ref slice = metadata

Now you’re saying:

    “Distributions are the actual content”

    payload slice IRI is abstract, referring to that distribution

These two pictures can’t both be true. You need to pick one of:

    Payload slice = content (RDF or not)
    Distribution is either not a separate node, or is just a synonym for “that content”.

    Payload slice = RDF description of a dataset, whose content lives in separate Distribution nodes (with URLs, mediaType, etc.).

Given your “payload node has been a node that refers to a dataset, and that dataset is contained in a flow” and “you can still SPARQL query if it is a dataset”, (2) is closer to what you’ve historically meant. Right now you’re mixing the two.
2. What you just said implies a very clean, DCAT-like model

If I take your latest sentences at face value:

    “Distributions are the actual content.”

    “Payload slice IRI refers to that distribution in the abstract.”

    Flow slices are per-flow, not mesh-wide.

    FlowWithVersions ⊑ DatasetSeries.

Then the sensible model is:

    A FlowWithVersions is a DatasetSeries: each slice is one version of the dataset.

    Each payload slice’s dataset contains RDF like:

    :payloadSlice_3 a sflo:PayloadSlice ;
        sflo:describesDataset :templateDataset ;
        sflo:hasDistribution   :templateDist_v3 .

    :templateDist_v3 a sflo:Distribution ;
        sflo:contentUrl  "_assets/templates/foo_v3.vto" ;
        sflo:mediaType   "text/x-vento" .

    The actual bytes (Vento, CSS, binary, whatever) live in _assets/....

So:

    Payload slice = “abstract version of the dataset” (RDF).

    Distribution = “actual content for that version”.

    _assets = physical storage for distribution content, not second-class, just the filesystem target.

That fully respects:

    Your existing “payload = dataset” assumption.

    Your desire for arbitrary content (by putting it behind distributions).

    Your statement that “Distributions are the actual content”.

    Your flow-slice version semantics.

Right now you’re drifting toward that without saying it explicitly.
3. Where _assets and binaries fit

    _assets could be for binary files. Although I guess binary files could be distributions.

You don’t need _assets to be conceptually special at all:

    _assets is just a conventional root for files that sflo:Distribution points to via sflo:contentUrl.

    Binary vs text doesn’t matter. The RDF says mediaType, and the engine handles it.

Versioning story:

    The RDF (payload slices, distributions) is versioned via FlowWithVersions.

    The bytes in _assets are versioned by Git and by path convention:

        e.g. each distribution uses a versioned path or a content hash in the filename.

If you want stricter guarantees, you can:

    put a content hash into the RDF metadata (e.g. sflo:contentSha256),

    so a payload slice + distribution + hash together define an immutable version.

That still keeps DatasetSeries + flow slices intact.
4. CMS angle: you’re right, this does make SF a CMS if you want it to

    payloads, whether markdown files or raw HTML or just text, could be woven via specific templates [...] into resource pages. Presto: CMS.

Correct. With this model, you can make SF a CMS:

    Content flows (Markdown, HTML, JSON, whatever) → payload flows.

    Template flows (Vento) → payload flows.

    TemplateMappings say which content flow uses which template flow slice.

    Renderer resolves:

        content payload slice → its Distribution → _assets/.../post.md

        template payload slice → its Distribution → _assets/.../template.vto

        runs them together → resource page.

Whether you treat that as “full CMS” or “publishing framework that can do CMS” is mostly branding. Architecturally, you’ve got all the pieces.
5. Pinning versions using flow slices

Given there are no mesh-slices, only flow slices, and FlowWithVersions ⊑ DatasetSeries, the pinning story is straightforward:

    Each FlowWithVersions has its own local slice index / ID.

    A TemplateMapping can refer to:

        the Template flow itself (meaning “current slice”), or

        a specific payload slice IRI (meaning “that version”).

To make this precise, you probably want:

sfconf:innerTemplate
    rdfs:range sflo:TemplateFlow .          # follow current payload slice

sfconf:innerTemplatePayloadSlice
    rdfs:range sflo:PayloadSlice .          # pin to exact version

where:

    sflo:TemplateFlow is a FlowWithVersions for templates.

    sflo:PayloadSlice is a class of slice nodes in that flow.

That matches exactly what you just said:

    you can pin a specific template version by just referring to a particular payload slice.

Yes, as long as “payload slice” is a first-class node with a stable IRI and not just an ephemeral internal index.
6. The subtle inconsistency you still need to resolve

The one thing that’s still muddy in your story is this:

    Is a “payload slice IRI” the dataset version (RDF that describes distributions),
    or is it the distribution in the abstract (which would make “distribution” vs “payload slice” almost redundant)?

Right now you’re talking as if:

    Payload slice IRI = abstract dataset version.

    Distribution = actual content.

That’s coherent. If that’s what you want, lock it in and write it down explicitly:

    A payload slice IRI denotes an abstract dataset version whose primary content is provided by one or more sflo:Distribution resources. The distributions are the actual content (files); the payload slice is the versioned descriptor.

If instead you mean:

    Payload slice IRI is the distribution “in the abstract” and the Distribution node is optional,

then you’re weakening the dataset semantics and you’ll get into trouble when you want more metadata.

Right now the first reading is cleaner and aligns better with “FlowWithVersions is a DatasetSeries”.
Recommended stance

Given everything you’ve just clarified, the most consistent position is:

    Keep “payload = dataset” as you asserted.
    Payload slices are dataset versions.

    Make distributions the place where “actual content” lives (with URLs into _assets).

    Treat FlowWithVersions as the concrete implementation of DatasetSeries.
    Each slice = one dataset version.

    Allow pinning by referencing payload slice IRIs directly from TemplateMappings.

    Don’t overload “payload slice = raw file content”.
    Let it be an RDF descriptor that points to a file-backed Distribution.

If you write those five bullets into your sflo docs, your template/story will stop wobbling.
You said:
I don't think we need separate properties for inter-template payload slice versus inter-template, period. We should just provide the URL to the distribution which can either include current or a version slice.
ChatGPT said:

This is workable, but it’s a regression in semantic clarity unless you’re very disciplined about what that “URL” is allowed to be.

You are proposing:

    TemplateMapping stores one value: a URL that resolves to “the distribution,” and that URL may encode either:

        “current” (deversioned), or

        a specific version/slice.

That can be made consistent, but it has sharp edges.
1. If you use only a URL, you’re choosing “addressing scheme = semantics”

Because the mapping no longer says:

    “this is a Template/ thing” (conceptual identity)

    “this is the pinned version” (version identity)

It says only:

    “fetch whatever is at this URL”

So the entire meaning becomes path-convention-dependent. That’s okay if and only if your URL design is stable and you accept that the ontology cannot express intent beyond that.

If you later want to answer “what template is this mapping using?” you now have to parse URLs or rely on inference from directory structure.

That’s a big downgrade relative to knops.
2. You’re also implicitly redefining what “distribution URL” points to

You said:

    provide the URL to the distribution which can either include current or a version slice.

In your own terms, a payload slice IRI refers to a distribution “in the abstract.” That suggests two different address types:

    Distribution identity IRI (abstract, describable in RDF)

    Content URL (a dereferenceable file location)

If your mapping stores “URL to the distribution”, which one is it?

You need to pick one and enforce it:
Option A (recommended): mapping points to a distribution identity IRI

Example:

    …/template/_payload/_dist/_current

    …/template/_payload/_dist/slice/5

Then the engine resolves:

    distribution IRI → distribution description (RDF) → contentUrl → actual bytes.

This keeps the semantic door open: distributions can have metadata, hashes, media type, etc.
Option B: mapping points directly to the content URL

Example:

    …/_assets/templates/foo.vto

    …/_assets/templates/foo.vto?ref=slice5

This throws away distribution metadata unless you reconstitute it elsewhere. It also makes it harder to guarantee immutability (hashes, mediaType) and provenance.

Given your earlier “Distributions are the actual content”, you’re leaning Option B, but Option B makes “distribution” not really an RDF-described thing anymore—just a file.
3. If you do this, you should drop the “Template/ knob” idea for mappings

If mappings store only distribution URLs, then Template/ knops become less valuable for mappings (they may still be useful as organizational nodes).

Because:

    The mapping no longer references the conceptual template identity.

    Therefore you lose queryability and reuse based on template identity.

If you keep Template/ knops, you should prefer referencing them from mappings, not bypassing them.

This is the key inconsistency: you can’t simultaneously say “templates are first-class knops” and then make the primary linkage ignore those knops in favor of raw URLs.
4. Minimal coherent design if you insist on “one property = URL”

If you want one property only, do it like this:

    The property is not “templateUrl”. It is templateDistribution (or similar).

    Its range is IRI (not “URL string”), and the IRI denotes a Distribution node (abstract identity), which may be _current or slice/N.

Example:

sfconf:innerTemplateDistribution
  rdfs:domain sfconf:TemplateMapping ;
  rdfs:range  sflo:Distribution .

and allow values like:

    <…/template/_payload/_dist/_current>

    <…/template/_payload/_dist/slice/5>

Then the distribution resource (RDF) provides:

:dist a sflo:Distribution ;
  sflo:contentUrl "_assets/templates/foo.vto" ;
  sflo:mediaType "text/x-vento" ;
  sflo:sha256 "…" .

You still only have one mapping property, but you haven’t reduced your system to “random URL string”.

If you instead store contentUrl directly in mappings, you’ve abandoned the distribution layer and should admit that.
5. What you gain vs lose
Gain

    Simple config surface area: one property.

    Pinning is path-based: _current vs slice/5.

Lose (unless you choose Option A above)

    Ability to attach metadata to distributions without indirection.

    Ability to ask “which Template knob is this?” without parsing URLs.

    Ability to validate “inner vs outer” at the ontology level (you can still do it at build-time).

Recommendation

If you want “one property” and you want the rest of your semantic machinery to stay coherent:

    Use one property that points to a Distribution IRI, not to raw file URLs.

    Encode current vs pinned as part of the Distribution IRI (_current vs slice/N).

    Keep distribution metadata (mediaType, hash, contentUrl) in _ref (or in a distribution-descriptor dataset).

If you instead store raw _assets/... file URLs in mappings, you should stop calling them “distributions” and acknowledge you’ve chosen a purely file-addressed system for templates.
You said:
Yeah, I think we should just drop template, and the template mappings can refer to any file. Not just a file that's been integrated into a mesh or a NOP somewhere, period. I also think maybe we can abandon slug and we just use whatever file name is in working to cut the version. This makes things less predictable, but if you have to look up slug anyhow, you can just look up in meta what the actual file is. And this allows, for example, file extension change, e.g. from a JPEG to a PNG. It's the same image, just represented differently. We drop supplying multiple RDF syntaxes as a convenience. People can just translate them if they want. And therefore every slice folder has two files, an index.html resource page and whatever the actual distribution file is.
ChatGPT said:

Dropping Template/ and letting mappings point to “any file” is internally consistent, but the moment you also drop slug and make slice cutting depend on “whatever file name is in working,” you create a fragile system with avoidable ambiguity. You’re trading away referential stability for convenience in a way that will leak everywhere.

Below are the concrete failure modes, then a tighter variant that preserves your goals (any file, extension changes, two files per slice) without turning the system into guesswork.
1. If mappings can refer to any file, you need a stable addressing scheme

If a mapping is allowed to point to arbitrary files (inside or outside the mesh), then:

    you cannot rely on “knop identity” or “flow membership” to recover provenance or ensure reproducibility

    “pinning” becomes entirely dependent on file paths and conventions

That’s fine only if you commit to one of these:

    (A) mappings always point to a slice-stable distribution IRI (or path) that never changes, or

    (B) mappings point to external URLs, and you accept you can’t reproduce builds unless you snapshot those bytes into a slice

Right now you’re proposing neither explicitly; you’re proposing “any file, period,” which implies external references are allowed. That will produce non-reproducible builds unless you require snapshotting.

If you adopt “any file,” you should also adopt a rule:

    If a referenced file is not already slice-addressed, weave must copy it into the slice (or record an immutable content hash and fail if it changes).

Otherwise pinning is a lie.
2. Dropping slug and using “whatever filename is in _working” is a trap

You are trying to use “filename” to represent identity of the distribution across time. Filenames are not stable identifiers; they are mutable labels.

Concrete breakages:

    Rename creates accidental “new artifact”: If a user renames template.vto → template-main.vto, you’ve created an apparent semantic change even if bytes are identical.

    Multi-file working directories: What if _working contains index.html, template.vto, README.md, thumb.png? Which one is “the distribution file”? You need a selection rule. “Whatever file name is in working” is not a rule.

    Ambiguous extension changes: You want image.jpg → image.png to be “same image, different representation.” With no stable identifier, you cannot express “same thing” except by manually reading _meta. That makes automation brittle.

You said: “if you have to look up slug anyhow, you can just look up in meta what the actual file is.”
That admits the truth: you still need a stable pointer. You’re just moving it from slug to “meta says which file.” That’s not simplification; it’s displacement.

If you drop slug, you must introduce a single authoritative pointer in _meta anyway.
3. “Two files per slice” is clean, but your proposed mechanism won’t reliably achieve it

You want each slice folder to contain:

    index.html resource page

    “the actual distribution file”

This is achievable, but only if the system can reliably answer:

    What is the distribution file for this flow/slice?

If you don’t have slug, you need an explicit property such as:

    primaryDistributionFilename

    primaryArtifact

    mainFile

Otherwise you’ll end up with heuristics like “the file that isn’t index.html,” which fails as soon as you add any other supporting file.
4. Dropping “multiple RDF syntaxes” is coherent, but it changes your published contract

If you stop shipping multiple serializations, you’re saying:

    your published artifact is one canonical syntax (e.g., Turtle or JSON-LD)

    clients are responsible for translating

That’s OK, but it means:

    any “content negotiation” expectations are gone (unless you implement it virtually)

    your “distribution” layer still matters, because you need to declare what format you published

If you also drop Template/ and make mappings point directly to files, you are implicitly drifting away from RDF-described distributions, which will make format declaration and toolchain behavior more ad hoc.
5. The JPEG→PNG argument is valid, but your conclusion doesn’t follow

You want to support “same image, new encoding.” That’s a representation change, not an identity change.

The right way to model that is:

    stable identity for the asset

    multiple representations over time (or side-by-side)

Dropping stable identity (slug) makes this harder, not easier. You end up with “whatever file happens to exist now,” plus a meta lookup to figure out what happened.

You don’t need slug specifically, but you need something stable.
A tighter variant that preserves your goals

You can get almost everything you want (any file, extension changes, two files per slice, no Template/) with one rule:
Keep a stable “artifact key” separate from filenames

    Each flow has an artifactKey (stable across slices) stored in _meta.
    Example: main, primary, payload, or a UUID-ish short key.

    Each slice contains exactly:

        index.html

        ${artifactKey}.${ext} (or ${artifactKey} with no extension if you insist)

When the file type changes (jpg→png), only the extension changes; the artifactKey stays the same.

This avoids slug while keeping determinism.

If you want to keep the user’s original filename visible, store it in _ref as metadata, but don’t let it be the identifier.
Minimal metadata requirement

In _meta (or flow config), keep one pointer:

    primaryArtifactKey: "main"

    optionally currentExt: "png" (or infer per slice)

If you refuse even that, you will reintroduce a new “slug” implicitly as the first non-index file you find, which will eventually break.
Mapping property naming under this new model

If mappings refer to “any file,” the property should not be innerTemplate at all. It should be a generic “renderer input” reference, e.g.:

    contentSource / sourceArtifact / inputArtifact

And it should point to either:

    a slice-addressed path/IRI (preferred), or

    an external URL with a rule that weave snapshots it into a slice.

Bottom line

    Dropping Template/ is fine if you accept you’re moving to a file-addressed pipeline.

    Dropping slug and using “whatever filename is in working” is not fine: it’s non-deterministic and will force heuristics or manual lookups everywhere.

    If you want extension changes and exactly two files per slice, you still need one stable key for the “main artifact,” even if you don’t call it slug.

If you proceed with your current proposal unchanged, you will end up reintroducing a “slug-like” field under a different name because the system cannot be deterministic without it.
You said:
I think non-reproducible builds are fine if someone updates an external template file. Then that's the risk you take by not referencing a template file that's in a version slice that is supposed to be immutable. You do lose being able to look up provenance directly from the URL. But if the URL has underscore payload, then you can assume that the parent IRI... ...will return a resource page when dereferenced. Period. Renaming the file is fine. You might even think of use cases where you need to rename the file for some reason. The thing that gives it identity is the NOP IRI. If working contains multiple files, it's an error. The stable identifier is the NOP IRI. Thank you for watching.
ChatGPT said:

You’ve now specified a coherent rule set, but a few parts are still underdefined or self-contradictory in ways that will cause real implementation and UX problems. Here’s the tightest reading of what you said, followed by the exact places you still need to nail down.
What you have (coherent core)
A. Two classes of references

    External file URL

        Non-reproducible by design if upstream changes.

        Accepted tradeoff.

    Slice-contained “immutable” distribution

        If the URL contains /_payload/ (or some canonical marker), it’s understood to be slice-backed and intended immutable.

B. Identity is not the filename

    Filename in _working is mutable.

    Stable identity = NOP IRI (flow identity).

C. Working directory invariant

    _working must contain exactly one “distribution file” (plus the generated index.html is not in working; or if it is, it’s excluded).

    If multiple candidate files exist, error.

This is consistent and implementable.
The remaining sharp edges you’re glossing over
1. “If URL has /_payload/, assume parent IRI dereferences to resource page”

This is an inference rule based on path structure. It can work, but it’s a contract you must formalize:

    What exactly counts as a “payload URL”?

        .../flow/_payload/slice/7/foo.css ?

        .../flow/_payload/_current/foo.css ?

        .../flow/_payload/foo.css ?

If you don’t freeze a canonical structure, you’ve just created another implicit slug system—only this time it’s implicit in path conventions.

Minimum needed: a single normative grammar for slice-backed distribution URLs.
2. “The stable identifier is the NOP IRI” vs “TemplateMappings can refer to any file”

If mappings can point to external URLs, then many mappings will not mention the NOP IRI at all.

So either:

    (A) You accept that the stable identifier exists only when you choose to use slice-backed URLs, or

    (B) You require mappings to always point at a NOP-relative IRI even for external content (e.g., an “imported external distribution” node).

Right now you’re saying both “stable identifier is NOP IRI” and “any file URL is allowed.” Those conflict unless you explicitly scope the identity claim to internal references.
3. “Renaming is fine”

Renaming is fine only if your “one file in working” rule doesn’t accidentally make rename equivalent to changing what version means.

Concretely: when you cut a slice, what is the slice path?

    If the slice folder contains whatever-the-filename-was-at-the-time.ext, then the slice’s distribution URL changes when the filename changes across slices. That’s okay if you accept that pinning is always to a specific slice+filename.

    But you just said the stable identifier is the NOP IRI. If you want stable identity for the “distribution in the abstract”, you need a stable intra-flow identifier, not just the flow/NOP. Otherwise “the distribution” is not a stable thing; only “the flow” is.

This is not fatal, but you should admit it: you have stable flow identity, not stable distribution identity.

If you’re okay with that, fine—just don’t claim otherwise later.
4. “Working contains multiple files ⇒ error”

Good, but you must define what counts as “a file”.

Obvious gotchas:

    .DS_Store, Thumbs.db

    editor swap files (.swp, ~)

    source maps (.css.map)

    sidecars (.md + .png)

You can keep the rule and still be sane by defining an allowlist/ignorelist. If you truly mean “any extra file is an error,” you’re going to punish normal tooling.
What I think you should write down as the spec (minimal, no extra ontology)

If you want to keep your current direction, you need these four normative rules in the docs:

    TemplateMapping targets are URIs of concrete files.
    They may be:

        external (https://…/foo.vto)

        internal slice-backed ({nopIri}/_payload/slice/{id}/{filename} or equivalent)

    Slice-backed URLs have a canonical grammar.
    If a URL matches that grammar, it is intended immutable and its parent NOP IRI is dereferenceable to a resource page.

    Flow identity is the stable identifier.
    The identity-bearing thing is the NOP/flow IRI. Filenames are not identifiers.

    Working-directory cut rule.
    To cut a payload slice:

        _working must contain exactly one non-ignored file.

        that file is copied into the new slice folder alongside index.html.

        if more than one candidate exists, weave fails with an explicit error.

If you adopt exactly that, your “Thank you for watching” stance is defensible.
The one thing I’d still push back on (hard)

If you truly allow external templates and accept non-reproducibility, you still need one safety valve:

    either record a content hash at weave time (in _ref) so users can detect drift later, or

    explicitly say “we do not even attempt to detect drift.”

Not doing either will create silent behavior changes that look like bugs.

Given your “risk you take” framing, hashing is consistent: it doesn’t prevent the risk; it makes it observable.

If you want, I can translate the above into a crisp “TemplateMapping resolution and slice-cutting spec” (a page you can drop into your docs) using your current terminology (NOP, _payload, _working, slice).
You said:
I think working slices just don't have resource pages. It's the one exception to every IRI's dereferenceable, but it makes sense. Resource pages are generated on Weave. And any resource page generated for working would be outdated, potentially, as soon as working changed. Can you turn off .DSSTORE? It's so annoying. I almost want to punish Mac users for supporting an operating system that does that. Recording a content hash is not a bad idea. Does it make sense to do it for current? And that file hash would be constantly changing.
ChatGPT said:
Working slices without resource pages

That’s coherent. Treat _working as explicitly non-dereferenceable (or dereferenceable only to a minimal “working is mutable” stub), and only generate resource pages at weave time for immutable slices.

If you don’t do even a stub, you should formalize the exception:

    “All slice IRIs are dereferenceable except _working IRIs, which are mutable staging areas and may 404 or return a minimal status page.”

Otherwise tooling and users will assume “broken link” when it’s actually “by design.”

A minimal stub is still useful and not “outdated” if it contains only invariant facts:

    “This is _working (mutable). No resource page is generated. Run weave to publish a slice.”

    Optionally: last modified time, and the currently selected candidate file name (not content-derived facts).

That stub can be generated cheaply and doesn’t become semantically stale in the same way a full page would.
Ignoring .DS_Store and other junk files

Yes. This should not even be optional; it should be part of your default candidate-file selection logic.

Recommended default ignore patterns:

    .DS_Store

    Thumbs.db

    Desktop.ini

    ._* (AppleDouble)

    *.swp, *.tmp, *~

    .gitkeep

And also ignore directories like:

    __MACOSX/

Implementation-wise: the candidate “single file in working” rule becomes:

    Count only non-ignored regular files. Exactly one must remain.

If you want “punish Mac users,” do it socially, not in the compiler: silent footguns are worse than .DS_Store.
Content hashes: do they make sense for _current?

You’re right that “hash for current” changes whenever the current file changes. That is not a problem if you’re clear about what it means.

You actually have two different hashing use cases:
1) Working/current drift detection (mutable)

Store on _working/_current:

    sha256 of the current bytes

    hashedAt timestamp

This is useful for:

    quickly detecting “did the file change since last time I looked?”

    caching

    logging and debugging

It is not meant to be stable.
2) Immutable slice integrity (stable)

Store on version slices:

    sha256 of the slice content (must never change)

    optionally byteSize, mediaType

This is the important one for reproducibility and auditability.

So: yes, hashing “current” can make sense, but only as an ephemeral status field. Hashing slices is the contractual integrity guarantee.
Critical edge you still need to decide

When you weave, do you:

    (A) copy _working content into a new slice and compute the slice hash (recommended), or

    (B) treat _working as the slice and just “mark it published” (bad idea; breaks immutability)

Your current direction implies (A). If so, the workflow is:

    _working changes freely (no resource page)

    weave copies the one candidate file into slice/N/filename.ext

    weave generates slice/N/index.html

    weave records stable sha256 for slice N (and optionally records the last _working hash you cut from)

Suggested minimal metadata fields

    For _working:

        sflo:candidateFilename

        sflo:sha256 (optional, mutable)

        sflo:lastModified (optional)

    For slice/N:

        sflo:filename

        sflo:mediaType

        sflo:byteSize

        sflo:sha256 (strongly recommended, stable)

This keeps “hash for current” useful without pretending it’s an identity primitive.
You said:
If we allow filenaming or renaming in working, then weaving will break current URL references. Maybe that's just a hazard we accept. If you want something immutable, you need to turn on versioning. The other possibility is we don't keep a distribution in current, just a resource page that points to the current distribution. But then we'd have to rethink unversioned and deversioned flows. Period. Maybe we just get rid of them. And if people want mutability, they can keep their RDF in an assets folder. We could add a snapshot retention config item that automatically removes old snapshots. Period. That breaks immutability too. Maybe some people will need that.
ChatGPT said:

You’ve identified the real fork. Right now your design goals are mutually incompatible unless you choose which invariant you care about:

    Stable “current” URLs (good DX, fewer broken links)
    vs

    Filename mutability in _working (flexibility, extension changes)
    vs

    Immutability guarantees (auditability, reproducibility)

You can’t have all three without adding an extra indirection layer.
1) Accept the hazard: current is unstable

If you keep “_current contains the distribution file itself,” and you allow renames, then:

    .../_payload/_current/foo.png breaks when foo.png becomes bar.png.

You can accept this and document it as a contract:

    _current is mutable and unstable; do not deep-link to filenames under _current. Use version slices for stability.

This is coherent and keeps your system simple.

But it conflicts with your earlier instinct that “if URL has _payload you can assume parent dereferences to a resource page.” That rule still works; what breaks is deep-link stability.

If you choose this, you should add a best practice:

    Prefer linking to the resource page (.../_current/) not the file (.../_current/foo.ext) unless pinned.

2) Make current stable by removing the filename from the stable URL

The cleanest solution is: don’t put the mutable filename in the addressable “current distribution” IRI.

Instead:

    _current/index.html exists (resource page).

    The actual current file is stored with any name you want, but is not the stable target.

    The resource page contains the link to the current distribution file.

If you also want a stable machine target, you add a canonical alias path:

    .../_current/_dist (no extension) or .../_current/_dist.bin (generic)
    that redirects/serves the actual file.

This gives you:

    stable “current distribution URL” independent of filename

    filename mutability

    still simple

This is an indirection layer, yes—but it’s the minimal indirection needed to reconcile your goals.

If you refuse indirection, you must accept broken links.
3) “If you want something immutable, you need to turn on versioning.”

That’s the right principle, but then you shouldn’t weaken it with “retention deletes old snapshots,” because:

    deleting old slices doesn’t merely “break immutability”

    it breaks referential integrity: any pinned reference can rot

If you add retention, you must treat it as garbage collection with explicit semantics:

    “This system may delete old slices; you may not reference them.”
    That is not immutability—it’s checkpointing.

Which is fine, but call it that.
4) “Maybe we just get rid of unversioned/deversioned flows”

This is likely the correct simplification.

If you keep unversioned/deversioned flows, you are implicitly promising:

    stable dereferenceable identifiers without the slice history machinery

But you keep rediscovering that mutability + stability requires indirection.

So you have two consistent endpoints:
Endpoint A — Everything is versioned (recommended)

    All payload flows are FlowWithVersions (DatasetSeries).

    _working is mutable staging; no stable file URLs.

    _current is either:

        just a resource page pointer to latest slice, or

        an alias to latest slice.

    Stability comes from slices.

This keeps your ontology and semantics clean.
Endpoint B — “Mutable mode” is explicitly outside the mesh

    If people want mutability, they keep files in _assets (or outside SF entirely).

    SF resource pages can link out, but SF does not claim immutability or stable addressing for that content.

This is clean too, but you lose the “SF manages the content” story.

Right now you’re oscillating between these; pick one.
5) If you go “all versioned,” the “current URL reference” problem disappears

Because “current” becomes a pointer, not a container of the distribution file:

    .../_payload/_current/ → redirects to .../_payload/slice/57/ (or serves a page that links there)

    “latest slice” is stable enough for humans; pinned links use slice numbers.

Then renaming in _working doesn’t break any canonical “current distribution URL,” because there isn’t one.
6) Snapshot retention: only safe if it’s opt-in and explicit in the URL contract

If you implement retention:

    It must be off by default if you still claim immutability.

    If enabled, it must also modify your contract:

        “Older slices may be collected; pinning is best-effort.”

    Ideally, you provide:

        a minimum-retained window (keepLast: N or keepDays: D)

        a “protected tags” mechanism (don’t delete slices referenced by a release marker)

Otherwise users will assume a pinned slice is permanent when it’s not.
My recommendation given everything you’ve said

    Drop unversioned/deversioned flows.

    Make everything FlowWithVersions (DatasetSeries).

    _working = mutable, not dereferenceable (or stub only).

    _current = dereferenceable pointer to latest slice (resource page and/or redirect), not a file container.

    Allow optional retention, but label it as “checkpointing/GC,” and make it opt-in with loud semantics.

This resolves:

    filename mutability

    stable “current”

    immutability (when retention off)

    pinning via slice IRIs

If you refuse pointer indirection for _current, then accept that _current deep links are unstable and document it bluntly. That is also coherent; it’s just worse DX.
You said:
I thought about doing a different flow, like dist, that has an immutable filename, period. But we can't get away with no extension or generic because we're targeting GitHub pages hosting. So there's no fancy redirection. The only thing we have is folder URLs returning index.html resource pages. We can't get away with no extension or generic because we're targeting GitHub pages hosting. We can't get away with no extension or generic because we're targeting GitHub pages hosting. We can't get away with no extension or generic because we're targeting GitHub pages hosting. We can't get away with no extension or generic because we're targeting GitHub pages hosting. We can't get away with no extension or generic because we're targeting GitHub pages hosting. We can't get away with no extension or generic because we're targeting GitHub pages hosting. Okay. I don't see endpoint A and endpoint B as incompatible. I don't see endpoint A and endpoint B as incompatible. Was Owies quote unmanaged unquote by semantic flow? We can't have current redirect. We can only have it serve a page that links. Because of the aforementioned GitHub pages limitations.
ChatGPT said:

You’re right about the GitHub Pages constraint in the important way: you should assume no server-side redirects and that the only reliable “pretty” dereferenceable thing is a folder URL that serves index.html. So “_current as an alias that serves the bytes of the current distribution” is off the table. The only viable “current” is a resource page that links to the current file.

That forces a specific architecture choice.
1) A and B are compatible only if you draw a hard boundary

Endpoint A (versioned, slice-backed immutability) and Endpoint B (mutable, unmanaged assets) can coexist only if you codify:

    Managed content: lives in flows/slices; weave produces new slice folders; immutability is promised (unless retention is enabled).

    Unmanaged content: lives in _assets or external URLs; weave does not promise reproducibility; links may drift.

The incompatibility happens when you try to make something both managed and unmanaged at once (e.g., treat _current distribution files as stable while also allowing renames).

So: yes, compatible—but the mapping semantics must distinguish which world you’re in.
2) You cannot have a stable “current distribution URL” on GitHub Pages

Given your constraints, the strongest stable guarantee you can make is:

    …/something/ → serves index.html

You cannot guarantee:

    …/something/current.<ext> always exists

    …/something/current serves bytes

    redirects

Therefore, the correct “current” design on GitHub Pages is:

    …/flow/_payload/_current/ (folder)

        contains index.html that links to the current distribution file

        may also include the current distribution file, but deep links are not stable

This is not a compromise; it’s forced by hosting.
3) Your earlier “dist flow with immutable filename” idea is the right way to recover stability

You said: make a different flow like dist that has an immutable filename.

That is actually the key move if you insist on stable deep links under GitHub Pages.

But it only works if you accept one of:
Option 3A — Canonical filename per content kind (per flow)

Pick a fixed filename per flow:

    payload flow always publishes payload.ttl (or payload.jsonld)

    template flow always publishes template.vto

    stylesheet flow always publishes style.css

    image flow publishes image.png (but then jpg→png breaks the rule)

This gives stable deep links but blocks extension changes unless you treat extension changes as a new flow.
Option 3B — Canonical filename per flow, extension fixed at flow creation

Flow has a fixed “artifact basename + extension” recorded at creation (effectively a slug again, just for the file).

    Stable deep links

    Allows rename in _working for author convenience but weave renames output to canonical

    Extension change becomes either:

        disallowed, or

        a migration event that changes canonical name (breaking links), or

        new flow

This is the only way to have stable deep links without redirects.
Option 3C — Drop stable deep links, keep only stable folder pages (recommended)

Don’t promise stable file URLs at all; promise only stable resource pages.

    Stable: …/_current/ and …/slice/7/ as folders

    File inside slice can be any name; link from index.html is authoritative

    Pinning is done by referencing the folder page, not the file

This is the simplest and best fit for GitHub Pages.
4) If you go with folder-only stability, your “rename hazard” becomes acceptable and clean

You were worried:

    renaming in working breaks current URL references

Under folder-only stability:

    …/_payload/_current/ never breaks

    …/_payload/_current/<filename> may break, and that’s explicitly “don’t do that”

So the contract becomes:

    Humans and machines should dereference folder IRIs, not file IRIs.

    File IRIs are convenience, not stable identifiers.

This aligns with your “NOP IRI is identity” claim much better.
5) About “unmanaged by Semantic Flow”

The right phrasing is:

    External URLs and _assets are unmanaged content.

    Slice-backed distributions are managed content.

TemplateMappings may point to either.

But you must define behavior:

    If a mapping points to unmanaged content, weave can optionally:

        snapshot it into a managed slice (recommended as an option), or

        just link to it and accept drift (your current stance)

That’s the compatibility of A and B: same system, different provenance/reproducibility guarantees.
6) Concrete recommendation given GitHub Pages constraints

    Make “folder IRIs are stable, file IRIs are not” the rule.

    _working/ has no page (or a minimal stub if you want), _current/ has an index.html that links to the current file.

    Each immutable slice is a folder with:

        index.html

        exactly one distribution file (any name)

    TemplateMappings store a single IRI:

        either a folder IRI (preferred) like …/template/_payload/slice/7/

        or an external URL (unmanaged)

    Add optional sha256 recording for:

        slices (strong)

        _current (ephemeral; still useful for change detection)

    If you later decide you need stable deep links to files, introduce a canonical output filename per flow (which is, in effect, resurrecting slug).

This gets you out of the redirect trap while keeping your design coherent on GitHub Pages.
You said:
A cananoical output (slug) filename seem problematic.  And do we output both according to the Knop name and the slug? What if the slug changes? Or are you suggesting a single canonical slug across all meshes, like "index.<ext>". What if the extension changes? 

It's nice if file IRIs are stable, and they are in general, just not in _current slices. If we don't provide a distribution in _current, it's not really a slice and we can just embed the pointer in the flow's resource page (and of course in the flow-specific metadata stored in the knop's _meta flow)

My primary use case that I keep coming back to is "world data" for a role playing experience platform. I guess losing _current isn't so bad, we'll just have to expect mesh-consuming applications to parse metadata to get the distribution IRIs. 
ChatGPT said:

A canonical output filename is problematic for exactly the reasons you list. If you try to make the filename itself the stable identifier, you immediately reintroduce “slug governance” (and migration rules) as a core part of the system. On GitHub Pages, that’s a tax you don’t need to pay if you accept that stable identifiers are folders (resource pages), not files.

Your revised direction—“drop _current distribution file; _current is just metadata/pointer in the flow’s resource page / _meta”—is the cleanest way out.
1) Canonical filenames: you’ve correctly identified the failure modes

    If you output both “knop name” and “slug”: you now have two canonical names and must define precedence, deprecation, and redirects (which you don’t have).

    If slug changes: any stable file IRI that depended on it breaks. Without redirects, that’s fatal.

    If you pick a universal name like index.<ext>:

        extension changes break links anyway

        and you risk collisions if you ever have more than one file per slice

So yes: canonical file naming as an identity strategy is a dead end under your constraints.
2) Stable file IRIs can exist (and do) for immutable slices

You’re right to distinguish:

    Immutable slice folders: stable, can contain a file with any name, and that file IRI is stable because the folder is immutable.

    _current: mutable, and therefore file IRIs inside it are not stable.

So the correct invariant is:

    File IRIs are stable in immutable slice folders. File IRIs under _current are unstable and should not be part of the contract.

That’s consistent.
3) If _current has no distribution file, it stops being a slice—and that’s good

Exactly. Don’t pretend _current is “the latest slice.” On GitHub Pages you can’t alias anyway.

Make _current purely a view, not a slice:

    .../flow/_payload/_current/ (optional) → resource page only

    or omit _current entirely and put “current pointer” on the flow resource page

Then:

    the “current version” is an RDF statement (or structured metadata) that points to the latest immutable slice folder

    clients follow the pointer

This is the same pattern as “latest release” pointers in package managers.
4) What applications must do (and why that’s fine for your world-data use case)

You said:

    we’ll just have to expect mesh-consuming applications to parse metadata to get the distribution IRIs.

That’s not a hardship; it’s the correct interface. Your consumers are not web browsers; they’re agents/applications that already have to deal with RDF-ish metadata.

For “world data”:

    determinism and immutability matter more than “guess the URL”

    applications can:

        dereference the flow folder (resource page and/or machine-readable sidecar)

        read currentSlice (or equivalent)

        dereference the slice folder

        pick “the one distribution file” (or follow an explicit distributionIri)

This is a clean contract.
5) The minimal spec you need to write down

To make this workable without canonical filenames, you must define one canonical pointer field.

Either in the flow’s _meta flow or in the flow’s resource-page machine-readable payload:

    sflo:currentPayloadSlice → IRI of immutable slice folder
    (e.g., .../_payload/slice/57/)

Optionally also:

    sflo:currentDistribution → IRI of the distribution file inside that slice
    (so clients don’t have to list files)

Because if you rely on “there is exactly one file besides index.html,” you’ve coupled your API to filesystem listing behavior. It can work, but it’s brittle.
6) Recommendation

    Do not pursue canonical filenames or slugs as stable identifiers.

    Treat immutable slice folders as the stable versioned unit.

    Remove _current as a distribution container; keep “current” as a pointer in metadata.

    Require consumers to follow the metadata pointer to resolve the current slice (and optionally the distribution file).

This aligns with:

    GitHub Pages hosting limitations

    your “identity is the NOP IRI” framing

    your world-data use case (clients are programmatic)

    and your desire to avoid slug governance.

If you want, I can propose exact property names and a tiny JSON(-LD) or Turtle snippet for the flow’s “current pointer” metadata that matches your existing _meta / flow layout.
You said:
I already have "hasCurrentSlice"  and hasDistribution. So no direct "hasCurrentDistribution" but that's probably okay.

I adopted the name "slice" instead of version because I wanted to differentiate VersionSlices from DefaultSlice and WorkingSlice. But if Current/default is no longer a thing, we can just call the Flow's constituents "Versions". 
ChatGPT said:

Your existing hasCurrentSlice + hasDistribution is sufficient if you make one thing explicit:

    hasCurrentSlice must point only to an immutable version slice, not to WorkingSlice or DefaultSlice.

If you allow it to point to mutable slices, you’ve rebuilt _current under a different name and reintroduced all the instability you’re trying to eliminate.

So the contract should be:

    Flow hasCurrentSlice → VersionSlice (immutable only)

    VersionSlice hasDistribution → Distribution (the file IRI)

That gives you an implicit “current distribution” without needing a separate property.
Rename “slice” to “version”?

Be careful: you’re not just renaming; you’re changing the conceptual lattice.

You originally used “slice” to cover multiple kinds:

    VersionSlice (immutable)

    DefaultSlice (non-version, possibly alias-ish)

    WorkingSlice (mutable)

If you delete Default/Current as first-class things, then you effectively have:

    Working (mutable staging)

    Versions (immutable history)

In that world, “slice” becomes needless ambiguity, and “version” becomes accurate and simpler.

The clean reframe is:

    Working is not a “version.” It’s a staging area.

    Everything else is a Version.

So:

    Flow hasWorking → WorkingState (or WorkingSnapshot if you insist)

    Flow hasVersion → Version (monotonic / immutable)

    Flow hasCurrentVersion (or keep your existing hasCurrentSlice but redefine range)

If you keep the property name hasCurrentSlice for backwards compatibility, at least tighten its range to Version and update comments accordingly.
Critical detail you should lock down

If you don’t want hasCurrentDistribution, then for consumers to resolve “current distribution” they must do:

    deref flow

    follow hasCurrentSlice → version

    from that version, follow hasDistribution

This is fine only if exactly one distribution is designated as primary, or you have a deterministic selection rule.

So you need one of:

    Constraint: VersionSlice sh:maxCount 1 on hasDistribution (recommended given your “one file per slice” invariant), or

    A role system: hasDistribution can be many, but one is primaryDistribution (you said you don’t want this), or

    A deterministic rule: pick the only distribution that isn’t index.html (fragile)

Given your stated invariants, the first is best: make it one distribution per version.
Recommendation

    Keep hasCurrentSlice, but redefine/document it as “hasCurrentVersion” semantically (range = immutable version only).

    Make hasDistribution functional on Version (exactly 1) if you want to avoid hasCurrentDistribution.

    Rename the conceptual container from “slices” to “versions” if Default/Current slices are truly gone; keep “Working” as a separate non-version concept.

If you do those three things, the model becomes simpler and your hosting constraints stop forcing awkward naming.
You said:
And we'll get rid of "versioningState" (unversioned, versioned, deversioned) as everything is versioned. If you don't want history, we can set a "retainNumVersions" property per Knop or per-flow.
ChatGPT said:

Dropping versioningState is the right simplification if you accept the consequence: you no longer have a first-class concept of “this flow is meant to be mutable.” Everything becomes “versioned, but possibly garbage-collected.”

That’s coherent, but you must stop calling old versions “immutable” if you enable retention. You can still make each version directory immutable while it exists; you just can’t promise it will exist forever.
1) Replace versioningState with two explicit policies
A. Append policy (always-on)

    New publishes create a new Version/ (monotonic ID).

    hasCurrentVersion points at the newest.

No more unversioned/deversioned.
B. Retention policy (optional)

A property like:

    retainNumVersions (integer, 0/absent = retain all)

Scope: prefer per-flow, with inheritance from knop-level defaults if you want.

Why per-flow?

    Payload vs ref will have different needs (you already cited “update _ref without updating _payload”).

    Templates vs world-data vs binary assets will have very different retention desires.

So:

    Knop retainNumVersionsDefault

    Flow retainNumVersions overrides default

2) Define retention semantics precisely (or it will be a footgun)

You need to decide: when retention prunes, what happens to references?

Recommended semantics:

    Versions are append-only; pruning happens only after a successful weave that has produced the new current version.

    Pruning removes the oldest versions beyond the retention count.

    If a pruned version is referenced by:

        hasCurrentVersion (it won’t be; it’s old)

        or any “pinned” link you expose in a catalog/resource page,

    you accept link rot by design.

To reduce surprises, add one more optional control:

    retainPinnedVersions: true|false (default true)

    A version is “pinned” if it is referenced by some explicit “release tag” or “bookmark” mechanism.

If you don’t want a pin system, then at least document: “pruning may break historical links.”
3) Important consistency constraint

If everything is versioned and you rely on:

    Flow hasCurrentVersion → Version

    Version hasDistribution → DistributionFile

then pruning must also update any indexes/resource pages that list old versions, otherwise your published site will contain dead links.

So pruning can’t just delete directories; it must also regenerate the flow resource page(s) and any catalogs.
4) Naming / ontology updates (minimal)

    Keep Version/ as the class for immutable (while retained) versions.

    Keep Working/ as a staging state (not a version).

    Replace:

        versioningState with nothing (deleted)

        add retainNumVersions on Flow/ (and optionally a default on Knop/)

That’s it.
5) The one thing to be careful about

Retention is logically incompatible with your earlier “slice IRIs are dereferenceable forever” posture. If you adopt retention, you must explicitly downgrade the guarantee to:

    “Dereferenceable while retained.”

That’s acceptable for many use cases (including a lot of world-state workflows), but it has to be stated.

If you want, I can write the exact SHACL constraints you’d use to enforce:

    hasCurrentVersion range Version/

    hasDistribution maxCount 1 on Version/

    retainNumVersions non-negative integer

    and the “Working is not a Version” disjointness.

You said:
we only promise pseudo-immutability anyhow. People can always delete files. We'll just make immutability easy if you want it to be. 

I also like Slice because if Flows have "Versions", that sounds like the entire Flow is included in each version. Maybe we do stick with Slice. 
ChatGPT said:

If you only promise pseudo-immutability, then retention and deletion are just variations of the same reality: the system is append-oriented and tries to preserve history, but operators can remove it. That’s a coherent stance—just make sure your docs don’t imply stronger guarantees than you intend.

On naming: your intuition about “Version” is correct. In common software language, “Flow v7” reads like a version of the whole flow object, not “a single constituent snapshot/edition within the flow.” “Slice” is better precisely because it signals “a piece/cut of the flow,” not “the whole thing.”

So: keep Slice, but tighten the taxonomy now that _current is gone.
Recommended minimal model (with your latest constraints)
Classes

    Flow/

    WorkingSlice/ (mutable staging)

    PublishedSlice/ (append-only intent; pseudo-immutable)

        optionally call it SnapshotSlice/ or ReleaseSlice/ if you want stronger connotation

    Distribution/ (actual file IRI)

Properties

    hasSlice (Flow → PublishedSlice)

    hasCurrentSlice (Flow → PublishedSlice) range must exclude WorkingSlice

    hasWorkingSlice (Flow → WorkingSlice) (or derive by convention)

    hasDistribution (Slice → Distribution) (ideally exactly 1 if you’re keeping “one file per slice”)

    retainNumSlices (Flow or Knop)

This keeps “slice” meaning “constituent” without sounding like the entire flow is duplicated.
Critical wording change you should make in docs

Instead of “immutable slices,” say something like:

    “Published slices are intended to be append-only and stable, but may be deleted by operators.”

That aligns with your “make immutability easy if you want it” philosophy without lying.
One constraint you still want (even with pseudo-immutability)

Even if operators can delete files, you should still enforce at the semantic level:

    hasCurrentSlice must not point to WorkingSlice

Otherwise clients will treat a mutable staging state as stable “current,” and you recreate the _current file-instability problem under a different name.

If you want to preserve the “version” flavor without the “whole flow” connotation, an alternative is Edition (“Flow hasEditions”), but “Slice” is already working for you and fits your weaving metaphor.
You said:
Is working still a Slice? I was thinking to make _working a convention, not even include it in RDF metadata.
ChatGPT said:

If you make _working purely a filesystem convention and omit it from RDF, then no: working is not a Slice (in your ontology). It becomes an implementation detail, not a first-class constituent of the Flow.

That is a good simplification, but you should be explicit about the consequences.
Option 1: _working is not modeled in RDF (recommended if you want simplicity)
Semantics

    A Flow’s RDF describes only published slices.

    _working is just a conventional input location used by the CLI/host during weave.

Implications

    hasSlice ranges over published (pseudo-immutable) slices only.

    hasCurrentSlice points to one of those published slices.

    There is no “current working” in the semantic layer, so clients never confuse mutable staging with published state.

Practical additions (still without modeling working)

You may still want to expose operational status somewhere, but not as core RDF. For example:

    the CLI can show “working dirty / last modified / candidate file” in console output

    optionally a generated local-only status file (not published) if you need it

This keeps your public contract clean.
Option 2: _working is modeled as a Slice (only if you need it)

Only do this if you have a hard requirement for:

    programmatic tooling to reason about working state remotely, or

    workflows where “working” must be dereferenceable and inspectable as data.

But you’ve already argued against working resource pages because they go stale immediately, and you’re targeting GitHub Pages. So the benefits are limited.
Critical constraint if you choose Option 1

You must ensure your code never relies on RDF to locate working inputs. Instead:

    Working location is derived by convention: .../_working/

    Candidate-file selection is purely filesystem-based (with ignore rules)

And the RDF only ever appears after weave, describing published slices and their distribution.

That matches your “resource pages generated on weave” principle cleanly.
Recommendation

Make _working not a Slice, not in RDF. Treat it as a conventional staging folder. Keep the semantic model focused on published slices only.


ChatGPT can make mistakes. Check important info.
