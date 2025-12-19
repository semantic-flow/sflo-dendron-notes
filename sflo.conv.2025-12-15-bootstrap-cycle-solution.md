---
id: zxthmvcc9l4pulcbzsl9t28
title: 2025 12 15 Bootstrap Cycle Solutio
desc: ''
updated: 1765869250874
created: 1765785361799
---
Skip to content
Chat history
You said:
bootstrap problem: I've eliminated the Slice/ subclasses (including _default, aka _current), and eliminated the unversioned/deversioned option. _working is just a convention, not exactly considered a Slice. To figure out the current slice, you need to reference the knop's specific flow's IRI (in the current _meta), but to figure out the current _meta flow, where do you look if there's no default? The only solution I can think of is to make _meta an unversioned  "special file" that sits right under the knop, i.e. /path/to/knop/_knop-meta.jsonld - that actually might be okay. Meta was always a weird flow, containing data for other flows. If we made it a file it would always contain only "current" stuff, its history would be lost, although we could keep some relevant history (optionally) in knop/_knop-opslog.ttl.
ChatGPT said:

You have a real bootstrap cycle, and your proposed escape hatch (a single unversioned “entrypoint” file) is basically the only clean way out given your current constraints (no _default/_current, no “unversioned slice”, _working is not a slice).

That said, your current proposal has a few sharp edges you should confront explicitly.
What’s inconsistent or risky in the current plan
1) You’re implicitly reintroducing a “default,” just not calling it one

/path/to/knop/_knop-meta.jsonld becomes the de facto canonical pointer to “what’s current.” That’s fine, but then you should model it as a bootstrap manifest, not as “meta flow data.” Otherwise you’ll keep tripping over: “is this RDF about the world, or operational state?”
2) “History would be lost” is only true if you require semantic history outside VCS

If the repo is the source of truth, then history exists in Git. The real question is: do you need in-band, machine-readable history (within your published namespace), or is Git history sufficient?

Right now you’re mixing:

    operational state (“what’s current right now”)

    published semantic artifacts (“what clients can dereference”)

If you don’t separate those, you’ll get perpetual ambiguity over what clients should trust.
3) If _knop-meta.jsonld is mutable, every consumer needs to understand “pointer semantics”

A mutable file that changes meaning over time is not a dataset version. That’s okay, but then it’s a pointer, not content. You’ll need explicit semantics like “this document is a moving pointer; do not cite it as stable.”
The decision you need to make (and then be consistent about)

You must pick one of these worldviews:
A) Git is the history; the mesh only needs “current pointers”

Then _knop-meta.jsonld being unversioned is fine, and _knop-opslog.ttl is optional.
B) The mesh itself must carry history in-band

Then you need either:

    an append-only opslog that’s first-class, or

    content-addressed “meta snapshots” referenced from the opslog, or

    a catalog series that records pointer evolution

Your note “optionally keep history in opslog” suggests you’re undecided. If you stay undecided, your system will be incoherent for users and for tooling.
A cleaner framing: introduce a Knop Manifest (not a Flow, not a Slice)

Treat _knop-meta.jsonld as a manifest whose only job is to break the bootstrap cycle.

Properties it should carry (conceptually):

    what flows exist

    for each flow: what is the “current published” version (or distribution)

    where _working lives (convention, not slice)

    optionally: the latest opslog entry id/hash

That avoids pretending “meta is a flow containing data for other flows.” It’s not. It’s a routing table.
Minimal shape (illustrative, not prescriptive)

    knop: the knop IRI (or intramesh id)

    flows: map of flow identifiers → { current, working?, distBase? }

    opslog: optional pointer

    generatedAt, generatedBy (tooling)

If you keep it JSON-LD, fine—just don’t let it sprawl into “general RDF about the knop.” Keep it sharply operational.
How to preserve history without reintroducing _current

If you want in-band history, the best approach is:
1) _knop-opslog.ttl is append-only (events)

Each event records:

    old pointer → new pointer

    timestamp

    reason/actor/tool version

    optionally a hash of the manifest content after the change

2) Optional: store manifest snapshots content-addressed

E.g. knop/_state/manifests/sha256/<hash>.jsonld
Then opslog entries reference the hash. That gives you stable history without making _knop-meta.jsonld itself versioned.

This avoids the “meta history is lost” complaint while keeping the bootstrap file simple.
Alternatives you should reject (given your constraints)
“Infer current meta flow by scanning”

You can always scan the filesystem, but then “current” becomes an emergent property of heuristics. That’s fragile, slow, and breaks determinism.
“Put meta in each flow”

That recreates the cycle (you still need to find the meta that tells you which meta is current).
“Bring back _current but claim it’s not a slice”

That’s just renaming the same concept and will reintroduce your earlier confusion.
Recommendation

Adopt your special file approach, but formalize it as:

    _knop-meta.jsonld = bootstrap manifest (mutable pointer document; operational semantics)

    _knop-opslog.ttl = optional append-only history of pointer changes

    optional content-addressed manifest snapshots if you truly need in-band history

The critical refinement is conceptual: stop treating this as “a weird flow.” It’s not a flow. It’s the thing that lets you find flows.

If you want, I can propose exact naming/IRI conventions for the manifest + opslog that align with your “path = identity (strict)” stance without smuggling _current back in.
You said:
I think _knop-meta.jsonld (or perhaps better, _knop-manifest.jsonld) will have to carry the inventory of slices as well. :/ I don't know where else it would live. 

I'd be interested to see you proposal. 
ChatGPT said:

You are drifting back toward recreating “default catalog state,” just embedded in a manifest. That can be fine, but only if you make the boundary explicit:

    The manifest is authoritative for discovery/routing.

    The actual slice artifacts remain authoritative for content.

    The manifest is allowed to be mutable and “current-biased,” but anything you want stable citation for must be elsewhere (opslog and/or snapshot files).

If you don’t enforce that split, the manifest becomes a second ontology and you will get consistency bugs and trust issues.
Proposed structure
Files at knop root

    /_knop-manifest.jsonld (required, mutable)
    Bootstrap inventory + pointers.

    /_knop-opslog.ttl (optional, append-only)
    History of pointer changes and lifecycle events.

    /_knop-state/ (optional)
    Content-addressed snapshots / derived indices.

Folder layout for flows and slices (example)

    /_<flowSlug>/ (flow folder)

        /_working/ (convention; mutable workspace)

        /slices/<sliceId>/... (immutable published slice material)

        optional: /slices/<sliceId>/_dist/... (if you keep a dist sublayer)

This keeps “what exists” (inventory) in the manifest, and “what it is” in the slice itself.
_knop-manifest.jsonld responsibilities
1) Inventory

    List flows (by intramesh id and/or dereferenceable IRI)

    For each flow, list known slices (ids + minimal metadata), and which is “current”

2) Pointers

    currentSlice per flow

    workingPath per flow (or a conventional derivation, if you want to avoid redundancy)

3) Integrity affordances (strongly recommended)

    sliceDigest (hash) for published slices

    manifestDigest (hash) for self-integrity / opslog linking

    generatedBy, generatedAt, sourceCommit (if this is derived during weave)

4) Explicit semantics

A flag that this is a mutable routing document (so consumers don’t treat it like stable data).
Concrete proposal (JSON-LD-ish, with minimal RDF commitments)

This is intentionally not your full ontology; it’s a bootstrap shape you can map to your vocab later.

{
  "@context": [
    "https://example.org/sflo/context.jsonld",
    { "xsd": "http://www.w3.org/2001/XMLSchema#" }
  ],
  "@id": "./",
  "@type": "sflo:Knop",

  "sflo:hasManifest": {
    "@id": "_knop-manifest.jsonld",
    "@type": "sflo:KnopManifest",
    "sflo:manifestSemantics": "sflo:MutableRoutingDocument",

    "sflo:knopName": "my-knop",
    "sflo:generatedAt": { "@value": "2025-12-14T00:00:00Z", "@type": "xsd:dateTime" },

    "sflo:hasFlowInventory": [
      {
        "@id": "_meta/",
        "@type": "sflo:Flow",
        "sflo:flowSlug": "_meta",

        "sflo:workingPath": "_meta/_working/",
        "sflo:currentSlice": { "@id": "_meta/slices/v7/" },

        "sflo:hasSliceInventory": [
          {
            "@id": "_meta/slices/v7/",
            "@type": "sflo:Slice",
            "sflo:sliceId": "v7",
            "sflo:publishedAt": { "@value": "2025-12-13T21:11:00Z", "@type": "xsd:dateTime" },
            "sflo:sliceDigest": "sha256:…",
            "sflo:hasDistribution": [
              { "@id": "_meta/slices/v7/index.jsonld", "@type": "sflo:Distribution" },
              { "@id": "_meta/slices/v7/index.ttl", "@type": "sflo:Distribution" }
            ]
          }
        ]
      }
    ],

    "sflo:opslog": { "@id": "_knop-opslog.ttl" }
  }
}

Notes (the hyper-critical bits)

    Do not let hasSliceInventory grow into full slice metadata. Keep it “enough to find it, verify it, and choose it.”

    If inventories get huge, you’ll want a split: _knop-manifest.jsonld points to _knop-inventory.jsonld (derived, big), keeping the manifest small and fast.

    If you publish these, you must define caching/consistency expectations: a consumer might fetch manifest and then fetch slice; they could be out of sync mid-deploy. Hashes mitigate this.

How _knop-opslog.ttl fits

If you care about in-band history, opslog entries record transitions:

    flow X: currentSlice moved from v6 → v7

    optionally: includes manifestDigest after the change

    optionally: includes reason, tool version, actor

This lets you keep _knop-manifest.jsonld “current” while still enabling auditability.
The key naming choice

Use _knop-manifest.jsonld, not _knop-meta.jsonld.

Calling it “meta” encourages you to stuff arbitrary RDF into it. “Manifest” makes it clear it’s an index/pointer doc.
What you must decide next (and you should decide now)

    Are slice IDs strictly version numbers (v7), timestamps, hashes, or arbitrary?
    If arbitrary, you need stable ordering rules (what does “current” mean?).

    Is hasSliceInventory required to be complete?
    If yes, it becomes a source of truth and must be validated against the filesystem on weave.

    Is the manifest hand-edited or generated?
    If hand-edited, you’ll get drift. If generated, you need a single input of truth (filesystem scan + conventions) and treat manual edits as unsupported.

If you accept “manifest is generated,” then the inventory problem is straightforward: weave scans flows and published slices and regenerates inventory. If you want hand-editing, you’re building a package manager.

If you want, I can also propose the minimal RDF vocabulary for this (classes/properties) that won’t collide with your existing Flow/Slice modeling, and a validation strategy (what must be checked during weave to prevent identity drift and manifest/slice divergence).
You said:
Maybe two files. _knop-pointers.jsonld is a concise "location of current slices for all flows", and then a _slice-inventory.ttl with the slice inventory. (append-oriented files are .ttl) 

It is tempting, to assure atomicity, that weave creates a separate copy of the targeted knops/submeshes, and moves them (with their updated pointer and inventory data) as the weave completes.
ChatGPT said:

Two-file split is directionally correct, but only if you are strict about which file is authoritative for what.
1) Two files: what each is allowed to mean
/_knop-pointers.jsonld

Purpose: fast bootstrap routing. Must stay small.

Allowed contents:

    knop identifier + maybe a manifest version/digest

    per-flow pointer(s): currentPublishedSlice, optionally workingPath

    optional pointer to the inventory file + its digest

Not allowed:

    slice lists

    per-slice metadata beyond the pointer target and maybe a hash

If you let “just one or two helpful fields” creep in, you will recreate the inventory in two places.
/_slice-inventory.ttl

Purpose: discovery + audit. “Append-oriented” is fine, but only if you define immutability rules.

You need to decide whether inventory is:

    (A) an index of what exists right now (derived, regenerable), or

    (B) a ledger of what has ever been published (append-only, canonical history)

If you try to make it both, you will fail: append-only ledger is not the same as a current index.

Recommendation: make it (B) — a ledger — and derive any “current index” view from it (or keep a separate derived index file if you need speed).
2) Your atomicity idea: good, but the “copy knops/submeshes” framing is too heavy

Copying entire knops/submeshes to stage and moving them at the end is tempting, but it will blow up on size, time, and failure modes (especially with assets). You only need atomicity for the pointer switch + any new slice artifacts + any new inventory lines.
What you actually want is a two-phase publish

Phase 1 (stage):

    write new slice artifacts into a staging area inside the same filesystem/volume

    write updated pointer file and inventory additions into staging copies

    optionally precompute digests

Phase 2 (commit / swap):

    atomically rename/move staged slice folder(s) into their final location

    atomically replace /_knop-pointers.jsonld

    atomically append to or replace /_slice-inventory.ttl depending on your chosen semantics

Atomic rename on the same filesystem is the primitive you can rely on. If you’re not on the same filesystem, you don’t have atomicity anyway and need a different story.
3) Critical constraints you must specify or you’ll get inconsistencies
Constraint: “append-oriented .ttl”

If _slice-inventory.ttl is append-only, you must define:

    What constitutes a valid append (one “publish event” block)

    How to detect partial writes (e.g., trailing marker, hash chaining, or write-then-rename via temp file)

    How consumers find the “latest state” without scanning the entire file every time (you may need a derived index or periodic compaction)

Append-only on its own does not guarantee correctness; it mostly guarantees accumulation.
Constraint: pointers must be verifiable

Your pointer file must include a digest of the inventory and (ideally) of the target slice. Otherwise you can point at nonsense and only discover it later.

At minimum:

    pointers file includes inventoryDigest

    each pointer includes targetSliceDigest (or references an inventory entry that includes it)

Constraint: avoid “double truth”

If a slice’s digest exists in both the slice folder metadata and the inventory, you must define which wins. Prefer: slice is self-describing; inventory references it.
4) Concrete minimal semantics (what I’d implement)
/_knop-pointers.jsonld (small, mutable)

    sflo:knop

    sflo:inventory = /_slice-inventory.ttl

    sflo:inventoryDigest = sha256:…

    sflo:flowPointers list:

        flow (IRI or intramesh id)

        currentPublishedSlice (IRI/path)

        currentPublishedSliceDigest (optional but recommended)

/_slice-inventory.ttl as an append-only ledger

Each publish appends one event describing:

    flow id

    slice id

    slice path/IRI

    slice digest

    published timestamp

    optional: previous current slice (for audit)

    optional: manifest/pointers digest after publish

    optional: actor/tool version

This gives you history and lets you rebuild “what exists” and “what was current when”.

If you need a fast “inventory index,” generate /_slice-index.jsonld (derived, replaceable) and keep the ledger append-only.
5) Staging strategy that doesn’t explode

Instead of copying knops, stage only what changes:

    Stage directory: /.weave-tmp/<txnId>/...

        new slice artifacts (folders/files)

        temp pointers file

        temp inventory file (either full rewrite or “append block” file)

Publish algorithm:

    Write new slice artifacts to staging

    Compute digests

    Prepare inventory append block (or staged rewritten inventory)

    Prepare staged pointers file referencing the new slice + inventory digest

    Swap:

        rename staged slice folder into final location

        replace pointers file via rename

        append inventory block safely (or replace inventory file via rename)

Important: appending to an existing file is not atomic in the same way rename is. If you truly care, you should treat the inventory as replace-by-rename too, or implement an append protocol (write block to separate file + later compaction).

A robust compromise:

    append blocks as separate files: /_slice-inventory/events/<txnId>.ttl (atomic via rename)

    periodically compact into /_slice-inventory.ttl (derived)
    This preserves your “append-oriented .ttl” desire while staying atomic.

6) Naming critique

    _knop-pointers.jsonld is good.

    _slice-inventory.ttl is fine, but if you go with event files, call the folder _slice-inventory/ and keep:

        _slice-inventory/index.ttl (compacted)

        _slice-inventory/events/<id>.ttl (append-only)

This avoids “single giant file forever” as an operational trap.
7) The one question you must answer to avoid architecture drift

Is _slice-inventory a ledger (canonical history) or an index (derived current snapshot)?

Pick one:

    If it’s a ledger: append-only events + optional compaction.

    If it’s an index: rewrite-on-weave, atomic replace.

You can have both, but then they must have distinct names and roles. Otherwise “inventory” will become another bootstrap cycle in a year.

If you choose ledger+events, your atomic publish story becomes clean and you don’t need to “copy knops/submeshes”; you publish a new slice + one new event + one pointer swap.
You said:
no knop identifier, that's inferred from access location. maybe workingPath. Maybe pointer to the inventory file, maybe digest. Let's just call it "_pointers.jsonld" - it's not pointing to knops. Also, importantly, there can be "redirect info"; probably replacing the per-flow pointers with a "newKnopLocation". I like this single, always correct, discoverable thing.

Also, in _knop-inventory, we can have "additionalSlicesLocatedAt" to point back (and forth) the moved knops. 
ChatGPT said:

Renaming to /_pointers.jsonld and dropping an explicit knop identifier is consistent only if you also commit to a hard rule:

    The knop’s identity for the purpose of routing is “whatever path I dereferenced to obtain /_pointers.jsonld.”

That implies /_pointers.jsonld must never contain statements that need to remain true if the knop is relocated. It can only contain relative references (or references to immutable, external identifiers).
1) /_pointers.jsonld as the single bootstrap surface
Minimum viable contents

    workingPath (optional)

    inventory pointer (optional)

    inventoryDigest (recommended if you care about correctness/caching)

    flows → currentSlice pointers (optional if you allow redirect-only mode)

    redirect block (optional; see below)

No knop id; all intra-knop references should be relative to the file location.
Critical constraint

If you allow redirect, you must define precedence:

    If redirect.newKnopLocation exists → ignore all flow pointers (they are stale by definition).

    Otherwise → flow pointers apply.

Do not allow “both are valid.” That creates ambiguous resolution.
2) Redirect semantics: don’t do “per-flow redirect”; do “knop-level redirect”

You said “replacing the per-flow pointers with a newKnopLocation.” That’s the right instinct, because a move is not a flow-level phenomenon; it’s a knop identity change under your “path = identity (strict)” stance.
What newKnopLocation must be

    A location that itself has a /_pointers.jsonld.

    Ideally the canonical new location, not a temporary.

Redirect must also include just enough to prevent loops and ambiguity

You need at least:

    redirectType: MovedPermanently vs Superseded vs Split (if you allow splits later)

    redirectedAt

    redirectReason (optional)

    redirectHopsAllowed or a loop-detection token (see below)

Loop detection (mandatory if you allow chains)

If A redirects to B and B redirects back to A, consumers will spin. Minimal fix:

    include a redirectChainId (uuid) or redirectDigest of the old pointers file

    consumers keep a visited set of locations; if repeated, error out

3) Inventory: your “additionalSlicesLocatedAt” idea is good but dangerous

It’s good because it solves “where did the old slices go?” without copying huge content.

It’s dangerous because it can easily become mutual recursion across moved knops that is expensive and inconsistent.
Make it one-way authoritative

Pick one:

    Option A (preferred): The new location’s inventory is authoritative; it lists priorLocations (for traceability). Old location is only a redirect pointer.

    Option B: The old location keeps a small “tombstone inventory” that says “additionalSlicesLocatedAt = new location.” New location doesn’t need to point back.

Do not require both directions to be maintained. That guarantees drift.
Keep “additionalSlicesLocatedAt” out of /_pointers.jsonld

