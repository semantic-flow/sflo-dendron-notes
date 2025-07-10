---
id: tsnodsj6rd7v6sa1ggh7l53
title: Learning from Weave
desc: ''
updated: 1752100486722
created: 1752100445900
---

Confusingly, I was working on another CLI and it was named "weave", see ../weave

## Key Patterns

1. Clean CLI/Core Separation (.9)

src/cli/: Command definitions and option parsing
src/core/: Business logic and operations
CLI commands are thin wrappers that delegate to core functions

2. Dependency Management (.8)

Centralized deps in src/deps/ with re-exports
Path mapping with @/ for clean imports
JSR packages for Cliffy (more modern than deno.land/x)

3. Configuration Architecture (.9)

Input vs Resolved types - separate interfaces for user input vs processed config
Global action pattern - main command handles global config setup
Config composition - merges defaults, files, and CLI options

4. Error Handling & Logging (.8)

Centralized error handling with handleCaughtError
Structured logging with configurable levels
Graceful degradation with fallbacks