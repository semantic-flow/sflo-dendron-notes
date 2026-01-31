---
id: lr0xiubg36nwxxq41cy1w3c
title: 2026 01 29 Dao Vs Gpdav Analysis
desc: ''
updated: 1769723645725
created: 1769718640009
---

seeAlso: https://chatgpt.com/s/dr_697bd6ecee34819182646f32b1821d61

I don’t see the uploaded paper attached in this chat (so I can’t incorporate it yet). If you re-upload it, I’ll fold its conclusions into the comparison. In the meantime, here’s the comparison you asked for.

## Is MITRE “DAO” separate, or part of D3FEND?

DAO is best understood as a *module/view* within the broader D3FEND knowledge model, not an independent “general digital artifact” ontology in the way you mean it. D3FEND explicitly introduces a “Digital Artifact Ontology (DAO)” as part of its approach, defining what counts as a “digital artifact” in their cybersecurity framing. 

## What DAO gives you (and what it doesn’t)

### DAO’s center of gravity

DAO is a domain taxonomy: a structured vocabulary of *cyber-relevant things* (files, configuration artifacts, devices, sensors, etc.) and their subtypes.

Its definition of “Digital Artifact” is intentionally broad in a security sense (info-bearing artifacts + physical digital entities) and includes the idea that “a digital object becomes a digital artifact when an actor interacts with it.” 

### Identity-over-revisions: DAO is basically not trying

DAO is not primarily about:

* “one artifact identity across many revisions,”
* revision chains,
* “mutable working state vs immutable published snapshots,”
* or representation modeling as *format families of the same content*.

It may contain *artifact types* like “snapshot” in the everyday sense (depending on the branch), but that’s different from providing a conceptual pattern for **revision tracking as first-class structure**. Your *FlowArtifact → Slice* split is doing work DAO doesn’t appear to target.

**Bottom line:** You can reuse DAO if you want its *type taxonomy* for security-flavored artifacts, but it will not replace your “humane artifact identity + revision” spine.

## What IAO gives you (and what it doesn’t)

### IAO’s center of gravity

IAO is grounded in the “information content entity” idea: information entities are generically dependent continuants “about some thing,” with examples like articles, data, layouts, graphs. ([data.4dnucleome.org][1])
IAO also explicitly distinguishes the *content* from a *digital file instance*: its “digital document” term is about an electronic file instance and “does not refer to the information content of the document.” ([GitHub][2])

IAO is rich in naming/identifier concepts (identifier, proper name, denotes, etc.). ([GitHub][2])

### Identity-over-revisions: IAO also doesn’t give you a full pattern

IAO has lots of ingredients you can *use* (identifier, denotes, aboutness, file-vs-content distinction), but it is not, by itself, a clean revision-tracking model. Its focus is: what information entities are, how they relate to what they’re about, and how they’re borne/concretized.

**Bottom line:** IAO is a good candidate to *align with* (especially for naming/identifier semantics and “content vs file instance”), but it still won’t replace your Flow/Slice concept.

## What your “general spine” has that DAO and IAO don’t

### The key differentiator: “FlowArtifact” as identity anchor + “Slice” as revision state

Your model bakes in a humane, operational distinction:

* **FlowArtifact** = the stable “thing we talk about”
* **Slice** = a particular revision state (working or historical)
* plus explicit sequencing and “currentness” (currentSlice, previousSlice, nextSliceSequenceNumber)

Neither DAO nor IAO gives you that packaged pattern.

If you want a ready-made *revision relation*, the closest widely-used semantics are PROV and PAV:

* PROV-O defines *revision* as a derivation that revises an entity into a revised version (prov:wasRevisionOf). ([W3C][3])
* PAV gives lightweight “current version / previous version / version number” properties (pav:hasCurrentVersion, pav:previousVersion, pav:version). ([pav-ontology.github.io][4])

These map extremely cleanly onto your Slice chain.

## Reuse strategy that keeps your ontology “humane”

### Recommendation: keep your small spine, but *align* to IAO/PROV/PAV/DCAT rather than importing DAO wholesale

* Keep your terms as the “product-facing” vocabulary.
* Add **mappings** (owl:equivalentClass / rdfs:subClassOf / skos:closeMatch) where they genuinely match.
* Import selectively only if you truly want their axioms/commitments.