That belongs in inventory/opslog, not routing. Otherwise you’ll bloat the bootstrap file and recreate your earlier “manifest sprawl” problem.
4) Concrete proposal
/_pointers.jsonld (bootstrap, always correct)

Conceptually:

    workingPath (optional)

    inventory: relative path or absolute URL

    inventoryDigest (optional but recommended)

    redirect: optional object

    flowPointers: optional list/map, omitted if redirect is present

Example shape (illustrative):

{
  "@context": "…",
  "@type": "sflo:Pointers",
  "sflo:workingPath": "_working/",
  "sflo:inventory": "_inventory/",
  "sflo:inventoryDigest": "sha256:…",

  "sflo:redirect": {
    "sflo:newKnopLocation": "https://example.org/new/knop/",
    "sflo:redirectType": "sflo:MovedPermanently",
    "sflo:redirectedAt": "2025-12-14T…Z",
    "sflo:redirectChainId": "uuid:…"
  }

  /* If redirect is absent:
  "sflo:flowPointers": {
     "_meta": { "sflo:currentSlice": "_meta/slices/v7/" },
     "_payload": { "sflo:currentSlice": "_payload/slices/v12/" }
  }
  */
}

/_inventory/ as append-oriented, atomic event files

I would not keep a single append-only _knop-inventory.ttl if you care about atomicity. Use:

    /_inventory/events/<eventId>.ttl (append-only by adding a new file via rename)

    /_inventory/index.ttl (derived/compacted; replaceable)

    /_inventory/indexDigest recorded in /_pointers.jsonld

This gives you append semantics and atomicity without fragile file appends.
Inventory vocabulary elements you mentioned

    sflo:additionalSlicesLocatedAt — if you keep it, define it narrowly:

        It points to a location (a knop root or specific inventory root), not “somewhere out there”

        It is used only when slices are intentionally left behind

But again: make it one-way authoritative.
5) Move procedure that preserves your “single discoverable thing” property

When moving a knop from old → new:

At new location

    publish new /_pointers.jsonld with normal flow pointers

    update inventory to reference any legacy slice locations if you didn’t copy them

At old location

    replace /_pointers.jsonld with redirect-only content (no flow pointers)

    optionally keep a minimal tombstone inventory (but not required if new inventory is authoritative)

This ensures that dereferencing either location yields a single decisive answer:

    old location: “go here”

    new location: “here are current slices”

6) The one unresolved inconsistency

You want “path = identity (strict)” and “old references keep working.” Redirects achieve this operationally, but semantically you must accept:

    The old knop is a distinct identifier that now denotes “the former location which redirects.”

That’s fine, but then you should stop saying “it’s the same knop moved”; it’s “a new knop that supersedes the old, with an explicit redirect relationship.” Otherwise you’ll keep colliding with OWL-ish identity intuitions.

If you want, I can write the exact resolution algorithm (in priority order) that a loader/weave/runtime uses when it encounters a location, including caching rules and failure modes (missing inventory, bad digest, redirect loops).
You said:
I'm lukewarm on redirect prevention. You could almost call it "additionalMeshLocations", and just load them all. Only one _pointers.jsonld should have currentFlows defined. 
ChatGPT said:

“Load them all” can work, but only if you accept two non-negotiable requirements:

    Deterministic selection of the single authoritative currentFlows source.

    Cycle/duplication control (even if you don’t call it “redirect prevention”).

Otherwise your bootstrap process becomes non-terminating or non-deterministic in the presence of a single bad pointer file.
Reframe: additionalMeshLocations is a graph, not a redirect

If /_pointers.jsonld can reference additionalMeshLocations, you’re defining a mesh-location graph. Any loader must traverse it. Traversal implies:

    a visited-set (cycle safety)

    a max-depth / max-nodes cap (DOS safety)

    stable ordering (determinism)

You can call this “loading all locations,” but implementation-wise it is cycle prevention.
Invariant you want: “Exactly one location defines currentFlows”

You said: “Only one /_pointers.jsonld should have currentFlows defined.” Good. Make it a hard invariant and enforce it during weave.
Consequence

Every other location must be in one of these states:

    alias location: has no currentFlows, only additionalMeshLocations and/or workingPath, and optionally inventory pointer/digest

    tombstone (optional): alias location that also carries a reason/date

Do not allow “sometimes it includes currentFlows but it’s stale.” That reintroduces ambiguity.
Loader resolution algorithm (must be specified)

Given an access location L0:

    Fetch L0/_pointers.jsonld.

    Initialize:

        queue = [L0]

        visited = ∅

        authorities = []

    While queue not empty:

        pop L

        if L in visited, continue

        add L to visited

        load L/_pointers.jsonld (or fail this node)

        if it has currentFlows, add L to authorities

        enqueue any additionalMeshLocations from L

    Decide:

        If authorities.length == 1: use it.

        If 0: error “No authoritative location found.”

        If >1: error “Multiple authoritative locations found.”

This makes “load them all” precise.
Minimal caps (not optional in practice)

Even if you dislike “redirect prevention,” you still need:

    maxLocations (e.g., 32)

    maxDepth (e.g., 8)

    per-location fetch timeout/size limit

Without caps, a malformed graph (or malicious remote) can hang tooling.
How to keep it “always correct, discoverable”

Make /_pointers.jsonld do only three things:

    declare whether it is authoritative (currentFlows present or absent)

    point to inventories (optional)

    link to other mesh locations (additionalMeshLocations)

Everything else belongs elsewhere.
Suggested /_pointers.jsonld fields (tight)

    workingPath (optional)

    inventory + inventoryDigest (optional)

    additionalMeshLocations (optional)

    currentFlows (authoritative only)

No per-location “reason” fields unless you’re comfortable with the pointers doc becoming an opslog.
Atomicity implications

Your “only one authority has currentFlows” rule makes atomic publishing simpler:

    When moving authority from A → B, you must ensure you never end up with two authorities or zero authorities (unless you accept a transient invalid state during deploy).

To guarantee that, you need a publish protocol, e.g.:

    Create B as non-authoritative (no currentFlows) and join graph.

    Publish slices/inventory updates to B.

    Flip authority:

        update B to include currentFlows

        update A to remove currentFlows

    Optional: keep A as alias forever.

But step 3 has a race unless you stage and swap carefully. If you publish over static hosting, there can be a window where clients see both or neither. If that matters, you need either:

    single deployment bundle swap (atomic at the site level), or

    an authority election tie-breaker (see below).

If you refuse cycle/duplication handling, you must add a tie-breaker

If you truly don’t want even a visited-set + caps, then you need a deterministic tie-breaker when multiple authorities exist, because it will happen (partial deploys, forks, manual edits).

A workable tie-breaker:

    choose the authority with the highest authorityEpoch (monotonic integer) and, if tied, lexicographically smallest canonical URL.

But this is strictly worse than “error if >1 authority,” because it can silently pick the wrong one and mask deployment problems.
Recommendation

    Use additionalMeshLocations and “load them all,” but formalize traversal with visited + caps. That is not ideological “redirect prevention”; it is basic graph hygiene.

    Enforce the invariant: exactly one authoritative currentFlows.

    Treat any >1 authoritative discovery as a hard error during weave (and ideally during runtime too), unless you adopt an explicit election mechanism.

If you want, I can propose a minimal JSON-LD shape for additionalMeshLocations that supports relative vs absolute references cleanly (and a rule for when relative entries are allowed), because that’s where these systems usually get subtly inconsistent.
You said:
Upon a (system-mediated) knop move, the old _pointers.jsonld(s) get updated to not include "current" flows, and to include additionalKnopLocations . 

So he loader doesn't need to traverse the additionalKnopLocations if they don't need historical data and a loaded one has a _pointers.jsonld that points to current flows. i.e., traversal can stop when current slices are identifier. "Visisted" can be internally represented. No need for authority determination (unless multiple point to current Flows) as all the data should be valid. 
ChatGPT said:

This works, but only if you explicitly accept (and harden against) the two cases you’re currently hand-waving away:

    Transient inconsistency during deploy / partial move (the exact scenario where “all the data should be valid” is false).

    Stale caching (client sees old _pointers.jsonld at one location and new at another).

If you don’t specify behavior for those, your “stop when you find current” rule becomes non-deterministic in practice.
What your rule implies

You’re proposing a graph where:

    exactly one location (the “active” location) has currentFlows

    other locations are aliases/tombstones that have additionalKnopLocations but no currentFlows

And your loader algorithm is:

    Load _pointers.jsonld at the access location.

    If it has currentFlows, stop.

    Else optionally follow additionalKnopLocations until you find one with currentFlows.

That’s fine. But you still need a visited set (internal is fine) and you still need a policy for conflicts and missing data.
Minimal loader spec that matches your intent

Given start location L0:

    visited = {} (internal)

    L = L0

    repeat up to maxHops:

        if L in visited: fail CycleDetected

        add L to visited

        fetch L/_pointers.jsonld

        if currentFlows present: return it (stop)

        else if additionalKnopLocations empty: fail NoCurrentFlowsFound

        else choose next L from additionalKnopLocations using a deterministic rule (below)

This is “stop when current identified” exactly, with internal visited representation.
Deterministic “choose next” rule is required

If you allow multiple additionalKnopLocations, you must specify ordering:

    prefer same-origin relative paths first, then absolute

    then lexical order (or an explicit priority field)
    Otherwise two different clients can take different paths and succeed/fail differently.

You can drop “authority determination,” but you can’t drop “conflict handling”

You said: “No need for authority determination (unless multiple point to currentFlows) as all data should be valid.”

That “unless” is the whole point. Multiple-current can happen even with correct tooling:

    during a move, if the new location is published before the old pointers are updated

    if CDN caches keep the old authoritative pointer around

    if a move is interrupted

So you need one of these:
Option A (strict, my recommendation): treat multiple-current as an error

If traversal encounters a currentFlows and later (or via alternative hop) could find another, you don’t search—so you might never notice. To make “multiple-current” detectable without full traversal, add a cheap invariant:

    Non-authoritative pointers must include forwardToCurrentAt with exactly one target (not a set), not “additionalKnopLocations.”

    Only authoritative file may omit it.

That makes the graph effectively a chain (or a tree with single outgoing “current” edge), and conflicts become detectable by tooling during weave, not at runtime.
Option B (tie-breaker): choose the “best” currentFlows

If you insist on sets and early-stop, you should include a monotonic field in currentFlows, e.g. currentEpoch or publishedAt, and specify:

    prefer highest currentEpoch

    if tie, prefer lexicographically smallest canonical URL

Without this, the loader can stop at the “wrong” current during partial deploy.
Your move procedure must be constrained to preserve the early-stop property

To keep your invariant “only one has currentFlows,” your move operation must be a two-phase commit from the perspective of the published site, otherwise early-stop can observe invalid intermediate states.

If you can do an atomic site-level publish (single bundle swap), great. If you can’t (typical GitHub Pages / static hosting), then partial visibility is real.

At minimum, your tooling should:

    update old location(s) to non-authoritative first, pointing to the new location

    then publish the new location authoritative

That avoids “two-current” but risks “zero-current” if the new publish fails. If “zero-current” is acceptable transiently, state it. If not, you need a tie-breaker/election or atomic deploy.
“Historical data” short-circuit is fine, but then don’t call it additionalKnopLocations

If the loader may ignore traversal once it finds current, those extra locations are not “additional” in a semantic sense; they are a forwarding graph.

Name it something like:

    nextKnopLocations (if multiple)

    forwardToKnopLocations

    or better: forwardToCurrentAt (single) + relatedKnopLocations (optional, for historical/audit)

Right now you’re mixing two different intents:

    forwarding to current

    discovering history/related locations

If you keep them merged, clients will misuse them.
Bottom line

Your approach is coherent if you tighten three specs:

    Traversal spec: internal visited + maxHops + deterministic next-choice.

    Conflict policy: either error on multiple-current via tooling guarantees (single forward edge), or define tie-breaker fields.

    Move publish order: specify whether you prefer avoiding two-current (at risk of zero-current) or vice versa, given non-atomic static deploys.

If you pick the strict forwarding-edge design (forwardToCurrentAt single target), you get:

    early stop

    no authority election

    minimal traversal

    conflicts become build-time errors instead of runtime mysteries

You said:
What do you call the "soft lockfile" that can indicate a weave is in process?
ChatGPT said:

You’re asking for a name, but the real issue is what semantic contract this file implies. I’ll give names, but I’ll also be explicit about which ones lie about their guarantees.

A “soft lockfile” in your system is not a mutex. It is:

    advisory

    human- and tool-visible

    possibly stale

    non-blocking

    informational about process, not state

If you name it like a lock, people (including future-you) will assume safety guarantees it does not provide.
What it actually is

Semantically, this file says:

    “A weave operation has started here, and the published state may be transient or incomplete.”

That is not a lock. It’s a sentinel / marker / intent record.

So names should reflect intent or activity, not exclusivity.
Strong recommendations (fit your model)
_weave-in-progress.jsonld

Best default.

    Honest: says exactly what it means.

    Clear to humans.

    No false locking semantics.

    Works well with your “always correct pointers” philosophy.

Downside: long, but that’s good here.
_weave-status.jsonld

Use if you expect it to evolve beyond “in progress”:

    status: weaving | idle | failed | aborted

    startedAt

    txnId

    expectedAuthorityChange

This is a process state descriptor, not a lock.
_weave-intent.jsonld

Good if you want to emphasize planned outcome:

    “I intend to move authority”

    “I intend to publish slices X, Y”

But weaker as a general signal to loaders.
Acceptable but slightly dangerous
_weave.lock / _weave.lock.jsonld

I do not recommend this unless you put a comment or ontology term explicitly stating:

    This file is advisory and does not guarantee mutual exclusion.

People will assume it’s a real lock. Tools will eventually misuse it.

If you use it, prefer _weave.advisory.lock (ugly but honest).
Names that will cause future confusion (avoid)

    _lock.jsonld

    _mutex

    _busy

    _deploying

    _current-weave

These imply guarantees you cannot actually provide on static hosting.
How loaders should treat it (this affects naming)

Regardless of name, specify:

    Loaders must not block on it.

    Loaders may warn or mark results as “unstable”.

    Weave tooling must remove it on success or failure.

    Optional TTL / heartbeat field to detect stale markers.

Minimal fields if JSON-LD:

    startedAt

    txnId

    actor / toolVersion

    optional expectedCompletion

My concrete recommendation

If you want the cleanest conceptual alignment with everything you’ve said so far:

    _weave-in-progress.jsonld

It:

    does not pretend to lock

    communicates transient semantics

    plays well with early-stop pointer resolution

    survives partial failure without lying

If you later decide you want richer lifecycle tracking, evolve it into _weave-status.jsonld, but start with _weave-in-progress.jsonld.

If you want, I can also propose:

    exact fields

    when loaders should ignore vs warn

    how it interacts with _pointers.jsonld atomic swaps

You said:
what's the difference between an advisory lockfile and a sentinel file? I think _weave-in-progress.jsonld is good. It can still contain "startedAt" timestamp and perhaps a "startedBy" string. I'd imagine it goes into .gitignore. Which, yikes, reminds me.... since knops are composable, if we don't drop gitignore in every one... we'll get _DS_store hell. Stupid macs.
ChatGPT said:
Advisory lockfile vs. sentinel file — the actual distinction

They overlap in practice, but the difference is semantic, not mechanical.
Advisory lockfile

Meaning:

    “A process is using or modifying this resource; others should not act concurrently.”

Key properties

    Implies coordination intent

    Readers are expected to change behavior (delay, avoid, or warn)

    Still non-enforcing (hence advisory)

    Carries a weak exclusivity implication, even if it cannot guarantee it

Mental model

    “Someone’s in the room—knock first.”

Why this is dangerous for you

    Suggests mutual exclusion that static hosting, git, and distributed meshes cannot guarantee

    Encourages future tooling to treat it as safety-critical

    Creates ambiguity when it’s stale: “Is it safe to proceed or not?”

Sentinel file

Meaning:

    “This condition exists; interpret surrounding state accordingly.”

Key properties

    Purely informational

    No implication of exclusivity

    No expectation that readers wait or block

    Absence does not imply safety; presence does not forbid action

Mental model

    “Wet paint sign.” You can touch it, but you’ve been warned.

Why it fits your system

    Your weave model is publish-and-swap, not shared mutation

    Loaders must already tolerate transient inconsistency

    You want to signal state of the world, not coordinate actors

_weave-in-progress.jsonld is semantically a sentinel, not a lock

Even if it has:

    startedAt

    startedBy

    txnId

…it is still a sentinel, because:

    Loaders don’t block

    Writers don’t wait

    There’s no exclusivity contract

    Staleness is survivable

You should explicitly describe it as a sentinel in docs, even if some people colloquially call it a “soft lock.”
Where it should live and how it should behave
Location

    Knop root: /<knop>/_weave-in-progress.jsonld

Lifecycle

    Created at weave start

    Removed at weave completion (success or failure)

    If stale: ignored or warned about

Loader behavior

    MAY warn or mark data as “unstable”

    MUST NOT block

    MUST NOT assume correctness or incorrectness

Git ignore + composability: you’ve hit a real landmine

You’re right to be alarmed. This is not cosmetic; it’s structural.
The problem

    Knops are composable

    You cannot rely on each repo having its own .gitignore

    macOS will happily inject:

        .DS_Store

        ._*

    Editors will inject:

        .vscode/

        .idea/

    Tools will inject:

        temp files

        caches

    And these will propagate into composed meshes unless stopped globally

You need a platform-level ignore contract

You cannot solve this by sprinkling .gitignore everywhere.
The correct solution (and there really is one)
1. Treat ignore rules as platform policy, not repo policy

Define a canonical ignore list that applies to:

    weave input scanning

    publishing

    inventory generation

This is independent of Git.

Examples:

    .DS_Store

    ._*

    .git/

    .gitignore

    .weave-*

    editor dirs

    OS junk

2. Enforce ignores in weave, not git

Your scanner should:

    never ingest ignored paths

    never publish them

    never consider them slices or assets

This guarantees composability even when repos are sloppy.
3. Optionally generate .gitignore, but don’t rely on it

You can:

    auto-generate a .gitignore at repo root as a convenience

    warn if it’s missing

    but never assume it exists

4. Treat _weave-in-progress.jsonld as ignored-by-default

    ignored by scanner

    ignored by inventory

    ignored by publish

    but discoverable by direct access

Minimal policy you should write down now

    Sentinel files

        Indicate transient operational state.

        Are never published as semantic artifacts.

        Are ignored during mesh composition.

        May exist or be stale without affecting correctness.

This single paragraph will save you months of future cleanup.
Bottom line

    Advisory lockfile ≠ sentinel file.

    You want a sentinel, not a lock.

    _weave-in-progress.jsonld is a good name.

    Do not rely on .gitignore for correctness.

    Define ignore behavior as part of Semantic Flow itself, not Git.

If you want, next we can:

    formalize the ignore vocabulary (sflo:IgnoredPath)

    define scanner precedence rules

    or design how weave reports “unstable” reads during in-progress states

You said:
If we copy knops away, do the weave, and then copy back, it should be a lockfile. Maybe copy back fails if modifications detected.
ChatGPT said:

If you “copy knops away, weave, then copy back,” you are no longer just signaling transient state. You’re attempting mutual exclusion (or at least optimistic concurrency with a commit gate). In that world, calling it a lockfile is reasonable — but only if you define what the lock actually guarantees.

