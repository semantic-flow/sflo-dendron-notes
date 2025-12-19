---
id: q25qyq8qgjgvgw9sxlcxjwt
title: Caching
desc: ''
updated: 1766170595443
created: 1752131799717
---

[[dave: claude jackpot!]]

## **HTMX Caching Behavior**

### **Default Behavior: No Automatic Caching** (.7)
By default, HTMX makes a fresh request each time:

```html
<!-- This will hit the server every time the button is clicked -->
<button hx-get="../fragments/config-panel.html" 
        hx-target="#main-content">Load Config</button>
```

### **Browser HTTP Caching** (.9)
HTMX respects standard HTTP caching headers:

```http
<!-- Server response headers -->
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Wed, 10 Jul 2025 10:00:00 GMT
```

Since you're generating static fragments, the web server (GitHub Pages, etc.) can set appropriate cache headers (.8).

## **HTMX-Specific Caching Solutions**

### **1. Request Caching Extension** (.9)
```html
<script src="https://unpkg.com/htmx.org@1.9.10/dist/ext/response-targets.js"></script>

<!-- Cache responses for this session -->
<div hx-ext="response-targets">
  <button hx-get="../fragments/config-panel.html" 
          hx-target="#main-content"
          hx-cache="true">Load Config</button>
</div>
```

### **2. Manual Content Preservation** (.8)
```html
<script>
// Keep content in DOM and show/hide
document.body.addEventListener('htmx:afterSwap', function(evt) {
  // Store swapped content
  const target = evt.detail.target;
  const content = target.innerHTML;
  
  // Store in a cache object or hidden div
  contentCache[evt.detail.requestConfig.path] = content;
});

// Check cache before making request
document.body.addEventListener('htmx:beforeRequest', function(evt) {
  const path = evt.detail.requestConfig.path;
  if (contentCache[path]) {
    evt.preventDefault();
    evt.detail.target.innerHTML = contentCache[path];
  }
});
</script>
```

### **3. Tab-Style Content Switching** (.9)
Better UX pattern - load once, then just show/hide:

```html
<div class="mesh-node-page">
  <nav class="tabs">
    <button hx-get="../fragments/config-panel.html" 
            hx-target="#config-content"
            hx-swap="innerHTML"
            data-tab="config"
            class="tab-button">Configuration</button>
            
    <button hx-get="../fragments/children-list.html" 
            hx-target="#children-content"
            hx-swap="innerHTML" 
            data-tab="children"
            class="tab-button">Children</button>
  </nav>
  
  <main class="tab-content">
    <div id="config-content" class="tab-panel" style="display: none;"></div>
    <div id="children-content" class="tab-panel" style="display: none;"></div>
  </main>
</div>

<script>
// Enhanced tab switching with caching
document.querySelectorAll('.tab-button').forEach(button => {
  button.addEventListener('click', function(e) {
    const tabName = this.dataset.tab;
    const target = document.getElementById(`${tabName}-content`);
    
    // If content already loaded, just show it
    if (target.innerHTML.trim()) {
      showTab(tabName);
      e.preventDefault();
      return false;
    }
    
    // Otherwise, let HTMX load it, then show
    this.addEventListener('htmx:afterSwap', () => showTab(tabName), { once: true });
  });
});

function showTab(tabName) {
  // Hide all panels
  document.querySelectorAll('.tab-panel').forEach(panel => {
    panel.style.display = 'none';
  });
  
  // Show selected panel
  document.getElementById(`${tabName}-content`).style.display = 'block';
  
  // Update active tab
  document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
  document.querySelector(`[data-tab="${tabName}"]`).classList.add('active');
}
</script>
```

## **Static Site Caching Advantages** (.9)

Since you're generating static fragments, you get excellent caching for free:

### **CDN/Browser Caching**
```
# Generated during weave process
mesh-node/_assets/fragments/config-panel.html
mesh-node/_assets/fragments/children-list.html

# Server sets cache headers
Cache-Control: max-age=86400, public  # 24 hours
```

### **Aggressive Caching Strategy**
```html
<!-- Cache aggressively since content only changes on weave -->
<meta http-equiv="Cache-Control" content="max-age=86400, public">

<!-- Or version-based URLs for cache busting -->
<button hx-get="../fragments/config-panel.html?v=abc123" 
        hx-target="#main-content">Load Config</button>
```

## **Optimal Pattern for Your Use Case** (.9)

Since mesh content only changes during weave operations:

### **1. Load-Once Pattern**
```html
<div class="mesh-interface" hx-ext="cache">
  <nav class="tabs">
    <button onclick="showCachedTab('config')" 
            data-load-url="../fragments/config-panel.html"
            data-target="#config-content">Config</button>
    <button onclick="showCachedTab('children')"
            data-load-url="../fragments/children-list.html" 
            data-target="#children-content">Children</button>
  </nav>
  
  <main>
    <div id="config-content" class="tab-panel"></div>
    <div id="children-content" class="tab-panel"></div>
  </main>
</div>

<script>
const loadedTabs = new Set();

async function showCachedTab(tabName) {
  const panel = document.getElementById(`${tabName}-content`);
  
  // Load content if not already loaded
  if (!loadedTabs.has(tabName)) {
    const button = document.querySelector(`[onclick="showCachedTab('${tabName}')"]`);
    const url = button.dataset.loadUrl;
    
    const response = await fetch(url);
    panel.innerHTML = await response.text();
    loadedTabs.add(tabName);
  }
  
  // Show the tab
  showTab(tabName);
}
</script>
```

### **2. Cache Invalidation on Weave**
```html
<!-- Include weave timestamp for cache busting -->
<script>
const meshVersion = "{mesh.lastWeaveTimestamp}"; // Injected during generation

// Add version to all HTMX requests
document.body.addEventListener('htmx:configRequest', function(evt) {
  if (!evt.detail.parameters.v) {
    evt.detail.parameters.v = meshVersion;
  }
});
</script>
```

**Bottom line:** HTMX doesn't cache by default, but you can implement effective caching strategies (.9). For static fragments that only change on weave, the combination of HTTP caching + load-once patterns gives you excellent performance.