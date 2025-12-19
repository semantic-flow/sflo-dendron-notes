---
id: g0gxw2uiu7f711skmnzxz7k
title: 2025 12 25 Payload Distribution Solution
desc: ''
updated: 1765869194962
created: 1765869190503
---

Skip to content
Chat history
You said:
At least for payload distributions, I feel like we need a way to have a predictable IRI for the latest distribution, to accommodate clients that don't have access to RDF querying. If the payload is an image, you might often want the latest version when you reference it as the img Src in a web page. Maybe payload becomes only for rdf data again, and other resource types just get put in (and updated in-place) in the _assets folder. You lose history that you'd get with keeping the resource as a distribution in slices.

Help me analyze the need and brainstorm possible solutions. Maybe keep the current version in assets and in slices? 
ChatGPT said:

Your stated need is real, but your proposed workaround (“payload only for RDF; everything else goes in _assets and is overwritten”) is a big semantic regression with knock-on problems you probably don’t want.
First, be precise about the actual requirement

You’re mixing two distinct needs:

    Non-RDF clients need a stable URL (predictable IRI) that always resolves to the latest bytes.

    The system wants immutable, versioned artifacts (so history, cacheability, provenance, and reproducibility are preserved).

Those are not in conflict. They only conflict if you insist that the stable URL itself must be immutable.

So the right framing is: provide a stable alias for convenience, while keeping versioned files as the source of truth.
Why “overwrite in _assets” is a trap

If you overwrite binaries in-place you lose:

    Reproducibility: an old HTML page that referenced logo.png now silently changes.

    Provenance: you can’t answer “which exact bytes were used in release v7?”

    Cache safety: aggressive caches/CDNs will serve stale bytes unless you do cache-busting or headers (which GitHub Pages makes awkward).

    Diff / review ergonomics: large-binary churn becomes invisible except via git LFS-ish workflows.

If you also keep versioned distributions, now you’re doing double-publishing and introducing the new failure mode: “current alias doesn’t match catalog current.”
Better: treat “latest” as an alias layer, not a storage strategy

You want two tiers:

    Tier A (immutable): versioned distribution IRIs (slice/versioned path) that never change.

    Tier B (mutable alias): a predictable “current/latest” IRI that always points to Tier A.

Non-RDF clients use Tier B. Everything serious (builds, referencing, provenance) uses Tier A.
Design options for the “latest” alias (pick one primary)
Option 1: “Well-known” mutable file path you overwrite on release (static hosting friendly)

Example patterns:

    .../knop/_payload/_latest/<name>.png

    .../knop/_dist/_latest/<slug>.<ext>

    .../knop/_assets/_latest/<name>.png

On each release, you regenerate _latest/<name>.png by copying bytes from the versioned artifact.