Right now your proposal still has a gap:

    A lockfile does not prevent writes by itself.

    Your “copy back fails if modifications detected” is what makes it meaningful: that’s optimistic locking (compare-and-swap).

What you’re actually designing
Model: transactional weave with optimistic commit

    Acquire lock intent (write lockfile)

    Snapshot/copy to staging

    Weave in staging

    Commit back only if the original hasn’t changed since snapshot

That’s a real contract.
Lockfile vs sentinel in this model

    A sentinel says “weave is happening; results may be unstable.”

    A lockfile says “a weave claims exclusive commit rights; other weaves should not proceed, and if they do, they will lose at commit time.”

Even then, it’s still not a hard lock unless you have an enforcing service. But with commit-time verification, it becomes a useful lock.
The critical part: how you detect “modifications”

You need a precise “base state fingerprint” taken at snapshot time, and rechecked at commit time.

Options, from strongest to weakest:
If the knop lives in a Git repo

    Record HEAD commit hash + git status --porcelain must be clean at start

    At commit time: require HEAD unchanged and still clean
    This is the cleanest and fastest.

If not a Git repo (filesystem only)

Record a fingerprint of the tree you intend to overwrite:

    Fast but weaker: (path, size, mtime) for all files

    Strong: content hash (sha256) per file (expensive for big assets)

    Practical compromise: hash for “semantic” files (.ttl, .jsonld, .trig, configs/templates) + metadata-only for large binaries

If you don’t define this, “modifications detected” will be vague and flaky.
Commit rule you probably want

At commit-back time:

    If fingerprint matches: replace atomically (rename/swap)

    If fingerprint differs: fail commit, keep staged output, and surface a conflict report

Do not “partially copy back.” Partial commits create unrecoverable mixed states.
Naming

If you adopt the transactional model, keep _weave-in-progress.jsonld as the human-friendly sentinel or replace it with a lock that encodes the transaction. I’d do one file to avoid drift:

    /_weave.lock.json (or .jsonld if you insist)
    Contains: txnId, startedAt, startedBy, stagingPath, and baseFingerprint.

Example fields (conceptual):

    txnId

    startedAt

    startedBy

    baseFingerprint (git HEAD or tree digest)

    leaseExpiresAt (optional but strongly recommended)

    status: staging | weaving | committing | failed

If you keep it JSON-LD, fine, but don’t pretend it’s semantically meaningful RDF; it’s operational metadata.
You still need stale-lock handling

If a weave crashes, the lock remains. You must define:

    lock is a lease (expires)

    or lock can be broken if no heartbeat update for N minutes

    and what happens to abandoned staging directories

Without this, one crash bricks a knop.
Bottom line

Yes: with “copy away + commit-back only if unchanged,” it’s a lockfile (optimistic lock + transactional commit). But you must specify:

    what fingerprint is recorded

    what “unchanged” means

    commit is atomic swap, not partial overwrite

    stale lock / lease policy

If you tell me whether knops always live in Git working trees (vs arbitrary filesystem roots), I can give you a concrete fingerprint scheme and commit protocol that doesn’t become unbearably slow on large _assets.
You said:
perhaps make "safe copies" of original knops in case of many knops and modifications happen before all folders can be copied.
ChatGPT said:

Making “safe copies” can help, but the naive version (“copy everything first, then weave”) is exactly where you’ll get killed on scale, time, and disk. The right pattern is per-knop transactional staging with cheap snapshots, not a monolithic pre-copy.
The core problem you’re trying to solve

If you have many knops and you start copying them, they can change mid-copy. Then your snapshot is not a snapshot. A “safe copy” only helps if it is consistent.

So you need either:

    a way to create an instantaneous snapshot (filesystem feature), or

    an optimistic snapshot with a verification step that detects mid-copy changes, plus retry.

Recommended approach: per-knop transactions, not whole-mesh
1) For each target knop K, do this as a unit:

    Fingerprint K at time T0 (Git HEAD+clean, or tree digest for selected files).

    Create staged working copy of K cheaply (see below).

    Weave using staged copy only.

    Re-fingerprint K at commit time T1.

    If fingerprints match: atomic swap/replace the minimal set of outputs (often just pointers + new slice folders).

    If mismatch: fail this knop’s commit and report conflict.

This scales because you can process knops independently; you don’t need every knop “safe-copied” before you can start weaving.
“Safe copy” without paying full copy cost
Best (fastest) if available: CoW snapshots

    APFS snapshots, Btrfs/ZFS snapshots, LVM snapshots, etc.
    If you can snapshot the whole workspace, you get a real consistent baseline. But don’t assume this exists everywhere.

Practical cross-platform: hardlink-based copy

    Copy tree using hardlinks so it’s near-instant and minimal disk, then modify staged outputs normally.

    Semantics: you must ensure you never mutate shared files in-place (use copy-on-write at the file level).

Practical cross-platform: rsync with --link-dest

    Creates a snapshot directory where unchanged files are hardlinked from a baseline.

    When files change, rsync writes new copies.

Low-tech: selective copying

If your weave only needs:

    semantic sources (.ttl, .trig, .jsonld)

    configs/templates

    not giant assets
    then copy only those into staging and reference assets read-only from the original. This reduces both race surface and disk.

Handling “modifications happen before all folders can be copied”

You don’t fix that by copying more. You fix it by changing the protocol:

    Don’t try to snapshot “many knops” in one continuous operation.

    Snapshot and weave one knop at a time, or small batches, and verify at commit.

If you really want a mesh-level “consistent run,” then you need a mesh-level fingerprint and you accept that any conflict causes the entire run to abort (expensive and fragile). Per-knop commit is typically better.
What “safe copies” should be used for

Use safe copies as:

    a rollback point if commit partially succeeds (shouldn’t happen if you only do atomic swaps)

    a debugging artifact (“here’s the exact input I wove”)

Do not use them as your only concurrency control. The real control is “verify unchanged at commit.”
One more critical constraint

If you use hardlinks / link-dest snapshots, you must enforce:

    staged directory is write-isolated (no in-place edits to linked files)

    any write triggers copy-on-write behavior (tools often do this naturally by rewriting files, but not always)

Minimal spec decision you need

Are knops always inside Git working trees?

    If yes: fingerprint = {repoRoot, HEAD, clean-status, plus maybe submodule SHAs} (fast and reliable).

    If no: you need a defined tree digest policy (which files are hashed vs metadata-only), otherwise conflicts will be either missed or too frequent.

If you answer that one, I can give a concrete per-knop transaction format (txnId, baseline fingerprint, staged path layout, commit steps) that matches your _pointers.jsonld + _weave-in-progress story without turning disk usage into a disaster.
You said:
I agree, one at a time. Ideally writing applications will honor the advisory lock, but we need to account for humans using an editor. 

Knops are NOT always in Git working trees. Remember, only _working SHOULD be mutable, so we do only need to copy those new slice folders and update the inventory and pointers files. 

Let's keep it as simple as possible.
ChatGPT said:

Good—this simplifies things a lot. Given your constraints, you can get correctness without heavy machinery if you are ruthless about scope and invariants.

Below is the simplest protocol that actually holds up when humans edit files mid-weave.
Core invariant (this is doing most of the work)

    Only _working/ is mutable.
    Published slices, inventory, and _pointers.jsonld are immutable except via weave.

Once you truly enforce that, you do not need to snapshot entire knops, trees, or assets.

Everything reduces to: did _working/ change while I was weaving?
What you must protect against

    A human edits _working/ during weave.

    A human edits _pointers.jsonld or inventory by hand (should be rare but must be detected).

    Two weaves overlap on the same knop.

You do not need to protect against:

    edits to published slices (they are immutable by policy)

    edits to other knops (you’re doing one at a time)

Minimal locking model (keep it honest)
File: _weave-in-progress.jsonld

This is still best thought of as a sentinel with teeth.

It does not block edits.
It does make concurrent weave commits fail safely.

Contents (minimal, no ontology ambition):

    startedAt

    startedBy

    txnId

    workingFingerprint (see below)

Put it in .gitignore, and also treat it as ignored-by-weave-input.
Fingerprinting: extremely simple and sufficient

You only fingerprint what matters:
At weave start (T₀)

Compute a fingerprint of:

    _working/ contents

    optionally _pointers.jsonld and inventory timestamps (not content hashes)

Working fingerprint options (pick one):

    Fast: recursive (path, size, mtime) tuple hash

    Safer: content hash of semantic files only (.ttl, .jsonld, .trig)

This is cheap because _working/ should be small.

Store this fingerprint in _weave-in-progress.jsonld.
Weave workflow (one knop)
1. Start

    Write _weave-in-progress.jsonld

    Record workingFingerprint₀

2. Stage outputs

    Generate new slice folder(s) in a staging area

    Generate updated inventory and _pointers.jsonld in staging

    Do not touch the live knop yet

3. Commit check (T₁)

Recompute:

    workingFingerprint₁

If workingFingerprint₁ ≠ workingFingerprint₀:

    Abort

    Leave staged results intact

    Report: “_working changed during weave; rerun required”

This handles human edits cleanly.
4. Commit (atomic where it matters)

If fingerprints match:

    Move new slice folders into place (rename)

    Replace inventory file(s) (rename)

    Replace _pointers.jsonld (rename)

    Remove _weave-in-progress.jsonld

No partial commits. No merge logic.
Why this works (and where it breaks)
Works because:

    Humans editing _working/ invalidate the commit → safe abort

    Editors touching published slices are violating policy; tooling doesn’t need to support that

    No reliance on Git, filesystem snapshots, or global locks

    No mesh-level coordination

Breaks if:

    You allow _working/ to be huge and frequently edited

    You allow “minor edits” to inventory/pointers outside weave

    You try to parallelize per-knop weaves without a scheduler

All acceptable tradeoffs for “keep it simple.”
Advisory lock semantics (clear and honest)

    Other weaves should refuse to start if _weave-in-progress.jsonld exists.

    Humans can still edit files.

    Tooling must not auto-wait; it should fail fast with a clear message.

    Stale sentinel is harmless; next weave can delete it after verifying no commit happened.

This is optimistic locking, not exclusion.
One hard rule you must document

    Anything under _working/ is provisional and may be discarded.
    Anything outside _working/ is immutable except via weave.

If you do not write this down and enforce it socially and in tooling, no lockfile or protocol will save you.
Bottom line

You do not need:

    full knob copies

    filesystem snapshots

    mesh-wide locks

    Git assumptions

You do need:

    _weave-in-progress.jsonld

    a cheap _working/ fingerprint

    atomic rename on commit

    a willingness to abort and retry

This is the smallest protocol that is actually correct in the presence of humans with editors.
You said:
Why not just use timestamps for the distributions in _working?

I really would like to be able to capture multiple knops at the same time and roll back if any file. If the application doesn't need that, they can do per-knop transactions. But for a role-playing game, you might need consistency across knops.

Also, don't know if it helps, but maybe it doesn't matter if working changes again!!! I don't think it does. Weave, move back the slices, and if working has changed, that's just future weave fodder.
ChatGPT said:
Why timestamps alone are a bad change detector (even in _working/)

They’re attractive because they’re “free,” but they’re not reliable enough for your correctness boundary.

Common failure modes:

    Coarse resolution: some filesystems/tools effectively give you 1s resolution; multiple edits within a second collapse into one timestamp.

    Timestamp preservation: copy tools, git checkouts, unzip, rsync, editors, and build steps can preserve or normalize mtimes.

    Clock skew: if any part of the pipeline runs in a different environment (container, network share, VM), mtimes become meaningless.

    Non-content changes: a file can be touched without content change, or content can change while mtime remains unchanged (it happens with some editor/write patterns).

    Directory mtimes are not enough: you’d still have to walk the tree to read file mtimes; at that point a cheap hash isn’t much harder.

If you want “as simple as possible” but still correct, the usual sweet spot is:

    fingerprint = hash of (relativePath, size, mtime) for all files under _working/
    (still uses timestamps, but not only timestamps, and it’s deterministic)

If you need higher confidence for semantic sources only:

    fingerprint = hash of file contents for *.ttl|*.trig|*.jsonld in _working/, and (size, mtime) for everything else.

That stays cheap because _working/ should be small.
Cross-knop consistency: your “roll back if any file changes” requirement conflicts with “keep it simple”

If you truly need consistency across knops (RPG session state is a good example), you need an atomic release boundary that is not per-knop.

Trying to “lock many knops and rollback if any changed” on a plain filesystem with humans editing is inherently brittle unless you introduce a single commit switch.

The simplest robust design is:
A single atomic switch: release bundles

Instead of committing slice/pointer changes directly into each knop as you go, you:

    Weave everything into a staging release directory (new slice folders, inventories, pointers).

    When done, flip one pointer at the mesh/site root to make the new release visible.

Two ways to implement the flip:
Option A: top-level _release.jsonld (or root _pointers.jsonld) selects a release

    /_release.jsonld contains currentRelease: releases/r123/

    Everything dereferences through the current release base.

Atomicity: replace /_release.jsonld via rename.
Option B: build into docs/releases/r123/, then replace docs/current/ (or equivalent)

Atomicity: rename current -> old, r123 -> current if same filesystem.

This gives you cross-knop consistency without multi-knop locking.

Per-knop transactions still exist internally during weave, but externally the world only sees “release N” or “release N+1”.
“Maybe it doesn’t matter if working changes again”

This is only safe if you explicitly define what a weave means.

There are two different semantics:
Semantics 1: “Weave publishes a snapshot”

Then you must ensure you’re weaving from a coherent snapshot of _working/ for each knop (or for the whole release). If _working/ changes mid-weave, you risk publishing a slice that never existed as a single coherent state.

This matters for RPG consistency.
Semantics 2: “Weave publishes whatever it managed to read”

Then yes—working changes “don’t matter,” but you have accepted that a published release may reflect a mixture of edits from different moments. That undermines your stated need for cross-knop consistency.

So you need to pick. Given your RPG comment, you want Semantics 1 at least at the release level.
Minimal multi-knop capture that isn’t a full lock system

If you want “rollback if any file changes” during a multi-knop weave, the simplest honest version is:

    At start, record a baseline fingerprint per knop (only _working/).

    Weave into a staging release directory.

    Before flipping the release pointer, re-fingerprint all targeted knops.

    If any differ: abort the release flip (no partial publish). Keep the staged release for debugging.

    Otherwise: flip the single release pointer.

No need to prevent humans from editing; you just refuse to publish an inconsistent release.

This gives you:

    per-knop simplicity

    cross-knop consistency when you need it

    a single atomic publish step

Recommendation given your goals

    Don’t use raw timestamps as the only signal.

    Implement release bundles with a single commit switch for the “RPG consistency” mode.

    Keep per-knop weaving as the internal unit of work, but make publication a single flip.

If you want, I can propose the exact minimal files involved (names + fields) that fit your existing direction:

    /_weave-in-progress.jsonld per knop (optional in release mode)

    /_release.jsonld at mesh root

    release directory layout for knops that preserves your _pointers.jsonld approach without duplicating immutable slice data unnecessarily.

You said:
Weave publishes a snapshot, so we only need to detect if _working (usually only one file) have been modified by the time the "copy" is made. It could be made into memory, we don't need a disk copy. 
ChatGPT said:

If “weave publishes a snapshot,” then the only correctness point is: did you capture a coherent snapshot of _working at a single moment per knop (or per release)? You do not need to care what happens after the snapshot is captured, except for what you choose to publish.

Given your clarification, you can make this very simple and avoid disk copies entirely.
The minimal correct protocol (per knop)
Goal

Capture _working snapshot bytes for the semantic sources, then weave from those bytes only.
Step 1: Capture snapshot

For each file you consider part of _working (often one file):

    stat1 = stat(path) → (size, mtimeNs) (or best available resolution)

    bytes = readFile(path)

    stat2 = stat(path)

    Accept snapshot iff stat1.size == stat2.size AND stat1.mtime == stat2.mtime

        Otherwise retry (bounded retries) or fail with “file changed during snapshot”

This is the standard “read-verify” pattern. It does not require hashing and it defeats the mid-read edit case.

Why it’s better than “just timestamps”: you’re using timestamps only as a stability check around an actual read, not as a proxy for content.
Step 2: Weave from memory

    Parse from bytes using an explicit base IRI derived from the distribution location (your earlier decision).

    Never touch _working again during the weave.

Step 3: Publish results

    Write new slice outputs and updated _pointers.jsonld / inventory as usual.

    You do not need to fail if _working later changes, because your snapshot is already taken. That change is “future weave fodder,” as you said.

This aligns perfectly with “snapshot semantics.”
Where the advisory sentinel fits

_weave-in-progress.jsonld becomes optional in this model.

You only need it if you want:

    to discourage simultaneous weaves (to avoid waste / races on publishing)

    to signal “published state may change soon”

But it is not required for correctness of the snapshot itself.

If you keep it:

    create it just before snapshot

    delete it after publish

    do not treat it as a gate for human edits

Multi-knop consistency (release snapshot)

If you need cross-knop consistency, extend the same idea:

    For each targeted knop, perform the same “read-verify” capture for its _working files into memory.

    Only after all snapshots are captured, weave and publish as a single release flip (if you adopt release bundles).

    If any file is “unstable during read,” abort before weaving (or retry that knob’s capture).

This gives you a coherent release snapshot without locking.
One critical caveat you must accept/document

This model assumes _working is captured as a set of files whose contents are self-consistent. If you ever allow _working to be a directory with many interdependent files, you can still do it, but you need to define what “coherent” means across multiple files (because you can’t atomically snapshot a directory on a generic filesystem). The simplest rule is:

    _working should be one file for semantic inputs (strongly preferred).

    If multiple files: capture each with read-verify; if any fail, restart the entire knop snapshot.

Practical naming for the snapshot boundary

If you record anything in _weave-in-progress.jsonld, the only field that matters for audit/debug is:

    snapshotTakenAt

    optionally snapshotDigest (hash of the bytes you read)

That lets you prove what you wove, even if _working changes 10 seconds later.

Bottom line: yes—memory snapshot is enough. The key is the read-verify capture step; after that, changes to _working are irrelevant to the correctness of the published snapshot.
You said:
YES!!! Yeehaw! We'll keep _weave-in-progress to discourage simultaneous weaves. I love it. I'd been so worried that applications writing without sentinel awareness would ruin everything. I am so happy. You be happy too!
ChatGPT said:

Agreed. This resolves the bootstrap, concurrency, and human-editing concerns cleanly.

Final shape (concise):

    Semantics: Weave publishes a snapshot.

    Snapshot capture: Read-verify _working into memory (stat → read → stat). Retry or fail if unstable.

    Post-snapshot edits: Irrelevant to the published snapshot; they are future weave fodder.

    Publishing: Generate new slice(s) and update inventory + _pointers.jsonld from the in-memory snapshot.

    Concurrency signal: Keep /_weave-in-progress.jsonld as a sentinel to discourage simultaneous weaves (non-blocking, advisory).

    Multi-knop consistency: Capture snapshots for all targeted knops first; only then publish (optionally via a single release flip).

    Immutability rule: Only _working is mutable; everything else changes only via weave.

    No Git dependency. No tree copies. No hard locks.