DCAT is also conceptually compatible with your representation story: DCAT treats a dataset as a conceptual entity that can be represented by one or more distributions/serializations. ([W3C][5])

## Concrete “what can I subclass or reuse” suggestions

### From PROV + PAV (revision tracking)

* Map **your `previousSlice`** to **prov:wasRevisionOf** (or make it a subPropertyOf).
* Consider making **Slice** a prov:Entity, and treat FlowArtifact as the “general” entity:

  * Slice prov:specializationOf FlowArtifact (common PROV pattern)
* Map:

  * `currentSlice` ↔ pav:hasCurrentVersion ([pav-ontology.github.io][4])
  * `previousSlice` ↔ pav:previousVersion ([pav-ontology.github.io][4])
  * `nextSliceSequenceNumber` ↔ pav:version (if you want literal version labels) ([pav-ontology.github.io][4])

### From IAO (content vs file instance; naming)

Strong matches:

* Your **Nomen / identifier** work can align with IAO’s identifier/proper name/denotation apparatus (IAO is unusually good here). ([GitHub][2])
* Your **LocatedFile** is conceptually close to IAO’s “digital document” *if* you mean “file instance” rather than “URL endpoint.” IAO is explicit that “digital document” is the file instance, not the content. ([GitHub][2])

### From DAO (only if you want security taxonomy)

* Reuse DAO classes as *optional* specializations (“LogFile”, “ConfigArtifact”, “Sensor”, etc.) when you’re describing cyber/ops artifacts.
* Do **not** rely on DAO for your identity/revision spine. 

## Two critical issues in your current spine (worth fixing before you “generalize” it)

### 1) `RdfDataset` can’t type Slices, even though Slices are where the dataset *content* lives

Right now:

* `RdfDataset ⊑ DigitalArtifact`
* but `Slice ⊑ Realizable` and explicitly **not** a DigitalArtifact identity.

That means you can’t say “this Slice is an RDF dataset” using `rdf:type` without contortions.

**Fix pattern (simple):**

* Make **RdfDataset** a subtype of **Realizable** (content-bearing), not of DigitalArtifact; OR
* Split into two:

  * `RdfDatasetIdentity ⊑ DigitalArtifact` (for Flow/Simple artifacts that are “datasets”)
  * `RdfDatasetContent ⊑ Realizable` (for Slices and SimpleArtifacts-as-content)

### 2) “LocatedFile” is doing two jobs: *file instance* and *access endpoint*

You define it as “retrievable bytes endpoint (typically with an extension).”

* If it’s an endpoint, it’s closer to **dcat:accessURL / downloadURL** than to a file instance.
* If it’s a file instance, it aligns better with IAO “digital document.” ([GitHub][2])

You’ll get fewer downstream contradictions if you separate:

* **FileInstance** (byte-bearing thing; hashable)
* **FileAccess** (URL/path/access mechanism)

This becomes important the moment you want to say “two URLs serve the *same bytes*” (you currently can’t say that without adding hash/byte identity).

## Minimal illustrative example (your pattern, aligned to PAV/PROV)

Conceptually:

* FlowArtifact = “Alice profile dataset”
* Slice v1, v2 = immutable snapshots
* currentSlice = working draft

If you align:

* `currentSlice` subPropertyOf pav:hasCurrentVersion ([pav-ontology.github.io][4])
* `previousSlice` subPropertyOf prov:wasRevisionOf ([W3C][3])

…then external tooling that already understands PROV/PAV will immediately “get” your revision chain, while humans still see your clean Flow/Slice story.

---

If you re-upload the “COMPARATIVE STUDY…” paper, I’ll extract (1) which ontologies it compares that specifically address *identity across revisions*, and (2) any recommended modeling patterns you can lift without importing their complexity.

[1]: https://data.4dnucleome.org/ontology-terms/IAO%3A0000030/ "information content entity – 4DN Data Portal"
[2]: https://raw.githubusercontent.com/information-artifact-ontology/IAO/v2022-11-07/iao.owl "raw.githubusercontent.com"
[3]: https://www.w3.org/ns/prov-o.owl?utm_source=chatgpt.com "Untitled"
[4]: https://pav-ontology.github.io/pav/?utm_source=chatgpt.com "PAV - Provenance, Authoring and Versioning"
[5]: https://www.w3.org/TR/vocab-dcat-3/?utm_source=chatgpt.com "Data Catalog Vocabulary (DCAT) - Version 3"
