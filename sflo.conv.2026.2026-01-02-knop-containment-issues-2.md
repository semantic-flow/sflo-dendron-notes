---
id: ouggsp2lvbuws4muakw99b4
title: 2026 01 02 Knop Containment Issues 2
desc: ''
updated: 1767439235212
created: 1767439231831
---

You said:
What do you make of "In DCAT 1 [VOCAB-DCAT-1] dcat:Dataset was a sub-class of dctype:Dataset, which is a member of the DCMI Types vocabulary [DCTERMS]. The scope of dcat:Dataset also includes other members of the DCMI Types vocabulary, such as various multimedia (imagery, sound, video) and text, so the sub-class relationship defined in DCAT 1 [VOCAB-DCAT-1] was removed in the DCAT 2 vocabulary - see Issue #98" -- can a binary image be a dataset?
ChatGPT said:

Short answer: yes, in DCAT a binary image can be a dataset — deliberately so — but that does not mean DCAT is saying “any file is a dataset,” nor that you should treat arbitrary payload bytes as datasets.

Here’s the careful unpacking.
What that DCAT passage is really saying

The quoted text reflects a deliberate broadening of dcat:Dataset between DCAT 1 and DCAT 2+.
DCAT 1

    dcat:Dataset ⊑ dctype:Dataset

    dctype:Dataset had a document-centric bias (collections of data, often tabular).

    This caused friction when people wanted to publish:

        satellite imagery

        audio collections

        videos

        single large scientific files

DCAT 2 change

    The subclass link was removed.

    Rationale: DCAT datasets should cover any publishable data asset, not just “rows-and-columns data.”

    The spec explicitly says the scope includes:

        imagery

        sound

        video

        text

        other media

This is why the question “can an image be a dataset?” has the answer “yes” in DCAT.
But “yes” does NOT mean what people often think it means
DCAT is not saying:

    “Every file is a dataset”

    “Binary blob == dataset”

    “Distributions and datasets are interchangeable”

DCAT is saying:

    If you are publishing an image (or audio, or video) as a data asset, with metadata, provenance, access, licensing, etc., then it is valid to model it as a dcat:Dataset.

In other words:

    Dataset-ness in DCAT is about publication intent, not file format.

Dataset vs Distribution still matters (a lot)

Even with this broad scope, DCAT still keeps a sharp conceptual line:

    dcat:Dataset
    = the thing being published (the asset, abstractly)

    dcat:Distribution
    = a representation of that dataset (format, encoding, access method)

So for a single image:

    Dataset: “Landsat image of area X on date Y”

    Distribution(s):

        GeoTIFF file

        JPEG preview

        IIIF service endpoint

That’s still the DCAT mental model.
Where this intersects with your ontology

This passage supports your instincts more than it undermines them.
It supports:

    Treating non-tabular, non-RDF, even binary things as datasets if and only if they are being published as datasets.

    Having dataset identity separate from file identity.

It does not require:

    Treating every payload byte sequence as a dataset.

    Treating every file as a dcat:Distribution.

    Collapsing “file” and “dataset” concepts.

In fact, DCAT’s own architecture assumes that dataset identity is not the same as file identity.
Applying this cleanly to your model

Your design already distinguishes three layers that DCAT conflates unless you’re careful:

    Payload bytes
    → your AbstractFile + LocatedFile
    → no DCAT semantics required

    Dataset identity (sometimes evolving)
    → KnopDataset, Flow, Version, OpslogDataset
    → may or may not map to DCAT

    DCAT publication view
    → opt-in overlay when you want interoperability

That’s exactly the right separation.

So your conclusion earlier remains correct:

    Do not make AbstractFile ⊑ dcat:Distribution.

    Allow some AbstractFile instances to also be DCAT Distributions, via a subclass or dual-typing, when publication intent warrants it.

Direct answer to the question

    Can a binary image be a dataset?

In DCAT terms:
Yes — if it is being published as a data asset.

In Semantic Flow terms:
Only if you decide that the image is a KnopDataset (or a Version, or an OpslogDataset, etc.), not merely a payload artifact.

That distinction is exactly why your model is stronger than raw DCAT.

If you want, next we can:

    formalize a “DCAT export profile” for Semantic Flow, or

    pin down exactly which SF classes are eligible to be DCAT datasets vs distributions, without leaking semantics.

You said:
So this is ok:

{
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": "KnopDataset/",
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Dataset (including Versions and current-only datasets) to one of its AbstractFiles."
    }
ChatGPT said:

Mostly, but not as-written given the other decisions you’ve made.
What’s OK about it

    Domain KnopDataset/ is right if you intend: any SF dataset (versions, pointers, opslog datasets, etc.) can have one or more abstract-file artifacts.

    Range AbstractFile/ is right for your “bytes-in-the-abstract with located endpoints” concept.

    Making it a subproperty of some SF-scoped containment predicate is fine if you really are minting the AbstractFile IRIs under the dataset path (i.e., you want to treat them as structurally contained resources).

What’s not OK / needs adjustment
1) rdfs:subPropertyOf containsSfloResource/ is only correct if AbstractFile is a structural child

Your containsSfloResource is “structural containment” and you’ve said it’s only asserted/derived within SF container boundaries.

So this subproperty claim asserts:

    Every hasAbstractFile link is a structural containment link.

That’s a strong commitment. It’s correct only if:

    every AbstractFile has its own container IRI under the dataset (e.g. <dataset/_dist/foo.jsonld/>), and

    you want to treat those as “contained SF resources,” not merely related artifacts.

If sometimes the AbstractFile is conceptual-only (not minted as a container resource) or lives elsewhere, then this becomes false.

If you want flexibility, do not make it a subproperty of containment; instead, keep it as a plain relation.
2) Your comment still says “Dataset” (generic) rather than “KnopDataset”

Not fatal, but you just renamed for a reason—your labels/comments should enforce the distinction.
3) Consider tightening the domain to “Concrete/Snapshot datasets” if you introduce that axis

Right now Flow is a KnopDataset. Do you want a Flow itself to “have abstract files,” or only its Versions?

    If only Versions have files, domain should not be KnopDataset/; it should be Version/ (or the future ConcreteKnopDataset/).

    If Flows can also have their own distributions (e.g., “current convenience copy”), then domain KnopDataset/ is fine.

You need to pick one semantic rule.
Recommended corrected form (minimal, keeps your current design)

If you want to keep the containment commitment:

    Keep as-is but fix the comment:

{
  "@id": "hasAbstractFile/",
  "@type": "owl:ObjectProperty",
  "rdfs:subPropertyOf": "containsSfloResource/",
  "rdfs:domain": "KnopDataset/",
  "rdfs:range": "AbstractFile/",
  "rdfs:label": "has abstract file",
  "rdfs:comment": "Links a KnopDataset (including Versions and current-only datasets) to one of its structurally contained AbstractFiles (artifact containers) within the Knop boundary."
}

If you want flexibility (I suspect you do), drop the subProperty link:

"rdfs:subPropertyOf": []

(or omit it), and keep containsSfloResource for cases where you truly mean path/structural containment.
The key decision you must make

Do Flows have AbstractFiles, or only Versions / concrete datasets?

    If only concrete datasets: change domain to Version/ (or a future ConcreteKnopDataset/).

    If both: keep KnopDataset/.

Right now your ontology says “both,” whether you meant it or not.
You said:
I'm not sure we need the flexibility for our purposes, but maybe AbstractFile could be more generally useful. 

One thing that's interesting, is that PayloadFlows represent an AbstractFile over time, Payload Versions are kind of overlapping AbstractFile.
ChatGPT said:

You’ve hit a real structural tension: you have two “axes of evolution” in the model:

    Dataset evolution (Flow → Versions)

    File/bytes evolution (an “abstract file over time”)

Right now you’re trying to express (2) using (1), but you also have AbstractFile as a timeless “exact byte sequence” container. That creates overlap/confusion unless you tighten definitions.
1) What you just said implies a category mismatch

    “PayloadFlows represent an AbstractFile over time”

But your PayloadFlow is currently:

    a Flow

    therefore a KnopDataset

    therefore a dcat:Dataset

Whereas AbstractFile is your “exact bytes” thing (and you were considering disconnecting it from DCAT Distribution).

If a payload flow is “an abstract file over time,” then it is not primarily a dataset identity; it’s primarily a file lineage.

So either:

    you accept that “payload flow is a dataset identity whose distributions are files that evolve,” or

    you introduce a distinct “file lineage” concept and stop pretending it’s a dataset flow.

Right now you’re oscillating between these, and the ontology can’t be coherent until you pick.
2) The overlap you’re noticing (“Payload Versions are overlapping AbstractFile”) is a symptom

A Version (dataset snapshot) has distributions (serializations) which are files.
A payload “version” is also “a file snapshot” in practice.

So you have:

    Version (dataset snapshot)

    AbstractFile (file snapshot)

If payload is “arbitrary artifact” (not necessarily RDF dataset), then calling the snapshots “Versions” of a dataset is already suspicious.
If payload is arbitrary bytes, the clean model is:

    PayloadFlow is a FileFlow

    Its members are FileRevisions (or PayloadRevisions)

    Each revision has one AbstractFile (the bytes for that revision)

Trying to model this as dataset versions will keep biting you.
3) Decide what PayloadFlow actually is (there are only two consistent options)
Option A (Dataset-centric; keep your current class tree)

PayloadFlow is a dataset identity (dcat:Dataset) whose versions are datasets.

    Payload Version = dataset snapshot

    Each Payload Version can have one or more AbstractFiles (distributions)

    “AbstractFile over time” is not a first-class thing; it is derived from the sequence of distributions across versions.

This works only if payload is conceptually a dataset (even if binary), i.e. you are comfortable with “a JPEG file as a dataset.” DCAT allows this; you may still dislike the semantics, but it’s permissible.
Option B (File-centric; introduce a parallel lineage concept)

PayloadFlow is not a dataset flow; it is a file lineage.

    It should not be a subclass of KnopDataset.

    It should evolve through FileRevisions, not Version.

    Each FileRevision points to exactly one AbstractFile (bytes).

    ReferenceFlow remains dataset-centric; payload is artifact-centric.

This matches your earlier insistence: payloads you don’t want to call distributions/datasets.

Given your stated discomfort with calling payloads “distributions,” Option B is more consistent with your values.
4) A third “hybrid” approach exists but is brittle

You can keep PayloadFlow as a Flow (dataset), but define:

    PayloadFlow’s “dataset content” is just metadata about the payload artifact

    The actual bytes are the referent artifact evolving

Then the Flow is about a “DigitalArtifact identity” and versions are metadata snapshots, not the bytes themselves.

But you’ll immediately need:

    payloadRevision or hasPayloadAbstractFile rules

    a separation between “payload metadata dataset” vs “payload bytes”

This becomes complicated fast and tends to collapse back into Option B anyway.
5) How this affects hasAbstractFile and containment

If you go with Option A (dataset-centric):

    it’s totally reasonable to say KnopDataset hasAbstractFile AbstractFile and treat it as contained (subPropertyOf containsSfloResource)—you’ll mint distributions under the dataset.

If you go with Option B (file-centric lineage):

    hasAbstractFile should still exist, but its domain should include the file lineage member class, not just KnopDataset.

    And you may want AbstractFile as a general SF artifact container, not necessarily tied to datasets.

That’s exactly the “flexibility” you were intuiting.
What I think (given your previous constraints)

You keep saying you don’t want payload files called distributions and you want single-referent clarity. Those push strongly toward Option B:

    Keep AbstractFile as an SF concept (not DCAT Distribution).

    Keep dataset flows for reference/meta/backlinks/etc.

    Introduce a parallel lineage for payload artifacts (file evolution).

If you don’t want to add new primitives yet, then the minimum consistent interim stance is Option A: treat payload as a dataset (DCAT permits binary datasets), accept that “payload flow versions” are datasets, and stop calling PayloadFlow “an abstract file over time” in prose. But that’s you lying to yourself in spec language; it’ll resurface later.
Minimal next step to resolve the tension without a huge refactor

Pick one sentence and bake it into the ontology comments (normative-ish):

    A: “PayloadFlow is a dataset identity; its versions are dataset snapshots whose representations are AbstractFiles.”

    B: “PayloadFlow is a file lineage; it is not a KnopDataset; its members are file revisions each realized by an AbstractFile.”

If you tell me which sentence you can live with, I can tell you the exact ontology diffs needed (and they are very different).
You said:
Instead of introducting FileFlow, we could just make Flows and Versions NOT datasets. Or we capture arbitrary dataset in PayloadFlow, and arbitrary other digital Artifacts as PayloadArtifacts. Then we maybe differentiate between DatasetVersions and ArtifactVersions. 

Flows do not have AbstractFile directly. But KnopDatasets that aren't versions, like OpslogDataset DO have AbstractFiles. 

Put a pin in this: differentiating carefully between structural (file/IRI) containment and conceptual containment. the refererent-iris structurally contain knops, but not conceptually.

ChatGPT said:

You’re circling the real fault line: you’re using “dataset” to mean two incompatible things:

    “thing with distributions / bytes / formats” (artifact-ish)

    “RDF graphs / data model thing” (dataset-ish)

DCAT muddies this on purpose; your ontology can’t if you want single-referent clarity.

Below is what needs tightening, and the least-painful way to do it.
1) “Make Flows and Versions NOT datasets” is viable, but it forces a rename and a DCAT break

If you remove Flow ⊑ KnopDataset ⊑ dcat:Dataset and Version ⊑ dcat:Dataset, you must answer:

    What are Flow and Version then?

They become structural temporal containers, not datasets. That’s coherent in your system (flow = evolution container; version = snapshot container), but then:

    You should stop using dcat:hasVersion, dcat:previousVersion (those are dataset-version relations).

    You need a different base class than KnopDataset for Flow/Version.

Minimal refactor if you go this route

Introduce:

    KnopDataResource/ (or KnopData/) – “a data-bearing SF resource”

    Flow/ ⊑ KnopDataResource

    Version/ ⊑ KnopDataResource

Then separately define:

    RdfDataset/ ⊑ KnopDataResource (or KnopRdfDataset/)

    Artifact/ ⊑ KnopDataResource (optional)

This avoids pretending the temporal structure is itself a dataset.

Cost: you lose the convenient DCAT alignment for flows/versions, but that alignment was already semantically strained.
2) Split payload into two lines: PayloadFlow for datasets, PayloadArtifact for non-datasets

This is the cleanest option if you want to keep “dataset” meaningful.

    PayloadFlow = only when payload is a dataset (including binary-as-dataset if you accept it).

    PayloadArtifact (or PayloadArtifactFlow) = when payload is a non-dataset digital artifact.

Then you need two “member” types:

    DatasetVersion (snapshot of a dataset identity)

    ArtifactRevision (snapshot of an artifact identity)

You can keep the generic structure “Flow has members” but don’t force all members to be Version/ if Version/ is dataset-specific.
What to do in ontology terms

    Rename your current Version/ to DatasetVersion/ (or introduce it as a subclass of Version and then migrate).

    Introduce ArtifactVersion/ (or ArtifactRevision/).

Then:

    DatasetFlow hasDatasetVersion DatasetVersion

    ArtifactFlow hasArtifactVersion ArtifactVersion

Or keep generic:

    Flow hasMember FlowMember

    DatasetFlow ⊑ Flow, ArtifactFlow ⊑ Flow

    DatasetVersion ⊑ FlowMember, ArtifactVersion ⊑ FlowMember

This avoids introducing “FileFlow” but still fixes the category error.
3) Your rule “Flows do not have AbstractFile directly” is correct, and you should enforce it

If Flows are identities, they shouldn’t directly own byte sequences. Byte sequences belong to snapshots (versions/revisions) or to current-only datasets.

So change:

    hasAbstractFile domain should not be KnopDataset/ if Flow ⊑ KnopDataset.

    Either:

        remove Flow from KnopDataset, or

        narrow hasAbstractFile domain to a “Concrete…” class.

Given what you just said:

    “KnopDatasets that aren't versions, like OpslogDataset DO have AbstractFiles.”

That implies the domain is “things that are concrete publishable datasets,” not “versions specifically.”

So create a class like:

    PublishableDataset/ (or ConcreteKnopDataset/)

        includes: DatasetVersion, PointersDataset, OpslogDataset, inventory dataset if it ever exists

        excludes: Flow