This is minimal, correct, and scalable.
You said:
I think a single file per _working (plus the stub index.html that shouldn't really change) is fine. Can you write a concise wiki article titled "consistency"
ChatGPT said:
Consistency

This document defines the consistency model used by Semantic Flow during weave operations.
Summary

Weave publishes a snapshot.
Consistency is achieved by capturing a stable, in-memory snapshot of _working inputs at the start of a weave. Subsequent edits do not affect the published result and are treated as input for future weaves.

No filesystem locks, Git assumptions, or directory-wide copies are required.
Scope of Mutability

    Mutable: _working/
    Typically a single file (plus a stub index.html, which is expected not to change).

    Immutable (by policy):

        Published slices

        Inventories

        _pointers.jsonld

        All non-_working content

Only _working participates in snapshot consistency.
Snapshot Semantics

A weave captures a coherent snapshot of _working by reading files into memory and verifying stability during the read.
Read–Verify Procedure (per file)

    stat₁ → record (size, mtime)

    read → read file bytes into memory

    stat₂

    Accept snapshot iff stat₁ == stat₂
    Otherwise retry or abort.

This prevents mid-read edits from corrupting the snapshot.

After a snapshot is accepted:

    All weaving is performed from the in-memory bytes.

    Any later edits to _working are ignored for this weave.

Multi-Knop Consistency

When consistency is required across multiple knops (e.g., a role-playing game state):

    Capture snapshots for all targeted knops first.

    If any snapshot is unstable, abort before weaving.

    Weave from the captured snapshots.

    Publish results together (optionally via a single release switch).

This ensures the published state corresponds to a single logical moment.
Concurrency Signaling

/_weave-in-progress.jsonld MAY be present to discourage simultaneous weaves.

    It is advisory, not a lock.

    Tools SHOULD refuse to start another weave if present.

    Humans may still edit _working.

    Presence or absence does not affect snapshot correctness.

The file may include:

    startedAt

    startedBy

    txnId (optional)

What Consistency Does Not Require

    No Git working tree

    No filesystem snapshots

    No directory-wide copying

    No hard locks

    No prevention of human edits

Correctness depends solely on snapshot capture, not on controlling writers.
Design Rationale

    Simplicity: One file per _working keeps snapshot logic trivial.

    Robustness: Read–verify is sufficient to detect instability.

    Composability: Works for local filesystems, static hosting, and composed meshes.

    Predictability: Published slices always reflect a real snapshot that existed.

Invariant (Must Hold)

    Anything under _working/ is provisional and may be discarded.
    Anything outside _working/ is immutable except via weave.

If this invariant is upheld, the consistency model is sound.
You said:
_pointers.jsonld and _slice-inventory.ttl are not really immutable, just not supposed to be touched by users (and their non-sflo processes). 

"No prevention of human edits" should be "No prevention of edits, although edits made during the brief "read all targeted working distributions" process will cause a failure." Or something like that. Basically the advisory sentinel can be removed after the targeted working distributions are all read.

txnId should be "weaveLabel" and yes remain optional incase useTimestamps: false

Remove this less-relevant or incorrect stuff stuff:
- "Composability: Works for local filesystems, static hosting, and composed meshes."
- "Anything under _working/ is provisional and may be discarded." nothing gets discarded here. 
ChatGPT said:
Consistency

This document defines the consistency model used by Semantic Flow during weave operations.
Summary

Weave publishes a snapshot.
Consistency is achieved by capturing a stable, in-memory snapshot of the targeted _working inputs at the start of a weave. Subsequent edits do not affect the published result and are treated as input for future weaves.

No filesystem locks, Git assumptions, or directory-wide copies are required.
Scope of Mutability

    Mutable (user-authored): _working/
    Typically a single file (plus a stub index.html, which is expected not to change).

    Managed by weave (not user-edited):

        Published slices

        _slice-inventory.ttl

        _pointers.jsonld

The latter files are not immutable, but must only be modified by sflo tooling.
Snapshot Semantics

A weave captures a coherent snapshot of _working by reading files into memory and verifying stability during the read.
Read–Verify Procedure (per file)

    stat₁ → record (size, mtime)

    read → read file bytes into memory

    stat₂

    Accept snapshot iff stat₁ == stat₂
    Otherwise retry or abort.

This prevents mid-read edits from corrupting the snapshot.

After all targeted _working distributions are successfully read:

    The snapshot is fixed for this weave.

    The advisory sentinel may be removed.

    Any later edits to _working are irrelevant to this weave and apply to future weaves.

Multi-Knop Consistency

When consistency is required across multiple knops (e.g., a role-playing game state):

    Capture snapshots for all targeted knops first.

    If any snapshot is unstable, abort before weaving.

    Weave from the captured snapshots.

    Publish results together (optionally via a single release switch).

This ensures the published state corresponds to a single logical snapshot across knops.
Concurrency Signaling

/_weave-in-progress.jsonld MAY be present to discourage simultaneous weaves.

    It is advisory, not a lock.

    Tools SHOULD avoid starting a new weave while it exists.

    Edits are not prevented; edits made during the brief snapshot read phase will cause the weave to fail.

    The sentinel MAY be removed immediately after all targeted _working distributions are read.

Optional fields:

    startedAt

    startedBy

    weaveLabel (optional; replaces txnId, especially when useTimestamps: false)

What Consistency Does Not Require

    No Git working tree

    No filesystem snapshots

    No directory-wide copying

    No hard locks

    No blocking or prevention of edits outside the snapshot read window

Design Rationale

    Minimal surface: One file per _working keeps snapshot logic simple.

    Correctness: Read–verify ensures snapshots reflect a real, coherent state.

    Predictability: Published slices always correspond to an actual snapshot that existed.

Required Invariant

    _working is the only user-authored mutable input to a weave.
    _pointers.jsonld and _slice-inventory.ttl are managed exclusively by sflo tooling.

If this invariant holds, the consistency model is sound.
You said:
I added this below snapshot semantics:

### Pruning Procedure

- targeted Knops (and their flows) may not have been modified since the previous weave.
- these can be discarded from the "read inventory" before the read-verify procedure starts
ChatGPT said:

Here’s a tightened, wiki-ready version of that section, aligned with the rest of the article and avoiding new ambiguities.

I’m being precise about what is pruned and when, because otherwise this can be misread as weakening snapshot guarantees.
Pruning Procedure

Before snapshot capture begins, the weave MAY prune targeted Knops (and their flows) that are known to be unchanged since the previous weave.

    A Knop (or specific flow within a Knop) is eligible for pruning if its _working distribution has not been modified since the last successful weave that produced the current published slice.

    Pruned Knops/flows are excluded from the snapshot read inventory and are not subjected to the read–verify procedure.

    Pruning occurs before any snapshot reads begin and does not affect snapshot correctness for included Knops.

Pruning is an optimization only. It does not change snapshot semantics:

    All non-pruned _working distributions are still captured via read–verify.

    The published result remains a coherent snapshot of the targeted inputs.

    If pruning information is incorrect or unavailable, the weave MUST fall back to reading all targeted _working distributions.

One subtle but important clarification (worth keeping implicit or explicit)

This pruning rule relies on a stable definition of “modified since previous weave,” typically based on:

    filesystem metadata (mtime/size), or

    sflo-managed metadata from the previous weave.

It does not rely on users refraining from edits; incorrect pruning merely causes stale inputs to be reused, not corruption. A subsequent weave will pick up changes normally.

If you want, the next natural follow-up sections would be:

    “Change Detection” (how pruning eligibility is determined)

    “Failure Modes” (what happens if pruning was wrong)

    “Release vs Per-Knop Weaves” (when pruning is safe across knops)

But the text above is already internally consistent with the rest of the article as written.
You said:
Each flow has a "_working", they don't exist for knops directly.
ChatGPT said:

Good catch — the pruning text was subtly wrong. Here is the corrected version, precise and consistent with your model.

You’re right: _working exists per flow, not per Knop. Knops are only containers.
Pruning Procedure

Before snapshot capture begins, the weave MAY prune targeted flows whose _working distributions are known to be unchanged since the previous weave.

    Each flow has its own _working distribution; Knops do not have _working directly.

    A flow is eligible for pruning if its _working distribution has not been modified since the last successful weave that produced the currently published slice for that flow.

    Pruned flows are excluded from the snapshot read inventory and are not subjected to the read–verify procedure.

    Pruning occurs before any snapshot reads begin and does not affect snapshot correctness for included flows.

Pruning is an optimization only. It does not change snapshot semantics:

    All non-pruned _working distributions are still captured via read–verify.

    The published result remains a coherent snapshot of the targeted flows.

    If pruning information is incorrect, unavailable, or ambiguous, the weave MUST fall back to reading the flow’s _working distribution.

Important Clarification

Pruning relies on sflo’s ability to determine whether a flow’s _working distribution has changed since the previous weave (e.g., via filesystem metadata or sflo-managed state).

Incorrect pruning may cause a flow’s previous snapshot to be reused, but it cannot corrupt a weave. Any missed changes will be incorporated on a subsequent weave.

This keeps the optimization strictly flow-scoped, avoids accidental Knop-level semantics, and preserves your core invariant:

    Snapshot correctness is defined at the flow _working distribution boundary.

If you want, the next refinement would be to explicitly define what “previous weave” means in multi-release vs per-flow contexts, but the text above is already safe and internally consistent.
You said:
Unfortunately, I think pruning detection has to happen after the stat1 is recorded. 
ChatGPT said:

Yes — and that’s an important correction.
If pruning happens before stat₁, it can race with edits and silently reuse stale data. The pruning check must be integrated into snapshot capture, not a pre-filter.

Here is the corrected and safe formulation, still minimal.
Pruning Procedure

Pruning is an optimization applied during snapshot capture, not before it.

For each targeted flow:

    Perform stat₁ on the flow’s _working distribution.

    Compare stat₁ with the metadata recorded from the previous successful weave for that flow.

    If the distribution is unchanged, the flow MAY be pruned:

        The previous snapshot for that flow is reused.

        The read–verify procedure is skipped for that flow.

    Otherwise, proceed with the normal read–verify procedure.

Pruning decisions are made after stat₁ is recorded and before any reads occur.
Correctness Guarantees

    Pruning never weakens snapshot correctness.

    A flow is pruned only if its _working distribution is demonstrably unchanged at the time snapshot capture begins.

    If pruning metadata is unavailable, ambiguous, or inconsistent, the weave MUST fall back to read–verify for that flow.

    If a flow changes after pruning (after stat₁), the change is outside the snapshot boundary and will be captured by a subsequent weave.

Rationale

Recording stat₁ before pruning ensures:

    Edits concurrent with weave startup are detected.

    Pruning cannot mask mid-edit changes.

    Snapshot semantics remain consistent whether or not pruning is applied.

Pruning remains an optimization only; correctness depends solely on the snapshot boundary established by stat₁.

This preserves your model cleanly:

    Snapshot boundary: stat₁

    Snapshot contents: either reused prior snapshot or newly read bytes

    Post-boundary edits: irrelevant to this weave

This version is tight, race-safe, and still simple.
You said:
I'm actually saying if the distribution is unchanged, don't include it in the weave.
ChatGPT said:

Got it. That’s a stronger and cleaner pruning rule, and it’s consistent with “weave publishes a snapshot” of only the changed inputs.

Here is the corrected wording, with the logic tightened and no accidental reuse semantics implied.
Pruning Procedure

Pruning is an optimization applied during snapshot capture, after stat₁ is recorded.

For each targeted flow:

    Perform stat₁ on the flow’s _working distribution.

    Compare stat₁ with the metadata recorded from the previous successful weave for that flow.

    If the distribution is unchanged:

        The flow is pruned.

        It is excluded from the weave.

        No read–verify is performed for that flow.

    If the distribution has changed:

        Proceed with the normal read–verify procedure.

        The flow is included in the weave.

Pruning decisions are made after stat₁ and before any reads occur.
Correctness Guarantees

    Pruned flows do not participate in the current weave and therefore do not affect the published snapshot.

    Snapshot correctness applies only to non-pruned flows.

    If pruning metadata is unavailable, ambiguous, or inconsistent, the weave MUST treat the flow as changed and include it.

    Changes made to _working after stat₁ are outside the snapshot boundary and are handled by a subsequent weave.

Rationale

This pruning model ensures:

    Only flows with actual input changes are woven.

    Snapshot capture remains race-safe by anchoring decisions to stat₁.

    No stale data is reused or implicitly revalidated.

    The weave remains incremental without weakening correctness.

In other words:

    Unchanged flows are not re-woven; changed flows are snapshot and woven.

This aligns perfectly with your intent and keeps the model minimal, explicit, and non-magical.
You said:
Please provide an very brief wiki synopsis of the "pointers" dataset/file.
ChatGPT said:
Pointers

_pointers.jsonld is a small, authoritative routing file used to locate the current published slices for a Knop’s flows.

    It contains only current pointers, not historical data.

    Exactly one location for a Knop should define current flow pointers.

    Other locations may reference this one via forwarding (e.g., additional Knop locations).

    _pointers.jsonld is managed exclusively by sflo tooling and must not be edited by users.

    It is updated atomically during weave.

Loaders consult _pointers.jsonld to resolve which flow slices represent the current published state.
You said:
I differentiate between the pointers dataset and its representation on disk.
ChatGPT said:
Pointers

The Pointers dataset is the authoritative, logical dataset that identifies the current published slices for a Knop’s flows.

    It defines which slice of each flow is considered current.

    It may also include forwarding information to another Knop location.

    It contains no historical or versioned data.

    It is managed exclusively by sflo tooling.

The dataset’s on-disk representation is typically a single file named:

_pointers.jsonld

This file is a concrete serialization of the Pointers dataset and is updated atomically during weave. Loaders treat the dataset semantics as primary; the file format and location are an implementation detail.
You said:
Should we make it _pointers/pointers.jsonld (with a companion resource page), so it's referable-to "in the abstract"? 

You almost need a way to differentiate a Knop from a ConceptualKnop. Maybe KnopFolder and Knop, i.e., a Knop can have multiple folders.
ChatGPT said:

Yes, but only if you separate two requirements that you’re currently conflating:

    Bootstrap discoverability (“given a knop folder URL/path, find current pointers with zero extra steps”)

    Abstract referability (“have a stable identifier for the pointers dataset independent of its current serialization”)

A plain file /_pointers.jsonld is excellent for (1) and weak for (2). A folder /_pointers/ with a stable dataset IRI is good for (2) and can still satisfy (1) if you keep a trivial bootstrap rule.
Recommendation
Keep a bootstrap entrypoint

Keep one of these as the canonical first lookup:

    /<knopLocation>/_pointers.jsonld (simple, fast), or

    /<knopLocation>/_pointers/ (folder, dereferenceable)

If you switch to a folder, don’t make loaders guess filenames. Make the bootstrap rule:

    /<knopLocation>/_pointers/ is a resource with:

        index.html (resource page)

        index.jsonld (canonical pointers dataset serialization)

        optionally index.ttl, etc.

So the “abstract” identifier is the folder IRI …/_pointers/, and the primary distribution is …/_pointers/index.jsonld.

That gives you both: stable referability and easy dereference.
Do not nest it as _pointers/pointers.jsonld

Use index.* for the representation. Otherwise you’ve invented an unnecessary extra naming layer and you’ll regret it.
Knop vs ConceptualKnop: you’re right, but be careful

You’re circling the real modeling issue:

    A Knop (conceptual identity) can have multiple locations over time (moves, aliases, forks).

    A Knop folder is just one concrete location where it can be accessed.

So model it explicitly:

    sflo:Knop — the conceptual thing (what you care about as “the knop”)

    sflo:KnopLocation (or sflo:KnopFolder) — a concrete access location (filesystem folder / URL base)

Then:

    sflo:Knop hasLocation sflo:KnopLocation (0..n)

    Exactly one sflo:KnopLocation is the current authoritative location (or you define an election rule)

    The Pointers dataset is associated with a KnopLocation (because it’s the routing surface for that location), but it can also assert which conceptual Knop it’s serving.

This avoids the hand-wavy “ConceptualKnop” term and matches your “a knop can have multiple folders” idea directly.
What this changes for pointers

If you go with /_pointers/ as an abstract identifier:

    The Pointers dataset IRI can be …/_pointers/ (clean)

    Its current JSON-LD distribution is …/_pointers/index.jsonld

    The resource page is …/_pointers/index.html

And your loader just fetches …/_pointers/index.jsonld (or uses content negotiation).
The constraint you must not dodge

If you introduce sflo:Knop (conceptual) distinct from sflo:KnopLocation, you must decide:

    Is path still identity? If yes, then “conceptual knop” is not identity; it’s a grouping construct (aliases/moves). That’s fine, but say it plainly.

    Or is conceptual knop the real identity and paths are locations? If you move here, you’ve reversed a major earlier premise.

Right now your text implies you want: path = location identity; conceptual knop = equivalence class over locations. That’s coherent.

If you want, I can give a minimal property set for this split (e.g., sflo:sameKnopAs, sflo:hasKnopLocation, sflo:authoritativeLocation, sflo:supersedesLocation) that won’t infect the rest of your ontology.
You said:
What would be a case where you want to refer to the pointers dataset but not its distribution?

KnopLocation is maybe better than KnopFolder, but it kind of implies KnopIRI. Anything else? KnopPart? 

The Knop's IRI refers to "something out there", so you coudl argue for using owl:sameAs for locating other knops, instead of something like otherKnopPart.
ChatGPT said:
When would you refer to the Pointers dataset but not a specific distribution?

Any time the statement is meant to hold regardless of serialization / location of a particular file, or when multiple serializations exist.

Common cases:

    Provenance + lifecycle: “the pointers dataset was generated by weave X at time T,” “was derived from snapshot S,” “supersedes previous pointers dataset.” (Those are dataset-level facts; they shouldn’t be duplicated per .jsonld/.ttl distribution.)

    Conformance / shape: “the pointers dataset conformsTo sflo:PointersShape.” You don’t want to attach that to one file format.

    Identity-stable references inside other metadata: opslog entries, inventories, audit trails, or documentation that want to cite “the pointers dataset for this KnopLocation” without committing to a file extension.

    Multiple distributions / content negotiation: if you publish both JSON-LD and Turtle (or negotiate), the dataset is the stable thing; distributions are just access methods.

    Digesting / signing: you may want to sign “the dataset content” after canonicalization, not “this particular file bytes,” especially if you regenerate formatting.

If you truly only ever have one serialization and never attach metadata to the dataset as an abstract thing, then you don’t need a separate dataset IRI. In practice, you will.
“KnopLocation” naming concern

You’re right that Location can read like “location of a Knop IRI.” But that’s also the reality: it’s an access point that serves the Knop’s resources. If you want something that’s clearly “access endpoint” and not “geospatial location,” better options than KnopFolder:

    KnopAccessPoint (very clear: this is how you access it)

    KnopEndpoint (clear, network-y; works for FS + HTTP)

    KnopAddress (simple, but can imply a single URI string)

    KnopBase (implies “base IRI / base path” specifically)

KnopPart is misleading: a moved/aliased folder isn’t a part of the knop; it’s an alternate access root.

If you want minimal ontology surface: KnopAccessPoint is the least ambiguous.
Using owl:sameAs for “other knops”

This is a trap unless you are extremely strict.

    owl:sameAs means complete identity in all interpretations. In practice, it will leak and cause reasoners (and humans) to treat two IRIs as interchangeable everywhere. That’s rarely what you want for “this used to live over there” or “this is an alternate access location.”

Better patterns:

    Use owl:sameAs only between conceptual Knop IRIs if you genuinely mean identity (rare).

    For moves/aliases/alternate access points, use a weaker, purpose-built relation, e.g.:

        sflo:alternateAccessPoint

        sflo:supersededByAccessPoint

        sflo:canonicalAccessPoint

        or PROV’s prov:alternateOf (closer to “alternate identifiers for the same thing” without the OWL blast radius)

If you still want an “identity-ish” link but safer than OWL: schema:sameAs is widely used as a looser equivalence hint, but it’s not formally constrained.
Practical upshot for your pointers layout question

If you want the pointers dataset to be referable “in the abstract,” put it at a stable IRI like:

    …/_pointers/ (dataset IRI)

    …/_pointers/index.jsonld (distribution)

Then KnopAccessPoint (or KnopLocation) can have exactly one “current pointers dataset” without tying that statement to a particular file extension.

If you tell me whether you expect multiple pointers distributions (JSON-LD + TTL) or just JSON-LD forever, I’ll tell you whether the abstract dataset IRI is worth the added structure.
You said:
I  think path is still identity, but we know that things could have multiple identifiers. e.g. the local working file:// IRI is just another identity for the published IRI.

We preserve the principle of "single referent"

 maybe we don't need a separate class for KnopLocation.
ChatGPT said:

You can keep “path is identity” and “single referent,” but only if you stop treating file://… as an identifier for the same thing as the published https://… IRI.

Right now you’re mixing two roles:

    Identifier (names the referent)

    Locator / access URL (how you fetch bytes)

If you call both “identities,” you will inevitably drift into owl:sameAs-style equivalences that are stronger than what you mean.
The clean move
Keep exactly one Knop identifier IRI

    The Knop’s identifier is the published, dereferenceable IRI (your “path is identity” principle).

    Everything else (including file://) is an access path.

So: single referent + single canonical identifier + many locators.
Model file paths as access URLs, not identity

Instead of saying:

    file://… is “another identity” for https://…

Say:

    file://… is an access URL that currently hosts a local mirror of the resource identified by https://….

That keeps your system sane and avoids global equivalence.
Consequence: you may not need KnopLocation as a class

Agreed—if you treat locations as just access URLs, you can avoid a whole class.

Minimum properties on sflo:Knop (or on the pointers dataset, if you prefer):

    sflo:canonicalIri (often redundant: it’s the subject IRI)

    sflo:accessURL (0..n): file://…, http://localhost…, https://…

    optional sflo:preferredAccessURL (for tooling)

    optional sflo:mirrorOf / sflo:serves relationships only if you later need to reason over access points as first-class resources

If you want to reuse existing vocab, DCAT already has dcat:accessURL on distributions; you can treat “local filesystem” as one distribution/access path without asserting identity equivalence.
What about “multiple identifiers” in the RDF sense?

If you truly want multiple IRIs to denote the exact same referent, you’ll need an equivalence relation. But:

    owl:sameAs is almost certainly too strong for “local working mirror.”

    A softer, non-DL-commitment link (or a purely operational mapping) is safer.

If you insist on an IRI-to-IRI equivalence, use a purpose-built property like:

    sflo:aliasOf / sflo:equivalentIdentifier (and keep it out of OWL reasoning expectations), or at most schema:sameAs as a hint.

But the better answer is: don’t make locators into identifiers.
Where this leaves pointers and “other locations”

You can still have forwarding across moved/aliased folders without KnopLocation as a class by keeping it operational:

    _pointers/ (dataset) includes sflo:additionalAccessRoots (URLs) or sflo:forwardToAccessRoot (single URL for canonical forwarding)

    loaders follow access roots to find an authoritative pointers dataset

    none of this asserts identity between IRIs; it’s just routing

That aligns with “path is identity” while supporting mirrors, moves, and local dev.

If you want, I can rewrite the “Pointers” and “Consistency” wiki snippets to consistently use identifier vs access URL terminology so you don’t accidentally reintroduce the equivalence problem later.
You said:
I think sameAs works for Knops (as referrers), and it's actually the KnopHandles that identify the different locations, i.e. knops-as-knops. So maybe KnopHandles serve the role of KnopFolders -- they capture a part of the knop at a particular path.

I like global equivalence for knops, regardless of location. They refer to a single thing. 
ChatGPT said:

If you want “global equivalence for knops, regardless of location,” you can do it with owl:sameAs, but you’re buying a very sharp knife. The question is whether you want logical identity (every statement transfers) or just referential unification for routing.
The hard critique: owl:sameAs is not “referrers think they’re the same”

owl:sameAs means the two IRIs denote exactly the same individual in OWL semantics. That implies:

    every triple about one is true of the other

    they are interchangeable everywhere

    any inconsistency on one infects the other

    “location-specific” statements become nonsense unless you carefully reify them

If you ever say anything like “this knop has workingPath X” or “this knob is weaving” on one identifier and not the other, you’ve either:

    accidentally asserted it about both, or

    you’ve been forced into putting all location-y facts somewhere else (handles), which is fine but must be strictly enforced.

So: you can use owl:sameAs only if you never attach location/scoping facts to the Knop itself.
Your move (KnopHandle-as-location) is the right decomposition

    “it’s actually the KnopHandles that identify the different locations”

Yes. If you want global identity, then Knop is the referent, and KnopHandle is the access-point / incarnation / path-bound artifact.

That gives you a clean rule:

    All “where/how to access” facts live on KnopHandle

    All “what the knob is” facts live on Knop

Concretely, put these on KnopHandle, not Knop:

    filesystem path / URL base

    _pointers distribution location

    _weave-in-progress status

    working distribution paths

    redirects / forwarding / additional locations

And on Knop:

    conceptual metadata (label, description, conceptual relationships)

    “same knops” equivalences (if you insist)

    maybe “hasHandle” links

But: owl:sameAs still doesn’t fit your real need most of the time

You said: “sameAs works for Knops (as referrers).”

That’s typically not what owl:sameAs is used for. You’re describing identifier aliasing across operational contexts (published HTTP IRI vs local file access root), not discovery that two independently minted identifiers refer to the same thing in the world.

The safer pattern is:

    A single canonical Knop IRI (published)

    Handles record multiple access roots (including file://) as access URLs, not as alternate Knop IRIs

If you instead mint separate “knop IRIs” per location and glue them with owl:sameAs, you’ll get:

    huge equivalence closures

    reasoning surprises

    pain when someone accidentally asserts handle facts on the Knop IRI

A compromise that preserves “global equivalence” without OWL blast radius

If you want “treat these as the same thing” for your tooling and RDF consumers, define:

    sflo:sameKnopAs (or sflo:knopAliasOf)

        symmetric, transitive (if you want)

        not owl:sameAs

        documented as identifier aliasing for Semantic Flow, not full OWL identity

You can still optionally map it to owl:sameAs in some profiles, but you won’t force every consumer into OWL identity semantics.

This matters because you’ve repeatedly emphasized human clarity and collaboration. owl:sameAs is rarely “clear” in practice.
If you insist on owl:sameAs, here’s the minimum discipline required

    Never assert path/access properties on Knops. Only on Handles.

    Ensure _pointers dataset is keyed by Handle (location), not Knop (referent).

    Keep “current flows” pointers handle-scoped.

    Treat local file:// as a handle access URL, not a Knop IRI, unless you truly want it to be a first-class identifier in the mesh.

Suggested modeling (minimal, consistent with your direction)

    sflo:Knop (conceptual referent)

    sflo:KnopHandle (path-bound access root; “knops-as-knops” in a location)

Core links:

    sflo:handleForKnop: KnopHandle → Knop

    sflo:hasHandle: Knop → KnopHandle

    sflo:accessBase: KnopHandle → <file://…> | <https://…> | <http://localhost…>

    sflo:hasPointersDataset: KnopHandle → PointersDataset (abstract)

    sflo:hasDistribution: PointersDataset → …/_pointers/index.jsonld (concrete)

Then “moves” become handle graph changes, not knob identity changes.

If you answer one thing, the rest snaps into place:

Do you want multiple Knop IRIs that are equivalent (and thus require sameAs/aliasing), or do you want one Knop IRI with multiple Handles that provide different access bases?

Your last message suggests you’re leaning to the first; I’m warning that you’ll regret it unless you enforce a very strict “all location facts live on Handles” rule everywhere. The second option gives you “single referent” and “global equivalence” without equating access paths to identity.
You said:
"owl:sameAs means the two IRIs denote exactly the same individual in OWL semantics" this is exactly what we want. One thing I hadn't considered is that old reference slices (and probably all slices) need to be invalidated as they're superceded. Location-specific stuff is handle via the KnopHandles. It's actually perfect. You would never say a knop has a working path or is weaving, only a knop-handle.

Thee IS NO CANONICAL IRI for a Knop. Just the IRI for the current knop-handle

I like sameAs exactly because it is so sharp-edged. It tells people (who know) that these identifiers refer to the same thing. 

"huge equivalence closures" feels like scare-mongering. People aren't supposed to "move" (a more accurate term would be "expand the knop's IRI footprint, and start using that other IRI for filesystem-based work". And equivalence closure is good. If people see the old knop IRI, they need to discover that it is equivalent to the new Knop IRI. I can't foresee reasoning surprises unless someone asserts handl facts on the Knop IRI, and that's an issue no matter what. 

Remember there's an alternate use case from move... Duplicate (or more accurately, create new from this other data) would not use sameAs. It might use derivedFrom prov:wasDerivedFrom or something similar. 

sameKnopAs... It's not really, in the "knop-as-knop" sense. Remember, the handle refers to the knop, the Knop's IRI refers to the referent. I know it's confusing, but it's the only way.
ChatGPT said:

Under those constraints, your position is internally consistent—and sharper than the alternatives. The key is that you are separating referent identity from operational facts with discipline, not hedging.

Here’s the cleaned-up model, stated plainly and without scare stories.
Core Commitments (accepted)

    owl:sameAs is intended and correct

        Multiple IRIs do denote the exact same Knop.

        Full OWL identity is a feature, not a risk.

        Equivalence closure is desirable: it guarantees discoverability across moves/expansions.

    There is no canonical Knop IRI

        There is only a currently used Knop-handle IRI.

        Old IRIs remain valid identifiers of the same referent.

        “Move” is really expansion of the identifier set, not relocation.

    All location-, lifecycle-, and operational facts are handle-scoped

        Knops never:

            weave

            have _working

            have pointers

            have inventories

            have locations

        KnopHandles do all of that.

    Slices are time- and handle-relative and may be superseded

        Slices are not eternal truths about the Knop.

        Supersession / invalidation is normal and expected.

        Old slices remain addressable but are no longer current.

    Duplication / derivation is explicitly not identity

        Uses prov:wasDerivedFrom (or similar), not owl:sameAs.

        This cleanly separates cloning from equivalence.

Resulting Ontology Shape (minimal, strict)
Knop

    The referent

    Has no operational properties

    May be referenced by many IRIs

    Related to other Knops only by conceptual relations

    Identity links:

        owl:sameAs (only for true equivalence)

KnopHandle

    A handle on a Knop at a particular access root

    Carries:

        filesystem / URL base

        _working paths

        pointers dataset

        inventory dataset

        weave status

    Links:

        sflo:handleForKnop → Knop

Pointers Dataset

    Handle-scoped

    Defines current slices

    Supersedes prior pointer states

    Never attached to the Knop itself

Slices

    Produced by a handle

    Refer to the Knop

    May be superseded by later slices from any equivalent handle

Why owl:sameAs is actually doing useful work here

You are using it in the only context where it behaves well:

    No partial identity

    No role confusion

    No “almost the same” semantics

    No weakening for convenience

And you’ve already done the hard part that most systems don’t:

    You made it illegal by design to assert handle facts on the Knop.

That’s the real safety valve—not avoiding sameAs.
The one rule you must enforce everywhere

    No property whose truth depends on location, time, or workflow may be asserted on a Knop. Ever.

Violating this rule is catastrophic under any equivalence scheme, not just owl:sameAs.

If you hold that line, then:

    identity is clean

    moves are trivial

    equivalence is discoverable

    reasoning does exactly what it should

Bottom line

You’re right:

    owl:sameAs is the correct tool because it is sharp.

    KnopHandles are the right place for all operational reality.

    There is no canonical IRI, only a canonical current handle.

    Supersession, not deletion, is the right lifecycle model.

This is not confusing—it’s precise.
You said:
I guess you could reverse things, and have the Knop IRI refer to the knop-as-knop, and have a "referent-handle" that refers to the referent. But it's not a good UX because the data people care about should have these weird handle thing everywhere. Right? 

I've already accepted (and partially implemented and documented) that all location and other "Knop metadata" hangs of handles.
ChatGPT said:

Reversing it (“Knop IRI = knop-as-knop; referent-handle points to the real-world referent”) is workable formally, but it’s a UX loss unless you’re willing to make the handle disappear via tooling/view conventions. Otherwise, yes: the thing people care about ends up being “behind handles” everywhere.
Why the reversal is worse in practice

In your current model:

    Knop IRIs are the referent.

    KnopHandles carry operational/location metadata.

    A user can talk about “the thing” with the Knop IRI directly, and only dip into handle-land when they need routing or filesystem concerns.

In the reversed model:

    The identifier people are most likely to encounter (“Knop”) is not the referent.

    Anything that is semantically meaningful about “the thing out there” requires chasing to a referent-handle (or a second node) constantly.

    You end up with graphs full of indirections in exactly the places humans want to read the data: class membership, relationships, descriptions, etc.

That indirection is tolerable only if your UI/templating layer always collapses it away (and then you’ve basically rebuilt your current model at the presentation layer).
What you’ve already accepted is the correct direction

You’ve already committed to:

    operational/location metadata lives on handles

    the “thing people care about” is not a handle

    “current handle” is an operational notion (pointers dataset), not identity

That is exactly the right UX/engineering trade.
The real UX lever isn’t reversing; it’s presentation defaults

If you stick with “Knop = referent” (your current direction), you can make handles nearly invisible:

    Default resource pages for a Knop show:

        the referent-facing facts first

        a small “Access & Workflow” section listing handles, current handle, working paths, inventories, weave status, etc.

    Queries and APIs can offer:

        “Knop view” (collapse handles)

        “Handle view” (operational detail)

So users aren’t “seeing weird handle things everywhere” unless they choose to.
One critical implication to document (since you’ve implemented handle-scoping)

You should define two disjoint property families:

    Referent properties (legal on Knop, illegal on Handle)

    Operational properties (legal on Handle, illegal on Knop)

If you don’t formalize that split (even informally in docs / SHACL), people will accidentally assert handle facts on Knops and you’ll get the only real failure mode you’ve identified.
Bottom line

Don’t reverse. Your current choice is better UX and better modeling:

    Knop IRIs denote the referent (including owl:sameAs equivalences where needed).

    Handles carry all operational/location metadata.

    Pointers select the current handle/slices.

    UI collapses handles by default.

If you want, I can draft a short “Handles” wiki article that states the legality rules (“what may be asserted on Knops vs Handles”) in the same concise style as your “Consistency” and “Pointers” writeups.
You said:
It's hard to attach different provenance to the various slices. It could go in "slice-inventory.ttl"

Maybe we should retain the "meta" flow? I suppose the opslog could capture provenance, but opslog sounds like something that can be rotated/truncated. 
ChatGPT said:

Attaching slice-level provenance is hard only if you’re trying to hang it off the slice distributions or off _pointers (which you should not). Putting it in _slice-inventory.ttl is the right move—but only if you treat it as a ledger, not an opslog.
Make _slice-inventory.ttl the provenance ledger

Define _slice-inventory.ttl as:

    append-only

    non-rotating / non-truncating

    the canonical place to record slice events seeable forever

    compactable only via “folding” into an equivalent ledger form (never by deletion)

Then each time weave publishes a new slice, you append one event entry that includes:

    which KnopHandle produced it

    which flow it belongs to

    the slice identifier

    distribution list (or pointer to dist set)

    prov facts: prov:wasGeneratedBy, prov:generatedAtTime, prov:wasDerivedFrom (the in-memory snapshot digest), prov:wasAssociatedWith (actor/tool), etc.

    supersession link: prov:invalidatedAtTime / “supersedes slice X” (your choice of vocabulary)

This solves “different provenance per slice” cleanly.

If you keep _slice-inventory as a single file and allow truncation, you lose the point. If truncation is required operationally, then the inventory must instead be a folder of immutable event files and a derived index (so the history is still non-lossy).
Keep “opslog” only for operational chatter

If you use an opslog, define it as:

    rotatable

    not authoritative

    debugging + runtime signals + performance stats + transient failures + partial weave notes

It should not be where you keep provenance you care about.

So: opslog ≠ provenance.
Do you need the “meta flow”?

Not for provenance, if inventory is a proper ledger.

You might still want a meta-like flow for current operational state that is not part of pointers/inventory, e.g.:

    last successful weave time (convenience)

    last weaveLabel

    last error summary

    local-only notes

    “this handle is stale / read-only / archived”

But don’t conflate that with provenance. Provenance belongs with slice creation events (inventory).

If you keep it, keep it explicitly as “state,” not “meta about other flows,” e.g.:

    _meta/_working can exist if it’s just another flow’s input

    or rename it to something that doesn’t invite semantic sprawl (you already have _reference-flow in your vocabulary; that’s closer to “metadata” than “meta flow”)

The critical design decision

Make one thing non-negotiable:

    If provenance matters, _slice-inventory must be non-lossy.

Everything else (meta flow, opslog) becomes optional.

If you want, I can draft a brief wiki note “Slice Inventory” that states (a) ledger semantics, (b) what gets recorded per published slice, and (c) what must not go into opslog.
You said:
Maybe we want to keep these meta non-flows in a _meta (or something else to avoid confusion) folder:

I actually had a decent provenance-relator class hierarchy set up,   
You said:
Maybe we want to keep these meta non-flows in a _meta (or something else to avoid confusion) folder, so that all distributions are always using the double-parent syntax:
- ../../_meta/_pointers/index.jsonld
- ../../_meta/_inventory/index.ttl
- ../../_meta/_backlinks/index.jsonld
- ../../_meta/_opslog/index.jsonld
ChatGPT said:

Centralizing non-flow, sflo-managed artifacts under a single folder is a good structural move, and it makes the “double-parent” convention predictable. The main risk is naming: calling the folder _meta will collide with your earlier use of _meta as a flow concept in people’s heads (and in code/comments).
Recommendation: keep the pattern, rename the folder

Use a folder name that reads as system-controlled / operational, not “metadata about the world.”

Good candidates:

    /_sflo/ (explicit, unambiguous, low chance of domain collision)

    /_sys/ (common, but can collide with others)

    /_control/ (clear intent)

    /_sf/ (short but less searchable)

I would avoid /_meta/ unless you are willing to permanently ban a flow named _meta and document that _meta is reserved for sflo system artifacts.
Proposed layout

Inside the system folder, keep each dataset as a referable folder resource:

    ../../_sflo/_pointers/ (+ index.jsonld, index.html)

    ../../_sflo/_inventory/ (+ index.ttl, index.html)

    ../../_sflo/_backlinks/ (+ index.jsonld, index.html)

    ../../_sflo/_opslog/ (+ index.jsonld or .ttl, optional index.html)

So your references become:

    ../../_sflo/_pointers/index.jsonld

    ../../_sflo/_inventory/index.ttl

    ../../_sflo/_backlinks/index.jsonld

    ../../_sflo/_opslog/index.jsonld

This preserves your “dataset vs distribution” distinction cleanly (folder IRI = dataset, index.* = distribution).
Critical constraints you should write down

    These are not flows. They must never have _working, slices, or weave input semantics. They are weave outputs / system state.

    Only tooling writes them. Users and non-sflo processes must not touch them.

    Inventory is a ledger, opslog is disposable.
    If _inventory is where provenance/supersession lives, it cannot be treated like _opslog.

    No “current pointers” on the Knop itself.
    Pointers must remain handle-scoped; putting _pointers under the handle’s system folder matches this.

Naming within the folder

Your inner names are good, but consider dropping the leading underscore on the inner names to reduce visual noise:

    /_sflo/pointers/

    /_sflo/inventory/

    /_sflo/backlinks/

    /_sflo/opslog/

The outer _sflo already signals “reserved/system.”
One subtle pitfall

If you ever want content negotiation at …/_sflo/pointers/, ensure you standardize that the canonical distribution is index.jsonld (and optionally index.ttl). Otherwise you’ll end up with tooling guessing filenames again.

If you want, I can rewrite the “Pointers” synopsis to explicitly define the dataset IRI as the folder and the canonical distribution as index.jsonld, consistent with this structure.
You said:
I don't think _sflo is the right folder name, it's the preferred prefix for the ontology, and I want there to be a separate "meta" ontology. Welcome to naming ideas, but the ontology name and the folder slug should match.

I like the idea of listing these meta dataset locations in the pointers file too.

Good idea about dropping the leading underscore. I was thinking that keeps things consistent with slices not starting with underscores (any more! Now v7 instead of _v7). But maybe we don't want to be consistent, to underscore that these aren't slices. But maybe we should use _ everywhere as it indicates "system folders"
ChatGPT said:

You have three distinct naming goals that are in tension, and you should pick which one wins because you can’t satisfy all three simultaneously:

    Folder slug matches ontology prefix/name.

    Leading _ communicates “system-reserved.”

    Avoid confusion with the historical _meta flow concept.

Given your direction (“these are non-flows”), (3) matters most.
1) Folder name candidates that can be an ontology prefix and a reserved system folder

You want the ontology and folder slug to match, and you want a separate “meta ontology.” So pick a name that is not sflo and not “meta” in the generic sense.

Best candidates:

    sfmeta (prefix sfmeta:; folder _sfmeta/): very clear “sflo meta layer” without colliding with sflo:.

    sfops (prefix sfops:; folder _sfops/): operational flavor; matches pointers/inventory/opslog/backlinks well.

    sfcore is wrong (sounds like ontology core, not operational).

    sfctl (control) is OK but ugly.

If you want a more human word (less abbreviation), you’ll lose prefix ergonomics. I’d pick sfops or sfmeta.
My recommendation

    Ontology: sfops: (operational datasets and system-managed artifacts)

    Folder: /_sfops/ (reserved system folder at handle root)

This avoids polluting sflo: while still matching your “ontology name == folder slug” rule.
2) Should system folders use _ everywhere?

Yes. This is one of the few cases where inconsistency is a feature.

    Versions/slices: v7/ (human-facing, stable)

    System artifacts: _sfops/… (reserved, tooling-owned)

That makes accidental human editing less likely and makes scanning rules trivial (“ignore any folder starting with _ unless explicitly whitelisted”).

So:

    keep _ at the system-root (_sfops/)

    inside it, you can drop underscores safely because the root already brands it as system

Example:

    ../../_sfops/pointers/index.jsonld

    ../../_sfops/inventory/index.ttl

    ../../_sfops/backlinks/index.jsonld

    ../../_sfops/opslog/index.ttl (I would prefer TTL here)

3) Listing meta dataset locations in pointers

