---
id: 3sqb645ltdzav06xxy9qdww
title: Report
desc: ''
updated: 1750924642542
created: 1750924640093
---

Great. I’ll create a structured summary and diagram of your semantic mesh
terminology—highlighting how compounds, system-resources, and distribution files
relate within the mesh/site distinction. I’ll clarify how names, locators, and
indicators interact with those structures and extract insights from your table
to ensure system filenames and IRIs are properly aligned.

I’ll let you know when everything is ready for review.

# Semantic Compounds in the Knowledge Mesh Structure

## Introduction

In the context of the knowledge “mesh” (a structured collection of linked
resources), the term **“node”** is being replaced by **“semantic compound”** (or
simply **“compound”**). A compound is a user-defined container that groups
related content and can itself participate as a unit within larger structures.
This document clarifies the semantic mesh structure, distinguishing **compound
containers** from **system resources**, and explains how names, locators, and
indicators function within this framework. We also illustrate these concepts
with an example hierarchy and discuss the naming scheme for distribution files.

## The Concept of “Compound”

A **compound** is a holistic resource container that encapsulates multiple
elements (datasets, files, and even other compounds) within the mesh. Each
compound is a part of a larger whole (e.g. the site or a parent compound) while
also being a self-contained whole with its own content – much like a **holon**,
which is “simultaneously a whole in and of itself, as well as a part of a larger
whole”. This part-whole nature means compounds can nest: a compound may contain
sub-compounds, forming a hierarchy. The term _compound_ emphasizes this
compositional nature (in contrast to the earlier term “node,” which was less
descriptive). Although we considered calling these just “resources,” _compound_
is preferred to avoid confusion – since in this system, **“resource”** can refer
to many things. Thus, _semantic compounds_ are the fundamental units of
user-defined content in the mesh.

Each compound corresponds to a directory (folder) in the content structure that
is **not** a reserved system folder (system folders are prefixed with an
underscore, as discussed below). Compounds serve as **containers** for content:
they may contain data files (like dataset distributions), special metadata
subfolders, and even other compounds. Importantly, a compound often represents a
distinct subject or entity in the knowledge base and has semantics beyond just a
folder – it is an _indicator_ for some subject in the domain.

## System Resources vs. Compound Containers

Within each compound’s directory, certain subdirectories and files are
designated as **system resources**. These have special names (prefixed with `_`)
and are used by the system to manage the content. By contrast, the **compound
container** itself (and any subfolders that are not system folders) represent
user-defined content resources (the “compounds”). Here is a breakdown:

- **System Resources:** Reserved folders that begin with an underscore (and
  their contents). These include:

  - **`_catalog/`** – the **catalog dataset** of a compound, containing metadata
    (and typically a listing of the compound’s contents). Every compound has its
    own `_catalog` dataset folder, which holds the compound’s metadata and child
    index.
  - **`_ref/`** – the **reference dataset** for a **reference compound**
    (explained below). This stores descriptive information about a real-world or
    conceptual entity that the compound represents.
  - **`_v-series/`** – the **version series dataset** folder, recording version
    history metadata for a versioned dataset.
  - **`_v1/`, `_v2/`, ...** – **versioned dataset** subfolders. Each is a
    snapshot of a dataset at a given version. For example, a folder `_v1/`
    contains the first version of that dataset and its distribution.
  - **`_assets/`** – the **asset file tree** for storing static files (media,
    documents, etc.) associated with the compound. Files under `_assets` (and
    similarly any files under a now-deprecated `_x/` “excluded” folder) are
    **not** themselves treated as semantic resources – only the `_assets` folder
    as a whole is a system resource container. In other words, the system does
    not semantically index every image or file; those are just binary content.
  - _(Note:_ A previously used hidden directory like `._x/` for unpublished
    content has been eliminated, since static site hosts (like GitHub Pages)
    ignore dot-prefixed folders. Instead, the mesh can use other means to mark
    content as unpublished, and “weave” (the site generator) simply ignores
    those files. This removal of `._x` avoids recursion or inaccessible content
    issues.)