Then:

    hasAbstractFile domain PublishableDataset

This single change will prevent a lot of future drift.
4) Opslog: your current modeling still needs the DatasetSeries split (if you actually mean series)

Given your earlier agreement:

    opslog datasets have dcat:inSeries in their own bytes

    discovery lives in metadata flow

Then you still need:

    OpslogDatasetSeries/ ⊑ dcat:DatasetSeries

    Knop hasOpslogSeries OpslogDatasetSeries

    OpslogDataset dcat:inSeries OpslogDatasetSeries

And OpslogDataset is in the “PublishableDataset” bucket above (it has AbstractFile).
5) Pin: structural vs conceptual containment (you’re right; make it explicit in the ontology)

You need one explicit sentence (normative tone) in the ontology comments for containsSfloResource and for Knop:

    Structural containment: IRI-path containment inside a Knop boundary (and only there) used for inventory/navigation/caching.

    Conceptual containment: domain semantics (the referent does not conceptually contain its knop even if the referent-token path contains _knop/).

Do not try to encode “referent structurally contains knop but not conceptually” as RDF containment triples; keep it as an operational rule + comment, exactly as you said.
Recommendation (most consistent with what you just wrote)

If you want minimum churn and maximum clarity:

    Keep KnopDataset/ as “publishable dataset snapshot” (DCAT dataset is OK here).

    Make Flow/ not a KnopDataset/.

    Replace Version/ with DatasetVersion/ (subclass of KnopDataset/).

    Add ArtifactVersion/ later if needed (for payload artifacts).

    Set hasAbstractFile domain to KnopDataset/ (which no longer includes Flow).

This preserves:

    opslog datasets have files

    flows don’t

    versions do

    payload can bifurcate later without rewriting everything

If you want, I can give you the exact JSON-LD diffs for (1)–(5) without touching anything else.
You said:
I think PayloadFlow and all Flows should only be KnopDatasets, i.e., one or more RDF graphs. 

Help me decide whether to differentiate AbstractDatasets and ConcreteDatasets. I don't love DatasetLineage, because DatasetSeries like OpslogDatasetSeries aren't the same dataset over time, they're just a series of related datasets. 

Maybe instead of KnopDataset, we should do RdfDataset, and define it to be a collection of one or more graphs. 
ChatGPT said:

If you commit to:

    All Flows are RDF datasets (one or more graphs)

    PayloadFlow is therefore always RDF (and “payload as arbitrary bytes” is handled elsewhere, not as PayloadFlow)

…then you should absolutely rename KnopDataset to something like RdfDataset (or KnopRdfDataset) because “dataset” is otherwise going to keep getting dragged back toward “bytes.” Your comment “collection of one or more graphs” is the crispest discriminator you have.

That decision also makes the Abstract-vs-Concrete question much easier, because the axis stops being “dataset vs file” and becomes “identity vs snapshot.”
1) Rename KnopDataset → RdfDataset (recommended)
Why this is the right naming correction

    It aligns with your normative intent: datasets are graphs, not arbitrary artifacts.

    It prevents the DCAT-broadened meaning of “dataset” (binary image etc.) from bleeding into your core SF modeling.

    It forces you to represent non-RDF payload artifacts explicitly (good; it avoids “PayloadFlow is an abstract file over time” confusion).

What to do with DCAT

You can still say RdfDataset ⊑ dcat:Dataset if you want DCAT compatibility, but now it’s clearly “RDF dataset is one kind of DCAT dataset.”
2) Abstract vs Concrete: you should differentiate, but do it with SF-specific semantics

Given your clarified scope, the meaningful axis is:

    Abstract = “dataset identity that organizes snapshots or members”

    Concrete = “dataset snapshot that can have distributions and be validated as content”

This is not about “dataset series vs flow” being the same—both are abstract organizers, but in different ways. You don’t need them to be “the same dataset over time” for them to belong in the same abstract bucket.
Proposed classes (names that don’t smuggle DCAT meaning)
RdfDataset/

“one or more RDF graphs” (your base dataset notion)
RdfDatasetOrganizingResource/ (pick a better label, but this is the role)

Superclass for dataset resources that organize other datasets.

    Subclasses:

        Flow/ (organizes versions of one identity)

        OpslogDatasetSeries/ (organizes weave-interval datasets)

RdfDatasetInstance/ (or RdfDatasetSnapshot/)

Superclass for dataset resources that are “content-bearing snapshots.”

    Subclasses:

        Version/ (if you keep that name) / FlowVersion/

        OpslogDataset/

        PointersDataset/

        (Inventory dataset if it ever becomes a dataset)

Why this works even though Flow and DatasetSeries differ

You’re not claiming they’re the same phenomenon. You’re saying:

    both are not “a snapshot dataset”

    both act as organizers over multiple datasets

That’s the purpose of the abstract class: to control rules like “organizers don’t have files directly.”
3) The decisive rule that makes this worth doing

You said:

    Flows do not have AbstractFile directly.
    But datasets that aren’t versions (OpslogDataset) do have AbstractFiles.

So define:

    hasAbstractFile domain RdfDatasetInstance/

    not RdfDataset/

    not Flow/

This single constraint is the practical reason to introduce Abstract vs Concrete.

If you don’t introduce the abstract/concrete split, you will keep fighting this via ad-hoc comments and exceptions.
4) What to call the “abstract” class (since you dislike DatasetLineage)

You’re right to dislike “DatasetLineage” if it implies “same dataset over time,” because DatasetSeries is not that. But you still need a bucket name for “non-snapshot organizing thing.”

Name options that are honest:

    DatasetOrganizer/

    DatasetGrouping/

    DatasetIndex/ (only if you accept “index” semantics)

    DatasetCollectionResource/ (careful: “collection” sounds like content, not organizing)

    DatasetRegistry/ (a bit too institutional)

    DatasetIdentity/ (wrong for DatasetSeries, like you said)

Best fit, semantically: DatasetOrganizer.

It doesn’t claim lineage; it just claims function.

So:

    RdfDatasetOrganizer/

    RdfDatasetInstance/

5) Should you actually do this now?
Do it now if:

    you want OWL/SHACL rules like “only instances have distributions”

    you want to prevent accidental attachment of files to flows

    you want the ontology to teach contributors the right mental model

Defer if:

    you aren’t going to validate/enforce anything yet

    you’re still changing these primitives weekly

Given your repeated back-and-forth on exactly this point, you’re at the stage where codifying the split will reduce churn, not increase it.
Recommendation

    Rename KnopDataset → RdfDataset and define it as “one or more RDF graphs.”

    Add:

        RdfDatasetOrganizer (subclass of RdfDataset)

        RdfDatasetInstance (subclass of RdfDataset)

    Make:

        Flow ⊑ RdfDatasetOrganizer

        OpslogDatasetSeries ⊑ RdfDatasetOrganizer (and dcat:DatasetSeries)

        Version, OpslogDataset, PointersDataset ⊑ RdfDatasetInstance

    Change:

        hasAbstractFile domain RdfDatasetInstance

If you want, I can give you the exact JSON-LD patch for these changes (minimal, no reformatting).
You said:
Is there any prior art for RdfDataset? Like well-known ontology that discriminates RDF vs non-RDF datasets?

For "RdfDatasetOrganizingResource", maybe: RdfDatasetContainer?

I'm not sure we need RdfDatasetInstance. It could just be RdfDataset. So we don't say RdfDatasetContainer is a RdfDataset. It might be a dcat:DatasetSeries, but RdfDatasetContainer doesn't directly have AbstractFiles of its own.

Which makes me think, maybe we subclass AbstractFile into RdfDatasetDistribution.

Version/ could be subclassed to RdfDatasetVersion, and we leave the Version/ superclass only for non-rdf artifacts, i.e. payload flows.
ChatGPT said:

I’ll answer this in three layers: prior art, naming/shape choices, and a concrete recommendation that minimizes class explosion while keeping you honest.
1. Prior art: is “RdfDataset” a thing?
Short answer

There is no single canonical ontology class that cleanly means “this dataset is RDF” and nothing else. But there is strong, convergent prior art that treats “RDF dataset” as a first-class kind of dataset via format, profile, or content constraints, not via DCAT core classes.
What exists (and what it implies)
a) RDF 1.1 Dataset (W3C RDF)

    “RDF Dataset” is formally defined as a set of RDF graphs (default + named).

    This is a data model concept, not an ontology class.

    Many ontologies implicitly assume it, but don’t reify it as a class.

Implication:
Your instinct to define RdfDataset as “one or more RDF graphs” is normatively correct, even if it’s not common as a named OWL class.
b) DCAT practice

DCAT intentionally does not discriminate RDF vs non-RDF at the dataset class level.

Instead, RDF-ness is expressed via:

    dcterms:format (e.g., text/turtle)

    dcat:mediaType

    dcat:profile (e.g., “this dataset conforms to RDF 1.1”)

Implication:
DCAT punts on the distinction; your ontology is justified in introducing it if it matters structurally (and it clearly does).
c) VOID (void:Dataset)

    VOID datasets are assumed to be RDF.

    But void:Dataset is not a subclass of dcat:Dataset.

    VOID is about describing RDF datasets as linked data sets, not about general publishing.

Implication:
VOID is close in spirit but not usable as your base class.
d) SHACL / SPARQL / RDF tooling ecosystems

These ecosystems routinely assume:

    “dataset” == RDF graphs

    “distribution” == file or endpoint

But again, this is implicit, not ontologized.
Conclusion on prior art

You are not inventing a weird thing by introducing RdfDataset.
You are making explicit a distinction that DCAT intentionally leaves implicit.

That’s legitimate and necessary given your design constraints.
2. Your proposed restructuring: what works and what doesn’t
A. RdfDataset as the base dataset class

This is a good move if you commit to:

    “All Flows are RDF datasets (graphs), not arbitrary byte artifacts.”

I would define it explicitly as:

    “A dataset whose content is a collection of one or more RDF graphs.”

Then:

    Flow ⊑ RdfDataset

    Version ⊑ RdfDataset (or renamed, see below)

    OpslogDataset ⊑ RdfDataset

    PointersDataset ⊑ RdfDataset

You may also say:

    RdfDataset ⊑ dcat:Dataset
    …but you don’t have to if you want to keep DCAT as a view layer.

B. RdfDatasetContainer vs RdfDatasetOrganizer

Your instinct here is sound.

If a thing:

    groups datasets

    has members

    but does not itself have AbstractFiles

then it should not be an RdfDataset.

So:

    RdfDatasetContainer (or RdfDatasetGroupingResource)
    is not a subclass of RdfDataset.

This avoids the awkwardness of:

    “a dataset that has no data”

What should be a RdfDatasetContainer?

    Flow (organizes versions)

    OpslogDatasetSeries (organizes opslog datasets)

They may be:

    dcat:DatasetSeries

    or just SF-specific containers

But crucially: they do not directly own bytes.
C. You probably do not need RdfDatasetInstance

You’re right to be suspicious of that extra layer.

If you adopt:

    RdfDataset = “has RDF content”

    RdfDatasetContainer = “organizes datasets, not itself a dataset”

Then you can simply say:

    hasAbstractFile domain RdfDataset

    RdfDatasetContainer has no hasAbstractFile

That’s enough.

You only need RdfDatasetInstance if:

    you later want to distinguish “snapshots” from “current-only datasets” within RDF datasets.

Right now, that seems premature.
3. AbstractFile vs Distribution: your new insight is the right direction
Subclassing AbstractFile → RdfDatasetDistribution

This is exactly the right move.

Pattern:

    AbstractFile
    = exact byte sequence, platform concept, no DCAT semantics

    RdfDatasetDistribution ⊑ AbstractFile ⊓ dcat:Distribution
    = “this abstract file is intended as a representation of an RDF dataset”

Then:

    RdfDataset hasAbstractFile RdfDatasetDistribution

    non-RDF payload artifacts:

        use AbstractFile

        not RdfDatasetDistribution

This cleanly resolves:

    your discomfort with calling payloads “distributions”

    DCAT leakage via downloadURL

    future DCAT export needs

This also resolves your PayloadFlow confusion

If PayloadFlow is RDF-only:

    its versions are RdfDataset

    their files are RdfDatasetDistribution

If you later want non-RDF payloads:

    they become PayloadArtifacts

    with AbstractFile, not RdfDatasetDistribution

No need to contort Flows.
4. Versions: your proposed split is sensible

    “Version/ could be subclassed to RdfDatasetVersion, and we leave the Version/ superclass only for non-rdf artifacts”

Yes — but only if you expect non-RDF versioned artifacts to actually exist.

If you do, then:

    Version = generic temporal snapshot (no dataset assumption)

    RdfDatasetVersion ⊑ Version ⊓ RdfDataset

    (future) ArtifactVersion ⊑ Version

If you don’t yet, you can skip the generic Version and just rename:

    Version → RdfDatasetVersion

That would be simpler and more honest for now.
5. Concrete recommendation (minimal, coherent, future-proof)

Given everything you’ve said, I recommend:

    Rename KnopDataset → RdfDataset

    Introduce RdfDatasetContainer

        NOT a subclass of RdfDataset

        used for:

            Flow

            OpslogDatasetSeries

    Make Flow ⊑ RdfDatasetContainer

    Keep Version ⊑ RdfDataset

        optionally rename to RdfDatasetVersion

    Split files cleanly

        AbstractFile (platform artifact)

        RdfDatasetDistribution ⊑ AbstractFile ⊓ dcat:Distribution

    Set

        hasAbstractFile domain RdfDataset

        not Flow, not containers

This gives you:

    no fake datasets

    no DCAT leakage

    no premature over-typing

    a clean story for RDF vs non-RDF payloads

If you want, next step I can:

    sketch the class hierarchy as a diagram, or

    give you the exact JSON-LD diff to apply.

You said:
Is a Flow (even conceptualized as an RdfDatasetFlow) really a dataset? Maybe, but I don't think it's an Dataset. It's a very specific thing... a series of RdfDatasetVersions in a single lineage.

We could have the non-RDF equivalent of a Flow is "EvolvableAbstractArtifact". Or we could make Flow more abstract, and have RdfFlow and NonRdfFlow. 

We''ll support arbitrary file/bytes artifacts, and if we re-use the Flow/Version conceptualization for that, I think it makes sense. 

So PayloadRdfDatasetFlow and PayloadArtifact flow subclass Flow? and maybe we don't need different subclasses of Version/


So RdfDatasetFlow ⊑ 
ChatGPT said:

If you want Flow/Version to apply to both RDF datasets and arbitrary byte artifacts, then Flow cannot be a Dataset (in your ontology) without either (a) lying, or (b) making “Dataset” so broad it becomes meaningless again.

That is the critical decision point.
Why “Flow is a dataset” is the wrong commitment for you

A Flow is not “content”; it is an evolution container / lineage identity. It organizes versions. In DCAT terms, that role is closer to dcat:DatasetSeries than to dcat:Dataset (even though DCAT makes DatasetSeries a Dataset, you don’t have to mirror that semantic flattening). You’ve already said: Flow ≠ DatasetSeries. So you’re already diverging from DCAT’s taxonomy.

So: treat Flow as its own SF construct, and then attach RDF-ness or artifact-ness via subclasses.
Recommended core model (minimal classes, no fake “dataset” typing)
1) Make Flow a KnopComponent, not a dataset

    Flow/ ⊑ KnopComponent/ ⊓ SfloContainerResource/

Definition: “A lineage identity that organizes Versions.”
2) Make Version the “content-bearing snapshot,” but don’t force it to be RDF

    Version/ ⊑ KnopComponent/ ⊓ SfloContainerResource/

Definition: “A concrete snapshot/member of a Flow.”

Then relate them:

    Flow hasVersion Version

    Flow hasCurrentVersion Version

    Version previousVersion Version

    Version versionSequence xsd:integer (only if you actually keep numeric sequences; you’ve said payload artifacts may use weave labels instead)

3) Add “kind” subclasses for Flow and Version, not separate Version base classes

This matches your instinct: “maybe we don’t need different subclasses of Version.”
RDF side

    RdfDataset/ = “one or more RDF graphs” (your data model class)

    RdfFlow/ ⊑ Flow/

    RdfVersion/ ⊑ Version/ ⊓ RdfDataset/

