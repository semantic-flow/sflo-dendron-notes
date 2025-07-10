---
id: ijaxksk04u89zixwae76mnk
title: File Watch
desc: ''
updated: 1752104919404
created: 1752104614421
---

## **File Watching Simplification** (.8)

Since everything's in memory:
- File changes just invalidate affected cache entries
- No complex database update logic
- Easy to rebuild affected subtrees

```typescript
// On .flow-config.jsonld change
onConfigFileChange(filePath: string) {
  // 1. Remove affected configs from cache
  this.invalidateConfigSubtree(filePath);
  
  // 2. Update RDF store
  this.updateConfigTriples(filePath);
  
  // 3. Configs will be lazily recomputed on next request
}
```


## **Service Lifecycle** (.8)

```typescript
// Simple lifecycle management
class FlowService {
  async start(meshRoot: string) {
    console.log("Scanning mesh structure...");
    await this.configService.initialize(meshRoot);
    
    console.log("Starting API server...");
    await this.startApiServer();
    
    console.log("Flow service ready");
  }
  
  async stop() {
    this.stopWatchers();
    this.stopApiServer();
    // Memory automatically freed
  }
}
```

