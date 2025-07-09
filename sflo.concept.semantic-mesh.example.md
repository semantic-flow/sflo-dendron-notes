---
id: s5ild34tbe4w2wt4m8ldllg
title: Example Mesh Hierarchy
desc: ''
updated: 1752025354992
created: 1750640592487
---

```file
/test-ns/                                        # namespace node
├── _meta-component/                                       # node component (metadata)
│   ├── _current/                                # component layer
│   │   ├── ns_meta.trig                         # system metadata about the namespace node
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── ns_meta.trig                         # draft system metadata
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _assets/                                     # asset tree
│   ├── _meta-component/                                   # node component (metadata)
│   │   ├── _current/                            # component layer
│   │   │   ├── ns_assets.trig                   # asset metadata
│   │   │   └── index.html                       # resource page
│   │   ├── _next/                               # component layer (draft)
│   │   │   ├── ns_assets.trig                   # draft asset metadata
│   │   │   └── index.html                       # resource page
│   │   └── index.html                           # resource page
│   ├── images/
│   │   └── logo.svg
│   └── index.html                               # resource page
└── index.html                                   # resource page

/test-ns/djradon/                                # reference node  
├── _node-handle/                                     # handle element
│   └── index.html                               # mesh node handle page
├── _ref-component/                                        # node component (reference data)
│   ├── _current/                                # component layer
│   │   ├── djradon_ref.ttl                      # reference data about the person djradon
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon_ref.ttl                      # draft reference data
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _meta-component/                                       # node component (metadata)
│   ├── _current/                                # component layer
│   │   ├── djradon_meta.trig                    # system metadata, backlinks, verification status
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon_meta.trig                    # draft system metadata
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _assets/                                     # asset tree
│   ├── _meta-component/                                   # node component (metadata)
│   │   ├── _current/                            # component layer
│   │   │   ├── djradon_assets.trig              # asset metadata
│   │   │   └── index.html                       # resource page
│   │   ├── _next/                               # component layer (draft)
│   │   │   ├── djradon_assets.trig              # draft asset metadata
│   │   │   └── index.html                       # resource page
│   │   └── index.html                           # resource page
│   ├── profile-photo.jpg
│   └── index.html                               # resource page
├── index.html                                   # resource page
└── README.md                                    # resource documentation

/test-ns/djradon-bio/                            # data node (unversioned dataset)
├── _data-component/                                       # node component (data)
│   ├── _current/                                # component layer
│   │   ├── djradon-bio.ttl                      # biographical data distribution
│   │   ├── djradon-bio.jsonld                   # alternative distribution
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-bio.ttl                      # draft biographical data
│   │   ├── djradon-bio.jsonld                   # draft alternative distribution
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _ref-component/                                        # node component (reference data) - OPTIONAL
│   ├── _current/                                # component layer
│   │   ├── djradon-bio_ref.ttl                  # reference data about the bio dataset concept
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-bio_ref.ttl                  # draft reference data
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _meta-component/                                       # node component (metadata)
│   ├── _current/                                # component layer
│   │   ├── djradon-bio_meta.trig                # dataset metadata, provenance
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-bio_meta.trig                # draft dataset metadata
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── djradon-bio.ttl                              # aggregated distribution
├── index.html                                   # resource page
└── README.md                                    # resource documentation

/test-ns/djradon/picks/                          # data node (versioned dataset)
├── _data-component/                                       # node component (data)
│   ├── _current/                                # component layer
│   │   ├── djradon-picks.ttl                    # current picks data
│   │   ├── djradon-picks.jsonld
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-picks.ttl                    # draft picks data
│   │   ├── djradon-picks.jsonld
│   │   └── index.html                           # resource page
│   ├── _v1/                                     # component layer (version 1)
│   │   ├── djradon-picks_v1.ttl                 # version 1 snapshot
│   │   └── index.html                           # resource page
│   ├── _v2/                                     # component layer (version 2)
│   │   ├── djradon-picks_v2.ttl                 # version 2 snapshot
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _ref-component/                                        # node component (reference data) - OPTIONAL
│   ├── _current/                                # component layer
│   │   ├── djradon-picks_ref.ttl                # reference data about the picks dataset
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-picks_ref.ttl                # draft reference data
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── _meta-component/                                       # node component (metadata)
│   ├── _current/                                # component layer
│   │   ├── djradon-picks_meta.trig              # versioning metadata, series info
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── djradon-picks_meta.trig              # draft versioning metadata
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── djradon-picks.ttl                            # aggregated distribution
├── index.html                                   # resource page
└── CHANGELOG.md                                 # resource documentation

/test-ns/djraon/playlists/                              # namespace node (container for playlist series)
├── _meta-component/                                       # node component (metadata)
│   ├── _current/                                # component layer
│   │   ├── playlists_meta.trig                  # metadata about playlist namespace
│   │   └── index.html                           # resource page
│   ├── _next/                                   # component layer (draft)
│   │   ├── playlists_meta.trig                  # draft metadata
│   │   └── index.html                           # resource page
│   └── index.html                               # resource page
├── index.html                                   # resource page
├── 1996-11-10/                                  # data node (individual playlist)
│   ├── _data-component/                                   # node component (data)
│   │   ├── _current/                            # component layer
│   │   │   ├── 1996-11-10.ttl                   # playlist data
│   │   │   └── index.html                       # resource page
│   │   ├── _next/                               # component layer (draft)
│   │   │   ├── 1996-11-10.ttl                   # draft playlist data
│   │   │   └── index.html                       # resource page
│   │   └── index.html                           # resource page
│   ├── _meta-component/                                   # node component (metadata)
│   │   ├── _current/                            # component layer
│   │   │   ├── 1996-11-10_meta.trig             # playlist metadata
│   │   │   └── index.html                       # resource page
│   │   ├── _next/                               # component layer (draft)
│   │   │   ├── 1996-11-10_meta.trig             # draft playlist metadata
│   │   │   └── index.html                       # resource page
│   │   └── index.html                           # resource page
│   ├── _assets/                                 # asset tree
│   │   ├── _meta-component/                               # node component (metadata)
│   │   │   ├── _current/                        # component layer
│   │   │   │   ├── 1996-11-10_assets.trig       # asset metadata
│   │   │   │   └── index.html                   # resource page
│   │   │   ├── _next/                           # component layer (draft)
│   │   │   │   ├── 1996-11-10_assets.trig       # draft asset metadata
│   │   │   │   └── index.html                   # resource page
│   │   │   └── index.html                       # resource page
│   │   ├── cover-photo.jpg
│   │   └── index.html                           # resource page
│   ├── 1996-11-10.ttl                           # aggregated distribution
│   └── index.html                               # resource page
└── 1996-11-17/                                  # data node (another playlist)
    ├── _data-component/                                   # node component (data)
    │   ├── _current/                            # component layer
    │   │   ├── 1996-11-17.ttl
    │   │   └── index.html                       # resource page
    │   ├── _next/                               # component layer (draft)
    │   │   ├── 1996-11-17.ttl
    │   │   └── index.html                       # resource page
    │   └── index.html                           # resource page
    ├── _meta-component/                                   # node component (metadata)
    │   ├── _current/                            # component layer
    │   │   ├── 1996-11-17_meta.trig
    │   │   └── index.html                       # resource page
    │   ├── _next/                               # component layer (draft)
    │   │   ├── 1996-11-17_meta.trig
    │   │   └── index.html                       # resource page
    │   └── index.html                           # resource page
    ├── 1996-11-17.ttl                           # aggregated distribution
    └── index.html                               # resource page

```