Now RDF-ness only appears where it is true: in the snapshot content.
Artifact side

    AbstractArtifact/ = “a digital artifact identity”

    ArtifactFlow/ ⊑ Flow/

    ArtifactVersion/ ⊑ Version/ ⊓ AbstractArtifact/

And for artifacts:

    ArtifactVersion hasAbstractFile AbstractFile (bytes live here)

    For RDF versions:

        either still use hasAbstractFile for RDF serializations (if you want)

        or treat RDF “content” as graphs and distributions as optional export

This lets you reuse Flow/Version without pretending Flow itself is “data.”
How Payload fits cleanly

    PayloadRdfFlow/ ⊑ RdfFlow/

    PayloadArtifactFlow/ ⊑ ArtifactFlow/

And then:

    PayloadRdfVersion ⊑ RdfVersion

    PayloadArtifactVersion ⊑ ArtifactVersion

If you don’t want more classes: skip the specialized Version subclasses and just rely on typing the Version instances as RdfDataset vs AbstractArtifact.
Where RdfDatasetFlow ⊑ … should land (your unfinished line)

If you adopt the above:

    RdfDatasetFlow ⊑ Flow

That’s it. Not ⊑ RdfDataset.

You can optionally also say:

    RdfDatasetFlow “organizes” RdfDatasetVersion via a range restriction (SHACL later; OWL restrictions if you really want)

So the intended statement is:

    RdfDatasetFlow ⊑ Flow

    and hasVersion for RdfDatasetFlow has range RdfDatasetVersion

You must also fix two existing property choices if you go this way
1) hasAbstractFile domain

Right now it’s on KnopDataset/RdfDataset in the earlier direction. With Flow not being a dataset, you want:

    hasAbstractFile domain Version (or “ConcreteResource”)

    because versions are the things that have concrete byte representations.

2) versionSequence vs weave-label

You’ve already said opslog uses weave labels like YYYY-MM-DD_HHMM_SS. That implies versionSequence should not be universal on Version/.

So either:

    move versionSequence to a subclass like SequencedVersion

    or replace it with a more general versionLabel datatype property and keep versionSequence optional.

My recommendation

Given your stated goals (re-use Flow/Version for both RDF and non-RDF, keep dataset meaning crisp):

    Flow is not a dataset. It is a lineage identity/container.

    Versions carry the content typing (RDF dataset vs artifact).

    Keep RdfDataset as a content class; do not force it onto Flow.

    Make hasAbstractFile apply to concrete members (versions / current-only snapshots), not flows.

If you want, I can translate this into an exact class/property diff against your current ontology (including where PayloadFlow, ReferenceFlow, BacklinksFlow land).
You said:
For hasLocatedFile, probably not 
      "rdfs:subPropertyOf": "dcat:downloadURL",

is there a good property I can use? 
ChatGPT said:

Yes. If you want a “good property I can use” without forcing DCAT typing, you have three realistic choices, depending on what you mean by “located file”.

You should first be explicit about the semantics you want:

    Is it direct download (bytes at the IRI)?

    Or some access endpoint (landing page, service endpoint, may require negotiation/auth)?

    Or generic “this resource can be retrieved from here” (not necessarily DCAT distribution semantics)?

Given your earlier statement (“IRI intended to retrieve its bytes”), you mean direct download. DCAT’s downloadURL matches that intent, but its domain is dcat:Distribution, so you correctly don’t want to reuse it as a superproperty.
Option 1 (recommended): keep sflo:hasLocatedFile as a standalone property

This is the cleanest and most honest:

    sflo:hasLocatedFile (domain rdfs:Resource, range sflo:LocatedFile)

    no subPropertyOf at all

Then create a second property only for DCAT exports:

    sflo:downloadURL (domain sflo:DcatDistribution or dcat:Distribution)

    rdfs:subPropertyOf dcat:downloadURL

This avoids semantic leakage while preserving an easy mapping when you do want DCAT.

Why this is “good”: it matches your platform semantics and doesn’t lie. It also avoids inheriting DCAT’s domain constraints.
Option 2: use an existing, broadly-scoped “is accessible at” predicate

If you strongly prefer reuse, the closest general-purpose prior art is:

    schema:contentUrl (Schema.org)

        Intended for “URL of the actual content media object”

        Domain in Schema is schema:MediaObject, but in practice it’s often used loosely.

Caveat: Schema.org is not an RDF-purist vocabulary and its semantics are web-content oriented. If you use it, you’re implicitly saying “this is content for a media object,” which may or may not fit your “LocatedFile” construct.

I would only use this if you already buy into Schema.org elsewhere (you do, a little: ResourcePageFile ⊑ schema:WebPage).
Option 3: use dcat:accessURL instead of downloadURL (not recommended for you)

dcat:accessURL is broader (“gives access”), but it has the same domain problem: dcat:Distribution. So it doesn’t solve your typing issue; it just loosens “direct download” semantics. It still leaks DCAT distribution typing.
What I think you should do

Do Option 1:

    Keep sflo:hasLocatedFile as your platform-neutral property with no DCAT superproperty.

    Add an explicit DCAT-facing property for the cases you want to say “this thing is a distribution”:

        either use dcat:downloadURL directly on sflo:DcatDistribution

        or define sflo:downloadURL as a subproperty with restricted domain.

This gives you:

    zero accidental inference of dcat:Distribution

    a clean DCAT export path

    consistent semantics for resource pages (since you also want referents to have hasResourcePageFile)

Concrete snippet (minimal changes)

{
  "@id": "hasLocatedFile/",
  "@type": "owl:ObjectProperty",
  "rdfs:domain": "rdfs:Resource",
  "rdfs:range": "LocatedFile/",
  "rdfs:label": "has located file",
  "rdfs:comment": "Links a resource to a LocatedFile whose IRI retrieves its bytes. Platform-level retrieval link; does not imply DCAT Distribution semantics."
},
{
  "@id": "DcatDistribution/",
  "@type": "rdfs:Class",
  "rdfs:subClassOf": ["AbstractFile/", "dcat:Distribution"],
  "rdfs:label": "DCAT distribution",
  "rdfs:comment": "An AbstractFile that is asserted to be a DCAT Distribution (a representation of a dataset)."
}

Then, in data (not ontology), only when appropriate:

    <some-abstract-file> a sflo:DcatDistribution; dcat:downloadURL <some-located-file>.

If you insist on also having an SF property that maps to DCAT, define:

    sflo:downloadURL subPropertyOf dcat:downloadURL

    domain sflo:DcatDistribution

    range sflo:LocatedFile

If you tell me whether you need “direct bytes” only, or also “landing page/service endpoints,” I can suggest the complementary properties (e.g., hasAccessPage, hasServiceEndpoint) in a way that stays DCAT-compatible without leaking types.
You said:
Right now, KnopComponent specifies "only things contained directly under a Knop", so Version/ wouldn't qualify. Neither would AbstractFile or LocatedFile. 

Should I really have a PayloadFlow superclass, subclasses RdfPayloadFlow and NonRdfPayloadFlow? Either way, I think we want two separate properties, hasRdfPayloadFlow and hasNonRdfPayloadFlow? 
ChatGPT said:
1) Your current KnopComponent definition is too narrow for the model you’re building

You’ve noticed the correct consequence: if KnopComponent means only direct children of the Knop, then:

    Version/ (contained under a Flow/) is not a KnopComponent.

    AbstractFile/ (contained under a dataset/version) is not a KnopComponent.

    LocatedFile/ (endpoint, not container) is not a KnopComponent.

That’s not a bug; it’s exactly what “direct child” means. The problem is: you also want to treat these as “SF structural resources,” and right now you only have two buckets: KnopComponent and “everything else.”
Recommendation

Keep KnopComponent as “direct child,” but add a broader class:

    SfloStructuralResource/ (or reuse SfloContainerResource/ + LocatedFile/)

        includes: Knop, KnopComponent, Flow, Version, AbstractFile, ResourcePageFile, etc.

        basically: “things that participate in SF addressable structure within _knop/ boundaries”

Then:

    containsSfloResource should range over SfloStructuralResource (or SfloContainerResource ⊔ LocatedFile), not rdfs:Resource, if you want it to be a meaningful structural predicate.

    hasKnopComponent remains the “direct child” relation.

This avoids trying to force Versions into the KnopComponent box.
2) Do you need PayloadFlow + RdfPayloadFlow + NonRdfPayloadFlow?

Only if payload can be either RDF datasets or non-RDF artifacts and you intend to treat both as flows with versions.

But you’ve recently said: “PayloadFlow and all flows should only be KnopDatasets (RDF graphs).” If that is still true, then NonRdfPayloadFlow is a contradiction and should not exist.

So you have two consistent designs:
Design A (RDF-only flows; simplest, consistent with your stated intent)

    PayloadFlow is always an RDF dataset flow.

    No NonRdfPayloadFlow.

    Non-RDF payload artifacts are modeled elsewhere (not as flows).

If you choose this, you do not need two properties.
Design B (two payload modalities; flows apply to both)

    PayloadFlow is an abstract superclass.

    Subclasses:

        RdfPayloadFlow

        ArtifactPayloadFlow (don’t call it “NonRdfPayloadFlow”; “non-RDF” is a negative class and will be ambiguous)

    Both are Flow but differ in what their versions are.

If you choose this, then yes, having two properties can improve ergonomics, but it’s not strictly necessary.
3) Should you have two properties: hasRdfPayloadFlow and hasNonRdfPayloadFlow?

You probably don’t need both, and if you add them you risk “property explosion” that you already hate (you keep doing this in other areas and then regretting it).

Preferred pattern:

    Keep one generic property: hasPayloadFlow

        domain Knop

        range PayloadFlow

Then define subclasses and optional subproperties only if you need fast selection without reasoning.

If you do want the subproperties for bootstrapping convenience:

    hasRdfPayloadFlow subPropertyOf hasPayloadFlow

        range RdfPayloadFlow

    hasArtifactPayloadFlow subPropertyOf hasPayloadFlow

        range ArtifactPayloadFlow

This is better than “non-RDF” and keeps semantics positive.

But note: you already have a “pointers dataset for bootstrapping.” If that dataset can be parsed, then you don’t need two properties just for performance; you can just query types.
4) The real decision you need to lock right now

You must pick one of these and stick to it:

    Flows are RDF-only.
    Then there is no non-RDF payload flow. Non-RDF payload lives as AbstractFile/LocatedFile artifacts directly under the knop (or in a separate artifact subsystem).

    Flows are general evolution containers for both RDF datasets and artifacts.
    Then Flow is not an RDF dataset itself; only versions carry RDF typing vs artifact typing.

Right now your messages oscillate between (1) and (2). You can’t design a coherent class tree until you stop oscillating.

Given your last explicit statement (“all flows are RDF graphs”), the consistent answer is:

    No NonRdfPayloadFlow.

    Keep PayloadFlow as the payload flow class (RDF).

    If you later want artifact payload evolution, introduce a different mechanism (or relax the “flows are RDF-only” constraint).

Recommended concrete steps (aligned with RDF-only flows)

    Keep KnopComponent = direct child containers only (good).

    Make:

        Flow ⊑ KnopComponent

        Version ⊑ SfloContainerResource but not KnopComponent

        AbstractFile ⊑ SfloContainerResource but not KnopComponent

        LocatedFile stays outside container

    Add SfloStructuralResource (or tighten containsSfloResource range) to capture “things in the structure.”

    Keep a single property:

        hasPayloadFlow (range PayloadFlow)

        PayloadFlow ⊑ Flow

No hasRdfPayloadFlow/hasNonRdfPayloadFlow unless you later reopen the “flows for artifacts” decision.

If you want, paste your current definitions of Flow/, Version/, and your intended IRI layout for payload artifacts, and I’ll give you the smallest coherent class/property adjustments.
You said:
I already have SfloContainerResource. That excludes LocatedFile, but I think that's fine, since those are the only things that aren't containers. 

"PayloadFlow and all flows should only be KnopDatasets (RDF graphs)" is superseded. Now we have two kinds of PayloadFlows. Flows are evolution containers.

I like NonRdfPayloadFlow because RDF Datasets are still artifacts. We've run into this before, and the only honest name is NonRdfPayloadFlow. 

We already have unique "has" properties for each type of Flow (hasKnopMetadataFlow, hasReferenceFlow, hasBacklinksFlow) and separate properties for non-flow things (hasOpslogDatasetSeries, hasOpslogDataset, hasPointersDataset).  I'd imagined this is useful for simplified constraint checking... you don't have to look up (or even have existing) the type of the target flow. And the SHACL becomes simple: you can only have one of each of those properties.

ChatGPT said:

Under your updated commitments, your “unique hasX properties” approach is internally consistent and does buy you simpler SHACL and simpler runtime constraint checks. The price is vocabulary growth, but you’ve already accepted that trade in other places.

Here’s what I think you should do, and the specific issues to resolve.
1) With “Flows are evolution containers”, Flow should not be a Dataset

If Flow is now an evolution container that can organize either RDF snapshots or non-RDF artifact snapshots, it should not be modeled as dcat:Dataset (or even as RdfDataset) because that forces “the container is also the content.”

So make:

    Flow ⊑ SfloContainerResource ⊓ KnopComponent (direct child of a Knop)

    Version ⊑ SfloContainerResource (not a KnopComponent; it is contained by a Flow)

Then treat “RDF datasetness” and “artifactness” as properties/classes of the versions, not the flow itself.

That aligns with your rule that only _knop/ direct children are KnopComponent and versions aren’t.
2) Your SfloContainerResource / LocatedFile split is fine, but fix what “contains” can point to

If the only non-containers are LocatedFile, then:

    containsSfloResource should have range SfloContainerResource, not rdfs:Resource, if you want it to mean anything structural.

Use separate predicates for “contains file endpoints”:

    hasLocatedFile (domain rdfs:Resource or SfloContainerResource, range LocatedFile)

This matches your intent: containment is only for containers; file endpoints are linked differently.
3) Payload flows: yes, you can have PayloadFlow + RdfPayloadFlow + NonRdfPayloadFlow

Given your stated rationale, “NonRdfPayloadFlow” is acceptable. It’s a negative name, but it is unambiguous and it preserves your previously discovered truth: RDF datasets are still artifacts; “artifact” alone doesn’t discriminate.

So:

    PayloadFlow ⊑ Flow

    RdfPayloadFlow ⊑ PayloadFlow

    NonRdfPayloadFlow ⊑ PayloadFlow

Now decide what their versions are.
You have two options for versions
Option A (recommended): don’t subclass Version, type the version content instead

Keep a single Version/ class and distinguish content by typing the version instance as:

    RdfDataset (one or more graphs)

    AbstractArtifact (non-RDF artifact identity)

Then use range constraints on the relevant “has version” property (below).

This avoids version-class explosion and lets you keep your “simple SHACL: max 1 per property” pattern.
Option B: subclass versions (RdfDatasetVersion, ArtifactVersion)

This is also coherent, but it tends to cascade (you’ll later want OpslogVersion etc.). I’d avoid it unless you need it for reasoning.
4) Your unique properties approach: keep it, but make them subproperties

Your pattern is:

    You don’t want to look up rdf:type of the target.

    You want SHACL to be “one property, maxCount 1”.

That implies specialized properties are the right fit.

But you should still keep a general backbone so the ontology stays navigable:

    hasFlow (Knop → Flow)

    hasPayloadFlow (Knop → PayloadFlow)

    hasRdfPayloadFlow (Knop → RdfPayloadFlow)

    hasNonRdfPayloadFlow (Knop → NonRdfPayloadFlow)

And define:

    hasPayloadFlow rdfs:subPropertyOf hasFlow

    hasRdfPayloadFlow rdfs:subPropertyOf hasPayloadFlow

    hasNonRdfPayloadFlow rdfs:subPropertyOf hasPayloadFlow

This preserves your ergonomic, constraint-friendly leaf properties while keeping the hierarchy sane.
SHACL becomes exactly what you want

    sh:maxCount 1 on hasRdfPayloadFlow

    sh:maxCount 1 on hasNonRdfPayloadFlow

    plus, if desired, a SHACL constraint that at most one of the two is present.

(That latter is a standard sh:or/sh:xone pattern.)
5) Version linkage needs to be split the same way (or you’ll reintroduce ambiguity)