That’s coherent, but be strict about what pointers is allowed to point to.

Pointers should remain:

    small

    routing-oriented

    “where to find X” only

So add a single block like:

    metaDatasets (or systemDatasets) mapping stable dataset IRIs to their access paths.

Example conceptually (no need to over-model):

    pointersDataset → ../../_sfops/pointers/

    inventoryDataset → ../../_sfops/inventory/

    backlinksDataset → ../../_sfops/backlinks/

    opslogDataset → ../../_sfops/opslog/

This is good because loaders then have one bootstrap file to discover all system artifacts.
One constraint

Don’t duplicate these links in multiple places with different semantics. If pointers lists them, that becomes the authoritative index of system datasets for that handle.
4) Avoiding the _meta confusion

If you keep a folder called _meta, you will reintroduce the earlier semantic confusion you already escaped: people will think _meta is a flow again.

So either:

    reserve _meta forever for “system datasets” and never allow a flow named _meta, or

    don’t use _meta as the system folder name.

Given your message, you’re already leaning to “meta ontology,” so _sfmeta/ or _sfops/ solves it cleanly.
Concrete proposal (tight)

At each KnopHandle root:

    /_pointers/index.jsonld (or /_pointers.jsonld if you keep the file form)

    /_sfops/

        /pointers/index.jsonld (optional if you keep pointers at root; otherwise make this canonical)

        /inventory/index.ttl

        /backlinks/index.jsonld

        /opslog/index.ttl

