---
id: eo43ueh0viren1xkjcjregc
title: Semantic Flow URLs
desc: ''
updated: 1750514962527
created: 1750368774797
---

## Types of URLs in Semantic Flow

| sflow URL type    | sense      | example                                                  | catalog             | versioned |
|-------------------|------------|----------------------------------------------------------|---------------------|-----------|
| namespace         | reference  | `https://ex.org/ns/people/`                              | catalog.namespace   | ❌         |
| thing             | reference  | `https://ex.org/ns/people/dave`                          | thing catalog       | ❌         |
| reference-page    | content    | `https://ex.org/ns/people/dave/index.html`               | ❌                  | ❌         |
| dataset           | reference  | `https://ex.org/ns/_data/dave-bio/`                      | dataset catalog     | ✅         |
| v-series          | reference  | `https://ex.org/ns/_data/dave-bio/_v-series/`            | with dataset        | ❌         |
| distribution      | content    | `https://ex.org/ns/_data/dave-bio/_v1/dave-bio_v1.trig`  | ❌                  | ❌         |
| asset             | content    | `https://ex.org/ns/assets/logo.svg`                      | ❌                  | ❌         |
| catalog           | reference  | `https://ex.org/ns/_data/dave-bio/_catalog`              | catalog.catalog     | ✅         |

Example:
- `ns/people/` = namespace containing people-related things
- `ns/people/dave/` = refers to Dave the person
- `ns/people/dave/index.html` = is the reference page about Dave
- `ns/_data/dave-bio/_v3/personal-info_v3.trig` = is the RDF distribution
- `ns/_assets/images/dave-headshot.jpg` = is an image asset


## URL Senses

### **Content URLs**

URLs that point to **concrete information resources** (files on disk or over HTTP):

* **Distribution URLs** → materialized datasets, e.g. `test.ttl`, `dave_v1.jsonld`, `README`, etc.
* **HTML Page URLs** → e.g. `index.html`
* **Asset URLs** → e.g. `.png`, `.css`, `.js`

These are *retrievable representations* (materialized content).

---

### **Reference URLs**

URLs that refer to **non-retrievable things or conceptual entities**, including:

* **Thing URLs** → People, places, concepts, characters, etc.
* **Namespace URLs** → Organizational containers
* **Dataset URLs** → The dataset-as-entity
    * **Dataset Series URLs** → Versioning containers
    * **Catalog URLs** → Collections of referents (or datasets)


### URL Pattern Semantics

| URL Type      | Trailing Slash? | Refers to…                                    | Example                                 |
|---------------|-----------------|------------------------------------|-----------------------------------------|
| Content URL   | No              | A fetchable document or asset                 | `https://example.org/ns/foo/index.html` |
| Reference URL | Yes (`/`)       | A real-world, fictional, or conceptual thing  | `https://example.org/ns/foo/`           |

Even though datasets live under `_data/`, their URLs refer to datasets **as abstract, non-retrievable entities**. Only Distribution URLs refer to concrete, downloadable data, i.e., dataset distributions.

---

### 🧾 Summary Rule of Thumb

- If the URL ends with a slash, it's a **Reference URL** and it has the sense of refering to an entity (e.g., thing, dataset, series, catalog)**
- Otherwise, it's a **Content URL** and has the sense of being the concrete, retrievable web resource that it locates**

### Why Sense matters

- In the past, people have tried to use content URLs to represent the things they refer to. Classic example is 