If flows organize versions of different kinds, you need either:
Option A: separate properties (best match to your current design philosophy)

    hasRdfPayloadVersion (RdfPayloadFlow → Version with rdf:type RdfDataset)

    hasNonRdfPayloadVersion (NonRdfPayloadFlow → Version with rdf:type AbstractArtifact)

    both subproperties of a generic hasVersion

This matches your “don’t look up type” philosophy at the cost of extra properties.
Option B: keep one hasVersion, enforce version typing via SHACL

    RdfPayloadFlowShape says:

        sh:path hasVersion

        sh:class RdfDataset

    NonRdfPayloadFlowShape says:

        sh:path hasVersion

        sh:class AbstractArtifact

This keeps the ontology smaller but requires type presence.

Given your stated goal (“don’t even need to have existing the type of the target flow”), you probably prefer Option A for payload, but you don’t necessarily need it for all flows. Payload is the only place where the member-type bifurcates.
6) Small correction: “RdfDatasetContainer” is misleading for Flow

If you call Flow a “container,” you’re going to collide with your existing “container = slash IRI structural” meaning and with SfloContainerResource. Flow is already a container resource. Naming it “RdfDatasetContainer” suggests it contains datasets (plural) in the same way a folder contains files.

Better naming for the abstract grouping role:

    EvolutionContainer

    Lineage

    EvolvableResource

    VersionedResource

Given your new definition (“Flows are evolution containers”), the class name Flow is already doing that job. I would not add “container” to the name; it will create terminological noise.
Concrete recommendations (minimal diffs, consistent with your pattern)

    Make Flow not a dataset

        remove Flow ⊑ RdfDataset / KnopDataset inheritance

        make Flow ⊑ KnopComponent ⊑ SfloContainerResource

    Keep Version ⊑ SfloContainerResource and not KnopComponent

    Add:

        PayloadFlow ⊑ Flow

        RdfPayloadFlow ⊑ PayloadFlow

        NonRdfPayloadFlow ⊑ PayloadFlow

    Add properties:

        hasPayloadFlow ⊑ hasFlow

        hasRdfPayloadFlow ⊑ hasPayloadFlow (range RdfPayloadFlow)

        hasNonRdfPayloadFlow ⊑ hasPayloadFlow (range NonRdfPayloadFlow)

        SHACL: maxCount 1 for each; optionally xone between them.

    For versions, pick one:

        A (property split): hasRdfPayloadVersion, hasNonRdfPayloadVersion

        B (single hasVersion): keep hasVersion, but SHACL enforces member typing per flow subclass.

If you tell me which version-linking option you prefer (A or B), I can give you the exact JSON-LD snippets for the ontology and the corresponding SHACL shapes that implement “one property, maxCount 1, no type lookups needed.”
You said:
But do we need PayloadFlow? It's never instantiated directly, doesn't really needits own property. I think we can skip it. I guess it does provide a convenient term for discussion/documentation. 

ChatGPT said:

You can skip PayloadFlow and nothing breaks as long as you’re honest about what you lose.
When skipping PayloadFlow is fine

Skip it if all of these are true:

    You’re satisfied with discussing “payload flows” purely as a role played by RdfPayloadFlow / NonRdfPayloadFlow (documentation can still say “payload flow” informally).

    You don’t need a single range for a generic hasPayloadFlow property.

    You don’t need constraints like “exactly one payload flow of either kind exists” expressed at the class level via one predicate.

Given your current pattern (you already like leaf properties like hasReferenceFlow, hasBacklinksFlow), you can absolutely live without the intermediate class.
What you lose by skipping it (and how to replace it cleanly)
Loss A: One unified property

Without PayloadFlow you’ll likely have:

    hasRdfPayloadFlow

    hasNonRdfPayloadFlow

That’s fine, but you can’t say “all payload flows are flows” via one range; you just say both ranges are Flow.
Loss B: One place to attach shared semantics

If later you decide both payload flows share common metadata (e.g., “payload source policy,” “payload licensing defaults,” “payload visibility rules”), you’d either:

    duplicate those comments/constraints on both classes, or

    reintroduce PayloadFlow later.

Reintroducing later is not hard, but you may prefer not to churn.
Loss C: Expressing “exactly one of the two” is slightly more verbose

Still doable in SHACL, just slightly uglier.
SHACL if you skip PayloadFlow

You can still express “at most one of each” plus “not both”:

    maxCount 1 on each property

    plus an XOR-style constraint:

    Either:

        sh:xone with two shapes (“hasRdfPayloadFlow present” vs “hasNonRdfPayloadFlow present”)

    Or:

        sh:not + sh:and to forbid both present simultaneously

So skipping PayloadFlow does not force type lookups.
My recommendation

Keep PayloadFlow only if you want a stable umbrella term for:

    docs

    shared constraints

    shared future properties

Otherwise, skip it and rely on:

    RdfPayloadFlow ⊑ Flow

    NonRdfPayloadFlow ⊑ Flow

    two leaf properties on Knop

Given your “unique has-properties per flow type” philosophy, skipping PayloadFlow is consistent and keeps the ontology smaller.

If you want the best of both worlds with almost no cost:

    keep the class PayloadFlow (documentation umbrella)

    but don’t create hasPayloadFlow

    and don’t require it to ever be instantiated separately

That gives you a named concept without adding a new predicate or runtime rule.
You said:
Could we express "either one, but not both" using the properties (hasRdfPayloadFlow and hasNonRdfPayloadFlow) ?
ChatGPT said:

Yes. You can express “either one, but not both” purely in terms of the two properties (no reliance on the target node’s rdf:type) using SHACL. Two common patterns:
Pattern 1: sh:xone (exactly one of the two property-presence shapes)

This enforces exactly one of the alternatives is satisfied.

    Each alternative shape requires minCount 1 on one property and maxCount 0 on the other.

{
  "@id": "KnopPayloadXoneShape/",
  "@type": "sh:NodeShape",
  "sh:targetClass": { "@id": "Knop/" },
  "sh:xone": [
    {
      "@type": "sh:NodeShape",
      "sh:property": [
        { "sh:path": { "@id": "hasRdfPayloadFlow/" }, "sh:minCount": 1, "sh:maxCount": 1 },
        { "sh:path": { "@id": "hasNonRdfPayloadFlow/" }, "sh:maxCount": 0 }
      ]
    },
    {
      "@type": "sh:NodeShape",
      "sh:property": [
        { "sh:path": { "@id": "hasNonRdfPayloadFlow/" }, "sh:minCount": 1, "sh:maxCount": 1 },
        { "sh:path": { "@id": "hasRdfPayloadFlow/" }, "sh:maxCount": 0 }
      ]
    }
  ]
}

Semantics: exactly one payload flow must exist, and it must be of one kind.

If you want “zero or one, but not both” (i.e., optional payload), use sh:or instead of sh:xone, with each branch forbidding the other, and don’t require minCount.
Pattern 2: “at most one each” + forbid both present (XOR without requiring one)

This expresses 0 or 1 of each, and not both at once.

{
  "@id": "KnopPayloadXorShape/",
  "@type": "sh:NodeShape",
  "sh:targetClass": { "@id": "Knop/" },

  "sh:property": [
    { "sh:path": { "@id": "hasRdfPayloadFlow/" }, "sh:maxCount": 1 },
    { "sh:path": { "@id": "hasNonRdfPayloadFlow/" }, "sh:maxCount": 1 }
  ],

  "sh:not": {
    "@type": "sh:NodeShape",
    "sh:and": [
      {
        "@type": "sh:NodeShape",
        "sh:property": [
          { "sh:path": { "@id": "hasRdfPayloadFlow/" }, "sh:minCount": 1 }
        ]
      },
      {
        "@type": "sh:NodeShape",
        "sh:property": [
          { "sh:path": { "@id": "hasNonRdfPayloadFlow/" }, "sh:minCount": 1 }
        ]
      }
    ]
  }
}

Semantics: you may have none, or one of either kind, but never both.
Which should you use?

    Use Pattern 1 (sh:xone) if every Knop must have exactly one payload flow.

    Use Pattern 2 if payload is optional but mutually exclusive.

Both satisfy your goal: constraint checking can be done without inspecting the target flow’s type.
You said:
I went with 2. 

OKAY, can you please review the ontology and tell me what I've forgotten to implement and point out issues and inconsistencies?

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:subPropertyOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": { "@id": "https://djradon.github.io/ns/djradon/" },
      "owl:versionInfo": "0.0.1",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/_payload/_default/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
    },
    {
      "@id": "SfloContainerResource/",
      "@type": "rdfs:Class",
      "rdfs:label": "SF container resource",
      "rdfs:comment": "A resource that participates in Semantic Flow structural containment (sflo:containsSfloResource). Typically represented by a slash-terminated IRI and associated with a resource page file (often index.html). This class is a platform-scoped structural notion and does not imply real-world, mereological, or metaphysical containment of the denoted referent."
    },
    {
      "@id": "containsSfloResource/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "SfloContainerResource/",
      "rdfs:range": "SfloContainerResource",
      "rdfs:label": "contains SF resource",
      "rdfs:comment": "Intra-Knop structural (AND conceptual?) containment relation. Intended for containment within Knop structure (e.g., within <.../_knop/> and its contained components). This relation is only asserted or derived inside Knop boundaries."
    },
    {
      "@id": "LocatedFile/",
      "@type": "rdfs:Class",
      "rdfs:label": "Located file",
      "rdfs:comment": "A resource whose IRI is intended to retrieve its bytes. LocatedFiles do not act as SF containers."
    },
    {
      "@id": "ResourcePageFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["LocatedFile/", "schema:WebPage"],
      "rdfs:label": "Resource page file",
      "rdfs:comment": "A LocatedFile intended to be a human-readable resource page (typically index.html) for a resource."
    },
    {
      "@id": "hasLocatedFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "LocatedFile/",
      "rdfs:label": "has located file",
      "rdfs:comment": "Links a resource to a LocatedFile whose IRI retrieves its bytes (download endpoint). Used for distribution-like resources (e.g., AbstractFile) and also for resource page files."
    },
    {
      "@id": "hasResourcePageFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasLocatedFile/",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "ResourcePageFile/",
      "rdfs:label": "has resource page file",
      "rdfs:comment": "Associates any resource (including referents) with a human-readable resource page file (typically index.html)."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop",
      "rdfs:comment": "A structural-conceptual unit that supports one referent (one-thing-per-knop). Knop instance IRIs are the Knop-structure IRIs (typically <.../_knop/>). A Knop is not itself a dataset; it is composed of datasets and other components."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop component",
      "rdfs:comment": "An SF container resource contained by a Knop that provides specific functionality. KnopComponents are typically addressed within the Knop structure namespace rooted at <.../_knop/>."
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
      "@id": "supportsReferent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "rdfs:Resource",
      "owl:inverseOf": "hasKnop/",
      "rdfs:label": "supports referent",
      "rdfs:comment": "Links a Knop to the resource it supports (its referent). This is an identification/support relation, not a containment relation."
    },
    {
      "@id": "hasKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Knop/",
      "owl:inverseOf": "supportsReferent/",
      "rdfs:label": "has knop",
      "rdfs:comment": "Associates a resource (referent) with a Knop that supports it."
    },
    {
      "@id": "RdfDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["KnopComponent/", "dcat:Dataset"],
      "rdfs:label": "RdfDataset",
      "rdfs:comment": "One or more RDF graphs,. Datasets are contained under Knops as KnopComponents. Some datasets are current-only; some are versioned via Flows."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["KnopComponent/"],
      "rdfs:label": "Flow",
      "rdfs:comment": "A KnopComponent that organizes Versions and provides a lineage identity "
    },
    {
      "@id": "RdfFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["Flow/"],
      "rdfs:label": "RdfFlow",
      "rdfs:comment": "A Flow that organizes versions of RDF datasets."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Metadata RdfFlow",
      "rdfs:comment": "Flow containing metadata about the Knop-as-Knop (including inventory snapshots computed at weave time). Typically addressed at <.../_knop/_meta/>."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Reference flow",
      "rdfs:comment": "Flow containing reference data about the Knop's referent. Typically addressed under <.../_knop/_ref/> (or another intra-knop convention)."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Payload flow",
      "rdfs:comment": "Flow containing versions of the referent digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "RdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PayloadFlow/",
      "rdfs:label": "RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent RdfDataset contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "NonRdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PayloadFlow/",
      "rdfs:label": "Non-RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent non-RDF digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
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
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has metadata flow",
      "rdfs:comment": "Links a Knop to its metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "ReferenceFlow/",
      "rdfs:label": "has reference flow",
      "rdfs:comment": "Links a Knop to its reference flow."
    },
    {
      "@id": "hasRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "RdfPayloadFlow/",
      "rdfs:label": "has RDF payload flow",
      "rdfs:comment": "Links a Knop to its RDF payload flow."
    },
    {
      "@id": "hasNonRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "NonRdfPayloadFlow/",
      "rdfs:label": "has Non-RDF payload flow",
      "rdfs:comment": "Links a Knop to its non-RDF payload flow."
    },
    {
      "@id": "Version/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["/", "dcat:Dataset"],
      "rdfs:label": "Version",
      "rdfs:comment": "One version of the artifact represented by a Flow. A Version represents the Flow's artifact at a particular point in time."
    },
    {
      "@id": "hasVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": ["containsSfloResource/", "dcat:hasVersion"],
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "has version",
      "rdfs:comment": "Associates a Flow with one of its contained Versions."
    },
    {
      "@id": "hasCurrentVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasVersion/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "has current version",
      "rdfs:comment": "Associates a Flow with its current Version."
    },
    {
      "@id": "previousVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "dcat:previousVersion",
      "rdfs:domain": "Version/",
      "rdfs:range": "Version/",
      "rdfs:label": "previous version",
      "rdfs:comment": "Links to the immediately-preceding Version of this Version."
    },
    {
      "@id": "versionSequence/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "Version/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "version sequence",
      "rdfs:comment": "Monotonic integer providing the sequence number of this Version within its Flow."
    },
    {
      "@id": "AbstractFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["SfloContainerResource/"],
      "rdfs:label": "Abstract file",
      "rdfs:comment": "A file in the abstract: an exact sequence of bytes that transcends any particular location. In Semantic Flow, AbstractFiles almost always have a 'canonical' and structually-contained and conceptually-associated LocatedFile. They optionally have one or more other conceptually-associated LocatedFiles (endpoints)."
    },
    {
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": "Version/",
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Version to one of its AbstractFiles."
    },
    {
      "@id": "PointersDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent/"],
      "rdfs:label": "Pointers dataset",
      "rdfs:comment": "A minimal current-only RdfDataset for bootstrapping; typically derived from the metadata flow and addressed as <.../_knop/_pointers/>."
    },
    {
      "@id": "hasPointersDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "PointersDataset/",
      "rdfs:label": "has pointers dataset",
      "rdfs:comment": "Links a Knop to its pointers dataset component."
    },
    {
      "@id": "OpslogDatasetSeries",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "RdfDataset/",
        "SfloContainerResource/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Opslog dataset series",
      "rdfs:comment": "A conceptual series of operational log datasets (non-flow)"
    },
    {
      "@id": "OpslogDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent"],
      "rdfs:label": "Opslog dataset",
      "rdfs:comment": "An append-only operational log dataset (non-flow) associated with a Knop. Typically stored under <.../_knop/_opslog/>."
    },
    {
      "@id": "hasOpslogDatasetSeries/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links a Knop to its opslog datasets."
    },
    {
      "@id": "hasOpslogDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "OpslogDatasetSeries/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links an OpslogDatasetSeries to its OpslogDatasets."
    },
    {
      "@id": "BacklinksFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Backlinks flow",
      "rdfs:comment": "Flow containing curated or computed backlinks about the referent. Addressing is a convention within the Knop structure namespace."
    },
    {
      "@id": "hasBacklinksFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "BacklinksFlow/",
      "rdfs:label": "has backlinks flow",
      "rdfs:comment": "Links a Knop to its backlinks flow (if present)."
    }
  ]
}
ChatGPT said:
High-impact issues / things you forgot
1) containsSfloResource/ range is wrong (and missing a /)

You have:

    "rdfs:range": "SfloContainerResource" (no trailing /)

Everywhere else you use slash-terminated IRIs for your own terms. This one will point to a different IRI.