Pros

    Works on dumb clients (no redirects, no JS, no RDF).

    Simple mental model.

    Lets you set short cache TTL for only _latest/* while leaving versioned artifacts cache-forever (if you control headers; if not, you can still do query-string cache busting from HTML builds).

Cons

    You must enforce an invariant: _latest must match catalog “current.”

    Requires build tooling discipline (but you already have “weave/release” semantics).

This is the closest to what you were groping toward, except it keeps history cleanly.
Option 2: HTTP redirect from a stable URL to the versioned URL

.../knop/hero.png → 302/301 → .../knop/_payload/v17/hero.png

Pros

    No duplicated bytes.

    Clean separation: alias is only a pointer.

Cons

    On GitHub Pages / static hosting, real HTTP redirects are nontrivial unless you accept:

        HTML “redirect pages” (breaks <img src>),

        JS redirects (breaks <img src>),

        platform-specific redirect config (Netlify, Cloudflare, etc.).

So for images specifically, redirects are often a dead end.
Option 3: Alias is a tiny JSON manifest next to the bytes

.../knop/_payload/latest.json contains { "href": ".../v17/hero.png", "sha256": "...", "mediaType": "image/png" }

Pros

    Dumb-ish clients can do one extra fetch and then a second fetch.

    Lets you attach integrity metadata cleanly.

Cons

    Still not as dumb as <img src="...">.

This is excellent for programmatic clients, not for HTML img tags.
Option 4: Query-string “current” indirection

hero.png?current or hero.png?version=latest

Pros

    Super easy for authors.

Cons

    Static hosts often treat this as the same file; caches vary; and you still need a server or rules to map it. Not reliable.

What I would not do

    Don’t repurpose _assets as “mutable latest for everything” unless you are willing to formally declare _assets as non-versioned, non-reproducible and accept that old pages are not stable.

    Don’t maintain “current bytes in assets” and “versioned bytes in slices” unless you add automated consistency checks and make one side derived-only (never edited by hand).

A clean synthesis that matches your philosophy

    Keep binaries as first-class distributions (yes, images, CSS, JS, PDFs). Treat them like any other payload artifact.

    Add a deterministic alias path for each binary distribution name:

        .../_dist/_latest/<dist-name>.<ext> (or _payload/_latest if you want it scoped).

    Make _latest/* derived output only (generated during release / weave).

    Adopt explicit caching semantics by convention:

        Versioned: “cache forever” (content-addressable or version-addressed).

        Latest: “may change” (short TTL; if you can’t control headers, your HTML generator can append ?t=<release-id> when it cares).

    Add a cheap invariant check in your build:

        For each _latest/*, validate it equals the bytes (or hash) of whatever _meta says is current.

This gives:

    Predictable stable URLs for dumb clients.

    Full history and provenance.

    No human-edited duplication (duplication exists, but it’s generated and verifiable).

One critical question you should force yourself to answer

Are you trying to make “latest” predictable per knop or globally?

    Per knop predictable is easy: knop/_dist/_latest/*.

    Globally predictable (e.g., “give me the latest logo of the whole mesh”) implies you need a stable registry location, which is a different problem (and likely belongs in platform defaults / site root).

If you conflate those, you’ll end up reintroducing a hidden _default-like mechanism accidentally.

If you want, I’ll propose concrete path conventions (including naming rules for the aliased filename) that avoid collisions and keep the “path = identity” principle intact while still providing stable “latest” affordances.
You said:
Could the stable "alias" (which is more acurately termed a "convenience copy") of the payload file just live directly under the knop? e.g. djradon/bio/bio_current.jsonld ? It's better than putting it in a flow, because then you'd expect it to be a slice. Perhaps reference data would also best be put there, e.g. djradon/bio/bio_ref.jsonld or maybe even djradon/bio/_current/{bio.jsonld|bio_ref.jsonld}

If we continue to allow filename changes of the distribution, there's a risk of breakage if the convience copy retains the filename of the distribution.
ChatGPT said:

Yes, putting a convenience copy directly under the knop can be the cleanest solution, but only if you’re willing to make a few hard, explicit commitments. Otherwise you’ll reintroduce identity drift and “mystery files.”
1) Putting convenience copies under the knop is defensible

A top-level file like:

    djradon/bio/bio_current.jsonld

works because it is clearly not a Flow distribution and therefore is not expected to be slice/versioned. Conceptually it’s an alias materialization (a stable fetch target), not part of the versioned artifact graph.

Where people get burned is when they mix: “top-level file is stable” + “top-level file name follows whatever the current distribution happened to be called.” That’s exactly the filename-change breakage you called out.

So: top-level convenience copies must have their own stable naming scheme independent of distribution filenames.
2) Your proposed naming has a hidden inconsistency

bio_current.jsonld and bio_ref.jsonld encode the knob slug (bio) into the filename even though you’re already inside .../bio/.

That redundancy matters because it creates unnecessary coupling:

    renaming the knob (or deciding slug != folder name) now implies renaming convenience files too

    authors will inevitably produce both bio_current.jsonld and current.jsonld over time

If the location already scopes the subject, the stable name should be minimal.

Better: inside a knop folder, make the convenience filenames constant across all knops.

Examples:

    djradon/bio/_current.jsonld

    djradon/bio/_reference.jsonld

    or djradon/bio/_current/bio.jsonld only if you truly need multiple “current” artifacts

3) Choose one of these two patterns and stick to it
Pattern A: Single-file convention (simplest, least breakable)

    .../bio/_current.jsonld

    .../bio/_reference.jsonld

Rules:

    filenames are invariant

    only the bytes change on release/weave

    versioned distributions live elsewhere (slices), and these are derived copies

This avoids the filename-change problem entirely because distribution filenames are irrelevant.
Pattern B: Folder convention (better if you have multiple current artifacts)

    .../bio/_current/bio.jsonld

    .../bio/_current/reference.jsonld (or _current/_ref.jsonld)

    optionally .../bio/_current/hero.png, etc.

Rules:

    _current/ is the “mutable alias area”

    inside it, filenames are stable logical roles, not “whatever the dist filename was”

    you can add more without cluttering knop root

If you anticipate more than 2–3 convenience targets, this is cleaner.
4) Your “reference data” point: don’t overload meanings

If “reference” means your canonical reference-flow data (formerly touchstone), you want it to be:

    stable fetch target for dumb clients and

    still fully versioned/provenanced in its own series elsewhere

So yes: having _reference.jsonld (or _current/_reference.jsonld) as a convenience copy is coherent.

But be strict: “reference” is a role, not a storage class. It’s still derived from a versioned thing.
5) Filename changes: forbid them at the convenience layer, not globally

You can keep allowing distribution filename changes in slices, but then you must decouple them:

    Versioned file names can change (annoying but survivable)

    Convenience filenames must never change (or you defeat the whole point)

To manage the mapping, you need a stable role name.

Example roles:

    payload-default

    reference-flow

    meta

    hero-image

Then your build system can map:

    role → current versioned distribution → convenience copy path

That also gives you a place to validate invariants (“is the convenience copy exactly the bytes of the role’s current distribution?”).
6) The key risk you still have (even if filenames are stable)

If “path = identity (strict),” introducing mutable files under the knop root muddies that principle unless you explicitly scope it:

    “Knop identity is path”

    “Versioned artifacts are immutable”

    “Convenience copies are mutable and are not identifiers for artifacts, only stable access paths”

If you don’t write that down, you’ll get people citing _current.jsonld as if it were a versioned thing.
Recommended concrete scheme

If you want minimum churn and maximum clarity:

    .../<knop>/_current/<role>.<ext>

        .../bio/_current/payload.jsonld

        .../bio/_current/reference.jsonld

Optionally add:

    .../<knop>/_current/_manifest.json containing hashes and pointers to the versioned sources (nice for verification, not required for dumb clients).

This solves your filename-change concern by making convenience filenames role-based and invariant, while still allowing distribution filenames to change in versioned areas without breaking clients.

If you disagree, the disagreement should be explicit: are you trying to keep knob roots “clean” (few files), or do you care more about absolute predictability even at the cost of “special files” in the root? That choice drives whether you use _current/ folder or direct root files.
You said:
filenames aren't exactly invariant, as the extension could change. :/ We could just (soft-)enforce no distribution file extension changes. 

I think one payload distribution ("artifact") per knop. That comes closest to preserving "single referent". Maybe we don't need the reference data as convenience copy. Presumably any consumer would be RDF-aware and could find the current ref slice by themselves.  

If, after moving a knop folder, we could try to keep the original _current.<ext> updated too. 

There's no elegant magic bullet. 
ChatGPT said:

You’re right: “invariant filename” is only invariant if you treat the extension as part of the invariant. If you allow the extension to change, you do not have a stable fetch target. Full stop. “Soft enforce no extension changes” is a reasonable policy, but be clear what you’re buying: predictability at the cost of format flexibility.
1) Pick what is actually stable: role vs bytes vs media type

You cannot simultaneously have all three:

    stable path

    arbitrarily changing media type/extension

    dumb clients

So choose one hard constraint:

Constraint A (recommended): stable path includes stable extension.
You can still change internal bytes/serialization details, but not the media type class.

That means you do not need “invariant filenames” generally—only a stable role alias with a stable extension.

Example:

    .../bio/_current.jsonld is the stable target (JSON-LD forever)

    the versioned distribution could still be .ttl, .trig, etc., but then the convenience copy is an always-JSON-LD “compiled output”

This is the cleanest way to reconcile “dumb clients” with “extension may change.” The convenience copy is explicitly a canonical convenience format, not “whatever the latest distribution happens to be.”
2) “One artifact per knop” is coherent but has a cost you’re underplaying

“One payload distribution per knop” does preserve a “single referent” story, but only if you also enforce:

    a single role (“payload”) and no other payload-ish files

    a single canonical serialization for the stable alias (or hard-ban extension changes)

    a single update authority (release/weave only)

Otherwise you’ll drift into “one artifact… except when we needed a PNG and a JSON-LD and a PDF,” which is exactly where this becomes messy.

If you’re serious about one artifact per knop, you should make it explicit that:

    other binaries are assets (supporting material) and are not “the payload artifact”

    the payload artifact has a declared media type and intended client contract

3) Dropping convenience copy for reference data is consistent

If your assumption is: “anyone consuming reference-flow is RDF-aware,” then you don’t need a dumb-client stable URL for it. That’s consistent.

But note the fragility: you’re assuming “reference” consumers are sophisticated forever. If you later want to embed reference-derived visuals or excerpts in plain HTML, you’ll regret not having some stable, dumb-friendly endpoint. Not saying you must add it now, but acknowledge you’re making a bet.
4) Moving knops and keeping old _current updated: you’re reintroducing dual-identity

If “path = identity (strict),” then keeping oldpath/_current.jsonld updated after a move is effectively a permanent alias for the knop. That’s not wrong, but it means:

    the old path remains a first-class public address

    you now have to track alias-of / redirect semantics indefinitely

    you can no longer claim “moving the folder changes identity” without caveats

If you do this, you should treat it as an explicit feature: tombstoned knops with maintained convenience aliases (like durable permalinks). Otherwise it will behave like accidental magic.
5) A concrete scheme that resolves the extension issue cleanly

Given your constraints, the least-bad design is:

    Exactly one payload artifact per knop (the “single referent”).

    Exactly one dumb-client stable convenience copy per knop, in a canonical convenience format whose extension never changes.

Example:

    Versioned, canonical artifact distribution(s) (whatever you want internally):

        .../bio/_payload/_series/_main/v17/bio.trig (or .ttl, .jsonld, etc.)

    Dumb-client stable convenience copy (always JSON-LD):

        .../bio/_current.jsonld

Rules:

    _current.jsonld is derived output, regenerated on release.

    The payload artifact may have multiple internal distributions if you want, but the alias always materializes JSON-LD (or always PNG, etc., depending on the knop’s payload media type contract).

    Extension changes are forbidden for _current.* because _current has exactly one allowed extension per knop type (or globally).

This gets you predictable <img src>-style usage when the payload is an image, but then your stable alias would be _current.png (or .svg), and that implies that knop’s payload media type contract is “image/*”. If you want knops to vary, you’ll need either:

    per-knop declared alias extension (still stable once chosen), or

    fixed alias extension per class of knop (data-knops vs image-knops)

6) The missing piece: declare “payload media type contract” somewhere stable

If you don’t want RDF querying, you still need some stable declaration of what _current.* is.

Options:

    convention by file extension (simple)

    a tiny .../bio/_current.meta.json sidecar (still dumb-friendly, one fetch)

    or accept that humans/tools know it from context

If you skip this, you’ll eventually have “_current.jsonld that is actually trig,” or similar footguns.
7) There is no magic bullet, but there is a clean “least magic” boundary

The boundary is:

    Versioned world: everything immutable, referentially stable, RDF-addressable.

    Convenience world: a small, explicitly “mutable alias surface” with stable naming and stable media types.

Your current discomfort is because you’re letting convenience inherit too much from the versioned world (filenames/extensions/moves), which forces you into contradictions.

If you want, I’ll write the exact invariants as a short spec (like 10 bullets) so you can test every future idea against them and avoid accumulating “just this one exception” behavior.
You said:
By "one payload distribution per knop" I meant one coneptual asset, not "only one distribtion". If we keep "arbitrary content" payloads, maybe we don't need _assets. 

So do we abandon old convenience materialized aliases after move? Delete them?
ChatGPT said:

You’re mixing two orthogonal axes again:

    Conceptual asset count (one “payload artifact” per knop)

    Distribution count (many serializations/encodings of that one artifact)

That’s coherent. But it forces a third decision you can’t avoid:

    Alias lifecycle semantics (what happens to stable convenience paths when identity-path changes)

On _assets: if payload can be “arbitrary content,” you can eliminate _assets as a concept only if you’re willing to say “assets are just payload sub-artifacts” (i.e., everything is a distribution of some conceptual artifact). That’s fine, but then you’ll need multiple conceptual artifacts per knop the moment you have “hero image + CSS + downloadable PDF,” unless you push those into other knops or accept “payload is a bundle.” So: dropping _assets is only clean if you either (a) bundle, or (b) allow multiple conceptual artifacts.
After a move: do you delete old convenience aliases?

There are only three defensible policies. Pick one; anything in-between becomes chaos.
Policy A: Identity-strict (delete and tombstone)

    Old path is no longer “the knob.”

    You do not keep serving updated _current.* at the old location.

    You may leave a tombstone:

        oldpath/index.html (human-facing) saying “moved to …”

        optionally oldpath/_moved.txt / oldpath/_redirect.json (machine-friendly)

    Old convenience alias is either deleted or replaced with a static tombstone file.

Pros: consistent with “path = identity.” No dual-addressing.
Cons: dumb clients break unless they follow human redirects; <img src> breaks.

If you choose this, accept that “stable convenience alias” is stable only until the knop moves. That’s an explicit contract.
Policy B: Permalink alias (keep forever, but it becomes a first-class alias)

    Old path remains a valid public access path indefinitely.

    You keep oldpath/_current.<ext> updated to match the new knob’s current.

    You record oldpath isSameKnopAs newpath (or equivalent) in the new knob’s meta and in an opslog/tombstone.

Pros: dumb clients never break; strong backward compatibility.
Cons: violates the naive reading of “path = identity” unless you update the principle to:

    “path is the primary identifier, but knops may have durable aliases.”

This is basically “DOI semantics.” If you do this once, you must support it forever and treat it as a core feature, not a hack.
Policy C: Grace-period alias (keep for N releases or N months)

    Old aliases continue to work for a bounded period.

    Eventually the old path tombstones and stops updating.

Pros: reduces breakage without permanent alias burden.
Cons: “predictable stable URL” becomes time-bounded and therefore less trustworthy; also requires time/version tracking and a deprecation mechanism.

This is the worst for user expectations unless you are very explicit and your audience is disciplined.
Recommendation given your stated values

If you truly mean “path = identity (strict),” then Policy A is the only internally consistent answer.

But you should admit what that means: a “convenience copy” is not a durable identifier. It’s a stable fetch target only while the knop stays put.

If you want dumb-client stability across moves, you don’t want “path = identity (strict)”—you want “path = default address, with supported aliases.” That’s Policy B.
Minimal, non-magical implementation choices

If you pick Policy A (delete/tombstone):

    Leave oldpath/index.html with a clear “Moved to …” and optionally a link rel="canonical" to the new path.

    For machine use, leave oldpath/_moved.json containing { "new": "...", "movedAt": "...", "reason": "..." }.

    Delete oldpath/_current.* (or replace it with a small non-image file that forces breakage loudly rather than silently serving stale content).

If you pick Policy B (keep alias):

    Never delete old _current.* once published; it becomes part of the public contract.

    Maintain a registry of aliases so you can prevent cycles and manage multiple moves.

    Ensure the alias file is generated, never hand-edited.

One critical inconsistency to resolve now

You can’t claim all three at once:

    predictable stable URL for dumb clients

    path = identity (strict)

    moving knops is a normal operation

You must sacrifice one:

    sacrifice (3): forbid moves (or treat move as new knob and old knob remains forever)

    sacrifice (2): allow durable aliases (Policy B)

    sacrifice (1): accept breakage/tombstones (Policy A)

Pick which one is least painful.
You said:
please generate a brief wiki article for "Current Convenience Copy"
ChatGPT said:
Current Convenience Copy

Current Convenience Copy is a non-versioned, materialized file published alongside a knop that provides a predictable, dumb-client-friendly access point to the knop’s current payload artifact.

It exists to support clients that cannot perform RDF queries, follow dataset catalogs, or resolve versioned distributions, such as HTML <img> tags, CSS url() references, or simple file fetchers.
Purpose

A Current Convenience Copy provides:

    A stable, human-guessable path for accessing the latest representation of a knop’s payload

    A non-RDF access mechanism for environments that require a direct URL

    A derived view of the current versioned artifact, not a primary source of truth

It deliberately trades immutability for accessibility.
Characteristics

    Non-versioned
    The file is overwritten when a new version of the payload is released.

    Derived
    It is generated from the knop’s current versioned distribution during a release or weave process.

    Not a Flow or Slice
    Because it is not versioned, a Current Convenience Copy is not treated as a Flow, Slice, or distribution.

    Stable role, not stable bytes
    The path is stable; the contents are expected to change over time.

    Media-type-constrained
    The file extension and media type are fixed by convention once established.

Typical Location

A Current Convenience Copy is published directly under the knop, or in a dedicated _current/ folder.

Examples:

    …/bio/_current.jsonld

    …/bio/_current/payload.png

    …/bio/_current/hero.svg

The filename represents a logical role (e.g., payload, hero), not a versioned distribution name.
Relationship to Versioned Artifacts

The versioned distributions remain authoritative and immutable.
The Current Convenience Copy is a materialized alias of whichever distribution is currently designated as “current” in the knop’s metadata.

Consumers that require provenance, reproducibility, or historical accuracy must resolve the versioned distribution via RDF metadata instead of using the convenience copy.
Movement and Identity

A Current Convenience Copy is scoped to the knop’s path.

If a knop is moved, the behavior of its Current Convenience Copy depends on the project’s alias policy:

    Identity-strict: the old copy is deleted or replaced with a tombstone

    Alias-preserving: the old copy continues to be updated as a durable alias

The copy itself does not define identity and must not be cited as a permanent identifier.
Non-Goals

A Current Convenience Copy is not intended to:

    Provide historical access

    Act as a canonical identifier

    Replace RDF catalogs or dataset versioning

    Support multiple media types at the same path

Summary

The Current Convenience Copy is an explicit, controlled escape hatch from strict immutability.
It enables practical web usage while preserving a clean separation between stable access paths and versioned semantic artifacts.


ChatGPT can make mistakes. Check important info.
