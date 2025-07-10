---
id: gwda0y5zp7jrz8w3hlylwxm
title: In Memory
desc: ''
updated: 1752104948354
created: 1752104846937
---

If the service needs to re-scan the filesystem on startup anyway to ensure consistency, persistence just adds complexity without much benefit.

### **Options for In-Memory Graph Storage** (.8)

**N3.js Store** (.9)
- Pure TypeScript, perfect for Deno
- Excellent SPARQL support
- No FFI or WASM complexity
- Fast in-memory operations

**RDF/JS Dataset** (.7)
- Standard interface, multiple implementations
- Could switch backends easily
- Good ecosystem support

**Custom Graph Structure** (.6)
- Optimized specifically for config inheritance
- Simpler than full RDF if you don't need SPARQL complexity