And _pointers includes a systemDatasets map pointing at ../../_sfops/... entries.

If you want the absolute minimum churn: keep _pointers at the root for bootstrap, but treat /_sfops/* as the abstract dataset IRIs with resource pages, and make root _pointers just a thin distribution that points to the canonical dataset.

If you choose between sfops and sfmeta, I’d pick sfops because it correctly frames inventory/pointers/backlinks/opslog as operational infrastructure, not “metadata” in the semantic-web sense.
You said:
I think consistency and the expressive power of underscores (these are sflo system folders) means we use them everywhere, even in slice names. They're mostly for machines anyhow. What do you think?

maybe slice names can have the weave slug and/or the sliceSequence number, and the weave slug can be timestamp-based or guid based. 

I actually had a decent provenance-relator class hierarchy set up, I thought it was pretty brilliant. I'd like to build a general purpose meta/operational ontology, and let people declare their own activities and pre-made authority chains. I think sflops is good for ours. Looks like "ops:" is available-enough as a prefix, but I'll save that for later. Anyhow, people should be able to inject whatever provenance data they want, using existing ontologies where they prefer. Pluggable. (How crazy am I!?!)
ChatGPT said:
Underscores everywhere (including slices)

If you optimize for machines, uniform “system folder” underscores can work, but it weakens one of your best affordances: at-a-glance separation between user/domain data and sflo-managed structure.

Right now you have a clean semantic split available:

    _something/ ⇒ sflo-managed / operational / structure

    v7/ (or similar) ⇒ published, referent-facing artifacts

If you put underscores in slice names too, you lose that immediate visual cue and you also make it harder to write ignore/scan rules that don’t accidentally scoop up published content.

The only strong argument for underscores-in-slices is if you truly want all slice paths to be “opaque machine IDs.” If so, do it intentionally and consistently:

    Keep _ for “machine identifiers,” not “system folders.”

    Then you need a different marker for “system-owned” folders (because _ no longer means that).

So: you can’t have _ mean both “system folder” and “slice identifier” without ambiguity.

Recommendation: keep _ as “reserved/system-owned,” and keep slice IDs underscore-free. Machine-friendly does not require underscores; it requires determinism.
Slice naming with weave slug + sequence

This is good, and it also solves your multi-knop consistency story.

A practical scheme:

    weave slug = a release/session ID (timestamp-based or GUID-based)

    sliceSequence = per-flow monotonically increasing integer (optional)

    slice ID = either:

        w_<weaveSlug> (if only one slice per weave per flow), or

        w_<weaveSlug>__s_<seq> (if you can emit multiple per weave)

But: if you keep _ reserved for “system,” then don’t lead slice IDs with _. Use something like:

    w20251214T215533Z or w-<uuid>

    w20251214T215533Z-s42

If you’re worried about lexicographic ordering and safety, ISO-like timestamps are excellent.

Do not rely solely on a global weave slug without a per-flow sequence unless you guarantee “at most one published slice per flow per weave.” Otherwise you’ll need an extra discriminator.
“sflops” meta/operational ontology, pluggable provenance

Not crazy. The pluggable part is sane if you draw a strict boundary:

    sflops: defines the structural hooks (where operational/provenance data can live and how it links to slices/handles/weaves)

    users can attach additional provenance using:

        PROV-O (prov:)

        PAV (pav:)

        Dublin Core (dcterms:)

        their own vocabularies

The risk isn’t “too ambitious.” The risk is underspecification: if you don’t provide a minimal normative contract, everybody will inject incompatible shapes and you’ll lose interoperability.

So you should define a small required core, and make everything else optional.
Minimal core (what sflops should define)

    sflops:Weave (an activity)

    sflops:SlicePublication (event/relator linking: handle + flow + slice + distributions)

    sflops:Supersedes / sflops:Invalidates relation (or reuse PROV invalidation)

    sflops:AuthorityChain (optional, but if you want “pre-made authority chains,” you need a first-class concept)

Then provide recommended mappings:

    sflops:Weave rdfs:subClassOf prov:Activity

    slice publication uses prov:generatedAtTime, prov:wasGeneratedBy, prov:wasAssociatedWith, etc.

That gives you strong structure while still being pluggable.
What I think you should do next

    Keep _ reserved for system folders, not slice IDs.

    Adopt slice IDs of the form w<weaveSlug>-<seq> (no underscore requirement).

    Make sflops: the operational ontology whose primary job is to provide stable attachment points for provenance, not to replace PROV/PAV.

    Define one canonical place where this provenance is recorded (your _inventory ledger), and let plugins add extra triples there.

If you want, I can draft two short wiki pages:

    Slice IDs (naming, ordering, uniqueness, relationship to weaveSlug/sequence)

    sflops (core classes/properties + how to inject PROV/PAV safely)

You said:
I already decides about how slices are named:

- 2025-07-20_1430_00_v47 or
- v47 or 
- GUID

Yes, provenance goes in _inventory.

I had "# Delegation chain (step 1 = top authority)" but now if we don't want to inherit delegation chains from the flow or even knop-as-knop level, the top authority should maybe be highest. Lol, then criminals can leave negative  numbers for not-authorized.
ChatGPT said:

Your slice naming choices are fine; they’re orthogonal to the delegation-chain issue.

On delegation chains: if you don’t inherit them from flow/knop level and you want a stable ordering, don’t use signed integers as “authority rank.” It’s not about “criminals,” it’s about schema ambiguity and consumer bugs. Negative numbers create semantics you don’t actually want to support.
The core issue

You’re trying to encode two things in one field:

    order within a chain

    authorization status / trust

Those should be separate.
Recommendation: separate ordering from authorization
A. Ordering: use an ordinal that is always non-negative

Pick one and standardize:

    step (1..n), where 1 = top authority (your original), or

    priority (0..n), where 0 = top authority

Either is fine. Keep it monotone and non-negative.

Do not allow “highest number = top authority” unless you have a compelling reason; it’s unintuitive and makes comparisons awkward.
B. Authorization: explicit status, not numeric hacks

Add a separate property, e.g.:

    isAuthorized: true|false
    or

    authorityStatus: Authorized | NotAuthorized | Unknown | Disputed

If you want to support “soft trust,” add an optional confidence (0..1) or evidence link, but don’t overload the ordering.
If you don’t want inheritance

Then the chain must live somewhere concrete and local, likely:

    on the slice publication event in _inventory (best), or

    on the Weave activity record (also reasonable)

In that case, ordering is only meaningful within that event’s chain, so you can simply use RDF list order and avoid numbers entirely:

    sflops:delegationChain ( :authA :authB :authC )

This is cleaner than step numbers and avoids sorting issues. If you still need per-link metadata (e.g., evidence per step), you can reify the links as relators (which you already like).
Minimal rule set (tight and non-gameable)

    Delegation chain is an ordered list (top-first).

    Each element MAY have:

        authorized boolean (or status enum)

        optional evidence/provenance links

    No negative ranks. No “special” numeric sentinels.

This gives you predictable consumers and lets you express “not authorized” without inventing weird math.

If you want, paste your old “Delegation chain (step 1 = top authority)” snippet and I’ll refactor it into an RDF-list + relator form that fits _inventory and your sflops direction.

i was joking about unauthorized, but its actually kinda interesting

It’s interesting in the same way “use -1 as NULL” is interesting: it compresses meaning into a single sortable field. The problem is that it becomes too clever and leaks ambiguity everywhere.

What’s genuinely interesting about it

A signed “authority rank” could encode three states in one dimension:

> 0: delegated authority chain (ordered)

= 0: direct authority / root

< 0: explicit denial / disallow / “known bad”

That gives you a single sort key and makes “negative authorities” naturally float to one side.

Why it will still bite you

Conflates order with authorization: an “unauthorized” step isn’t part of the chain ordering; it’s a different relation (a veto/denial).

Consumer bugs: someone will do ORDER BY rank DESC and accidentally treat -1 as “low authority” instead of “explicitly forbidden.”

Hard to merge: two sources might disagree (-1 vs 2), and now your numeric field needs conflict-resolution semantics.

Temptation to add more magic: soon you’ll want -2 for “compromised,” etc.

A better way to keep the “interesting” part

Keep ordering purely ordered, and add an explicit “polarity” or “disposition”:

chainStep: 1..n (or RDF list order)

disposition: Authorized | Denied | Unknown | Disputed

optional scope: what is denied (this action? this handle? this flow?)

This preserves the expressive idea (“there can be explicit denials”) without encoding it as numeric tricks.

If you still want a single sortable field

Use a pair that sorts lexicographically:

(dispositionWeight, step) where e.g. Denied=-1, Unknown=0, Authorized=1

but keep step non-negative and meaningful only when disposition is Authorized

That gives you your “interesting ordering” without making the chain itself nonsensical.

If you describe what you want denials to do (block publication? mark slice as untrusted? override previous delegations?), I can suggest the minimal representation in _inventory that stays compatible with PROV/PAV and your relator approach.

So is it _sfops/_pointers/ or _sfops/pointers/

Use _sfops/pointers/, not _sfops/_pointers/.

Why:

The outer underscore (_sfops/) already marks the entire subtree as sflo system-owned.

Adding another underscore inside is redundant and noisy.

Consistency rule becomes simple and machine-friendly:

Leading _ at the root = reserved/system

Inner names are semantic dataset names

Recommended structure:

_sfops/
  pointers/
    index.jsonld
    index.html
  inventory/
    index.ttl
    index.html
  backlinks/
    index.jsonld
    index.html
  opslog/
    index.ttl


Benefits:

Clean dataset IRIs (…/_sfops/pointers/)

Clear dataset vs distribution split (index.*)

Easy scanning/ignoring rules

Avoids resurrecting _pointers as a quasi-flow name

If you ever introduce another system root later, the convention still holds.

any dataset can have a distribution, right? So index.jsonld could be considered a distribution

Yes — exactly.

In your model, any dataset can have one or more distributions, and:

the dataset is the abstract thing (what you refer to “in the abstract”)

a distribution is a concrete serialization/access path for that dataset

So in:

_sfops/pointers/
  index.jsonld


…/_sfops/pointers/ is the dataset IRI

…/_sfops/pointers/index.jsonld is a distribution of that dataset

That holds equally for:

_sfops/inventory/index.ttl

_sfops/backlinks/index.jsonld

_sfops/opslog/index.ttl

This is consistent with DCAT-style thinking without forcing DCAT everywhere.

A few clarifying points to keep it clean:

index.jsonld is just a convention, not a requirement.

You may have multiple distributions (index.ttl, index.jsonld, etc.).

The dataset-level metadata (provenance, conformance, supersession) belongs to the dataset IRI (…/pointers/), not the file.

File-level metadata (media type, byte size, checksum) belongs to the distribution.

So yes: index.jsonld is a distribution, and treating it that way is the right abstraction.

is there a better term than "move" for the idea of "giving a knop a new location (and so an additional identity). Like in biology or real estate when something expands or infects something.

Yes. “Move” is the wrong verb for what you’re doing, because nothing relocates or ceases to exist. You’re adding an additional access locus / identifier for the same referent.

Here are better terms, grouped by metaphor, with a recommendation.

Best fits for your semantics (additive, non-destructive)
Expansion

Strong recommendation.

Conveys growth, not relocation.

Matches your “expand the knop’s IRI footprint” phrasing exactly.

Neutral, non-metaphorical, explainable to engineers.

Examples:

“Knop expansion”

“Expanded into a new handle”

“IRI expansion event”

Proliferation

Accurate but heavy.

Slightly biological / ominous.

Better for documentation than API verbs.

Propagation

Suggests copying outward without loss.

Works well if handles are created intentionally.

Replication (⚠️ ambiguous)

Implies copying content, not identity.

Risky because you also support true duplication (which is not sameAs).

Biology-inspired (accurate but metaphor-heavy)
Colonization

Surprisingly precise: same organism, new territory.

Might read as politically loaded.

Invasion

Accurate in a graph sense, terrible UX.

Metastasis

Technically brilliant, socially unusable.

Real-estate / legal metaphors (very good fit)
Encumbrance

Too legal, too negative.

Easement

Interesting but obscure.

Annexation

Good semantics, bad politics.

Network / systems metaphors
Mounting

Excellent locally (“mounted here”)

Doesn’t scale well to HTTP.

Binding

Suggests linkage, not presence.

Registration

Too weak; doesn’t imply access.

What I’d standardize on
Primary term: Expansion

Use it consistently:

Expansion event (in _inventory)

Expanded handle

Knop expanded at <path>

Expansion supersedes nothing (important!)

This makes the lifecycle clear:

Expansion ≠ duplication

Expansion ≠ move

Expansion ≠ fork

What to avoid

Move (implies loss)

Rename (misleading)

Relocate (implies old is invalid)

Alias (too weak; doesn’t convey presence)

One-line definition you can drop into the wiki

Expansion is the act of introducing a new KnopHandle for an existing Knop, thereby extending the set of IRIs by which the Knop may be accessed, without invalidating existing identifiers.

If you want, I can also:

rename related opslog events

define sflops:Expansion as a PROV activity

contrast Expansion vs Duplication in one tight table

and what about a term for, when creating an RDF payload, taking all the local terms mentioned in that dataset and giving them their own knops.

You’re naming a structural normalization operation, not a semantic one. The key properties of the act are:

it is systematic

it is non-creative (no new meaning invented)

it externalizes implicit identifiers into first-class knops

it preserves referents; it changes representation

So terms that imply invention, derivation, or interpretation are wrong.

Below are the good candidates, then a recommendation.

Strong candidates (in descending order)
Extraction

Best overall fit.

Precisely means “take something already present and make it explicit.”

Widely used in compilers, IR pipelines, and knowledge systems.

Neutral: does not imply copying, invention, or semantic change.

Examples:

“Term extraction”

“Knop extraction”

“Extract local terms into knops”

This matches exactly what you described.

Externalization

Very good, slightly more abstract.

Emphasizes “was internal, now has its own boundary.”

Common in modeling and systems architecture.

Slightly heavier, but very accurate.

Examples:

“Externalize local terms”

“Externalization pass”

Materialization

Good, but slightly risky.

Implies “making concrete.”

Sometimes implies inference (“materialize inferred facts”), which you are not doing.

Works if you’re careful in docs.

Promotion

Tempting but misleading.

Sounds like a value judgment (“this term is more important now”).

Might suggest semantic elevation.

Instantiation (❌ no)

Wrong direction: implies creating instances, not identifiers.

Minting (❌ no)

You are minting IRIs, but the operation is not minting; it’s structural extraction.

Minting is a sub-step, not the name.

Biology / graph metaphors (accurate but optional)
Differentiation

Biologically elegant.

Might be too metaphorical for a core primitive.

Speciation

Cute, but misleading (speciation implies divergence).

My recommendation
Use: Extraction

And be explicit about what is extracted:

Term extraction

Local-term extraction

Knop extraction

Identifier extraction

One-sentence definition you can drop straight into the wiki:

Extraction is the process of identifying locally-scoped terms within an RDF payload and externalizing them as first-class Knops, without changing their referent or semantics.

If you want a verb phrase for tooling:

extract-knops

extract-local-terms

extract-identifiers

Optional refinement (if you want two phases)

If you later want to distinguish:

finding terms

creating knops

You can cleanly split:

Identification (detect local terms)

Extraction (create knops + rewire references)

But you don’t need to expose that unless you want debugging hooks.

If you want, next we can:

define exactly which RDF nodes qualify for extraction

name the inverse operation (folding / inline?)

write a short “Extraction” wiki page matching your existing style

and I do think we need a name for the "Knop parts" that correspond with different folders (under expansion), and try to differentiate between knops as folder/identifier-spanning things, and particular parts of the expanded knop.

You’re right: once you commit to Expansion, you need a first-class name for the path-bound manifestations of a Knop. Otherwise everything collapses back into “folder” or “location,” which underspecifies what’s going on.

Let’s be precise and then pick.

What these things are (constraints)

A “Knop part”:

corresponds to one concrete access root (filesystem path / URL base)

is not the referent

carries operational state (pointers, inventory, opslog, working)

participates in expansion

can be current or non-current

can supersede nothing (expansion is additive)

So the name must imply:

boundedness

partiality

access/manifestation

non-identity

Strong candidates (ranked)
KnopHandle ✅ (still the best fit)

You already converged here for good reasons, and this use case actually cements it.

Why it works:

“Handle” is explicitly not the thing itself

Widely understood as “how you grab/use/access something”

Naturally plural (many handles per Knop)

Neutral across filesystem + HTTP + future transports

Reads well with Expansion:

“Expansion creates a new KnopHandle”

“This handle supersedes no other handle”

“Current handle”

It also scales cleanly:

currentHandle

handleForKnop

expandedHandle

This is the rare case where the obvious name is also the right one.

KnopFacet

Tempting, but wrong here.

Facets usually describe aspects of the same representation, not distinct access roots.

Suggests partial description, not operational embodiment.

You already use facets elsewhere; collision risk.

KnopManifestation

Conceptually excellent, UX-heavy.

Philosophically correct: same thing, different manifestations.

But long, academic, and annoying to type.

Good as an explanatory synonym, not a class name.

KnopInstance ❌

Wrong in RDF terms.

Implies instance-of a class, not access handle.

KnopAnchor

Nice metaphor, but too abstract.

Doesn’t clearly signal “this is a folder / access root.”

KnopShard ❌

Implies partitioning or splitting identity.

Opposite of expansion semantics.

Recommended terminology (tight and consistent)
Canonical terms

Knop — the referent (identity, owl:sameAs applies here)

KnopHandle — a concrete, path-bound handle created via expansion

Core relations

sflo:handleForKnop — KnopHandle → Knop

sflo:hasHandle — Knop → KnopHandle

sflops:expandedAt — Expansion event creates a new KnopHandle

Expansion phrasing

“Expand Knop K at path P → new KnopHandle H”

“This Knop has three handles”

“This handle is current”

One sentence you can drop into the wiki

A KnopHandle is a path-bound, operational manifestation of a Knop created through expansion; it provides a concrete access root for the Knop without constituting a new referent.

Why this doesn’t fight owl:sameAs

owl:sameAs applies only between Knops

KnopHandles never participate in identity equivalence

All location/time/process facts live on Handles

The model stays sharp and enforceable

If you want, next we can:

lock down a Handle lifecycle (current / historical / archived)

define what it means for a handle to be “superseded” (if ever)

write a ½-page “Expansion & Handles” wiki entry matching your style

I have reservations. A knop handle is a way of referring to a knop (or, now, a way of referring to a knop part). 

Your reservation is valid, and it’s pointing at a real semantic overload in the word handle.

A handle conventionally means “a way to refer to something,” whereas what you’re modeling is closer to “a concrete embodiment of part of the thing’s lifecycle and state.” So if KnopHandles now contain slices, pointers, inventory, opslog, working state, they are no longer just referential handles.

Let’s disentangle the roles cleanly.

The crux of the problem

You now have three distinct notions that were previously collapsed:

The referent — the Knop (identity; owl:sameAs applies)

An access embodiment — a folder / base IRI that hosts operational state

A referential alias — a way to name or point to the referent

The word handle fits (3) very well, but you are actually modeling (2).

So your discomfort is correct.

Better names for (2): the expanded, path-bound “knop part”

Here are candidates that more accurately reflect what this thing is, not how you refer to it.

KnopIncarnation ⭐️

Strong recommendation

Conveys: same thing, different concrete embodiment

Used in philosophy, distributed systems, and identity modeling

Does not imply mere reference

Reads correctly with expansion:

“Expansion creates a new KnopIncarnation”

“This incarnation is current”

“This incarnation carries operational state”

Crucially: an incarnation is of something, not a name for something.

KnopEmbodiment

Also very good.

Slightly more physical / material

Clear that this is where things live

Slightly longer, but very accurate

KnopPresence

Decent, but weaker.

Emphasizes “being present somewhere”

Less clear that it owns state

KnopInstance ❌

No — wrong in RDF/OWL semantics.

KnopManifestation

Accurate but heavy.

Fine for documentation

Painful as a core class name

Where does “handle” still belong?

You can still keep handle as a property, not a class.

Example split:

Knop — referent

KnopIncarnation — path-bound operational embodiment

sflo:hasIncarnation — Knop → KnopIncarnation

sflo:accessHandle — KnopIncarnation → URL / path / locator

This is actually cleaner:

The incarnation owns _working, _sfops, slices, etc.

The handle is just how you point at that incarnation.

This aligns perfectly with your intuition that:

“a handle is a way of referring to a knop (or part), not the part itself.”

How this plays with owl:sameAs

Nothing changes:

owl:sameAs is between Knops

Incarnations are never in equivalence relations

Expansion adds a new incarnation, not a new identity

Multiple incarnations may exist simultaneously

Minimal vocabulary shift (low churn)

If you want to minimize refactors:

Rename KnopHandle → KnopIncarnation

Introduce accessHandle (string/IRI) as a property

Keep your operational logic exactly the same

One-sentence definition (wiki-ready)

A KnopIncarnation is a concrete, path-bound embodiment of a Knop created through expansion; it hosts operational state and published artifacts without constituting a new referent.

If you want, next we can:

sanity-check this term against your existing ontology (Kosmion-style relators)

decide whether incarnations can ever be superseded or only accumulated

rewrite the “Expansion” terminology using Incarnation consistently

The referent is not the Knop. The knop is the things that defines an identifier for the referent. 

Incarnation implies death, but the old knop part continues to function, immutable. 

Maybe it will help if we untangle whether an expanded knop's various knop-handles all share the same referent. You warned earlier about, with sameAs, if you say something about any one of them, you say it about all of them. I think that's actually ok, or even good. But maybe some examples will help.

You’ve got a definitional inconsistency that you need to resolve, because it changes whether owl:sameAs is even expressing what you think.

Inconsistency

You just said:

“The referent is not the Knop. The Knop is the thing that defines an identifier for the referent.”

But earlier you also want:

“a knop gets a new location (and so an additional identity)” and owl:sameAs between those identities.

In RDF/OWL, an IRI already is an identifier for a referent, and using that IRI in triples is how you assert facts about the referent. If you treat the Knop as an “identifier object” (a designator), then two different Knops are two different identifiers, and owl:sameAs would collapse them into one identifier, which contradicts “additional identity.”

So you must pick one of these two interpretations:

Interpretation A (standard OWL): Knop IRI denotes the referent

Knop = referent (in RDF semantics)

“additional identity” means: a second IRI that denotes the same referent

owl:sameAs between those IRIs is correct

Location/process facts must live elsewhere (handles/parts), or they leak across identities

This matches your desire for sharp-edged owl:sameAs.

Interpretation B (designator model): Knop is an identifier object that denotes a referent

Knop ≠ referent; Knop denotes referent

Multiple knops can denote the same referent

owl:sameAs between knops is not what you want (it makes them the same identifier)

You want a property like sflops:denotes + (optionally) a separate node for the referent that is owl:sameAs-ed

This matches your sentence, but conflicts with your stated goals.

You can’t have “Knop is just the identifier object” and also use owl:sameAs to represent “additional identifiers” without collapsing the identifiers.

Examples to untangle “expanded knop parts share a referent”
Example 1: What you say you want (Interpretation A)

You expand from old access root to new, but both IRIs denote the same referent:

@prefix ex: <https://example/ns/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

ex:knop-old owl:sameAs ex:knop-new .
ex:knop-new ex:hasSpouse ex:Alice .  # domain fact


A reasoner will infer the spouse fact for ex:knop-old too. You say that’s good.

What must NOT happen: any location/operational fact asserted on either Knop IRI, because it would immediately apply to both.

So this is only safe if you enforce:

Knop IRIs only carry referent-facing facts

all operational facts go on a different node (handle/part)

Example 2: What goes wrong if someone asserts handle facts on the Knop IRI
ex:knop-old owl:sameAs ex:knop-new .
ex:knop-old ex:accessBase "file:///Users/dave/project/..." .


Now the reasoner gives ex:knop-new ex:accessBase ... too. That’s exactly the “leak” I warned about—and it’s not hypothetical; it’s the first mistake people will make unless the ontology makes it hard.

So your rule “only handle nodes can have accessBase / weaving / workingPath” isn’t optional; it’s required for sanity.

Example 3: If you really mean “Knop is a designator object” (Interpretation B)

Now you model identifiers explicitly:

ex:knop-old a ex:KnopIdentifier ; ex:denotes ex:Referent1 .
ex:knop-new a ex:KnopIdentifier ; ex:denotes ex:Referent1 .


Here, owl:sameAs between ex:knop-old and ex:knop-new would be wrong, because they are intentionally two distinct identifiers.

If you still want OWL identity somewhere, it belongs on ex:Referent1 (or whatever you use as the referent node), not on the identifier objects.

“Incarnation implies death”

Agreed—if the old part remains perfectly usable/immutable, “incarnation” is a bad vibe.

If you keep the “handle/part” node as the operational locus, better names than incarnation are ones that don’t imply succession or mortality:

KnopPresence (neutral: “it is present here”)

KnopFootprint (matches your “expand IRI footprint” phrasing)

KnopOutpost (metaphorical but additive)

KnopAccessPoint (plain and precise)

KnopSite (good, but “site” can be overloaded)

If you want one word that fits expansion best: Footprint is unusually aligned.

Bottom line

If you want owl:sameAs to be “exactly what we want,” you must operate in Interpretation A:

Knop IRIs denote the referent.

Expanded identifiers are equivalent via owl:sameAs.

All operational/location facts live on separate “part/presence/access-point” nodes.

If you truly insist “Knop is the thing that defines an identifier for the referent” (Interpretation B), then owl:sameAs between knops is semantically wrong for “additional identity,” and you should owl:sameAs the referent nodes instead.

Pick one; you can’t straddle them without subtle contradictions.

Sorry, when a knop expands to a new location, its referent gets an additional identifier (at the "main" IRI), and the extended-knop also gets an additional identifier (via a new _knop-handle/ IRI).

A knop's IRI always ends with /_knop-handle/. But I often slip and use "knop IRI" to refer to the main identifier. i.e., the folder/IRI where the knop is based, which denotes its referent. 

Good — that clarification resolves the apparent contradiction. What you’re really fighting is terminology drift, not model incoherence. Let me restate your model crisply, then suggest how to lock the language so you stop tripping over it.

The model you are actually using (made explicit)
1. Referent

“The thing out there.”

Never directly represented as a filesystem object.

Denoted by one or more main IRIs.

You do not model the referent as a separate node. It exists only via denotation.

2. Main Knop IRI (what you often call “the knop IRI”)

This is the identifier that denotes the referent.

It is the IRI people mean when they say “this Knop.”

It is not the _knop-handle/ IRI.

Multiple such IRIs may exist over time due to expansion.

These IRIs are related by owl:sameAs.

This is where owl:sameAs applies, and you are absolutely right that:

if you say something about one, you are saying it about all

That is exactly what you want at the referent level.

3. KnopHandle (your _knop-handle/)

A path-bound operational node.

Every KnopHandle IRI always ends in /_knop-handle/.

It is not an identifier of the referent.

It is the locus of:

_working

_sfops/*

pointers

inventory

opslog

weave state

Each main Knop IRI has exactly one local KnopHandle beneath it.

When a Knop expands:

the referent gains an additional main Knop IRI

that new main IRI has its own _knop-handle/

old main IRIs and their handles continue to exist and function

This is additive, not substitutive.

Why your earlier discomfort makes sense

You were mixing these phrases:

“Knop”

“Knop IRI”

“Knop handle”

“Knop part”

…to refer to different layers of the system.

Once you pin down that:

main Knop IRI = referent-denoting identifier

KnopHandle IRI = operational anchor under a main IRI

the confusion largely disappears.

Language you should standardize on (strong recommendation)

To avoid further slips, I suggest you lock these terms:

Use consistently

Main Knop IRI
The IRI that denotes the referent and participates in owl:sameAs.

KnopHandle IRI
Always …/_knop-handle/. Never denotes the referent.

Expansion
Adds a new main Knop IRI and a corresponding KnopHandle.

Avoid or qualify

Avoid saying just “Knop IRI” without “main” or “handle”.

Avoid calling KnopHandles “identifiers”.

If you want shorthand internally:

“knop root” → main Knop IRI

“handle” → _knop-handle/

Example (fully disambiguated)

Initial state:

https://example.org/foo/
└── _knop-handle/


Expansion:

https://example.org/foo/
└── _knop-handle/

https://mirror.example.net/foo/
└── _knop-handle/


RDF facts:

<https://example.org/foo/> owl:sameAs <https://mirror.example.net/foo/> .


Operational facts live only here:

<https://example.org/foo/_knop-handle/> sflops:hasWorkingDistribution ...
<https://mirror.example.net/foo/_knop-handle/> sflops:hasWorkingDistribution ...


No contradiction. No leakage.

Final verdict

Your use of owl:sameAs is correct given this clarified distinction.

The problem was not the ontology; it was that “knop IRI” was being used to mean two different things. Once you say:

“A Knop’s referent is denoted by one or more main Knop IRIs, each of which has an associated _knop-handle/ IRI for operational state”

the whole system snaps into focus and stays sharp-edged in the way you want.

If you want, next we can:

write a one-page “Identity & Expansion” wiki entry using this vocabulary

define the exact properties linking main Knop IRIs to their KnopHandles

add SHACL rules that forbid operational predicates on main Knop IRIs

Yes. Except, for payload knops, it is represented in the filesystem. I guess we should work out the semantics of the payload sub-IRIs. 

Maybe we call the main IRI the "Referent IRI" and the _knop-handle IRI is the "Knop IRI"

Renaming can help, but your proposed swap (“Referent IRI” for the main IRI, “Knop IRI” for /_knop-handle/) will confuse almost everyone who reads the system later, because “Knop IRI” sounds like “the thing’s identifier,” not “the operational anchor under it.”

A better way is to name by role and keep the intuitive association:

main IRI: Knop Root IRI (or Knop Base IRI) — denotes the referent

/_knop-handle/: Handle IRI (or KnopHandle IRI) — operational locus

If you really want “referent” in the name, use it only for what denotes the referent:

Referent IRI = Knop Root IRI

…but don’t call the handle “Knop IRI.”

Payload knops: semantics of payload sub-IRIs

You need to decide whether payload sub-IRIs denote:

A) parts/aspects of the referent, or
B) artifacts about the referent (datasets, distributions, pages), or
C) filesystem objects that are purely operational.

Given your architecture, payload sub-IRIs are usually artifacts (B), not “parts of the referent.” The referent might be “a person,” while the payload is “a dataset describing the person.”

Recommended rule (clean and composable)

Knop Root IRI denotes the referent.
Example: https://…/dave-richardson/

Everything under the Knop Root IRI denotes artifacts (datasets, distributions, pages) that are about the referent, unless explicitly modeled otherwise.

So:

…/_knop-handle/ denotes the operational handle (artifact)

…/_payload/ (or whatever you call it) denotes the payload dataset (artifact)

…/_payload/index.jsonld denotes a distribution of that payload dataset

…/v47/… denotes a specific slice dataset (artifact) or a distribution set under that slice, depending on how you structure it

This avoids the trap where sub-IRIs accidentally become “body parts” of the referent.

What about “payload knops are represented in filesystem”?

That’s fine. Filesystem representation does not change RDF denotation. It only means the artifact has a local concrete form.

Minimal vocabulary to make this explicit

You don’t need a separate “referent node” if you don’t want one. Just make the aboutness clear:

Payload dataset: dcterms:subject or schema:about → Knop Root IRI

Inventory/provenance: prov:wasGeneratedBy etc.

Example (conceptual):

…/_payload/ schema:about …/

…/_payload/index.jsonld dcat:distribution of …/_payload/

(You can use your own sflops: predicates if you prefer, but the point is: payload is about the root.)

Naming recommendation

Use these terms consistently:

Knop Root IRI (aka Referent IRI): the main IRI that denotes the referent

KnopHandle IRI: ends with /_knop-handle/, operational anchor

Payload Dataset IRI: e.g. …/_payload/ (dataset)

Payload Distribution: e.g. …/_payload/index.jsonld (distribution)

This keeps your mental model correct and prevents the handle from being mistaken as “the knop.”

If you paste your current payload folder naming (_payload vs _payload-flow vs whatever you settled on), I can propose the exact sub-IRI semantics in one tight section you can drop into the wiki.

I think "Referent IRI" is good. It underscore "this is the thing that refers." And "Handle IRI", which refers to the knop and I think can be sameAs the other Knop parts' Handle IRIs. 

This actually clicks into a clean, self-consistent two-layer identity model once you state it this way. Let me rest it back to you and then point out the one constraint you must lock in.

Terminology (now coherent)
Referent IRI

The IRI that denotes the referent (“the thing out there”).

This is what most domain statements are about.

Multiple Referent IRIs may exist due to expansion.

These IRIs are related via owl:sameAs.

This is what you previously (sometimes) called the “main knop IRI.”

Your phrasing is good:

“Referent IRI underscores: this is the thing that refers.”

In RDF terms, it denotes, but your wording is directionally clear.

Handle IRI

The IRI that refers to the Knop part at a particular access root.

Always path-bound (e.g. ends in /_knop-handle/).

Hosts operational state:

_working

_sfops/*

pointers

inventory

opslog

Does not denote the referent.

Crucially:

Handle IRIs may be owl:sameAs other Handle IRIs iff they are understood to refer to the same Knop part / operational embodiment.

That last clause is what makes this sharp but valid.

Identity layers (explicit)

You now have two independent equivalence planes:

Plane 1: Referent identity

owl:sameAs between Referent IRIs

Means: same real-world referent

All referent-facing facts propagate

Plane 2: Handle identity

owl:sameAs between Handle IRIs

Means: same operational Knop part, accessible at multiple locations

All operational facts propagate

These planes must never cross.

No statement should ever assert owl:sameAs between:

a Referent IRI and a Handle IRI

That’s the invariant.

Expansion semantics (now very crisp)

On expansion to a new location:

A new Referent IRI is introduced
→ owl:sameAs existing Referent IRIs

A new Handle IRI is introduced under that Referent IRI
→ MAY be owl:sameAs an existing Handle IRI (if it truly is the same Knop part replicated)
→ or MAY be distinct (if it’s a fresh operational embodiment)

This distinction lets you model both:

mirrored access to the same handle state

independent handles that just happen to serve the same referent

That’s powerful and correct.

The one constraint you must document (non-negotiable)

You must state in writing (docs + SHACL):

Referent IRIs must never carry operational predicates.
Handle IRIs must never carry referent-facing predicates.

This is what keeps owl:sameAs from becoming radioactive.

Examples:

❌ ReferentIRI sflops:hasWorkingDistribution …

❌ HandleIRI foaf:name "Alice"

Those are category errors.

Why this is actually elegant (not overcomplicated)

You did not invent a third “referent node.”

You used OWL identity exactly where it is strongest.

You allowed two independent equivalence closures.

You avoided weakening semantics with custom “almost same” properties.

You made filesystem/URL mirroring first-class without polluting domain identity.

Most systems collapse under this pressure. Yours doesn’t, because the split is principled.

Suggested one-sentence definitions (wiki-ready)

Referent IRI

An IRI that denotes a referent and may be equivalent to other Referent IRIs via owl:sameAs.

Handle IRI

An IRI that denotes a concrete Knop handle at a specific access root, hosting operational state and optionally equivalent to other Handle IRIs.

If you want, next we can:

write the SHACL shapes that enforce the predicate separation

formalize when Handle IRIs may be owl:sameAs

define payload sub-IRI semantics precisely under this model

Maybe pointers, inventory, opslog are located under _knop-handle/, underscoring that they are concerned with the knop, not its referent. config flows could go under there too, although splitting the flows up (leaving _ref and _payload directly under the referent/main folder) means things that are flows should probably get "-flow" in the folder name.

Backlinks, if any, can just go into _ref.

Yes — putting pointers/inventory/opslog under /_knop-handle/ is the right alignment with your “Referent IRI vs Handle IRI” split. It makes the semantics self-documenting:

Referent folder: things about the referent (payload, reference view)

Handle folder: things about operating the knop (current routing, ledgers, logs, working inputs)

Proposed structure (tight)

Under the Referent IRI (root):

_payload/ (dataset(s) about the referent; “payload” content)

_ref/ (reference-flow / reference slices; also a reasonable home for backlinks if they are “about the referent”)

Under the Handle IRI (.../_knop-handle/):

_sfops/

pointers/ (authoritative current pointers for this handle)

inventory/ (ledger + provenance for slice publications / supersessions)

opslog/ (rotatable debugging log; not authoritative)

_working/ (per-flow working distributions, if you keep _working under the handle)

About “config flows under the handle”

Correct if the configs are operational (how this handle weaves/builds). If configs are referent-facing (rare), they belong under the referent root, but most config is handle-scoped.

Your “-flow” suffix idea is good, but apply it consistently and sparingly:

Use -flow only for directories that are actual flows (have _working, are snapshotted/weavable).

Do not call _sfops/* “flows.” They’re datasets produced/managed by tooling.

Example: config-flow/ is clearer than _cfg-local/ if you want to eliminate underscore magic, but if underscores are your “system-owned” marker, _config-flow/ is also defensible.

Backlinks in _ref

Agree if “backlinks” are computed about the referent (incoming edges, references). If you ever need “operational backlinks” (e.g., which handles mention this handle), that’s a different dataset and should live under /_knop-handle/_sfops/.

One constraint to keep you out of trouble

If pointers live under the handle, your bootstrap needs to be unambiguous:

Loader finds Handle IRI from Referent IRI deterministically (you already have /_knop-handle/ for that).

Then it finds pointers deterministically under the handle (e.g., /_sfops/pointers/index.jsonld).

That removes the temptation to put pointer-ish facts on the Referent IRI.

If you want, I can write a short wiki section “Directory Semantics” that states the rule: everything operational is under /_knop-handle/; everything referent-facing is under the Referent root, and defines what counts as a flow vs a non-flow dataset.

I was thinking _working goes in the corresponding flow, next to the slices. But I guess it doesn't have to. 

We can get rid of sfops/ if we put its stuff under _knop-handle. 

Regarding sameAs for handles... I don't think they have to have identical physical contents to be sameAs. The Knop is an extended thing, with "parts" potentially located at different paths. But it's the same conceptual thing. Is there anything we can't say about a knop part because they're sameAs'd? 
_working location

Putting _working “inside the flow next to slices” is fine, but it weakens the one clean semantic split you just established:

Referent folder: referent-facing artifacts

Handle folder: operational state

_working is operational input, so the best semantic home is under /_knop-handle/, even if the flow is conceptually “next to slices.”

If you insist on _working living with the flow directories, you can still keep semantics clean by making those flow directories themselves handle-scoped (i.e., under /_knop-handle/), not under the referent root.

So either:

.../_knop-handle/<flow-name>/_working/... (recommended), or

.../<flow-name>/_working/... only if <flow-name>/ is declared handle-owned (easy to confuse later).

Dropping _sfops/

You can drop _sfops/ and put:

.../_knop-handle/pointers/

.../_knop-handle/inventory/

.../_knop-handle/opslog/

That is structurally fine.

The reason to keep _sfops/ is not necessity; it’s namespace hygiene and future-proofing:

it gives you one “system subtree” to ignore/whitelist

it avoids collisions with future flow names like inventory-flow/ or opslog-flow/

it makes “these are not flows” visually obvious

If you drop it, you should reserve those names (pointers, inventory, opslog) as non-flow dataset names and forbid flows from using them.

owl:sameAs for handles without identical physical contents

This is where you’re about to step on the rake I warned about earlier.

If two Handle IRIs are owl:sameAs, then by OWL semantics they denote the same individual, so:

any statement about handle A is a statement about handle B

they are interchangeable for all properties

So if the two handles have different physical contents, you must not model those physical differences as properties of the Handle individual, or you create contradictions.

You can still say they are sameAs and have different physical realizations only if you represent “physical contents” on a separate node, e.g.:

Handle = conceptual/operational handle individual

HandleLocation / HandleRealization = a particular filesystem/URL manifestation with actual bytes

Then:

both manifestations link to the same Handle

the Handle itself can be sameAs across IRIs

differences live on the manifestation nodes, not the Handle

Otherwise, the moment you assert something like:

“this handle’s pointers distribution is at path X”

“this handle’s working file is at path Y”
you’ve asserted it about both.

Direct answer: “Is there anything we can’t say about a knop part because they’re sameAs’d?”

Yes: you cannot say anything that should be true for one location but not the other.

Concrete examples of things you would be forbidding yourself from expressing on the Handle if you sameAs-collapse handles across locations:

that one location has a local _working file at a particular path

that one location is currently weaving / locked

that one location’s pointers file forwards elsewhere

that one location has only a partial subset of slices present

that one location is stale/offline/archived

If you want any of those, they must be asserted on a location/manifestation node, not on the Handle.

What you can safely say about a sameAs-collapsed Handle

Things that are intended to be globally true across all access roots:

“this handle is for referent R”

“this handle’s current pointers dataset is P” (as an abstract dataset identity, not a file path)

“this handle published slice S at time T” (if that’s global)

provenance about the handle conceptually (tooling, authority chain), if it’s truly shared

Recommendation (given your stated intent)

Use owl:sameAs freely for Referent IRIs (your “main IDs”).

Be much stricter for Handle IRIs:

only sameAs them if you are willing to treat them as the same operational handle individual with no location-scoped facts

otherwise, keep handle IRIs distinct and relate them with an “additional locations / forwarding” mechanism (not sameAs), or introduce a separate “HandleRealization” concept.

Right now your sentence:

“parts potentially located at different paths… same conceptual thing”

describes exactly the situation where you want one conceptual handle plus many realizations. Without that third node, owl:sameAs on handles will erase distinctions you will almost certainly need.

If you tell me whether you ever need to say “this location is currently weaving” or “this location has _working changes,” I can tell you definitively whether sameAs on handles is viable without adding a realization layer.

you're right about sameAs for handles. 

I think we use underscores everywhere for consistency.

alice/
  _ref-flow/
  _payload-flow/
  _knop-handle/
    _local-cfg-flow/
    _inh-cfg-flow/
    _pointers/
      index.jsonld
    _inventory/
      index.ttl

This structure is coherent and internally consistent with everything you’ve locked in. A few precise observations and one small recommendation.

The structure you proposed (validated)
alice/
  _ref-flow/
  _payload-flow/
  _knop-handle/
    _local-cfg-flow/
    _inh-cfg-flow/
    _pointers/
      index.jsonld
    _inventory/
      index.ttl

What this gets right

Referent vs handle split is unambiguous

alice/ = Referent IRI

alice/_knop-handle/ = Handle IRI

Nothing operational leaks onto the referent.

Flows are clearly marked as flows

*_flow/ convention cleanly distinguishes weavable things.

_ref-flow and _payload-flow being sibling flows under the referent makes semantic sense.

Operational-only artifacts live under the handle

_pointers and _inventory are unambiguously handle-scoped.

No need for sfops/ once this convention is adopted.

Underscores everywhere are now doing one job

“This is system- or machine-oriented.”

You’re no longer trying to overload _ to mean multiple different things.

Inventory as ledger

_inventory/index.ttl is the right place for provenance, supersession, delegation chains, etc.

Non-flow, append-oriented, authoritative.

The one thing I’d tighten
Consider whether _working belongs inside each *-flow/

If flows are first-class weavable units, the cleanest rule is:

Every flow directory MAY contain:

_working/

published slices (v47/, timestamps, GUIDs, etc.)

That suggests:

_knop-handle/
  _local-cfg-flow/
    _working/
    v3/
  _inh-cfg-flow/
    _working/
    v1/


rather than putting _working elsewhere.

This preserves:

“_working is per-flow”

“flows are the unit of snapshotting”

“handles own all operational state”

If you instead centralize _working at the handle level, you’ll end up re-inventing per-flow routing logic anyway.

On sameAs (now resolved)

Referent IRIs: owl:sameAs freely and correctly

Handle IRIs: no sameAs unless you intentionally erase location distinctions

Physical realization differences: stay out of the handle entirely (filesystem is not RDF)

You made the right call backing off sameAs for handles.

Naming sanity check

_ref-flow — acceptable, though slightly odd linguistically; consistent with your earlier _reference-flow choice

_payload-flow — good

_local-cfg-flow / _inh-cfg-flow — clear, explicit, boring (good)

_pointers / _inventory — correctly not flows

I would reserve these names permanently so no one ever tries to create _inventory-flow/ later.

Bottom line

This layout:

respects OWL semantics

keeps identity sharp

keeps provenance attachable

keeps humans from accidentally asserting nonsense

scales to expansion cleanly

You’re not overengineering here — you’re finally aligning terminology, filesystem structure, and RDF semantics so they stop fighting each other.

hasLocalConfigFlow's domain is obviously KnopHandle

hasReferenceFlow domain is KnopHandle? Or Knop/

BTW, I'm reviewing Knop/'s usage in the ontology, but I think we change it to ReferentIRI?
1) hasLocalConfigFlow domain

Yes: KnopHandle.

A “local config flow” is inherently handle-scoped (it controls how this handle behaves). If you ever allow multiple handles for the same Referent IRI, local config cannot safely live at the referent level without becoming ambiguous.

So:

sflops:hasLocalConfigFlow domain sflops:KnopHandle

(range: sflops:Flow)

Same for hasInheritableConfigFlow if that inheritance is “handle inheritance” (which it is in your filesystem layout).

2) hasReferenceFlow domain: KnopHandle or Knop?

Given the structure you just committed to:

alice/                 (Referent IRI)
  _ref-flow/
  _payload-flow/
  _knop-handle/
    ...


_ref-flow and _payload-flow are under the Referent IRI, not under the handle.

So the clean modeling is:

hasReferenceFlow domain = the thing identified by the Referent IRI (what you previously called Knop/)

hasPayloadFlow domain = same (Referent IRI node)

And then the handle can optionally point to them as a convenience (but that should be a different property, or clearly documented as redundant/derived).

If you instead make hasReferenceFlow domain KnopHandle, you’re implying reference/payload are operational handle-owned flows, which contradicts your layout and will cause “why is the reference flow about the referent but attached to the handle?” confusion.

So:

sflops:hasReferenceFlow domain = sflops:Knop (or whatever you rename it to)

sflops:hasLocalConfigFlow domain = sflops:KnopHandle

3) Renaming Knop/ to “ReferentIRI?”

You should not name a class “ReferentIRI.” An IRI is a lexical identifier; the class is the kind of thing the IRI denotes.

Better options:

Keep the class IRI as sflops:Knop (for backward compatibility), but change:

rdfs:label to “Referent”

rdfs:comment to explicitly define it as “the node denoted by the Referent IRI”

Or introduce a new class with a clearer name and relate it:

sflops:Referent (or sflops:KnopReferent)

declare sflops:Knop owl:equivalentClass sflops:Referent (or deprecate Knop)

Critical point: “Referent IRI” is a role/term in documentation, not a good OWL class name. The class should be “Referent” (or “ReferentResource”), and your docs can say: “The Referent IRI denotes a sflops:Referent.”

Quick decision table
Property	Domain should be	Why
hasLocalConfigFlow	KnopHandle	handle-specific behavior
hasInheritableConfigFlow	KnopHandle	still handle-scoped in your layout
hasReferenceFlow	Referent (formerly Knop)	flow lives under referent; about referent
hasPayloadFlow	Referent	same

If you want, I can propose the minimal ontology diff (new class name + deprecation strategy) without breaking existing data.

I think sflo:Knop is the thing with the _knop-handle IRI. So I'm going to rename KnopHandle to Knop. And the an sflo:Referent is the thing with the base IRI

Maybe this alleviates the tension of "The Knop's IRI refers to the thing in the world".

I do have a HandleAsComponent class, intended to be identified with a fragment identifier, e.g.".../_knop-handle/#". But the only place I use it is as the domain for isHandleFor.

I suspect we still need the KnopExtension construction. Maybe KnopNode, as Node has the sense of "part of a larger whole".  Then we can make assertions like "hasWorkingFolder" about the KnopNode, and Knops can be sameAs. 

This is directionally good for UX (it matches how people will think: “the Knop is the concrete folder-thing with _knop-handle”), but you’re about to reintroduce the original semantic hazard unless you keep the identity layers explicit.

Here’s the hyper-critical read:

1) Renaming KnopHandle → Knop is fine only if you also rename the old “Knop” concept

If you do:

sflo:Knop = the thing at …/_knop-handle/ (operational anchor)

sflo:Referent = the thing at the base IRI (…/)

then you’ve moved the semantic load where it belongs operationally, and it avoids the misleading phrase “the Knop’s IRI refers to the thing in the world.”

That’s good.

But you must then be disciplined about what is owl:sameAs with what.

New equivalence expectations

owl:sameAs applies between Referents (base IRIs), not between Knops (/_knop-handle/), if you want “these denote the same thing in the world.”

If you owl:sameAs Knops (handles) you are asserting two operational anchors are identical in OWL semantics, which you already accepted you do not want (because of location-scoped facts like working folders, pointers, etc.).

So: do not “Knops can be sameAs” if Knop = operational anchor.

That sentence is the main contradiction in your message.

2) “We still need KnopExtension / KnopNode” — yes, but be precise

You’re trying to get three things at once:

A referent identity plane (owl:sameAs works here)

A concrete operational anchor plane (the Knop at /_knop-handle/)

A way to represent “this referent has expanded to additional access roots” without collapsing operational facts

You do not need KnopNode for (2). You already have Knop.

You might need a third concept for (3), but naming it “KnopNode” is risky because “node” sounds like the referent again.

Better: call it what it is:

ReferentAccessRoot / ReferentFootprintMember / ExpansionRoot

i.e., “a base IRI that participates in the referent’s identity set”

This is the thing that was previously the “main IRI” that you owl:sameAs across expansions.

If you make that a class, then:

it’s basically your sflo:Referent instances themselves, since the base IRI denotes the referent

you don’t need a separate node unless you want to attach location-ish facts to “the base IRI as a site,” which you said you don’t want

So “KnopExtension” is only needed if you want to treat “the set of base IRIs used for this referent” as a first-class object (useful sometimes), but not required.

3) “hasWorkingFolder about KnopNode, and Knops can be sameAs” is a regression

You already accepted:

owl:sameAs causes facts to propagate

working folder facts are location-specific

So you must not attach hasWorkingFolder to anything you intend to owl:sameAs across different physical locations.

If you rename KnopHandle→Knop, then:

hasWorkingFolder domain: sflo:Knop

and sflo:Knop must not be owl:sameAs across locations

Instead:

Referents are owl:sameAs

Knops are not owl:sameAs; they are linked by “isKnopFor” / “isOperationalKnopFor” to the Referent

That’s the clean two-level model.

4) HandleAsComponent with fragment # is unnecessary complexity unless you have a concrete need

Using …/_knop-handle/# to denote “the handle as component” is going to confuse consumers, and fragment semantics vary by media type and retrieval context. You gain very little.

If the only use is the domain of isHandleFor, delete it and set domain to sflo:Knop (after your rename).

You can always reintroduce it later if you have a real component model (e.g., a handle has components like pointers/inventory/opslog as nodes you want to talk about independently).

Recommended revised model (consistent with your renames)
Classes

sflo:Referent — denoted by base IRI (alice/)

sflo:Knop — denoted by handle IRI (alice/_knop-handle/)

Key relations

sflo:isKnopFor (domain sflo:Knop, range sflo:Referent)

sflo:hasKnop (inverse)

Identity

owl:sameAs only between sflo:Referent IRIs (base IRIs)

No owl:sameAs between sflo:Knop IRIs

Operational datasets live under the Knop

sflo:hasPointersDataset domain sflo:Knop

sflo:hasInventoryDataset domain sflo:Knop

config flows domain sflo:Knop

reference/payload flows domain sflo:Referent

This alleviates exactly the tension you mentioned without reintroducing handle sameAs problems.

If you still want a “KnopExtension” concept

Only do it if you want to talk about “the expansion event / footprint” as a first-class object. In that case:

sflo:Expansion (prov:Activity)

connects oldReferentIri → newReferentIri

records why, who, when

does not require owl:sameAs on Knops

But don’t make “KnopNode” the dumping ground for “things I want to assert without thinking about identity.” That’s how you end up with semantic soup.

Bottom line

Renaming KnopHandle → Knop and main node → Referent is fine and probably improves UX.

Do not make Knops owl:sameAs across locations.

Drop HandleAsComponent unless you can point to a concrete consumer requirement.

Put hasWorkingFolder etc. on sflo:Knop.

Put owl:sameAs on sflo:Referent.

If you want, paste your current ontology snippets for Knop, KnopHandle, isHandleFor, and KnopExtension, and I’ll give you a minimal diff that preserves backward compatibility while enforcing the corrected identity rules.