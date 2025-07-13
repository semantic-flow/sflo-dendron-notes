---
id: 9rrjnwvkqecnckbs22iyuzh
title: Logging
desc: ''
updated: 1752385127790
created: 1752384875012
---

# Development (colorized logs, debug level)
$env:DENO_ENV = "development"
$env:LOG_LEVEL = "DEBUG"

# Production (Sentry logs enabled)
$env:DENO_ENV = "production"
$env:SENTRY_DSN = "https://5b31ce060f1ccbf2d8cd83efa37f33e8@o4509659529150464.ingest.us.sentry.io/4509659530592256"