Also: the comment says “Intra-Knop structural (AND conceptual?) containment” — that reintroduces the exact ambiguity you said you wanted to pin. This property should be structural only (within _knop/ boundaries), not conceptual.

Fix:

    rdfs:range: "SfloContainerResource/"

    comment: structural containment only; explicitly not conceptual.

2) RdfDataset/ is still a KnopComponent/ (probably wrong now)

You defined:

    RdfDataset ⊑ KnopComponent ⊓ dcat:Dataset

But you’ve moved to: Flows are evolution containers; versions are contained under flows; not everything is a direct child of the Knop.

So: are datasets ever nested under flows? If yes, RdfDataset cannot be a subclass of KnopComponent (because that would imply it is always a direct child of a Knop). Right now you’ve made “RDF dataset” synonymous with “direct knob child dataset,” which will fight you.

You likely want:

    RdfDataset ⊑ SfloContainerResource (or just a conceptual class, not structural)

    and separate “direct child dataset” vs “version dataset”.

If you intend “any RDF dataset in the system is directly under a Knop” then fine—but that contradicts your Version model (versions are RDF datasets).
3) Version/ is broken: rdfs:subClassOf includes "/"

You have:

"rdfs:subClassOf": ["/", "dcat:Dataset"]

"/" is an IRI relative to base, i.e. https://semantic-flow.../ (the ontology IRI). That’s nonsensical as a class.

Also: you said versions are contained under flows, so they should be SfloContainerResource/ not KnopComponent/ — you did not include that.

Fix:

    remove "/"

    add "SfloContainerResource/" to subClassOf

    decide whether Version is a dcat:Dataset at all (see DCAT leakage below).

4) DCAT leakage is back in the wrong places

You have:

    RdfDataset ⊑ dcat:Dataset (maybe acceptable)

    Version ⊑ dcat:Dataset (maybe acceptable for RDF versions, but not for non-RDF versions)

    hasVersion subPropertyOf dcat:hasVersion

    previousVersion subPropertyOf dcat:previousVersion

But your Version is now “version of the artifact represented by a Flow” (could be non-RDF). That makes DCAT dataset semantics wrong for non-RDF versions, and dcat:hasVersion becomes misleading.

If you keep DCAT alignment, you need to split:

    RdfVersion ⊑ Version ⊓ dcat:Dataset

    non-RDF versions are not dcat:Dataset

Or drop the dcat:* superproperties on the versioning predicates and keep them SF-only.

Right now you’re in an inconsistent middle.
5) Flow/ should probably be SfloContainerResource/ (it is via KnopComponent) but you’ve lost the RDF story

Your comments suggest:

    RdfFlow organizes versions of RDF datasets

    but you have no constraint that versions under RdfFlow are RDF datasets.

That may be fine (SHACL handles it), but at minimum your modeling story is incomplete: you have RdfDataset/ but you never relate it to versions except via text.

If you’re relying on SHACL later, ok; but right now the ontology doesn’t even provide an obvious predicate to state “this Version is an RDF dataset.”
6) hasOpslogDatasetSeries/ is wrong in three ways

You defined the class:

    OpslogDatasetSeries (missing trailing / in @id!)
    and later you reference OpslogDatasetSeries/ in hasOpslogDataset domain.

Then the property:

"@id": "hasOpslogDatasetSeries/",
"rdfs:range": "OpslogDataset/",
"rdfs:label": "has opslog dataset",
"rdfs:comment": "Links a Knop to its opslog datasets."

This is supposed to link Knop → OpslogDatasetSeries, but you made the range OpslogDataset/ and duplicated label/comment from the dataset property.

Fix:

    Make the class IRI consistent: OpslogDatasetSeries/

    hasOpslogDatasetSeries range should be OpslogDatasetSeries/

    label/comment should say “has opslog dataset series”

    keep hasOpslogDataset as series → dataset

7) You have several missing trailing slashes / inconsistent term IRIs

Examples:

    containsSfloResource range missing /

    OpslogDatasetSeries missing / in @id

    KnopComponent is referenced sometimes as "KnopComponent" (no /) e.g. OpslogDataset subClassOf includes "KnopComponent"; should be "KnopComponent/".

These will silently mint different IRIs.
8) PointersDataset/ redundantly subclasses KnopComponent/

You have:

    PointersDataset ⊑ RdfDataset ⊓ KnopComponent

But RdfDataset already subclasses KnopComponent currently, so this is redundant. Once you fix RdfDataset to not be KnopComponent, you’ll need to decide whether pointers are direct children (they probably are), so keeping PointersDataset ⊑ KnopComponent is fine—just don’t double-assert.
9) BacklinksFlow/ is not a RdfFlow/ (probably should be)

You made BacklinksFlow ⊑ Flow/, not RdfFlow/. Earlier you’d moved backlinks toward RDF-ish computed data. If backlinks are RDF graphs (they almost certainly are), then they should be RdfFlow/ for consistency with your flow taxonomy.

If you intend backlinks to sometimes be non-RDF artifacts, keep it as Flow/, but then why is it “backlinks”?
Conceptual inconsistencies / unclear commitments
A) What is a Version “made of”?

    You’ve said Flow is a lineage container for an artifact.

    You’ve also introduced RdfDataset/ as a subclass of dcat:Dataset and KnopComponent/.

    But Version is not declared to be an RdfDataset/, and you haven’t introduced an AbstractArtifact/ (or similar) class that non-RDF versions would be instances of.

So right now Version is a dataset (via DCAT) but also “artifact snapshot”, without any formal hook to discriminate RDF vs non-RDF. Your properties (hasRdfPayloadFlow vs hasNonRdfPayloadFlow) do the discrimination at the flow level, but the versions themselves remain typeless.

You will need either:

    version subclasses (RdfVersion, NonRdfVersion), or

    a property like hasRdfDataset / hasAbstractFile patterns + SHACL constraints.

You already have hasAbstractFile on Version, which is good. But for RDF versions, do they also have abstract files? If yes, then non-RDF vs RDF is not distinguished by presence of files.
B) “Intra-Knop structural (AND conceptual?) containment” is still unresolved

You said you pinned the conceptual vs structural issue. Yet your containment property comment still questions it. Decide and codify:

    containsSfloResource = structural only

    conceptual associations use other predicates (supportsReferent, etc.)

Lower-level problems (but worth fixing)

    RdfDataset/ comment: has a typo (“graphs,.”) and repeats “datasets contained under Knops as KnopComponents” which may become false once versions are not direct children.

    Flow/ comment is truncated (“identity ” with trailing space).

    AbstractFile/ comment says “structually-contained” (typo).

    OpslogDatasetSeries subclasses both RdfDataset/ and dcat:DatasetSeries. If you keep RdfDataset ⊑ dcat:Dataset, you’re effectively saying opslog series is an RDF dataset (content). That’s probably not what you want; series is an organizer.

What you likely meant (and should implement)

Based on your current direction, the minimum coherent adjustments are:

    Fix all missing / in class/property references.

    Make containsSfloResource purely structural, with range SfloContainerResource/.

    Make Flow ⊑ KnopComponent/ (already) and keep Version ⊑ SfloContainerResource/ (not KnopComponent/).

    Decide DCAT alignment strategy:

        Either keep DCAT only for RDF datasets/versions and split classes, or

        keep versioning predicates SF-only (no dcat:* superproperties).

    Fix opslog:

        OpslogDatasetSeries/ should not be an RdfDataset/ unless you truly mean it contains RDF graphs itself.

        hasOpslogDatasetSeries should point to the series, not datasets.

    If backlinks are RDF, make BacklinksFlow ⊑ RdfFlow/.

If you want a single “what you forgot” item: OpslogDatasetSeries + hasOpslogDatasetSeries are currently incorrect enough that they will break discovery and SHACL. The second biggest is the silent IRI mismatches from missing slashes.

If you want, I can rewrite just the corrected fragments (not the whole ontology) as a patch-style list of replacements.
You said:
I kinda want to leave the domain of RdfDataset loose, so other people could use it.

Version is now just a SfloContainerResource. I feel like we can get a Version's RDF-ness from its Flow. But maybe we should introduce RdfVersion and NonRdfVersion?

Here's the updated version:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:subPropertyOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": { "@id": "https://djradon.github.io/ns/djradon/" },
      "owl:versionInfo": "0.0.1",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/_payload/_default/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
    },
    {
      "@id": "SfloContainerResource/",
      "@type": "rdfs:Class",
      "rdfs:label": "SF container resource",
      "rdfs:comment": "A resource that participates in Semantic Flow structural containment (sflo:containsSfloResource). Typically represented by a slash-terminated IRI and associated with a resource page file (often index.html). This class is a platform-scoped structural notion and does not imply real-world, mereological, or metaphysical containment of the denoted referent."
    },
    {
      "@id": "containsSfloResource/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "SfloContainerResource/",
      "rdfs:range": "SfloContainerResource/",
      "rdfs:label": "contains SF resource",
      "rdfs:comment": "Intra-Knop structural containment relation. Intended for containment within Knop structure (e.g., within <.../_knop/> and its contained components). This relation is only asserted or derived inside Knop boundaries."
    },
    {
      "@id": "LocatedFile/",
      "@type": "rdfs:Class",
      "rdfs:label": "Located file",
      "rdfs:comment": "A resource whose IRI is intended to retrieve its bytes. LocatedFiles do not act as SF containers."
    },
    {
      "@id": "ResourcePageFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["LocatedFile/", "schema:WebPage"],
      "rdfs:label": "Resource page file",
      "rdfs:comment": "A LocatedFile intended to be a human-readable resource page (typically index.html) for a resource."
    },
    {
      "@id": "hasLocatedFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "LocatedFile/",
      "rdfs:label": "has located file",
      "rdfs:comment": "Links a resource to a LocatedFile whose IRI retrieves its bytes (download endpoint). Used for distribution-like resources (e.g., AbstractFile) and also for resource page files."
    },
    {
      "@id": "hasResourcePageFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasLocatedFile/",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "ResourcePageFile/",
      "rdfs:label": "has resource page file",
      "rdfs:comment": "Associates any resource (including referents) with a human-readable resource page file (typically index.html)."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop",
      "rdfs:comment": "A structural-conceptual unit that supports one referent (one-thing-per-knop). Knop instance IRIs are the Knop-structure IRIs (typically <.../_knop/>). A Knop is not itself a dataset; it is composed of datasets and other components."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop component",
      "rdfs:comment": "An SF container resource contained by a Knop that provides specific functionality. KnopComponents are typically addressed within the Knop structure namespace rooted at <.../_knop/>."
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
      "@id": "supportsReferent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "rdfs:Resource",
      "owl:inverseOf": "hasKnop/",
      "rdfs:label": "supports referent",
      "rdfs:comment": "Links a Knop to the resource it supports (its referent). This is an identification/support relation, not a containment relation."
    },
    {
      "@id": "hasKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Knop/",
      "owl:inverseOf": "supportsReferent/",
      "rdfs:label": "has knop",
      "rdfs:comment": "Associates a resource (referent) with a Knop that supports it."
    },
    {
      "@id": "RdfDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["dcat:Dataset"],
      "rdfs:label": "RdfDataset",
      "rdfs:comment": "One or more RDF graphs. Some datasets are current-only; some are versioned via Flows."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["KnopComponent/"],
      "rdfs:label": "Flow",
      "rdfs:comment": "Abstract Superclass for a KnopComponent that organizes Versions and provides a lineage identity."
    },
    {
      "@id": "RdfFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["Flow/"],
      "rdfs:label": "RdfFlow",
      "rdfs:comment": "A Flow that organizes versions of RDF datasets."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Metadata RdfFlow",
      "rdfs:comment": "Flow containing metadata about the Knop-as-Knop (including inventory snapshots computed at weave time). Typically addressed at <.../_knop/_meta/>."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Reference flow",
      "rdfs:comment": "Flow containing reference data about the Knop's referent. Typically addressed under <.../_knop/_ref/> (or another intra-knop convention)."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Payload flow",
      "rdfs:comment": "Flow containing versions of the referent digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "RdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["PayloadFlow/", "RdfFlow"],
      "rdfs:label": "RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent RdfDataset contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "NonRdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PayloadFlow/",
      "rdfs:label": "Non-RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent non-RDF digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "BacklinksFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Backlinks flow",
      "rdfs:comment": "Flow containing curated or computed backlinks about the referent. Addressing is a convention within the Knop structure namespace."
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
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has metadata flow",
      "rdfs:comment": "Links a Knop to its metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "ReferenceFlow/",
      "rdfs:label": "has reference flow",
      "rdfs:comment": "Links a Knop to its reference flow."
    },
    {
      "@id": "hasRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "RdfPayloadFlow/",
      "rdfs:label": "has RDF payload flow",
      "rdfs:comment": "Links a Knop to its RDF payload flow."
    },
    {
      "@id": "hasNonRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "NonRdfPayloadFlow/",
      "rdfs:label": "has Non-RDF payload flow",
      "rdfs:comment": "Links a Knop to its non-RDF payload flow."
    },
    {
      "@id": "hasBacklinksFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "BacklinksFlow/",
      "rdfs:label": "has backlinks flow",
      "rdfs:comment": "Links a Knop to its backlinks flow (if present)."
    },
    {
      "@id": "Version/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["SfloContainerResource"],
      "rdfs:label": "Version",
      "rdfs:comment": "One (abstract) version of the artifact represented by a Flow. A Version represents the content of the Flow's artifact at a particular point in time, but not a particular byte-representation of it."
    },
    {
      "@id": "hasVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": ["containsSfloResource/"],
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "has version",
      "rdfs:comment": "Associates a Flow with one of its contained Versions."
    },
    {
      "@id": "currentVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasVersion/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "has current version",
      "rdfs:comment": "Associates a Flow with its current Version."
    },
    {
      "@id": "previousVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Version/",
      "rdfs:range": "Version/",
      "rdfs:label": "previous version",
      "rdfs:comment": "Links to the immediately-preceding Version of this Version."
    },
    {
      "@id": "versionSequence/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "Version/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "version sequence",
      "rdfs:comment": "Monotonic integer providing the sequence number of this Version within its Flow."
    },
    {
      "@id": "AbstractFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["SfloContainerResource/"],
      "rdfs:label": "Abstract file",
      "rdfs:comment": "A file in the abstract: an exact sequence of bytes that transcends any particular location. In Semantic Flow, AbstractFiles almost always have a 'canonical' and structually-contained and conceptually-associated LocatedFile. They optionally have one or more other conceptually-associated LocatedFiles (endpoints)."
    },
    {
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": "Version/",
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Version to one of its AbstractFiles."
    },
    {
      "@id": "PointersDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent/"],
      "rdfs:label": "Pointers dataset",
      "rdfs:comment": "A minimal current-only RdfDataset for bootstrapping; typically derived from the metadata flow and addressed as <.../_knop/_pointers/>."
    },
    {
      "@id": "hasPointersDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "PointersDataset/",
      "rdfs:label": "has pointers dataset",
      "rdfs:comment": "Links a Knop to its pointers dataset component."
    },
    {
      "@id": "OpslogDatasetSeries",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SfloContainerResource/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Opslog dataset series",
      "rdfs:comment": "A conceptual series of operational log datasets (non-flow)"
    },
    {
      "@id": "OpslogDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent/"],
      "rdfs:label": "Opslog dataset",
      "rdfs:comment": "An append-only operational log dataset (non-flow) associated with a Knop. Typically stored under <.../_knop/_opslog/>."
    },
    {
      "@id": "hasOpslogDatasetSeries/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "OpslogDatasetSeries/",
      "rdfs:label": "has opslog dataset series",
      "rdfs:comment": "Links a Knop to its opslog DatasetSeries."
    },
    {
      "@id": "hasOpslogDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "OpslogDatasetSeries/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links an OpslogDatasetSeries to its OpslogDatasets."
    }
  ]
}
ChatGPT said:
Immediate correctness problems (you will mint wrong IRIs)

    Missing trailing slashes in class references

    Version/ has rdfs:subClassOf: ["SfloContainerResource"] (missing /)

    RdfPayloadFlow/ has rdfs:subClassOf: ["PayloadFlow/", "RdfFlow"] (missing /)

    OpslogDatasetSeries class @id is missing /, but you reference OpslogDatasetSeries/ elsewhere