- **Compound Containers (User Resources):** Every directory that is not one of
  the above system folders (and not located _within_ an `_assets` or similar
  file tree) is a **compound**. These are often user-named folders representing
  meaningful entities or datasets. For example, in a namespace, a folder like
  `djradon/` (representing a person “DJ Radon”) or `bio/` (representing a
  biography dataset) are compound containers. **All such compound folders are
  considered “user resources”** – they contain content and/or other
  sub-resources. In the structure, _every folder outside of the `_assets` and
  other system directories is a compound._

Each compound container will _always_ include a `_catalog/` system dataset as
part of its contents (required), and depending on the compound’s type, it may
include other system elements (like `_ref/` if it’s a reference compound, or
distribution files if it’s a dataset compound). However, the compound itself –
identified by its folder path – is the primary unit we talk about.

### No Recursive Catalogs for System Folders

One important design detail: system resource folders do not themselves contain
further `_catalog` datasets. The system datasets (like the catalog or version
series) are terminal in the hierarchy – their metadata is self-contained in
their distribution files. In other words, we don’t nest a catalog within a
catalog. The information about, say, a `_catalog/` or `_v-series/` is stored in
its own dataset file, so we avoid recursive structures. This keeps the hierarchy
clean: the chain of catalogs stops at the point of a system folder. The
compound’s `_catalog` file already describes the compound (and possibly its
children), so the system folder doesn’t need another catalog of its own.

## Types of Compound Resources

Not all compounds are the same; they can play different roles. The key types of
compounds include:

- **Pure Container (Namespace) Compounds:** These are compounds that primarily
  serve as containers for other compounds, without being a dataset or reference
  themselves. For example, a top-level **namespace** folder might group a set of
  resources (e.g., an `ns/` root that contains various entities). Such a
  compound will still have a `_catalog` dataset to describe its contents, but it
  has no direct data of its own (no distribution file at its root, no `_ref`).
  It exists to organize other compounds. Any compound that contains
  sub-compounds can be thought of as a namespace for those children. In effect,
  such a compound _becomes_ a local namespace within the mesh hierarchy.

- **Dataset Compounds:** A compound that represents an **information dataset**.
  These typically have a _distribution file_ at the root of the folder (for
  example, a file like `bio.trig` inside a `bio/` compound folder) containing
  the data. This kind of compound is an **information resource** in itself – its
  IRI (identifier) refers to that dataset. Dataset compounds will include:

  - A `_catalog/` folder (for metadata about the dataset compound).
  - The main distribution file (e.g., `bio.trig`). This is a concrete file
    (often in RDF TriG, Turtle, JSON-LD, etc.) containing the dataset’s content.
  - Optionally, a `_v-series/` folder if the dataset is version-controlled, plus
    one or more version folders (`_v1/`, `_v2/`, etc.) each with a snapshot of
    the dataset (e.g., `bio_v1.trig` for version 1).
  - Optionally, an `_assets/` folder for any binary files linked to the dataset.
  - The compound could also contain further sub-compounds if needed (though
    typically a dataset compound is a leaf node, it’s **not forbidden** for it
    to have sub-compounds – reflecting the holonic nature). In practice, if a
    dataset compound holds sub-compounds, it effectively acts like a series or
    container as well (see next type).

- **Reference Compounds:** A compound designated to represent a
  **non-information resource**, i.e. a real-world or conceptual entity. Such a
  compound does _not_ have a distribution file at its root. Instead, it must
  contain a `_ref/` dataset. The `_ref` dataset holds descriptive information
  **about** the entity that the compound represents. For example, if `djradon/`
  is a reference compound for a person named DJ Radon, then
  `djradon/_ref/djradon_ref.trig` might contain RDF triples describing that
  person (name, birthdate, etc.). The key point is that the IRI of the compound
  (e.g., `.../djradon`) is intended to **identify the person** (a non-document
  resource), not the data file. In semantic web terms, DJ Radon (the person) is
  a **non-information resource**, which cannot be directly retrieved; “you
  cannot retrieve \[a person] by \[an] identifier. By contrast a document is an
  information resource – you can assign it an identifier and retrieve it”.
  Therefore, to get information about the person, we provide a reference dataset
  (an information resource) in the `_ref` folder. The system knows that when an
  IRI corresponds to a reference compound, it shouldn’t return a dataset
  directly but rather link to or display the reference info. (All reference
  compounds are also container compounds in the sense that they can hold other
  content like datasets or sub-entities, since a real-world entity can have
  related information datasets under its node.)

