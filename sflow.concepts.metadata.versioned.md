---
id: mn0de58rs14u2majc0z1kcb
title: Versioned
desc: ''
updated: 1750742045953
created: 1750731946890
---

If a metadata dataset gets updated, it might make sense to start versioning it, i.e., with a versioned dataset.


But to avoid an infinite loop with versioned datasets of metadata, _id and _ref versioned-datasets (v1, v2, etc)and their containing series (_v) do not have _id or _ref of their own.  

```file
my-thing/
├─ _id/
│   ├─ my-thing_id.trig           ← working identity metadata
│   └─ _v/                        ← version series (no own metadata)
│       └─ v1/
│           └─ my-thing_id.trig   ← v1 snapshot of my-thing identity
├─ _ref/
│   ├─ my-thing_ref.trig          ← working referent metadata
│   └─ _v/                        ← version series (no own metadata)
│       └─ v1/
│           └─ my-thing_ref.trig  ← snapshot of v1 referent
└─ … other data or child resources
```

As always with datasets, metadata dataset versioning is configurable. By default, it should be off.