These are not cosmetic; they create different IRIs.

Fix all of these to slash-terminated IRIs consistently.

    currentVersion/ is mislabeled and inconsistent naming

    @id is currentVersion/

    label says “has current version”

    earlier you had hasCurrentVersion/

Pick one. If you keep “hasX” naming elsewhere, rename the property to hasCurrentVersion/.

    hasVersion subPropertyOf list mixes an IRI with a string error
    You have:

"rdfs:subPropertyOf": ["containsSfloResource/"]

This is fine syntactically, but it’s odd to keep it as a one-element array. Not wrong, just inconsistent with most of your other singletons.

    previousVersion/ is now no longer aligned with anything
    You removed dcat:previousVersion superproperty (good, given non-RDF), but you also removed any statement that it’s a subproperty of something in your own ontology. That’s OK, but you should at least decide whether it’s part of your versioning backbone (probably yes).

Structural/conceptual inconsistencies

    You can’t “get a Version’s RDF-ness from its Flow” in RDF/OWL without rules
    Relying on “a version is RDF if its flow is an RdfFlow” is:

    fine operationally (your code can do it),

    but it is not expressible as simple RDFS/OWL typing without extra axioms (restrictions) or SHACL rules.

If you want the ontology to stand alone (even lightly), you should not pretend this is derivable unless you actually encode it somewhere.

This is the best argument for adding RdfVersion and NonRdfVersion:

    it makes the typing explicit and discoverable without walking “up” to the flow.

Given your current goals (simple constraints, avoid needing target typing lookups), you have two coherent options:

    Option A (explicit classes): introduce RdfVersion/ and NonRdfVersion/, and then create specialized version properties for payload flows:

        hasRdfPayloadVersion range RdfVersion

        hasNonRdfPayloadVersion range NonRdfVersion
        This preserves your “don’t look at target types” philosophy.

    Option B (no new classes): keep one Version/ and enforce the relation in SHACL:

        Shape on RdfPayloadFlow says all hasVersion values must be instances of RdfDataset (or have RDF serializations, etc.).

        Shape on NonRdfPayloadFlow says the opposite.
        This requires type assertions or at least file-shape evidence.

Right now you’re in-between: you have RdfFlow, but versions are untyped and there’s no SHACL in this snippet.

    Your definition of Version conflicts with hasAbstractFile
    You say:

    Version … “not a particular byte-representation of it.”

But then:

    Version hasAbstractFile AbstractFile

    AbstractFile is “an exact sequence of bytes”
    That is a byte-representation.

You can reconcile this, but you must tighten the wording:

    Either: Version is content-level snapshot, and AbstractFile(s) are representations of that content.

    Or: Version is a representation-level snapshot and the “not byte-representation” comment is wrong.

Given your model, it’s clearly the first. Rewrite the Version comment accordingly.

    AbstractFile comment mixes structural + conceptual again
    You reintroduced “structurally-contained and conceptually-associated LocatedFile” language. That’s exactly the conflation you said you wanted to pin.

If LocatedFile is not an SfloContainerResource, then it is not part of containsSfloResource anyway—so “structurally-contained” is false in your current formal model unless you introduce a different containment predicate for files.

Given your earlier decision, you should say:

    AbstractFile is a container resource

    It is associated with one or more LocatedFile endpoints via hasLocatedFile

    Structural containment (in the sense of containsSfloResource) applies only to container resources, not located files

Then drop “structurally-contained” from that comment.
Taxonomy issues / places you’ve left ambiguity

    RdfDataset/ ⊑ dcat:Dataset is “loose” but still drags in DCAT semantics
    You said you want RdfDataset loose so others can use it. But subclassing DCAT makes it less generally reusable unless you actually want “dataset in a catalog sense.”

If your primary meaning is “RDF dataset = one or more graphs” (RDF 1.1 dataset notion), DCAT is not the right superclass.

You should either:

    remove dcat:Dataset superclass and keep RdfDataset purely SF/semantic-web meaning, or

    keep it but acknowledge you’re defining “RDF datasets that are also DCAT datasets”.

Right now the label/comment claim the former, the superclass asserts the latter.

    PayloadFlow/ is still present despite earlier “maybe skip it”
    You kept it (fine), but now it’s doing real work as the common parent. If it’s not instantiated, that’s fine, but then you should ensure you’re not expecting a hasPayloadFlow predicate anywhere (you aren’t). So this is purely a documentation/organization node—acceptable.

    OpslogDatasetSeries still missing /
    This will break your opslog properties as written. Fix to OpslogDatasetSeries/.

Also: you are using dcat:DatasetSeries here but you said earlier you don’t want to follow DCAT’s “series is a dataset” flattening. Yet you are now importing DCAT’s series class anyway. That’s okay if you treat DCAT as an external interoperability hook, but then you should be consistent about which parts are “export semantics” vs “core semantics.”
Things you’ve probably forgotten to re-add (based on your prior design)

    You removed hasCurrentVersion/ but didn’t fix naming
    You have currentVersion/. If you standardize on hasX, rename it and keep it parallel to hasVersion/.

    No property connecting AbstractFile to LocatedFile
    You have hasLocatedFile/ but you don’t declare:

    AbstractFile uses hasLocatedFile
    That’s okay because domain is rdfs:Resource, but conceptually you probably want to say in the comment or add a more specific property:

    AbstractFile hasLocatedFile LocatedFile

If you want constraints later, you’ll need that expectation.

    No explicit statement that Flows contain Versions via containment
    You did this by making hasVersion ⊑ containsSfloResource. Good. But note: containsSfloResource range is container resources only. That implies Versions must be SfloContainerResource (you intended), but you currently have the missing slash bug (SfloContainerResource not SfloContainerResource/), which undermines this.

Should you introduce RdfVersion and NonRdfVersion?

Given your stated operational philosophy (“use properties so you don’t need target type lookups”), yes, if you also want the ontology to carry meaning without external logic.

But: introducing those classes alone doesn’t solve the “don’t inspect type” goal unless you also introduce specialized version properties (or SHACL rules). Otherwise you still need to type the Version instances.

Minimal approach aligned with your pattern:

    Classes:

        RdfVersion/ ⊑ Version/

        NonRdfVersion/ ⊑ Version/

    Properties:

        hasRdfVersion/ ⊑ hasVersion/ (domain RdfFlow/, range RdfVersion/)

        hasNonRdfVersion/ ⊑ hasVersion/ (domain Flow/ or NonRdfPayloadFlow/, range NonRdfVersion/)

Then your SHACL can be property-only, just like you prefer, and consumers don’t need to chase types.

If you don’t want the extra properties, then adding the classes doesn’t buy you much; you’ll still have to assert rdf:type on each Version or add SHACL that infers it.
Summary of concrete fixes you should do now

    Fix all missing / IRIs: SfloContainerResource/, RdfFlow/, OpslogDatasetSeries/, etc.

    Rename currentVersion/ → hasCurrentVersion/ for consistency.

    Rewrite Version/ and AbstractFile/ comments to avoid contradicting hasAbstractFile and to remove “structurally-contained LocatedFile” wording.

    Decide whether RdfDataset/ really subclasses dcat:Dataset. If “loose for others” is the goal, it probably shouldn’t.

    If you want “version RDF-ness” to be checkable without following the flow and without type lookups, add RdfVersion/ + NonRdfVersion/ and specialized version properties; otherwise accept that “RDF-ness by flow” is an application rule, not an ontology truth.

You said:
Versions could have multiple AbstractFiles, e.g. in different RDF Syntaxes, or different image formats.

One issue I see is that hasAbstractFile only hangs on Versions, but I think the OpsLogDataset should also have AbstractFiles and LocatedFiles.)

I think adding RdfDataset/ to the domain fixes it without breaking anything:

{
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": ["Version/", "RdfDataset/"],
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Version to one of its AbstractFiles."
    }
ChatGPT said:

Your intent is right (Opslog datasets need files too), but that specific change has two problems:
1) In OWL, rdfs:domain with an array is intersection, not “either”

If you assert:

"rdfs:domain": ["Version/", "RdfDataset/"]

then the formal meaning is:

    if s hasAbstractFile o then s is a Version and a RdfDataset.

That is not “Version OR RdfDataset”. It will force every subject of hasAbstractFile to be both classes. That is exactly what you said you don’t want: you’d accidentally make every Version into an RdfDataset whenever it has a file.

This is a classic OWL footgun.
If you truly want “either,” you must use a union

In OWL 2 (in RDF) you’d need an owl:unionOf class expression as the domain. JSON-LD can express it, but it’s more verbose. Example (conceptually):

    domain = Version ⊔ RdfDataset

2) It’s conceptually wrong to say “RdfDataset has abstract file”

You have two kinds of RDF datasets now:

    direct, current-only datasets (PointersDataset, OpslogDataset) — yes, these have files.

    versioned datasets — their files live on the Version.

If you put RdfDataset in the domain, you’re saying any RDF dataset can directly have files, including abstract organizer datasets you might add later. That’s probably too loose.
Better fixes (pick one)
Option A (cleanest): introduce a shared superclass for “things that have AbstractFiles”

Create:

    FileBearingResource/ ⊑ SfloContainerResource/

Then:

    Version/ ⊑ FileBearingResource/

    OpslogDataset/ ⊑ FileBearingResource/

    PointersDataset/ ⊑ FileBearingResource/ (if it has downloadable distributions)

And set:

    hasAbstractFile domain = FileBearingResource/

This avoids OWL union complexity and avoids leaking file-bearing semantics onto every RdfDataset.
Option B (minimal change): loosen domain to SfloContainerResource/

If you don’t care about OWL-level typing precision:

    domain = SfloContainerResource/

    range = AbstractFile/

and rely on SHACL to constrain where it’s allowed.

This matches your “platform structural” model and avoids the OWL intersection trap.
Option C (OWL-precise): domain is union(Version, RdfDataset)

Do it with owl:unionOf (but it’s noisier), and you still probably shouldn’t use RdfDataset for the reasons above—use a narrower class like CurrentRdfDataset/ if you go this route.
What I’d do given your current philosophy

Option A.

It keeps the ontology legible, avoids tricky OWL constructs, and matches your design that “versions are abstract snapshots but have multiple concrete file representations.”

Concrete steps:

    Add FileBearingResource/.

    Make Version/, OpslogDataset/, PointersDataset/ subclasses of it (and anything else that has downloadable files).

    Set hasAbstractFile domain to FileBearingResource/.

    Update the comment (“Links a file-bearing resource (e.g., Version or OpslogDataset) to one of its AbstractFiles.”).

If you want, paste your current OpslogDataset/ and PointersDataset/ definitions as they stand now (with trailing slashes fixed), and I’ll give the exact JSON-LD blocks to drop in.
You said:
I do think any RDF dataset can directly have files. Our only abstract organizer, the Flow, is not a RdfDataset. And with my current conceptualization, I don't think anything could be an RdfDataset without having AbstractFiles. 

FileBearingResource or FileBearingContainer ? I think having this concept is illuminating.
ChatGPT said:

Two separate points here: (1) your modeling claim “anything that is an RdfDataset has AbstractFiles” and (2) how to encode it without OWL footguns.
1) If you mean “every RdfDataset has at least one AbstractFile”

That is a strong commitment, but it’s coherent with your system (everything materializes as files). If you commit to it, encode it as a restriction on the class, not by stuffing RdfDataset into the property domain.
Why not via rdfs:domain?

Because domain only tells you: “if the property is used, the subject is of some class.” It does not express “instances of this class must have the property.”

So you want two separate axioms:

    Domain: where the property is allowed/expected structurally.

    Existential restriction: what members of a class must have.

In OWL terms:

    RdfDataset ⊑ (hasAbstractFile some AbstractFile)

And if you also want “must be a container resource”:

    RdfDataset ⊑ SfloContainerResource (or more precisely: ⊑ SfloContainerResource is already implied if it has containsSfloResource links, but better explicit)

Practically, you’ll likely enforce “minCount 1” in SHACL, but the ontology can still state the intent.
2) How to fix hasAbstractFile without union-domain complexity

If you really believe any RdfDataset and any Version can have abstract files, you can simply do:

    hasAbstractFile domain = SfloContainerResource/

    hasAbstractFile range = AbstractFile/

    plus class constraints:

        RdfDataset must have at least one hasAbstractFile

        Version must have at least one hasAbstractFile (or optional if you want “abstract version” with no representations yet)

This avoids the OWL “domain intersection” trap and stays consistent with your structural notion: only container resources participate.

Your earlier attempt (domain: [Version, RdfDataset]) was trying to encode permission and encode requirement simultaneously, and it accidentally encoded intersection.
3) FileBearingResource vs FileBearingContainer

Given your vocabulary:

    You already have SfloContainerResource/ = “container in SF structural sense.”

    “File-bearing” is not a general “resource” notion; it’s a property of SF containers that have file representations.

So prefer:

    FileBearingContainer/ (subclass of SfloContainerResource/)

Then you can state:

    RdfDataset ⊑ FileBearingContainer

    Version ⊑ FileBearingContainer

    and optionally other things (maybe AbstractFile itself is file-bearing if it has a resource page + located file endpoints).

This is illuminating and it lets you tighten the property domain cleanly:

    hasAbstractFile domain = FileBearingContainer/

Now the model says:

    only file-bearing containers have abstract files

    and you can still assert existential requirements on RdfDataset.

Recommended minimal set of axioms

    FileBearingContainer/ ⊑ SfloContainerResource/

    hasAbstractFile:

        domain FileBearingContainer/

        range AbstractFile/

    RdfDataset/ ⊑ FileBearingContainer/

    Version/ ⊑ FileBearingContainer/ (if you really want every version to have ≥1 representation)

    (Optional) AbstractFile/ ⊑ FileBearingContainer/ if abstract files also always have a resource page / canonical located file.

4) One more inconsistency you should resolve explicitly

You currently define Version as “not a particular byte-representation,” but with Version ⊑ FileBearingContainer + mandatory abstract files, a Version becomes “content snapshot with at least one representation.” That’s fine, but update the comment so it’s not self-contradictory.
Bottom line

Yes: introduce FileBearingContainer/ (not Resource). Then:

    use it as the domain of hasAbstractFile

    make RdfDataset a subclass of it

    enforce “must have at least one abstract file” as a class constraint (OWL restriction if you want, SHACL minCount if you’re actually validating)

If you paste your current RdfDataset/, Version/, and hasAbstractFile/ blocks, I can rewrite them with the exact JSON-LD you’re already using (including the owl:Restriction form if you want the ontology to state it, not just SHACL).
You said:
ResourcePages are only represented as LocatedFiles. The things that have ResourcePages don't have AbstractResource pages... partially because it goes against the structural patterns... resourcepages need to be located directly under the things they represent, so there's no room for an AbstractFile IRI. 

No FileBearingContainer directly has a particular byte-representation. It's a content-level thing. particular-byte representations happen at the AbstractFile level. 
ChatGPT said:

You’re mixing two incompatible commitments, and you need to pick one explicitly:

    “Resource pages are located directly under the thing they describe; no AbstractFile container exists for them.”

    “All byte-representations happen at AbstractFile; file-bearing containers never directly have byte-representations.”

Those can coexist, but only if you accept that resource pages are not treated as “byte-representations” of the thing in the same sense as payload distributions. Right now your ontology says hasResourcePageFile ⊑ hasLocatedFile, and hasLocatedFile is explicitly “IRI retrieves its bytes.” So resource pages are byte-representations in the minimal sense of “a located file endpoint.”

So the correct reconciliation is:

    hasResourcePageFile attaches a located file endpoint that is not mediated by AbstractFile.

    hasAbstractFile attaches representation bundles (abstract byte sequences) that are mediated by AbstractFile.

That’s fine—but then “byte-representation” splits into two notions:

    direct endpoint (resource pages)

    abstract-file mediated representation (payload/data distributions)

You should encode that split, otherwise your comments will keep contradicting your structure.
Fix the conceptual model with one clear rule
Rule A: Resource pages are presentation endpoints, not “distributions”

Keep:

    hasResourcePageFile: rdfs:Resource → ResourcePageFile (subproperty of hasLocatedFile)

