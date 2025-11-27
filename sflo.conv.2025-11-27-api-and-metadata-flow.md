---
id: ullhcee1jol7b0e0s5641jr
title: 2025 11 27 API and Metadata Flow
desc: ''
updated: 1764266363999
created: 1764266119686
---
Skip to content
Chat history
You said:
I've been envisaging the CLI as a fairly-thing REST client for the sflo-api. But it'd be nice to have an offline mode for portable weaving. Shouldn't be too hard.
ChatGPT said:

Do it, but design it as a first-class execution target, not a “fallback.”
High-leverage design

    Single engine, two transports. Keep one weave executor. Swap transport adapters:

        remote: sflo-api REST.

        local: file:// + Git working tree.

    Deterministic builds. Offline must produce byte-identical artifacts given the same inputs.

        Pin library versions and template hashes.

        Sort graph materialization, stable blank-node elimination (you already forbid bnodes), stable JSON-LD compaction contexts.

    Snapshot identity. Replace monotonic versions with a timestamp ID you mentioned, e.g., YYYYMMDDhhmmssSSS in UTC plus a content hash:

        snapshotTimestamp and snapshotDigest (SHA-256 of build manifest) → snapshotId = {timestamp}-{digest8}.

    Portable bundle. Introduce a .weavepack (tar+zstd):

        /manifest.json (inputs, graph counts, digests, toolchain versions).

        /inputs/ (normalized RDF sources or their digests with fetch map).

        /outputs/ (built datasets, catalogs, docs).

        /provenance/ (PAV, sflo: refs; build log).

    Queue + reconcile. Record operations when offline:

        ops.jsonl with idempotent ops (create node, bump _main, release, attach dist, tag current).

        On sync, replay against sflo-api with 3-way merge rules.

File layout (local mode)

.sflo/
  cache/               # CAS: sha256/<xx>/<digest>
  locks/               # advisory .lock files
  toolchain.json       # versions of CLI, Node, templates, contexts
  weave.json           # project config (namespace roots, templates, rules)
  ops.jsonl            # pending ops
builds/
  <snapshotId>/
    manifest.json
    outputs/
    logs/

Minimal manifest

{
  "schema": "sflo.weave.manifest@1",
  "snapshotId": "20251109T181158123-9f3a1c7a",
  "snapshotTimestamp": "2025-11-09T18:11:58.123Z",
  "snapshotDigest": "sha256:9f3a1c7a…",
  "engine": { "cli": "weave 0.4.0", "node": "22.10.0" },
  "contexts": { "jsonld": ["…/hydra/context.jsonld", "…/sflo/api/v1/context.jsonld"], "hash": "sha256:…" },
  "inputs": [{ "iri": "…/djradon/bio/", "source": "file", "digest": "sha256:…" }],
  "templates": [{ "name": "default-page", "digest": "sha256:…" }],
  "outputs": [{ "iri": "…/djradon/bio/index.trig", "bytes": 12345, "digest": "sha256:…" }],
  "stats": { "quads": 128734, "graphs": 17 }
}

Commands

    weave init → writes .sflo/weave.json, bootstraps contexts, templates.

    weave build --offline → produces builds/<snapshotId>/ and .weavepack.

    weave diff [A] [B] → RDF-aware diff of datasets, catalogs, vsets.

    weave check → validates shapes, context hashes, deterministic ordering.

    weave op <verb> <args> --offline → append to ops.jsonl only.

    weave sync → authenticate, replay ops to sflo-api, upload bundles, reconcile.

    weave release <node> --series _main → stamps catalog in local mode; on sync, promotes via API.

Reconciliation rules

    Catalog pointers (current, next): server is source of truth. Local next only becomes current on successful release.

    Conflicts on datasets: 3-way with policy:

        If server changed since base and local changed → mark divergent and require weave merge.

        Else apply op and attach provenance linking both snapshot digests.

    Idempotency: each op carries opId = hash(operation JSON). Server ignores duplicates.

Locks and concurrency

    Node cannot lock a subtree portably. Use advisory lock files:

        .sflo/locks/build.lock for executor.

        Acquire with O_EXCL; include PID and hostname; stale detection with mtime + liveness probe.

    Avoid global FS locks; serialize writes inside the CLI.

Determinism checklist

    Canonical N-Quads before hashing.

    Fixed JSON key order and spacing.

    Normalize line endings to \n.

    Always UTC. Never locale formats.

    Embed versions of JSON-LD contexts and templates. Fail build if hashes drift.

Security

    No secrets in .weavepack.

    Redact paths and usernames unless --include-host-meta.

    Sign manifests optionally: weave sign → provenance/signature.asc.

API parity

    Define a thin capability map fetched from server on sync:

        Supported verbs, schema versions, max upload size.

        CLI performs version negotiation and can down-convert manifests.

Open risks

    Context drift: remote context URLs may change content. Pin by hash and mirror locally.

    Git vs CAS: choose CAS as primary. Git is for human workflows. Store CAS objects under .sflo/cache.

    Bundle bloat: consider dedup via CAS and reference inputs by digest when re-packing.

Implementation order

    Core CAS + canonicalization + manifest writer.

    Offline build path producing deterministic outputs.

    Ops log + replay client with backoff and conflict codes.

    .weavepack import/export.

    RDF diff and merge helpers.

This keeps your CLI “fairly thin” while making offline weaving robust. If you want, I’ll sketch the manifest@1 JSON Schema and the op verbs next.
You said:
No need to q and reconcile because everything gets written the metadata flow regardless , it's not like the API would do something different 
ChatGPT said:

Right, if that’s true then most of what I said about op queues is over-engineering.

Let me make the assumptions explicit and then tighten the design around them.
Your implied model

You’re implicitly saying:

    Single source of truth:
    The canonical state of the mesh is the checked-in RDF (esp. the metadata flow).
    Everything else (API views, indexes, derived quads) is derived.

    API is not a separate state machine:
    Mutating API calls (if any) either:

        Don’t exist, or

        Ultimately write back to the same metadata flow artifacts that the CLI would modify.

    No extra semantics server-side:
    The API doesn’t “invent” anything that would make an API-performed operation meaningfully different from “run the flow locally and write the same RDF.”

If all three hold, then:

    Offline CLI writes mesh → you later push/publish → API sees that mesh.

    Online CLI calling sflo-api would ultimately result in the same mesh changes, just executed remotely.

Under that regime, “op queue + reconcile” really is redundant. Git (or your storage) is the reconciliation mechanism.
What you still need (minimal)

Even without queues:

    One implementation of semantics

        Implement weave/flow execution in a shared library (call it @sflo/engine).

        Both:

            sflo-api and

            sflo-cli --offline
            call the same library on a given “mesh snapshot.”

        That makes “API wouldn’t do something different” true in practice, not just by intention.

    Collision detection, not reconciliation
    You don’t need op logs, but you do need a guard against blind overwrites:

        Put a meshDigest / flowDigest in the metadata flow header (SHA of canonical form).

        CLI offline:

            Reads flow with digest D_old.

            Runs execution, writes new flow with D_new and a “previousDigest = D_old”.

        Server (or CI) that publishes from Git (or object store) can:

            Detect if two branches changed from the same D_old but diverged (normal Git conflict).

            Or you just rely on plain Git merges and don’t care further.

    You don’t need an application-level op log; Git already provides the history and conflict surface.

    Mode separation at the CLI level

    Something like:

        sflo weave --offline

            Directly manipulates the local mesh (metadata flows + payload flows).

        sflo weave --remote

            Serializes the same intent (which flow, which node, which config) into an HTTP request.

            Server runs @sflo/engine on its copy of the mesh and writes back its metadata flow.

            If the server writes to the same Git repo, ordinary Git semantics handle conflicts.

    There is no extra “queue + replay” layer in the CLI; it either runs locally or delegates.

What you can drop from my previous proposal

Given your model:

    Drop: ops.jsonl, op IDs, weave sync replays.

        There is no offline “pending operations” concept; you’re just editing files in the mesh like anything else.

    Optional: .weavepack bundles.

        Might still be useful for:

            Porting a mesh snapshot to another machine.

            Shipping a self-contained analysis artifact.

        But it’s not needed for coherence between API and CLI.

    Keep in spirit: deterministic builds & manifests.

        Even if you don’t “sync,” it’s still valuable to be able to say:

            “This set of outputs came from this exact input mesh + toolchain + contexts.”

        That’s about reproducibility, not reconciliation.

The key architectural refinement

If you really want “no reconciliation layer,” then you need to enforce:

    No API-only mutations that bypass the mesh:

        Any write that changes semantics must result in RDF/flows that live alongside your metadata flow.

        If you later add a web UI that lets people click “release”, that UI must still go through:

            either a Git commit to the mesh, or

            a write to the same backing store that the CLI operates on.

    No hidden server state that matters for semantics:

        Caches, indexes, search, et cetera can diverge; they’re derived.

        But all authoritative information must be in the mesh.

If you ever break either of those, you’ve silently reintroduced the need for reconciliation logic, because “what the API does” would become strictly more than “what the metadata flow says.”
Concrete simplification of the earlier sketch

You can shrink it to:

    .sflo/ still exists (toolchain, contexts, maybe CAS), but no ops log.

    sflo-cli has:

        weave build → default is offline, manipulates metadata flow and payload flows directly.

        weave build --via-api (or --remote) → sends the same high-level request to sflo-api.

Internally:

    @sflo/engine takes:

        A reference to the “mesh handle” (local FS, remote store, whatever).

        A flow/node to execute.

    Both CLI and API instantiate it.

Then your statement “it’s not like the API would do something different” becomes almost tautological: they literally share the same engine and the same RDF artifacts.

If you agree with that, the next thing to pin down is: how much metadata about builds you still want in the flows (snapshot IDs, digests, toolchain info), because that’s not about reconciliation, it’s about provenance.
You said:
What you call @sflo/engine, I was planning to call "core", and in the monorepo it lives in shared/core/

Can we just make jsonld canonical by:

- using no blank nodes
- sorting every level of the tree alphabetically

'
ChatGPT said:

Short answer:
No, “no blank nodes + sort every level of the tree alphabetically” is not, by itself, a robust canonicalization scheme. It can work if you tighten the rules a bit, but you’re missing a few critical details.

I’ll separate two things:

    What you want for core (shared/core) to compute stable digests.

    What “sort every level of the tree alphabetically” would actually need to mean.

1. For digests, JSON-LD is the wrong level to canonicalize at

Given your constraints:

    You already intend to forbid blank nodes.

    You care about graph equivalence, not JSON prettiness.

    You only ever generate JSON-LD through your own code paths (core, sflo-api, CLI).

The cleanest, least-surprising move is:

    Take your JSON-LD.

    Convert to an RDF dataset once (in core) with fixed options.

    Emit canonical N-Quads and hash that.

Because you have no blank nodes, canonical N-Quads is trivial:

    Sort triples by (graph, subject, predicate, object) lexicographically.

    Serialize with a fixed rendition of literals:

        Escapes, datatype IRIs, language tags, etc., all normalized.

    No need for URDNA2015 or complicated bnode canonicalization.

Then:

    Your digest is over the sorted N-Quads, not over an incidental JSON tree.

    Any JSON-LD shape differences that still map to the same RDF graph become invisible to the digest (which is what you want if you care about RDF-level equality).

JSON-LD stays as “one of the surface formats,” not the canonical substrate.

If you’re serious about “the metadata flow is truth,” this RDF-level canonicalization in shared/core is the right locus.
2. If you still want JSON-LD canonicalization, your current rule is underspecified

If you insist on canonicalizing JSON-LD-as-JSON directly, then “no bnodes + sort every level alphabetically” is not enough unless you define more:
2.1. Objects vs arrays

Your phrase “every level of the tree” is ambiguous:

    Sorting object keys alphabetically is straightforward.

    Sorting array elements is not, unless you define:

        How to compare scalar values.

        How to compare objects.

        How to handle heterogeneous arrays (scalars + objects).

If you don’t canonicalize arrays, you’ve done nothing about:

{ "roles": ["admin", "user"] }
{ "roles": ["user", "admin"] }

which are semantically equal in RDF but would hash differently.

If you do want to canonicalize arrays, you need something like:

    For arrays of scalars: sort by canonical string value.

    For arrays of objects:

        If each object has @id, sort by @id.

        Otherwise, sort by canonical stringification of the object (which itself assumes you’ve already canonicalized the object’s keys and nested arrays).

2.2. Numbers and booleans

You also need to normalize scalar formatting:

    1, 1.0, 1e0 are the same numeric value but will serialize differently if you’re sloppy.

    You need a rule such as:

        Always emit numbers without a plus sign, no leading zeros (except zero itself), minimal exponent form, etc.

    Booleans are easier (true/false only), but you must forbid "true" vs true ambiguity in your own data.

If you’re hashing JSON.stringify() output, different numeric representations will break equality even though your RDF graph is unchanged.
2.3. Unicode normalization

If IRIs or string literals can contain non-ASCII characters, you must decide:

    Which Unicode normalization form you use (usually NFC).

    You then require all strings to be normalized before canonicalization.

Otherwise, visually identical strings can hash differently.
2.4. Context shape

JSON-LD lets you represent the same RDF with many different @context layouts. If you don’t constrain this, you get:

    Structurally different but graph-equivalent JSON-LD documents that your “sort the tree” step will treat as different.

So if you want JSON-level canonicalization, you must:

    Fix a single canonical context for each flow/dataset.

    Or canonicalize context objects too (same recursion: sort keys, arrays, normalize IRIs).

At that point you’re reinventing a local version of JCS + JSON-LD profiling anyway.
3. A minimal, realistic scheme for core

Given everything above, a sane compromise for shared/core:
Option A (recommended): Canonicalize RDF, not JSON

    From JSON-LD to RDF:

        Use a fixed documentLoader, fixed options (e.g., ordered: true) so expansion is deterministic.

        Ensure no blank nodes at this point; fail validation if any appear.

    From RDF dataset to canonical N-Quads:

        Emit N-Quads with:

            Fixed escaping rules.

            No pretty-printing, one quad per line.

        Sort lexicographically by (graph IRI, subject, predicate, object).

    Hash the canonical N-Quads to get your digest.

Everything else — JSON-LD, Trig, whatever — is just another view of that canonical RDF. This also lines up with your “payload node” vs dataset distinction: the digest lives at the dataset level.
Option B (if you really want JSON-LD-level canonicalization)

Then define an explicit “sflo JSON canonicalization” (and implement it once in core):

    Normalize RDF → JSON-LD:

        Convert to RDF dataset and back to JSON-LD using:

            Fixed canonical @context per flow type.

            Fixed options (ordered: true, no framing variations, etc.).

    Canonicalize JSON:

    Recursively:

        If value is an object:

            Canonicalize all children.

            Sort keys alphabetically.

        If value is an array:

            Canonicalize all elements.

            If all elements have @id: sort by @id string.

            Else: sort by JSON.stringify(element) of the canonical form.

        Normalize scalars:

            Numbers: single canonical textual form.

            Strings: Unicode-normalize.

            Booleans/null: as-is.

    Hash JSON.stringify(canonicalJson).

But notice: step 1 is already working at the RDF level. At that point, you might as well stop at canonical N-Quads and not drag JSON into it.
4. Concrete takeaway

So to answer your question directly:

    No, “no blank nodes + sorting every level of the tree alphabetically” is not sufficient as stated.

    You must at minimum also define:

        Array ordering rules.

        Scalar normalization (numbers, strings, Unicode).

        Context normalization (or strict profile).

Given that you already have a strong RDF mental model and you’ve banned blank nodes, the simplest, least fragile spec for core is:

    Canonical form = sorted N-Quads of the RDF dataset; digest = hash of that.

JSON-LD then becomes a deterministic projection of that, not the canonical thing itself.
You said:
Tell me more about collisison detection, "You don’t need op logs, but you do need a guard against blind overwrites"
ChatGPT said:

Collision detection is just you being explicit about “what version of the mesh did I think I was editing?” and refusing to overwrite if reality changed underneath you.

Right now, without it, both CLI and API are susceptible to:

    A: Read mesh

    B: Modify in memory

    C: Write new mesh, regardless of what happened between A and C

If some other process changed the mesh between A and C, you silently stomp their work → “blind overwrite”.

You said “everything gets written to the metadata flow regardless.” That’s good, but it doesn’t solve this. It just means everyone is stomping the same truth in the same place.

So you need a cheap, explicit guard.
1. The core idea: optimistic concurrency on the mesh

You can model this exactly like HTTP If-Match/ETags or DB row-versioning:

    Each “authoritative thing” (mesh, node, flow, dataset – you choose granularity) has a version token.

        In your world, that token should be a digest over canonical RDF (not JSON), e.g. meshDigest, flowDigest, or datasetDigest.

    Any writer must present the version token it started from.

    The write is only allowed if the current token still equals that value.

If it doesn’t match, you abort the write and tell the caller:

    The mesh changed since you last read it; you need to re-run or merge.

No op log required; you just refuse to blindly overwrite.
2. Where to attach the digest (granularity choice)

You have to choose what carries the version token.

Reasonable options in your setup:

    Whole mesh digest

        One meshDigest value for the entire repo/mesh.

        Pros: trivial to compute and store; any change anywhere is detected.

        Cons: any unrelated change triggers a “collision,” which might be too coarse for frequent edits.

    Per-flow digest

        Each metadata flow (or payload flow) has its own digest, e.g. sflo:flowDigest.

        Pros: more precise. Two people editing unrelated flows don’t collide.

        Cons: you have to implement digesting per flow and enforce when writing those flows.

    Per-dataset or per-node digest

        Each dataset series or mesh node has its own digest.

        Pros: maximum precision for your semantics.

        Cons: more bookkeeping; you need to be consistent about what constitutes a “unit of concurrency”.

Given your “metadata flow as truth” stance, per-flow or per-dataset digests are the sweet spot. “Whole mesh” will get annoying fast in a multi-user or CI-heavy environment.

For the rest of this answer I’ll talk in terms of flowDigest, but you can mentally replace with meshDigest or datasetDigest.
3. Concrete shape in the metadata flow

Inside a flow’s RDF (JSON-LD view):

    A current digest for the flow’s content.

    A previous digest (the one the writer based on).

Something like:

{
  "@id": "…/ns/djradon/_metadata-flow",
  "sflo:flowDigest": "sha256:abcdef1234…",
  "sflo:previousFlowDigest": "sha256:7890abcd…"
  // ...
}

Semantics:

    sflo:flowDigest = digest of the current canonical RDF content of this flow.

    sflo:previousFlowDigest = “the digest of the flow I read before making these changes” (optional but useful for provenance chains).

You don’t need previousFlowDigest to do collision detection, only flowDigest. But it’s nice for history (“this change was based on that version”).
4. What the writer must do (CLI and API, symmetrically)

Both CLI and API should follow exactly the same pattern, in core:

    Read phase

        Load the flow’s RDF from storage.

        Compute currentDigest_read over canonical N-Quads.

        Keep that in memory.

    Modify phase

        Apply the user’s requested transformation to the in-memory dataset.

        Compute newDigest over the modified canonical N-Quads.

    Check-and-write phase (critical)

    Before writing:

        Re-read the current value of sflo:flowDigest from storage (or from a fast side-channel if you maintain it externally).

        Compare it to currentDigest_read.

    Cases:

        If currentDigest_read == currentDigest_storage:

            Safe to write.

            Replace the flow content with modified dataset.

            Set:

                sflo:previousFlowDigest = currentDigest_read

                sflo:flowDigest = newDigest

        If they differ:

            Refuse to write.

            Return “conflict” to caller, including both digests and maybe a diff hint.

In CLI:

    That means a clean error:

        Flow … changed since you started. Please pull/rebase and re-run weave.

In API:

    That means an HTTP 409 Conflict (or 412 Precondition Failed) with machine-readable info.

If you skip the re-read/compare step, you’re back to blind overwrites.
5. Where the “collision” is actually coming from

Let’s make the adversarial case concrete.
Scenario A: Two CLIs on same repo

    CLI-1:

        git pull

        Reads flowDigest = A

        Makes a change and keeps A as currentDigest_read.

    CLI-2:

        git pull

        Reads flowDigest = A

        Makes different change, writes it first:

            Re-checks: currentDigest_storage == A.

            Writes new RDF and flowDigest = B.

    CLI-1 tries to write:

        Re-checks: currentDigest_storage == B, but it expects A.

        Detection fires → CLI-1 fails with conflict.

Without the guard, CLI-1 would silently clobber B with C, and the fact that B ever existed is only discoverable in Git history after the fact.
Scenario B: CLI vs API

    CLI edits locally and commits/pushes.

    API sits on the same repo or backing store.

Collision detection is still useful because:

    If the API receives a request that was conceptually based on digest A, but the current flowDigest is B, it must refuse that update, not just blast over B.

Otherwise, an old UI page or script that is working from a stale view of the mesh can send mutations that silently revert newer changes.
6. Interaction with Git (and why Git alone is not enough)

You might be tempted to say: “Git already catches conflicts; isn’t that enough?”

Problems with “let Git handle it” only:

    Non-overlapping text edits can still be semantically conflicting.
    RDF graphs might merge textually without Git raising a conflict, yet the overall semantics are wrong (same node defined in two places, catalog pointers inconsistent, etc.).

    Automation and CI.
    If your API or build pipeline auto-commits derived flows, it’s easy to accidentally:

        Rebase/force-push in CI.

        Use “ours/theirs” merge strategies that prefer one side and silently drop the other.

    Multiple writers on non-Git storage.
    If some deployments write to S3, a DB, or some other store instead of a bare repo, you don’t get Git’s semantics anyway.

The digest-based guard is internal to the domain: sflo’s own notion of version equality. Git is a coarse outer layer and can still be relied on, but shouldn’t be your only defense if multiple writers exist.
7. How “hard” do you want the guard to be?

There are three reasonable policies once a collision is detected:

    Hard fail (recommended initially)

        Refuse to write at all.

        Caller must fetch the latest mesh, re-run core, and attempt again.

        Simple, correct, forces people to acknowledge concurrent changes.

    Auto-merge + fail on semantic conflicts

        Try to merge the concurrent changes in-memory:

            Textual or RDF-level merge.

            Run domain-specific invariants (SHACL, custom integrity checks).

        If invariants pass, write merged result with new digest.

        If not, fail with explicit “semantic conflict”.

    This is significantly more complex. You can always add it later; don’t start here.

    Last-writer-wins but logged

        Still detect the mismatch, but allow override if caller explicitly sets a --force or similar.

        Log that a blind overwrite occurred (including both digests) into a provenance graph.

This is the right order to implement in time: (1) now, (2/3) later if needed.
8. How this looks in shared/core API surface

In your core module, you want something like:

interface FlowVersionInfo {
  flowIri: string;
  currentDigest: string; // from storage
}

interface FlowWriteResult {
  newDigest: string;
}

async function updateFlowSafely(
  flowIri: string,
  expectedDigest: string,
  transform: (dataset: RdfDataset) => RdfDataset,
): Promise<FlowWriteResult> {
  const currentDataset = await loadFlowDataset(flowIri);
  const digestAtRead = digestDataset(currentDataset); // canonical N-Quads

  if (digestAtRead !== expectedDigest) {
    // Stale read or external modification
    throw new FlowConflictError(flowIri, {
      expectedDigest,
      digestAtRead,
    });
  }

  const newDataset = transform(currentDataset);
  const newDigest = digestDataset(newDataset);

  // Re-read just before write if storage is shared
  const datasetBeforeWrite = await loadFlowDataset(flowIri);
  const digestBeforeWrite = digestDataset(datasetBeforeWrite);

  if (digestBeforeWrite !== expectedDigest) {
    throw new FlowConflictError(flowIri, {
      expectedDigest,
      digestBeforeWrite,
    });
  }

  await writeFlowDataset(flowIri, newDataset, {
    previousDigest: expectedDigest,
    newDigest,
  });

  return { newDigest };
}

Notes (this is where you should be critical):

    I’m deliberately doing the compare twice (at read and just before write) if storage is shared or slow, because:

        Another writer might sneak in between loadFlowDataset and writeFlowDataset.

    In practice, you can optimize this:

        If you know the storage is “local, single-process” for the CLI, you may not need the second read.

        For API, you probably want the second read or rely on storage-native atomics.

The important part is:

    expectedDigest comes from the caller.

    You never write unless the storage still reflects that digest.

9. When could you theoretically skip this?

If, and only if, all of these are true:

    There is exactly one writer to the mesh.

    That writer never writes based on stale state (i.e., no caching; always read-modify-write in one short critical section).

    There is no remote API that can write to the mesh separately.

    Git merges and CI never auto-write derived flows without running core against the latest HEAD and verifying.

In your world (CLI + API + possible web UIs + CI), this is unlikely to remain true. So you’re better off baking in digest-based collision detection now while the system is still relatively small.
Summary

    “Guard against blind overwrites” = optimistic concurrency control using digests.

    Attach a digest (flowDigest / meshDigest) to whatever unit of data you care about (flow, dataset, node).

    Writers must present the digest they based their change on; writes are rejected if the stored digest changed.

    Git alone is not sufficient, especially once you have multiple writers and automated processes.

    Start with hard fail on conflict; consider smart merge or force-override + provenance later if you actually need it.

If you tell me which level you’re leaning toward (whole mesh vs per-flow vs per-dataset), I can sketch the exact RDF / JSON-LD pattern for the digest triples and the core API signature to enforce it.
You said:
JSON-LD is an RDF dataset. But more importantly, transposability requires that BASE not specified and all "in-mesh" IRIs are "local". BASE has to be inferred. So not sure we can convert to N-Quads.
ChatGPT said:

Good, this is the core tension you actually have to resolve:

    You want transposability → local IRIs, BASE inferred later.

    You also want canonical digests → need some notion of an RDF dataset that is independent of deployment base.

Those are compatible, but only if you stop thinking in terms of “real HTTP IRIs” inside core and instead work in terms of intramesh identifiers.

I’ll walk through what that means and where N-Quads still fits.
1. Separate two layers: intramesh vs published

Right now you’re mentally conflating:

    Intramesh layer

        All @ids for “in-mesh” things are local:

            e.g. "@id": "djradon/bio/", "@id": "_flows/_metadata", etc.

        No @base in the JSON-LD.

        No assumption about https://djradon.github.io/ns/… yet.

    Published web layer

        That same "djradon/bio/" is interpreted as
        https://djradon.github.io/ns/djradon/bio/ in one site,
        and something else in another.

        BASE is a deployment concern.

For collision detection and canonicalization, you only care about layer 1.
You want a digest that says:

    “This intramesh dataset (over local IDs) has not changed.”

Whether that dataset will later be published as https://foo/ns/… or https://bar/ns/… is irrelevant and must not affect the digest.

Conclusion: canonicalization must operate on intramesh terms, not on final HTTP IRIs.
2. You can still do N-Quads — but over intramesh IRIs

The RDF 1.1 purist says: IRIs are absolute.
Your mesh says: I’m going to use “local IRIs” until I choose a BASE.

The way out is:

    Define an intramesh RDF dataset model where terms are:

        LocalIri("djradon/bio/") for in-mesh stuff.

        AbsoluteIri("https://schema.org/Person") for externals.

        Literals as usual.

    Canonicalization operates on this model:

        For hashing, you treat the lexical form "djradon/bio/" as-is; you never prepend the real base.

        Sorting is lexicographic on these strings, with a stable rule that always treats LocalIri one way and AbsoluteIri another.

So “convert to N-Quads” really means:

    Convert JSON-LD to a quad set in your abstract model, then emit a canonical textual form that you define.

That textual form can look like “N-Quads but with relative IRIs allowed”; it doesn’t have to be strictly W3C-valid N-Quads if you don’t want it to be.

You’re not feeding this to generic RDF engines; you’re using it to compute a digest and compare it with an earlier digest.
3. How to avoid letting BASE leak into digests

Your concern is valid: if you call a generic JSON-LD processor with a real document URL as base, it will resolve djradon/bio/ → https://djradon.github.io/ns/djradon/bio/.

Then:

    Move the same mesh to https://semanticflow.net/ns/ → digests change.

To avoid that, you have two viable patterns:
Option A – Use a synthetic, fixed “mesh base”

For canonicalization only:

    Pick a synthetic base that never changes, e.g.
    urn:sflo:mesh: or mesh:/.

    When expanding JSON-LD in core:

        Pass base: "mesh:/" (or embed @base: "mesh:/" in your canonical context).

        Now "djradon/bio/" always becomes "mesh:/djradon/bio/" in the internal RDF dataset.

    Canonicalization:

        Operates on those mesh:/… IRIs.

        Sorting and hashing are stable.

        Deployment base (https://djradon.github.io/ns/, https://semanticflow.net/ns/, etc.) is applied in a later step, outside of core.

So your digest is over an RDF dataset whose IRIs start with mesh:/…, not HTTP.
Transposability is preserved because the digest doesn’t know about the HTTP host at all.
Option B – Don’t resolve locals at all

If you don’t want to play games with @base:

    Treat JSON-LD as mesh-JSON, not generic JSON-LD:

        @id that lacks a : is a LocalIri.

        @id with : is an AbsoluteIri.

    Build the intramesh quad set yourself (or via a very constrained expansion step that doesn’t do base resolution for locals).

    Your canonical quad serialization just prints local IDs as-is:

    djradon/bio/  a   sflo:MeshNode  .
    djradon/bio/  sflo:hasFlow  _flows/_metadata  .

No BASE, no resolution. Your “N-Quads” is technically a custom dialect, but it’s simple and deterministic.

In both options, the key is: don’t ever introduce the real HTTP base into the dataset you hash.
4. How this fits collision detection

Recall the pattern from before:

    You need a flowDigest (or similar) that says “this flow’s intramesh dataset, in canonical form, hashes to X.”

With the intramesh canonicalization:

    Read:

        Load JSON-LD for the flow.

        Convert to intramesh quads (Option A or B).

        Canonicalize (sort quads) and hash → digest_read.

    Modify:

        Apply transformation at dataset or JSON-LD level.

        Recompute → digest_new.

    Guarded write:

        Before writing, load current JSON-LD again, recompute digest_current.

        If digest_current !== digest_read, conflict.

        Else write new JSON-LD and store flowDigest = digest_new.

Because the digest is computed over mesh:/… or raw local ids (not real HTTP IRIs), the following are all true:

    Moving from GitHub Pages to some other host does not change digests.

    Compiling the same intramesh mesh into different published namespaces yields the same canonical digest.

    Collision detection stays purely about semantic equality within the mesh, not deployment details.

5. Where JSON-LD still fits

You’re right that “JSON-LD is an RDF dataset” in the sense that:

    There is a well-defined mapping from JSON-LD to an RDF dataset.

But for you:

    JSON-LD is the storage format for intramesh datasets.

    For hashing/collision detection, you should not rely on whichever document URL or @base the JSON-LD processor happens to use.

    Instead, core should define a mesh-aware expansion:

        Either via a synthetic base (mesh:/) or by treating local IDs as first-class local terms.

Then you can still:

    Use JSON-LD contexts for everything (terms, @vocab, etc.).

    Serialize/distribute JSON-LD without @base.

    Preserve transposability.

The only extra rule is:

    When shared/core computes digests or compares flows, it operates on an intramesh dataset where local IDs are normalized in a way that does not involve the real deployment base.

6. What I’d recommend concretely

If you want a concrete decision:

    Pick Option A (synthetic mesh base) for simplicity:

        Define a reserved base, e.g. urn:sflo:mesh: or mesh:/.

        In core’s JSON-LD handling:

            If @id is local (no :), treat it as relative to mesh:/.

            If @id is absolute, leave as is.

        Canonical N-Quads over that dataset → flowDigest.

        Published IRIs are obtained later by replacing the mesh:/ prefix with the chosen namespace root for that site.

    Document this as:

        “The sflo mesh canonicalization layer interprets all in-mesh identifiers as mesh:/… for the purposes of hashing and comparison. Deployment bases are applied only when publishing, and never participate in digests.”

That gives you:

    Proper RDF-ish canonicalization.

    Stable digests for collision detection.

    Full transposability of JSON-LD files between namespaces.

If you want next, I can spell out the exact rules for “local vs external” IDs and the canonical quad ordering you should bake into shared/core.
You said:
I think all elements have to have an ID. otherwise, blank nodes. Is Unicode-normalize different than just straight jsonld text normalize?

Regarding collision detection, there is no "modify in memory"... the _working snapshot is the only thing that should change, and it's only on disk. If two processes/users overwrite each others _working changes, so be it.
ChatGPT said:

You are mixing three different issues here, and they need to be separated cleanly:

    Blank-node avoidance

    Canonicalization (for digests)

    What collision detection actually means if the “working snapshot” is the only mutable thing

I’ll take them one by one and highlight the inconsistencies.
1. “I think all elements have to have an ID. Otherwise, blank nodes.”

Correct:
If you want no blank nodes anywhere, then every node in every flow must have an @id.

But beware of a subtle trap:
JSON-LD can inject blank nodes even if you think you avoided them

Examples:

    A value object without an @id but with nested properties → blank node.

    A @graph array of objects without @id → blank nodes.

    A list @list containing objects without @id → blank nodes.

    Using a context that turns something into a node map → blank nodes.

Conclusion:
You must enforce, in shared/core, a rule:

    Every node in the intramesh JSON-LD must carry @id.
    If JSON-LD expansion produces any _:b0, the shape is invalid.

This is not optional if canonicalization is based on subject IRIs.
2. Unicode-normalize vs JSON-LD “text normalize”

These are not the same thing.
Unicode-normalize (NFC/NFD/NFKC/NFKD)

This operates on string codepoints.
Its purpose: prevent "é" (single codepoint) vs "e" + "́" (two codepoints) from being considered different.
JSON-LD “text normalize”

JSON-LD canonicalization (URDNA2015) does:

    lexical escaping normalization

    datatype normalization

    literal compare rules

    producing canonical N-Quads

It does not enforce Unicode normalization on plain string values.

So yes, you must explicitly Unicode-normalize string literals before hashing.
3. “There is no ‘modify in memory’ … _working snapshot is the only thing that should change.”

This is the contradiction you must resolve.

You previously agreed:

    The metadata flow is the authoritative source of truth.

    Multiple writers (CLI, API, maybe future UIs, maybe CI pipelines) could write the working snapshot.

    You want to avoid blind overwrites?
    → That requires detecting when the working snapshot changed since you last saw it.

Now you’re saying:

    “If two processes/users overwrite each other’s _working changes, so be it.”

This eliminates the whole need for collision detection, but it introduces a logical problem:
If “working snapshot” is the sole mutable state, then any overwrite destroys history unless you have version gating.

With “so be it,” you are implicitly adopting a last-writer-wins model with no concurrency control.

That is valid only if you are OK with:

    Users losing edits silently,

    API calls based on stale read producing wrong output,

    CI pipelines producing nondeterministic results depending on race timing,

    No protection against someone overwriting the working snapshot with totally unrelated data.

You can choose this, but you should be aware that:
With “working snapshot only, no guards,” the system is not append-only, not monotonic, and not semantically stable.

Which is directly at odds with your usual mental model for flows (genesis → tombstone, deltas, content digests, versioning semantics).

So you must decide intentionally which world you're in.
4. A critical question:

What is the purpose of the working snapshot?

Two possible answers:
A. The working snapshot is merely a staging area, not authoritative

Then overwrites don’t matter.
The authoritative data is in versioned flow datasets (v1, v2, …).
Collision detection is not needed.

But then you must guarantee:

    Writers never operate based on the in-progress working snapshot,

    Only one process writes new versions,

    API read operations never depend on the working snapshot.

B. The working snapshot is authoritative until released

Then blind overwrites will corrupt semantic state, and you do need collision detection.

Right now you've been operating under B all day.
Now you are jumping to A, which invalidates your previous constraints.

You need to pick one.
5. If you truly want A (“working snapshot can be overwritten without guards”)

Then your architecture becomes:

    Working snapshot = buffer
    Always mutable, last write wins. No digest checking.

    Versioned datasets = authoritative
    These are immutable (v1, v2, …) and guarantee provenance.
    Only a release operation freezes something.

In that world:

    There is no point in collision detection at the working level.

    Collision detection only matters for versioned flows, not for working snapshots.

    The canonical digest applies to versioned data, not working data.

This is consistent and workable.

But:
Your earlier wish for “no silent stomping of others’ edits” is impossible under this model.
6. If you actually want B (“metadata flow is truth until released”)

Then your statement “so be it” is not acceptable; it breaks correctness.

You need:

    per-flow digest

    guarded writes

    mesh digest or working-snapshot digest to avoid blind overwrites

and your earlier design stands.
7. My critical recommendation

You must answer one binary decision:
Do you want the working snapshot to be authoritative or ephemeral?

If authoritative:
Collision detection is required.
Blind overwrites destroy semantic correctness.

If ephemeral:
Collision detection for working snapshot is unnecessary.
But then all authoritative state must be versioned-only (immutable v1/v2), and working state must not be used by any reader except the writer.

Right now your statements conflict.

You must pick, or the architecture can’t be made consistent.

If you tell me which side you choose, I will redraw the simplest consistent model for flows + working snapshot + digests.
You said:
I want "The working snapshot is merely a staging area, not authoritative"
ChatGPT said:

Good, that makes the model much simpler, but you have to really commit to it.

I’ll treat two things:

    What your “working snapshot is staging only” choice implies for collision detection.

    The loose ends you raised (IDs / blank nodes, Unicode normalization).

1. If working snapshot is staging, collision detection moves elsewhere

Your choice means:

    The only thing you care about preserving carefully is released, versioned data, not the in-progress working copy.

So:

    _working (or whatever you call it) is:

        Mutable.

        Last-writer-wins.

        Not authoritative.

        Safe to overwrite with no guards.

    Authoritative truth lives only in:

        Versioned dataset IRIs: _main/v1, _main/v2, … (or _default/v1, etc.).

        Catalog pointers to “current” version.

If that’s really the model, then:
a. No collision detection needed on _working

You can literally:

    Open JSON-LD file on disk,

    Write changes,

    Close file.

If two users/processes stomp each other → only the staging copy is lost. That is acceptable by your stated choice.

But that’s only safe if you enforce:

    No consumer that matters (API, web UI, other tools) ever treats _working as the source of truth.

APIs must read from released versions only (via catalog), or you’re back to blind-overwrite semantics on authoritative state.
b. Releases become the interesting operations

The only operation where correctness really matters is:

    “Take the working snapshot and mint a new released version out of it.”

That operation should:

    Read _working once.

    Create a new immutable dataset version (vN+1) from it.

    Update catalogs / “current” pointers to say:

        _main/current → vN+1 (or the equivalent pattern you settle on).

Things you don’t do during release:

    You do not overwrite existing vN content.

    You do not mutate historical versions.

So even if two people release different _working snapshots “based on the same previous version,” you don’t lose history; you just get:

    v3 and v4 both existing.

    Catalog pointing to whichever “current” one wins in Git / merges.

Git conflicts and merges will sort that out on the catalog file; but both versions still exist.
c. Where collision detection still makes sense (and is cheap)

You might still want collision detection at the catalog pointer level, not at _working:

    Say your catalog has:

    <_series/_main/dataset> sflo:currentVersion <_series/_main/dataset/v3> .

    A release based on v3 expects to bump currentVersion to v4.

You have two processes:

    Release-A: intends to do v3 → v4.

    Release-B: also intends to do v3 → v4, based on an earlier snapshot.

If you don’t care which one wins, it’s fine to let Git sort it out.

If you do want to avoid surprising “jump backwards” behavior, then:

    Attach a datasetDigest (or seriesDigest) to the catalog entry,

    Check it before bumping currentVersion.

But notice: this is collision detection on versioned catalogs, not on _working. You are protecting the integrity of history and “current” pointers, not in-progress edits.

If you’re genuinely okay with last-writer-wins on currentVersion, you can skip even this and let Git merges be the only conflict mechanism.
2. Implications of your choice (places you can screw this up)

If you really want _working to be staging-only, you must not do these things:

    Do not serve _working from the API for anything that matters.

        No “preview” endpoints that others treat as real.

        No web UI that writes directly to _working and expects that to be durable truth.

    Do not branch semantics based on _working.

        Any SHACL checks, invariants, or cross-flow integrity checks that determine “is the mesh valid?” must run on the released graph, not _working.

        Otherwise the answer depends on who happens to be editing.

    Do not tie release numbering to _working content.

        Version numbers (v3, v4) are monotonically increasing, independent of whether _working changed once or a hundred times.

        The release operation is “snapshot the staging area,” not “track every staging change.”

If you violate any of those, you’ve implicitly made _working authoritative again, and then your “so be it” stance on concurrent overwrites becomes dangerous.
3. IDs and blank nodes

You’re right: in your world, every node should have an ID; otherwise JSON-LD will create blank nodes.

You should make this a hard rule:

    Rule: All intramesh nodes must have @id.

    Enforcement: In core, after expansion, scan the dataset:

        If any subject or named graph is a blank node (_:…), that’s a validation error.

        Don’t even try to release such data.

That’s totally compatible with your “ephemeral _working” model:

    You can choose to enforce it only for releases (i.e., you can have messy _working, but only clean data becomes a version), or

    Enforce it whenever _working is written (stronger, saner).

Given how fussy your overall design is, I’d enforce it on any write: no blank nodes anywhere, ever.
4. Unicode normalization vs “JSON-LD text normalize”

You asked:

    Is Unicode-normalize different than just straight jsonld text normalize?

Yes:

    JSON-LD canonicalization (URDNA2015) normalizes literals in terms of RDF datatypes, escaping, etc.

    It does not guarantee Unicode NFC/NFD normalization of the actual text payload.

So, if you care about byte-accurate digests of versioned datasets, and you allow non-ASCII text:

    You should pick one Unicode normalization form (almost always NFC).

    Normalize all string literals to that form before you generate canonical N-Quads and hash.

This affects versioned data, not _working:

    _working can contain whatever as long as on release you normalize before writing the versioned dataset and computing its digest.

    That gives you stable digests across platforms and editors.

5. What you actually need now

Given your decision, a minimal consistent design is:

    _working:

        Single JSON-LD file (or set of files) per dataset series.

        Mutable, last-writer-wins.

        May be dirty, but you should still forbid blank nodes if you don’t want surprises later.

    release operation:

        Reads _working.

        Validates: no blank nodes, shapes pass, etc.

        Normalizes:

            Intramesh IRIs (e.g., mesh:/ base) for canonicalization.

            Unicode strings to NFC.

        Emits canonical N-Quads for hashing.

        Computes datasetDigest.

        Writes a new immutable version (vN+1) and updates catalog/current pointers.

    Collision detection:

        Optional for “current version” if you care about not flipping back and forth.

        Not used at all on _working.

That matches your “staging only” stance and doesn’t drag you back into full-blown optimistic concurrency control.

If you want, next step would be to pin down:

    Exact filenames/paths for _working vs vN vs catalog,

    and where datasetDigest lives in the RDF for a versioned dataset.

You said:
I'd decided to move from _vN snapshot naming to weave-time timestamp based. Since weave time will also be capture in (RDF) metadata, the benefits are mostly user-experience... you can see roughly when the snapshot was woven, length of time between snapshots, etc. I rationalized "as-of across meshes" but didn't really nail down what that means concretely. Here's my task ticket for the change. Please help me evaluate whether this approach is really better that _vN.

---
---
id: 8akbl2qj0nz38yrvet4oq3k
title: 2025-11-09-replace-monotonic-vN-with-timestamp
desc: ''
updated: 1762878216601
created: 1762712674617
---

## Decisions

- to facilitate "as-of" across meshes, we decided to move from monotonic (_vN) versions to "snapshotTime" versions with millisecond precision, e.g. 20251109181158123. On weave, lock CRUD (if using API; if using fs, you could consider locking files but meh), check local clock's accuracy, pick a weaveTime, and use it for "releasing" all the "working" distributions. 

## Prompt

You are migrating Semantic Flow from monotonic integer versions (_v1, _v2, …) to timestamp-based snapshot IDs.

GOAL
- Replace all versioned snapshot folders and references with a UTC timestamp identifier in compact form (YYYYMMDDhhmmssSSS), called snapshotTimestamp.
- Keep behavior identical or better: immutable snapshots, easy “as-of time” queries, stable manifests, and optional default materialization.
- Preserve backward compatibility reads for _vN paths until deprecation.

TERMS
- snapshotTimestamp (folder name and identifier): 17-digit UTC time “YYYYMMDDhhmmssSSS”. Example: 20251109181158123.
- Working copy: _working/
- Default snapshot: _default/


REQUIREMENTS
1) Folder layout
   - For each Flow directory:
     - _working/
     - <snapshotTimestamp>/   # immutable
     - _default/              # optional, copy of latest published snapshot
   - Example:
     /ns/djradon/_reference-flow/
       _working/
       20251109181158123/
       20251109182247044/

2) Identifier format and generation
   - snapshotTimestamp format: ^\d{17}$ (UTC, pad ms to 3 digits).
   - Source of truth is server time from sflo-host, although we could consider an NTP check

3) Metadata
   - On each snapshot dataset (folder):
     - prov:generatedAtTime "2025-11-09T18:11:58.123Z"^^xsd:dateTime .
   - On _default/ :
     - prov:wasDerivedFrom </…/20251109182247044/> .
   - Ontology additions:
     - sflo:snapshotTimestamp a owl:DatatypeProperty ; schema:domainIncludes sflo:Snapshot, dcat:Dataset ; schema:rangeIncludes xsd:string ; rdfs:comment "Compact UTC timestamp ID YYYYMMDDhhmmssSSS for an immutable snapshot." .
     - (Optional) sflo:hasDefaultSnapshot a owl:ObjectProperty ; rdfs:domain sflo:Flow ; rdfs:range dcat:Dataset .

4) Manifest (atomic cut)
   - Path: /_manifests/manifest-<snapshotTimestamp>.trig
   - Shape:
     :m a sflo:WeaveManifest ;
       prov:generatedAtTime "…"^^xsd:dateTime ;
       sflo:select [ sflo:flow </ns/djradon/_reference-flow/> ; sflo:snapshot </ns/djradon/_reference-flow/20251109182247044/> ] , … .
   - Write manifests using stage → fsync → atomic rename. Treat manifest publish as the commit point.
   - Readers that require consistency MUST resolve via a manifest.

5) API changes
   - POST /flows/{id}/snapshots → 201 Created
     - Location: …/<snapshotTimestamp>/
     - Body: { "snapshotTimestamp": "20251109181158123", "generatedAt": "2025-11-09T18:11:58.123Z" }
   - POST /weaves → 201 Created (or 202 Accepted for async job)
     - Returns manifest URI and its snapshotTimestamp.
   - GET endpoints accept either legacy _vN/ or timestamp; server resolves both while legacy is supported.

6) Backward compatibility
   - Read path adapter: map _vN/ → its recorded snapshotTimestamp if present; otherwise serve legacy.
   - Deprecation plan: warn for _vN/ in logs; remove after N releases.

7) SHACL guards (optional)
   - Folder IRI pattern for snapshots:
     sh:targetClass dcat:Dataset ;
     sh:sparql "FILTER regex(str(?this), '/[0-9]{17}/$')" .
   - Property constraint on snapshotTimestamp:
     sh:path sflo:snapshotTimestamp ; sh:pattern "^[0-9]{17}$" ; sh:minCount 1 ; sh:maxCount 1 .

8) “As-of” selection
   - Given an instant T, for each Flow pick the max snapshotTimestamp ≤ T. No inner file timestamps required.
   - Provide helper: GET /flows/{id}/snapshots?asOf=2025-11-09T18:12:00Z → returns chosen timestamp and URI.

9) Filesystem publish semantics
   - No global locks. For each new snapshot:
     1) Write to ._staging/<snapshotTimestamp>/, fsync.
     2) Rename into place (same volume).
     3) Write manifest, rename into place.
     4) Optionally refresh _default/ via atomic directory swap.
   - Document guarantee as “snapshot publish”: manifest yields point-in-time consistency; _default/ is convenience.

10) Clock drift
   - sflo-host is authoritative, but check internet time and record skew if detected:
     sflo:clockSkew "+00:00:01.234" on the snapshot.
   - Always store prov:generatedAtTime in UTC.

11) Testing and acceptance
   - Unit: format generator, collision bump, parsing.
   - Integration: create two snapshots in the same ms → unique IDs.
   - Manifest atomicity: readers never see partial cuts.
   - Backcompat: _vN/ and timestamp both resolve during deprecation window.
   - As-of correctness: queries select expected snapshots across multiple flows.

DELIVERABLES
- documentation changes (look for "_vN"), ID generation
- Ontology and SHACL updates (as above).

## TODO

ChatGPT said:

Short version: timestamp-based IDs are workable, but they don’t automatically buy you what you’re claiming. As written, the design is more of a UX refactor than a semantic improvement over _vN, and it introduces some new edge cases (clock, collisions, cross-mesh “as-of” semantics) that you haven’t fully closed.

I’ll walk through:

    What your ticket actually guarantees.

    Where timestamps are strictly better / strictly worse than _vN.

    The shaky part: “as-of across meshes.”

    A concrete recommendation.

1. What your ticket really does

Stripped down, the change is:

    Replace _vN/ with /<snapshotTimestamp>/, e.g. 20251109181158123/.

    Keep _working/ as staging, and _default/ as a convenience alias to the latest published snapshot.

    Record the weave time both:

        in the folder name (the timestamp ID), and

        as prov:generatedAtTime in RDF.

    Add an optional manifest (/_manifests/manifest-<snapshotTimestamp>.trig) that captures a point-in-time cut across flows.

    Support API queries like “give me the snapshot as-of time T” per flow.

Semantically:

    Snapshots remain immutable.

    “Current” is _default/ or whatever the catalog points to.

    The only real change in semantics is: version identity comes from a timestamp instead of a simple counter.

Everything else you describe (atomic publish, manifests, prov metadata) could exist equally well with _vN.

So the real question is: does timestamp-as-identifier justify itself versus a simpler _vN?
2. Timestamps vs _vN: concrete trade-offs
Where timestamps are strictly better

    Human temporal intuition at a glance

    You can see “roughly when this was woven” and spacing between snapshots just from the folder name:

        20251109181158123/ vs 20251111103000000/ is self-explanatory.

        _v3/ vs _v4/ tells you nothing without opening RDF or logs.

    That’s a real UX gain for:

        Browsing the filesystem.

        Debugging chronologies without tooling.

        Explaining to other humans.

    Natural ordering by lexicographic sort

    With fixed width digits, YYYYMMDDhhmmssSSS sorts lexicographically the same as time. A simple directory listing is time-ordered snapshots.

    _vN also sorts lexicographically if you zero-pad (e.g. _v0003), but you didn’t — and the ticket explicitly uses timestamps with padding.

    Cross-mesh visual alignment

    Not “as-of semantics” (that’s more subtle, see below), but purely:

        If multiple meshes weave around the same time, their snapshot IDs look aligned.

        That’s useful when eyeballing two repos and wanting a rough sense “these builds happened around the same epoch.”

    Still, that’s in the UX/aesthetics bucket, not semantics.

Where timestamps are strictly worse or more fragile

    Clock dependency and drift

    You now depend on:

        A single authoritative clock (sflo-host) for correctness of ordering.

        NTP or some drift detection.

    Edge cases:

        NTP adjustment backwards → you can get timestamps that go “backwards in time” relative to earlier snapshots.

        Running the weave offline / in a different environment → no shared authoritative time.

    With _vN, you don’t care what the wall-clock does. Sequence numbers are inherently monotonic within a series.

    Uniqueness under concurrency

    Your spec hints at “collision bump” tests, but the ticket doesn’t fully specify:

        If two snapshots are created within the same millisecond on the same host, what happens?

            You either:

                Allow collisions → broken invariants.

                Or you tweak the timestamp (e.g., increment the ms field) and now the ID no longer matches real time.

                Or you add a suffix (counter, digest), which breaks the ^\d{17}$ contract.

    This is a concrete gap. _vN trivially avoids this: you take a lock / CAS on the version number and bump it.

    Loss of simple ordinality

    Humans and tools often want “latest, previous, N versions ago.”

    With _vN:

        “Previous” is trivially _v(N-1).

        “Two versions ago” is _v(N-2).

    With timestamps:

        You must list all snapshots, sort, and then pick the N-th from the end.

        That’s fine for tools, but worse for human mental model.

    You can reintroduce a monotonically-incrementing integer in RDF metadata (e.g., sflo:sequenceNumber) to recover this, but that undercuts “simpler than _vN”.

    Tight coupling of name to one particular narrative of time

    You’ve implicitly said “snapshot ID == weave time”:

        If you ever want to:

            Recreate a snapshot from an earlier commit,

            Re-derive the same snapshot while fixing a tooling bug,

        then using “real weave time” as ID becomes semantically muddy:

            Do you reuse the old timestamp (implying time-travel),

            Or mint a new one (implying two different IDs for semantically-identical snapshots)?

    _vN is intentionally narrative-agnostic: it just represents “the N-th release,” not when you made it.

3. “As-of across meshes”: this is still underspecified

You wrote:

    to facilitate "as-of" across meshes, we decided to move from monotonic (_vN) versions to "snapshotTime" versions

This is the weakest part of the justification. Concretely:
What do you actually mean by “as-of across meshes”?

The ticket hints:

    Given time T, for each Flow:

        Pick the max snapshotTimestamp ≤ T.

    “As-of across meshes” then means:

        For many flows in many meshes, select snapshots by the same wall-clock T.

But there are problems:

    Dependency graph vs wall-clock

    Suppose:

        Mesh A’s snapshot at 2025-11-09T18:11:58.123Z was built using Mesh B’s snapshot at 2025-11-10T00:00:00.000Z (e.g., A ingests B via some import that lagged).

        A naive “as-of T” selection across meshes based solely on timestamps can produce inconsistent worlds where:

            A snapshot “refers to” (or is known to have been built with) B’s snapshot, but your as-of selection picks an older B snapshot, because its timestamp ≤ T.

    That means “as-of T” is not a true “coherent world,” just “per-flow nearest ≤ T.”

    You can fix that only with manifests or explicit dependency graphs, not with directory names.

    You’re already introducing manifests

    Your own design says:

        A weave writes / _manifests/manifest-<snapshotTimestamp>.trig.

        That manifest selects which snapshot of each flow belongs to that weave.

    That is already:

        A precise, cross-mesh “as-of” snapshot in SFLO terms.

    You don’t need timestamp-based folder names to do this. A _vN world where each weave has a manifest referencing _vNs yields equivalent “as-of” semantics, with less clock fuss.

    Nothing stops you from storing time in RDF under _vN

    You already plan to:

        Add prov:generatedAtTime to each snapshot dataset.

    This trivially supports “as-of by wall-clock across meshes,” regardless of whether the folder name is _v3/ or 20251109181158123/.

    Tools can:

        Load the catalog/manifest for each flow.

        Filter snapshots by generatedAtTime ≤ T.

        Pick the max.

    So the “as-of” story does not require the timestamp to be baked into the path; only into the metadata.

In other words: timestamp names slightly simplify naive filesystem-only as-of selection, but for any serious as-of semantics, you’re already relying on manifests and RDF metadata.
4. Is timestamp really “better” than _vN?

Given all of that:
What timestamp IDs genuinely give you

    Better filesystem-level UX:

        Readable temporal order.

        Easy eyeballing of time gaps between snapshots.

    A uniform per-snapshot name space that is naturally sortable by time.

    Some emergent cross-mesh alignment value (“these snapshots are from roughly the same period”).

What they do not give you (that you might be implicitly attributing)

    True cross-mesh “as-of world” semantics.
    For that, you still need manifests or dependency information.

    Stronger or simpler semantics than _vN for:

        Immutability.

        Linear history.

        Current pointer.

    Any replacement for digests; you still need content hashes for equality and collision detection (if you later add it) at the versioned level.

What they cost you

    Reliance on clock correctness and collision-avoidance rules.

    Loss of trivial ordinal “v3, v4, v5” semantics, unless you reintroduce a sequenceNumber in RDF.

    More opaque filenames for many humans; 17-digit numbers are less “speakable” than _v7.

So no, this is not objectively “better” than _vN. It is a trade:

    You trade simple ordinals for temporal UX and time-encoded IDs, while taking on clock concerns.

    Semantically, both schemes support the same invariants once you have:

        immutable snapshots,

        generatedAtTime,

        and manifests.

5. A concrete recommendation

If you want a consistent and defensible design:
Option 1 – Commit fully to timestamp IDs, but fix the gaps

If you keep the timestamp scheme:

    Add a sequence number in RDF, per series:

        sflo:sequenceNumber (integer) on each snapshot/dataset.

        That restores “vN” semantics for tools/humans that care about ordinality, without encoding it in the path.

    Specify collision resolution explicitly:

        If two snapshots are created within the same ms:

            Either:

                Require that snapshot creation be serialized per series (e.g., via a mutex in sflo-host).

                Or allow a suffix to the folder name (e.g., 20251109181158123a) and relax the ^\d{17}$ assumption.

        Right now, your tests mention “collision bump” but the spec is silent.

    Stop pretending “timestamp path == as-of semantics”.

        Document that cross-mesh “as-of” must go through manifests or generatedAtTime, not just directory names.

        The timestamp ID helps humans and simple scripts, but is not the authoritative “as-of” mechanism.

This makes the timestamp choice honest: it’s a UX and tooling convenience, not a semantic upgrade.
Option 2 – Go back to _vN, keep temporal metadata and manifests

If you want simplicity and robustness:

    Keep _vN/ folder naming as the versioned series.

    Add/keep:

        prov:generatedAtTime in RDF for each _vN.

        A manifest per weave that references specific _vNs.

    Support “as-of T” across meshes via:

        Queries over generatedAtTime, or

        manifests created at time T.

You can still:

    Expose timestamp-rich info in the manifest and catalogs.

    Provide API helpers for ?asOf=... that internally translate to “pick the _vN with max generatedAtTime ≤ T.”

This keeps everything conceptually cleaner and avoids clock-collision semantics leaking into path identity.

If your sole strong reason for timestamp IDs is “better filesystem UX and visual time,” Option 1 is fine as long as you fix the collision rule and stop over-claiming “as-of across meshes” benefits.

If you actually want the simplest consistent semantics and are willing to sacrifice timestamp-encoded folder names, Option 2 is more defensible and strictly easier to reason about.
You said:
Explain "Dependency graph vs wall-clock" more.  I'm suspicious that it's really an issue.

I don't think my design has any manifest, it just has a metadata flow. (One interesting question is whether metadata flows each include complete history or just ops since the last weave).  It's slightly related in that, if we have a metadata bump on every weave of any set of other flows, at least that metadata snapshot identifier will match the other flows. Otherwise, we'll have v100001 metadata flow and v3 reference flow.

The ordering alone almost makes timestamp snapshot names worth it.... we could zero-pad, but inevitably whatever number we choose will be an arbitrary limitation.


Metadata 
  [[concept.metadata.provenance]] (entities, agents, activities)
- Copyright and licensing
- validation / consistency checks
- metrics

I think it'd be okay to only include provenance for the latest weave, instead of complete history. And then repeat copyright/licensing, updating if necessary. So two kinds of metadata: timeless (creator) vs versioned (author).
ChatGPT said:

I’ll focus on two things:

    What “dependency graph vs wall-clock” actually means, with concrete SFLO-ish examples.

    How that interacts with your “metadata flow instead of manifest” and timestamp vs _vN.

1. Dependency graph vs wall-clock: what I actually meant

When you say “as-of across meshes at time T”, there are at least two possible meanings:

    Per-flow wall-clock cut
    For each flow/mesh, pick the latest snapshot whose own timestamp ≤ T.

    Consistent world cut along a dependency graph
    Pick a set of snapshots such that:

        Each snapshot’s timestamp ≤ T, and

        The snapshots respect your dependency graph (e.g., if A was computed from B’s snapshot X, the “as-of-T” choice for B is ≥ X).

Wall-clock timestamps only directly support (1). They do not automatically guarantee (2), which is where “dependency graph vs wall-clock” bites.
Very simple scenario

Flows: A and B.

    A depends on B (A reads some dataset from B during weave).

    Both are part of some larger “world” you’d like to do as-of queries on.

Timeline (UTC):

    At t1 = 2025-11-09T00:00:00.000Z, you weave B, producing snapshot B@t1.

    At t2 = 2025-11-09T01:00:00.000Z, you weave A, and A explicitly uses B@t1 internally. So A@t2 is semantically built “on top of” B@t1.

Now consider “as-of T = 2025-11-09T00:30:00Z”.

Using per-flow wall-clock cut:

    For B: latest snapshot ≤ T is B@t1 (OK).

    For A: no snapshot ≤ T (A’s only snapshot A@t2 is later than T) → you either:

        exclude A (if you interpret as “state as known up to T”), or

        pick an earlier A snapshot if one exists.

If you later have:

    At t3 = 2025-11-09T02:00:00.000Z, you weave B again → B@t3.

Then for “as-of T = 2025-11-09T03:00:00Z”:

    For A: latest snapshot ≤ T is still A@t2.

    For B: latest snapshot ≤ T is B@t3.

But semantically, A@t2 was built with B@t1, not B@t3.

So the “as-of 03:00” pair (A@t2, B@t3) is not a world that ever existed as a coherent weave. It’s: A from before B changed, B from after.

Is that wrong? Depends what “as-of” means to you:

    If “as-of T” just means “last known commit per flow by that time”, this is fine.

    If “as-of T” is supposed to approximate “a consistent global state of the world at time T”, then you actually care about the dependency graph, not just individual timestamps.

Now amplify this across meshes (separate repos, asynchronous weaves, ingestion lags, etc.) and you see the general shape of the problem: time order alone doesn’t encode which snapshot was used to build which other snapshot.

That information lives in your provenance (prov:wasDerivedFrom, sflo:imports, etc.), not in timestamps.
2. Relating this to your design (metadata flow instead of manifest)

You’re right: your current concrete design doesn’t really talk about manifests; it talks about a metadata flow that may or may not capture:

    which flows were woven in each “weave run”, and

    which snapshot of each other flow they depended on.

Conceptually, the “manifest” I was referring to is just:

    “A graph that says: in weave run W, flow X produced snapshot Sx, using snapshots Sy, Sz, … of other flows.”

You can implement that as:

    a dedicated manifest file under /_manifests/..., or

    a metadata flow that gets bumped each time you weave anything and records the dependencies.

If you:

    bump the metadata flow on every weave, and

    have it record, for each weave run:

        the snapshotTimestamp of the flows it just produced, and

        the snapshotTimestamps (or digests) of other flows it read,

then that metadata flow is your dependency-aware “as-of manifest”, regardless of whether you call it that.
Where the dependency vs wall-clock tension comes in for you

You said:

    Otherwise, we'll have v100001 metadata flow and v3 reference flow.

Exactly. With monotonic _vN per series, their counters drift independently:

    Metadata flow: _v100001

    Reference flow: _v3

Nothing says “these were produced in the same weave run”.

If instead you:

    adopt timestamp-based snapshot names for both, and

    ensure that every flow woven in a given run gets the same snapshotTimestamp,

then:

    metadata snapshot 20251109181158123/

    reference flow snapshot 20251109181158123/

are aligned exactly because they share the same timestamp.

So far so good — this is a real benefit of using weave-time-based IDs, but only if you share them across outputs of a single weave.

But note what’s actually doing the work here:

    It’s not “timestamps vs _vN” by themselves.

    It’s the invariant:

    “All flows produced in the same weave run share a common weave identifier.”

You could have modeled that with _vN plus a “weave ID” or “build ID” in metadata; timestamps just let you reuse weave time as both “build ID” and folder name.

So the dependency vs wall-clock issue is only mitigated if you commit to:

    One weave run → one “weave ID” (you’re proposing snapshotTimestamp) used across all flows touched in that weave.

    Metadata flow records which flows/snapshots were produced in that weave, and which earlier snapshots they used.

In that world:

    “As-of weave W” is manifest-style (using metadata), not “max timestamp ≤ T per flow”.

3. Is “dependency vs wall-clock” really an issue for your use case?

Given your clarified intent, I’d break it down like this:
Case 1: You care about “as-of weave”, not “as-of physical time”

If the main query you actually want is:

    “Give me the mesh as-of weave run W”
    (e.g., for debugging, reproducing a site, etc.)

Then you should:

    Treat the weave identifier (timestamp-based or not) as primary.

    Use metadata flow to say “these snapshot IDs belong to weave W.”

In that case, the “dependency vs wall-clock” concern is mostly academic:

    You’re not selecting by an arbitrary time T; you’re selecting by a weave, which is a unit that already bakes in dependencies.

    The filesystem timestamp path is mostly a convenience; the real as-of semantics live in metadata.

Case 2: You really want “as-of physical time T across meshes”

This is closer to what your ticket text describes: ?asOf=2025-11-09T18:12:00Z.

Then:

    If you simply do “max snapshotTimestamp ≤ T per flow”, you will get combinations where:

        Some flows were woven with knowledge of future snapshots of other flows,

        But your as-of selection picks older snapshots for those dependencies because of the ≤ T filter.

Is that wrong? In a strict information-theoretic sense, yes:

    The “state of the world as-known-by-A” could include information from B that your as-of(B) selection ignores.

Is it practically catastrophic? Maybe not:

    Most users of your system won’t be reasoning at that granularity.

    If “as-of T” is presented as a best-effort, per-flow time cut, this is acceptable.

The problem is only serious if you want to present “as-of T” as a rigorous, global, dependency-consistent state. You haven’t actually specified a use case that strong yet.

Right now, your candid text sounds more like:

    “I want to approximate ‘what did this mesh look like around T’,”

    “and I want folder names that make temporal browsing sane.”

For that, the wall-clock-based IDs are fine. You just have to be honest with yourself that:

    The eventual rigorous story for cross-mesh consistency is in metadata (provenance graph), not in directory names.

4. Timestamp snapshot names vs _vN, revisited in light of this

You said:

    The ordering alone almost makes timestamp snapshot names worth it.... we could zero-pad, but inevitably whatever number we choose will be an arbitrary limitation.

Two separate points here:

    Ordering

        Timestamp IDs: lexicographic order == time order; humans can eyeball it.

        _vN: you can trivially zero-pad to any width and get lexicographic == numeric order too. There is no real limitation; you can choose _v0000000001 and never hit the ceiling in practice.

    So ordering by itself is not a killer argument. It’s nice, but solvable for _vN.

    “Arbitrary limitation”

        With _vN, yes, you choose some width if you care about lexicographic sort without zero-padding file system operations. But nothing forces you to bound N conceptually.

        With timestamps, you’ve implicitly chosen a different arbitrary limitation: 17 digits to millisecond precision, relative to the epoch.

            You still have collision and clock issues.

            You are now binding semantics of “ID” to a specific time encoding.

The real distinctive advantages of timestamp IDs in your context are:

    A single value can be reused as:

        “Weave ID” (shared across flows in one run),

        Folder name,

        And human-readable approximate time.

Which means:

    If you maintain the invariant “all flows woven in the same run share that ID”, you get:

        No v100001 vs v3 misalignment: everything from that run has the same timestamp as suffix.

        A human can see both sequencing and approximate time from a single token.

That’s the serious justification, not ordering alone.
5. Metadata flow: full history vs latest only

You wrote:

    I think it'd be okay to only include provenance for the latest weave, instead of complete history. And then repeat copyright/licensing, updating if necessary. So two kinds of metadata: timeless (creator) vs versioned (author).

This is orthogonal to timestamp vs _vN, but important:

    Timeless metadata (creator, copyright owner, base license, perhaps high-level description) is naturally attached once and rarely changes.

    Versioned metadata (who ran this weave, with which toolchain, which flows and snapshots were inputs, metrics, validation results) is weave-specific.

If you only keep provenance for the latest weave in the metadata flow:

    You lose the ability to:

        Answer questions like “what did the metrics/provenance look like for snapshot from 6 months ago?”

        Reconstruct dependency-respecting “as-of weave W” other than “current”.

Given your tendency to care about debugging and reproducibility, that’s a real tradeoff.

A sane compromise:

    Metadata flow carries:

        Timeless metadata about the flow/mesh as a whole.

        A small number of latest weave records (e.g., last N).

    Historical, detailed provenance (per snapshot) is pushed into:

        Either its own *_provenance-flow that is append-only, or

        A separate log dataset.

That way:

    You don’t have to bloat the main metadata flow indefinitely.

    But you still have the ability later to do serious forensic “why did snapshot X look like that?” work.

6. Bottom line on your suspicion

Your suspicion is partly right:

    The serious dependency-vs-wall-clock correctness problem only matters if you want strong, global “as-of T” semantics.

    If you mostly want per-flow “latest snapshot before T” plus human-friendly browsing, timestamps are acceptable and the graph-theoretic purism is overkill.

But:

    The thing that really buys you “clean alignment between metadata and reference flows across a weave” is the invariant “same weave ID reused across flows”, not timestamps per se.

    Timestamps just conveniently give you a weave ID that doubles as a human-readable time and folder name.

If you keep that in mind, the timestamp approach is defensible. Just don’t fool yourself into thinking the name alone solves cross-mesh consistency; your metadata flow (or a proper provenance/log flow) is where the real as-of semantics will end up living.
You said:
I see, you're saying if A references B, B advances, and you take an as-of timeshot after A was woven, the world isn't consistent. But A is (hopefully) referencing B@t1 so it doesn't really matter that B@t3 is more recent.

You could actually include _vN after the timestamp to get ordinality and still get sortability. e.g. 20251109181158123_v1
ChatGPT said:

Right, you’ve got the dependency thing basically right, but let me tighten it and then poke at your timestamp_vN idea.
1. Dependency vs wall-clock, clarified

Your restatement:

    If A references B, B advances, and you take an as-of timeshot after A was woven, the world isn't consistent. But A is (hopefully) referencing B@t1 so it doesn't really matter that B@t3 is more recent.

That’s the crux:

    A is internally consistent:
    If A’s content explicitly says “I used B@t1”, then A@t2 is semantically well-defined regardless of any later B@t3.

    The “world” you reconstruct via asOf(T) may not be:
    If your asOf(T) algorithm is “for each flow, pick the latest snapshot ≤ T”, you might reconstruct:

        A@t2 (built with B@t1), and

        B@t3 (built later),
        which is not any actual weave state that ever existed, just a Frankenstein world.

Whether that matters depends on what “as-of” is for:

    If it’s “what did A know at that time?” → follow A’s internal pointer to B@t1; fine.

    If it’s “give me a global state that could have existed at time T” → you need dependency-aware selection (metadata/provenance), not raw timestamps.

So yes, A referencing B@t1 makes A itself fine. The inconsistency only appears if you treat “as-of T across meshes” as “just pick max(timestamp ≤ T) independently for every flow” and pretend that is the world state.

You seem okay with “A is right, and as-of(T) is only approximate per-flow”, which is a reasonable stance. Just don’t sell asOf(T) as something stronger than it is.
2. timestamp_vN as snapshot folder name

    You could actually include _vN after the timestamp to get ordinality and still get sortability. e.g. 20251109181158123_v1

Mechanically, yes, that works:

    Lexicographic sort groups all variants of a given timestamp together:

        20251109181158123_v1

        20251109181158123_v2

    Within that group, _v1, _v2 give you ordinality.

But there are several questions you’re not resolving by just concatenating the two:
2.1. What does _vN actually count?

For a given flow/dataset series, you already have a concept of “release count”:

    v1 = first release, v2 = second, etc.

If your ID is 2025…_v1, then:

    Is _v1 scoped per flow (most likely)?

    Does every snapshot for that flow follow the same timestamp+vN pattern?

    What happens when two releases end up sharing the same timestamp (same ms)?

You now have to define:

    How _vN increments:

        Independently of the timestamp → i.e., …1123_v7, …2044_v8 etc.

    And what the timestamp means if it no longer uniquely identifies a release.

At that point, you’ve just encoded two orthogonal axes into the name:

    “When” (timestamp)

    “Which release number” (vN)

That’s not wrong, but it’s more complexity than you currently admit.
2.2. What problem does _vN in the path actually solve?

If you want ordinality, you can store it in RDF:

    sflo:sequenceNumber 7 on the snapshot dataset.

Then you get:

    Path: 20251109181158123/

    RDF: sequenceNumber 7, generatedAtTime 2025-11-09T18:11:58.123Z

You still have:

    Clear ordinality (sequenceNumber),

    Clear time (generatedAtTime),

    Simple path naming.

Putting _vN into the path only buys you “I can see the ordinal just from the folder name”, which is purely a UX frosting. It doesn’t add semantics you couldn’t express more flexibly in the metadata.

Given how much work you’re already putting into metadata flows, I’m not seeing strong justification for mixing the axes into the path rather than the graph.
2.3. Collision handling still hasn’t gone away

Even with timestamp_vN:

    If you generate two snapshots within the same ms for a given series, you have to choose:

        Either use _vN to distinguish them (so you get 2025…123_v7 and 2025…123_v8), in which case:

            The timestamp is no longer unique.

            And the real disambiguator is _vN.

        Or enforce “one snapshot per ms per series” (sneaky serialization) which is practically okay but needs to be stated.

If _vN is doing the uniqueness work in collisions, you’re halfway back to the original _vN scheme with timestamp just as decoration.
3. Where timestamped IDs are actually pulling their weight for you

The real advantages of timestamp-based snapshot names in your design are:

    Cross-flow alignment within a single weave
    If you consistently:

        Pick a single weaveTimestamp, and

        Use it for all flows touched in that weave,

    then:

        Metadata flow and reference flow from that weave share …/20251109181158123/ as suffix.

        Human debugging “which flows changed together?” becomes easy.

    Time sort at the filesystem level
    You get time-ordered directory listings for free and can eyeball “how long between snapshots” without parsing RDF.

Those are real, concrete UX wins.

Neither of them requires _vN in the path:

    You can align flows by timestamp alone.

    You can have ordinality in RDF.

4. What I’d push you toward

Given your constraints and your tendency to overload path semantics:

    Keep the path as:

<flow>/<snapshotTimestamp>/
<flow>/_working/
<flow>/_default/

Put ordinality into RDF, not into the folder name:

    <…/20251109181158123/> a dcat:Dataset ;
        sflo:snapshotTimestamp "20251109181158123" ;
        sflo:sequenceNumber 7 ;
        prov:generatedAtTime "2025-11-09T18:11:58.123Z"^^xsd:dateTime ;
        …

        snapshotTimestamp = ID and path fragment.

        sequenceNumber = per-series ordinal.

        generatedAtTime = real wall-clock.

    Use a “weave ID” explicitly in metadata flow (which can be the same timestamp):

        For each weave run W (timestamp TW), metadata flow records:

            sflo:weaveId TW

            sflo:producedSnapshot links to all datasets that got that timestamp.

            sflo:usedSnapshot links to the inputs (B@t1, etc.)

That gives you:

    Human-readable folder names (timestamps).

    Per-series ordinality (sequence numbers in RDF).

    Ability to reconstruct dependency-aware “as-of weave W” from metadata.

    No weird composite IDs like 2025…_v7 that encode two axes into one string.

If you still want _vN in the folder name after that, it’s almost certainly because you like the look of it, not because it’s semantically needed. At that point it’s aesthetic, not architectural, and you should be honest with yourself about that.
You said:
To answer your questions: _v1 is scoped per flow, every snapshot for that flow follows the same pattern, and if you can somehow manage to weave twice in the same millisecond, no problem, the second weave gets a vN+1. It's actually a nice argument for reducing the length of the timestamp. You could easily reduce to seconds or even minutes or hours and get much better readability. You could even do days. 
ChatGPT said:

Good, this clarifies your mental model. Let’s stress-test it.

You’re now implicitly proposing something like:

    ID pattern (per flow):
    SNAPSHOT_ID = <timestamp>_v<sequence>
    where:

        <timestamp> can be coarse (seconds/minutes/hours/days).

        _vN is a strictly monotonic per-flow counter (v1, v2, v3…).

        Multiple snapshots can share the same <timestamp>; _vN disambiguates.

I’ll walk through what that actually means and where the traps are.
1. What this scheme really says about time

If you allow multiple snapshots per timestamp bucket:

    The timestamp portion is not a unique weave ID.

    It’s just a time bucket label (“this snapshot happened sometime on 2025-11-09” or “sometime in this hour/minute/second”).

In other words:

    True identity is (flow, vN) (or (flow, timestamp, vN) if you insist).

    Timestamp gives you approximate temporal information, no more.

That’s fine if you accept it, but you need to be explicit:

    If you drop down to days:

        All snapshots woven on the same day share the same <timestamp> part.

        Ordering within that day is entirely by _vN.

    If you drop down to hours:

        Same story at the hour granularity.

So you’re no longer encoding “weave moment”; you’re encoding “weave bucket + ordinal in bucket”.

If that’s really what you want, you should probably call it that in the ontology:

    sflo:snapshotBucket (e.g., "2025-11-09" or "2025-11-09T18")

    sflo:sequenceNumber (integer)

instead of pretending the timestamp part is a precise “snapshotTimestamp” ID.

Right now your previous ticket treated snapshotTimestamp as a unique compact ID (^\d{17}$). This new idea undermines that assumption.
2. Lexicographic ordering & UX: not as clean as it looks

You’re saying:

    “Ordering alone almost makes timestamp snapshot names worth it… we could reduce to seconds/minutes/hours/days; you still get sortability.”

Not automatically.

If you do 20251109_v1, 20251109_v2, etc:

    Lexicographic sort within that bucket will give you:

        20251109_v1

        20251109_v10

        20251109_v2

        …

unless you zero-pad the _vN part (_v0001, _v0002, …). So you’re right back to having to choose an arbitrary width.

You’ve just moved the padding pain from a pure _vN scheme to the suffix of a timestamp_vN scheme.

So:

    If you care about simple “ls” giving sensible order:

        You still need zero padding on _vN, or

        You accept that humans must use tools or sort by metadata, not filename.

Once you admit “you need padding somewhere,” the argument “we avoid an arbitrary width with timestamps” evaporates. You’ve just split the width across two segments instead of one.
3. Cross-flow alignment: you’re drifting from your earlier invariant

Earlier, your ticket said:

    On weave, lock CRUD, pick a weaveTime, and use it for “releasing” all the “working” distributions.

That implies a global weave time per weave run:

    In that model:

        All flows released in that run share the same <timestamp> part.

        Metadata flow and reference flow from that run align by timestamp.

Now you’re saying:

    “_v1 is scoped per flow, every snapshot for that flow follows the same pattern, and if you can somehow manage to weave twice in the same millisecond, no problem, the second weave gets vN+1.”

This is a different invariant:

    Per-flow weaves, each with their own (timestamp, vN) sequence.

    No guarantee that a given <timestamp> is tied to a single weave run that touched multiple flows.

If you really want the cross-flow alignment benefit you liked earlier, you need to preserve:

    A single weaveId per weave operation (the thing that may touch multiple flows).

    All flows touched in that operation share the same weaveId in their metadata, regardless of how you name their folders.

Your current “per-flow _vN and per-flow timestamp bucket” story moves you away from that:

    Two flows can coincidentally share the same <timestamp> but come from different weaves with totally different dependency sets.

    “Same timestamp prefix” no longer reliably means “same weave”.

So:

    Either you re-introduce an explicit weaveId in metadata (which is orthogonal to path),

    Or you’re relying on timestamp buckets for a correlation they no longer actually encode.

4. “Reduce to days / hours / minutes” – what are you giving up?

If you collapse precision:

    To seconds: you still know rough order; collisions are plausible but rare.

    To minutes/hours: many snapshots can share a bucket during active development.

    To days: it’s basically a “daily release” grouping key.

This is fine if your use cases are:

    “Which snapshots happened around that day/hour?”

    “What was the most recent snapshot as of day X?”

It’s not fine if you later want:

    To reconstruct intra-day sequences precisely from filenames.

    To reason about roughly “what was the mesh state at 18:11 vs 18:50” without opening RDF.

Advocating for day/hour-level precision is basically admitting:

    The timestamp in the path is decorative.

    generatedAtTime in RDF is your real time value.

And at that point, honestly, you might as well:

    Put only _vN in the folder name, and

    Reserve timestamps for RDF, not filenames.

5. What actually makes sense, given everything you’ve said

Putting all your constraints together:

    You care about:

        Per-flow ordinality.

        Human-readable rough time.

        Cross-flow alignment per weave (at least for debugging).

    You do not care about:

        Strong “as-of physical instant” semantics across meshes.

        Path names being the sole source of truth for time.

Given that, a more coherent design would be:

    Canonical identity per snapshot (per flow)

        Use a simple numeric sequence per flow:

            Path: <flow>/snapshots/7/

            RDF:

                sflo:sequenceNumber 7 (authoritative “vN”)

                prov:generatedAtTime precise datetime (millis or higher)

                Possibly sflo:snapshotBucket "2025-11-09" if you want day-level grouping.

    Weave-level alignment

        For each weave run, assign a sflo:weaveId (can be a timestamp, or a GUID).

        In metadata flow:

            weaveId → producedSnapshots and usedSnapshots.

    Now “as-of weave W” is defined via metadata, not path conventions.

    If you insist on timestamp in the folder name

    Then at least be honest about semantics:

        Path: <flow>/snapshots/2025-11-09T18_v007/
        Where:

            The date/time part is a bucket (day/hour/minute),

            _vNNN is the true ordinal.

        RDF:

            sflo:sequenceNumber 7

            prov:generatedAtTime "2025-11-09T18:11:58.123Z"

            sflo:snapshotBucket "2025-11-09T18" (if you want).

    And do not use something called snapshotTimestamp with a regex that implies unique 17-digit identity; it’s a bucket label, not a unique key.

6. The uncomfortable conclusion

Your latest idea (“we can drop to seconds/minutes/hours/days, vN handles collisions”) is internally consistent, but:

    It collapses the timestamp from “identity” to “coarse label”.

    The real version identity becomes vN again.

    To preserve alignment semantics, you still need a weave-level identifier in metadata.

    You haven’t actually escaped the need to choose an arbitrary padding width for _vN if you care about lexicographic ordering.

So in practice:

    Either accept that sequenceNumber is primary, and treat timestamps as purely metadata/labels (possibly also in the path if you like how it looks), or

    Commit to full-precision timestamps as the actual ID and deal with the clock/collision semantics there.

Right now you’re edging toward a hybrid that gives you more moving parts without a clear semantic win beyond aesthetics.
You said:
I'm worried that 20251109181158123 is too impenetrable. We're doing this for humans after all. I wouldn't call it purely decorative, but if we assume weave frequency of < 1s is unlikely, folder names of 2025-11-24_0142_v1 (with no padding) is fairly readable, and mostly-sorted (unless you get more than 10 weaves per minute, which is also unlikely). And all nodes and flows woven together at least have the same "to-the-minute" folder name, so you can see what's likely to have been woven together. (More than one weave per minute seems unlikely too, it takes humans a minute to confirm provenance. I'm picturing much-more-slowly changing data... equity trading and twitch gaming aren't going to be using this slow, text-file-based storage system as their parimary storage.

the canonical isn't a timestamp or a version, it's an IRI. 

But I appreciate your critical take on all this.
ChatGPT said:

Your instincts here are mostly right, but you’re mixing three different layers and that’s where it can bite you:

    Human-facing folder names (UX, browsing).

    Semantic identity (which IRI means “this dataset”).

    Provenance/time (what happened when, with what).

Let me separate them and show where your 2025-11-24_0142_v1 scheme fits cleanly and where you’re still being a bit hand-wavey.
1. Readability: yes, the long timestamp is bad

I agree: 20251109181158123 is opaque. For a slow, human-driven system, your proposed format:

    2025-11-24_0142_v1

is much more usable:

    Date is obvious.

    “0142” is clearly 01:42.

    _v1 is legible ordinality.

Given your stated environment:

    “weave frequency < 1/s is unlikely”

    “>10 weaves per minute is unlikely”

    “this is slow, text-file-based, human-confirmed provenance”

it’s reasonable to accept minute-level buckets + a local _vN counter.

But that has to be called what it is: a time bucket plus an ordinal, not a precise “snapshot timestamp”.

Concretely:

    2025-11-24_0142_v1 means: “first released snapshot for this flow in the 01:42 UTC minute of 2025-11-24.”

    You cannot safely interpret the suffix as the exact weave time.

You already implicitly accept that, which is fine. Just don’t design later features assuming the string is a precise, unique instant.
2. “Canonical isn’t a timestamp or version, it’s an IRI” – be precise about which IRI

That sentence is correct but dangerously vague.

You really have at least three levels of IRI:

    Abstract / payload node IRI – “the thing in the world”

        e.g. <…/djradon/bio/> as a payload node.

    Abstract dataset / series IRI – “all snapshots of the dataset about that thing”

        e.g. <…/djradon/bio/_series/_main/dataset> (or whatever pattern you settle on).

    Snapshot dataset IRIs – “this particular released version”

        e.g. <…/djradon/bio/_series/_main/dataset/2025-11-24_0142_v1>.

When you say “canonical is an IRI”, that should mean:

    “Canonical subject is the payload node or abstract dataset IRI. Snapshots are subordinate IRIs.”

If you let the snapshot path become the canonical subject, you’re back to binding semantics to the folder name.

So for this scheme to be coherent:

    Keep the abstract IRI stable and short.

    Let 2025-11-24_0142_v1 live one level deeper, as the snapshot’s IRI or just as a folder mapping to that IRI.

Example layout:

…/djradon/bio/                    # payload node (canonical thing)
  _series/
    _main/
      dataset/                    # abstract dataset IRI
        2025-11-24_0142_v1/       # snapshot dataset IRI (or just folder for its files)
        2025-12-01_1030_v2/
      current/                    # pointer, or resolved via catalog

RDF-wise, you’d have:

# Abstract dataset
<…/djradon/bio/_series/_main/dataset> a dcat:DatasetSeries(?) ;
    sflo:hasSnapshot <…/djradon/bio/_series/_main/dataset/2025-11-24_0142_v1> .

# Snapshot dataset
<…/djradon/bio/_series/_main/dataset/2025-11-24_0142_v1> a dcat:Dataset ;
    sflo:snapshotLabel "2025-11-24_0142_v1" ;
    sflo:sequenceNumber 1 ;
    prov:generatedAtTime "2025-11-24T01:42:33.456Z"^^xsd:dateTime .

Then it’s true in a meaningful way that:

    Canonical identity is the IRI of the abstract dataset (and payload node).

    The 2025-11-24_0142_v1 bit is a label for one of its snapshots, not the essence of the resource.

3. “Mostly-sorted” folder names: good enough, but know what you’re buying

You wrote:

    folder names of 2025-11-24_0142_v1 (with no padding) is fairly readable, and mostly-sorted (unless you get more than 10 weaves per minute…)

Yes, but “mostly-sorted” here is specifically:

    For a given day:

        Snapshots within a day are grouped by minute (prefix).

        Within a minute, lexicographic sort of _v1, _v2, _v10 is not numeric; _v10 comes before _v2.

In practice, I’d summarize it as:

    Sorted by day + minute bucket.

    Randomish within a bucket if you don’t pad.

Given your “weaves per minute are rare” assumption, that’s probably fine:

    If there are only 1–2 snapshots per minute, intra-minute ordering visually doesn’t matter.

    You can always use tooling (RDF generatedAtTime) to get exact ordering if you really care.

Just be explicit in your mental model:

    Path sort gives you “time buckets in order”, not strict “snapshot sequence order”.

    Exact sequence is controlled by RDF (sequenceNumber, generatedAtTime), not filenames.

4. “Flows woven together share the same to-the-minute label” – how true is that?

You said:

    And all nodes and flows woven together at least have the same "to-the-minute" folder name, so you can see what's likely to have been woven together.

This is only true if you preserve one invariant:

    A “weave run” has a single weaveLabel (here, the YYYY-MM-DD_HHMM), and any flow touched in that run uses that label.

If you instead:

    Allow flows to be woven independently,

    Each one computing their own “now” when they release,

then you can easily get:

    Flow A: 2025-11-24_0142_v1

    Flow B: 2025-11-24_0143_v1

even if logically they were part of a conceptual “same change”.

So:

    If you care about “likely woven together”, treat 2025-11-24_0142 as a weave-level label, not “wall-clock at the instant of each flow’s write”.

    That means core needs to pick the label once at the start of the weave and reuse it.

This is independent of resolution. You’ll have the same issue with ms-precision or minute-precision: the question is whether a weave run shares a label.
5. Where this leaves you, practically

Given everything you’ve said, a coherent set of rules could be:

    Human-facing snapshot label

        Format: YYYY-MM-DD_HHMM_vN.

        Resolution: minute is fine for your “slow, human-confirmed” use case.

        Semantics: “in minute YYYY-MM-DD HH:MM, this was release N of this flow.”

    Canonical identity

        Payload node and abstract dataset IRIs are the canonical “things”.

        Snapshot IRIs are derived from the abstract dataset IRI + snapshot label.

    Ordering and “as-of” logic

        Exact ordering: prov:generatedAtTime (millisecond or better) in RDF.

        Ordinality per flow: sflo:sequenceNumber in RDF.

        “Likely woven together”: shared weaveLabel (the YYYY-MM-DD_HHMM part) across flows in the same run, recorded in metadata flow.

    Folder semantics

        Folder names: use weaveLabel + _vN for UX.

        Do not rely on pure lexicographic order for anything important.

        All serious logic (as-of, deriving “current”, debugging) should go through RDF metadata, not filename parsing.

That respects:

    Your concern about impenetrable 17-digit blobs.

    Your desire for human-readable, roughly sortable names.

    Your statement that “canonical isn’t timestamp or version, it’s an IRI.”

The main discipline you need is: treat the path label as UI sugar over RDF, not as the place where semantics live. The “canonical IRI” story only really works if the graph is what defines identity and history; the filenames should be for humans.
You said:
Yes, yes. When I say Canonical I mean for the snapshot dataset. Flows can't be woven indepedently because their weave metatdata lives in the node's metadata. So yes, core needs to pick the label once and reuse it.

Since the actually ordinality will be an integer recorded in RDF metadata (per flow), and we recognize that the snapshot label is just a label, we could pad a zero or even two. But yes, even then, we don't rely on lexicographic order for anything important.
ChatGPT said:

Good, this is finally coherent. Let me crystallize what you’ve actually decided and point out the remaining constraints you need to bake into core so this doesn’t get fuzzy later.
1. Snapshots: what’s canonical, what’s just a label

Your clarified model:

    Canonical identity (for a snapshot dataset) = its IRI, not the folder label pieces.

    Each snapshot has:

        A weave label, shared across everything woven in that run.

        A per-flow ordinal, stored as an integer in RDF.

    The filesystem folder name is just a human-facing label, e.g.:

        2025-11-24_0142_v01

        2025-11-24_0142_v02

So for a given series:

    RDF:

    # abstract dataset for the series
    <.../_series/_main/dataset> a dcat:Dataset .

    # snapshots
    <.../_series/_main/dataset/2025-11-24_0142_v01> a dcat:Dataset ;
        sflo:weaveLabel "2025-11-24_0142" ;
        sflo:sequenceNumber 1 ;
        prov:generatedAtTime "2025-11-24T01:42:33.456Z"^^xsd:dateTime .

    <.../_series/_main/dataset/2025-11-24_0142_v02> a dcat:Dataset ;
        sflo:weaveLabel "2025-11-24_0142" ;
        sflo:sequenceNumber 2 ;
        prov:generatedAtTime "2025-11-24T01:42:51.100Z"^^xsd:dateTime .

Path follows from the snapshot IRI, not the other way around.

That keeps your “canonical = IRI” statement actually true.
2. Weave label: single choice, reused everywhere

You said:

    Flows can't be woven independently because their weave metadata lives in the node's metadata. So yes, core needs to pick the label once and reuse it.

Concretely, that implies:

    A weave run is the unit of work for core.

    At the start of a weave run, core:

        Picks one weaveLabel (e.g., 2025-11-24_0142 in UTC).

    Every snapshot created in that run:

        Uses that weaveLabel in its folder name.

        Records the same label in metadata (sflo:weaveLabel).

So, for a weave that touches:

    node A’s reference-flow

    node A’s metadata-flow

    node B’s payload-flow

all of those snapshots share:

    weaveLabel = "2025-11-24_0142"

That’s how “things woven together look like they belong together” is guaranteed, rather than just being a clock coincidence.

You need to treat weaveLabel as a weave-run identifier, not as “whatever the clock says when each flow finishes.”
3. Ordinality: integer in RDF, padding in paths is cosmetic

You also clarified:

    Since the actual ordinality will be an integer recorded in RDF metadata (per flow), and we recognize that the snapshot label is just a label, we could pad a zero or even two. But yes, even then, we don't rely on lexicographic order for anything important.

That’s the right way to think about it:

    Authoritative ordinal: sflo:sequenceNumber (integer, per series).

    Path suffix: _v01, _v02, … purely for humans.

So:

    If you want nicer directory listings, pick a padding width (e.g. 2 or 3 digits) and be done with it.

    Any real logic (previous version, N versions back, etc.) should use sequenceNumber, not string-munging _vNN.

The important part: do not leak “vN is authoritative” back into semantics later. It’s metadata in the graph.
4. Resolution & “slow data” assumptions

Given your environment:

    Human-confirmed provenance,

    Slow-changing text-file data,

    Weaves-per-minute << 10,

minute-level weaveLabel is acceptable:

    YYYY-MM-DD_HHMM gives you:

        Coarse “when” at a glance.

        Good-enough grouping for “likely woven together.”

    Precise time still lives in prov:generatedAtTime with ms (or higher) precision.

You just need to accept explicitly:

    weaveLabel is a bucket key, not a precise instant.

    If you ever crank up weave throughput in the future, you might need to tighten resolution or treat weaveLabel as purely cosmetic and rely more heavily on generatedAtTime + sequenceNumber.

5. The remaining discipline you need

If you stick with this scheme, core needs to enforce:

    Single weave label per run

        Chosen once at weave start (UTC).

        Reused across all snapshots created in that run.

    Per-series monotonic sequenceNumber

        Integer, incremented per snapshot for that series.

        No reuse, no gaps re-filled.

    RDF as source of truth

        Any “as-of” logic, previous-version logic, and debugging should:

            Prefer sequenceNumber and generatedAtTime from RDF,

            Treat path labels (YYYY-MM-DD_HHMM_vNN) as hints for humans, not semantics.

If you actually hold to those three, your 2025-11-24_0142_v01 scheme is a reasonable human-oriented compromise and meshes with “canonical = IRI” in a clean way.
You said:
It's tempting to try to make weave labels unique. Maybe weaveBucket is better. Or you could add a letter for subsequent (within a minute) weaves to give you up to 27 unique weave labels per minute. eg. 2025-11-24_0142a for the 2nd weave of the minute.
ChatGPT said:

Right now you’re trying to get a single label to do three jobs at once:

    “Time-ish” (when the weave happened).

    “Run-ish” (which weave run it was).

    “Human-readable-ish” (so you can eyeball folders).

That’s why this is feeling slippery.

Let me unpack the “unique vs bucket” decision and then look at the 2025-11-24_0142a idea.
1. Do you actually need weave labels to be unique?

You already have:

    Per-flow ordinals (sequenceNumber in RDF).

    Exact times (prov:generatedAtTime).

    A clear notion of a weave run (core picks one label and uses it for everything in that run).

If you add a weaveRunId in metadata (even just a full ISO timestamp or UUID):

    That is the canonical identifier of the run.

    The folder label becomes pure UX: “roughly when”.

In that model, weaveBucket is enough:

    weaveBucket = "2025-11-24_0142"

    Maybe many runs share that bucket.

    “Which exact run?” → look at weaveRunId in metadata.

That’s the cleanest separation:

    Bucket = human grouping.

    RunId = canonical, maybe opaque.

    Folder name = bucket + optional fluff, never used for real semantics.

Making the label itself unique is only needed if you insist on using it as the run identifier rather than just a bucket.
2. If you do want unique weave labels: what your …0142a idea really does

You’re proposing:

    First weave in the minute: 2025-11-24_0142 (or _0142a as first, you didn’t fully say).

    Second weave in that minute: 2025-11-24_0142a

    Third: 2025-11-24_0142b

    … up to z → about 27 labels per minute.

This is effectively:

    weaveBucket = "2025-11-24_0142"

    weaveOrdinalWithinBucket = 0, 1, 2, …

    weaveLabel = weaveBucket + suffix(weaveOrdinalWithinBucket).

So:

    Time = bucket.

    Uniqueness = suffix.

    Label = combined.

That’s internally consistent, but you need to be explicit about rules:

    Generation

        Core tracks weaveOrdinalWithinBucket per bucket.

        On each new weave in the same minute:

            If ordinal = 0: label = "2025-11-24_0142" (no suffix) or "…_0142a" (if you want uniformity).

            If ordinal > 0: label = "…_0142" + suffix(ordinal).

    Suffix space

        If you cap at 26 letters, you’ve just created a hard limit: >26 weaves in a minute = fail.

        That’s “unlikely” in your current mental model, but you’re baking in a silent assumption about future usage.

    Semantics

        The label says “this is a weave run in bucket 2025-11-24_0142, ordinal X”.

        You still need RDF to say what flows were involved, what exact time, etc.

So yes: it works.
But it’s not simpler conceptually than:

    weaveBucket = "2025-11-24_0142"

    weaveRunId = "2025-11-24T01:42:33.456Z" (or a UUID)

    Path label = weaveBucket (optionally plus an ordinal if you want).

The letter suffix is just embedding weaveOrdinalWithinBucket in the path instead of in RDF.
3. The real choice: bucket + opaque ID vs. “clever label”

You have two sane options:
Option A: Bucket + opaque run ID (recommended)

    Folder label:
    2025-11-24_0142 (maybe always just that; no a/b/c).

    RDF per weave run:

        sflo:weaveBucket "2025-11-24_0142".

        sflo:weaveRunId "2025-11-24T01:42:33.456Z" (or a UUID).

    Each snapshot’s metadata:

        Links to weaveRunId, weaveBucket, sequenceNumber, generatedAtTime.

Properties:

    Folder names stay clean and human.

    Multiple runs in the same minute share a bucket; that’s fine:

        “These happened around 0142” is all the label claims.

    If you need to distinguish runs, you use weaveRunId in metadata, not encoded into the folder name.

This respects your “humans first” goal without overloading the label.
Option B: Make label itself unique using suffix letters

    Folder label:
    2025-11-24_0142, 2025-11-24_0142a, …0142b, etc.

    Internally:

        weaveBucket = "2025-11-24_0142".

        weaveOrdinalWithinBucket = n.

        weaveRunId can still exist in RDF but is isomorphic to (bucket, ordinal).

Properties:

    Humans can see “0142”, “0142a”, “0142b” and infer “same minute, different runs.”

    You take on:

        A fixed “weaves per bucket” limit (26 if you stick to letters),

        Slightly more complex label generation logic.

Given your stated low frequency, it’s probably fine, but it’s still a constraint.
4. Given your own priorities, what actually aligns?

You’ve said:

    “We’re doing this for humans.”

    “Data changes slowly; weaves-per-minute are low.”

    “Canonical identity is the IRI, not timestamp or version.”

    “We don’t rely on lexicographic order for anything important.”

Under those constraints, the simplest, least fragile design is:

    Weave label = bucket only, not forced unique.

        2025-11-24_0142

    Weave run identity = opaque, stored in RDF.

        e.g., sflo:weaveRunId "2025-11-24T01:42:33.456Z" or a UUID.

    Per-flow ordinals = integers (sflo:sequenceNumber).

    Folder names = weaveLabel + "_vNN" purely for humans.

The …0142a scheme is clever, but its only advantage is letting humans distinguish multiple runs in one minute from folder names alone. You pay for that with:

    A fixed cap on runs per bucket.

    Extra rules you must document and enforce in core.

    An implicit assumption that “more often than 26 weaves/minute will never happen”.

If you’re honest about not using folder names for anything critical, that extra complexity is almost certainly not worth it.

So, bluntly:

    “WeaveBucket” is the right concept name.

    You should stop trying to make the bucket label do all the ID work.

    If you want humans to see multiple runs in a bucket, suffix letters are acceptable but strictly cosmetic. Treat them that way, and don’t lean on them for semantics.

You said:
Sorry, I didn't intend to drop the _vN. So it'd be e.g. 2025-11-24_0142_v3 followed by 2025-11-24_0142a_v4
ChatGPT said:

Got it. So the concrete proposal is now:

    One weave bucket per minute: 2025-11-24_0142

    Zero or more bucket suffixes: '', 'a', 'b', … for multiple weaves in that same minute

    Per-flow ordinal: _v3, _v4, …, with the real ordinality stored in RDF

So snapshot folder names look like:

    First weave in that minute touching this flow: 2025-11-24_0142_v3

    Second weave in that minute: 2025-11-24_0142a_v4

    Third: 2025-11-24_0142b_v5

    etc.

I’ll evaluate this as-is.
1. Semantics implied by this scheme

You’ve now encoded three things into the label:

    Bucketed time: 2025-11-24_0142 (minute resolution)

    Weave ordinal within that bucket: implicit in the suffix '', a, b, …

    Per-flow release ordinal: _v3, _v4, etc. (but authoritative in RDF)

This is workable if and only if:

    bucket and suffix are global to the weave run (not per-flow).

    _vN is per series.

So for weave run W (say “second weave of 01:42”):

    weaveBucket = "2025-11-24_0142"

    weaveSuffix = "a" (global for that run)

    Every flow touched by W gets a snapshot with that same bucket+suffix prefix, and its own _vN.

Example:

    Flow A: 2025-11-24_0142a_v3

    Flow B: 2025-11-24_0142a_v7

    Metadata flow: 2025-11-24_0142a_v42

That’s coherent: you can see “these all belong to the same weave run at roughly 01:42”.

If you ever let different flows in the same run pick different suffixes, you lose that benefit; so core has to manage weaveSuffix centrally, per run.
2. Does this actually solve your problems?
Readability

    2025-11-24_0142_v3 is clearly “Nov 24, ~01:42, version 3 for this flow.”

    2025-11-24_0142a_v4 is “another weave around 01:42, next release for this flow.”

This is vastly more legible than 20251109181158123, and acceptable for slow, human-driven weaves. So on the “for humans” axis: yes, this is a real improvement.
Cross-flow alignment

Because bucket+suffix is shared per weave run:

    You can spot “what was likely woven together” by scanning directory names.

    That’s the same property you wanted from timestamp-only IDs, but now with better readability.

So yes, this preserves the key benefit from earlier: a single weave label per run, visible in folder names, shared across flows.
Ordinality

Since you’ve already committed to sequenceNumber in RDF:

    _v3 / _v4 suffixes are mostly redundancy for humans.

    You can pad them or not; lexicographic order is no longer important for semantics.

So this doesn’t hurt anything, and it gives a nice “release count” at a glance.
3. Where the complexity actually lives now

You are quietly accepting:

    Weave suffix allocation logic

        Per bucket (per minute), core must track how many weaves have already been done:

            1st → '' suffix (or _vX without a letter if you want that convention)

            2nd → 'a', 3rd → 'b', etc.

        This is a global counter per bucket, not per flow.

    An upper bound baked into the alphabet

        If you use a–z:

            You have at most 27 runs per minute (including the “no suffix” first run).

        You’re asserting that this will never be hit.

        That’s probably true today, but it’s a hard constraint you’ll forget about in 3 years when someone writes an automated process.

    Label ≠ identity

        The label is a composite of:

            weaveBucket (minute)

            weaveOrdinalWithinBucket (suffix)

            sequenceNumber (vN)

        The canonical identity must still be:

            Snapshot dataset IRI

            RDF fields:

                sflo:weaveBucket

                sflo:weaveSuffix (or sflo:weaveOrdinalWithinBucket)

                sflo:sequenceNumber

                prov:generatedAtTime

If you ever let logic depend on parsing 2025-11-24_0142a_v4 instead of reading RDF, you’ll eventually regret it. You already know that, but you should treat it as a hard rule in core.
4. Is the suffix worth it, given your own constraints?

The only benefit of adding the letter suffix on top of _vN is:

    You can distinguish multiple weave runs in the same minute from the folder name alone.

Everything else (time, run identity, exact sequencing) is better carried in RDF.

Given your stated reality:

    “weave frequency < 1/s is unlikely”

    “more than one weave per minute seems unlikely too”

    “data changes slowly, humans confirm provenance”

you’re optimizing for a corner case that:

    Almost never happens in your actual expected workload.

    Can already be diagnosed from RDF (weaveRunId, generatedAtTime) if it ever does.

So the trade is:

    Keep folder names simpler and document that:

        bucket = YYYY-MM-DD_HHMM

        multiple weaves in the same bucket are distinguished only by metadata

    Or:

        Introduce per-bucket suffix logic, alphabet limits, and another moving part, in exchange for slightly better folder-only visibility in rare multi-weave minutes.

Given everything you’ve said about:

    “canonical is IRI”

    “we don’t rely on lexicographic order”

    “metadata is where provenance lives”

the suffix reads like an over-optimization.
5. Recommendation

If you want a design that’s internally consistent and doesn’t age badly:

    Use bucket-only weave labels for folders

        Snapshot folder: 2025-11-24_0142_v03

        No a/b/c suffix.

        One bucket label per weave run (per your “flows can’t be woven independently” rule).

        If you ever do two runs in the same minute, they share the same bucket label; you distinguish them via metadata only.

    Carry the real semantics in RDF

        For each snapshot:

            sflo:weaveBucket "2025-11-24_0142"

            sflo:weaveRunId "2025-11-24T01:42:33.456Z" (or UUID)

            sflo:sequenceNumber 3

            prov:generatedAtTime "2025-11-24T01:42:33.456Z"^^xsd:dateTime

    If you absolutely insist on suffixes, treat them as pure decoration:

        Define sflo:weaveOrdinalWithinBucket in RDF.

        Guarantee that all logic uses that + weaveRunId, not parsed letters.

        Accept the 27-runs-per-minute constraint as a real spec limit.

Right now, your core idea—minute-level YYYY-MM-DD_HHMM + _vN + RDF ordinals—is solid. The extra a/b suffix doesn’t add semantic capability you can’t get from RDF; it only complicates label generation for a workload you’ve already said will be sparse.
You said:
If you just keep adding letters, you get infinite runs per minute. No padding necessary to maintain lex order. e.g. 2025-11-24_0142z_v31 -> 2025-11-24_0142za_v32

I'm not sure anything must track. We can't rely on the same process being alive the whole minute, so we'll either need to keep track in metadata, or just compute it... Is there already a metadata weaveBucket that starts with this minute? If so, bump the suffix. Is there already one? you could accomplish it with sparql sorting. Or a prevVersion rdf check.
ChatGPT said:

Right now you’re trying to get weave labels to be:

    Human-readable,

    Lexicographically nice,

    And unique per weave run,

all encoded into a single string. The “infinite letters” trick fixes my earlier 27-per-minute bound, but it does not fix the deeper problems: you’ve just moved state tracking into RDF+SPARQL and you still haven’t handled races.

I’ll break it down.
1. What your scheme actually is now

Given a minute 2025-11-24_0142, your full label is:

    2025-11-24_0142 – first weave in that minute

    2025-11-24_0142a – second weave

    2025-11-24_0142b – third weave

    ...

    2025-11-24_0142z

    2025-11-24_0142za

    2025-11-24_0142zb

    ...

and the snapshot folder per flow is something like:

    2025-11-24_0142_v31

    2025-11-24_0142a_v32

    2025-11-24_0142za_v33

    etc.

Internally:

    weaveMinute = "2025-11-24_0142"

    weaveSuffix = "", "a", "b", ..., "z", "za", ... (infinite base-26 over [a–z])

    Combined weaveLabel = weaveMinute + weaveSuffix

    Ordinality per flow in RDF: sflo:sequenceNumber

Property: lexicographic order of weaveLabel matches “base-26 increment” order if you define increment correctly. So far, fine.
2. “We don’t track, we compute next suffix via SPARQL”: you’re still tracking

Your algorithm idea is implicitly:

    Compute weaveMinute = minute(now), e.g. "2025-11-24_0142".

    Query metadata for existing labels with that minute prefix:

        SELECT ?label WHERE { ... sflo:weaveLabel ?label . FILTER(STRSTARTS(?label, "2025-11-24_0142")) }

    If none:

        weaveSuffix = "", label = "2025-11-24_0142".

    Else:

        Take max label lexicographically, say "2025-11-24_0142z".

        Increment its suffix (z → za) to get next label: "2025-11-24_0142za".

You’re right: that means no in-memory counter. But you are tracking state:

    The RDF store is the state store.

    The SPARQL query is your “what’s the current max?” operation.

    The string increment logic bakes in a global invariant about how you interpret these labels.

That’s not automatically bad, but it’s a global mutable counter encoded in the graph, not “no tracking”.
3. You still haven’t handled races

Consider two weave runs in the same minute, launched concurrently against the same mesh:

    Both compute weaveMinute = "2025-11-24_0142".

    Both run the SPARQL query.

    Both see current max label "2025-11-24_0142z".

    Both compute next label "2025-11-24_0142za".

Now what?

There are only three possible outcomes:

    Both succeed writing "2025-11-24_0142za" as weaveLabel

        Now you have two distinct weave runs with the same label.

        All the effort to make labels unique via suffixes is gone; you’re back to bucket semantics.

    Your storage layer rejects the second write (some constraint, file/dir conflict, SHACL, whatever)

        Then you need a retry strategy:

            Re-run SPARQL, get "2025-11-24_0142za" as max, increment to "2025-11-24_0142zb", rewrite.

        That’s a CAS/optimistic concurrency loop. You’ve re-introduced a lock-like mechanism, just a janky one built out of SPARQL + error handling.

    Git-style merge conflict

        If the mesh is Git-backed and these two runs are in different clones:

            They both commit "2025-11-24_0142za" labels locally.

            On merge, you get conflicts in the metadata flow and/or directories.

            Someone has to resolve manually or via custom tooling.

In all three cases, you have to define behaviour. Merely saying “we can compute the suffix with SPARQL” doesn’t solve concurrency; it just hides it.
4. Cross-process, cross-host reality

You said:

    We can't rely on the same process being alive the whole minute

Fine. But also:

    You can’t rely on “only one process ever weaving this mesh”.

    You can’t rely on “only one host”.

    You want the system to behave sensibly if:

        A local CLI does weave.

        A remote sflo-host does weave.

        CI does weave.

All potentially within the same minute.

So any scheme that tries to make path labels globally unique without a coordination primitive (lock, CAS, transactional store) is lying to itself.

Your SPARQL max+increment is already an attempt at coordination; it just lacks any atomicity guarantee.
5. What’s the actual value of uniqueness at the label level?

You’re going through all of this to guarantee:

    “No two weave runs in the same minute share the same weaveLabel.”

Ask yourself what you actually need:

    Per-run identity

        You already get this if you store weaveRunId in RDF:

            A UUID

            Or a full ISO timestamp with ms (or ns) resolution.

        That’s trivially unique without fancy path schemes.

    Human grouping

        YYYY-MM-DD_HHMM already groups things into minute buckets.

        Humans can see “these are roughly the same time” just from that.

    Human disambiguation between runs in a bucket

        This is the one thing the suffix letters give you:

            “_0142” vs _0142a vs _0142za clearly signal multiple runs that minute.

Is that worth:

    A graph-level “max + increment” algorithm,

    Edge-case race semantics,

    And yet another convention you have to document and implement everywhere?

Maybe. But be clear: this suffix does not carry any semantics you can’t get from RDF; it is purely an affordance for eyeballing folders.
6. If you insist on suffixes, make the logic explicit

If you go ahead with the infinite-suffix scheme, you need to commit to:

    Suffix increment function

        Represent suffix as base-26 over [a–z]:

            "" → "a"

            "a" → "b"

            …

            "z" → "za"

            "za" → "zb"

            "zz" → "zza"

        Define that formally and implement it consistently (core, CLI, anything that needs to compute “next”).

    SPARQL-based allocation protocol with optimistic retry

    Pseudocode:

    function allocateWeaveLabel(mesh, minute):
      base = minuteLabel(UTC now)  # "2025-11-24_0142"

      while true:
        labels = SELECT ?label WHERE {
          ?w sflo:weaveLabel ?label .
          FILTER(STRSTARTS(?label, base))
        }
        if labels empty:
          candidate = base
        else:
          maxLabel = max(labels)
          suffix = maxLabel.removePrefix(base)
          candidate = base + incrementSuffix(suffix)
        
        try:
          write new weave metadata with weaveLabel = candidate
          commit/flush
          return candidate
        catch ConflictError:
          # another process took candidate; retry
          continue

    That’s a lock-free CAS loop. It’s acceptable, but you have to recognize it for what it is: stateful allocation, not “no tracking”.

    RDF still carries real identity

    Every weave run must still record:

        sflo:weaveRunId – globally unique

        sflo:weaveLabel – your 2025-11-24_0142za

        sflo:weaveMinute – "2025-11-24_0142" if you want it separated

        prov:generatedAtTime – exact time

        etc.

    And all real logic (repro, as-of selection, provenance) should use weaveRunId and generatedAtTime, not the label.

At that point, suffixes are well-defined sugar, not sneaky semantics.
7. The simpler alternative you keep skating past

Given your own constraints:

    Slow, human-mediated weaves.

    Canonical identity is the IRI + RDF, not the folder name.

    You already plan to store:

        sequenceNumber

        prov:generatedAtTime

        and weave metadata.

You can very sensibly do:

    Folder label: YYYY-MM-DD_HHMM_vN without any suffix.

    Bucket semantics only:

        Multiple weaves in the same minute share the same YYYY-MM-DD_HHMM component.

    Distinguish runs by:

        weaveRunId (UUID) in RDF,

        or even by the set of snapshots they produced.

Then:

    There is no allocation logic for suffixes.

    No races in labels.

    Humans still see:

        “These vN’s happened around 2025-11-24 at 01:42.”

If, once a year, you get two runs in the same minute and you want to know which is which, you open RDF or use a tiny tool. That’s it.
Bottom line

Your infinite-letter suffix trick is technically workable if you:

    Accept you’re doing CAS over RDF (via SPARQL + retry),

    Define the increment and race behaviour precisely,

    And still treat RDF (weaveRunId, generatedAtTime) as the source of truth.

But the suffix is pure UX. It doesn’t provide any semantic guarantee that RDF can’t provide more cleanly.

Given your “canonical is IRI + metadata” stance and your slow-weave assumptions, the simpler, more honest design is:

    weaveLabel = YYYY-MM-DD_HHMM

    folder = weaveLabel + "_vN"

    weaveRunId + generatedAtTime in RDF for anything that needs real identity.

If you still want suffix letters after that, it’s because you like how they look, not because they solve a real architectural problem.
You said:
I do agree we could use some mechanism to indicate a "soft lock". Like on weave, write a dotfile (.weave-lock) into each mesh (or sub-mesh) root node. But regardless, only one process should be responsible for weaving a given mesh (or at least sub-mesh) at a time. 

Git-wise it's okay for conflicts... those will have to be sorted/aborted.

Yes, the only benefit of adding the suffixes is to be able to identify weave runs. Ultimately, people can choose their own weave identifiers, as long as metadata is discoverable. That really underscores the importance of "_default" - and suggests default should reference  either the previous metadata snapshot (and all snapshots reference their previous), or every previous snapshot.
ChatGPT said:

You’re converging on something workable, but you’re muddling three different concerns again:

    Locking / “who is allowed to weave”

    Weave identifiers (human vs canonical)

    What _default actually means

I’ll push on each.
1. “Soft lock” with .weave-lock: what it really buys you

You said:

    Like on weave, write a dotfile (.weave-lock) into each mesh (or sub-mesh) root node. But regardless, only one process should be responsible for weaving a given mesh (or at least sub-mesh) at a time.

A .weave-lock can work as an advisory lock, but only if you’re clear about:

    Semantics

        “If .weave-lock exists, other weavers SHOULD NOT start.”

        If they do, behavior is undefined (possible conflicts, stomping, etc.).

    Atomicity

        Creation: you must create the lock atomically:

            E.g. create .weave-lock.tmp then rename → .weave-lock.

        That’s your “test-and-set”.

    Stale lock handling

        What happens when a process dies mid-weave?

            Lock is left behind.

        You need a policy:

            Store PID + timestamp + host in the lock file.

            Allow override after N minutes with a --force flag.

        Without that, a crash leaves the mesh “wedged” until a human deletes the file.

    Scope

        “mesh (or sub-mesh) root node” needs to be precisely defined:

            If you can weave sub-mesh A and sub-mesh B independently, you need lock files at each sub-mesh root, not a single global one.

        Otherwise you serialize unrelated work unnecessarily.

Even with a lock file, you still have:

    Git-level conflicts when different clones weave and push.

    You’re explicitly OK with that, but be aware: the lock is then per working copy, not global. It does nothing to prevent concurrent weaves in different clones.

Conclusion: .weave-lock is fine as local hygiene, not a global coordination mechanism. Don’t treat it as more than that.
2. Given “only one process per mesh”, the suffix complexity is overkill

If you actually expect:

    One active weaver per mesh/sub-mesh in practice, and

    Weaves per minute to be low,

then:

    The complex suffix allocation (_0142, _0142a, _0142za, …) is solving a problem you are also solving by policy (“only one weaver”).

In that world:

    A simple YYYY-MM-DD_HHMM_vN label is enough.

    The only time you’d get same-minute collisions is:

        If you run two weaves back-to-back manually, or

        If you have automation cranking at sub-minute intervals.

Both are inherently “someone did something unusual” situations; RDF metadata can disambiguate them when needed.

Trying to guarantee weave-run uniqueness at the path level and via a soft lock and via RDF is triple coverage. One of those is redundant. The suffix is the least robust of the three.
3. “People can choose their own weave identifiers” – be careful

You said:

    Ultimately, people can choose their own weave identifiers, as long as metadata is discoverable.

That’s dangerous if you don’t demarcate roles:

    There must be a canonical machine identifier that:

        Is unique (within a mesh, at least),

        Respects a known character set and pattern,

        Can be queried reliably (sh:pattern, etc.).

If you let people arbitrarily set the label that appears in paths, you’ll get:

    Weird characters,

    Collisions,

    And tools that have to escape or special-case everything.

The sane separation:

    Machine ID (canonical)

        sflo:weaveRunId (UUID or ISO8601 with ms/ns).

        Generated by core, not user.

    Human label

        sflo:weaveLabel (your 2025-11-24_0142 or 2025-11-24_0142a).

        Can be user-overridden in metadata if they really want prettier labels.

        Used for folder names, but only as decoration.

    Discovery

        All tools rely on weaveRunId + prov:generatedAtTime for logic.

        weaveLabel is optional sugar.

If you conflate “weave identifier” with “path label” and let users redefine it, you break invariants everywhere.
4. _default: what it should and should not be

You said:

    That really underscores the importance of "_default" - and suggests default should reference either the previous metadata snapshot (and all snapshots reference their previous), or every previous snapshot.

You’re drifting into making _default both:

    “latest materialization” and

    “historical index/log”.

That’s a mistake. Distinguish:
4.1. _default as view, not history

_default should be:

    “The dataset you get when you dereference the node/flow with no version parameters.”

    It represents current state only.

It can contain pointers to historical things, but it should not itself be “the log”. Otherwise:

    Every weave keeps growing _default with more and more history.

    Git diffs get noisy.

    Consumers have to read and filter a huge blob when they only care about “now”.

So I’d make _default:

    A thin dataset with:

        Pointers to:

            current snapshot dataset for this series,

            maybe a small local chain (previous-only) if you really want.

4.2. Where to put history/links

You proposed two variants:

    “Default references previous metadata snapshot”

    That implies either:

        _default has something like:

    :currentMetadataSnapshot sflo:previousSnapshot :metadataSnapshot_2025-11-23_1842_v2 .

    Or _default is itself the metadata snapshot, with a previous pointer.

Better pattern:

    Each snapshot dataset has:

        :thisSnapshot sflo:previousSnapshot :priorSnapshot .

        _default just points to :thisSnapshot as the current one.

    That gives you a linked list of snapshots without bloating _default.

    “Default references every previous snapshot”

    This is not scalable:

        N weaves → O(N) pointers in _default.

        You re-write all of them every time; Git history becomes obnoxious.

        SHACL and clients have to process a big list for no real reason.

If you need “all previous snapshots” as a set, better to:

    Put that in a dedicated index dataset (e.g., _history-index or a provenance flow), or

    Derive it dynamically using a query that walks sflo:previousSnapshot.

So yes, _default is important, but it should remain:

    A single stable entry point to “current”, not a carrier of the entire history.

5. A more disciplined shape

Given everything you’ve said, a coherent pattern looks like:

    For each flow / node:

        Abstract dataset IRI: <…/_series/_main/dataset>

        Snapshots: <…/_series/_main/dataset/<weaveLabel>_vN>

        _default/ contains a dataset that says:

    # in _default
    <…/_series/_main/dataset> sflo:currentSnapshot <…/_series/_main/dataset/2025-11-24_0142_v3> .

Each snapshot dataset includes:

    <…/_series/_main/dataset/2025-11-24_0142_v3> a dcat:Dataset ;
        sflo:weaveRunId <urn:uuid:...> ;
        sflo:weaveLabel "2025-11-24_0142" ;
        sflo:sequenceNumber 3 ;
        prov:generatedAtTime "2025-11-24T01:42:33.456Z"^^xsd:dateTime ;
        sflo:previousSnapshot <…/_series/_main/dataset/2025-11-18_0930_v2> .

    (Optional previousSnapshot if you want a chain.)

    Metadata flow:

        Records weave-run–level info, including which snapshots were produced and their dependencies.

        Can be summarized in its own _default as “latest weave”, not every weave ever.

    Locking:

        .weave-lock at mesh/sub-mesh root:

            PID/host/timestamp inside.

            Optional --force to break stale locks.

        Advisory only; Git still handles cross-clone conflicts.

And then:

    You can keep your YYYY-MM-DD_HHMM_vN labels.

    You don’t need suffix gymnastics unless you really want label-unique runs per minute.

    _default stays lean and meaningful as “current pointer,” not “dumping ground for history.”

If you still want suffixes to visually distinguish same-minute runs, fine, but that’s cosmetic. The important part is that _default remains a view, and history is expressed via snapshot-level metadata and/or a dedicated provenance/index flow, not crammed into _default.
You said:
Regarding "the log", I'm only talking about metadata flows. _default's content should always be identical to the latest version. In general, they don't contain complete history. And to keep them growing too much, I think we can say we've decided that they only contain the latest metadata but reference the previous version.
ChatGPT said:

Good, that’s a lot cleaner than some of the earlier “everything in _default” ideas, but there are a couple of places where your wording is slightly self-contradictory and you should nail them down.

You said:

    _default's content should always be identical to the latest version.
    In general, they don't contain complete history.
    … they only contain the latest metadata but reference the previous version.

Those three sentences only fit together if you’re very precise about where the “reference to the previous version” lives.

Let me spell out a consistent pattern and then point out the edge cases you need to think about.
1. Make _default literally just “current snapshot”

For metadata flows:

    There is a series of metadata snapshots:

        M1, M2, …, Mn (each a dcat:Dataset or sflo:MetadataSnapshot).

    _default is simply “whatever Mn is right now”.

Concretely:

    Filesystem:

/node/_metadata-flow/
  _default/          # materialization of Mn
  2025-11-24_0142_v1/
  2025-11-29_1010_v2/
  ...

RDF (inside the snapshots themselves):

    # Snapshot n
    <.../_metadata-flow/2025-11-29_1010_v2> a sflo:MetadataSnapshot ;
        sflo:sequenceNumber 2 ;
        sflo:previousSnapshot <.../_metadata-flow/2025-11-24_0142_v1> ;
        prov:generatedAtTime "2025-11-29T10:10:00Z"^^xsd:dateTime ;
        ... latest metadata content ... .

Then:

    _default/ just contains an identical dataset to 2025-11-29_1010_v2/ (maybe literally the same file copied/symlinked).

    The reference to the previous snapshot (sflo:previousSnapshot) lives in the snapshot dataset itself, not in “some extra graph only in _default”.

That way, your sentence:

    "_default's content should always be identical to the latest version"

remains true, because:

    Whatever RDF exists under _default is exactly the RDF that exists under the current snapshot folder.

If you were imagining:

    “_default has the latest metadata and also some extra pointer that only lives in _default” – then _default is no longer identical to the latest version. That’s the inconsistency you’d be introducing without noticing.

2. The “log” lives in the chain, not in _default

With the above pattern:

    History retrieval works by walking the chain:

        Start from currentSnapshot (what _default mirrors).

        Follow sflo:previousSnapshot backwards.

That gives you:

    O(1) update cost per weave:

        Create new snapshot dataset.

        Set its sflo:previousSnapshot to the prior one.

        Make _default mirror the new snapshot.

    No growth of _default independent of snapshot size.

    All history semantics expressed at the snapshot level, not crammed into _default.

This matches what you said you want:

    “In general, they don't contain complete history.
    … only contain the latest metadata but reference the previous version.”

Correct interpretation:

    “They only contain the latest metadata, and that latest snapshot object is the one that references the previous snapshot.”

Wrong interpretation (don’t do this):

    “_default contains the latest metadata and also an extra pointer that isn’t in the snapshot dataset.”

3. How this interacts with weave IDs / labels

For metadata flows in particular, you now have a nice, clean shape:

    Each weave run W:

        Produces a metadata snapshot M_n with:

            sflo:weaveRunId

            sflo:weaveLabel (your YYYY-MM-DD_HHMM…)

            sflo:sequenceNumber

            sflo:previousSnapshot M_{n-1}

    _default for the metadata flow:

        Mirrors M_n exactly.

So, to reconstruct:

    “What did the metadata look like at weave run W?”

        Use weaveRunId or weaveLabel to find the right snapshot(s).

    “What was the immediate previous metadata?”

        Follow sflo:previousSnapshot exactly once.

    Past history:

        Keep following sflo:previousSnapshot until you hit the genesis.

You don’t need (and shouldn’t try) to have _default reference “every previous snapshot”; the chain is enough.
4. Edge cases you should decide on now

    Snapshot deletion / compaction

    If you ever compact history (e.g., delete very old metadata snapshots), you must:

        Either rewire sflo:previousSnapshot to skip the removed ones, or

        Explicitly terminate the chain at a cut point.

    Otherwise, you end up with dangling previousSnapshot IRIs.

    Branching / non-linear history

    If you do merges/branches at the metadata level (e.g. merge two divergent histories):

        A single sflo:previousSnapshot pointer only allows a single parent.

        You need to decide:

            Do you want a linearized history (one parent chosen as “canonical”),

            Or do you want a multi-parent relation (e.g. sflo:previousSnapshot with maxCount > 1)?

    For now, your “single previous version” assumption implies “linearized history”. That’s fine; just recognise it.

    Reference direction

    You’re choosing:

        current → previous via sflo:previousSnapshot.

    That’s the cheapest for updates (you never touch old snapshots).
    Don’t later introduce the inverse (previous → next) unless you’re okay updating old snapshots or maintaining a separate index dataset.

5. Summary of the consistent rule

For metadata flows:

    _default:

        Always a byte-for-byte mirror of the latest snapshot dataset.

        No extra history beyond what that snapshot itself contains.

    Latest metadata snapshot (say Mn):

        Contains:

            all current metadata content,

            sflo:previousSnapshot Mn-1 (if any),

            weave/time fields (weaveRunId, weaveLabel, generatedAtTime, etc.).

    History:

        Reconstructed solely by following sflo:previousSnapshot, not by reading _default.

With that, your stated design is internally consistent and doesn’t secretly overload _default with log semantics.
You said:
On compaction, it's probably fine to leave previousSnapshot dangling. I don't think we can merge divergent meshes, but the next version can always just have whatever the merged data would be. 

I think we're decided about a lot of stuff. Can you rewrite the task for me?
ChatGPT said:

Here is a rewritten, internally consistent, sharply-scoped task spec that matches everything you’ve now decided, eliminates contradictions, and bakes in the clarified semantics for:

    snapshot naming,

    weave identifiers,

    _default,

    metadata flows,

    chain-of-history,

    compaction behavior,

    no merging of divergent meshes,

    dangling previousSnapshot allowed.

It does not introduce anything new; it consolidates your actual decisions.
TASK — Replace Monotonic _vN Snapshots With Human-Readable Weave Labels + Linear Snapshot Chain

ID: 2025-11-XX-weave-label-redesign
Title: Replace _vN folder versions with human-readable weave labels and linear snapshot metadata
Created: 2025-11-XX
Updated: 2025-11-XX
GOAL

Replace _vN snapshot folder naming with a more human-readable weave label of the form:

YYYY-MM-DD_HHMM[_suffix]_vN

Examples:

    2025-11-24_0142_v1

    2025-11-24_0142a_v2

    2025-11-24_0142za_v3

While preserving:

    Immutable snapshots

    Consistent _default semantics

    A linear previousSnapshot chain for metadata flows

    Minimal reliance on labels for semantics (canonical identity remains the dataset IRI + metadata)

MOTIVATION

    _vN gives ordinality but no human context.

    Full millisecond timestamps (20251109181158123) are too visually dense for humans.

    Weaves are slow, human-mediated operations, so:

        Minute-resolution labels are adequate, and

        Same-minute multiple runs are rare but allowed.

    Human-friendly labels improve UX without affecting machine identity, which is based on metadata.

SPECIFICATION
1. Snapshot Label Format

Each snapshot folder uses:

<weaveLabel>_v<sequenceNumber>

Where:

    weaveLabel: a human-oriented identifier, generated once per weave run:

YYYY-MM-DD_HHMM
YYYY-MM-DD_HHMMa
YYYY-MM-DD_HHMMb
...
YYYY-MM-DD_HHMMz
YYYY-MM-DD_HHMMza
...

(infinite base-26 suffix using lowercase a–z)

sequenceNumber:
A strictly monotonic integer per flow, stored inside snapshot metadata:

    sflo:sequenceNumber 17 .

Sequence number provides ordering; weave label provides readability.
2. Weave Label Allocation

    Only one process should weave a mesh/sub-mesh at a time.

    Advisory soft lock:

        At start of weave: create .weave-lock in the mesh root (atomic rename).

        Lock file contains: PID, host, timestamp.

        Stale locks may be broken with --force.

    Within a single weave run:

        Core computes weaveLabel once and reuses it for all flows.

    Suffix selection logic:

        Query existing snapshot labels for the same minute.

        Select lexicographically max existing label.

        Increment suffix.

        If no existing, suffix = empty string.

This is non-atomic across clones; Git conflicts remain acceptable.
3. Snapshot Creation (per flow)

Each flow snapshot directory:

/<node>/<flow>/
  YYYY-MM-DD_HHMM[_suffix]_vN/
  _default/
  _working/    # unchanged

A snapshot dataset includes at minimum:

:ThisSnapshot a dcat:Dataset, sflo:Snapshot ;
    sflo:weaveRunId "urn:uuid:…" ;
    sflo:weaveLabel "2025-11-24_0142a" ;
    sflo:sequenceNumber 7 ;
    prov:generatedAtTime "2025-11-24T01:42:55Z"^^xsd:dateTime ;
    sflo:previousSnapshot :PriorSnapshot .   # optional if no prior

Snapshots are immutable.
4. _default Semantics

_default/ for any flow is:

    A byte-for-byte mirror of the latest snapshot dataset.

    Contains no additional metadata not present in the snapshot itself.

    Updated on weave by replacing its contents atomically.

Therefore:

    _default = “current snapshot”

    Snapshot = “current dataset + history pointer”

    All historical links live inside the snapshot, not in _default.

This preserves your rule:

    _default should be identical to the latest version and not accumulate history content.”

5. Metadata Flows

Metadata flows follow the same snapshot pattern:

    Snapshot includes:

        All current metadata (provenance, validation, metrics, copyright),

        sflo:previousSnapshot,

        sflo:weaveRunId, sflo:weaveLabel, prov:generatedAtTime, etc.

    History model:
    A linear linked list via sflo:previousSnapshot.

    Compaction:
    If an old snapshot is deleted, leaving sflo:previousSnapshot dangling, this is acceptable and will not break weaves.

    No merge semantics between divergent histories; next weave simply materializes the correct latest metadata.

6. Canonical Identity

Canonical identity of a snapshot is:

IRI of the snapshot dataset
+
the metadata inside it

NOT the folder label.

Folder names are for:

    Human readability,

    Local organization,

    Optional suffix disambiguation.

Sequence numbers (_vN) remain canonical for ordering; weave labels do not.
7. Required Deliverables

    Core implementation

        Weave label generator

        Suffix increment function

        Previous-snapshot linking

        Snapshot folder creation

        _default synchronization

        .weave-lock advisory lock support

    Ontology updates

        sflo:weaveLabel

        sflo:sequenceNumber

        sflo:previousSnapshot

        Clarified comments for snapshot semantics

    CLI updates

        Display weaveLabel when weaving

        Warn on stale .weave-lock

        --force lock override

    Documentation

        Replace all _vN references

        Add weave label specification

        Update snapshot layout diagrams

        Add rules for metadata flows and history

NOTES / RATIONALE

    Sequence number handles ordinality cleanly.

    Weave labels provide mnemonic, clustered human context.

    _default remains simple and does not accidentally become a log.

    History is captured cleanly inside snapshot metadata (single predecessor).

    Compaction is graceful; dangling previous pointers are allowed.

    No merging of divergent histories reduces complexity without limiting real workflows.

If you want, I can also generate:

    A minimal SHACL profile for snapshots,

    A folder-layout diagram,

    Or the corresponding JSON-LD contexts for sflo:weaveLabel, etc.

You said:
Here's my ontology at the moment. Please provide feedback and suggest next steps.

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "prov": "http://www.w3.org/ns/prov#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/"
  },
  "@graph": [
    {
      "@id": "/",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": { "@id": "https://djradon.github.io/ns/djradon/" },
      "owl:versionInfo": "0.1.0",
      "owl:versionIRI": { "@id": "https://semantic-flow.github.io/ontology/semantic-flow/_payload-flow/_default/sflo.ttl" },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow/"
    },
    {
      "@id": "Resource/",
      "@type": "rdfs:Class",
      "rdfs:label": "Resource",
      "rdfs:comment": "Abstract base class for all Semantic Flow resources. Provides fundamental identification and documentation affordances."
    },
    {
      "@id": "FileResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Resource/" },
      "rdfs:label": "File Resource",
      "rdfs:comment": "A resource that locates and corresponds directly to a file."
    },
    {
      "@id": "NamingResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Resource/" },
      "rdfs:label": "Naming Resource",
      "rdfs:comment": "A resource that establishes a name, optionally denoting a non-file thing. Naming resources' IRIs end with a slash and may or may not have a referent."
    },
    {
      "@id": "hasResource/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Resource/" },
      "schema:rangeIncludes": { "@id": "Resource/" },
      "rdfs:label": "has resource",
      "rdfs:comment": "Abstract base property for associating resources."
    },
    {
      "@id": "AssetTree/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "Component/" },
        { "@id": "NamingResource/" }
      ],
      "rdfs:label": "AssetTree",
      "rdfs:comment": "Asset trees contain collections of files and directories that are not MeshResources. Can be contained in any NamingResource."
    },
    {
      "@id": "MeshNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NamingResource/" },
      "rdfs:label": "Mesh Node",
      "rdfs:comment": "Structural units that organize and contain other resources. Nodes form the structure of the mesh."
    },
    {
      "@id": "BareNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Namespace",
      "rdfs:comment": "Bare nodes - containers for organizing other mesh nodes. Node identifiers refer to the nodes themselves as organizational units."
    },
    {
      "@id": "ReferenceNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Reference",
      "rdfs:comment": "Nodes that refer to some external 'thing' or concept other than themselves."
    },
    {
      "@id": "PayloadNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Data",
      "rdfs:comment": "Nodes that contain and refer to 'payload data' via a payload flows."
    },
    {
      "@id": "hasComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasResource/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "Component/" },
      "rdfs:label": "has component",
      "rdfs:comment": "Associates nodes with their supporting components."
    },
    {
      "@id": "isHandleFor/",
      "@type": "owl:ObjectProperty",
      "owl:inverseOf": { "@id": "hasHandle/" },
      "rdfs:domain": { "@id": "HandleAsComponent/" },
      "rdfs:range": { "@id": "MeshNode/" },
      "rdfs:label": "is handle for",
      "rdfs:comment": "Indicates that this handle component provides identification and routing for a specific node."
    },
    {
      "@id": "hasHandle/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasComponent/" },
      "owl:inverseOf": { "@id": "isHandleFor/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "HandleAsComponent/" },
      "rdfs:label": "has handle",
      "rdfs:comment": "Associates a mesh node with its persistent handle component."
    },
    {
      "@id": "slug/",
      "@type": "owl:AnnotationProperty",
      "schema:domainIncludes": [
        { "@id": "MeshNode/" },
        { "@id": "Flow/" }
      ],
      "schema:rangeIncludes": { "@id": "xsd:string" },
      "rdfs:label": "has slug",
      "rdfs:comment": "A concise, human-readable, kebab-case identifier for a node or a flow, suitable for use in filenames and URLs."
    },
    {
      "@id": "Component/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Resource/" },
      "rdfs:label": "Component",
      "rdfs:comment": "Terminal resources that provide specific mesh functionality and support nodes."
    },
    {
      "@id": "Handle/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NamingResource/" },
      "rdfs:label": "Handle",
      "rdfs:comment": "Conceptual handle that refers to a node 'as a node' (as opposed to the thing/concept referred to by the parent node). This is the abstract handle concept."
    },
    {
      "@id": "HandleAsComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "Component/" },
        { "@id": "Resource/" }
      ],
      "rdfs:label": "Handle As Component",
      "rdfs:comment": "A handle considered as a mesh component in its own right, addressable via IRI and subject to mesh operations."
    },
    {
      "@id": "DocumentationResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "FileResource/" },
      "rdfs:label": "Documentation Resource",
      "rdfs:comment": "Documentation files that provide human-readable information about resources."
    },
    {
      "@id": "ResourcePage/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Resource Page",
      "rdfs:comment": "Human-readable index.html pages that provide dereferenceability for NamingResources."
    },
    {
      "@id": "Changelog/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Changelog",
      "rdfs:comment": "Changelog files that document version history and changes for resources."
    },
    {
      "@id": "Readme/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Readme",
      "rdfs:comment": "Readme files that provide introductory documentation and usage information."
    },
    {
      "@id": "hasDocumentationResource/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "DocumentationResource/" },
      "rdfs:label": "has DocumentationResource",
      "rdfs:comment": "Associates a resource with a documentation resource."
    },
    {
      "@id": "hasResourcePage/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "ResourcePage/" },
      "rdfs:label": "has ResourcePage",
      "rdfs:comment": "Associates a NamingResource with its index.html resource page file."
    },
    {
      "@id": "hasReadme/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "Readme/" },
      "rdfs:label": "has Readme",
      "rdfs:comment": "Associates a NamingResource with its README documentation file."
    },
    {
      "@id": "hasChangelog/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "Changelog/" },
      "rdfs:label": "has Changelog",
      "rdfs:comment": "Associates a NamingResource with its changelog documentation file."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "dcat:DatasetSeries" },
        { "@id": "Resource/" }
      ],
      "rdfs:label": "Flow",
      "rdfs:comment": "Abstract evolvable datasets for node metadata, reference data, payload data, and configuration. Flows are DatasetSeries that are realized through flow snapshots representing specific states or versions."
    },
    {
      "@id": "NodeMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_node-metadata-flow",
      "rdfs:label": "Metadata Flow",
      "rdfs:comment": "Flow containing metadata about the mesh node itself."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_reference-flow",
      "rdfs:label": "Reference Flow",
      "rdfs:comment": "Flow containing reference data about the thing or concept referred to by the mesh node."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_payload-flow",
      "rdfs:label": "Payload Flow",
      "rdfs:comment": "Flow containing the dataset payload associated with the mesh node."
    },
    {
      "@id": "NodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "rdfs:label": "Node Config Flow",
      "rdfs:comment": "Abstract flow for mesh node configuration data."
    },
    {
      "@id": "InheritableNodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NodeConfigFlow/" },
      "slug/": "_config-inheritable-flow",
      "rdfs:label": "Inheritable Node Config Flow",
      "rdfs:comment": "Node configuration flow whose settings can be inherited by child nodes."
    },
    {
      "@id": "OperationalNodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NodeConfigFlow/" },
      "slug/": "_config-operational-flow",
      "rdfs:label": "Operational Node Config Flow",
      "rdfs:comment": "Node configuration flow whose settings apply only to the specific node."
    },
    {
      "@id": "hasFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasComponent/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "Flow/" },
      "rdfs:label": "has Flow",
      "rdfs:comment": "Links mesh nodes to their flows (metadata, reference, data, config)."
    },
    {
      "@id": "hasNodeMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "NodeMetadataFlow/" },
      "rdfs:label": "has NodeMetadataFlow",
      "rdfs:comment": "Links mesh nodes to their metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "ReferenceFlow/" },
      "rdfs:label": "has ReferenceFlow",
      "rdfs:comment": "Links mesh nodes to their reference flow."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "PayloadFlow/" },
      "rdfs:label": "has PayloadFlow",
      "rdfs:comment": "Links mesh nodes to their payload flow."
    },
    {
      "@id": "hasInheritableNodeConfigFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "InheritableNodeConfigFlow/" },
      "rdfs:label": "has InheritableNodeConfigFlow",
      "rdfs:comment": "Links mesh nodes to their inheritable node config flow."
    },
    {
      "@id": "hasOperationalNodeConfigFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "OperationalNodeConfigFlow/" },
      "rdfs:label": "has OperationalNodeConfigFlow",
      "rdfs:comment": "Links mesh nodes to their operational node config flow."
    },
    {
      "@id": "Snapshot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": [
        { "@id": "NamingResource/" },
        { "@id": "dcat:Dataset" }
      ],
      "rdfs:label": "Snapshot",
      "rdfs:comment": "Snapshots represent distinct states of a node flow (dataset), and contain actual RDF data files (aka Distributions)."
    },
    {
      "@id": "VersionSnapshot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": { "@id": "Snapshot/" },
      "rdfs:label": "VersionSnapshot",
      "rdfs:comment": "Historical version snapshot of a node flow, representing a specific versioned snapshot."
    },
    {
      "@id": "DefaultSnapshot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": { "@id": "Snapshot/" },
      "rdfs:label": "DefaultSnapshot",
      "rdfs:comment": "The default snapshot of a node flow, representing the latest state."
    },
    {
      "@id": "hasSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "prov:specializationOf" },
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "Snapshot/" },
      "rdfs:label": "hasSnapshot",
      "rdfs:comment": "Associates a Flow with its snapshots. This property enables version management within flows and helps implement the prov: model"
    },
    {
      "@id": "hasDefaultSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasSnapshot/" },
      "rdfs:domain": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "DefaultSnapshot/" },
      "rdfs:label": "hasDefaultSnapshot",
      "rdfs:comment": "Associates a Flow with its default snapshot."
    },
    {
      "@id": "hasWorkingSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasSnapshot/" },
      "rdfs:label": "has working snapshot",
      "rdfs:comment": "Links to the working snapshot of this flow.",
      "rdfs:domain": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "DefaultSnapshot/" }
    },
    {
      "@id": "hasVersionSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasSnapshot/" },
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "VersionSnapshot/" },
      "rdfs:label": "hasVersionSnapshot",
      "rdfs:comment": "Associates a Flow with its version snapshots."
    },
    {
      "@id": "hasNextSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "dcat:nextVersion" },
      "rdfs:label": "has working snapshot",
      "rdfs:comment": "Links to the working version snapshot of this snapshot.",
      "schema:domainIncludes": { "@id": "VersionSnapshot/" },
      "schema:rangeIncludes": { "@id": "VersionSnapshot/" }
    },
    {
      "@id": "hasPreviousSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "dcat:previousVersion" },
      "rdfs:label": "has previous snapshot",
      "rdfs:comment": "Links to the previous version snapshot of this snapshot.",
      "schema:domainIncludes": { "@id": "VersionSnapshot/" },
      "schema:rangeIncludes": { "@id": "VersionSnapshot/" }
    },
    {
      "@id": "VersioningState/",
      "@type": "owl:Class",
      "rdfs:label": "Versioning State",
      "rdfs:comment": "Enumeration of possible versioning states for a flow.",
      "owl:oneOf": {
        "@list": [
          { "@id": "VersioningState/versioned/" },
          { "@id": "VersioningState/unversioned/" },
          { "@id": "VersioningState/deversioned/" }
        ]
      }
    },
    {
      "@id": "VersioningState/versioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "Versioned",
      "rdfs:comment": "The flow maintains version snapshots with distinct version keys."
    },
    {
      "@id": "VersioningState/unversioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "unversioned",
      "rdfs:comment": "The flow does not maintain separate version snapshots; only working snapshots exist."
    },
    {
      "@id": "VersioningState/deversioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "deversioned",
      "rdfs:comment": "The flow previously maintained version snapshots but has been deversioned; historical versions may still exist."
    },
    {
      "@id": "versioningState/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "versioning state",
      "rdfs:comment": "The versioning state of a flow.",
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "VersioningState/" }
    },
    {
      "@id": "versionKey/",
      "@type": "owl:DatatypeProperty",
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "xsd:decimal" },
      "rdfs:label": "version key",
      "rdfs:comment": "A sortable numeric identifier for a flow's snapshots. The value encodes the UTC date and time of creation (weave) in compact decimal form (YYYYMMDD.HHMMSS or YYYYMMDD.HHMMSSmmm), enabling simple chronological (lexical AND numeric) comparison in SPARQL and across file-system hierarchies. Used as the identifier for folder-named snapshots and as a lightweight temporal ordering key when full RDF provenance data is unavailable.",
      "rdfs:seeAlso": [
        { "@id": "dcat:version" },
        { "@id": "prov:generatedAtTime" }
      ]
    },
    {
      "@id": "hasOperationalConfigFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "OperationalNodeConfigFlow/" },
      "rdfs:label": "opertaional config",
      "rdfs:comment": "Links mesh nodes to their operational config flow."
    },
    {
      "@id": "hasInheritableConfigFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "InheritableNodeConfigFlow/" },
      "rdfs:label": "inheritable config",
      "rdfs:comment": "Links mesh nodes to their operational config flow."
    },
    {
      "@id": "hasMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "NodeMetadataFlow/" },
      "rdfs:label": "metadata flow",
      "rdfs:comment": "Links mesh nodes to their metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "Reference/" },
      "rdfs:label": "reference flow",
      "rdfs:comment": "Links mesh nodes to their reference flow."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "Data/" },
      "rdfs:label": "payload flow",
      "rdfs:comment": "Links mesh nodes to their data flow."
    },
    {
      "@id": "_:disjoint1",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "FileResource/" },
          { "@id": "NamingResource/" }
        ]
      }
    },
    {
      "@id": "_:disjoint2",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "BareNode/" },
          { "@id": "ReferenceNode/" },
          { "@id": "PayloadNode/" }
        ]
      }
    },
    {
      "@id": "_:disjoint3",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "HandleAsComponent/" },
          { "@id": "DocumentationResource/" }
        ]
      }
    },
    {
      "@id": "_:disjoint4",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "NodeMetadataFlow/" },
          { "@id": "ReferenceFlow/" },
          { "@id": "PayloadFlow/" },
          { "@id": "InheritableNodeConfigFlow/" },
          { "@id": "OperationalNodeConfigFlow/" }
        ]
      }
    }
  ]
}
ChatGPT said:
You said:
I also have a separate meta-flow-ontology for metadata flows:


{
  "@context": {
    "base": "https://semantic-flow.github.io/ontology/meta-flow/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "sh": "http://www.w3.org/ns/shacl#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "prov": "http://www.w3.org/ns/prov#",
    "schema": "https://schema.org/",
    "meta": "https://semantic-flow.github.io/ontology/meta-flow/",
    "mesh": "https://semantic-flow.github.io/ontology/mesh/",
    "node": "https://semantic-flow.github.io/ontology/node/",
    "flow": "https://semantic-flow.github.io/ontology/flow/"
  },
  "@graph": [
    {
      "@id": "meta:",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Metadata-Flow Ontology",
      "dcterms:description": "Comprehensive provenance and metadata schema for nodes, their flow datasets, and weaving operations. Defines properties for tracking dataset series information, versioning states, provenance contexts, delegation chains, agent roles, and weave activities. Captures complex authorship scenarios and collaborative activities within the Semantic Flow mesh ecosystem. The recommended prefix is 'meta' (alternatives: 'metaflow', 'meta-flow', 'mf').",
      "dcterms:creator": "Semantic Flow Project",
      "owl:versionInfo": "0.2.0",
      "mesh:preferredNamespacePrefix": "meta",
      "mesh:alternateNamespacePrefix": ["metaflow", "meta-flow", "mf"]
    },
    {
      "@id": "MetadataClass/",
      "@type": "rdfs:Class",
      "rdfs:label": "Metadata Class",
      "rdfs:comment": "Classes that structure metadata information within meta-flow distributions."
    },
    {
      "@id": "ProvenanceClass/",
      "@type": "rdfs:Class",
      "rdfs:label": "Provenance Class",
      "rdfs:comment": "Classes that structure provenance information within meta-flow distributions.",
      "rdfs:subClassOf": { "@id": "MetadataClass/" }
    },
    {
      "@id": "ProvenanceContext/",
      "@type": "rdfs:Class",
      "rdfs:label": "Provenance Context",
      "rdfs:comment": "A relator that captures complex authorship scenarios for mesh operations, binding together activities, agents, delegation chains, and agent roles. Uses prov:wasAttributedTo to identify the primary responsible agent.",
      "rdfs:subClassOf": { "@id": "ProvenanceClass/" }
    },
    {
      "@id": "DelegationChain/",
      "@type": "rdfs:Class",
      "rdfs:label": "Delegation Chain",
      "rdfs:comment": "A collection of delegation steps that form an authorization chain.",
      "rdfs:subClassOf": { "@id": "ProvenanceClass/" }
    },
    {
      "@id": "DelegationStep/",
      "@type": "rdfs:Class",
      "rdfs:label": "Delegation Step",
      "rdfs:comment": "An individual step in an authorization chain, showing one agent acting on behalf of another using prov:agent and prov:actedOnBehalfOf.",
      "rdfs:subClassOf": { "@id": "ProvenanceClass/" }
    },
    {
      "@id": "AgentRoleCollection/",
      "@type": "rdfs:Class",
      "rdfs:label": "Agent Role Collection",
      "rdfs:comment": "A collection of agent role assignments for collaborative activities.",
      "rdfs:subClassOf": { "@id": "ProvenanceClass/" }
    },
    {
      "@id": "AgentRole/",
      "@type": "rdfs:Class",
      "rdfs:label": "Agent Role",
      "rdfs:comment": "Describes the specific role an agent plays in a collaborative activity, using prov:agent to identify the agent.",
      "rdfs:subClassOf": { "@id": "ProvenanceClass/" }
    },
    {
      "@id": "RoleType/",
      "@type": "rdfs:Class",
      "rdfs:label": "Role Type",
      "rdfs:comment": "Types of roles agents can play in activities."
    },
    {
      "@id": "MeshActivity/",
      "@type": "rdfs:Class",
      "rdfs:label": "Mesh Activity",
      "rdfs:comment": "Base class for all Semantic Flow mesh operations including weaving, grafting, and initialization activities.",
      "rdfs:subClassOf": { "@id": "prov:Activity" }
    },
    {
      "@id": "WeaveActivity/",
      "@type": "rdfs:Class",
      "rdfs:label": "Weave Activity",
      "rdfs:comment": "Base class for all Semantic Flow mesh weaving operations.",
      "rdfs:subClassOf": { "@id": "MeshActivity/" }
    },
    {
      "@id": "NodeCreation/",
      "@type": "rdfs:Class",
      "rdfs:label": "Node Creation",
      "rdfs:comment": "Activity for creating new nodes in mesh repositories.",
      "rdfs:subClassOf": { "@id": "MeshActivity/" }
    },
    {
      "@id": "RepoInitialization/",
      "@type": "rdfs:Class",
      "rdfs:label": "Repository Initialization",
      "rdfs:comment": "Activity for initializing new mesh repositories with their foundational structure.",
      "rdfs:subClassOf": { "@id": "MeshActivity/" }
    },
    {
      "@id": "NodeInitialization/",
      "@type": "rdfs:Class",
      "rdfs:label": "Node Initialization",
      "rdfs:comment": "Activity for initializing individual nodes within repository initialization.",
      "rdfs:subClassOf": { "@id": "RepoInitialization/" }
    },
    {
      "@id": "ConfigWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Config Weave",
      "rdfs:comment": "Weaving operation that updates configuration flows.",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "ReferenceWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Reference Weave",
      "rdfs:comment": "Weaving operation that updates reference flows.",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "PayloadWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Payload Weave",
      "rdfs:comment": "Weaving operation that updates payload data flows.",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "MetaWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Meta Weave",
      "rdfs:comment": "Weaving operation that updates meta-flows (automatically triggered).",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "NodeWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Node Weave",
      "rdfs:comment": "Weaving operation that updates all flows within a node.",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "NodeTreeWeave/",
      "@type": "rdfs:Class",
      "rdfs:label": "Node Tree Weave",
      "rdfs:comment": "Weaving operation that recursively updates nodes and their contained nodes.",
      "rdfs:subClassOf": { "@id": "WeaveActivity/" }
    },
    {
      "@id": "dataExtraction/",
      "@type": "RoleType/",
      "rdfs:label": "data extraction",
      "rdfs:comment": "Responsible for extracting or gathering data."
    },
    {
      "@id": "qualityAssurance/",
      "@type": "RoleType/",
      "rdfs:label": "quality assurance",
      "rdfs:comment": "Responsible for validating and ensuring quality."
    },
    {
      "@id": "finalApproval/",
      "@type": "RoleType/",
      "rdfs:label": "final approval",
      "rdfs:comment": "Responsible for final review and approval."
    },
    {
      "@id": "coordination/",
      "@type": "RoleType/",
      "rdfs:label": "coordination",
      "rdfs:comment": "Responsible for coordinating between other agents."
    },
    {
      "@id": "versioningState/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "versioning state",
      "rdfs:comment": "The versioning state of a flow: 'versioned', 'unversioned', or 'deversioned'.",
      "schema:domainIncludes": { "@id": "node:Flow/" },
      "schema:rangeIncludes": { "@id": "xsd:string" }
    },
    {
      "@id": "forActivity/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "for activity",
      "rdfs:comment": "Links a provenance context to the activity it describes.",
      "schema:domainIncludes": { "@id": "ProvenanceContext/" },
      "schema:rangeIncludes": { "@id": "prov:Activity" }
    },
    {
      "@id": "forSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "for snapshot",
      "rdfs:comment": "Links a provenance context to the specific snapshot it describes.",
      "schema:domainIncludes": { "@id": "ProvenanceContext/" },
      "schema:rangeIncludes": { "@id": "flow:Snapshot/" }
    },
    {
      "@id": "delegationChain/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "delegation chain",
      "rdfs:comment": "Links to a collection of delegation steps showing the authorization chain.",
      "schema:domainIncludes": { "@id": "ProvenanceContext/" },
      "schema:rangeIncludes": { "@id": "DelegationChain/" }
    },
    {
      "@id": "agentRoles/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "agent roles",
      "rdfs:comment": "Links to a collection of agent role assignments.",
      "schema:domainIncludes": { "@id": "ProvenanceContext/" },
      "schema:rangeIncludes": { "@id": "AgentRoleCollection/" }
    },
    {
      "@id": "hasStep/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has step",
      "rdfs:comment": "Links a delegation chain to its individual steps.",
      "schema:domainIncludes": { "@id": "DelegationChain/" },
      "schema:rangeIncludes": { "@id": "DelegationStep/" }
    },
    {
      "@id": "stepOrder/",
      "@type": "owl:DatatypeProperty",
      "rdfs:label": "step order",
      "rdfs:comment": "The order of this step in the delegation chain (1, 2, 3, etc.).",
      "schema:domainIncludes": { "@id": "DelegationStep/" },
      "schema:rangeIncludes": { "@id": "xsd:integer" }
    },
    {
      "@id": "hasRole/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "has role",
      "rdfs:comment": "Links an agent role collection to individual role assignments.",
      "schema:domainIncludes": { "@id": "AgentRoleCollection/" },
      "schema:rangeIncludes": { "@id": "AgentRole/" }
    },
    {
      "@id": "role/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "role",
      "rdfs:comment": "The specific role or function the agent performs.",
      "schema:domainIncludes": { "@id": "AgentRole/" },
      "schema:rangeIncludes": { "@id": "RoleType/" }
    },
    {
      "@id": "wasTriggeredBy/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "was triggered by",
      "rdfs:comment": "Links an automatic activity (like MetaWeave) to the activity that triggered it.",
      "schema:domainIncludes": { "@id": "prov:Activity" },
      "schema:rangeIncludes": { "@id": "prov:Activity" }
    },
    {
      "@id": "FlowVersioningStateShape/",
      "@type": "sh:PropertyShape",
      "sh:targetClass": { "@id": "node:Flow/" },
      "sh:path": { "@id": "versioningState/" },
      "sh:in": {
        "@list": [
          "versioned",
          "unversioned",
          "deversioned"
        ]
      },
      "sh:message": "Versioning state must be 'versioned', 'unversioned', or 'deversioned'"
    },
    {
      "@id": "ProvenanceContextShape/",
      "@type": "sh:NodeShape",
      "sh:targetClass": { "@id": "ProvenanceContext/" },
      "sh:property": [
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "forActivity/" },
          "sh:class": { "@id": "prov:Activity" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Provenance context must reference exactly one activity"
        },
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "forSnapshot/" },
          "sh:class": { "@id": "flow:Snapshot/" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Provenance context must reference exactly one snapshot"
        },
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "prov:wasAttributedTo" },
          "sh:class": { "@id": "prov:Agent" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Provenance context must have exactly one primary agent"
        }
      ]
    },
    {
      "@id": "DelegationStepShape/",
      "@type": "sh:NodeShape",
      "sh:targetClass": { "@id": "DelegationStep/" },
      "sh:property": [
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "prov:agent" },
          "sh:class": { "@id": "prov:Agent" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Delegation step must have exactly one agent"
        },
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "stepOrder/" },
          "sh:datatype": { "@id": "xsd:integer" },
          "sh:minInclusive": 1,
          "sh:message": "Step order must be a positive integer"
        }
      ]
    },
    {
      "@id": "AgentRoleShape/",
      "@type": "sh:NodeShape",
      "sh:targetClass": { "@id": "AgentRole/" },
      "sh:property": [
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "prov:agent" },
          "sh:class": { "@id": "prov:Agent" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Agent role must have exactly one agent"
        },
        {
          "@type": "sh:PropertyShape",
          "sh:path": { "@id": "role/" },
          "sh:class": { "@id": "RoleType/" },
          "sh:minCount": 1,
          "sh:maxCount": 1,
          "sh:message": "Agent role must specify exactly one role type"
        },
        {
          "@id": "DelegationStepConditionalShape/",
          "@type": "sh:NodeShape",
          "sh:targetClass": { "@id": "DelegationStep/" },
          "sh:or": [
            {
              "@type": "sh:NodeShape",
              "sh:property": [
                {
                  "@type": "sh:PropertyShape",
                  "sh:path": { "@id": "stepOrder/" },
                  "sh:hasValue": 1
                }
              ]
            },
            {
              "@type": "sh:NodeShape",
              "sh:property": [
                {
                  "@type": "sh:PropertyShape",
                  "sh:path": { "@id": "stepOrder/" },
                  "sh:minExclusive": 1
                },
                {
                  "@type": "sh:PropertyShape",
                  "sh:path": { "@id": "prov:actedOnBehalfOf" },
                  "sh:class": { "@id": "prov:Agent" },
                  "sh:minCount": 1,
                  "sh:maxCount": 1,
                  "sh:message": "Delegation steps after step 1 must specify exactly one entity the agent acts for"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
ChatGPT said:

I’ll focus on inconsistencies and misalignments with your current design, then suggest concrete next steps.
1. High-level alignment with your current mental model

Your recent decisions:

    Flows are evolvable dataset series.

    Snapshots are immutable datasets, with a linear previousSnapshot chain.

    _default is a byte-for-byte mirror of the latest snapshot, not a distinct conceptual class.

    _working is staging only, not authoritative.

    You’re shifting from versionKey/_vN to weave labels + per-flow ordinals.

Given that, the current ontology is clearly an earlier iteration. The main work is to:

    Remove or demote the older versioning constructs.

    Introduce the new weave/snapshot pattern cleanly.

    De-duplicate / reconcile meta-flow with sflo.

2. Critical issues in the main sflo ontology
2.1 Conflicting / broken has*Flow properties

You define:

    hasReferenceFlow/ (MeshNode → ReferenceFlow/)

    hasPayloadFlow/ (MeshNode → PayloadFlow/)

    etc. (domain MeshNode/, range the relevant Flow/ subclasses)

Later you define again:

{
  "@id": "hasReferenceFlow/",
  "@type": "owl:ObjectProperty",
  "schema:domainIncludes": { "@id": "Handle/" },
  "schema:rangeIncludes": { "@id": "Reference/" },
  "rdfs:label": "reference flow",
  "rdfs:comment": "Links mesh nodes to their reference flow."
},
{
  "@id": "hasPayloadFlow/",
  "@type": "owl:ObjectProperty",
  "schema:domainIncludes": { "@id": "Handle/" },
  "schema:rangeIncludes": { "@id": "Data/" },
  "rdfs:label": "payload flow",
  "rdfs:comment": "Links mesh nodes to their data flow."
}

Problems:

    Same property IRIs reused with different domain/range.

    Ranges Reference/ and Data/ do not exist (you have ReferenceNode/, PayloadNode/, ReferenceFlow/, PayloadFlow/).

    Comments talk about “mesh nodes” but domain is Handle/.

This is hard-broken. You need to:

    Decide: are flows attached to MeshNode/, Handle/, or both?

    Use one set of properties for flows; don’t redefine under the same IRIs.

    Fix the ranges to the right classes (ReferenceFlow/, PayloadFlow/).

Right now a reasoner or tooling cannot trust these properties.
2.2 Snapshot/versioning stack is out of sync with your decisions

You currently have:

    Classes:

        Snapshot/ ⊑ NamingResource/, dcat:Dataset

        VersionSnapshot/ ⊑ Snapshot/

        DefaultSnapshot/ ⊑ Snapshot/

    Properties:

        hasSnapshot/ ⊑ prov:specializationOf

        hasDefaultSnapshot/

        hasWorkingSnapshot/ (range DefaultSnapshot/ but label says working!)

        hasVersionSnapshot/

        hasNextSnapshot/ ⊑ dcat:nextVersion

        hasPreviousSnapshot/ ⊑ dcat:previousVersion

Issues:

    You’ve explicitly decided:

        _default is just “latest snapshot” – not a fundamentally different thing. In your new model, DefaultSnapshot is not a separate class.

        _working is a staging area, not an authoritative snapshot – it shouldn’t be modeled as a first-class Snapshot or via hasWorkingSnapshot.

        You want a single previousSnapshot-style chain, not full DCAT next/previous plus separate “working snapshot” semantics.

So:

    VersionSnapshot/, DefaultSnapshot/, hasWorkingSnapshot/, hasVersionSnapshot/, hasNextSnapshot/ are now conceptually redundant or wrong.

    The simple pattern you described verbally is:

        One Snapshot class.

        On each Snapshot: a sflo:previousSnapshot pointer (0 or 1).

        On Flow: a pointer to the current snapshot (what _default mirrors).

The ontology should be refactored to reflect that.
2.3 versionKey/ is obsolete relative to weave labels

You currently have:

{
  "@id": "versionKey/",
  "@type": "owl:DatatypeProperty",
  "schema:domainIncludes": { "@id": "Flow/" },
  "schema:rangeIncludes": { "@id": "xsd:decimal" },
  "rdfs:label": "version key",
  "rdfs:comment": "A sortable numeric identifier ... encodes UTC date and time in decimal form (YYYYMMDD.HHMMSS or YYYYMMDD.HHMMSSmmm)..."
}

But you’ve moved to:

    Human-readable weave label (YYYY-MM-DD_HHMM[…]).

    Per-flow integer sequence number.

    RDF provenance via prov:generatedAtTime.

This decimal versionKey approach is now a competing scheme. Unless you really intend to keep both:

    Either:

        Replace versionKey/ with something like sflo:weaveLabel (xsd:string) and sflo:sequenceNumber (xsd:integer); or

        Mark versionKey/ explicitly “legacy; do not use”.

Right now the ontology is pushing an encoding you explicitly walked away from.
2.4 Versioning state is double-defined (sflo vs meta-flow)

In sflo:

    VersioningState/ is an OWL class with individuals:

        VersioningState/versioned/, …/unversioned/, …/deversioned/.

    versioningState/ is an object property with range VersioningState/.

In meta-flow:

    versioningState/ is a datatype property on node:Flow/ with range xsd:string.

    SHACL FlowVersioningStateShape/ enforces values "versioned", "unversioned", "deversioned".

So you have:

    Same local name versioningState/

    Completely different modeling:

        one as object property to enumeration,

        one as string with SHACL.

Pick one pattern and make the other reuse it:

    If you like the OWL enumeration:

        Keep sflo’s VersioningState/ + object property.

        Change meta-flow’s versioningState/ to that same object property and adjust the SHACL to sh:class VersioningState/ instead of sh:in over strings.

Right now anyone consuming both ontologies gets contradictions.
2.5 Handle vs node domains

At the end of sflo you have:

{
  "@id": "hasOperationalConfigFlow/",
  "schema:domainIncludes": { "@id": "Handle/" },
  "schema:rangeIncludes": { "@id": "OperationalNodeConfigFlow/" },
  "rdfs:comment": "Links mesh nodes to their operational config flow."
}

The comments say “links mesh nodes”, but domain is Handle/. Same for hasInheritableConfigFlow/, hasMetadataFlow/.

You need to decide:

    Are flows conceptually attached to:

        The MeshNode,

        The Handle (node-as-node),

        Or both, with explicit equivalences?

Right now it’s contradictory.
2.6 Minor but real issues

    hasNextSnapshot/ label is “has working snapshot” – clearly a copy-paste bug.

    hasWorkingSnapshot/ range is DefaultSnapshot/ – doesn’t match the name/comment.

    Typo: "opertaional config" label.

    HandleAsComponent/ is declared as subclass of both Component/ and Resource/ even though Component/ is already subclass of Resource/ – redundant but harmless.

3. Structural / modular concerns
3.1 IRI and prefix consistency

Main sflo:

    Uses @base: "https://semantic-flow.github.io/ontology/semantic-flow/".

    Classes and properties are "Resource/", "Flow/", "Snapshot/", etc.

Meta-flow:

    Uses meta, mesh, node, flow prefixes:

        node:Flow/

        flow:Snapshot/

But in sflo your Flow/ and Snapshot/ are under the semantic-flow base, not the node: and flow: bases meta-flow references.

Either:

    You’ve separated ontologies (mesh, node, flow, meta) and the snippet you pasted for sflo is actually “node/flow” combined; or

    The prefixes in meta-flow are out of date relative to your current merged sflo ontology.

In any case, meta-flow should reference the actual IRIs you are using for Flow/Snapshot, or you’ll end up with duplicate parallel hierarchies conceptually describing the same thing.
3.2 Snapshot as NamingResource + dcat:Dataset

You’ve modeled:

    Snapshot/ ⊑ NamingResource/, dcat:Dataset.

This says:

    Every Snapshot is both:

        A dcat:Dataset (logical dataset),

        A naming resource (something with trailing-slash IRI, dereferenceable, etc.).

Given your file layout (snapshot folders as directories), that’s defensible, but think through:

    Is a Snapshot “the dataset as such”, or “a naming resource that organizes dataset distributions”?

    In DCAT, typically:

        Dataset: abstract thing,

        Distribution: physical files.

You’re sort of merging “dataset as thing” and “directory as naming resource” into one class. That might be fine for your “mesh as file tree” worldview, but be aware you’re deviating from DCAT’s conceptual separation.
4. meta-flow ontology: issues and alignment
4.1 JSON-LD context

    base in the context should be @base if you want JSON-LD to treat it as a base IRI. Right now "base": "…" does nothing for JSON-LD processors.

4.2 ProvenanceContext modeling

ProvenanceContext/:

    rdfs:subClassOf ProvenanceClass/

    But you use prov:wasAttributedTo, whose domain is prov:Entity.

    Consider adding rdfs:subClassOf prov:Entity to ProvenanceContext/ so you’re not violating PROV’s domain expectations.

4.3 Shapes nesting error

In AgentRoleShape/ you’ve nested DelegationStepConditionalShape/ inside its sh:property array. That’s almost certainly unintended:

    DelegationStepConditionalShape/ should be a top-level sh:NodeShape, not a property of AgentRoleShape.

Also, DelegationStepConditionalShape is declared with sh:targetClass DelegationStep/, but then you put it inside the AgentRole shape. That’s structurally wrong.

You want:

    DelegationStepShape/: basic constraints (agent, stepOrder).

    DelegationStepConditionalShape/: separate NodeShape with sh:targetClass DelegationStep/ and sh:or for stepOrder 1 vs >1.

Right now any SHACL engine will treat your JSON-LD as “AgentRoleShape has a property whose value happens to be another NodeShape”, not as two independent shapes.
4.4 VersioningState duplication (see 2.4 above)

meta-flow is currently string-encoding versioning state, while sflo has an OWL enumeration. Pick one pattern and reuse it.
5. Suggested concrete next steps

Given all of the above, here’s a focused sequence.
Step 1: Align versioning/snapshot model with your current design

In the sflo ontology:

    Introduce the properties that match your verbal model:

        sflo:weaveLabel (DatatypeProperty, range xsd:string, domain Snapshot/ or Flow/ or both).

        sflo:sequenceNumber (DatatypeProperty, range xsd:integer, domain Snapshot/).

        sflo:previousSnapshot (ObjectProperty, domain Snapshot/, range Snapshot/, max 1).

        Optionally sflo:currentSnapshot (ObjectProperty, domain Flow/, range Snapshot/) to model what _default mirrors.

    Demote or delete old constructs:

        Remove or mark deprecated:

            VersionSnapshot/, DefaultSnapshot/.

            hasWorkingSnapshot/, hasVersionSnapshot/, hasNextSnapshot/.

        Replace hasPreviousSnapshot/ with sflo:previousSnapshot (no need to tie it to dcat:previousVersion unless you really want to).

    Replace versionKey/ with weaveLabel/sequenceNumber or clearly mark as legacy.

Step 2: Resolve has*Flow and handle vs node

    Decide the canonical attachment point for flows:

        My suggestion: MeshNode is the primary domain for flows; Handle may have aliases but doesn’t own them.

    Keep one set of hasNodeMetadataFlow/, hasReferenceFlow/, hasPayloadFlow/, etc., with:

        rdfs:domain MeshNode/

        rdfs:range the corresponding Flow/ subclass.

    If you want handle-centric shortcuts, either:

        Define new properties handleHasPayloadFlow etc., or

        State that Handle/ is in some relation to MeshNode/ and let applications traverse via hasHandle.

But don’t reuse the same property IRIs with incompatible domains/ranges.
Step 3: Unify versioningState across sflo + meta-flow

    Keep the OWL enumeration VersioningState/ + object property versioningState/ in sflo.

    In meta-flow:

        Change its versioningState/ to be the same IRI and type (ObjectProperty).

        Update FlowVersioningStateShape/ to:

            sh:path versioningState/

            sh:class VersioningState/

        Drop the string sh:in enumeration.

This gives you one place where the states are defined, and SHACL just checks usage.
Step 4: Fix meta-flow shapes & PROV alignment

    Move DelegationStepConditionalShape/ out of AgentRoleShape/ into its own top-level NodeShape.

    Add rdfs:subClassOf prov:Entity to ProvenanceContext/.

    Check all uses of prov:* properties to ensure their domain/range expectations are at least not grossly violated.

Step 5: Prefix/IRI consolidation

    Decide: Are Flow/ and Snapshot/ in the semantic-flow ontology, or in flow:? Same for MeshNode/ vs node:.

    Then:

        Update meta-flow to reference the real IRIs (not hypothetical node:Flow/, flow:Snapshot/ if those aren’t actually defined elsewhere).

        Or, if you really do have separate node/flow ontologies, split the sflo snippet accordingly and ensure the IRIs match what meta-flow uses.

Step 6: Add minimal SHACL for the new snapshot model

Once you’ve refactored:

    Add a SnapshotShape:

        sh:targetClass Snapshot/

        sh:property:

            sh:path sflo:sequenceNumber, sh:datatype xsd:integer, sh:minCount 1, sh:maxCount 1.

            sh:path sflo:weaveLabel, sh:datatype xsd:string, sh:minCount 1, sh:maxCount 1.

    Add a FlowCurrentSnapshotShape:

        sh:targetClass Flow/

        sh:property:

            sh:path sflo:currentSnapshot, sh:class Snapshot/, sh:maxCount 1.

That will catch the most egregious modeling mistakes early.

If you want, next step I’d suggest is: take one concrete node (with a metadata flow and a payload flow), and I can produce a small, fully consistent example graph using the cleaned-up ontology: Flow, Snapshots, weaveLabel, sequenceNumber, previousSnapshot, ProvenanceContext, WeaveActivity, etc. That will smoke out any remaining conceptual gaps.
You said:
Could you give these good IDs?


    {
      "@id": "_:disjoint1",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "FileResource/" },
          { "@id": "NamingResource/" }
        ]
      }
    },
    {
      "@id": "_:disjoint2",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "BareNode/" },
          { "@id": "ReferenceNode/" },
          { "@id": "PayloadNode/" }
        ]
      }
    },
    {
      "@id": "_:disjoint3",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "HandleAsComponent/" },
          { "@id": "DocumentationResource/" }
        ]
      }
    },
    {
      "@id": "_:disjoint4",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "NodeMetadataFlow/" },
          { "@id": "ReferenceFlow/" },
          { "@id": "PayloadFlow/" },
          { "@id": "InheritableNodeConfigFlow/" },
          { "@id": "OperationalNodeConfigFlow/" }
        ]
      }
    }
  ]
}
ChatGPT said:

You should not keep these as blank nodes.
Disjoint axioms are part of the public ontology; they need stable, readable IRIs so downstream tools can refer to them, annotate them, or override them.
Principles for good IDs

    Put them under a dedicated sub-namespace
    e.g.,
    https://semantic-flow.github.io/ontology/semantic-flow/disjoint/…

    Make the name describe the “axis” of disjointness, not the OWL construct (AllDisjointClasses) itself.

    Avoid opaque numbering (disjoint1), because it will eventually become unintelligible.

    Use kebab-case or CamelCase consistently within your ontology; you’ve been using PascalCase for classes, so follow that.

Proposed IDs (clean, descriptive, stable)
1. File vs Naming resources

These represent two mutually exclusive resource categories in your model.

{
  "@id": "disjoint/FileVsNamingResources",
  "@type": "owl:AllDisjointClasses",
  "owl:members": {
    "@list": [
      { "@id": "FileResource/" },
      { "@id": "NamingResource/" }
    ]
  }
}

Rationale: Expressive, future-proof, avoids numeric suffixes.
2. Node structural roles

Bare, Reference, Payload nodes are mutually exclusive.

{
  "@id": "disjoint/NodeStructuralRoles",
  "@type": "owl:AllDisjointClasses",
  "owl:members": {
    "@list": [
      { "@id": "BareNode/" },
      { "@id": "ReferenceNode/" },
      { "@id": "PayloadNode/" }
    ]
  }
}

Rationale: “Structural roles” is semantically aligned with your node semantics.
3. Component vs Documentation resources

HandleAsComponent and DocumentationResource shouldn't overlap.

{
  "@id": "disjoint/ComponentVsDocumentationResources",
  "@type": "owl:AllDisjointClasses",
  "owl:members": {
    "@list": [
      { "@id": "HandleAsComponent/" },
      { "@id": "DocumentationResource/" }
    ]
  }
}

Rationale: Directly expresses the semantic boundary.
4. Flow type disjointness

NodeMetadataFlow, ReferenceFlow, PayloadFlow, etc.

{
  "@id": "disjoint/FlowTypes",
  "@type": "owl:AllDisjointClasses",
  "owl:members": {
    "@list": [
      { "@id": "NodeMetadataFlow/" },
      { "@id": "ReferenceFlow/" },
      { "@id": "PayloadFlow/" },
      { "@id": "InheritableNodeConfigFlow/" },
      { "@id": "OperationalNodeConfigFlow/" }
    ]
  }
}

Rationale: These are clearly “flow types,” so name the disjointness accordingly.
Optional refinement

If you want even more explicit grouping, you can nest these under:

    semantic-flow/disjoint/resources/FileVsNamingResources

    semantic-flow/disjoint/nodes/NodeStructuralRoles

    semantic-flow/disjoint/components/ComponentVsDocumentationResources

    semantic-flow/disjoint/flows/FlowTypes

You’re free to adopt that, but the simple single-level disjoint/… is sufficient and clean.

If you'd like, I can also rewrite the ontology fragment with these IDs inserted correctly, validate for JSON-LD correctness, and check for other structural inconsistencies.
You said:
I guess _default is importantly different in the sense that it is a mutable flow. I think having a separate class is fine. But yes, maybe _default and _working are "SpecialSnapshots"? What's the objection to hasWorkingSnapshot?

I removed hasNextSnapshot, but probably all those properties should be in the meta-ontology?
ChatGPT said:

You’re mixing three different things under the word “snapshot”:

    Immutable historical dataset (2025-11-24_0142_v3/)

    The mutable pointer _default/ (which always mirrors the latest immutable one)

    The staging area _working/ (which is not authoritative at all)

If you don’t keep those roles cleanly separated, the ontology will lie about what’s actually going on.

I’ll address:

    Whether _default and _working should be “SpecialSnapshots”

    What’s wrong with hasWorkingSnapshot

    Which properties belong in sflo vs meta-flow

1. _default vs immutable snapshots vs _working

Let’s be explicit:
Thing	On disk	Semantics	Should it be a Snapshot?
Versioned	YYYY-MM-DD_HHMM_vN/	Immutable, historical, authoritative	Yes
_default	_default/	Mutable pointer to “current snapshot”	Conceptually: no
_working	_working/	Staging, may be incomplete/invalid, discardable	Definitely no

You keep saying:

    _default is “importantly different” because it’s mutable as a directory.

    But also: _default’s content should be identical to the latest immutable snapshot.

That’s not a difference in kind of dataset. It’s a difference in role:

    The dataset is the immutable snapshot.

    _default is how you mount that dataset at the “current” location.

So in RDF terms:

    “Current snapshot” is a property on Flow → Snapshot, not a different class of snapshot.

    _default is an implementation detail of how you map that current snapshot to a filesystem path.

If you make DefaultSnapshot a separate class, you’re implicitly claiming:

    “There exists a dataset that is structurally/semantically different from versioned snapshots.”

That’s not true in your design; _default is just a mirror.

So: if you insist on a DefaultSnapshot class, the only non-lying interpretation is:

    DefaultSnapshot ⊑ Snapshot

    There is exactly one DefaultSnapshot per Flow, and it is owl:sameAs some Snapshot that also happens to live under a versioned folder.

That’s an ugly modeling pattern and adds no value. It’s much simpler and more honest to:

    Keep a single Snapshot class.

    Add sflo:currentSnapshot on Flow to point at the latest one.

2. Why hasWorkingSnapshot is problematic

You asked:

    What’s the objection to hasWorkingSnapshot?

Two main objections.
2.1 Category error: “working” is not a snapshot

A snapshot, by your own definition, is:

    Distinct state of a flow’s dataset.

    Immutable, addressable, something you can reproduce/compare.

_working is none of that:

    It changes under your feet.

    It may be invalid or partially edited.

    It’s explicitly non-authoritative (“staging only”).

If you model it as a Snapshot (or via hasWorkingSnapshot to a Snapshot-like thing), consumers will:

    Treat it as if it’s a stable, comparable dataset.

    Potentially persist it in SPARQL queries, SHACL, or external tools as if it were a real past state.

You’ve fought hard to keep “working” as “not a commit”. Putting it into the same class vocabulary is a regression.
2.2 Ephemeral runtime vs durable model

_working is a runtime artifact:

    Exists only in a particular working copy.

    May disappear on reset, clean, or if someone discards uncommitted changes.

    Can differ between clones at the same commit.

Trying to model it in a durable RDF ontology creates two mismatches:

    The ontology will reference things that frequently don’t exist on disk.

    Two clones at the “same” commit will have different _working but the same metadata.

That leads to either:

    Lying RDF (“there is a working snapshot X” but it’s not real on this machine), or

    Requiring that every local working copy writes local-only metadata to model _working, which destroys your ability to treat the metadata flow as part of the mesh state.

Better: do not model _working in RDF at all. Let it be purely:

    Filesystem convention

    CLI / service runtime concept

If you absolutely need to talk about “there exists a working state”, keep that in the operational config of the tool, not the mesh ontology.
3. Should _default and _working be “SpecialSnapshots”?

Given the above:

    _working: no, it’s not a snapshot.

    _default: no, it’s not a different kind of dataset; it’s just the mount point for the current snapshot.

What you can have is:

    One Snapshot class.

    One sflo:previousSnapshot property (Snapshot → Snapshot, max 1).

    One sflo:currentSnapshot property (Flow → Snapshot, max 1).

Then the mesh implementation ensures:

    _default/ on disk mirrors whatever sflo:currentSnapshot points to.

    _working/ is just staging and never appears in RDF.

That keeps the ontology aligned with how you want people to think about the system:

    Snapshots = committed, immutable.

    Current snapshot = a pointer.

    Working = not in the semantic model.

4. Which properties belong in sflo vs meta-flow?

You said:

    I removed hasNextSnapshot, but probably all those properties should be in the meta-ontology?

No, you should split by responsibility, not by “everything about snapshots goes into meta”.

Rough split:
4.1 Core structural model (sflo ontology)

Belongs in sflo:

    Classes:

        Flow

        Snapshot

    Properties describing the version graph itself:

        sflo:currentSnapshot (Flow → Snapshot)

        sflo:previousSnapshot (Snapshot → Snapshot)

        sflo:sequenceNumber (Snapshot → xsd:integer)

        sflo:weaveLabel (Snapshot → xsd:string)

    Optional:

        sflo:versioningState (Flow → VersioningState individual)

        The VersioningState enumeration

These are intrinsic to what a mesh is and how flows evolve—every consumer needs to understand them even if they don’t care about deep provenance.
4.2 Provenance and activity detail (meta-flow ontology)

Belongs in meta-flow:

    Classes:

        MeshActivity, WeaveActivity, PayloadWeave, etc.

        ProvenanceContext, DelegationChain, AgentRole, etc.

    Properties:

        meta:forActivity (ProvenanceContext → prov:Activity)

        meta:forSnapshot (ProvenanceContext → Snapshot)

        prov:wasGeneratedBy (Snapshot → WeaveActivity)

        meta:wasTriggeredBy (prov:Activity → prov:Activity)

        Delegation / agent-role details (delegationChain, agentRoles, role, etc.)

These are about how snapshots were produced (who, how, why), not about the fact that snapshots exist and are linked.

So the version chain itself (previousSnapshot, currentSnapshot) absolutely belongs in sflo, not meta-flow.
5. Concrete recommendations

Given your current direction, if you want a clean, honest model:

    Remove or deprecate:

        DefaultSnapshot/ as a separate class.

        hasWorkingSnapshot/.

        Any attempt to make _working a Snapshot.

    Define instead (in sflo):

    sflo:Snapshot a owl:Class ;
        rdfs:subClassOf dcat:Dataset , sflo:Resource .

    sflo:currentSnapshot a owl:ObjectProperty ;
        rdfs:domain sflo:Flow ;
        rdfs:range  sflo:Snapshot ;
        rdfs:comment "The snapshot currently considered 'default' for this flow." ;
        rdfs:label "current snapshot" .

    sflo:previousSnapshot a owl:ObjectProperty ;
        rdfs:domain sflo:Snapshot ;
        rdfs:range  sflo:Snapshot ;
        rdfs:comment "The immediately previous snapshot in the linear history of this flow." ;
        rdfs:label "previous snapshot" ;
        # optional: functional:
        owl:propertyDisjointWith sflo:currentSnapshot .

    And your implementation guarantees:

        _default/ mirrors whatever sflo:currentSnapshot points to.

        Version folders are the various sflo:Snapshot instances.

    Keep _working purely imperative:

        CLI and services know how to read/write _working/.

        No RDF statements about _working exist in any flow datasets.

    Keep provenance in meta-flow:

        prov:wasGeneratedBy from Snapshot to WeaveActivity.

        meta:forSnapshot linking ProvenanceContext to Snapshot.

        No hasWorkingSnapshot-style properties there either.

If you really insist on having a class that “names” the _default role, the least-damaging pattern is:

    sflo:DefaultRole as an annotation at the Flow level, not a separate DefaultSnapshot class.
    But I’d still argue it adds more complexity than value.

In short: treat _default and _working as implementation roles, not different ontological kinds of snapshot. The core model is “Flow with a linear chain of immutable Snapshots, and one designated current snapshot.” Everything else is tooling.
You said:
_default isn't a pointer, it's a copy.  its distributions (and resource page) are addressable/dereferenceable . 

_working isn't ephemeral, it always exists, and it's mutable, but "after weave" its content is identical to "_default" and the latest snapshot. Maybe we don't model it in RDF, but it is part of a mesh, and maybe will get its own resource page generated on weave. "operational config" is for mesh settings, not mesh structure. 

I definitely feel like there's not much to say about specific _default and _working, RDF-wise, but they exist in the mesh (permanently, and every flow has them, always) so that's almost reason enough to have classes for them. But I am open to not defining their classes until we need them. 
ChatGPT said:

Good, this clarifies your intent. Let’s be precise about what follows from what you just said, and where the modeling pressure actually is.

I’ll separate:

    What you’ve fixed as facts about the mesh

    What that implies for RDF classes/properties

    What I think you should concretely do next

1. The facts you’ve committed to

From your message:

    _default:

        Is a copy of the latest snapshot, not a symbolic pointer.

        Has its own distributions and resource page.

        Is permanently present for every flow.

        After each weave, _default ≡ latest snapshot (at the byte level, or very close).

    _working:

        Always exists for every flow (not ephemeral).

        Is mutable.

        After weave, its content is also identical to _default and latest snapshot (i.e., weave “flushes” working → snapshot → default).

        May get its own resource page.

    RDF-wise:

        “There’s not much to say” about _default/_working individually beyond the above invariants.

        You’re open to not defining classes yet, but you feel their permanent presence in the mesh might justify them being explicit resources.

Those are all coherent, but they don’t all force you to introduce OWL classes.
2. Existence in the mesh ≠ automatic need for an OWL class

Key point:

You’re conflating “has an IRI / is a node in the mesh” with “deserves its own RDFS/OWL class”.

You already have many things that “exist in the mesh” that are not separate OWL classes:

    The actual distribution files for each snapshot.

    The index.html ResourcePage under each NamingResource.

    Any random non-RDF assets in an AssetTree.

All of those:

    Have IRIs,

    Are dereferenceable,

    May be documented in metadata,

But you don’t create separate classes like LatestTarballRootIndexDistribution for each one.

You reserve OWL classes for things where:

    You need to express invariants that matter for reasoning / validation, or

    You want consumers to be able to query/find them by type rather than by path convention.

By your own description, _default and _working are:

    Structural conventions (“these folders always exist here”),

    Not conceptually different datasets from snapshots (for _default), and

    Not authoritative datasets at all (for _working).

That is not a strong case for separate Snapshot subclasses.
3. _default as copy, not pointer: what that really means

You’re right: physically, _default is a copy. That has consequences:

    You have duplicated bytes on disk.

    You can dereference _default without knowing which versioned snapshot it mirrors.

But conceptually, it is the same dataset state as the latest snapshot. So you have two options:

    Treat _default as a separate dataset

        Then you have to reconcile:

            Is it a new snapshot in the chain?

            Does it have its own previousSnapshot?

            Does it get its own provenance activity separate from the “real” snapshot?

        That quickly becomes nonsense: every weave would create “real snapshot S_n” and “mirror snapshot S_n-default” for no benefit.

    Treat _default as a projection of an existing snapshot dataset

        On disk: copy of the same files.

        In RDF:

            The dataset identity is the snapshot’s IRI.

            _default is a convenience mount point for that dataset.

        If you want to talk about it at all, it’s a FileResource/Distribution, not a separate Snapshot.

Given your design, 2 is the only defensible story.

If you later want to say “this distribution is currently the default one for this flow”, that’s:

    A property on the Flow (or the Snapshot) pointing at a particular Distribution,

    Not a separate Snapshot class.

4. _working as “permanent but non-authoritative”

You’ve made _working more “real” than I assumed:

    It always exists.

    After weave, it is reset to match _default/latest snapshot.

    It may get its own ResourcePage.

Notice what that implies:

    Over time, _working cycles between:

        “equals current snapshot” (right after weave),

        “dirty state” (while user edits),

        “equals current snapshot again” (after next weave completes).

No matter how often it changes, it never becomes an authoritative historical state. It’s always “current editing buffer”.

That is exactly the kind of thing you do not want to reify in the same vocabulary as immutable snapshots, because:

    It will never appear in a correct “as-of” view.

    It is inherently specific to the clone / local working copy.

    Its existence is an implementation detail of authoring, not of the mesh’s committed history.

Given that, the clean story is:

    _working is a FileResource (and maybe has a ResourcePage) that your tools understand as “staging area”.

    It is not a Snapshot and should not be addressed with snapshot semantics.

If you want a class at all, it should be:

    Something like WorkingDirectory ⊑ AssetTree or FileResource, not a Snapshot.

But again: you have no constraints or reasoning needs around it yet, so that class would be pure taxonomy bloat.
5. Where to put the semantics you do care about

Things you actually care about formally:

    There is exactly one latest snapshot per Flow.

    Snapshots form a linear chain via previousSnapshot.

    Weaves update:

        latest snapshot,

        _default copy,

        _working reset.

Those semantics sit cleanly in three places:

    sflo ontology

        sflo:Snapshot (class)

        sflo:previousSnapshot (Snapshot → Snapshot, max 1)

        sflo:currentSnapshot (Flow → Snapshot, max 1)

        Optionally weave label + sequence number on Snapshot

    meta-flow ontology

        MetaWeave, PayloadWeave, etc.

        prov:wasGeneratedBy (Snapshot → WeaveActivity)

        meta:forSnapshot (ProvenanceContext → Snapshot)

    Implementation invariants (documented, not modeled)

        _default/ directory must always mirror sflo:currentSnapshot.

        _working/ directory must be copied from latest snapshot on successful weave.

        _default and _working exist for every Flow in the file layout.

There is nothing in there that requires a DefaultSnapshot or WorkingSnapshot class.
6. Answering your “what’s the objection?” directly

    But yes, maybe _default and _working are "SpecialSnapshots"? What's the objection to hasWorkingSnapshot?

Sharp answer:

    Because _working is not a snapshot in your own terms:

        It is mutable.

        It is not historical.

        It is clone-local staging.

    Calling it a Snapshot (via hasWorkingSnapshot) invites consumers to treat it as historical, resolvable state, which it is not.

    I definitely feel like there's not much to say about specific _default and _working, RDF-wise, but they exist in the mesh (permanently, and every flow has them, always) so that's almost reason enough to have classes for them.

Counterpoint:

    OWL classes should represent distinctions you need to reason about.

    Right now, all of your interesting invariants revolve around:

        immutable snapshots and their ordering,

        which snapshot is current,

        how snapshots are produced.

    _default and _working are just two conventional directory names that your implementation uses to expose those invariants.

Existence + permanence alone is not a sufficient reason for a class; otherwise you’d have classes for every directory name in the layout.
7. Concrete recommendation (short)

Given what you just clarified, the least-wrong, least-regret path is:

    In the sflo ontology:

        Keep a single Snapshot class.

        Define:

            sflo:previousSnapshot (Snapshot → Snapshot, max 1)

            sflo:currentSnapshot (Flow → Snapshot, max 1)

        Drop DefaultSnapshot, VersionSnapshot, hasWorkingSnapshot, etc.

    Do not model _default or _working as Snapshot subclasses.

        Treat them as layout conventions.

        If/when you need them in RDF:

            Model them as FileResource (or a sub-class) attachable to the Flow, not as Snapshot.

    Document in prose (and maybe SHACL on the filesystem-metadata graph, if you ever have one):

        Every Flow has:

            A _default/ directory mirroring sflo:currentSnapshot.

            A _working/ directory used as staging and reset on weave.

    Defer introducing DefaultSnapshot/WorkingSnapshot classes until you hit a concrete use case where:

        Some reasoning or validation depends on “is this default vs not”, and

        It can’t be expressed as a property (sflo:currentSnapshot, sflo:role) on existing Snapshot/Flow classes.

Right now, you’re paying ontological complexity for zero functional gain. I would lock the version-chain model in sflo + meta-flow, and keep _default/_working as implementation-level conventions until a real semantic need surfaces.
You said:
Actually, I didn't intend to have anything in the mesh that didn't have an OWL class except for the random resources the can be contained in an AssetTree. Distribution files are SnapshotDistribution, the index.html are ResourcePage. 

_default and _working are not SnapshotDistributions, but they contain SnapshotDistributions. They are definitely not FileResource because only "information resources" (downloadable files,  with an extension) are FileResource. They can be considered NamingResource I suppose.)


But yes, you've convinced me for now.

(Note that I'm calling my distributions SnapshotDistributions to delineate them from dcat:Distributions and potentially sflo:AggregateDistribution - what do you think, should I just call them Distribution?)

ChatGPT said:

Good, that helps a lot; the “everything has an OWL class” rule changes the calculus.

Given that:

    Distribution files are SnapshotDistribution

    index.html are ResourcePage

    _default / _working are (or will be) NamingResource-ish containers of those

the real question is: do you want SnapshotDistribution to be the distribution class, or a specialization of a more general Distribution?
1. Conceptually, you’re mixing two layers

Right now your wording suggests:

    Distribution files are SnapshotDistribution.

But DCAT’s dcat:Distribution is not “the file”; it’s the abstract distribution that then has:

    dcat:downloadURL / dcat:accessURL to the file(s),

    format, media type, etc.

You’re compressing:

    “abstract distribution” and

    “concrete file”

into one class (SnapshotDistribution).

That’s acceptable as a simplification, but then the name “SnapshotDistribution” is doing double duty:

    distribution-of-a-snapshot, and

    distribution-as-a-file.

If you later introduce sflo:AggregateDistribution (bundle of many snapshots), they are still both distributions in the DCAT sense.

That’s the main reason I would not make SnapshotDistribution the top class.
2. Recommended pattern

I’d do this:

sflo:Distribution a owl:Class ;
    rdfs:subClassOf dcat:Distribution , sflo:FileResource ;
    rdfs:label "Distribution" ;
    rdfs:comment "Concrete distribution of a dataset in the mesh (usually a single file), aligned with dcat:Distribution." .

sflo:SnapshotDistribution a owl:Class ;
    rdfs:subClassOf sflo:Distribution ;
    rdfs:label "Snapshot distribution" ;
    rdfs:comment "Distribution that carries the data for exactly one Snapshot dataset version." .

sflo:AggregateDistribution a owl:Class ;
    rdfs:subClassOf sflo:Distribution ;
    rdfs:label "Aggregate distribution" ;
    rdfs:comment "Distribution that aggregates multiple datasets or snapshot versions in a single package (e.g. tarball of all versions)." .

Notes:

    sflo:Distribution is your generic distribution.

    SnapshotDistribution is explicitly “this is for one snapshot”.

    AggregateDistribution is explicitly “this bundles multiple things”.

Yes, you’re still collapsing file + distribution, but the hierarchy is honest about the conceptual relationships.

If you feel strongly that only “information resources with an extension” are FileResource, you can drop that superclass and simply document that sflo:Distribution are always files in practice.
3. Answering your direct question

    Should I just call them Distribution?

My recommendation:

    Yes: introduce sflo:Distribution as the primary class.

    Keep sflo:SnapshotDistribution as a subclass if you genuinely distinguish “single-snapshot distribution” from other forms (e.g. aggregates, multi-graph dumps).

    Do not use SnapshotDistribution as the only or top-level distribution class.

This gives you:

    A clean place to hang any future common constraints (format, media type, SHA-256, etc.): sflo:Distribution.

    Room for AggregateDistribution without violating the conceptual meaning of the base class.

    Less awkwardness when you inevitably have distributions that aren’t literally “for a snapshot only” (e.g. “all flows for this node as one file”, “all metadata flows for this mesh”, etc.).

You said:
OOo, good catch on dcat:Distribution. So probably snapshot is a distribution. But assuming we want to support multiple files per distribution, in different formats, we might need a new concept under snapshot that is a naming resource that refers to a particular file?
ChatGPT said:

Short answer: you don’t strictly need a new concept under Snapshot; you either:

    Treat each file as its own Distribution (DCAT-style, simplest), or

    Introduce SnapshotDistribution as a NamingResource that owns multiple FileResources (if you really want grouping).

Let me walk through the tradeoffs.
1. What DCAT actually buys you

In DCAT:

    dcat:Dataset = abstract dataset.

    dcat:Distribution = one concrete way to access that dataset.

    The “file” is usually represented via dcat:downloadURL or dcat:accessURL.

Typical pattern:

    One dataset

        Distribution 1: Turtle file

        Distribution 2: JSON-LD file

        Distribution 3: ZIP archive, etc.

So multiple formats = multiple Distributions, not multiple files under one Distribution.

If you align with that, your mesh can be:

    sflo:Snapshot ⊑ dcat:Dataset

    For each concrete data file:

        A sflo:Distribution ⊑ dcat:Distribution ∧ FileResource

        sflo:distributionOf (or reuse dcat:distribution) links Snapshot → Distribution

No extra “naming resource under snapshot” is needed; the “distribution” node is the thing that points to the concrete file.
2. If you really want “multi-file per distribution”

You said:

    assuming we want to support multiple files per distribution, in different formats, we might need a new concept under snapshot that is a naming resource that refers to a particular file?

If you insist that a single “SnapshotDistribution” should group multiple files (e.g. .ttl, .jsonld, maybe .trig), then a clean mesh-aligned model is:

    sflo:Snapshot (Dataset, NamingResource)

    sflo:SnapshotDistribution (NamingResource, dcat:Distribution)

        one per bundle of representations of that snapshot

    sflo:DistributionFile (or SnapshotFile) ⊑ FileResource

        one per actual file: foo.ttl, foo.jsonld, etc.

Links:

    sflo:hasDistribution (Snapshot → SnapshotDistribution)

    sflo:hasFile (SnapshotDistribution → DistributionFile)

And on the filesystem:

/ns/foo/_payload-flow/2025-11-24_0142_v3/
  distribution/              # SnapshotDistribution (NamingResource)
    foo.ttl                 # DistributionFile (FileResource)
    foo.jsonld              # DistributionFile (FileResource)

Pros:

    Keeps your “everything has an OWL class” rule intact:

        Snapshot = dataset

        SnapshotDistribution = bundle for that dataset

        DistributionFile = concrete file

    Lets you express metadata at both levels:

        On the bundle (formats offered, “canonical” URL),

        On individual files (checksum, size, exact media type, etc.).

Cons:

    You are deliberately deviating from DCAT’s “one format = one Distribution” norm.

    Tools expecting vanilla DCAT may misinterpret or ignore your grouping.

3. Recommendation

Given your goals and your “class for everything except pure junk in AssetTree” rule:

    Decide if you actually need multi-file per distribution.

        If you’re happy with “TTL and JSON-LD are separate Distributions”, do the DCAT-ish thing:

            sflo:Distribution ⊑ dcat:Distribution ∧ FileResource

            One per file; Snapshot → Distribution via dcat:distribution or sflo:hasDistribution.

        If you really want a “distribution that is a group of files”:

            Introduce:

            sflo:SnapshotDistribution ⊑ dcat:Distribution , NamingResource .
            sflo:DistributionFile ⊑ FileResource .
            sflo:hasDistribution  (Snapshot → SnapshotDistribution).
            sflo:hasFile          (SnapshotDistribution → DistributionFile).

            Keep your “SnapshotDistribution contains SnapshotDistributions” comment aligned: Snapshot doesn’t directly contain files; the Distribution does.

    Do not put a new concept directly under Snapshot just to refer to a particular file.
    Either:

        Treat the file itself as a dcat:Distribution (simplest, DCAT-aligned), or

        Use SnapshotDistribution (NamingResource) → DistributionFile (FileResource) as the two-level structure.

Right now, your instinct (“we might need a new concept under snapshot that is a naming resource that refers to a particular file”) is a bit off: the “naming resource that refers to a particular file” is either the Distribution itself or a FileResource; you don’t need a third kind unless you’re grouping multiple files.
You said:
let's definitely stick with one format = one distribution. the snapshot (just Snapshot, not SnapshotDistribution) is a group of files. 

If we have DistributionFile, what is the IRI? Could it just be the URL to the file?
ChatGPT said:

You’ve actually just simplified things enough that you probably don’t need DistributionFile at all.

Given:

    One format = one distribution

    Snapshot is the group (the dataset)

    Only “information resources with an extension” are FileResource

The cleanest model is:

    Snapshot ⊑ dcat:Dataset ∧ NamingResource

    Each concrete file (TTL, JSON-LD, TRIG, …) is a:

sflo:Distribution
    a dcat:Distribution , sflo:FileResource .

The IRI of that sflo:Distribution is the file URL itself:

<https://example.org/ns/foo/_payload-flow/2025-11-24_0142_v3/foo.ttl>
    a sflo:Distribution ;
    dcat:mediaType "text/turtle" ;
    dcterms:format <https://www.iana.org/assignments/media-types/text/turtle> ;
    dcat:downloadURL <https://example.org/ns/foo/_payload-flow/2025-11-24_0142_v3/foo.ttl> ;
    # plus whatever checksum/size/etc.

Snapshot → distribution:

    <https://example.org/ns/foo/_payload-flow/2025-11-24_0142_v3/>
        a sflo:Snapshot ;
        dcat:distribution <https://example.org/ns/foo/_payload-flow/2025-11-24_0142_v3/foo.ttl> ,
                          <https://example.org/ns/foo/_payload-flow/2025-11-24_0142_v3/foo.jsonld> .

So:

    You can introduce DistributionFile as another class, but if each distribution is exactly one file, then “distribution” and “file” are the same node:

        IRI = file URL

        Types: sflo:Distribution, dcat:Distribution, FileResource

Introducing a separate DistributionFile node only makes sense if:

    A single Distribution can cover multiple files, or

    You want to model some indirect access pattern (symbolic link, content negotiation, etc.).

You explicitly said “one format = one distribution” and “snapshot is the group of files,” so the least-friction, most honest modeling is:

    No DistributionFile class.

    Each distribution’s IRI is the URL to the file.

You said:
OK, clear to me. A distribution corresponds to a single file and is a file resource. I actually love how you can use a file url for semantically sensible purposes. 
ChatGPT said:

Good. Then your ontology should reflect exactly that, explicitly.

Something like:

sflo:Distribution a owl:Class ;
    rdfs:subClassOf dcat:Distribution , sflo:FileResource ;
    rdfs:label "Distribution" ;
    rdfs:comment "A single downloadable file that provides one concrete representation (format) of a Snapshot dataset." .

And your pattern becomes:

    sflo:Snapshot ⊑ dcat:Dataset ∧ NamingResource.

    Each on-disk data file:

        IRI = file URL.

        Types: sflo:Distribution, dcat:Distribution, sflo:FileResource.

    Link with dcat:distribution (or sflo:hasDistribution if you want a subproperty):

sflo:hasDistribution a owl:ObjectProperty ;
    rdfs:subPropertyOf dcat:distribution ;
    rdfs:domain sflo:Snapshot ;
    rdfs:range  sflo:Distribution ;
    rdfs:label "has distribution" .

Next concrete step I’d suggest: add those three declarations (class + property + comment) to the sflo ontology and remove any leftover “SnapshotDistribution” mentions so you don’t end up with ghost concepts from the previous iteration.
You said:
Any changes to my task?
ChatGPT said:

Short answer: the core of the task (weave labels, snapshot naming, previousSnapshot chain) does not need to change.

What does need updating is:

    Any mention of DefaultSnapshot / WorkingSnapshot classes.

    Any mention of SnapshotDistribution.

    The ontology deliverables around distributions.

I’ll be explicit.
1. Snapshot / default / working

Your current design (as of this thread) is:

    Exactly one Snapshot class.

    A linear chain via something like sflo:previousSnapshot.

    A notion of “current snapshot” that _default/ mirrors.

    _working/ is staging, never a snapshot.

So in your task doc, check for and change:

    If you still refer to:

        DefaultSnapshot / VersionSnapshot

        hasDefaultSnapshot / hasWorkingSnapshot / hasVersionSnapshot

    Replace with:

        sflo:Snapshot (single class)

        sflo:previousSnapshot (Snapshot → Snapshot, max 1)

        sflo:currentSnapshot (Flow → Snapshot, max 1)

If your task mentions “model _default as a DefaultSnapshot” or “hasWorkingSnapshot”, those lines are now wrong and should be removed or rewritten to match:

    _default/ is a directory that mirrors whatever sflo:currentSnapshot points at,

    _working/ is a directory used for staging and is not modeled in RDF at all.

You don’t have to add new work to the task, just correct the ontology section to not promise classes you no longer want.
2. versionKey vs weave label

Anywhere in the task that talks about:

    versionKey (decimal encoding of timestamp)

should now be:

    sflo:weaveLabel (string, e.g. 2025-11-24_0142a)

    sflo:sequenceNumber (per-flow integer)

You already implicitly made that shift; make sure the task’s “Ontology and SHACL updates” section reflects that (and does not ask you to define versionKey anymore).
3. Distributions

Your latest decisions:

    Snapshot is the dataset (group of files).

    One format = one distribution.

    Each on-disk file is:

        A sflo:Distribution

        A dcat:Distribution

        A sflo:FileResource

    The IRI of the distribution is the file URL.

So in the task’s Deliverables / Ontology updates, you should add (or update) to require:

    sflo:Distribution class:

        sflo:Distribution ⊑ dcat:Distribution , sflo:FileResource

    A property from Snapshot to distributions:

        Either reuse dcat:distribution, or define:

        sflo:hasDistribution ⊑ dcat:distribution
        domain: sflo:Snapshot
        range:  sflo:Distribution

And remove any references to:

    SnapshotDistribution (if it appears in the task),

    “multi-file per distribution” semantics.

The task’s main behavior (weave labels, snapshot folders, manifests / as-of selection, etc.) doesn’t change because of this, but the ontology bullet list does.
4. sflo vs meta-flow split

One last consistency tweak for the task:

    Anything about:

        Snapshot identity, previousSnapshot chain, current snapshot, weave label / sequence number

        Belongs in sflo (core ontology).

    Anything about:

        WeaveActivity, MetaWeave, delegation, agent roles, provenance context

        Belongs in meta-flow.

So if the task currently says “add hasPreviousSnapshot / hasDefaultSnapshot / hasWorkingSnapshot to meta-flow ontology”, that should be corrected:

    Core snapshot graph and properties in sflo.

    Activity/provenance shapes and properties in meta-flow.

If you paste the exact current version of the task, I can give you a diff-style patch, but based on what you’ve shared so far, those are the only substantive changes required. The high-level behavior you described in the task still matches what you want to build.
You said:
Please generate jsonld for weaveLabel and sequenceNumber. 

Right now it seems like meta-flow mostly has provenance. And lots of stuff that's going to be kept in the metadata flow (like weaveLabel, sequenceNumber) is in the main ontology. Maybe we rename meta-flow ontology to sflo-prov or something like that?
ChatGPT said:

Here’s JSON-LD for weaveLabel and sequenceNumber that matches your current ontology style and decisions.
1. JSON-LD for weaveLabel and sequenceNumber

Drop these into the @graph of your main sflo ontology (same place you defined versionKey/, etc.):

{
  "@id": "weaveLabel/",
  "@type": "owl:DatatypeProperty",
  "schema:domainIncludes": [
    { "@id": "Flow/" },
    { "@id": "Snapshot/" }
  ],
  "schema:rangeIncludes": { "@id": "xsd:string" },
  "rdfs:label": "weave label",
  "rdfs:comment": "Human-readable label for a weave run (e.g., '2025-11-24_0142' or '2025-11-24_0142a'). All snapshots created by the same weave share the same weaveLabel. Not guaranteed to be globally unique across all meshes."
},
{
  "@id": "sequenceNumber/",
  "@type": "owl:DatatypeProperty",
  "schema:domainIncludes": { "@id": "Snapshot/" },
  "schema:rangeIncludes": { "@id": "xsd:integer" },
  "rdfs:label": "sequence number",
  "rdfs:comment": "Monotonic integer identifying this snapshot within its Flow. 1 for the first committed snapshot of a flow, 2 for the next, etc. Used for ordering and for detecting gaps or compaction at the flow level."
}

Notes / implications (just to make sure the semantics are pinned down):

    weaveLabel/

        On Flow/: “this flow was last woven in weave X”.

        On Snapshot/: “this snapshot was created in weave X”.

        You can choose to only materially attach it to Snapshot/ if you want, but keeping it allowed on Flow/ gives you flexibility (e.g., metadata flow summarizing “this node was last woven in X”).

    sequenceNumber/

        Only on Snapshot/ (not Flow/), since you’ve decided ordinality is per-snapshot-per-flow.

        Implementation invariant (not in the ontology, but you should document it): sequenceNumber is scoped to a (Flow, Snapshot) chain, not global.

If you ever want SHACL, the obvious shape is:

{
  "@id": "SnapshotSequenceNumberShape/",
  "@type": "sh:PropertyShape",
  "sh:targetClass": { "@id": "Snapshot/" },
  "sh:path": { "@id": "sequenceNumber/" },
  "sh:datatype": { "@id": "xsd:integer" },
  "sh:minCount": 1,
  "sh:maxCount": 1,
  "sh:message": "Snapshots must have exactly one integer sequenceNumber."
}

(But you didn’t ask for shapes right now.)
2. Where these live vs. meta-flow

You’re right about the split:

    weaveLabel and sequenceNumber are core mesh/flow versioning semantics.

        They belong in the main sflo ontology (the one you just pasted).

        Their ABox lives in the metadata flow datasets for each node/flow.

    The current “meta-flow” ontology is almost entirely provenance:

        Activities (WeaveActivity, NodeWeave, MetaWeave, etc.).

        Delegation, agent roles.

        Provenance contexts.

Those really are a provenance schema for sflo, not “generic metadata”.

So, yes, renaming meta-flow to something like sflo-prov (or sfprov) is more honest:

    It clarifies that:

        Versioning core (flows, snapshots, labels, sequence numbers, previousSnapshot) = sflo.

        Who/what/when/why (activities, agents, delegation, attribution) = sflo provenance.

If you rename:

    Pick a stable base, e.g.:

        https://semantic-flow.github.io/ontology/sflo-prov/

        Prefix: sfprov or sflo-prov.

    Update:

        The ontology IRI (owl:Ontology node).

        The context in your JSON-LD ("sfprov": "…") instead of "meta":.

        References in other ontologies: meta:ProvenanceContext/ → sfprov:ProvenanceContext/, etc.

You should do this before you publish anything you care about dereferencing, to avoid later redirect/alias headaches.

But the key structural point is: you were correct to keep weaveLabel and sequenceNumber in the main ontology; the provenance ontology is “how we record who/when/why this label/number was produced,” not where the basic versioning vocabulary lives.
You said:
Oh, crap, I think we have to re-address _default. If a flow is unversioned, when _working gets woven, it only copies it to _current, no weaveLabel'd snapshot.
ChatGPT said:

Right, this does break the “every weave creates a Snapshot and _default just mirrors it” story. You now have two distinct behaviors:

    versioned flows: weave → new Snapshot + _default mirror

    unversioned flows: weave → _default only, no Snapshot at all

We don’t have to blow up the model, but we do need to make _default explicitly versioning-mode-dependent.

I’ll walk through the implications and tell you exactly what to change in the ontology/constraints.
1. Clarify the behaviors you just implied

I’m going to normalize names and call it _default (you wrote _current once, which looks like a slip).

For a given Flow:
If versioningState = versioned

On weave:

    Read _working/.

    Write new immutable snapshot folder (e.g. 2025-11-24_0142_v7/).

    Copy that snapshot’s distributions into _default/ (mirror).

    Reset _working/ from _default/.

RDF-wise:

    Create a new Snapshot instance:

        sequenceNumber = 7

        weaveLabel = "2025-11-24_0142" (or whatever)

    Set sflo:currentSnapshot on the Flow to that Snapshot.

    _default/ is just how that snapshot gets mounted.

If versioningState = unversioned

On weave:

    Read _working/.

    Copy it into _default/ (or mutate _default/ in place).

    Reset _working/ from _default/.

    No new snapshot folder created.

RDF-wise:

    No new Snapshot instance.

    No sflo:currentSnapshot change (because there is none).

    You might still record:

        latest weaveLabel on the Flow itself,

        provenance (WeaveActivity, agents, etc.) in the metadata flow.

So _default is doing double duty:

    For versioned flows: “mirror of latest Snapshot”.

    For unversioned flows: “the dataset itself”.

That’s the core tension you just uncovered.
2. What this means for _default as a class (or not)

The good news: this doesn’t force you to add a DefaultSnapshot class or anything like that.

What it really forces is:

    You cannot write ontology/SHACL that assumes:

        Every Flow has at least one Snapshot.

        Every Flow has a sflo:currentSnapshot.

Instead, you need:

    Conditional constraints based on versioningState:

        If versioningState = versioned:

            Flow must have:

                at least one Snapshot,

                exactly one sflo:currentSnapshot.

        If versioningState = unversioned:

            Flow must not have Snapshot instances (or at least not new ones).

            Flow must not have sflo:currentSnapshot.

        If versioningState = deversioned:

            Legacy Snapshots may exist.

            No new ones created; sflo:currentSnapshot probably frozen or dropped.

So _default stays in the file-system / layout domain:

    Always exists.

    For versioned: “mirror of current Snapshot”.

    For unversioned: “only dataset location”.

No separate OWL class required; the distinction lives in versioningState + presence/absence of Snapshots.
3. Changes you should make in the ontology / constraints
3.1 Versioning state must gate snapshots

You already have a VersioningState enum and versioningState/ property.

You now need to encode:

    “Snapshots are only valid when versioningState is versioned or deversioned”

    “currentSnapshot exists only when versioningState is versioned”

In SHACL terms (rough sketch, not full JSON-LD):

    Flow shape:

        Constraint: if versioningState = "versioned":

            sflo:currentSnapshot minCount = 1, maxCount = 1.

            There exists at least one Snapshot with sequenceNumber >= 1 for that Flow.

        If versioningState = "unversioned":

            sflo:currentSnapshot maxCount = 0.

            There SHOULD NOT be any new Snapshots created after you mark it unversioned.

You can implement this:

    Either in SHACL with some SPARQL conditions, or

    In code as invariants and keep SHACL simpler.

But the design must treat Snapshots as versioned-only features, not baseline features of every Flow.
3.2 Flow → Snapshot properties must become optional

Any ontology text or shapes that currently say or imply:

    “Every Flow has sflo:currentSnapshot”

    “Every Flow has hasSnapshot”

has to be weakened to:

    “Flows may have snapshots; only versioned ones must.”

    Use conditional language in comments, and avoid global shapes that enforce presence across the board.

3.3 Meta-flow (sflo-prov) can still assume weaveLabel exists even without snapshots

Your earlier design for weaveLabel and sequenceNumber:

    weaveLabel/ domainIncludes: Flow/, Snapshot/.

    sequenceNumber/ domainIncludes: Snapshot/ only.

That actually aged well:

    Versioned Flow:

        weaveLabel on Flow + Snapshots.

        sequenceNumber on Snapshots.

    Unversioned Flow:

        weaveLabel on Flow only.

        No Snapshots, no sequenceNumber.

So you don’t have to change those properties. Just be explicit in the comments:

    “Unversioned flows may have weaveLabel but no sequenceNumber and no snapshots.”

4. What you do not need to change

You do not need to reintroduce DefaultSnapshot or any class for _default.

The behavior for _default is now:

    It always exists on disk.

    For versioned flows, its content mirrors the sflo:currentSnapshot dataset.

    For unversioned flows, it is the authoritative dataset; there is no Snapshot to mirror.

That’s still entirely expressible without a special class:

    For versioned flows:

        You can derive which snapshot _default should mirror from sflo:currentSnapshot.

    For unversioned flows:

        There is no snapshot, but clients know “if versioningState = unversioned, read _default directly”.

The ontological model remains:

    Flow (always)

    Optional Snapshots + sflo:currentSnapshot (versioned/deversioned only)

    _default is a layout convention, not a separate semantic kind.

5. One thing you should think about but haven’t yet

You now have a transition problem:

    What happens when a Flow moves from:

        unversioned → versioned?

        versioned → deversioned?

You need to decide:

    When switching unversioned → versioned:

        Do you immediately create the first Snapshot from the existing _default?

        Does sequenceNumber = 1 correspond to “the first weave after the switch”, or “retro snapshot of current state at switch time”?

    When switching versioned → deversioned:

        Do you keep old Snapshots as read-only history?

        Does sflo:currentSnapshot remain, or get cleared?

        Do subsequent weaves ignore the snapshot chain entirely and only update _default?

Those transitions matter because they determine:

    How “as-of” queries behave across the boundary.

    Whether people can rely on “there is a snapshot representing the state at the time we turned versioning on/off”.

You don’t have to solve it right this second, but you shouldn’t finalize shapes/ontology text until you’ve picked a story there.
TL;DR

    You’re right to flag this: unversioned flows do not create Snapshots, so _default is the only committed dataset there.

    You don’t need a DefaultSnapshot class.

    You do need:

        Snapshot-related constraints gated by versioningState,

        Documentation that _default has different semantics in versioned vs unversioned flows,

        A decision about state transitions between versioning modes.

You said:
Flows are dcat:DataSetSeries. Unversioned flows still have to have a dcat:Dataset. If not a Snapshot, _current has to be referrable to, and it should be at least a dcat:Dataset. 

I kinda think _current should just be a Snapshot. 
ChatGPT said:

You’ve tripped over a real inconsistency, so good catch. Let’s pin it down instead of hand-waving.

You’re asserting all of this at once:

    Flow ⊑ dcat:DatasetSeries.

    Even unversioned flows must have at least one dcat:Dataset.

    For unversioned flows, weaving only updates _current (no timestamped snapshot dirs).

    You’ve been using Snapshot to mean “distinct state (ideally immutable) of a flow’s dataset.”

    Now you “kinda think _current should just be a Snapshot.”

Those don’t line up cleanly unless you relax what “Snapshot” means.

I’ll walk through the options and give you a clear recommendation.
1. What DCAT actually forces

If Flow is a dcat:DatasetSeries, DCAT wants:

    A series (Flow) has members that are dcat:Datasets.

    Whether there is 1 or 20 datasets doesn’t matter; a series with 1 dataset is still valid.

So for unversioned flows you still need:

    At least one dcat:Dataset node.

    A link like dcat:hasPart (or your own subproperty) from the Flow to that Dataset.

That dataset is “whatever lives under _current”.

So we need a class + property pattern for:

sflo:Flow a dcat:DatasetSeries .

<…/_current/> a dcat:Dataset (and some sflo:* class) .

sflo:hasMemberDataset (or dcat:hasPart) : Flow → Dataset .

The conflict is not with DCAT; it’s with your semantics for Snapshot.
2. If _current is a Snapshot, what breaks?

If you simply say:

    “Fine, _current is a Snapshot.”

you have to accept one of these:

    Snapshots are not necessarily immutable.
    Then:

        For unversioned flows:

            There is one Snapshot whose content changes on every weave.

        For versioned flows:

            You also have timestamped Snapshots that are treated as immutable by convention.

    You’ve diluted “snapshot” from “frozen state” to “whatever state we’re currently in.”
    Any consumer that assumed Snapshots are immutable will be misled by _current.

    Or you pretend _current is immutable and ignore that you overwrite it on every weave.
    That’s just lying in RDF.

Neither is great. If you’re going to keep “Snapshot = commit-like state”, then _current for unversioned flows is not a snapshot.

You can of course weaken the Snapshot comment to:

    “Represents a dataset state; for versioned flows these are treated as immutable historical states; for unversioned flows a flow might have only one Snapshot whose content can change.”

But at that point, “Snapshot” no longer carries the strong semantic guarantees you wanted.
3. Cleaner pattern: separate “current dataset” from “snapshot”

If you want to preserve the intuitive meaning of Snapshot (immutable, commit-like) and satisfy DCAT and model unversioned flows cleanly, the least self-contradictory pattern is:
3.1 Classes

Add one new class:

sflo:DatasetState a owl:Class ;
    rdfs:subClassOf dcat:Dataset , Resource/ ;
    rdfs:label "Dataset State" ;
    rdfs:comment "Any dataset state belonging to a Flow: current live dataset for unversioned flows, or immutable snapshots for versioned flows." .

sflo:Snapshot a owl:Class ;
    rdfs:subClassOf sflo:DatasetState ;
    rdfs:label "Snapshot" ;
    rdfs:comment "Immutable, committed dataset state belonging to a Flow. Versioned flows have one Snapshot per weave; unversioned flows may have none." .

sflo:CurrentDataset a owl:Class ;
    rdfs:subClassOf sflo:DatasetState ;
    rdfs:label "Current dataset" ;
    rdfs:comment "The live dataset state for a Flow. For unversioned flows, this is the only dataset; for versioned flows, it normally mirrors the latest Snapshot." .

Then _current is always a CurrentDataset.
For versioned flows, that CurrentDataset mirrors the latest Snapshot’s content.
For unversioned flows, it’s the only DatasetState.
3.2 Properties

sflo:hasDatasetState a owl:ObjectProperty ;
    rdfs:domain Flow/ ;
    rdfs:range sflo:DatasetState ;
    rdfs:subPropertyOf dcat:hasPart ;
    rdfs:label "has dataset state" .

sflo:currentDataset a owl:ObjectProperty ;
    rdfs:domain Flow/ ;
    rdfs:range sflo:CurrentDataset ;
    rdfs:label "current dataset" ;
    rdfs:comment "The dataset state currently considered live for this Flow (mapped onto _current/ in the file layout)." .

sflo:hasSnapshot a owl:ObjectProperty ;
    rdfs:domain Flow/ ;
    rdfs:range sflo:Snapshot ;
    rdfs:label "has snapshot" ;
    rdfs:subPropertyOf sflo:hasDatasetState ;
    rdfs:comment "Associates a Flow with one of its immutable snapshot dataset states." .

Then:

    Unversioned flow:

        Has one CurrentDataset (at _current/).

        Has no Snapshots.

    Versioned flow:

        Has one CurrentDataset at _current/, which mirrors the latest Snapshot.

        Has N Snapshots at your timestamped dirs.

You can still have sequenceNumber on Snapshot only, which keeps the version chain clean.

This preserves:

    DCAT correctness (every member is a dcat:Dataset).

    “Snapshot = immutable commit” semantics.

    A clear, explicit place for _current without lying about immutability.

Yes, it’s one extra class, but it solves the semantic tangle instead of smearing it.
4. If you really insist on “_current is a Snapshot”

You can do it, but be honest about the consequences:

    You must weaken the class comment on Snapshot to not promise immutability.

    You must accept that for unversioned flows:

        There is exactly one Snapshot.

        Its content is overwritten on each weave.

        sequenceNumber either:

            stays the same (then it stops being a sequence), or

            changes on each weave (but there’s still only one physical folder _current/, which breaks the “folder per Snapshot” story).

If you go that way, you lose the clean invariant:

    “Each Snapshot maps to one immutable folder on disk.”

And any “as-of” or diff reasoning is meaningless for unversioned flows (which might be acceptable, but you should say it explicitly and not pretend otherwise).
5. Direct answer to your last line

    I kinda think _current should just be a Snapshot.

If you care about:

    clear semantics,

    future reasoning,

    and not surprising yourself later,

I’d push back and say:

    _current should be a CurrentDataset (a dcat:Dataset),

    Snapshot should stay reserved for immutable, commit-like datasets.

You can fold them and call everything Snapshot, but then “Snapshot” is just a fancy word for “dataset in a flow”, and you’ve thrown away the distinction you were using it to express.

If you want, I can give you JSON-LD for DatasetState, Snapshot, CurrentDataset, hasDatasetState, currentDataset, and hasSnapshot in the exact style of your current ontology so you can drop it in and remove a lot of the ambiguity.
You said:
Sorry, I meant _default. (Old names  die hard). Maybe we call it a FlowState class, and Snapshots are a subclass, Default is a subclass, and maybe Working is a subclass too.
ChatGPT said:

Good correction; this is specifically about _default, not _current.

Your instinct—introduce a FlowState umbrella and hang Snapshot, Default, maybe Working under it—is reasonable, but you need to be very explicit about what each subclass commits you to, especially w.r.t. DCAT and versioningState.

Given everything you’ve said so far, here’s the least-self-contradictory way to do it.
1. Semantics recap (with _default)

Per-flow, you now have:

    Versioned flow

        One _default/ directory: the “current” dataset, mirroring the latest snapshot.

        Many timestamped snapshot directories: immutable historical states.

        One _working/ directory: staging, reset on weave.

    Unversioned flow

        One _default/ directory: the only authoritative dataset location.

        No snapshot directories created on weave.

        One _working/ directory: staging, reset on weave.

DCAT constraints:

    Flow is a dcat:DatasetSeries.

    DatasetSeries must have dcat:Dataset members.

    In unversioned flows, that member can only be _default (since there are no snapshots).

So _default must be a dcat:Dataset in all modes.
Snapshots are dcat:Dataset in versioned/ deversioned mode.
_working is not a committed dataset; you probably don’t want it to be dcat:Dataset at all.

That leads directly to a FlowState pattern.
2. Recommended class structure

Introduce a FlowState umbrella, then specialize:

    FlowState = any dataset-like “state” that belongs to a Flow

    Snapshot = committed, immutable flow state

    DefaultFlowState = the “live” dataset mapped to _default/

    WorkingFlowState = staging state mapped to _working/ (optional, and not a dcat:Dataset)

2.1 FlowState

{
  "@id": "FlowState/",
  "@type": "owl:Class",
  "rdfs:subClassOf": { "@id": "Resource/" },
  "rdfs:label": "Flow State",
  "rdfs:comment": "Any dataset-like state belonging to a Flow: current default dataset, committed snapshots, or working state. FlowStates live under a Flow's directory in the mesh layout."
}

2.2 Snapshot (revised)

Replace your existing Snapshot/ class with:

{
  "@id": "Snapshot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowState/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Snapshot",
  "rdfs:comment": "Immutable, committed dataset state of a Flow (e.g., a timestamped weave folder). Versioned flows have one Snapshot per committed weave. Unversioned flows may have no Snapshots."
}

2.3 DefaultFlowState

{
  "@id": "DefaultFlowState/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowState/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Default Flow State",
  "rdfs:comment": "The current default dataset state for a Flow, materialized under the '_default/' directory. For versioned flows, its content mirrors the latest Snapshot. For unversioned flows, it is the only authoritative dataset state."
}

This makes _default/:

    A DefaultFlowState.

    A dcat:Dataset.

    A NamingResource (slash IRI, container for distributions).

2.4 WorkingFlowState (optional but consistent)

If you want _working/ to “exist in the model” without pretending it’s a committed dataset, do not make it a dcat:Dataset:

{
  "@id": "WorkingFlowState/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowState/" },
    { "@id": "NamingResource/" }
  ],
  "rdfs:label": "Working Flow State",
  "rdfs:comment": "Mutable staging state for a Flow, mapped to the '_working/' directory. Used for edits before weave. Not a committed dataset and not part of the version history."
}

This preserves:

    “Everything has a class” (you can type _working/),

    Without lying to DCAT consumers that _working is a proper dataset.

3. Linking Flow ↔ FlowState

You’ll also want an explicit property from Flow to its states:

{
  "@id": "hasFlowState/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasComponent/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "FlowState/" },
  "rdfs:label": "has FlowState",
  "rdfs:comment": "Associates a Flow with one of its state resources (default, snapshot, or working)."
},
{
  "@id": "defaultFlowState/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowState/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "DefaultFlowState/" },
  "rdfs:label": "default FlowState",
  "rdfs:comment": "Links a Flow to its default dataset state under '_default/'. There MUST be exactly one DefaultFlowState per Flow."
},
{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowState/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "Snapshot/" },
  "rdfs:label": "has Snapshot",
  "rdfs:comment": "Associates a Flow with one of its immutable snapshot states."
}

You can add a workingFlowState/ property analogously if you really want to surface _working/ in RDF.
4. How this interacts with versioningState

This structure plays nicely with your versioningState enum:

    versioned

        Flow:

            defaultFlowState = one DefaultFlowState (at _default/).

            One or more Snapshots.

        Invariant: DefaultFlowState’s content mirrors the latest Snapshot.

    unversioned

        Flow:

            defaultFlowState only.

            No Snapshots created on weave.

    deversioned

        Flow:

            May still have historical Snapshots (read-only).

            Future weaves only touch DefaultFlowState.

You can enforce the conditional bits in SHACL or just in implementation; the ontology is now expressive enough that you’re not forced into calling _default a Snapshot or pretending _working is a dataset.

If you want, next step would be to:

    Patch your existing Snapshot/ definition to this structure,

    Add FlowState/, DefaultFlowState/, and hasFlowState/ / defaultFlowState/ / hasSnapshot/,

    And then we can revise your “versioning” task text so it doesn’t accidentally talk in terms of “every weave creates a Snapshot in all modes.”

You said:
This echos a debate we had a while back. Conceptually, I'm on board. Naming, I have concerns. "FlowState" seems like it would describe the whole flow. We need something more like "FlowSlice" or ???
ChatGPT said:

You’re right to be suspicious of “FlowState” as a name; it sounds like “the whole flow’s state” rather than “one cut through it.”

You need a superclass that clearly means:

    “One concrete dataset instance that belongs to a Flow (default, snapshot, working), not the flow-as-a-series.”

1. Why “FlowSlice” is off

“Slice” usually implies:

    A subset of the data (projection/filter),

    Not “the entire dataset at a particular time.”

Your Snapshots / _default are full states, not slices. Using “slice” will come back to bite you when you introduce actual subsetting later (views, projections, partial exports).

So I’d avoid “FlowSlice” for what is really “full dataset at a time.”
2. A name that actually fits: FlowFrame

You already use cinematic / time-evolution metaphors (snapshot, weave) and separately chose “Frame” as the core type in Weave. That’s not an accident: it’s exactly the right abstraction:

    Flow = series (film, animation).

    Frame = one frame in that series.

    Snapshots, default, and working are all frames; some are committed, some are live/staging.

So:

    Superclass: FlowFrame

    Subclasses:

        Snapshot (committed frame)

        DefaultFlowFrame (the _default/ frame)

        WorkingFlowFrame (the _working/ frame, if you choose to model it)

That avoids the “sounds like the whole flow” problem of FlowState, and avoids the “subset” implication of Slice.
3. Concrete JSON-LD sketch

Here’s how that looks in your ontology style (drop into @graph of main sflo ontology, replacing the old Snapshot/ block and adding the others):

{
  "@id": "FlowFrame/",
  "@type": "owl:Class",
  "rdfs:subClassOf": { "@id": "Resource/" },
  "rdfs:label": "Flow Frame",
  "rdfs:comment": "One concrete dataset instance belonging to a Flow: a committed snapshot, the default frame under '_default/', or the mutable working frame under '_working/'. Frames reflect the Flow's data at a particular stage, not the Flow as a whole."
},
{
  "@id": "Snapshot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowFrame/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Snapshot",
  "rdfs:comment": "Immutable, committed flow frame (dataset state), typically materialized as a timestamped weave directory. Versioned flows have one Snapshot per committed weave; unversioned flows may have none."
},
{
  "@id": "DefaultFlowFrame/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowFrame/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Default Flow Frame",
  "rdfs:comment": "The Flow's default dataset frame under the '_default/' directory. For versioned flows, its content mirrors the latest Snapshot. For unversioned flows, it is the only authoritative dataset frame."
},
{
  "@id": "WorkingFlowFrame/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowFrame/" },
    { "@id": "NamingResource/" }
  ],
  "rdfs:label": "Working Flow Frame",
  "rdfs:comment": "Mutable staging frame for a Flow, mapped to the '_working/' directory. Used for edits before weave. Not a committed dataset and normally not exposed in version history or as-of queries."
},
{
  "@id": "hasFlowFrame/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasComponent/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "FlowFrame/" },
  "rdfs:label": "has FlowFrame",
  "rdfs:comment": "Associates a Flow with one of its frames (default, snapshot, or working)."
},
{
  "@id": "defaultFlowFrame/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowFrame/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "DefaultFlowFrame/" },
  "rdfs:label": "default FlowFrame",
  "rdfs:comment": "Links a Flow to its default frame under '_default/'. Each Flow must have exactly one DefaultFlowFrame."
},
{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowFrame/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "Snapshot/" },
  "rdfs:label": "has Snapshot",
  "rdfs:comment": "Associates a Flow with one of its committed snapshot frames."
}

You can keep your existing weaveLabel/ and sequenceNumber/ with Snapshot/ as their domain where appropriate; nothing in this renaming breaks them.

If you really hate “Frame” after staring at it a bit, the only other name that doesn’t mislead is something like DatasetState or FlowDatasetState, but those are clunkier and less aligned with your existing metaphors. “Frame” stays closest to what you actually mean: one temporal step in the series.
You said:
I like slice because there are physics metaphors about spacetime as a loaf, and you can have slices of that loaf. 

One problem with Frame, is that in weaving, the frame is the thing that holds the whole piece. In weaving, is a row of weft yarn called a pick. I think it can also be referred to as a shot. 

So we could have FlowShots, that include Snapshots, DefaultShot, and WorkingShot?
ChatGPT said:

Good, this actually gives you a clean way out of the naming mess, and keeps the weaving metaphor.

You’re trying to name:

    “One concrete dataset instance belonging to a Flow (_default/, a committed snapshot, or _working/), i.e., a temporal cross-section of the flow.”

Using weaving:

    The loom frame holds the whole piece (bad for a per-state concept).

    A shot / pick is one pass of weft: a discrete action, a step in time.

So treating each committed state as a “shot” of the flow is defensible.

I see two main concerns:

    Does “Shot” read too much like “gunshot / camera shot” to future devs?

    Does it accurately communicate “this is a full dataset state, not a subset”?

Given your preference and the metaphors you’re already using (“weave”), I think FlowShot is workable, as long as you’re precise in the comments.
1. Semantics I’d lock in

    FlowShot = “one concrete dataset instance of a Flow” (full state, not a slice/subset).

    Snapshot = committed, immutable FlowShot in a timestamped directory.

    DefaultShot = FlowShot under _default/:

        For versioned flows: mirror of the latest Snapshot.

        For unversioned flows: only authoritative dataset state.

    WorkingShot = mutable FlowShot under _working/:

        Staging; not a committed dataset; not necessarily dcat:Dataset.

So:

    Flow = dcat:DatasetSeries.

    FlowShot ⊑ Resource/.

    Snapshot, DefaultShot ⊑ FlowShot ∧ NamingResource/ ∧ dcat:Dataset.

    WorkingShot ⊑ FlowShot ∧ NamingResource/ (but not dcat:Dataset).

That keeps DCAT happy and your semantics honest.
2. JSON-LD for FlowShot + subclasses

Here’s a concrete block you can drop into your main ontology’s @graph, replacing the old Snapshot/ definition and adding the others:

{
  "@id": "FlowShot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": { "@id": "Resource/" },
  "rdfs:label": "Flow Shot",
  "rdfs:comment": "One concrete dataset instance belonging to a Flow: a committed snapshot, the default shot under '_default/', or the mutable working shot under '_working/'. A FlowShot represents the Flow's data as a whole at a particular stage, not a subset of it."
},
{
  "@id": "Snapshot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowShot/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Snapshot",
  "rdfs:comment": "Immutable, committed flow shot (dataset state), typically materialized as a timestamped weave directory. Versioned flows have one Snapshot per committed weave; unversioned flows may have none."
},
{
  "@id": "DefaultShot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowShot/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Default Shot",
  "rdfs:comment": "The Flow's default dataset shot under the '_default/' directory. For versioned flows, its content mirrors the latest Snapshot. For unversioned flows, it is the only authoritative dataset state."
},
{
  "@id": "WorkingShot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowShot/" },
    { "@id": "NamingResource/" }
  ],
  "rdfs:label": "Working Shot",
  "rdfs:comment": "Mutable staging shot for a Flow, mapped to the '_working/' directory. Used for edits before weave. Not a committed dataset and normally not exposed in version history or as-of queries."
},
{
  "@id": "hasFlowShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasComponent/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "FlowShot/" },
  "rdfs:label": "has FlowShot",
  "rdfs:comment": "Associates a Flow with one of its shots (default, snapshot, or working)."
},
{
  "@id": "defaultShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "DefaultShot/" },
  "rdfs:label": "default shot",
  "rdfs:comment": "Links a Flow to its default shot under '_default/'. Each Flow must have exactly one DefaultShot."
},
{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "Snapshot/" },
  "rdfs:label": "has Snapshot",
  "rdfs:comment": "Associates a Flow with one of its committed snapshot shots."
}

You can add workingShot/ as a property if you decide you do want to reference _working/ in RDF:

{
  "@id": "workingShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "WorkingShot/" },
  "rdfs:label": "working shot",
  "rdfs:comment": "Links a Flow to its mutable working shot under '_working/'. Typically used only for operational tooling, not long-term provenance."
}

3. Things you should clean up after this rename

    Anywhere in the ontology or task docs that currently says “FlowState” / “FlowFrame” → update to FlowShot.

    Confirm that:

        sequenceNumber/ has domain Snapshot/ (not all FlowShots).

        weaveLabel/ has domainIncludes [ Flow/, Snapshot/ ] (still fine).

    Be careful in prose to avoid implying that WorkingShot is immutable or a proper historical version.

As long as you keep “shot = full dataset state at a time” and “snapshot = committed shot,” this naming scheme is internally consistent and doesn’t force you to lie to DCAT or to yourself later.
You said:
I think we call WorkingShot a Dataset too... even if it has syntax errors, it supposedly starts as a Dataset after every Weave, it makes it back to a valid Dataset before the next weave. Isn't a Dataset in draft still a dataset?
ChatGPT said:

Yes, calling WorkingShot a dcat:Dataset is defensible, but it has consequences you should consciously accept.

Conceptually:

    DCAT doesn’t require immutability or validity; it’s about role (“this thing is a dataset”).

    A draft dataset with syntax/shape errors is still intended to be a dataset.

    So modeling WorkingShot as a dcat:Dataset is fine in principle.

Where you can get bitten is tooling and validation:

    Any SHACL shapes targeting dcat:Dataset will fire on WorkingShot too.

    If you treat SHACL violations as “this dataset is invalid,” you’ll constantly see WorkingShot flagged while someone is editing.

    So you should:

        Either target shapes at Snapshot/ and DefaultShot/ (not all dcat:Datasets),

        Or have separate “draft-tolerant” shapes vs “publishable” shapes.

Given that, your revised WorkingShot class should just mirror DefaultShot’s dcat:Dataset status, but keep the comment very explicit that it’s mutable and not part of historical/as-of semantics.

Here’s the JSON-LD version, updated from the last sketch:

{
  "@id": "WorkingShot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "FlowShot/" },
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  "rdfs:label": "Working Shot",
  "rdfs:comment": "Mutable staging shot for a Flow, mapped to the '_working/' directory. Intended to be a dataset in draft form: after each weave it starts from a valid dataset state and may temporarily be syntactically or structurally invalid while edits are in progress, but must be brought back to a valid dataset before the next weave. Not a committed historical version and normally excluded from as-of queries."
}

If you’re also using the object property:

{
  "@id": "workingShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "WorkingShot/" },
  "rdfs:label": "working shot",
  "rdfs:comment": "Links a Flow to its mutable working shot under '_working/'. Used for staging and tooling rather than long-term provenance or as-of semantics."
}

Key invariant to document in prose / code (not in OWL):

    “Only Snapshot and DefaultShot participate in version history and as-of selection; WorkingShot is always ignored for those purposes, even though it’s a dcat:Dataset.”

You said:
OK, how's this look:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "prov": "http://www.w3.org/ns/prov#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/"
  },
  "@graph": [
    {
      "@id": "/",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": { "@id": "https://djradon.github.io/ns/djradon/" },
      "owl:versionInfo": "0.1.0",
      "owl:versionIRI": { "@id": "https://semantic-flow.github.io/ontology/semantic-flow/_payload-flow/_default/sflo.ttl" },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow/"
    },
    {
      "@id": "Resource/",
      "@type": "rdfs:Class",
      "rdfs:label": "Resource",
      "rdfs:comment": "Abstract base class for all Semantic Flow resources. Provides fundamental identification and documentation affordances."
    },
    {
      "@id": "FileResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Component/" },
      "rdfs:label": "File Resource",
      "rdfs:comment": "A resource that locates and corresponds directly to a file."
    },
    {
      "@id": "NamingResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Resource/" },
      "rdfs:label": "Naming Resource",
      "rdfs:comment": "A resource that establishes a name, optionally denoting a non-file thing. Naming resources' IRIs end with a slash and may or may not have a referent."
    },
    {
      "@id": "hasResource/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Resource/" },
      "schema:rangeIncludes": { "@id": "Resource/" },
      "rdfs:label": "has resource",
      "rdfs:comment": "Abstract base property for associating resources."
    },
    {
      "@id": "AssetTree/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "Component/" },
        { "@id": "NamingResource/" }
      ],
      "rdfs:label": "AssetTree",
      "rdfs:comment": "Asset trees contain collections of files and directories that are not MeshResources. Can be contained in any NamingResource."
    },
    {
      "@id": "MeshNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NamingResource/" },
      "rdfs:label": "Mesh Node",
      "rdfs:comment": "Structural units that organize and contain other resources. Nodes form the structure of the mesh."
    },
    {
      "@id": "BareNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Namespace",
      "rdfs:comment": "Bare nodes - containers for organizing other mesh nodes. Node identifiers refer to the nodes themselves as organizational units."
    },
    {
      "@id": "ReferenceNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Reference",
      "rdfs:comment": "Nodes that refer to some external 'thing' or concept other than themselves."
    },
    {
      "@id": "PayloadNode/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "MeshNode/" },
      "rdfs:label": "Data",
      "rdfs:comment": "Nodes that contain and refer to 'payload data' via a payload flows."
    },
    {
      "@id": "hasComponent/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasResource/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "Component/" },
      "rdfs:label": "has component",
      "rdfs:comment": "Associates nodes with their supporting components."
    },
    {
      "@id": "isHandleFor/",
      "@type": "owl:ObjectProperty",
      "owl:inverseOf": { "@id": "hasHandle/" },
      "rdfs:domain": { "@id": "HandleAsComponent/" },
      "rdfs:range": { "@id": "MeshNode/" },
      "rdfs:label": "is handle for",
      "rdfs:comment": "Indicates that this handle component provides identification and routing for a specific node."
    },
    {
      "@id": "hasHandle/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasComponent/" },
      "owl:inverseOf": { "@id": "isHandleFor/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "HandleAsComponent/" },
      "rdfs:label": "has handle",
      "rdfs:comment": "Associates a mesh node with its persistent handle component."
    },
    {
      "@id": "slug/",
      "@type": "owl:AnnotationProperty",
      "schema:domainIncludes": [
        { "@id": "MeshNode/" },
        { "@id": "Flow/" }
      ],
      "schema:rangeIncludes": { "@id": "xsd:string" },
      "rdfs:label": "has slug",
      "rdfs:comment": "A concise, human-readable, kebab-case identifier for a node or a flow, suitable for use in filenames and URLs."
    },
    {
      "@id": "Component/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Resource/" },
      "rdfs:label": "Component",
      "rdfs:comment": "Terminal resources that provide specific mesh functionality and support nodes."
    },
    {
      "@id": "Handle/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NamingResource/" },
      "rdfs:label": "Handle",
      "rdfs:comment": "Conceptual handle that refers to a node 'as a node' (as opposed to the thing/concept referred to by the parent node). This is the abstract handle concept."
    },
    {
      "@id": "HandleAsComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "Component/" },
        { "@id": "Resource/" }
      ],
      "rdfs:label": "Handle As Component",
      "rdfs:comment": "A handle considered as a mesh component in its own right, addressable via IRI and subject to mesh operations."
    },
    {
      "@id": "DocumentationResource/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "FileResource/" },
      "rdfs:label": "Documentation Resource",
      "rdfs:comment": "Documentation files that provide human-readable information about resources."
    },
    {
      "@id": "ResourcePage/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Resource Page",
      "rdfs:comment": "Human-readable index.html pages that provide dereferenceability for NamingResources."
    },
    {
      "@id": "Changelog/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Changelog",
      "rdfs:comment": "Changelog files that document version history and changes for resources."
    },
    {
      "@id": "Readme/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "DocumentationResource/" },
      "rdfs:label": "Readme",
      "rdfs:comment": "Readme files that provide introductory documentation and usage information."
    },
    {
      "@id": "hasDocumentationResource/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "DocumentationResource/" },
      "rdfs:label": "has DocumentationResource",
      "rdfs:comment": "Associates a resource with a documentation resource."
    },
    {
      "@id": "hasResourcePage/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "ResourcePage/" },
      "rdfs:label": "has ResourcePage",
      "rdfs:comment": "Associates a NamingResource with its index.html resource page file."
    },
    {
      "@id": "hasReadme/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "Readme/" },
      "rdfs:label": "has Readme",
      "rdfs:comment": "Associates a NamingResource with its README documentation file."
    },
    {
      "@id": "hasChangelog/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "NamingResource/" },
      "schema:rangeIncludes": { "@id": "Changelog/" },
      "rdfs:label": "has Changelog",
      "rdfs:comment": "Associates a NamingResource with its changelog documentation file."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        { "@id": "dcat:DatasetSeries" },
        { "@id": "Component/" }
      ],
      "rdfs:label": "Flow",
      "rdfs:comment": "Abstract evolvable datasets for node metadata, reference data, payload data, and configuration. Flows are DatasetSeries that are realized through flow snapshots representing specific states or versions."
    },
    {
      "@id": "NodeMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_node-metadata-flow",
      "rdfs:label": "Metadata Flow",
      "rdfs:comment": "Flow containing metadata about the mesh node itself."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_reference-flow",
      "rdfs:label": "Reference Flow",
      "rdfs:comment": "Flow containing reference data about the thing or concept referred to by the mesh node."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "slug/": "_payload-flow",
      "rdfs:label": "Payload Flow",
      "rdfs:comment": "Flow containing the dataset payload associated with the mesh node."
    },
    {
      "@id": "NodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "Flow/" },
      "rdfs:label": "Node Config Flow",
      "rdfs:comment": "Abstract flow for mesh node configuration data."
    },
    {
      "@id": "InheritableNodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NodeConfigFlow/" },
      "slug/": "_config-inheritable-flow",
      "rdfs:label": "Inheritable Node Config Flow",
      "rdfs:comment": "Node configuration flow whose settings can be inherited by child nodes."
    },
    {
      "@id": "OperationalNodeConfigFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": { "@id": "NodeConfigFlow/" },
      "slug/": "_config-operational-flow",
      "rdfs:label": "Operational Node Config Flow",
      "rdfs:comment": "Node configuration flow whose settings apply only to the specific node."
    },
    {
      "@id": "hasFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasComponent/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "Flow/" },
      "rdfs:label": "has Flow",
      "rdfs:comment": "Links mesh nodes to their flows (metadata, reference, data, config)."
    },
    {
      "@id": "hasNodeMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "NodeMetadataFlow/" },
      "rdfs:label": "has NodeMetadataFlow",
      "rdfs:comment": "Links mesh nodes to their metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "ReferenceFlow/" },
      "rdfs:label": "has ReferenceFlow",
      "rdfs:comment": "Links mesh nodes to their reference flow."
    },
    {
      "@id": "hasPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "PayloadFlow/" },
      "rdfs:label": "has PayloadFlow",
      "rdfs:comment": "Links mesh nodes to their payload flow."
    },
    {
      "@id": "hasInheritableNodeConfigFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "InheritableNodeConfigFlow/" },
      "rdfs:label": "has InheritableNodeConfigFlow",
      "rdfs:comment": "Links mesh nodes to their inheritable node config flow."
    },
    {
      "@id": "hasOperationalNodeConfigFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlow/" },
      "rdfs:domain": { "@id": "MeshNode/" },
      "rdfs:range": { "@id": "OperationalNodeConfigFlow/" },
      "rdfs:label": "has OperationalNodeConfigFlow",
      "rdfs:comment": "Links mesh nodes to their operational node config flow."
    },
    {
      "@id": "FlowShot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": [
        { "@id": "NamingResource/" },
        { "@id": "dcat:Dataset" }
      ],
      "rdfs:label": "Flow Shot",
      "rdfs:comment": "One concrete dataset instance belonging to a Flow: a committed snapshot, the default shot under '_default/', or the mutable working shot under '_working/'. A FlowShot represents the Flow's data as a whole at a particular stage, not a subset of it."
    },
    {
      "@id": "Snapshot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": { "@id": "FlowShot/" },
      "rdfs:label": "Snapshot",
      "rdfs:comment": "Immutable, committed flow shot (dataset state), typically materialized as a timestamped weave directory. Versioned flows have one Snapshot per committed weave; unversioned flows may have none."
    },
    {
      "@id": "DefaultShot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": { "@id": "FlowShot/" },
      "rdfs:label": "Default Shot",
      "rdfs:comment": "The Flow's default dataset shot under the '_default/' directory. For versioned flows, its content mirrors the latest Snapshot. For unversioned flows, it is the only authoritative dataset state. For deversioned flows, it is the current"
    },
    {
      "@id": "WorkingShot/",
      "@type": "owl:Class",
      "rdfs:subClassOf": { "@id": "FlowShot/" },
      "rdfs:label": "Working Shot",
      "rdfs:comment": "Mutable staging shot for a Flow, mapped to the '_working/' directory. Used for edits before weave. Not a committed dataset and normally not exposed in version history or as-of queries."
    },
    {
      "@id": "hasFlowShot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasComponent/" },
      "rdfs:domain": { "@id": "Flow/" },
      "rdfs:range": { "@id": "FlowShot/" },
      "rdfs:label": "has FlowShot",
      "rdfs:comment": "Associates a Flow with one of its shots (default, snapshot, or working)."
    },
    {
      "@id": "defaultShot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
      "rdfs:domain": { "@id": "Flow/" },
      "rdfs:range": { "@id": "DefaultShot/" },
      "rdfs:label": "default shot",
      "rdfs:comment": "Links a Flow to its default shot under '_default/'. Each Flow must have exactly one DefaultShot."
    },
    {
      "@id": "hasSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
      "rdfs:domain": { "@id": "Flow/" },
      "rdfs:range": { "@id": "Snapshot/" },
      "rdfs:label": "has Snapshot",
      "rdfs:comment": "Associates a Flow with one of its committed snapshot shots."
    },
    { "@id": "Distribution/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
          { "@id": "dcat:Distribution" },
          { "@id": "FileResource/" },
          { "@id": "Component/" }
      ],
      "rdfs:label": "Distribution",
      "rdfs:comment": "A distribution (file) that contains the RDF data for a snapshot."
    },
    { "@id": "hasDistribution/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "dcat:distribution" },
      "schema:domainIncludes": { "@id": "FlowShot/" },
      "schema:rangeIncludes": { "@id": "Distribution/" },
      "rdfs:label": "has Distribution",
      "rdfs:comment": "Associates a FlowShot with its data distribution files."
    },
    {
      "@id": "hasSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "prov:specializationOf" },
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "Snapshot/" },
      "rdfs:label": "hasSnapshot",
      "rdfs:comment": "Associates a Flow with its snapshots. This property enables version management within flows and helps implement the prov: model"
    },
    {
      "@id": "hasCurrentShot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "hasSnapshot/" },
      "rdfs:domain": { "@id": "Flow/" },
      "schema:rangeIncludes": [ 
        { "@id": "Snapshot/" },
        { "@id": "DefaultShot/" }
      ],
      "rdfs:label": "hasCurrentSnapshot",
      "rdfs:comment": "Associates a Flow with its current snapshot."
    },
    {
      "@id": "previousSnapshot/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": { "@id": "dcat:previousVersion" },
      "rdfs:label": "previous snapshot",
      "rdfs:comment": "Links to the previous version snapshot of this snapshot.",
      "schema:domainIncludes": { "@id": "VersionSnapshot/" },
      "schema:rangeIncludes": { "@id": "VersionSnapshot/" }
    },
    {
      "@id": "VersioningState/",
      "@type": "owl:Class",
      "rdfs:label": "Versioning State",
      "rdfs:comment": "Enumeration of possible versioning states for a flow.",
      "owl:oneOf": {
        "@list": [
          { "@id": "VersioningState/versioned/" },
          { "@id": "VersioningState/unversioned/" },
          { "@id": "VersioningState/deversioned/" }
        ]
      }
    },
    {
      "@id": "VersioningState/versioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "Versioned",
      "rdfs:comment": "The flow maintains version snapshots with distinct version keys."
    },
    {
      "@id": "VersioningState/unversioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "unversioned",
      "rdfs:comment": "The flow does not maintain separate version snapshots; only working snapshots exist."
    },
    {
      "@id": "VersioningState/deversioned/",
      "@type": [
        "owl:NamedIndividual",
        "VersioningState/"
      ],
      "rdfs:label": "deversioned",
      "rdfs:comment": "The flow previously maintained version snapshots but has been deversioned; historical versions may still exist."
    },
    {
      "@id": "versioningState/",
      "@type": "owl:ObjectProperty",
      "rdfs:label": "versioning state",
      "rdfs:comment": "The versioning state of a flow.",
      "schema:domainIncludes": { "@id": "Flow/" },
      "schema:rangeIncludes": { "@id": "VersioningState/" }
    },
    {
      "@id": "weaveLabel/",
      "@type": "owl:DatatypeProperty",
      "schema:domainIncludes": [
        { "@id": "Flow/" },
        { "@id": "Snapshot/" }
      ],
      "schema:rangeIncludes": { "@id": "xsd:string" },
      "rdfs:label": "weave label",
      "rdfs:comment": "Human-readable label for a weave run (e.g., '2025-11-24_0142' or '2025-11-24_0142a'). All snapshots created by the same weave share the same weaveLabel. Not guaranteed to be globally unique across all meshes."
    },
    {
      "@id": "sequenceNumber/",
      "@type": "owl:DatatypeProperty",
      "schema:domainIncludes": { "@id": "Snapshot/" },
      "schema:rangeIncludes": { "@id": "xsd:integer" },
      "rdfs:label": "sequence number",
      "rdfs:comment": "Monotonic integer identifying this snapshot within its Flow. 1 for the first committed snapshot of a flow, 2 for the next, etc. Used for ordering and for detecting gaps or compaction at the flow level."
    },
    {
      "@id": "hasOperationalConfigFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "OperationalNodeConfigFlow/" },
      "rdfs:label": "opertaional config",
      "rdfs:comment": "Links mesh nodes to their operational config flow."
    },
    {
      "@id": "hasInheritableConfigFlow/",
      "@type": "owl:ObjectProperty",
      "schema:domainIncludes": { "@id": "Handle/" },
      "schema:rangeIncludes": { "@id": "InheritableNodeConfigFlow/" },
      "rdfs:label": "inheritable config",
      "rdfs:comment": "Links mesh nodes to their operational config flow."
    },
    {
      "@id": "disjoint/FileVsNamingResources",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "FileResource/" },
          { "@id": "NamingResource/" }
        ]
      }
    },
    {
      "@id": "disjoint/NodeStructuralRoles",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "BareNode/" },
          { "@id": "ReferenceNode/" },
          { "@id": "PayloadNode/" }
        ]
      }
    },
    {
      "@id": "disjoint/ComponentResources",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "HandleAsComponent/" },
          { "@id": "DocumentationResource/" },
          { "@id": "AssetTree/"},
          { "@id": "Flow/" },
          { "@id": "Snapshot/" },
          { "@id": "Distribution/" }
        ]
      }
    },
    {
      "@id": "disjoint/FlowTypes",
      "@type": "owl:AllDisjointClasses",
      "owl:members": {
        "@list": [
          { "@id": "NodeMetadataFlow/" },
          { "@id": "ReferenceFlow/" },
          { "@id": "PayloadFlow/" },
          { "@id": "InheritableNodeConfigFlow/" },
          { "@id": "OperationalNodeConfigFlow/" }
        ]
      }
    }
  ]
}
ChatGPT said:

Short answer: structurally you’re close, but there are some hard conflicts and some comments that no longer match the model. I’ll call out the real problems first, then the subtler ones, and then suggest concrete fixes.
1. Hard problems you must fix
1.1 hasSnapshot/ is defined twice (and inconsistently)

You have:

{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "Snapshot/" },
  ...
},
...
{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "prov:specializationOf" },
  "schema:domainIncludes": { "@id": "Flow/" },
  "schema:rangeIncludes": { "@id": "Snapshot/" },
  ...
}

Same @id, two different nodes. JSON-LD will merge them, but you’re now implicitly asserting:

    hasSnapshot ⊑ hasFlowShot and ⊑ prov:specializationOf,

    domain both via rdfs:domain and schema:domainIncludes,

    range both via rdfs:range and schema:rangeIncludes.

That’s not catastrophic, but it’s sloppy and confusing.

You should have one definition and, if you want both superproperties, put them in one place:

{
  "@id": "hasSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": [
    { "@id": "hasFlowShot/" },
    { "@id": "prov:specializationOf" }
  ],
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "Snapshot/" },
  "rdfs:label": "has Snapshot",
  "rdfs:comment": "Associates a Flow with one of its committed snapshot shots and specializes the Flow as a DatasetSeries into this particular Snapshot dataset."
}

And drop the other hasSnapshot/ block entirely.
1.2 hasCurrentShot/ contradicts your own semantics

Current block:

{
  "@id": "hasCurrentShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasSnapshot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "schema:rangeIncludes": [ 
    { "@id": "Snapshot/" },
    { "@id": "DefaultShot/" }
  ],
  "rdfs:label": "hasCurrentSnapshot",
  "rdfs:comment": "Associates a Flow with its current snapshot."
}

Problems:

    It’s a subproperty of hasSnapshot, but you explicitly allow DefaultShot/ in the range. For unversioned flows, the “current” is a DefaultShot, not a Snapshot.

    Label and comment say “Snapshot”, but semantically this is “current shot”.

You have two options:

    Either: restrict this to only snapshots (for versioned flows),

    Or: treat “current shot” independently of “snapshot”.

Given your own story (“unversioned: only DefaultShot; versioned: DefaultShot mirrors latest Snapshot”), the clean option is to decouple:

{
  "@id": "currentShot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlowShot/" },
  "rdfs:domain": { "@id": "Flow/" },
  "rdfs:range": { "@id": "FlowShot/" },
  "rdfs:label": "current shot",
  "rdfs:comment": "Associates a Flow with the shot currently considered live. For versioned flows this SHOULD be the DefaultShot mirroring the latest Snapshot; for unversioned flows it is the only DefaultShot; WorkingShot is never a currentShot."
}

And remove hasCurrentShot/ as a subproperty of hasSnapshot/.

If you want a stronger invariant later, you can enforce:

    currentShot value ∈ { DefaultShot } for all flows,

    plus “DefaultShot mirrors latest Snapshot” for versioned flows,

but that’s an implementation/SHACL concern, not OWL.
1.3 previousSnapshot/ refers to a non-existent VersionSnapshot/

You have:

{
  "@id": "previousSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "dcat:previousVersion" },
  "schema:domainIncludes": { "@id": "VersionSnapshot/" },
  "schema:rangeIncludes": { "@id": "VersionSnapshot/" }
}

But there is no VersionSnapshot/ class any more.

If you intend any Snapshot to participate in a predecessor chain, this should be:

{
  "@id": "previousSnapshot/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "dcat:previousVersion" },
  "rdfs:label": "previous snapshot",
  "rdfs:comment": "Links a Snapshot to its previous committed Snapshot within the same Flow.",
  "rdfs:domain": { "@id": "Snapshot/" },
  "rdfs:range": { "@id": "Snapshot/" }
}

If you want this chain at all; otherwise, just drop it.
1.4 Config-flow properties are duplicated/conflicting

You define:

{
  "@id": "hasOperationalNodeConfigFlow/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": { "@id": "hasFlow/" },
  "rdfs:domain": { "@id": "MeshNode/" },
  "rdfs:range": { "@id": "OperationalNodeConfigFlow/" },
  ...
},
...
{
  "@id": "hasOperationalConfigFlow/",
  "@type": "owl:ObjectProperty",
  "schema:domainIncludes": { "@id": "Handle/" },
  "schema:rangeIncludes": { "@id": "OperationalNodeConfigFlow/" },
  ...
}

and same pattern for hasInheritableNodeConfigFlow/ vs hasInheritableConfigFlow/.

These look like leftovers from two incompatible design choices:

    configs attached to nodes, vs

    configs attached to handles.

Right now you have both sets of properties, with extremely similar names. That’s going to be a reliability trap.

You need to choose:

    Either:

        keep the MeshNode-domain properties (hasOperationalNodeConfigFlow/, hasInheritableNodeConfigFlow/),

        and remove the handle-based ones, or

    explicitly define the handle-based ones as separate, with intentional comments about why configs also hang off the handle.

As written, it’s just ambiguity.
1.5 VersioningState/unversioned/ comment is now wrong

You say:

"rdfs:comment": "The flow does not maintain separate version snapshots; only working snapshots exist."

But per your new design:

    unversioned flows still have a DefaultShot dataset under _default/,

    they do not create Snapshots per weave,

    WorkingShot is draft/staging, not “only authoritative state.”

So this should say something like:

"rdfs:comment": "The flow does not maintain separate committed snapshots; only a single default shot exists and is updated on each weave (with a working shot used for staging)."

Otherwise, anyone reading the ontology will get the wrong mental model.
2. Structural / modeling issues
2.1 FlowShot/ subclasses

You currently have:

{
  "@id": "FlowShot/",
  "@type": "owl:Class",
  "rdfs:subClassOf": [
    { "@id": "NamingResource/" },
    { "@id": "dcat:Dataset" }
  ],
  ...
}

That’s fine, but note:

    FlowShot is already a Resource via NamingResource/ ⊑ Resource/.

    If you want to be explicit (for human readers), you could add Resource/ as a third superclass, but it’s redundant logically.

Your earlier “FlowShot ⊑ Resource” idea is effectively already true.
2.2 Distribution/ comment too narrow

You say:

"rdfs:comment": "A distribution (file) that contains the RDF data for a snapshot."

But:

    hasDistribution/ has schema:domainIncludes FlowShot/, which includes Snapshot, DefaultShot, and WorkingShot.

    So distributions will also be attached to DefaultShot and WorkingShot.

Either:

    restrict domain to Snapshot/, or

    update the comment to match reality:

"rdfs:comment": "A distribution (file) that contains the RDF data for a FlowShot (snapshot, default shot, or working shot)."

Given your current wiring, the latter is more consistent.
2.3 WorkingShot/ comment vs class

You’ve now made WorkingShot/ ⊑ FlowShot/, and FlowShot/ ⊑ dcat:Dataset. That matches your “draft dataset is still a dataset” stance, which is fine.

Just make sure the comments are not accidentally implying:

    that it is not a dataset,

    or that it participates in version history.

Your current comment:

    “Not a committed dataset and normally not exposed in version history or as-of queries.”

is actually fine: “not committed” ≠ “not a dataset”. Just keep it that way.
3. Minor but worth fixing
3.1 Truncated DefaultShot/ comment

You have:

"rdfs:comment": "The Flow's default dataset shot under the '_default/' directory. For versioned flows, its content mirrors the latest Snapshot. For unversioned flows, it is the only authoritative dataset state. For deversioned flows, it is the current"

That last sentence just trails off. Finish or delete it.
3.2 disjoint/ComponentResources might be too broad

"owl:members": {
  "@list": [
    { "@id": "HandleAsComponent/" },
    { "@id": "DocumentationResource/" },
    { "@id": "AssetTree/"},
    { "@id": "Flow/" },
    { "@id": "Snapshot/" },
    { "@id": "Distribution/" }
  ]
}

This asserts, for example, that:

    Flow and Snapshot can never share an individual (fine),

    Distribution is disjoint from DocumentationResource (fine),

    but you’ve not included DefaultShot or WorkingShot, so by default, instances of those could be any combination you haven’t disallowed.

If your intent is “these are all mutually exclusive leaf-ish resource types,” then either:

    include all shot types (Snapshot, DefaultShot, WorkingShot), or

    rename this to something less prescriptive.

Not a hard bug, but it’s unclear what invariant you’re actually trying to capture here.
4. Recommended next steps

    Deduplicate and reconcile properties:

        Merge the two hasSnapshot/ definitions into one (with both superproperties if you want them).

        Replace hasCurrentShot/ with a clean currentShot/ (or drop entirely if you don’t need it).

        Fix or drop previousSnapshot/.

        Decide on node vs handle config attachment and delete the “other” property set.

    Fix comments to match the new semantics:

        VersioningState/unversioned/ (no longer “only working snapshots exist”).

        Distribution/ comment (FlowShot vs “snapshot” only).

        Finish DefaultShot/ comment.

    Optionally:

        Add an owl:AllDisjointClasses for { Snapshot, DefaultShot, WorkingShot } if you want to guarantee a FlowShot instance is exactly one of those roles.

Once you clean those up, the Flow/FlowShot/Shot-type structure will actually line up with your narrative: flows as dataset series, shots as concrete dataset instances, and _default vs _working vs timestamped snapshots differentiated without cheating DCAT or your own intended semantics.
You said:
Just to get back to the weaveLabeling, I still want to include vN (ordinality) because it's a good check of whether anything is missing, how many versions we've had. Any downsides?
ChatGPT said:

Including vN (ordinality) is fine in principle, but only if you’re disciplined about where it lives and what it means.

You’re really talking about two different roles:

    weaveLabel = cross-flow, “what weave run is this from?”

    sequenceNumber (vN) = per-flow, “which committed snapshot is this within this flow?”

Those are orthogonal. Mixing them too tightly is where trouble starts.
1. Real upsides of keeping vN

You’re not wrong about the benefits:

    Gap detection per flow

        If you keep sequenceNumber as a strict 1,2,3,… per Flow, you can detect:

            missing snapshots (… 4, 5, 7 ⇒ 6 is gone or never committed),

            compaction anomalies (if you enforce contiguity and forbid renumbering).

    Cheap human-readable count

        “This flow has had 27 committed versions” is cognitively nicer than “it has snapshots with labels 2025-11-24_0142a, 2025-11-24_1020, …”.

    Easy per-flow ordering

        Even if weaveLabel is time-like, sequenceNumber lets you order snapshots with no need for parsing or cross-flow meaning.

Those are all legitimate.
2. The actual downsides / traps

The problems aren’t “vN exists”; the problems are where you encode it and how much you rely on it for identity.
2.1 Duplication of truth

If you:

    put v3 into the folder name: 2025-11-24_0142_v3/, and

    also store sequenceNumber = 3 in RDF,

you’ve created two sources of truth. Sooner or later, something will drift:

    A buggy tool increments the folder suffix but forgets to bump sequenceNumber.

    A manual rename happens in Git.

    A compaction / migration script rewrites folder names but not RDF (or vice versa).

Once that happens, you’ve lost your “gap detection” reliability unless you always recompute from RDF and ignore paths. At that point, the _vN in the folder name is just cosmetics that can lie.

Mitigation: treat sequenceNumber in RDF as the only authoritative ordinality and consider any _vN in the folder name as purely decorative.
2.2 Compaction and rewriting history

You’ve already talked about compaction. There are two incompatible choices:

    Never renumber: keep v1, v3, v7 even if v2, v4, v5, v6 are deleted.

        Pro: you preserve ordinality semantics over time (no lies).

        Con: you lose “no gaps” as a simple integrity check. Gaps now mean “deleted, maybe intentionally,” not “corrupt or missing.”

    Renumber after compaction: pack surviving snapshots as 1..N again.

        Pro: you regain the “compact, no-gap” guarantee.

        Brutal Con: every external reference to “snapshot v5” is now wrong. You’ve literally rewritten history.

You cannot have both “no gaps ever” and “stable identity” for vN. Pick one:

    If vN is a diagnostic (good for debugging/tests), gaps are acceptable.

    If vN is an identity handle, don’t ever change it and accept gaps forever.

Anything else is self-sabotage.
2.3 Concurrency and branching

Even with “only one process weaves a mesh at a time” as a local rule, Git breaks that assumption:

    Two branches both weave the same flow:

        branch A gets snapshots with sequenceNumber = 5,

        branch B also gets a snapshot with sequenceNumber = 5.

    After merge, you now have two different snapshots claiming v5.

If folder names also carry _v5, you now have a naming collision or a forced rename, and one of them is lying about its sequence.

You either:

    treat sequenceNumber as per-branch (then ordinality is branch-local, not global), or

    treat one branch as canonical and rewrite the other branch’s sequence numbers on merge (which again breaks the “identity” semantics).

You don’t get concurrency for free just because the filesystem is single-writer at a time; Git introduces parallel timelines.
3. How to keep vN without hurting yourself

Given all that, the least-damaging pattern is:

    Keep ordinality as RDF-only metadata

        sequenceNumber per Flow:

            xsd:integer,

            1-based, per-Flow monotone as intended,

            but you accept that gaps or duplicates may appear after merges / compaction, and treat them as “signals,” not “bugs that must be repaired by rewriting history.”

    Keep weaveLabel purely time-ish / human-ish

        weaveLabel is global to a weave run; multiple flows can share it.

        It does not encode vN.

        You can still display both together: “weaveLabel=2025-11-24_0142, seq=3”.

    Folders named only by weave-ish label

        e.g. 2025-11-24_0142/, 2025-11-24_0142a/.

        Do not bake _v3 into the folder name unless you are willing to treat that suffix as purely cosmetic and never trust it as the source of truth.

    Use sequenceNumber where it’s actually helpful

        Simple SPARQL:

            List snapshots for a flow ordered by sequenceNumber.

            Detect “this flow has N snapshots”.

        Debug:

            “This Snapshot has sequenceNumber = 7 but there is no 6; either compaction or lost commit.”

That gives you all the benefits you want without entangling ordinality with file-system naming and cross-flow weave identities.
4. Direct answer to your question

    “I still want to include vN (ordinality) because it's a good check of whether anything is missing, how many versions we've had. Any downsides?”

    Keeping a vN as a per-flow sequenceNumber in RDF:
    No fundamental downside, as long as you:

        accept gaps / duplicates as potential signals rather than rewriting them away, and

        don’t pretend it’s globally unique or immutable under branching/merging.

    Embedding vN into weaveLabel or folder names:
    The downsides are:

        duplicated source of truth (name vs RDF),

        painful migrations/compaction,

        nasty behaviour on Git merges.

So: yes, keep ordinality, but keep it where it belongs (sequenceNumber metadata), and keep weaveLabel and directory names independent of it.
You said:
I'm imagining no branches, or if there are branches, never merging them. I'm going to keep ordinality in the folder names (which means it's also in the IRIs)... it's just cosmetic-ish.

Moving on, now we have FlowShots, which can be Snapshots, DefaultShots, and WorkingShots. It's really no different than before we had Snapshots, which can be VersionSnapshots, DefaultSnapshots, and WorkingSnapshots. Is the naming really better?
ChatGPT said:

On the folder vN thing: as long as you’re consciously treating it as cosmetic and you’re okay with “no branches or no merges,” that’s consistent with what you’ve said so far. Future-you may regret the “never merge branches” part, but that’s a separate decision.

On the actual question:

    Now we have FlowShots, which can be Snapshots, DefaultShots, and WorkingShots. It's really no different than before we had Snapshots, which can be VersionSnapshots, DefaultSnapshots, and WorkingSnapshots. Is the naming really better?

Structurally: no, it’s not different. It’s the same pattern with a different root word.

The only thing that’s changed is the overloading of “Snapshot” and the introduction of “Shot” as the root term. So the real question is: is that reduction in overloading worth paying for a slightly weirder root word?

Let’s be precise.
1. What you had vs what you have
Before

    Snapshot – abstract “flow state”

        VersionSnapshot – committed, immutable

        DefaultSnapshot – _default

        WorkingSnapshot – _working

Problems:

    “Snapshot” is doing double duty:

        the general concept (“any state”),

        and the colloquial sense (“a committed version”).

    When you say “snapshot” in prose or code, you have to constantly clarify “I mean VersionSnapshots, not WorkingSnapshots.”

Now

    FlowShot – abstract “flow state”

        Snapshot – committed, immutable

        DefaultShot – _default

        WorkingShot – _working

You’ve pushed the overloading down a level:

    “Snapshot” now clearly means “committed, immutable version.”

    The generic thing is “FlowShot.”

So yes, the main benefit is:

    fewer overloaded uses of “snapshot,”

    clearer mental model: Snapshot = only committed state.

Everything else is isomorphic.
2. Critique of the new naming
Clear wins

    “Snapshot” is now unambiguous.

    When you say “snapshot” in docs or API, everyone can safely assume:

        it’s a committed, immutable dataset state,

        not _default, not _working.

    That’s actually a big practical win for docs, conversation, and API naming.

    The hierarchy reads more cleanly.

        FlowShot –> Snapshot / DefaultShot / WorkingShot

        You’re not mixing “VersionSnapshot” vs “DefaultSnapshot,” which always sounded like “isn’t DefaultSnapshot also a kind of ‘version’?”

Real downsides

    “FlowShot” is not self-explanatory.

    Outside your head, “FlowShot” could mean:

        a run of some ETL job (“shot” of the flow),

        a single record in a stream,

        or something camera-like (“shot of the flow”).

    “Snapshot” is already a well-understood term in data/versioning; “FlowShot” is not. You’re paying a clarity tax for new readers.

    The triad is now slightly asymmetrical.

        FlowShot

            Snapshot

            DefaultShot

            WorkingShot

    Why is only one of those called “Snapshot”, and the others “Shot”? It’s logically fine, but from a naming aesthetics standpoint, it’s a bit off-balance.

    Searchability / interop.

    If someone looks for “snapshot” in the ontology, they’ll find the committed subclass but not the root. That’s okay, but they may miss the fact that _default/ and _working/ are also “states” in the same family unless the comments spell it out.

3. Is it really better?

Net:

    Conceptually: you’ve removed the “Snapshot is both generic and specific” confusion. That is genuinely better.

    Semantically: the structure is the same; you didn’t gain extra modeling power.

    Naming-wise: you traded an overloaded but familiar term (“Snapshot”) for a cleaner hierarchy that uses a non-standard root term (“FlowShot”).

So my answer:

    It is marginally better in terms of conceptual clarity about “what is a committed version.”

    It is not a slam-dunk improvement; you’re paying with a slightly odd new term.

If “FlowShot” doesn’t bother you aesthetically, it’s a reasonable place to land and you can keep it. If it does keep nagging at you, the cleaner alternative is:

    Go back to Snapshot as the root type, but be explicit in the subclass name:

        Snapshot/ – generic flow state (default, working, committed)

            CommittedSnapshot/ (or VersionedSnapshot/) – committed, immutable

            DefaultSnapshot/

            WorkingSnapshot/

The real fix there is naming the committed subclass as “CommittedSnapshot” instead of “VersionSnapshot,” so “snapshot” in natural language doesn’t have to carry both meanings.

Right now, you’ve solved the overloading problem; you just did it by inventing “FlowShot.” Whether that tradeoff is acceptable is largely an aesthetic/communication choice, not a correctness one.
You said:
Maybe you can merge, but it's always got to be a "straight merge" -- no overlapping branches get merged back/intermingled. It wouldn't make sense anyhow. 
ChatGPT said:

“Straight merge only” is coherent with everything you’ve said so far, but you need to be precise about what you’re forbidding and what you’re implicitly relying on.

Let me unpack the implications and edge cases.
1. What “straight merge” has to mean here

For your vN + folder-label scheme to stay sane, the rule really has to be:

    Only fast-forward merges onto the canonical branch.

    No merges where both sides added commits after the fork point (“overlapping branches”).

In Git terms:

    Allowed: local branch feature that simply moves ahead of main, then git merge --ff-only feature (or just git push if main tracks it).

    Not allowed: main advances, feature also advances, then git merge feature with a merge commit.

If you enforce --ff-only on the canonical branch (GitHub: “Require linear history”), you mechanically prevent the “intermingled histories” you’re trying to avoid.

Given that, your invariants become:

    There is exactly one linear history for each mesh.

    vN per Flow is strictly monotone along that line, never duplicated on parallel branches that later rejoin.

    Folder names with _vN are never “renumbered” as part of merges.

That’s consistent.
2. What this buys you for vN + weaveLabel

With those rules:

    A snapshot with sequenceNumber = 7 and folder 2025-11-24_0142_v7/ is globally unique for that Flow in the canonical history.

    You never have to reconcile “two different v7s” after a merge, because such merges are impossible.

    weaveLabel can remain purely weave-run–scoped; the combination (Flow IRI, vN) is enough to talk about a specific committed snapshot.

So under your no overlapping merges policy, the earlier concurrency/branching traps around vN essentially go away.
3. The real risks are social/operational, not logical

Where this can still crack:

    Multiple writers + Git friction

        Two people both clone main, both weave and commit.

        First pushes: OK.

        Second tries to push: rejected (non-fast-forward).

        They must git pull --ff-only, resolve conflicts, and re-weave if necessary.

    If your tooling doesn’t make that “pull + re-weave” path explicit, you’ll get confusion or someone trying to “fix” it by hacking directories or disabling linear-history protection.

    Forks treated as “not really branches”

        If someone forks the repo and treats their fork as a real mesh (publishing their Pages), you now have two independent lineages with overlapping vNs and weaveLabels.

        That’s fine as long as you never try to merge them back, but you should treat them as distinct meshes, not as branches of the same mesh.

    Manual history surgery

        git rebase, git filter-branch, or history rewriting will break your “vN reflects ordinal history” assumption unless you explicitly treat those operations as creating a new mesh lineage.

These are all policy/documentation problems, not ontology problems, but they’re the places where your “only straight merges” story will actually be tested.
4. What you should explicitly codify

If you want this to be robust and not just “Dave knows the rules,” I’d suggest:

    Git policy for meshes

        Protected canonical branch (e.g. main) with:

            “Require linear history” (fast-forward only).

            “Prevent force pushes” (no history rewriting on canonical).

        Any experimental work happens:

            Either in branches that are never merged back, or

            In separate repos treated as distinct meshes.

    Mesh invariant (human-readable)

    Something like:

        “Each mesh has exactly one canonical commit history. Version ordinality (vN) for each Flow is defined along that history. Branches that diverge and later merge are not supported; any divergent histories must either remain separate meshes or be reconciled via a new weave on the canonical branch.”

    Tooling guardrails

        sflo-cli refuses to weave if:

            the working tree is not on the canonical branch, or

            the local branch is behind origin.

        Optionally: a “mesh status” command that:

            checks for local commits not yet pushed, or

            warns if you’re on a non-canonical branch.

5. On “it wouldn’t make sense anyhow”

You said:

    “Maybe you can merge, but it's always got to be a straight merge — no overlapping branches get merged back/intermingled. It wouldn't make sense anyhow.”

It would be possible to define a merge policy where:

    branch A weaves Flow X,

    branch B weaves Flow Y (disjoint),

    a later merge just unions the new snapshots for X and Y.

That’s logically coherent.

You’re choosing not to support it because:

    it complicates vN and directory naming,

    it complicates “as-of” reasoning (interleaved weaves from different branches),

    it complicates the mental model of “this mesh has one timeline.”

That’s a defensible design choice. Just be clear: it’s a simplicity tradeoff, not an impossibility.

Net: given your constraints and the “vN in folder names” decision, banning overlapping merges and insisting on straight/fast-forward merges is consistent and actually necessary. Just don’t leave it as an implicit norm—bake it into Git settings, CLI guardrails, and a short, explicit “Mesh Git Policy” section in your docs.
You said:
generate a concise markdown wikipage describing git constraints on semantic meshes.
ChatGPT said:

# Git Constraints for Semantic Meshes

Semantic Meshes rely on **strictly linear Git history** to guarantee that
`sequenceNumber (vN)`, snapshot IRIs, and weave output remain stable and
interpretable.  
The following rules define the required Git workflow for any mesh repository.

---

## 1. Canonical Branch Policy

Each mesh has **one canonical branch**, typically `main`.  
All weave operations must run on this branch.

**Requirements:**

- Canonical branch must be configured as:
  - **Fast-forward merges only** (GitHub: “Require linear history”)
  - **Force pushes disabled**
- Local weave runs are **rejected** if the canonical branch is not current:
  - Must `git pull --ff-only` successfully before weaving
  - Any conflict must be resolved *before* weave runs

---

## 2. No Overlapping Branch Merges

Branches that diverge and both accumulate commits **cannot** be merged back.

Allowed:
- Feature branch that *only adds commits ahead* of canonical → fast-forward merge

Not allowed:
- Divergent histories where both branches add commits after the fork point
- Any merge that produces a merge commit
- Any operation that rewrites history and reassigns vN ordinality

If divergent work exists, it must either:
- Stay permanently separate, forming its own mesh lineage, or
- Be reconciled manually and re-woven on canonical as a fresh commit

---

## 3. Weave Preconditions

`sflo-cli weave` must enforce:

- Working directory is on canonical branch  
- Working tree is clean (no uncommitted changes)  
- Local tip matches remote (`git pull --ff-only` cleanly)  

If any check fails, the weave is aborted.

---

## 4. Snapshot Ordinality Invariant

For every Flow:

- `sequenceNumber (vN)` is **monotonic** and defined solely along canonical history
- Snapshot directories containing `_vN` must never be reused or renumbered
- vN corresponds exactly to the count of committed snapshots for that Flow

This invariant only holds under strictly linear history.

---

## 5. Forks and Experimental Branches

Forks or non-canonical branches may exist, but:

- They represent **independent mesh lineages**  
- Their vN and weaveLabels are meaningful **only inside that lineage**  
- They must **not** be merged back into the canonical branch

---

## 6. Summary

A Semantic Mesh requires:

- One linear Git history  
- Fast-forward only merges  
- No overlapping-branch merges  
- No history rewriting  
- Weaves run only on an up-to-date canonical branch  
- vN ordinality preserved across time

These constraints ensure that IRIs, snapshots, and weaveLabel semantics remain stable, comparable, and unambiguous across the life of the mesh.

You said:
"it wouldn't make sense" because a flow captures data change over time. you can't have the same data evolve in two directions simultaneously. 

I'm not sure about "Weave Preconditions" - Why "Working tree is clean (no uncommitted changes)"?
ChatGPT said:

A clean working tree is not obviously required, and your instinct is correct to challenge it.

Below is the critical analysis and the corrected conclusion.
Why “Working tree must be clean” is not a justified requirement
1. Weave is the operation that reads the working tree to produce _working and ultimately the next snapshot.

That means:

    Editing files → modifies the working tree

    Weave → copies the working state into _working and then creates snapshots

If we required a clean working tree before weave, you would never be allowed to weave anything, because the whole purpose is to weave those uncommitted edits.

So requiring “clean working tree” contradicts the function of weave itself.
2. The only thing that must be clean is: the canonical commit pointer.

The real requirement is:

    Before weaving, you must be on the latest commit of the canonical branch (fast-forward merged)

    But your working directory may contain edits

    Those edits become the new snapshot

Thus the relevant precondition is:

“Your HEAD commit must match the canonical branch tip (after a fast-forward pull), but your working tree may contain uncommitted edits.”
3. Allowing uncommitted changes is required for versioned flows.

You want:

    _working to reflect the current uncommitted dataset changes

    weave to freeze those edits into a new snapshot

    _default to update after that

If you forced a commit before weave, you would:

    introduce meaningless commits whose only purpose is to satisfy a precondition

    cause vN ordinality to be desynchronized from semantic change

    mix file-system operations (weave) with Git operations (committing), which muddies the semantics

4. Weave itself creates commits, so forcing pre-commits would cause commit noise.

Your design intends:

    weave → generate the files and then create a commit recording that state

If you forced a clean working tree before weave:

Your edits → forced commit
Then weave → commit again

You’d double the Git noise for no benefit.
5. What actually must be prevented is:

    weaving on an outdated branch

    overlapping histories that break vN and snapshot semantics

Not:

    editing files before weave

Correct Rule

The correct precondition is:

    Weave may run with uncommitted edits, as long as git pull --ff-only succeeds first.

More explicit:
Required

    On canonical branch

    The canonical branch is up-to-date (fast-forwardable)

    No merge commits or rewritten history

    No untracked or conflicting files that would be ignored by weave

Allowed

    Modified files

    Staged files

    Unstaged edits

    Untracked files

Forbidden

    A local branch tip that can't fast-forward to canonical

A more accurate phrasing for the wiki page

Replace:

    “Working tree is clean (no uncommitted changes).”

With:

“Weave reads the working tree directly. The working tree may contain uncommitted changes; those changes become the new _working and snapshot content.”

And the true constraint:

“Weave requires the HEAD commit to match the canonical branch tip (after git pull --ff-only).”

If you want, I can regenerate the entire Git constraints wiki page with this correction integrated.
You said:
I don't think weave creates commits. And I think you can commit before weave, if you want to save your WIP or let someone else pick it up.
ChatGPT said:

Right, thanks for correcting that—two separate issues here:

    Weave does not create commits.

    Committing before weave is allowed (and sometimes desirable).

Given that, the “Git constraints” story needs to decouple much more cleanly from the weave operation itself.

Let me reframe the model and then answer the “clean working tree” point directly.
1. Correct mental model: weave vs Git

    Weave:

        Purely a filesystem-level operation.

        Reads the current working tree (tracked files in the mesh) and:

            updates _working/ FlowShots,

            possibly creates new Snapshot directories (…_vN/),

            updates _default/ FlowShots.

        Increments sequenceNumber per Flow according to your rules.

        Does not touch Git at all.

    Git:

        Records whatever the filesystem currently looks like when you choose to commit.

        You can:

            commit before weave (checkpoint WIP),

            commit after weave (checkpoint new snapshots),

            commit both before and after, or not at all for a while.

So there is no inherent requirement that weave and commits line up 1:1. We only care that Git’s history remains linear so that filesystem-based ordinality + folder naming doesn’t become ambiguous after merges.
2. Why “working tree must be clean” is wrong here

Given the model above, this constraint:

    “Working tree is clean (no uncommitted changes)”

is unjustified and actively harmful, because:

    The whole point of weave is to materialize current edits (even uncommitted ones) into _working and new snapshots.

    If you required a clean tree, you’d force people to commit before every weave, which:

        generates extra WIP commits they may not want,

        couples Git history to weave frequency in an artificial way.

The actual requirement is:

    The working tree must be in a state that the weaver can understand:

        no half-written JSON-LD that your parser can’t handle (or you fail the weave),

        no missing mandatory files for that mesh config.

    But that has nothing to do with whether those changes are committed.

So: you can absolutely weave with uncommitted changes. Those changes are exactly what’s being woven.
3. What Git constraints still matter

The constraints that still make sense, even with offline + “weave doesn’t commit,” are:

    Linear canonical history

        Canonical branch (e.g. main) uses fast-forward only.

        No merges of overlapping branches back into canonical.

        No force-push / history rewrite on canonical.

    “Straight merge” interpretation

        You can have branches, but:

            either they never merge back, or

            when they do, it’s a simple fast-forward (no divergent history).

        That keeps vN and folder-based snapshot labels meaningful along a single timeline.

    Offline edit + weave + later push

        You can:

            clone,

            edit, weave, commit (or not),

            later git push.

        If someone else has advanced canonical in the meantime, your push:

            triggers a non-fast-forward error,

            and you must git pull --ff-only (or similar) and decide how to resolve.

        Your policy is effectively: if history diverged, you don’t merge back; you rebase and re-weave, or treat that work as a separate lineage.

None of that requires a clean working tree at weave time.
4. Where “commit before weave” fits

You said:

    “I think you can commit before weave, if you want to save your WIP or let someone else pick it up.”

That’s totally compatible. It just defines patterns like:

    Pattern A – “Weave-first”:

        edit → weave → run tests → commit

    Pattern B – “Checkpoint-first”:

        edit → commit WIP → weave → commit “weave output” (optional)

    Pattern C – “Heavy WIP”:

        edit, weave many times locally → occasional commits when happy

Your constraints don’t need to bless one of these as “the” pattern. The only real invariant is:

    sequenceNumber and folder _vN ordinality are determined by weave runs, not by commits.

    Git history must remain linear enough that those _vN’s never need to be reconciled across branches.

If you’d like, I can update the Git constraints wiki snippet to:

    remove the “clean working tree” rule,

    explicitly state that weave consumes the current working tree (committed or not),

    emphasize linear history + no overlapping merges as the only hard Git-side constraints.

You said:
yes please, new entire page.
ChatGPT said:

Below is a clean, concise, technically accurate wiki-page tailored to your mesh/weave/Git model.
Critical tone preserved: no hand-waving, no unjustified constraints.
Git Constraints for Semantic Meshes

Status: normative
Audience: implementers, mesh maintainers
Scope: applies to all Semantic Flow meshes and flows

Semantic Flow meshes store state directly in the filesystem. Weave operations create immutable snapshots, update default shots, and rewrite _working/ shots. Git is used only to version the resulting files; the weave engine itself never writes commits.

Because of this tight coupling between filesystem semantics and version-ordering, meshes require a restricted set of Git behaviors to guarantee that FlowShots remain interpretable and ordinal.

This page defines those constraints.
1. Core Principles
1.1 Weave does not interact with Git

Weave reads and writes the working tree only.
It does not:

    create commits,

    stage files,

    modify Git metadata.

Git history is independent of weave activity.
1.2 The working tree may contain uncommitted changes

Uncommitted edits are exactly what weave consumes and materializes into:

    _working/ shots,

    _default/ shots,

    new Snapshot directories (e.g., 2025-11-24_0142_v3/).

There is no requirement for a clean working tree before weave.
Edits can be committed before, after, or not at all.
1.3 Flow ordinality is filesystem-driven

Each flow maintains:

    sequenceNumber (integer),

    timestamped + suffixed weaveLabel (2025-11-24_0142, …0142a, etc.),

    _vN folder ordinality.

These are determined purely by weave, not by Git commits.
2. Why Git Constraints Exist at All

Semantic meshes assume that a Flow evolves along a single linear timeline.
If Git introduces merges that splice incompatible histories together, you produce:

    impossible composite timelines (e.g., _v5 and _v8 both claiming to follow _v4),

    gaps or duplications in sequenceNumber,

    weaveLabel collisions that cannot be interpreted causally.

Thus, the constraints are about preventing Git from creating fictional histories that the mesh cannot represent.
3. Canonical Branch Rules
3.1 Canonical branch must remain linear

The canonical branches (main, or any branch designated as authoritative for a mesh) must obey:

    Fast-forward only
    No merge commits that join divergent histories.

    No merges of overlapping branches
    Branches may exist, but:

        either they never merge back, or

        they merge only via fast-forward.

    No force-push on canonical
    Canonical history must never be rewritten.

This ensures that _vN ordinality and sequenceNumber remain meaningful.
3.2 Local branches may diverge, but reconciliation is limited

You may create feature branches or WIP branches freely.
However:

    If canonical advanced while your branch diverged, you must rebase, not merge.

    If conflicts occur during rebase, you resolve them in the filesystem and then re-weave, because weaveLabel and sequenceNumber must follow the new canonical lineage.

No “real” merges of independent weave timelines are permitted.
4. Weave and Git Interactions
4.1 You may commit at any time

Common safe patterns:

A. weave → test → commit
B. commit WIP → weave → commit weave outputs
C. edit → weave many times → commit rarely

All are valid. The weave engine treats the working tree as the source of truth, regardless of commit status.
4.2 You may push at any time, subject to FF-only rules

If pushing fails because canonical moved:

    git pull --ff-only must succeed,

    if not, rebase and re-weave.

This guarantees that Snapshot ordinality remains a single chain.
4.3 Offline work is fully supported

You can:

    clone a mesh

    edit, weave, commit locally

    come online and push

If canonical advanced, rebase + re-weave.
No merges.
5. Mesh Integrity Expectations
5.1 Snapshots must remain immutable

Once created, Snapshot directories (…_vN/) must never be edited.
Git reinforces this, because any mutation would create diffs.
5.2 _default/ must always mirror last successful weave

If a weave completes:

    _default/ must reflect the latest stable dataset state,

    _working/ must reset to match _default/.

If Git rebase rewrites files under _default/ or _working/ into divergence, the next weave will reconcile them by re-deriving canonical state.
5.3 Branch merges cannot create two “next snapshots”

Because only one timeline is allowed:

    two parallel _working/ edits from different branches cannot be auto-merged into a single snapshot,

    the rebase enforces which one comes first,

    the other must weave again on top.

This preserves causality for the dataset evolution.
6. What is Explicitly Allowed

    Uncommitted edits during weave.

    Arbitrary number of local branches.

    Arbitrary commits before, after, between weaves.

    Directory-level snapshot naming (timestamp+suffix_vN).

    Offline editing and delayed pushes.

    Rebasing canonical with re-weave afterward.

7. What is Explicitly Forbidden

    Merge commits that unify diverged histories.

    Force-push rewriting canonical.

    Modifying snapshot directories after creation.

    Any Git action that makes the flow’s _vN ordinality ambiguous.

    Merging two different weave timelines back into one.

8. Summary

Semantic meshes impose minimal but strict Git constraints:

    Linear history,

    Fast-forward only on canonical,

    No merges of overlapping branches,

    Rebase + re-weave when forks occur,

    Weave consumes the working tree regardless of commit state.

These rules ensure that filesystem-based weave semantics remain coherent and that every Flow evolves along a single, consistent timeline.

If you want, I can also generate:

    A companion page describing Weave Semantics,

    A page documenting FlowShot lifecycle,

    A page defining Recovery After a Bad Merge,

    Or machine-readable SHACL for the FlowShot invariants.

You said:
"Uncommitted edits are exactly what weave consumes and materializes into:" isn't right.
ChatGPT said:

Right, good catch. I overgeneralized.

Weave does not “consume uncommitted edits” in the Git sense; it consumes whatever the engine considers the current flow state (especially each Flow’s _working shot), which might or might not correspond to arbitrary edited files elsewhere in the repo.

Let me correct the mental model and then regenerate the page.
What weave actually consumes

Given everything you’ve said:

    Each Flow has a _working/ directory that holds its current editable dataset (FlowShot).

    You (or tools) edit those _working distributions.

    A weave run:

        reads the _working shots,

        validates / processes them,

        produces a new Snapshot (…_vN/ etc.) for versioned flows,

        updates _default/ to match the latest Snapshot (or to be the authoritative dataset for unversioned flows),

        resets _working/ to match _default/ after a successful weave.

So more precise wording:

    Weave consumes the current contents of _working FlowShots (plus whatever configuration flows drive its behavior).

    Git commit status is orthogonal: those files might be committed or not; weave doesn’t care.

It does not mean “any uncommitted change anywhere in the repo is consumed.”
Updated wiki page (corrected)

# Git Constraints for Semantic Meshes  
**Status:** normative  
**Audience:** implementers, mesh maintainers  
**Scope:** applies to all Semantic Flow meshes and flows

Semantic Flow meshes store state directly in the filesystem. Weave operations
operate on FlowShots (`_working/`, snapshots, `_default/`) and write new
datasets into those directories. Git is used only to version the resulting
files; the weave engine itself never writes commits.

Because FlowShot ordinality and naming are filesystem-based, meshes require
a restricted set of Git behaviors to keep histories linear and interpretable.

This page defines those constraints.

---

## 1. Core Principles

### 1.1 Weave does not interact with Git

Weave is a pure filesystem operation. It:

- reads current FlowShots (especially each Flow’s `_working/` directory),
- validates and processes them,
- writes new snapshot directories and `_default/` content,
- updates `_working/` to match `_default/` after a successful weave.

Weave **does not**:

- create commits,
- stage files,
- modify any Git metadata.

Git history is independent of weave activity.

---

### 1.2 Weave operates on current FlowShots, not commit state

Weave consumes the **current contents of the mesh’s FlowShots in the working
tree** (primarily `_working/` for each Flow), regardless of whether those files
are committed to Git.

Therefore:

- The working tree **does not need to be clean**.
- You may have modified, staged, or untracked files.
- What matters is whether the Flow’s `_working/` datasets are in a state the
  weaver can understand (valid enough to process), not whether they’re committed.

Git commits are optional checkpoints, not prerequisites.

---

### 1.3 Flow ordinality is filesystem-driven

Each versioned Flow maintains:

- a `sequenceNumber` (monotonic integer per Flow),
- a weaveLabel (e.g. `2025-11-24_0142`, `2025-11-24_0142a`),
- snapshot directories with suffixes like `_vN`.

These are:

- assigned by weave, based on Flow state,
- independent of the Git commit graph.

---

## 2. Why Git Constraints Exist

Semantic meshes assume that each Flow evolves along **one linear timeline**.

If Git introduces merges joining divergent histories, you can end up with:

- multiple incompatible “next” snapshots for the same Flow,
- ambiguous or duplicated `_vN` ordinality,
- weaveLabels that no longer line up with a single causal sequence.

So the Git constraints are solely about preventing Git from fabricating
histories that the FlowShot model cannot represent.

---

## 3. Canonical Branch Rules

### 3.1 Canonical branch must remain linear

Each mesh designates one **canonical branch** (typically `main`). For this branch:

- **Fast-forward only**  
  - No merge commits that join divergent histories.
- **No overlapping-branch merges**  
  - Branches may exist, but if they diverge from canonical and canonical advances,
    reconciliation must use rebase, not merge.
- **No force-push**  
  - Canonical history must not be rewritten.

This preserves a single linear sequence for FlowShot ordinality.

---

### 3.2 Local branches may diverge, but reconciliation is restricted

You can create arbitrary feature/WIP branches.

If canonical has moved ahead while your branch diverged:

- You must rebase onto canonical, then re-weave as needed.
- You must **not** merge back divergent histories into canonical.

If you want to keep a divergent lineage, you treat that branch (or fork) as
a separate mesh lineage, not something that merges back into the original mesh.

---

## 4. Weave and Git Interactions

### 4.1 You may commit at any time

All of the following patterns are valid:

- **A. Weave-first**  
  `edit → weave → test → commit`
- **B. Checkpoint-first**  
  `edit → commit WIP → weave → commit weave outputs`
- **C. Heavy local WIP**  
  `edit → weave multiple times → commit occasionally`

Weave always reads the Flow’s `_working/` state in the working tree, regardless of
commit status.

---

### 4.2 Pushing requires linear history

You may push whenever you like, subject to remote branch protection:

- If pushing to canonical fails (non-fast-forward), you must:
  - `git fetch`,
  - `git rebase` (or equivalent) onto canonical,
  - re-weave as needed on top of the updated filesystem state,
  - then push.

No merge commits that join two separate weave timelines.

---

### 4.3 Offline work is supported

You can:

1. clone a mesh,
2. edit, weave, and commit locally,
3. later push when back online.

If canonical advanced while you were offline:

- rebase your local commits onto canonical,
- reconcile filesystem conflicts,
- re-weave as needed,
- then push.

No merges; only rebased linear histories.

---

## 5. Mesh Integrity Expectations

### 5.1 Snapshots are immutable

Once a Snapshot directory (e.g. `…_v3/`) is created:

- its contents must not be edited by hand,
- any change to the Flow must go through `_working/` and a new weave,
- Git diffs should only ever see new snapshots added, not old ones mutated.

---

### 5.2 `_default/` mirrors the latest committed dataset state

After a successful weave:

- `_default/` must reflect the current authoritative dataset for the Flow:
  - for versioned flows, it matches the latest Snapshot,
  - for unversioned flows, it is *the* dataset state.
- `_working/` is reset from `_default/` to provide a clean starting point.

If Git rewrites `_default/` or `_working/` via rebase, the next weave will
normalize them by recomputing snapshots.

---

### 5.3 No parallel “next snapshots”

Two different weave runs from divergent Git histories cannot both claim to be
the next `_vN` for the same Flow in canonical.

- Rebasing chooses which sequence is “first.”
- The other must be re-woven atop the new canonical state.

---

## 6. Explicitly Allowed

- Uncommitted edits during weave.  
- Arbitrary local branches and WIP commits.  
- Editing `_working/` FlowShots directly.  
- Offline edit + weave + later push.  
- Rebasing local history onto canonical, followed by re-weave.

---

## 7. Explicitly Forbidden

- Merge commits that combine diverged histories on canonical.  
- Force-push to canonical.  
- Editing Snapshot directories in-place.  
- Any Git operation that makes Flow `sequenceNumber` or `_vN` ambiguous.  
- Merging two different weave timelines back into a single canonical line.

---

## 8. Summary

Semantic meshes impose a **small, strict** set of Git rules:

- One linear canonical history (fast-forward only, no force-push).  
- Divergent work is reconciled via rebase + re-weave, not merge.  
- Weave operates on FlowShots in the working tree, independent of commit state.  
- Snapshot ordinality is defined by weave runs along that single timeline.

These constraints keep filesystem-based weave semantics coherent and ensure that
every Flow evolves along a single, interpretable timeline.

You said:
I have this:

* Suffix selection logic:
  * Query existing snapshot labels for the same minute
  * Select lexicographically max existing label
  * Increment suffix
  * If no existing, suffix = empty string

BUT, querying existing snapshot labels for large meshes could be extremely expensive. Maybe we drop the suffix? Or change it to some kind of guid?
ChatGPT said:

You’re right to be suspicious of that algorithm as written. It’s only cheap if “query existing snapshot labels” is tiny and local; it becomes pathological as meshes grow.

Let’s separate the concerns and fix the design instead of papering over it with GUIDs.
1. What’s actually wrong with your current algorithm

        Query existing snapshot labels for the same minute

        Select lexicographically max existing label

        Increment suffix

The landmine is “query existing snapshot labels for the same minute” if interpreted as:

    “Scan all snapshots on disk / in the store for this minute, then compute max.”

For a large mesh:

    That query becomes O(#snapshots) or worse.

    If it’s a SPARQL query over a big meta-flow, it gets slower as the mesh grows.

    If it’s a filesystem scan per weave, you’re effectively doing a mini-index rebuild every time.

You only need one piece of information: “What was the last weaveLabel for this mesh (and how many weaves have I already done in this minute)?” You don’t need to re-derive that from all history.

So the problem is where you’re reading the information from, not the existence of a suffix.
2. Avoid the expensive query: store the state once

Instead of querying “all snapshot labels for same minute,” store a tiny bit of weave state in your metadata flow (per mesh, or per repo if you prefer):

    lastWeaveLabelBase (e.g. 2025-11-24_0142)

    lastWeaveOrdinalInMinute (integer ≥ 0)

Algorithm per weave:

    Compute currentBase from wall-clock: e.g. YYYY-MM-DD_HHmm.

    Load just the current metadata shot (no history scan).

    If lastWeaveLabelBase == currentBase:

        k = lastWeaveOrdinalInMinute + 1

    Else:

        k = 0

    Encode suffix from k:

        k = 0 → "" (no suffix)

        k = 1 → "a"

        k = 2 → "b"

        … with your base-26 scheme.

    New weaveLabel = currentBase + suffix.

    Write back:

        lastWeaveLabelBase = currentBase

        lastWeaveOrdinalInMinute = k

Cost is O(1): read one metadata shot, write one metadata shot. No global scan.

This assumes what you already said you want anyway:

    one weaver per mesh at a time (you already floated .weave-lock),

    no concurrent weaves that race to update metadata.

If that invariant holds, you don’t need any expensive lookups.
3. Should you drop the suffix?

Let’s be precise about what you lose or keep.

You already have:

    per-flow sequenceNumber (vN), monotonic and local to the Flow,

    consistent metadata per weave run (node metadata flow bumps each time).

The suffix gives you:

    the ability to see that within the same minute, there were multiple distinct weaves, and their order (…_0142, …_0142a, …_0142b).

    a visual cue that these snapshots belong to the “same minute cluster.”

If you drop the suffix and just use YYYY-MM-DD_HHmm:

    Multiple weaves in that minute share the same weaveLabel, even though they are different runs.

    You still have sequenceNumber per Flow, so ordinality is intact.

    But you can’t visually distinguish “first weave at 01:42” vs “second weave at 01:42” from the weaveLabel alone.

Given your stated expectation that:

    weaves are relatively infrequent,

    < 1 weave per minute is the common case,

this is probably acceptable. The suffix is “nice to have,” not structurally required, because sequenceNumber is the real ordinality.

So yes, you can safely drop the suffix and keep the logic trivial:

    weaveLabel = YYYY-MM-DD_HHmm

    no query, no suffix, no extra metadata beyond what you already store.

4. What about GUID-style suffixes?

You mentioned:

    “Or change it to some kind of guid?”

That would:

Pros:

    Avoid any need to query existing labels.

    Guarantee uniqueness even under concurrency.

Cons:

    You lose the human-friendly clustering you were explicitly aiming for.

    Lexicographic order no longer equals chronological order.

    The label stops being a “you can roughly see what happened when” affordance and becomes just another opaque ID — and you already have IRIs for that.

Given your original goals (human-recognizable, roughly sortable labels), a GUID suffix is overkill and undercuts the design.
5. Recommendation

You have two realistic, clean options:
Option A – Keep suffix, but make it O(1) via metadata state

Only change your implementation, not the external semantics:

    Store lastWeaveLabelBase + lastWeaveOrdinalInMinute in the node metadata flow.

    Derive the next suffix from that in constant time.

    Enforce “one active weaver per mesh” via .weave-lock or equivalent.

This preserves your minute+suffix design, is scalable, and keeps weaveLabel human-legible.
Option B – Drop suffix entirely

If you’re willing to accept that multiple weaves in a minute share the same weaveLabel:

    Drop suffix logic altogether.

    weaveLabel = YYYY-MM-DD_HHmm (or even YYYY-MM-DD_HH if you’re feeling bold).

    Rely on sequenceNumber for exact ordinality, and weaveLabel only as a coarse temporal grouping.

Given your emphasis on human affordances over perfect technical purity, I’d lean to:

    Pick A if you really care about the “a, b, c” nuance and you’re happy to write a tiny bit of metadata-management code.

    Pick B if you want to keep the system conceptually simple and accept that “twice in a minute” is rare enough that sharing a label is fine.

What absolutely doesn’t make sense is: keeping the current algorithm that requires scanning existing labels. That’s the part you should definitely kill.
You said:
With sflo-host, you can have multiple meshes loaded simultaneously. Should you be able to do cross-mesh weaves? Should you be able to do multiple intra-mesh weaves simultaneously? For big meshes and potentially expensive weaves (checking validity for large datasets), subtree weave resolution could be great. BUT, I was expecting weave lock to work at the mesh level. But I think we'll have to make it at the submesh level, so starting a weave, you might have to recurse up to the top of a mesh. Probably not too expensive, I don't really expect more than a few or maybe 10 levels of hierarchy.

If we expect weaves to take at least a second, we could include second-level precision in the weaveLabel, like 2025-11-25_0834_59_v1. 

ChatGPT said:

I’m going to be pretty blunt: you’re flirting with complexity here that you probably don’t actually need, especially around locking and cross-mesh semantics.

Let me tackle your questions directly and then fold in the weaveLabel precision point.
1. Multiple meshes loaded in sflo-host

Yes, sflo-host can (and should) handle multiple meshes in memory at once. That part is harmless.

The key question is what guarantees you want across meshes.
Cross-mesh weaves: should they exist?

Semantically, a “cross-mesh weave” means:

    you treat a set of meshes as if they were one logical unit for:

        weaveLabel assignment,

        provenance/activity records,

        and possibly “as-of” consistency.

To do that properly, you’d need:

    multi-mesh locking (lock mesh A + mesh B together),

    error handling that can unwind partial weaves if one mesh fails,

    provenance that explicitly says “this weave touched these N meshes”.

That is very close to a two-phase-commit–style system.

Given everything you’ve said so far (slow, human-triggered weaves, text-file store, Git-based history), I would say:

    Don’t introduce cross-mesh transactional weaves in v1.

    Treat each mesh as an independently weavable unit.

    If someone wants cross-mesh consistency, they can:

        orchestrate multiple per-mesh weaves from a higher-level script,

        and accept that there’s a window where one mesh is ahead of another.

You can still record that these weaves are conceptually related via provenance (meta:WeaveActivity pointing at multiple meshes), but you don’t need ACID-like behavior.

Recommendation:

    Yes to multiple meshes loaded.

    No to engine-level cross-mesh weaves (for now).

    Document cross-mesh consistency as best-effort orchestration, not a guarantee.

2. Concurrent intra-mesh weaves

You’re mixing two orthogonal questions:

    Can you weave different meshes in parallel?
    – Yes, that’s trivial: independent locks per mesh.

    Can you weave different parts of the same mesh in parallel?
    – That’s the hard one.

Mesh-wide vs submesh locks

Current default mental model:

    Mesh-level lock: only one weave per mesh at a time.

        Simple invariant: “a mesh’s timeline is always modified by exactly one weave at a time”.

        Easy to reason about: no risk of two weaves touching overlapping nodes/flows.

What you’re proposing:

    Submesh-level lock:

        Start a weave at some subtree root.

        Recurse up to find the mesh root (to know what you’re locking against).

        But actually, what you need is to ensure:

            no other weave is running on any ancestor or descendant of the subtree root.

So the true locking rule becomes:

    You can weave subtree S iff there is no active lock:

        on any ancestor of S (e.g., whole mesh root), and

        on any descendant of S.

Which means:

    You need a lock registry at mesh root anyway (some .weave-lock.json that lists all currently locked subtrees).

    Every weave has to:

        check that registry,

        ensure no ancestor/descendant conflicts,

        update it atomically.

This is not conceptually impossible, but you’re buying:

    more complex code,

    more places for deadlock or “stranded lock” bugs,

    and you still must handle the case where node metadata flows and global meta-flows are shared (see below).

The hidden coupling: metadata flows

You already said:

    metadata flows bump every time you weave,

    weave-level metadata is recorded in node metadata flows and/or in a meta-flow.

That means:

    Even if you weave “only subtree A”, you still touch:

        all flows in that subtree, and

        some higher-level metadata (e.g., repo-level weave log or root-node metadata).

So two “disjoint” subtree weaves may still both want to touch the same mesh-level meta-flow.

You either:

    forbid that and still effectively have a mesh-level bottleneck for some flows, or

    start splitting metadata into:

        per-subtree meta-flows,

        per-mesh meta-flows with their own locks,

        and so on.

At that point you’ve invented a partial transaction system by accident.
Is subtree weave resolution still worth it?

There are two distinct goals here:

    Scope resolution:
    “Only process this subtree, don’t touch other nodes.”
    – This is obviously useful and cheap. The weave engine can take a subtree root and ignore the rest of the mesh. No locking complexity required beyond “one weave per mesh.”

    Parallelization:
    “Process different subtrees in parallel within one mesh.”
    – This is where locking gets nasty.

Given your earlier assumptions:

    weaves are relatively heavy (validation, SHACL, etc.),

    operators are human,

    meshes are not twitch-level or HFT-level systems,

the probability that you truly need intra-mesh parallel weaves is low.

Recommendation:

    v1:

        Mesh-level lock only (one weave at a time per mesh).

        But subtree targeting is absolutely allowed:

            you can weave “just this subtree,” but when that is running, nobody else weaves anything in that mesh.

    If you later actually hit “we need parallel weaves,” you:

        introduce submesh locks, and

        redesign metadata flows to avoid global coupling.

Right now, I’d say submesh concurrency is an over-optimization for a system whose bottleneck is more likely human + SHACL than CPU.
3. WeaveLabel and precision

    “If we expect weaves to take at least a second, we could include second-level precision in the weaveLabel, like 2025-11-25_0834_59_v1.”

Given your evolving constraints:

    you’re likely to enforce one weave at a time per mesh,

    weaves are non-trivial (validation, I/O, etc.),

    sflo-host can still weave different meshes in parallel,

then:

    second-level precision (YYYY-MM-DD_HHmm_ss) is more than enough to avoid accidental label collisions at the mesh level.

    if, for some reason, two weaves start within the same second (e.g., automatic re-weaves / batch jobs), you still have:

        per-Flow sequenceNumber,

        per-Flow snapshot _vN suffix in the folder name.

In other words:

    weaveLabel ≈ human-readable temporal cluster ID;

    sequenceNumber and _vN ≈ true ordinality.

So you can safely:

    adopt YYYY-MM-DD_HHmm_ss (or your compact variant 2025-11-25_0834_59),

    keep _vN for ordinality,

    drop the per-minute suffix logic entirely, or keep it only as a fallback if you ever allow parallel intra-mesh weaves in the future.

Given a mesh-level lock, the per-minute suffix is unnecessary complexity.
4. Concrete answer to your questions

    “Should you be able to do cross-mesh weaves?”

    Conceptually: yes, as a coordination pattern, but:

        not as a single atomic operation in v1.

        treat it as “run weave on mesh A, then mesh B, record one meta WeaveActivity that points to both.”

    Engine-level transactional cross-mesh weaves: no (too much complexity for now).

    “Should you be able to do multiple intra-mesh weaves simultaneously?”

    No, not in v1.

    Support scoped weaves (subtree), but protect the entire mesh with a single .weave-lock.

    Do not attempt parallel submesh weaves until you have a demonstrated need.

    “WeaveLabel with second-level precision?”

    Yes, that’s a reasonable adjustment:

        e.g. 2025-11-25_08-34-59_v3 or 2025-11-25_083459_v3.

    With a mesh-level lock, you don’t need additional suffixes for uniqueness; _vN + provenance already gives you ordering and history.

If you want, I can write a short spec snippet for:

    mesh-level .weave-lock behavior, and

    a normative definition of weaveLabel format with second resolution and _vN.

You said:
The use case I always go back to is my Stagecraft role-playing experience platform. I imagine each player character getting their own mesh, and the simulation service does a kind of double-accounting where updates happen simultaneously in GM-owned (or system-owned) simulation mesh(es) and in the player-character meshes. So cross-mesh weave seems important.

Doing cross-mesh locking doesn't seem any more difficult than sub-mesh locking. The process just needs to check recursively up the tree, assuming we allow locking of entire submeshes based on a parent lock. I'm not sure if we support both "submesh locking" and explicit locks, but probably.

I'm not so sure that all weaves will be human-triggered, and they probably won't require interactivity. 
ChatGPT said:

You’re mixing three different questions here and treating them as one:

    Do we need cross-mesh weaves at all?

    Do we need submesh locking inside a mesh?

    What does that do to weave labels when weaves are automated and potentially frequent?

I’ll walk through them against the Stagecraft use case and point out where you’re about to overcomplicate things.
1. Cross-mesh weaves for Stagecraft

Your Stagecraft mental model:

    One or more GM/system meshes that hold the canonical simulation state.

    One PC mesh per character that holds player-facing state.

    The simulation service does double-entry updates: when something happens, the world mesh and the relevant character mesh(es) are updated “together”.

That absolutely does argue for a notion of cross-mesh weave – but note what that really means:

    Conceptually: “this weave run updates {mesh A, mesh B, …} as a single simulation event”.

    Implementation: “take a consistent set of FlowShots across those meshes, apply changes, publish new snapshots together”.

That does not require:

    submesh-level locking inside each mesh,

    or a fully general tree-locking scheme.

You can treat each PC root + each world root as an independent mesh unit for locking purposes, even if they live in a common Git repo and filesystem tree.

Concrete simplification:

    Define “mesh” at the level of the Stagecraft entities you care about:

        e.g. mesh:world, mesh:pc:gunaar, mesh:pc:nixx, etc.

    sflo-host maintains a lock per mesh root (logical mesh ID), not an arbitrary subtree.

    A cross-mesh weave is: “acquire locks on all involved mesh roots, run, release”.

That’s enough to support your double-accounting scenario.
Recommendation

    Yes, sflo-host should support cross-mesh weaves.

    Implement them as “weave groups” that lock multiple mesh roots at once.

    Don’t invent general submesh locking for this; just be precise about what counts as a “mesh root” in Stagecraft.

2. Submesh locking vs mesh-level locking

Your intuition:

    “Doing cross-mesh locking doesn't seem any more difficult than sub-mesh locking.”

Partially right:

    Locking multiple resources atomically (for cross-mesh) is straightforward if:

        you have a global lock registry, and

        you always acquire locks in a deterministic order to avoid deadlock.

Where it gets hairy is submesh locking combined with:

    recursion up the tree,

    plus other processes potentially locking other subtrees.

For Stagecraft, do you actually need submesh-level weaves? Think concretely:

    In your own words, each player character gets their own mesh.

    If the “mesh root” is the player character node, and you weave under that, that’s already as fine-grained as you need for PC-level updates.

    If a mesh contains many internal nodes (inventory, relationships, quest logs), those are still part of one logical PC mesh; you typically don’t need two weaves racing inside that one character.

The only strong argument for submesh locking is:

    a very large mesh where you want to weave disjoint subtrees in parallel (e.g. a single mega-world mesh where different regions are independent and you care about throughput).

That’s a different optimization problem than “keep world + PCs in sync”.
Recommendation

For now:

    Mesh-level locks only:

        one active weave per mesh root.

        cross-mesh weave = lock {mesh1, mesh2, …} together.

    Reserve submesh locks for a future “massive world mesh” scenario.

        If/when you go there, you’ll need a much more careful design around:

            ancestor/descendant conflicts,

            shared meta-flows,

            deadlock, stranded locks, etc.

Right now, submesh locking is a distraction from the Stagecraft pattern you actually care about.
3. Concurrency and automation

You also undercut an earlier assumption:

    “I’m not so sure that all weaves will be human-triggered, and they probably won't require interactivity.”

Good. That actually strengthens the case for:

    Well-defined locking rules, and

    Simple, predictable contention behavior.

But automated weaves don’t change the fundamental picture:

    You still want one weave at a time per mesh to keep FlowShot ordinality simple.

    You can run many weaves in parallel across different meshes (e.g. per character).

    A cross-mesh simulation step is just one more kind of weave:

        acquire locks: {mesh:world, mesh:pc:gunaar, mesh:pc:enemy123}

        run the sim update

        write snapshots

        release locks

The main extra thing you need for automation:

    A lock-acquisition order to avoid deadlock when multiple automated weavers want overlapping sets of meshes:

        e.g. sort mesh IDs and always lock in that order.

    A clear failure mode:

        if you can’t get all locks quickly (because some mesh is currently weaving), either:

            back off and retry, or

            treat it as “simulation tick skipped for this group”.

You don’t need submesh locking for any of that.
4. Weave label under automated, possibly frequent weaves

Given you’re moving toward:

    automated (non-interactive) weaves,

    possibly many events per minute per mesh,

    cross-mesh groups,

I’d refine the weaveLabel story this way:

    Keep ordinality in sequenceNumber per Flow (and _vN in the directory name).

        That’s your ground truth for “how many versions have there been”.

    Use weaveLabel as a human-readable group ID:

        likely attached to the weave activity and shared across meshes in that group.

        can be time-based, tick-based, or scenario-based.

Given we’re now talking about sim events, I’d decouple weaveLabel from “wall-clock minute” obsession:

    You can still use wall clock, but you’re allowed to have many weaves per second.

    You might even consider:

        2025-11-25T08:34:59Z#42 (timestamp + tick number),

        or session-17.turn-42, etc.

If you do keep time-based labels like 2025-11-25_0834_59_v3:

    Second-level precision is fine.

    With mesh-level locking, you do not need the minute-suffix gymnastics.

    Cross-mesh weave simply reuses the same weaveLabel across all affected meshes.

5. Direct answers to your latest points

    “So cross-mesh weave seems important.”

Agreed — for Stagecraft, yes. Implement it as “weave groups” that lock multiple meshes at once, not as a weird global transaction layer.

    “Doing cross-mesh locking doesn't seem any more difficult than sub-mesh locking.”

Correct if you:

    only lock at mesh root granularity, and

    always acquire locks in a consistent ordering.

It becomes significantly more complex if you combine cross-mesh + arbitrary submesh locks.

    “I'm not so sure that all weaves will be human-triggered…”

That’s an argument for simple, robust locking semantics, not more exotic ones. Automated weaves make it more important that your lock model is predictable and conservative.

If you want a next step, I’d specify (in a short doc):

    What counts as a “mesh root” (Stagecraft-flavored).

    The exact protocol for:

        acquiring multiple mesh locks (ordering, timeout/backoff),

        recording a weave group (shared weaveLabel, shared WeaveActivity).

    An explicit non-goal: “No submesh locking in v1; at most one weave per mesh at a time.”

You said:
But meshes can be large, weaves can be expensive. I'm imagining maybe best-effort "buffer writes while submesh is locked" to keep things consistent. I think we have to have more granular locking for weaves.

I don't see how we need one weave at a time per mesh to keep FlowShot ordinality simple. Ordinality is per node, so inherently simple. 

If you get a lock for a node, it can only be woven by the locking process, so one weave at a time. But if weaves end up taking minutes, and we buffer writes during locks...

Maybe weave is two-phase. Phase 1: lock nodes, copy _working to _current (and snapshot folder if versioning enabled); Phase 2: update changelogs, metadata, and generate resource pages, 
ChatGPT said:

You’re right that “one weave at a time per mesh” is over-conservative if meshes get big and weaves get expensive. But “buffer writes while submesh is locked” is where you’re wandering into dangerous territory.

Let me separate concerns and tighten the model instead of layering hacks on top.
1. Ordinality vs locking: what you actually need

    “I don't see how we need one weave at a time per mesh to keep FlowShot ordinality simple. Ordinality is per node, so inherently simple.”

Correct, if you enforce:

    “At most one weave touches a given node at a time.”

Then for each Flow:

    sequenceNumber is local to that Flow.

    Ordinality is just “last sequenceNumber + 1” for that Flow.

    It doesn’t matter what other nodes are doing.

So the real invariant you need is per-node (or per-flow) exclusivity, not mesh-wide exclusivity.

Where things go sideways is when you mix that with “buffer writes” and long-running weaves.
2. “Buffer writes while submesh is locked” is a trap

You’re imagining:

    Submesh lock is held for a long time (minutes).

    Writes targeting that submesh are somehow buffered and later applied “under” the weave.

Major problems:

    Where do writes go?

        Writes normally go to _working.

        If _working is “locked”, you either:

            block (bad UX, timeouts), or

            write them somewhere else, then replay → you’ve just invented an operation log / queue.

    What is the snapshot boundary?

        If you buffer, do buffered writes belong to:

            the current weave, or

            the next weave?

        Any answer creates weird interleavings unless you’re very strict.

    Failure modes become nasty:

        If the weave fails in Phase 2, what happens to buffered writes?

        If the process dies with a buffer on disk, you now need recovery semantics.

That’s a lot of machinery for something you can get much more cleanly by changing what “lock” means.
3. Better mental model: “weave cuts a snapshot; working is always open”

Instead of “lock submesh, freeze everything, buffer writes”, treat weave as:

    Take a cut of current _working state,

    materialize new Snapshots + DefaultShot from that cut,

    and do not try to freeze _working for the duration.

Concretely:

    Writers:

        always write to _working.

        never touch Snapshots or DefaultShot.

    Weaver:

        For each Flow in scope:

            read _working once,

            copy to new Snapshot folder (weaveLabel + vN),

            copy to _default/,

            bump sequenceNumber for that Flow.

        After that “cut”, _working is free to drift again; those new edits belong to the next weave.

You still need per-node (or per-flow) exclusivity during the cut so you don’t race with someone who is half-writing _working while you copy it, but that lock is short-lived:

    lock → copy → unlock (for that node),

    not “lock for the entire multi-minute weave”.

No buffering, no blocked writers, no op-log.
4. Two-phase weave: you’re almost there, but tighten the phases

Your sketch:

    “Maybe weave is two-phase.
    Phase 1: lock nodes, copy _working to _current (and snapshot folder if versioning enabled);
    Phase 2: update changelogs, metadata, and generate resource pages,”

This is the right idea, but you’re overloading “lock” with too much.

I’d rephrase it as:
Phase 1: Snapshot cut (short, exclusive per node)

For each node (or each Flow) in the weave scope:

    Acquire weave lock for that node:

        conceptually: “nobody else is taking a snapshot of this node right now”.

        writers may still be writing _working, but see below.

    Perform the cut for each Flow:

        open _working distribution file(s) once,

        copy to new Snapshot directory {flowRoot}/{weaveLabel_vN}/…,

        copy to _default/…,

        write minimal RDF in the Snapshot dataset:

            sequenceNumber,

            weaveLabel,

            prov:generatedAtTime.

    Release the node’s weave lock.

Key point: you must treat the _working read as atomic at the file level (e.g., open + read entire file). You do not need to block writes globally; if someone saves mid-copy, that’s a normal race in any file-based system. The expectation is:

    “If you edit during a weave, your changes may land in the next weave, not this one.”

That’s acceptable and simple, and it avoids buffering.
Phase 2: Non-exclusive, slower tasks

Now you have stable Snapshots + DefaultShots.

Phase 2 can be:

    update changelogs,

    update metadata flows beyond the minimal per-snapshot triples,

    generate resource pages,

    update any “weave log” nodes.

These:

    do not need to block writers on _working,

    and, if they fail, the Snapshots still exist and have valid minimal metadata.

If you want atomic visibility of “Phase 2 is done”, you can:

    either live with eventual consistency, or

    have a small status flag in appropriate metadata, updated at the end.

But that’s a separate concern from locking.
5. Granular locking: how fine do you go?

You’re right that we don’t need mesh-wide exclusivity anymore if:

    ordinality is per Flow,

    we cut Snapshots per node/Flow,

    and we define clear semantics around writes during a weave.

So your locking granularity choices:

    Per node (submesh):

        lock covers: the node and all its flows.

        you can weave different nodes in parallel inside a large mesh (as long as they don’t share flows).

    Per flow:

        more granular, but risk of complexity where NodeMetadataFlow and PayloadFlow are woven by different processes (do you ever want that?).

Given your design:

    Node = smallest unit you care about semantically,

    flows hang off the node,

it’s cleaner to say:

    Lock per node:

        “Weave of node X” holds the weave lock for X during its Phase-1 cut,

        anything writing _working for that node should either:

            be blocked briefly during the copy, or

            accept that its writes may be seen in this weave or the next, depending on timing.

I would not introduce buffering. If you really want to be strict, you can:

    block _working writes while the node’s weave lock is held, but that lock should be held for milliseconds, not minutes, if you keep Phase 1 small and push the heavy work into Phase 2.

6. How this plays with cross-mesh and big meshes

With this model:

    A cross-mesh weave:

        is just a weave whose scope spans nodes from multiple meshes.

        you acquire node locks for all involved nodes (across meshes) during Phase 1.

        you can still order lock acquisition to avoid deadlocks.

    A big mesh:

        can run multiple weaves in parallel as long as:

            their node sets don’t overlap.

        expensive validation/SHACL checks can live mostly in Phase 2, after the Snapshot cut, and can even be done on a worker that only reads the new Snapshots.

You get:

    more concurrency,

    simpler semantics:

        no buffering,

        no “one weave per mesh” bottleneck,

    and ordinality remains trivial: one sequenceNumber per Flow, incremented only when a new Snapshot is minted.

7. Concrete takeaway

You’re right to push for more granular locking given:

    large meshes,

    expensive weaves,

    possibly automated weaves.

But the way to get there is:

    Granular, short-lived locks for the snapshot cut (Phase 1),

    No buffering; writers always hit _working, and their writes become part of “the next weave that sees them”,

    Phase 2 for heavy, non-critical work: metadata, docs, changelogs, validation reporting.

If you’d like, I can draft:

    a pseudo-API spec for a “node-scoped weave” with the two phases spelled out,

    plus explicit rules about what happens to _working edits made during each phase.

You said:
We're on the right track here. So yeah, maybe we can't buffer writes easily so I'm okay with not trying that at first. But ideally we have some way the weave is consistent across nodes at a given time, i.e., we lock ALL the in-scope distributions from writes while we do the snapshot cut. And obviously, we want to prevent additional weaves until Phase 2 is complete. Can NodeJS lock each of the distribution files before it makes a copy?
ChatGPT said:

Short answer: Node can’t give you robust, cross-platform kernel file locks out of the box. You should not design sflo around “lock every distribution file with OS locks.” You’ll get a simpler and more reliable system by:

    enforcing who can write (sflo-host only, via API), and

    using application-level locks (per node / per weave) rather than per-file OS locks.

Let me walk through this against your constraints.
1. What you actually want from locking

From your last message, the invariants are:

    Weave snapshot consistency (Phase 1):
    For some scope S (set of nodes/flows), when you do the “snapshot cut”:

        All in-scope distributions are read in a way that is consistent with each other (no partial write interleavings).

        No other writes to those distributions occur during the cut.

    No overlapping weaves on that scope:
    While a weave is in Phase 1 or Phase 2 for scope S:

        No other weave run may start that touches the same nodes/flows.

    Writers are controlled:
    _working is only mutated by sflo-host (and possibly via CLI that talks to sflo-host), not random external processes editing files behind its back.

Given (3), you don’t need kernel locks on disk; you need mutual exclusion within your own process(es).
2. Node’s file locking reality

Node core:

    Does not give you a portable API like flock() or fcntl(F_SETLK) across platforms.

    fs.open, fs.read, fs.write do not lock; they only control how this handle behaves.

Your options for true OS locking are:

    Native addons / third-party modules that wrap flock/LockFileEx etc.

        Possible, but:

            deployment complexity (native builds),

            platform quirks,

            still requires every writer to participate correctly.

    Lock files / lock directories:

        Use fs.open(lockFile, 'wx') or fs.mkdir(lockDir) as an atomic “claim lock” operation.

        This is portable and purely Node; everyone who wants to write must:

            obtain the lock file/dir,

            do their work,

            then remove it.

Option (2) is usually enough for your use case, but note:

    You still only get mutual exclusion among cooperating code.

    If someone edits files by hand in VS Code, you can’t stop them anyway.

3. A better plan: lock nodes/flows, not each distribution file

Locking “ALL in-scope distributions” at the OS level is:

    expensive (lots of small locks),

    fragile (per-file error handling),

    overkill if all writes already flow through sflo-host.

Instead, do this:
3.1. Application-level locks

Introduce two logical locks:

    Node/Flow write lock

        Granularity: per node (covering its flows) or per flow if you really need that.

        Purpose: serialize the snapshot cut vs writes to _working.

    Weave-scope lock

        Granularity: per weave run, over a set of nodes.

        Purpose: prevent other weaves from overlapping the same nodes.

Implementation strategies:

    Single-process sflo-host (recommended initial model):

        Use an in-memory mutex/lock map:

            Map<NodeId, LockState> or Map<FlowId, LockState>.

            Writers and weavers both go through the same process, so this is enough.

    Multi-process / multi-host sflo-host (future):

        Add lock files per node/flow:

            e.g. <nodeRoot>/_locks/write.lock or <flowRoot>/_locks/weave.lock.

            Acquire via fs.mkdir or fs.open('wx').

            Release by removing the lock dir/file.

You do not need to lock individual .jsonld distributions if every write goes through these locks.
3.2. Phase 1 snapshot cut with these locks

For a given weave scope S (set of nodes):

    Acquire weave-scope lock (in memory or via a lockfile):

        e.g. at a mesh or submesh level: "mesh:world", "mesh:pc:gunaar".

    For each node in S:

        Acquire its node write lock.

        Read _working distributions once and copy to:

            new Snapshot directory (with weaveLabel_vN),

            _default/ directory.

        Update minimal RDF in the snapshot dataset (sequenceNumber, weaveLabel, prov:generatedAtTime).

        Release node write lock.

    After all nodes processed, keep weave-scope lock held until Phase 2 finishes.

This satisfies your consistency requirements without per-file kernel locks:

    While snapshotting a node, no writes to _working occur because writers must take the same node write lock.

    No other weave can overlap because the weave-scope lock covers the same nodes.

4. “Prevent additional weaves until Phase 2 is complete”

This is straightforward in the same model:

    A weave run has an ID, e.g. weaveRunId = weaveLabel + randomSuffix.

    You maintain a per-mesh (or per-root) active weave record:

        If mesh M has an active weave, no other weave that touches M may start.

    Phase 2 runs under that weave lock:

        changelog updates,

        metadata flows,

        resource page generation.

You don’t block writers during Phase 2, because they write _working, and Phase 2 shouldn’t be reading _working at all—it should operate on the new Snapshots and/or _default.

So:

    Weave lock gates new weaves.

    Node write locks gate reads/writes to _working during Phase 1 only.

5. Directly answering your concrete question

    “Can NodeJS lock each of the distribution files before it makes a copy?”

    There is no portable, built-in, kernel-level file locking in Node core that you can just flip on and forget.

    You can:

        use lock files / lock directories (atomic create) near each node/flow root, and treat those as your locks; or

        use native modules for flock/fcntl, at the cost of complexity.

But in your architecture:

    It’s much cleaner to:

        forbid direct external writes,

        funnel all writes through sflo-host,

        implement logical locks at node/flow and weave scope,

        and keep Phase 1 small so you don’t need to buffer writes.

If you want, next step could be a small spec snippet for:

    NodeWriteLockManager and

    WeaveScopeLockManager

in TypeScript terms, with clear rules on how writers and weavers interact.
You said:
But I want applications (and especially humans) to be able to mutate WorkingShot distributions directly, e.g. via text editor.
ChatGPT said:

Right now you’re trying to simultaneously demand:

    “Humans and apps can edit _working directly with a text editor at any time.”

    “Weaves get a consistent, cross-node cut of the world.”

    “We stop writes while we cut, ideally via file locks.”

You can’t have all three in a strong sense. If arbitrary tools can write the files, they will not cooperate with your locks or protocols. Node (and POSIX/Win32 generally) won’t magically protect you from vim or VS Code saving a file halfway through your copy.

So you need to decide which compromises you’re willing to live with.
1. What file locking can and cannot give you here

Even if you use an OS-level lock (via some native module wrapping flock / LockFileEx):

    Every cooperating writer that uses that API will block or fail if the file is locked.

    A plain text editor that just calls open(O_TRUNC|O_WRONLY) and write() will happily blow past any advisory locks.

There is no cross-platform, mandatory, “nobody can write this file at all” mechanism that Node can safely expose and every tool will honor.

So:

    NodeJS can give you lock files / lock directories or advisory locks for your own code.

    NodeJS cannot stop humans from saving a file at exactly the wrong moment.

If “edit _working directly with any editor” is non-negotiable, you have to design for detection and fail-fast, not perfect prevention.
2. Reasonable target: best-effort atomic weave, not frozen disk

Given that constraint, I’d reframe your invariants:

    No overlapping weaves on the same node(s):
    At most one weave that touches a given node is in progress at any time. That’s easy: sflo-host lock at node/submesh level, not per file.

    Weave is atomic from the published world’s perspective:
    A weave run either:

        publishes a complete, self-consistent set of Snapshots / DefaultShots, or

        fails and publishes nothing.

    Direct _working edits are allowed at all times, but are taken on a “best effort” cut:
    If someone saves while you’re cutting, you detect it and treat that as “weave aborted because concurrent edits”.

That’s different from “nobody can write while we cut,” but it’s something you can actually implement without lying to yourself.
3. How to do Phase 1 with external edits allowed

For each in-scope FlowShot (really: each _working distribution) in Phase 1:

    Stat + hash before read (cheap version can skip hash initially):

        Capture size, mtime (and optionally a cheap checksum).

    Read _working once:

        Either into memory, or copy directly into a temp file under a staging dir.

    Stat (and hash) again:

        If size or mtime changed during the read, or the hash changed:

            mark this flow as “dirty during weave”.

    After you’ve done this for all flows in scope:

        If any were dirty: abort the weave (Phase 1 fails, nothing is published).

        If none were dirty: treat the staged copies as your immutable snapshots.

Then:

    Only when Phase 1 succeeds do you:

        move the staged snapshots into their permanent weaveLabel_vN and _default/ locations (atomic rename per directory),

        bump sequenceNumber, etc.

This gives you:

    No buffering,

    No hard write blocking for editors,

    A clean all-or-nothing weave outcome, and

    A meaningful error you can surface in metadata and logs:

        “Weave failed: concurrent edits to _working during snapshot cut.”

From the human’s POV:

    If they are not editing at that exact moment, their changes are included.

    If they are editing at that exact moment, the weave bounces and they’ll have to try again.

That’s realistically as good as you get with free-for-all direct file edits.
4. Where Node-level locking still helps

Even though editors won’t respect locks, sflo-host itself should:

    Maintain application-level locks for:

        “weave in progress for node X / mesh Y” (so no second weave starts).

        “state mutation” operations (CLI, API) that also write _working.

Those locks can be:

    In-memory (single sflo-host process), or

    Lock files/dirs (multi-process / multi-host).

This means:

    sflo-host’s own mutations won’t race with its own weave logic.

    Only external editors are “uncooperative”, and for them you rely on the stat/hash detection above.

So the answer to your earlier desire:

    “Lock ALL the in-scope distributions from writes while we do the snapshot cut.”

is:

    You can lock them from sflo-host writes and weaves,

    You cannot lock them from arbitrary editors,

    But you can detect if editors touched them during the cut and abort the weave.

5. Phase 2 and preventing additional weaves

This one is simpler:

    Once Phase 1 succeeds, you:

        have staged snapshots in temp dirs,

        or already atomically renamed them into final locations.

While Phase 2 (changelogs, rich metadata, resource pages) runs:

    You hold a “weave lock” for that weave scope:

        sflo-host refuses to start another weave that touches those nodes.

    You do not need to block _working writes anymore:

        Phase 2 should operate only on the newly minted snapshots and _default/.

If Phase 2 fails:

    The snapshots are still valid,

    You either:

        mark “weave incomplete” in metadata, or

        roll back any partial Phase 2 artifacts (e.g., re-generate on next weave).

6. Direct answer to your question

    “Can NodeJS lock each of the distribution files before it makes a copy?”

Practically, for your requirements:

    You should not rely on per-file OS locks as the main mechanism.

    You should:

        use sflo-host–level locks for weaves and its own writes,

        allow arbitrary editors to write _working at any time,

        detect concurrent modification via stat/hash before/after,

        and abort or retry the weave run if any file changed during the cut.

If you want, next step could be:

    Define a precise Phase-1 algorithm for a “multi-node weave with external edits allowed”:

        exact stat/hash fields,

        when to mark a weave as failed,

        how to report that into the NodeMetadataFlow/meta-flow.

You said:
OK, sounds good. I have a document file called concept.weave-process.weave-lock that's empty. But weave-lock is maybe not the right term. I also have this existing concept.weave-process doc, maybe you can update it for me?

---
id: rall4fbxm369okmy5383sf8
title: Weave Process
desc: ''
updated: 1762707513747
created: 1751128698638
---

## Functionality

- checks for required [[facet.system]] folders and files and creates them if missing
- optionally removes extraneous files, interactively if requested
- for changed [[facet.user]] datasets (i.e., need version bump)
  - if versioning is on:
    - creates a new [[mesh-resource.node-component.flow-snapshot.version]] 
    - updates version metadata
  - regardless of whether versioning is on:
    - copies _working to _default
    - updates _node-metadata-flow with new version information
- regenerates affected [[mesh-resource.node-component.documentation-resource.resource-page]]
- ensure that [[folder.node]] are only contained in other [[folder.node]].
- detect [[transpositions|principle.transposability]] and fix any 



file
/repo-root/
├── _assets/
│   ├── _templates/
│   │   ├── default.html
│   │   ├── ontology.html
│   │   └── person.html
│   └── _css
│   │   ├── default.css
│   │   ├── ontology.css
│   │   └── person.css
└── my-ontology/
    ├── _config-operational-flow  ← Node config
    ├── _assets/                  ← Optional node-specific assets
    │   ├── _templates/           ← Optional templates
    │   └── _css                  ← Optional css
    └── _payload-flow



## Quirks

- if there's a pre-existing index.html under _assets, don't overwrite it?
  - or maybe use AI to update it.
  - (might need some marker html to indicate index.html was generated by weave process)

## Best Practices

### Weave Before Push

This ensures that in published meshes and sites:

- broken references are cleaned up
- [[mesh-resource.node-component.flow-snapshot.default]] is identical to the latest version

## Features

### Interactive Mode

- to encourage data quality, the weave process can include an interactive mode, or modes. Perhaps one mode is naive, like "step through every bit of metadata" or "review every inferred metadata item", but you could have an AI-driven mode that optimizes for importance, identifying possible errors based on usage, etc.

### Tombstoning

If you know a sub-mesh is permanently moving to a new location (or even if a branch is being created somewhere else), you should be able to tell weave to insert references to the new location

### Resource Page Generation

- uses the [[product.service.design.in-memory]] to calculate template usage
- if no templates specified, and no "default template" exists in the root, it can generate its own
  - perhaps there's a default template and css distributed with the service in case its missing from the mesh root

## Scope

- **Single flow weave**: Update one (user) flow (data, ref, or config) + the corresponding meta-flow
- **Node weave**: Update all (user) flows in a node (each flow + meta gets woven)
- **Node tree weave**: Recursively weave nodes and their contained nodes

**Meta-flow co-weaving**: whenever any flow in a node gets woven, the meta-flow updates to reflect the new state. This ensures the meta-flow always accurately describes the current node state without requiring separate meta-weaving operations.

This gives you:
- **Local consistency**: Each node stays internally consistent
- **Flexible granularity**: Can weave at flow, node, or tree level as needed
- **Automatic meta updates**: No manual meta-flow management
- **Simple model**: No complex cross-node locking or coordination

For contained nodes mentioned in config flows, any references ideally point to the appropriate version snapshot. If there's inconsistency during concurrent operations, it's temporary and resolves when operations complete (.7).

This seems like the right balance between consistency guarantees and implementation complexity for the mesh architecture. We can always add more sophisticated coordination later if specific use cases demand it.


ChatGPT said:

Below is a rewritten and updated “Weave Process” page, incorporating:

    FlowShots (SnapshotShot / DefaultShot / WorkingShot)

    snapshot cut semantics (Phase 1 / Phase 2)

    submesh-locking model

    concurrent external edits allowed but detected

    flow/node/tree scopes

    meta-flow responsibilities

    new weaveLabel + sequenceNumber

    no assumptions that weave creates commits

    no “uncommitted changes” requirement

    correct relationship between _working, _default, and snapshot folders

    correct invariants for versioned and unversioned flows

    removal of outdated language about “version bumps”

    integration of your new architecture for cross-mesh and subtree weaves

This is built to be used as-is in your knowledge base.
Weave Process

(concept.weave-process)
Purpose

The weave process transforms in-progress Flow data (the WorkingShot) into:

    a new immutable SnapshotShot (if versioning is enabled),

    an updated DefaultShot (always),

    updated metadata (meta-flow),

    regenerated resource pages,

    and a stable, cross-node consistent “cut” of the mesh state.

Weave provides the atomic publication boundary of a mesh: everything within the weave scope becomes internally consistent, visible to other tools, and ready for downstream consumption.
Conceptual Model

A Flow is a dcat:DatasetSeries.
A Flow always contains three permanent subdirectories:

    _working/ – mutable; edited by humans & apps

    _default/ – mutable; holds latest published FlowShot

    Snapshot folders – immutable; each a FlowShot with weaveLabel + sequenceNumber

Weave operates on FlowShots:

    sflo:WorkingShot

    sflo:DefaultShot

    sflo:SnapshotShot (immutable)

A weave run never deletes snapshots; it only produces a new one (if enabled) and refreshes _default/.
Weave Outcomes

Every weave produces:

    A consistent cut of all _working/ FlowShots in scope
    → optionally stored as a new immutable SnapshotShot

    A refreshed _default/ FlowShot
    → always updated to match the weave result

    Updated meta-flow
    → includes:

        new weaveLabel

        new sequenceNumber

        previousFlowShot

        provenance info
        (Only includes latest provenance; older provenance lives in older snapshots.)

    Regenerated Resource Pages
    → stable HTML index pages for nodes and flows

    (Optional) Tombstone Updates
    → for submesh relocation or branch rehoming

Two-Phase Weave

Weave is structured as:
Phase 1 — Snapshot Cut (Atomic Input Capture)

Goal: capture a consistent, static copy of all FlowShots being woven.

Steps:

    Acquire weave locks for all nodes/flows in scope

        Subtree locking permitted

        Cross-mesh locking permitted

        Locks prevent other weaves and sflo-host writes

    For each _working/ FlowShot:

        Stat + hash before read

        Copy contents into a staging directory

        Stat + hash again

        If changed during read → Phase 1 abort (dirty during weave)

    If all FlowShots are stable:

        Create the new weaveLabel

        Increment sequenceNumber

        Create new SnapshotShot folders (if versioning enabled)

        Create new DefaultShot folders (always)

At the end of Phase 1:

    You have a complete staging area containing the cut.

    Nothing has been published yet.

    No writes into the published mesh have occurred.

If Phase 1 fails:

    No mesh state changes.

    Error appears in meta-flow logs on next successful weave.

Phase 2 — Materialization (Atomic Output Publish)

Goal: publish the staged FlowShots as the official new state.

Steps:

    Atomic rename (per directory) from staging into:

        …/<weaveLabel_vN>/ (SnapshotShot)

        …/_default/ (DefaultShot)

    Update meta-flow:

        sflo:weaveLabel

        sflo:sequenceNumber

        sflo:previousFlowShot

        provenance fields
        (Only latest provenance is stored; snapshots retain historical provenance.)

    Generate documentation Resource Pages

    Release weave locks

If Phase 2 fails:

    Snapshots are still valid (they were already atomically written)

    Resource pages may be regenerated in a later weave

    Meta-flow will reflect incomplete status until next weave

Locking Model
Weave locks are at the node (or submesh) level, not per-file.

This prevents:

    simultaneous weaves in the same submesh

    sflo-host writes colliding with weave

Editors may still modify _working, which is why Phase 1 uses stat/hash detection.
Cross-mesh weaves are allowed

sflo-host must acquire locks across all referenced meshes before Phase 1 begins.
Lock propagation

Requesting a lock for a node implicitly requires locking all ancestors up to the mesh root.
This prevents parallel weaves on overlapping submeshes.
Stability Guarantees

Weave guarantees:

    Atomic publication: readers see the new state only after Phase 2 completes.

    Consistency within the weave scope: all flows included in the weave are aligned to the same cut.

    Monotonic sequenceNumber per Flow: ordinality remains clean even with multiple meshes.

    Weave labels sorted lexicographically within each minute (if using suffix model).

Weave explicitly does not guarantee:

    prevention of human text editors saving during the cut
    (→ but it will detect and abort)

Scope Levels
Flow Weave

    Affects one flow + its meta-flow

Node Weave

    Weaves all flows under one node

    Meta-flow updates accordingly

Node Tree Weave

    Recursively weaves node and descendants

    Ideal for large cohesive units (directories)

Cross-Mesh Weave

    Allowed

    Good for Stagecraft’s multi-character simulations where updates must hit player-character mesh + GM-owned simulation mesh simultaneously

Locking spans both meshes, forming a single weave consistency domain.
DefaultShot Behavior

    _default/ is a real FlowShot

    It is not a pointer; it contains real Folder/Files

    It always mirrors the latest SnapshotShot (or staged result if versioning is off)

    Its IRI is stable, so tools may dereference it without knowing weave labels

Unversioned flows

    Still produce a DefaultShot

    You can treat _default/ as the active published dataset

WorkingShot Behavior

    Always present

    Always mutable

    Not modeled as an RDF class (optional)

    Synchronized to _default/ after each weave

    Errors or partial edits allowed between weaves

Features Preserved From Older Model
Interactive Mode

    Step-through validation

    AI-assisted metadata review

Tombstoning

    Node relocation support

Resource Page Generation

    Uses in-memory config cascade and templates

    Node-specific templates override mesh-level templates

Transposition Detection

    Ensures mesh transposability

    Fixes any IRI base issues or structural inconsistencies

Best Practices
“Weave Often”

Produces clean snapshots and stable metadata.
“Weave Before Push”

Ensures _default/ is in sync with mesh state before publishing.
“Keep _working Small”

Because direct edits are allowed, large files increase dirty-during-weave likelihood.
Quirks

    Respect pre-existing hand-written index.html under _assets/
    (unless explicitly allowed to regenerate)

    Generated pages may include an invisible marker to allow safe regeneration

File & Directory Expectations

/repo-root/
├── _assets/
│   ├── _templates/
│   │   ├── default.html
│   │   ├── ontology.html
│   │   └── person.html
│   └── _css/
│       ├── default.css
│       ├── ontology.css
│       └── person.css
└── my-ontology/
    ├── _config-operational-flow/
    ├── _assets/
    │   ├── _templates/
    │   └── _css/
    └── _payload-flow/

If you'd like, I can also generate:

    A parallel document called concept.weave-process.locking

    A process flow diagram

    A SHACL model for FlowShots

    A step-by-step pseudo-code spec of the weave engine

You said:
Thanks. Don't rewrite it, but I wanted to ask your opinion: to replace SnapshotShot more concisely, should we do Snapshot or SnapShot ?
ChatGPT said:

Snapshot is the correct choice.

Reasons:

    CamelCase capital-S SnapShot looks like a typo pattern—your ontology already uses consistent CamelCase (FlowShot, WorkingShot, DefaultShot). Within that system, compounds are formed as Flow + Shot, not Flow + flowShot.

    Snapshot is already a standard term in DCAT-ish, Git-ish, filesystem-ish semantics. Using the canonical spelling increases recognizability and lowers cognitive friction.

    SnapshotShot was only noisy because of the doubled “shot”, not because “snapshot” was wrong. Reverting to “Snapshot” gives you the clean pair:

        FlowShot

            Snapshot

            DefaultShot

            WorkingShot

    Internal consistency:

        FlowShot = abstract superclass

        Snapshot = committed cut

        DefaultShot = published cut

        WorkingShot = mutable working cut

No other term in the hierarchy uses two capital letters mid-word.
So Snapshot is the only spelling that stays consistent, readable, and idiomatic.

If you want an even cleaner trio, I can show variants like:

    SnapshotShot → Snapshot

    DefaultSnapshot

    WorkingSnapshot

…but only if you want stronger naming parallelism.
You said:
In terms of the weave label, where did we land? Seconds instead of suffix?
ChatGPT said:

Where we landed, distilled:
1. Seconds-level timestamp is the right backbone

YYYY-MM-DD_HHMM_SS
This gives you:

    human readability

    natural grouping of all FlowShots created in the same weave

    monotonicity

    no suffix lookup

    trivial to generate

    works well even if weaves take < 1 second (rare)

Example:
2025-11-25_0834_59
2. Ordinal _vN stays

You decided (correctly) that:

    ordinal per-flow sequence numbers are still valuable

    _vN should appear in the folder/IRI, not just in metadata

    it's cosmetic but also helpful for sanity checks, tooling verification, and debugging

So a snapshot folder becomes:

2025-11-25_0834_59_v3

The monotonic integer in RDF remains the authoritative version order.
The _v3 in the label is simply a convenience mirror.
3. We dropped suffix letters

Your concerns were correct:

    to detect “existing suffixes for this minute” you must look at all snapshots in the flow, possibly all flows in a mesh

    that scaling cost is unacceptable for large meshes

    suffix increment logic requires read-compare-write coordination, which undermines transposability

    minutes granularity is too coarse anyway for parallel weaves or automated weaves

    seconds-level solves the collision issue cleanly

So the final position was:

    Use second-level resolution to avoid collisions; drop suffixes entirely; do not attempt letter suffix scanning or prefix-max lookups.

4. Final form

weaveLabel = <timestamp-to-second> + "_v" + sequenceNumber

Examples:

    2025-11-25_0834_59_v1

    2025-11-25_0834_59_v2

    2025-11-25_0835_00_v1

5. Why this is the correct trade-off

    No scanning

    No global coordination

    No suffix collision handling logic

    Perfectly readable

    Universally sortable (timestamp first, ordinal second)

    Easy to mint in a distributed system

    Cross-mesh safe

    No need for per-mesh “current suffix cursor”

6. Optional note

If you ever wanted sub-second resolution, you could extend with _SSS (milliseconds) but you shouldn’t need that unless people start weaving at >1/sec, which is improbable.

If you want, I can update the ontology text for the weaveLabel comment to reflect the final semantic definition.


ChatGPT can make mistakes. Check important info.