- **Series (or Grouping) Compounds:** A compound that is both a dataset and a
  container for other datasets. This often occurs when a resource is logically
  an **aggregate or collection of sub-resources**. For example, consider a
  compound `playlists/` which represents “DJ Radon’s Playlists” as a whole but
  contains individual playlist datasets for specific dates as sub-compounds. The
  `playlists/` compound might have its own dataset (e.g., an overview or index
  stored in `playlists.trig`) and a `_catalog` to list the sub-datasets. Its
  subfolders could be individual dated playlist compounds like `1997-11-22/` and
  `1997-12-01/`, each of which is a dataset compound with its own distribution
  (e.g., `1997-11-22.trig` inside that folder). In this way, a series compound
  acts as a **hybrid**: an information resource in itself and a namespace for a
  set of related resources. The design allows a compound to fulfill multiple
  roles (again echoing the holon idea).

Each type of compound has certain combinations of required or forbidden
elements:

- A _reference compound_ requires a `_ref/` dataset and has **no** root
  distribution file (it represents an external thing, not an internal dataset).
- A _dataset compound_ requires a distribution file and no `_ref/` (because it
  _is_ an information resource, not just referring to one).
- A _series compound_ usually requires a distribution file (to describe the
  collection) **and** a `_catalog/` (to index its members), and of course it
  contains sub-compound folders (the members of the series).
- Pure container/namespaces require a `_catalog` but neither a direct
  distribution nor a `_ref` (they’re not data themselves, just organizational).
  The top-level `ns/` in our example is of this kind – it’s just a container to
  hold other compounds, with `ns/_catalog/` describing them.

All compounds share one trait: **each compound has an associated `_catalog`
system dataset**, which serves as the metadata hub for that compound. This
catalog dataset typically includes links or descriptions for the compound’s
contents and possibly some metadata about the compound itself. Because of this,
whenever you see a folder that represents a compound, you will also see a
`_catalog` subfolder inside it (populated with at least a distribution file for
the catalog, e.g. `*name*_catalog.trig`). The catalog is what enables the mesh
to be self-describing, as every compound can enumerate its own parts.

## Names, Locators, and Indicators

In the semantic mesh, it’s helpful to distinguish between **resource names**,
**indicators**, and **locators**:

- **Name:** This is the textual identifier of a resource in the context of the
  mesh. For example, `djradon` or `bio` or `playlists` as folder names serve as
  local _names_ for those resources. Names are used to construct IRIs within the
  mesh’s namespace. In a **local mesh** (unpublished), these might be used as
  relative paths or identifiers.

- **Indicator (Identifier):** An IRI (Internationalized Resource Identifier)
  that denotes a resource or entity. Each compound’s path corresponds to an IRI
  that acts as an **identifier** for some subject. Importantly, depending on the
  compound type, that subject might be:

  - An **information resource** (if the compound is a dataset, series, or pure
    container). In these cases, the IRI “refers to itself,” meaning it
    identifies the information resource embodied by that compound (its data or
    its collection). For example, the IRI ending in `/bio` would indicate the
    bio dataset resource itself. The IRI ending in `/playlists` could indicate
    the collection resource.
  - A **non-information resource** (if the compound is a reference). In this
    case, the compound’s IRI is an indicator for something in the real world or
    abstract domain. For example, `.../djradon` identifies the actual person “DJ
    Radon,” not the document. This distinction is critical: as noted, a person
    can have an identifier but cannot be “retrieved” directly – one retrieves a
    description of the person. So the compound’s IRI in a reference scenario is
    acting as a **proxy identifier** for an external thing.

- **Locator:** A direct reference (usually a full URI including file name or
  endpoint) that can be used to retrieve a representation (file or data). In our
  structure, **distribution files** and **asset files** have locators. For
  example, `.../djradon/_ref/djradon_ref.trig` is a locator for the TriG data
  file describing DJ Radon. Similarly, `.../bio/bio.trig` is a locator for the
  bio dataset content, and `.../bio/_assets/photo.png` might be a locator for an
  image file. These are typically **URLs** on a published site that point to
  actual files. They can be accessed to download or fetch the content.