Do not force them into AbstractFile. That matches your structural constraint (“no room for AbstractFile IRI”).
Rule B: AbstractFile is only for representation bundles you want versioned/typed/checksummed/etc.

Keep:

    hasAbstractFile: FileBearingThing → AbstractFile

    AbstractFile hasLocatedFile LocatedFile (canonical and alternates)

Now you can truthfully say:

    “No FileBearingThing directly has a particular byte-representation” except via hasLocatedFile endpoints used for resource pages.

But you must stop using “byte-representation” as if it’s a single channel.
What to rename / how to avoid the contradiction

Your term FileBearingContainer is now misleading given your new sentence:

    No FileBearingContainer directly has a particular byte-representation. Particular-byte representations happen at the AbstractFile level.

But resource pages are particular-byte located files, and they attach directly to the represented thing. So either:
Option 1 (recommended): Rename the class to what you actually mean

Call it something like:

    AbstractFileMediatedResource/

    RepresentedResource/ (if you want to stress “has mediated representations”)

    HasAbstractFileResource/ (blunt but precise)

This class means: “resources that have one or more AbstractFiles.”

Then your statement becomes true:

    “No HasAbstractFileResource directly has byte-representations; it has AbstractFiles, and those have LocatedFiles.”
    Resource pages remain outside this class.

Option 2: Keep FileBearingContainer but admit it’s not exclusively abstract-file mediated

Then you’re forced to say:

    “FileBearingContainer may have direct located files (resource pages) and/or mediated ones via AbstractFile.”
    That dilutes the concept; you said you find it illuminating, so this is probably not what you want.

Concrete ontology adjustments implied by your message

    Do not make resource pages participate in AbstractFile

    Keep ResourcePageFile ⊑ LocatedFile

    Keep hasResourcePageFile ⊑ hasLocatedFile

    Do not add a ResourcePageAbstractFile or similar.

    Add an explicit class for “things that have AbstractFiles”
    Example:

    AbstractFileBearingResource/ ⊑ SfloContainerResource/

Then:

    RdfDataset/ ⊑ AbstractFileBearingResource/ (if you truly mean every RDF dataset has abstract files)

    Version/ ⊑ AbstractFileBearingResource/ (if every version must have at least one representation bundle)

    Define:

    hasAbstractFile domain = AbstractFileBearingResource/ (not Version, not RdfDataset, not union)

    hasAbstractFile range = AbstractFile/

    Update comments to stop claiming exclusivity about byte-representations

    Resource pages are direct located endpoints.

    AbstractFile is the mediated byte-sequence identity.

One more inconsistency to decide now

If “resource pages need to be located directly under the things they represent,” then for a Version with abstract files in multiple syntaxes, you currently have:

    Version/ container exists

    resource page file exists directly under it (good)

    abstract files are containers under it (also good)

That’s structurally consistent. The “no room for AbstractFile IRI” argument applies to resource pages because you want index.html directly under the represented resource, not to versions/payload.

So: you’re not banning AbstractFile generally; you’re banning AbstractFile for the resource page itself. That should be stated explicitly so readers don’t generalize it incorrectly.

If you want, I can propose exact names that preserve your aesthetic and avoid “FileBearingContainer” lying about direct located files. The key is: name the class after the mediated representation mechanism (hasAbstractFile), not after “files” broadly.
You said:
ResourcePages are not in Abstract file:

{
      "@id": "ResourcePageFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["LocatedFile/", "schema:WebPage"],
      "rdfs:label": "Resource page file",
      "rdfs:comment": "A LocatedFile intended to be a human-readable resource page (typically index.html) for a resource."
    }

And ResourcePages are not FileBearingThing, they are not AbstractFile. They are not byte-representations of an AbstractFile, or representations of anything else except themselves. I think it's consistent. 
ChatGPT said:

Your class story is consistent, but your semantics are still muddled in one specific way: you’re equivocating on “representation.”
What is consistent

    ResourcePageFile/ ⊑ LocatedFile/ and not AbstractFile/: consistent.

    You are not claiming resource pages are “contained in” an AbstractFile: consistent.

    Resource pages can be “about” anything without implying it is an SfloContainerResource: consistent.

The remaining inconsistency (it’s in your wording, not your classes)

You said:

    ResourcePages … are not byte-representations … they are not representations of anything else except themselves.

But you also define:

    hasResourcePageFile = “Associates any resource with a human-readable resource page file … for a resource.”

That is a representational relation in the ordinary sense (“this file is a page for that resource”). You’re trying to reserve “representation” for payload/distribution representations (i.e., things mediated by AbstractFile)—which is fine—but then you must not use blanket language like “not representations of anything else.” They are presentation representations even if they are not AbstractFile-mediated.

So the ontology is internally consistent, but the conceptual boundary is underspecified:

    AbstractFile-mediated representation: byte-sequence identity, alternative endpoints, formats, checksums, etc.

    Resource page representation: a web page intended to present/describe a resource to humans, directly located under the resource.

Those are two different representational mechanisms. Your model supports that distinction. Your prose currently denies it.
Minimal fix

Do not say “resource pages are not representations.” Say:

    “Resource pages are not AbstractFile-mediated representations and are not AbstractFiles.”

    “They are standalone located web pages whose relationship to a resource is hasResourcePageFile (a presentation association).”

One more place this matters (for your FileBearingThing / FileBearingContainer debate)

If you keep a class like FileBearingContainer meaning “has AbstractFiles,” then resource pages being direct LocatedFiles does not conflict at all—because they’re attached via hasResourcePageFile, not via hasAbstractFile.

So: yes, it’s consistent as long as you’re precise that “byte-representation” means AbstractFile-mediated representation. If you mean “any downloadable bytes,” then resource pages contradict that.
Concrete wording tweaks you should make

    In hasLocatedFile comment, avoid “download endpoint” if you don’t want people to read it as “representation.” Call it “byte-retrieval endpoint.”

    In hasResourcePageFile comment, explicitly say: “presentation page; not an AbstractFile; not a distribution mechanism.”

That will eliminate the only real inconsistency you still have.
You said:
OK, I think we're good now:

{
  "@context": {
    "@base": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "dcterms": "http://purl.org/dc/terms/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "sh": "http://www.w3.org/ns/shacl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "schema": "https://schema.org/",
    "vann": "http://purl.org/vocab/vann/",
    "rdfs:subClassOf": { "@type": "@id" },
    "rdfs:subPropertyOf": { "@type": "@id" },
    "rdfs:domain": { "@type": "@id" },
    "rdfs:range": { "@type": "@id" }
  },
  "@graph": [
    {
      "@id": "",
      "@type": "owl:Ontology",
      "dcterms:title": "Semantic Flow Ontology",
      "dcterms:description": "The Semantic Flow ontology defines the classes and properties used for defining and describing semantic meshes.",
      "dcterms:creator": { "@id": "https://djradon.github.io/ns/djradon/" },
      "owl:versionInfo": "0.0.1",
      "owl:versionIRI": {
        "@id": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/_payload/_default/sflo.ttl"
      },
      "vann:preferredNamespacePrefix": "sflo",
      "vann:preferredNamespaceUri": "https://semantic-flow.github.io/ontology/semantic-flow-ontology/"
    },
    {
      "@id": "SfloContainerResource/",
      "@type": "rdfs:Class",
      "rdfs:label": "SF container resource",
      "rdfs:comment": "A resource that participates in Semantic Flow structural containment (sflo:containsSfloResource). Typically represented by a slash-terminated IRI and associated with a resource page file (often index.html). This class is a platform-scoped structural notion and does not imply real-world, mereological, or metaphysical containment of the denoted referent."
    },
    {
      "@id": "containsSfloResource/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "SfloContainerResource/",
      "rdfs:range": "SfloContainerResource/",
      "rdfs:label": "contains SF resource",
      "rdfs:comment": "Intra-Knop structural containment relation. Intended for containment within Knop structure (e.g., within <.../_knop/> and its contained components). This relation is only asserted or derived inside Knop boundaries."
    },
    {
      "@id": "LocatedFile/",
      "@type": "rdfs:Class",
      "rdfs:label": "Located file",
      "rdfs:comment": "A resource whose IRI is intended to retrieve its bytes. LocatedFiles do not act as SF containers."
    },
    {
      "@id": "ResourcePageFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["LocatedFile/", "schema:WebPage"],
      "rdfs:label": "Resource page file",
      "rdfs:comment": "A LocatedFile intended to be a human-readable resource page (typically index.html) for a resource."
    },
    {
      "@id": "hasLocatedFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "LocatedFile/",
      "rdfs:label": "has located file",
      "rdfs:comment": "Links a resource to a LocatedFile whose IRI retrieves a byte-retrieval endpoint."
    },
    {
      "@id": "hasResourcePageFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasLocatedFile/",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "ResourcePageFile/",
      "rdfs:label": "has resource page file",
      "rdfs:comment": "Associates any resource (including referents) with a human-readable presentation page."
    },
    {
      "@id": "Knop/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop",
      "rdfs:comment": "A structural-conceptual unit that supports one referent (one-thing-per-knop). Knop instance IRIs are the Knop-structure IRIs (typically <.../_knop/>). A Knop is not itself a dataset; it is composed of datasets and other components."
    },
    {
      "@id": "KnopComponent/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "SfloContainerResource/",
      "rdfs:label": "Knop component",
      "rdfs:comment": "An SF container resource contained by a Knop that provides specific functionality. KnopComponents are typically addressed within the Knop structure namespace rooted at <.../_knop/>."
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
      "@id": "supportsReferent/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Knop/",
      "rdfs:range": "rdfs:Resource",
      "owl:inverseOf": "hasKnop/",
      "rdfs:label": "supports referent",
      "rdfs:comment": "Links a Knop to the resource it supports (its referent). This is an identification/support relation, not a containment relation."
    },
    {
      "@id": "hasKnop/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "rdfs:Resource",
      "rdfs:range": "Knop/",
      "owl:inverseOf": "supportsReferent/",
      "rdfs:label": "has knop",
      "rdfs:comment": "Associates a resource (referent) with a Knop that supports it."
    },
    {
      "@id": "RdfDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["dcat:Dataset", "FileBearingContainer/"],
      "rdfs:label": "RdfDataset",
      "rdfs:comment": "One or more RDF graphs. Some datasets are current-only; some are versioned via Flows."
    },
    {
      "@id": "Flow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["KnopComponent/"],
      "rdfs:label": "Flow",
      "rdfs:comment": "Abstract Superclass for a KnopComponent that organizes Versions and provides a lineage identity."
    },
    {
      "@id": "RdfFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "RdfFlow",
      "rdfs:comment": "A Flow that organizes versions of RDF datasets."
    },
    {
      "@id": "KnopMetadataFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Metadata RdfFlow",
      "rdfs:comment": "Flow containing metadata about the Knop-as-Knop (including inventory snapshots computed at weave time). Typically addressed at <.../_knop/_meta/>."
    },
    {
      "@id": "ReferenceFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Reference flow",
      "rdfs:comment": "Flow containing reference data about the Knop's referent. Typically addressed under <.../_knop/_ref/> (or another intra-knop convention)."
    },
    {
      "@id": "PayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "Flow/",
      "rdfs:label": "Payload flow",
      "rdfs:comment": "Flow containing versions of the referent digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "RdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["PayloadFlow/", "RdfFlow/"],
      "rdfs:label": "RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent RdfDataset contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "NonRdfPayloadFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "PayloadFlow/",
      "rdfs:label": "Non-RDF PayloadFlow",
      "rdfs:comment": "Flow containing versions of the referent non-RDF digital artifact contained in the Knop. Typically addressed under <.../_knop/_payload/>."
    },
    {
      "@id": "BacklinksFlow/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": "RdfFlow/",
      "rdfs:label": "Backlinks flow",
      "rdfs:comment": "Flow containing curated or computed backlinks about the referent. Addressing is a convention within the Knop structure namespace."
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
      "@id": "hasKnopMetadataFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "KnopMetadataFlow/",
      "rdfs:label": "has metadata flow",
      "rdfs:comment": "Links a Knop to its metadata flow."
    },
    {
      "@id": "hasReferenceFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "ReferenceFlow/",
      "rdfs:label": "has reference flow",
      "rdfs:comment": "Links a Knop to its reference flow."
    },
    {
      "@id": "hasRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "RdfPayloadFlow/",
      "rdfs:label": "has RDF payload flow",
      "rdfs:comment": "Links a Knop to its RDF payload flow."
    },
    {
      "@id": "hasNonRdfPayloadFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "NonRdfPayloadFlow/",
      "rdfs:label": "has Non-RDF payload flow",
      "rdfs:comment": "Links a Knop to its non-RDF payload flow."
    },
    {
      "@id": "hasBacklinksFlow/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasFlow/",
      "rdfs:range": "BacklinksFlow/",
      "rdfs:label": "has backlinks flow",
      "rdfs:comment": "Links a Knop to its backlinks flow (if present)."
    },
    {
      "@id": "Version/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["FileBearingContainer/"],
      "rdfs:label": "Version",
      "rdfs:comment": "One (abstract) version of the artifact represented by a Flow. A Version represents the content of the Flow's artifact at a particular point in time, but not a particular byte-representation of it."
    },
    {
      "@id": "RdfVersion/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["Version/"],
      "rdfs:label": "RDF Version",
      "rdfs:comment": "A Version of the RDF Dataset represented by a RdfFlow."
    },
    {
      "@id": "NonRdfVersion/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["Version/"],
      "rdfs:label": "Non-RDF Version",
      "rdfs:comment": "A Version of the non-RDF digital artifact represented by a NonRdfFlow."
    },
    {
      "@id": "hasVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "has version",
      "rdfs:comment": "Associates a Flow with one of its contained Versions."
    },
    {
      "@id": "currentVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasVersion/",
      "rdfs:domain": "Flow/",
      "rdfs:range": "Version/",
      "rdfs:label": "current version",
      "rdfs:comment": "Associates a Flow with its current Version."
    },
    {
      "@id": "previousVersion/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "Version/",
      "rdfs:range": "Version/",
      "rdfs:label": "previous version",
      "rdfs:comment": "Links to the immediately-preceding Version of this Version."
    },
    {
      "@id": "currentVersionSequenceNumber/",
      "@type": "owl:DatatypeProperty",
      "rdfs:domain": "Flow/",
      "rdfs:range": "xsd:integer",
      "rdfs:label": "current version sequence number",
      "rdfs:comment": "Monotonic integer providing the currentsequence number of this Version within its Flow."
    },
    {
      "@id": "AbstractFile/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["SfloContainerResource/"],
      "rdfs:label": "Abstract file",
      "rdfs:comment": "A file in the abstract: an exact sequence of bytes that transcends any particular location. In Semantic Flow, AbstractFiles almost always have a 'canonical' and structually-contained and conceptually-associated LocatedFile. They optionally have one or more other conceptually-associated LocatedFiles (endpoints)."
    },
    {
      "@id": "hasAbstractFile/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "containsSfloResource/",
      "rdfs:domain": ["FileBearingContainer/"],
      "rdfs:range": "AbstractFile/",
      "rdfs:label": "has abstract file",
      "rdfs:comment": "Links a Version to one of its AbstractFiles."
    },
    {
      "@id": "FileBearingContainer/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["SfloContainerResource/"],
      "rdfs:label": "File-bearing container",
      "rdfs:comment": "A SfloContainerResource that may contains one or more AbstractFiles."
    },
    {
      "@id": "PointersDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent/"],
      "rdfs:label": "Pointers dataset",
      "rdfs:comment": "A minimal current-only RdfDataset for bootstrapping; typically derived from the metadata flow and addressed as <.../_knop/_pointers/>."
    },
    {
      "@id": "hasPointersDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "PointersDataset/",
      "rdfs:label": "has pointers dataset",
      "rdfs:comment": "Links a Knop to its pointers dataset component."
    },
    {
      "@id": "OpslogDatasetSeries/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": [
        "SfloContainerResource/",
        "dcat:DatasetSeries"
      ],
      "rdfs:label": "Opslog dataset series",
      "rdfs:comment": "A conceptual series of operational log datasets (non-flow)"
    },
    {
      "@id": "OpslogDataset/",
      "@type": "rdfs:Class",
      "rdfs:subClassOf": ["RdfDataset/", "KnopComponent/"],
      "rdfs:label": "Opslog dataset",
      "rdfs:comment": "An append-only operational log dataset (non-flow) associated with a Knop. Typically stored under <.../_knop/_opslog/>."
    },
    {
      "@id": "hasOpslogDatasetSeries/",
      "@type": "owl:ObjectProperty",
      "rdfs:subPropertyOf": "hasKnopComponent/",
      "rdfs:domain": "Knop/",
      "rdfs:range": "OpslogDatasetSeries/",
      "rdfs:label": "has opslog dataset series",
      "rdfs:comment": "Links a Knop to its opslog DatasetSeries."
    },
    {
      "@id": "hasOpslogDataset/",
      "@type": "owl:ObjectProperty",
      "rdfs:domain": "OpslogDatasetSeries/",
      "rdfs:range": "OpslogDataset/",
      "rdfs:label": "has opslog dataset",
      "rdfs:comment": "Links an OpslogDatasetSeries to its OpslogDatasets."
    }
  ]
}
ChatGPT said:

