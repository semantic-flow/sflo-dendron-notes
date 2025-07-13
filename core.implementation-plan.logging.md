---
id: bg53f29iezjqt92curqx33a
title: Logging
desc: ''
updated: 1752429103309
created: 1752427701148
---
# Implement Shared Three-Tier Logging System

## Overview
Create a comprehensive logging system in `flow-core` that can be shared across flow-service, flow-cli, and flow-web. The system supports three output channels: console, file, and Sentry. Each channel should have independently configurable log levels.

## Requirements

### Log Levels
Define standard log levels:
- `debug`: Detailed debugging information
- `info`: General information messages
- `warn`: Warning messages for potential issues
- `error`: Error messages for failures

### Three Logging Channels

#### 1. Console Logging
- Output to stdout/stderr using console.log/error
- Configurable via `CONSOLE_LOG_LEVEL` environment variable
- Default level: `info` in production, `debug` in development

#### 2. File Logging
- Write to local log file with rotation
- Configurable via `FILE_LOG_ENABLED`, `FILE_LOG_LEVEL`, `FILE_LOG_PATH`
- Default path: `./logs/service.log`
- Include timestamps and structured format

#### 3. Sentry Logging
- Send to Sentry when enabled
- Use Sentry Logs (beta) for log messages
- Use Sentry Errors for actual Error objects
- Configurable via `SENTRY_ENABLED`, `SENTRY_LOG_LEVEL`, `SENTRY_DSN`

### Configuration Interface

```typescript
interface LoggingConfig {
  console?: {
    enabled: boolean;
    level: LogLevel;
  };
  file?: {
    enabled: boolean;
    level: LogLevel;
    path: string;
  };
  sentry?: {
    enabled: boolean;
    level: LogLevel;
    dsn?: string;
  };
}
```

## Logger Interface

```typescript
interface Logger {
  debug(message: string, context?: Record<string, any>): Promise<void>;
  info(message: string, context?: Record<string, any>): Promise<void>;
  warn(message: string, error?: Error, context?: Record<string, any>): Promise<void>;
  error(message: string, error?: Error, context?: Record<string, any>): Promise<void>;
}
```

## Implementation Details

### Environment Variables
```env
# Console logging
CONSOLE_LOG_LEVEL=debug

# File logging
FILE_LOG_ENABLED=true
FILE_LOG_LEVEL=info
FILE_LOG_PATH=./logs/service.log

# Sentry logging
SENTRY_ENABLED=true
SENTRY_LOG_LEVEL=warn
SENTRY_DSN=your-sentry-dsn
```

### Sentry Integration
- For messages without Error objects: Use Sentry Logs (addBreadcrumb with log category)
- For messages with Error objects: Use Sentry captureException with message in extra context
- Map log levels: debug→debug, info→info, warn→warning, error→error

### File Logging
- Create logs directory if it doesn't exist
- Use structured format with timestamps
- Consider basic log rotation (optional)
- Async file writes to avoid blocking

### Level Filtering
Each channel should only log messages at or above its configured level:
- debug < info < warn < error
- If channel level is "warn", only log warn and error messages

## Usage Examples

```typescript
// Basic usage
await logger.info("Server started on port 8000");
await logger.warn("Config file not found, using defaults");
await logger.error("Failed to connect to database", dbError);

// With context
await logger.info("User authenticated", { userId: "123", method: "oauth" });
await logger.error("Validation failed", validationError, { input: userInput });
```

## File Structure
Create the logging system in `flow-core/src/logging/` with:
- `types.ts` - LogLevel enum and interfaces
- `config.ts` - Configuration schema and defaults
- `loggers/` - Individual logger implementations
  - `console-logger.ts` - Console output (no dependencies)
  - `file-logger.ts` - File output (uses Deno file APIs)
  - `sentry-logger.ts` - Sentry integration (optional dependency)
- `logger.ts` - Main Logger class that coordinates all channels
- `factory.ts` - createLogger function for different configurations
- `index.ts` - Export public interface

## Package Structure

### flow-core/src/logging/
Core logging infrastructure with layered dependencies:

```typescript
// Base interfaces (no external dependencies)
export interface Logger {
  debug(message: string, context?: Record<string, any>): Promise<void>;
  info(message: string, context?: Record<string, any>): Promise<void>;
  warn(message: string, error?: Error, context?: Record<string, any>): Promise<void>;
  error(message: string, error?: Error, context?: Record<string, any>): Promise<void>;
}

// Factory function with optional channels
export function createLogger(config: LoggingConfig): Logger;
```

### Per-Application Usage

#### flow-service (full logging)
```typescript
import { createLogger } from "@flow/core/logging";

const logger = createLogger({
  console: { enabled: true, level: "info" },
  file: { enabled: true, level: "warn", path: "./logs/service.log" },
  sentry: { enabled: true, level: "error", dsn: process.env.SENTRY_DSN }
});
```

#### flow-cli (console only)
```typescript
import { createLogger } from "@flow/core/logging";

const logger = createLogger({
  console: { enabled: true, level: "info" }
  // No file/sentry needed for CLI
});
```

#### flow-web (console + sentry)
```typescript
import { createLogger } from "@flow/core/logging";

const logger = createLogger({
  console: { enabled: true, level: "debug" },
  sentry: { enabled: true, level: "warn", dsn: process.env.SENTRY_DSN }
  // No file logging in browser
});
```

## Integration
- Create logging configuration in each application's startup
- Use createLogger factory with app-specific config
- Export logger instance for use throughout each application
- Each app can extend with app-specific loggers if needed

## Dependency Management
- Core interfaces have no external dependencies
- File logger uses only Deno standard library
- Sentry logger imports Sentry SDK (optional)
- Factory function only imports needed loggers based on config

## Error Handling
- File logging failures should not crash the application
- Sentry failures should not crash the application
- Always ensure console logging works as fallback
- Log configuration errors to console during logger creation
- Graceful degradation when optional dependencies unavailable
