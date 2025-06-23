---
id: s5ild34tbe4w2wt4m8ldllg
title: Example Mesh Hierarchy
desc: ''
updated: 1750643130757
created: 1750640592487
---
```file
/ns/
├── _id/          
│   └── ns_id.trig          
│       # metadata about the namespace           itself (e.g. backlinks to contained things)
│          
└── djradon/          
    ├── _assets/          
    ├── _id/          
    │   └── djradon_id.trig                      # author, published url, labels, comments, backlinks for the “djradon” URI
    │                 
    ├── _ref/          
    │   └── djradon_ref.jsonld                   #  human-readable description of the referent; disambiguation
    │                 
    ├── _x/                                      # file tree
    │          
    ├── bio/                                     # unversioned dataset
    │   ├── _id/          
    │   │   └── bio_id.trig                      # metadata + backlinks for the “bio” dataset
    │   │                 
    │   └── bio.trig                             # single distribution
    │          
    ├── picks/                                   # system-versioned dataset
    │   ├── _id/          
    │   │   └── picks_id.trig                    # metadata + backlinks for the “picks” dataset
    │   │                 
    │   ├── picks.trig                           # “current” distribution
    │   └── _v/          
    │       ├── picks_v1/          
    │       │   └── picks_v1.trig                # version 1
    │       └── picks_v2/          
    │           └── picks_v2.trig                # version 2
    │          
    └── playlists/                               # contained (user) series
        ├── _id/          
        │   └── playlists_id.trig                # metadata + backlinks for the “playlists” series  
        │                 
        ├── _ref/          
        │   └── playlists_ref.jsonld             # description of the series itself
        │          
        ├── 1996-11-10/
        │   ├── _assets/
        │   │   └── photo.jpg
        │   ├── _id/
        │   │   └── 1996-11-10_id.trig
        │   └── 1996-11-10.trig
        │
        └── 1996-11-17/
            ├── _id/
            │   └── 1996-11-17_id.trig
            └── 1996-11-17.trig
```