Not “good now.” It’s close, but there are several concrete breakages and a couple of conceptual contradictions you’re currently papering over.
Hard errors / broken IRIs / OWL semantics

    RdfDataset references FileBearingContainer/ before it is defined
    Not technically illegal, but it’s avoidable and makes human review harder. More importantly, it hides that you haven’t actually made RdfDataset an SfloContainerResource directly (you’re relying on FileBearingContainer for that).

    Missing trailing slashes in a few places
    You use slash-terminated IRIs for classes/properties, but you have several without /:

    OpslogDatasetSeries/ is fine, but earlier you had OpslogDatasetSeries in some versions; ensure consistency everywhere.

    In your current snippet: SfloContainerResource appears without / nowhere—good. But:

        In hasAbstractFile you wrote "rdfs:domain": ["FileBearingContainer/"] as an array. That’s harmless but pointless and suggests you may still be thinking arrays mean “OR”. For domain, a single IRI is clearer.

    hasAbstractFile comment is wrong
    You changed its domain to FileBearingContainer, but the comment still says “Links a Version…”. It now links any file-bearing container.

    FileBearingContainer comment has grammar + semantic looseness

    “may contains one or more AbstractFiles”

    grammar: “may contain”

    semantics: “may” is too weak if you’re using it as the superclass for RdfDataset and Version and you believe those always have AbstractFiles. Either:

        keep “may” but then don’t subclass RdfDataset under it if your intent is “must”; or

        keep the subclassing and add a SHACL/OWL restriction to encode “at least 1.”

    containsSfloResource range is too narrow
    You set range to SfloContainerResource/. That means you cannot use containsSfloResource to contain AbstractFile’s canonical LocatedFile if you ever want to model that containment. You keep saying “structurally-contained LocatedFile” inside AbstractFile—but your containment predicate forbids containing LocatedFile because it’s not an SfloContainerResource. You need to decide:

    Either: structural containment in RDF only applies to container resources, and file containment is not expressed (only via hasLocatedFile).

    Or: allow containment of LocatedFile and set range to rdfs:Resource (or create a second predicate for file containment).

Right now your comments claim one thing while your range enforces the other.
Conceptual mismatches you haven’t actually resolved

    You’re implying “every RdfDataset has AbstractFiles,” but you did not encode it
    You said that’s your conceptualization. In the ontology you only wrote:

    RdfDataset ⊑ FileBearingContainer

But FileBearingContainer does not require hasAbstractFile, it just “may contain”. So your actual model currently permits an RdfDataset with zero abstract files.

If you mean “cannot exist without at least one,” add a SHACL constraint (preferred in your world) or an OWL restriction.

    You’re treating AbstractFile as “exact byte sequence” but refusing to give it a byte identity hook
    If AbstractFile is “exact bytes,” you usually want at least one of:

    checksum/digest

    byte size

    canonical located endpoint

You have hasLocatedFile but no way to say which is canonical vs mirror, and no integrity property. That might be intentionally out-of-scope, but your comment says “almost always have a canonical located file.” If that’s part of the conceptual model, you should provide:

    canonicalLocatedFile (subProperty of hasLocatedFile)

    optionally mirrorLocatedFile

Otherwise “canonical” remains hand-wavy.

    currentVersionSequenceNumber is a strange surrogate
    You already have currentVersion linking to a Version. The “current sequence number” belongs either on the current Version (as versionSequence) or is derivable. Duplicating it on Flow creates easy inconsistency (sequence number doesn’t match currentVersion’s actual sequence). If you keep it, you need an explicit rationale: “optimization for pointer-only clients.” Otherwise delete.

    previousVersion has no subProperty relation
    You removed dcat:previousVersion and also didn’t make it a subProperty of anything. Fine if you’re divorcing from DCAT, but then your earlier DCAT alignment is partly undone. If you keep it as SF-specific, consider:

    rdfs:subPropertyOf: something SF-specific like relatedVersion (if you care)
    Or just accept it’s standalone.

    RdfVersion / NonRdfVersion are not connected to their flows
    You created the subclasses, but nothing constrains:

    RdfFlow hasVersion only RdfVersion

    NonRdfPayloadFlow hasVersion only NonRdfVersion
    Without SHACL, these are just labels. If you want simplified checking (you do), you need SHACL anyway, so at least note it.

    The core “quasi-inconsistency” is still only in prose
    You’ve correctly avoided making <referent-token>/ a SfloContainerResource, but you still need one sentence in the ontology comments somewhere (probably on Knop/) that formalizes:

    “Path containment outside _knop/ is not interpreted as containsSfloResource.”

    “containsSfloResource begins at Knop-structure boundaries.”

Right now that rule is split across multiple comments and can still be misread.
What you likely forgot (based on your stated goals)

    A property linking a Knop to its “root token IRI”
    You have supportsReferent, but you previously wanted “knop can support multiple IRIs.” That implies either:

    allow multiple supportsReferent values (then it’s not “one referent”), or

    introduce supportedIdentifier / supportedIri as a datatype-ish IRI list, leaving supportsReferent to mean the canonical referent IRI.

You said you wanted numerous IRIs; the ontology currently doesn’t capture that without weakening single-referent.

    A way to state “Knop structure IRI corresponds to referent-token IRI”
    You were worried about the base IRI being mistaken for a container. A small, explicit property helps, e.g.:

    knopForToken or structuralRootFor (Knop → referent-token IRI)
    But you may be intentionally keeping that out of RDF. If so, the comments should say it’s an operational convention, not an ontology relation.

The specific change I recommend (minimal, consistent with your current direction)

    Change containsSfloResource range to rdfs:Resource or fix the AbstractFile comment to stop claiming structural containment of located files.

Given your current model (located files are not containers), I’d do:

    keep range = SfloContainerResource/

    update AbstractFile comment to remove “structurally-contained” for LocatedFile and say “conceptually associated via hasLocatedFile; filesystem may place a canonical located file under the abstract file IRI.”

That avoids inventing RDF containment for non-container things.

And:

    Replace hasAbstractFile domain array with a single IRI.

    Fix FileBearingContainer comment + add a SHACL minCount=1 for RdfDataset and Version if that’s truly invariant.

If you want, paste the SHACL file you’re using for these constraints and I’ll point out the exact shapes to add for:

    RdfFlow → only RdfVersion

    NonRdfPayloadFlow → only NonRdfVersion

    xor between hasRdfPayloadFlow and hasNonRdfPayloadFlow

    minCount for hasAbstractFile on RdfDataset and Version

You said:
Regarding #5, I'm thinking to change the property to "containsSfloContainer" and changing "SfloContainerResource" to "SfloContainer". We have a different property for containing LocatedFiles. 

every RdfDataset should have AbstractFiles, but I don't want to require it. We can create a SHACL constraint with Warning.

Versions will have a Version identifier that is not necessarily just the sequence number. I keep currentVersionSequenceNumber on the Flow so applications don't have to derive the next sequence number by looking at the Versions.

Knops may have multiple supportedReferents. Don't get confused with an IRI having a single referent. 


ChatGPT said:
1) Rename SfloContainerResource → SfloContainer and containsSfloResource → containsSfloContainer

Yes. That fixes the ambiguity and also forces you to stop implying you can contain LocatedFile with the same predicate.

Do it as a clean split:

    SfloContainer/ (class)

    containsSfloContainer/ (object property)

        domain: SfloContainer/

        range: SfloContainer/

Then introduce a separate containment-ish relation for files only if you truly need it in RDF:

    containsLocatedFile/

        domain: SfloContainer/ or AbstractFile/ (depending on whether you want file containment only under AbstractFile)

        range: LocatedFile/

But be careful: if you add containsLocatedFile, you are asserting a conceptual/structural relationship, not just “filesystem layout.” If you only care that the bytes are retrievable, hasLocatedFile is enough and you can skip any “contains” predicate for files.
2) “Every RdfDataset should have AbstractFiles, but I don’t want to require it”

That’s coherent if you treat it as a platform convention rather than an ontological truth.

Do not encode it in OWL (no existential restriction). Do encode it as SHACL with severity Warning (or whatever severity you use). That matches your stated intent.

One thing to tighten: your current subclassing RdfDataset ⊑ FileBearingContainer is now misleading if you don’t require files. It says “this is a file-bearing container,” but you’re explicitly allowing file-less datasets. Options:

    Keep the class name but weaken the comment: “intended to have abstract files.”

    Or rename to something intention-shaped, e.g. AbstractFileCapableContainer/ (ugly) or RepresentableContainer/.

    Or drop the subclass and rely purely on SHACL.

Given your preference for “should but not must,” the least confusing is: keep it, but phrase it as intent and enforce via SHACL warning.
3) currentVersionSequenceNumber on Flow

Your rationale (“clients shouldn’t have to scan versions to compute next sequence”) is valid only if:

    versions can be pruned/hidden/remote so scanning is expensive, or

    you want a pointer-only client to work without enumerating versions.

But you still have a consistency risk. Minimal mitigation:

    Make the property name match what it actually is: it’s the next allocatable sequence or the current max, not “current version sequence number” unless it must equal currentVersion/versionSequence.

        Better: latestVersionSequenceNumber/ (max existing)

        Or: nextVersionSequenceNumber/ (next allocatable)

    If you keep your current name, at least update the comment to specify whether it equals the current version’s sequence or the next.

Right now your comment says “current sequence number of this Version within its Flow” (and it’s on Flow). That’s contradictory phrasing.
4) Multiple supportsReferent values

This is where you’re still at risk of stepping on your “single referent” principle.

If a Knop can have multiple supportsReferent values, then either:

    you’ve loosened the “single referent” principle to “single supported thing but multiple identifiers” (which is fine), or

    you’re actually allowing multiple referents.

Your sentence:

    Don’t get confused with an IRI having a single referent.

…doesn’t resolve the modeling ambiguity, because in RDF multiple IRIs can denote different things, and your system won’t know whether you mean aliases vs distinct referents.

So, you need to disambiguate at the predicate level:

    Keep supportsReferent/ functional (0..1) for the canonical supported thing.

    Add supportsIdentifier/ (or supportsReferentIdentifier/) for additional IRIs that are asserted to be identifiers of the same referent.

If you refuse to introduce the second property, then your ontology no longer encodes “one-thing-per-knop” in any checkable way; it becomes prose-only. That’s not automatically wrong, but it is a regression versus your earlier “simplified constraint checking” goal.
5) One more issue you didn’t address: NonRdfPayloadFlow typing

You currently have:

    RdfPayloadFlow ⊑ RdfFlow

    NonRdfPayloadFlow is only a PayloadFlow (not disjoint, not typed as non-rdf)

If you care about classification, introduce a parallel:

    NonRdfFlow/ ⊑ Flow/

    NonRdfPayloadFlow ⊑ NonRdfFlow/

Otherwise you’ve created an asymmetry that will leak into SHACL (you’ll have to special-case non-rdf payload flows).
Recommended concrete edits (minimal)

    Rename:

    SfloContainerResource/ → SfloContainer/

    containsSfloResource/ → containsSfloContainer/

    Fix hasLocatedFile comment:

    “Links a resource to a LocatedFile whose IRI retrieves its bytes.” (your current wording “retrieves a byte-retrieval endpoint” is malformed)

    Decide the “multiple referents” story explicitly in vocabulary:

    Add supportsIdentifier/ (Knop → rdfs:Resource) with comment “alternative identifiers for the same supported referent.”

    Rename or clarify currentVersionSequenceNumber/:

    either rename to nextVersionSequenceNumber/ and define it as “next allocatable”

    or keep name but state whether it equals the current version’s sequence

If you paste the updated ontology after the renames, I’ll do another consistency pass and point out the next round of issues (especially around FileBearingContainer becoming “may have AbstractFiles” while being used as a superclass).
You said:
The single referent principle is that a given IRI should only have a single referent. A knop has an IRI, but it is not an IRI. I think I just want to leave as-is. 

I think NonRdfPayloadFlow is fine: PayloadFlow are not typed as RDF so there's no ambiguity. 

Please summarize your SHACL suggestions/TODO list in a prompt for a new conversation.
ChatGPT said:

Below is a clean prompt you can paste into a new conversation to drive SHACL design work. It reflects your current ontology decisions (including keeping supportsReferent as-is and not over-typing NonRDF flows) and focuses only on constraints, warnings, and TODOs, not ontology redesign.
Prompt: SHACL Review and Constraint Design for Semantic Flow Ontology

I have a mostly-stabilized ontology for Semantic Flow. I want to add SHACL constraints (with appropriate severities) to enforce platform expectations without over-constraining the ontology.

Please help me design SHACL shapes based on the following TODO list and intentions.
General Principles

    OWL classes express what things are; SHACL expresses platform expectations.

    Many constraints should be warnings, not violations.

    Do not force single-referentness via SHACL; supportsReferent may be multi-valued.

    Do not require RDF reasoning to validate constraints (simple SHACL preferred).

SHACL TODOs / Constraints to Design
1. File-bearing expectations (Warnings)

    RdfDataset

        SHOULD have at least one hasAbstractFile.

        This is a Warning, not a Violation.

    Version

        SHOULD have at least one hasAbstractFile.

        Also a Warning, not a Violation.

    FileBearingContainer

        May have hasAbstractFile, but is not required to.

2. Flow structure constraints

    Flow

        SHOULD have at least one hasVersion (Warning).

        SHOULD have at most one currentVersion (Violation).

    currentVersion

        MUST point to a Version that is also linked via hasVersion (Violation).

3. Payload flow exclusivity

    Knop

        MAY have hasRdfPayloadFlow or hasNonRdfPayloadFlow

        MUST NOT have both at the same time.

        This should be enforced via SHACL (Violation).

4. Version typing consistency (Warnings)

    If a Version is contained in:
    -ustom-class for FileBearingContainer

        a RdfFlow, it SHOULD be a RdfVersion (Warning).

    If a Version is contained in:

        a NonRdfPayloadFlow, it SHOULD be a NonRdfVersion (Warning).

5. AbstractFile containment rules

    AbstractFile

        MUST be contained by exactly one FileBearingContainer via containsSfloContainer (Violation).

    LocatedFile

        MUST NOT be the object of containsSfloContainer (Violation).

        MAY be linked only via hasLocatedFile.

6. Resource page rules

    ResourcePageFile

        MUST be a LocatedFile.

        MUST NOT be an AbstractFile.

        MUST NOT be contained via containsSfloContainer.

7. Opslog modeling constraints

    OpslogDatasetSeries

        MUST NOT be a Flow.

    OpslogDataset

        MUST be linked to exactly one OpslogDatasetSeries via hasOpslogDataset (Violation).

        SHOULD have at least one hasAbstractFile (Warning).

8. Knop structure sanity checks

    Knop

        SHOULD have exactly one KnopMetadataFlow (Warning).

        SHOULD NOT directly contain Version or AbstractFile (Violation).

    KnopComponent

        MUST be contained by exactly one Knop (Violation).

Output Requested

    Propose SHACL NodeShapes and PropertyShapes for the above.

    Clearly label each constraint with its severity (Violation vs Warning).

    Do not suggest ontology changes unless strictly necessary for SHACL to work.

    Keep shapes readable and modular (one concern per shape where possible).

If you want, the next step after this can be:

    a “minimal SHACL profile” (only violations), and

    a “platform advisory profile” (warnings + best practices).


ChatGPT can make mistakes. Check important info.