In practice, an IRI can serve both as an identifier and (when dereferenced) as a
locator, depending on how the system is implemented. In this mesh:

- If you use a **compound’s IRI (without a file extension)** in a web context
  (for a published site), you will generally get a **resource page** about that
  compound. For instance, accessing the URL corresponding to the compound
  `.../djradon` might return an HTML page (or RDF description via content
  negotiation) about DJ Radon. The system is set up such that IRIs for reference
  compounds, dataset compounds, etc., **do not directly return the raw `.trig`
  or data file**; instead, they return a human-readable page or a redirection.
  In semantic web terms, the server might respond with a 303 See Other redirect
  to the actual data file or embed the relevant info. In our static site
  scenario, likely each compound has an `index.html` generated (the “resource
  page”). Thus, compound IRIs act as _indicators_ that lead you to information
  _about_ the resource (the description or metadata page).
- If you use a **distribution file path or asset path** (with file extension),
  you are using a direct locator. For example, `.../bio/bio.trig` would initiate
  download of the RDF dataset file. This distinction aligns with web best
  practices: **non-information resource IRIs yield descriptions, while
  information resource URLs yield content**.

In summary:

- _Names_ are the building blocks of IRIs (they form the paths for compounds and
  files).
- _Indicators_ are the compound IRIs that identify resources (either an info
  resource or a real-world entity).
- _Locators_ are specific URIs that retrieve concrete files (distributions,
  images, etc.).

Finally, consider **mesh vs. site**: the mesh’s compounds can be thought of as
**local names** when you’re editing or curating content. Once published to a
site (with a domain), those same paths become part of **universal IRIs**. For
example, locally you might refer to `ns/djradon/bio` as a resource name in your
mesh. On the live site, it could resolve to
`https://mydomain.com/ns/djradon/bio` – now a global identifier/locator on the
web. The structure supports both contexts. “Local” just means the base URI might
be something like a local file path or a dummy namespace, whereas “universal”
means a real HTTP URL that others can dereference. The compound paths themselves
are the same; it’s the base that differs. So **compound paths correspond to
resource names** – “local” in the mesh (e.g., for internal reference) and
becoming “universal” on the published site (for external access).

## Example: Hierarchical Structure of Compounds and System Elements

_Figure: An example semantic mesh structure._ The top-level `ns/` is a namespace
compound. It contains a system `_catalog` dataset (metadata for the namespace)
and could have an `_assets` folder for any media at that level. Inside `ns/`, we
have a user compound `djradon/` representing a person (a reference compound).
The `djradon/` folder contains its own `_catalog` (listing and metadata) and a
`_ref` dataset describing the person. The `djradon/` compound also contains two
sub-compounds: **“bio”** (a dataset) and **“playlists”** (a series container).

Within **`djradon/bio/`** (the bio dataset compound), we see a file `bio.trig` –
the dataset’s distribution file containing DJ Radon’s biographical data. The bio
compound also has optional components: a `_v-series` folder (for version history
of the bio dataset, containing e.g. a `bio_v-series.trig` file with version
metadata) along with a first version snapshot in `_v1/` (containing
`bio_v1.trig`), and an `_assets/` folder which could hold images or documents
related to the bio. All these pieces together compose the “bio” compound. The
bio compound’s IRI (e.g., `.../djradon/bio`) identifies the bio dataset
resource; if accessed via a browser, it would show a page about DJ Radon’s bio
dataset, whereas the `bio.trig` file is the raw data (accessible for machines or
download).

The **`djradon/playlists/`** compound is marked as a series. It has its own
`_catalog` dataset (to index the individual playlist entries) and also a
distribution file `playlists.trig` (not explicitly shown in the figure text, but
implied by the requirement) that might describe the playlist collection as a
whole. Under `playlists/`, there are sub-compounds for each playlist by date.
For example, **`djradon/playlists/1997-11-22/`** is a compound representing the
playlist on Nov 22, 1997. It is a dataset compound: it contains a distribution
file `1997-11-22.trig` (the data for that playlist). It would also have its own
`_catalog` (even if just to note any assets or sub-items; if it has no further
sub-compounds, the catalog might be minimal). Similarly, `1997-12-01/` would be
another playlist dataset compound, with `1997-12-01.trig`, and so on. In this
way, the `playlists/` series compound contains multiple dataset compounds. Each
of those could, if needed, have their own version folders or assets (not shown
for brevity).

