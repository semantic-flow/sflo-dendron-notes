---
id: fnz4d2mluhnv6qtqjj0lo4y
title: Design
desc: ''
updated: 1752105211411
created: 1752104846944
---



## **Implementation Path**

1. **Start with N3.js in-memory store** (.8)
2. **Build config resolution with SPARQL queries**
3. **Add file watching and cache invalidation**
4. **Benchmark with realistic mesh sizes**
5. **Add persistence later only if startup time becomes problematic**

## **Memory Usage Considerations** (.7)

For most mesh sizes, config data should be tiny:
- Text-based JSON-LD configs
- Minimal RDF triples for structure
- Resolved configs are small objects

Even large meshes (thousands of nodes) would likely fit comfortably in memory.
