---
id: 3s13kd7qv0297vpi5md7902
title: Strategy
desc: ''
updated: 1752104882276
created: 1752104577368
---

flow-service startup:
1. Scan filesystem for .flow-config.jsonld files
2. Build RDF graph in memory
3. Resolve all config inheritance chains
4. Start file watchers
5. Ready to serve requests


```typescript
// Fast startup with incremental loading
class MeshConfigService {
  private rdfStore = new N3.Store();
  private configCache = new Map<string, ResolvedConfig>();
  
  async initialize(meshRoot: string) {
    // 1. Quick scan for structure
    const nodes = await this.scanMeshStructure(meshRoot);
    
    // 2. Load configs lazily or in background
    await this.loadConfigsIncremental(nodes);
    
    // 3. Start file watchers
    this.startWatchers(meshRoot);
  }
  
  async resolveConfig(nodePath: string): Promise<ResolvedConfig> {
    // Check cache first, compute if needed
    if (!this.configCache.has(nodePath)) {
      this.configCache.set(nodePath, await this.computeConfig(nodePath));
    }
    return this.configCache.get(nodePath)!;
  }
}
```