This example illustrates how **compounds can nest** and carry different
semantics:

- `ns/` (namespace compound containing others),
- `djradon/` (reference compound for a person, containing reference data and
  related datasets),
- `bio/` (dataset compound),
- `playlists/` (series compound acting as a container for other datasets,
  possibly also a dataset itself if it has a summary),
- `1997-11-22/` (a dataset compound within the series).

The **system resources** like `_catalog`, `_ref`, `_v-series`, `_v1`, `_assets`
are present where needed to manage data and metadata, but the overall hierarchy
of compounds defines the logical structure of the knowledge.

## Distribution File Naming Scheme

The naming of **distribution files** (the actual data files in the datasets)
follows a consistent pattern, using the compound’s name and the type of dataset.
This makes it easy to see context from the file name and avoid ambiguity.
Essentially, the file name is constructed from the compound (or resource) name
plus an indicator of the dataset type, all the way up to the “compound-root” of
that dataset.

For example:

- A **catalog dataset** distribution file is named with the compound’s name +
  `_catalog`. If the compound is `djradon/`, its catalog file might be
  `djradon_catalog.trig` in `djradon/_catalog/`. The root namespace `ns/` has
  `ns_catalog.trig` in `ns/_catalog/`.
- A **reference dataset** file uses the compound’s name + `_ref`. For `djradon/`
  (reference compound), the file is `djradon_ref.trig` inside `djradon/_ref/`.
- A **main dataset** file for a dataset compound typically just uses the
  compound’s own name. In our example, `bio/` has `bio.trig` as its distribution
  (since “bio” is the dataset name). Likewise, a playlist subfolder
  `1997-11-22/` uses `1997-11-22.trig` as the file name.
- **Version series** datasets include `_v-series` in the name, usually prefixed
  by the relevant resource name. For instance, the version history of the
  `djradon/_catalog` might be stored in `djradon_catalog_v-series.trig` (under
  `djradon/_catalog/_v-series/`). A version history for the `bio` dataset would
  analogously use `bio_v-series.trig`. _(The attached schema table suggests
  using the compound’s own root name in the file. If there was an inconsistency,
  it should be normalized so that the file name clearly ties to the dataset it
  versions.)_
- **Version snapshots** include the version number: e.g., the first version of
  the catalog for `djradon` is `djradon_catalog_v1.trig` in
  `djradon/_catalog/_v1/`. The first version of `bio` would be `bio_v1.trig` in
  `bio/_v1/`, and so on for v2, v3, etc. Each version file name reflects its
  dataset and version index.

By constructing filenames “using the terms back up to their compound-root,” one
ensures that, for example, a file from a `_ref` or `_catalog` dataset is
unambiguously associated with the parent compound. Reading the name
`djradon_ref.trig` tells us this is the reference data for the “djradon”
compound. Similarly, `ns_catalog.trig` is clearly the catalog for the “ns”
container. This scheme is particularly useful in static file hosting where each
distribution is a separate file – the names carry context, reducing risk of
collisions and aiding clarity when files are seen out of folder context (for
instance, in search results or logs).

## Conclusion

In summary, the **semantic compound** is the core concept replacing the vaguer
“node” terminology. A compound is a **holistic container** composed of various
elements (system datasets like catalogs, optional reference or distribution
files, etc.) and possibly other compounds. The mesh is organized into these
compounds, with **system resources** providing structure and metadata. We
differentiate between how an IRI is used as an **indicator** of a resource (or
real-world entity) versus as a **locator** of a specific file. The careful
organization into catalogs, reference datasets, and version folders prevents
ambiguity and recursion in the system, making each compound self-descriptive and
manageable. By adopting clear naming conventions (for both folder names and file
names) and by clarifying the role of each part of the structure, the semantic
mesh becomes easier to navigate and reason about for both humans and machines.
This refined terminology and structure should support the creation of a robust,
self-describing knowledge system where each “compound” truly encapsulates a unit
of meaning and content within the greater whole.
