---
id: bg53f29iezjqt92curqx33a
title: Logging
desc: ''
updated: 1752509668035
created: 1752427701148
---
# Enhanced Structured Logging Implementation Plan

## Overview

Implement a comprehensive structured logging system for Semantic Flow, building on the existing sophisticated JSON-LD configuration system while adding structured data support, complete file logging, and shared infrastructure across flow-service, flow-cli, and flow-web.

## Current State Analysis

### Existing Implementation Strengths
- **✅ Sophisticated JSON-LD configuration system** with ontology-based structure
- **✅ Three-tier logging configured**: Console, File, and Sentry channels
- **✅ Sentry integration** fully implemented and working
- **✅ Console logging** with colors, formatting, and environment-based levels
- **✅ Configuration cascading**: CLI → Environment → Config file → Defaults
- **✅ ServiceConfigAccessor** for typed configuration access

### Current Gaps
- **❌ File logging** configured but NOT implemented (no actual file writing)
- **❌ No structured logging** - all messages are plain strings
- **❌ No log rotation, retention, or directory creation**
- **❌ No shared infrastructure** for flow-core (CLI/web need logging too)

## Structured Logging Design

### Core Structured Interface

```typescript
// Base structured context for all log entries
interface LogContext {
  // Operation tracking
  operation?: 'scan' | 'weave' | 'validate' | 'config-resolve' | 'api-request' | 'startup';
  requestId?: string;

  // Mesh/Node context
  meshPath?: string;
  nodePath?: string;
  nodeType?: 'data' | 'namespace' | 'reference';

  // Performance metrics
  duration?: number;
  startTime?: number;
  nodeCount?: number;
  fileCount?: number;

  // Configuration context
  configSource?: 'file' | 'inheritance' | 'defaults' | 'api' | 'environment';
  schemaVersion?: string;

  // Error context
  errorCode?: string;
  violations?: string[];
  cause?: string;

  // API context
  endpoint?: string;
  statusCode?: number;
  userAgent?: string;

  // Service context
  component?: 'mesh-scanner' | 'api-handler' | 'config-resolver' | 'weave-processor';

  // Arbitrary additional context
  [key: string]: unknown;
}

// Structured-first logger interface
interface StructuredLogger {
  debug(message: string, context?: LogContext): Promise<void>;
  info(message: string, context?: LogContext): Promise<void>;
  warn(message: string, context?: LogContext): Promise<void>;
  error(message: string, error?: Error, context?: LogContext): Promise<void>;

  // Contextual logger factories
  withContext(baseContext: LogContext): StructuredLogger;
}
```

### Usage Examples

```typescript
// Mesh operations with rich context
await logger.info('Mesh scan started', {
  operation: 'scan',
  component: 'mesh-scanner',
  meshPath: '/research/datasets',
  requestId: crypto.randomUUID()
});

await logger.info('Node processed', {
  operation: 'weave',
  component: 'weave-processor',
  nodePath: '/research/datasets/experiment-1',
  nodeType: 'data',
  configSource: 'inheritance',
  duration: 150,
  nodeCount: 1
});

// API operations
await logger.info('Config updated', {
  operation: 'api-request',
  component: 'api-handler',
  endpoint: 'PUT /api/mesh/{meshPath}/config',
  statusCode: 200,
  meshPath: '/research/datasets',
  duration: 850
});

// Error handling with full context
await logger.error('SHACL validation failed', validationError, {
  operation: 'config-resolve',
  component: 'config-resolver',
  nodePath: '/research/datasets/experiment-1',
  violations: ['node:versioningEnabled must be boolean'],
  schemaVersion: '1.0.0',
  errorCode: 'SHACL_VALIDATION_FAILED',
  configSource: 'file'
});

// Contextual loggers
const meshLogger = logger.withContext({
  component: 'mesh-scanner',
  operation: 'scan'
});
await meshLogger.info('Processing node', {
  nodePath: '/data/experiment-1',
  nodeType: 'data'
});
```

## Enhanced Configuration Schema

### Updated JSON-LD Configuration

```typescript
interface EnhancedLogChannelConfig {
  readonly "@type": "fsvc:LogChannelConfig";
  readonly "fsvc:logChannelEnabled": boolean;
  readonly "fsvc:logLevel": "debug" | "info" | "warn" | "error";
  readonly "fsvc:logFormat": "json" | "pretty" | "compact";

  // File channel specific
  readonly "fsvc:logFilePath"?: string;
  readonly "fsvc:maxFileSize"?: string;     // e.g., "10MB"
  readonly "fsvc:maxFiles"?: number;       // rotation count
  readonly "fsvc:retentionDays"?: number;  // auto-cleanup
  readonly "fsvc:rotationInterval"?: "daily" | "weekly" | "size";

  // Sentry channel specific
  readonly "fsvc:sentryDsn"?: string;
  readonly "fsvc:separateErrorReporting"?: boolean;
  readonly "fsvc:sampleRate"?: number;
}
```

### Environment Variables

```env
# Existing (already implemented)
FLOW_SENTRY_ENABLED=true
FLOW_SENTRY_DSN=https://...
FLOW_FILE_LOG_ENABLED=true
FLOW_FILE_LOG_LEVEL=warn

# New structured logging options
FLOW_CONSOLE_LOG_FORMAT=pretty    # pretty | json | compact
FLOW_FILE_LOG_FORMAT=json         # json | pretty | compact
FLOW_FILE_LOG_MAX_SIZE=10MB
FLOW_FILE_LOG_MAX_FILES=5
FLOW_FILE_LOG_RETENTION_DAYS=30
FLOW_FILE_LOG_ROTATION=daily

# Sentry structured logging
FLOW_SENTRY_STRUCTURED=true       # Use Sentry's structured logging API
FLOW_SENTRY_SAMPLE_RATE=1.0
```

### Default Configuration

```jsonld
{
  "@context": { "fsvc": "https://semantic-flow.github.io/ontology/flow-service/" },
  "fsvc:hasLoggingConfig": {
    "@type": "fsvc:LoggingConfig",
    "fsvc:hasConsoleChannel": {
      "@type": "fsvc:LogChannelConfig",
      "fsvc:logChannelEnabled": true,
      "fsvc:logLevel": "info",
      "fsvc:logFormat": "pretty"
    },
    "fsvc:hasFileChannel": {
      "@type": "fsvc:LogChannelConfig",
      "fsvc:logChannelEnabled": true,
      "fsvc:logLevel": "warn",
      "fsvc:logFormat": "json",
      "fsvc:logFilePath": "./logs/flow-service.log",
      "fsvc:maxFileSize": "10MB",
      "fsvc:maxFiles": 5,
      "fsvc:retentionDays": 30,
      "fsvc:rotationInterval": "daily"
    },
    "fsvc:hasSentryChannel": {
      "@type": "fsvc:LogChannelConfig",
      "fsvc:logChannelEnabled": false,
      "fsvc:logLevel": "error",
      "fsvc:logFormat": "json",
      "fsvc:separateErrorReporting": false,
      "fsvc:sampleRate": 1.0
    }
  }
}
```

## Implementation Architecture

### File Structure

```
flow-core/
├── src/
│   ├── logging/
│   │   ├── types.ts              # StructuredLogger interface, LogContext
│   │   ├── channels/             # Channel implementations
│   │   │   ├── console-channel.ts  # Pretty/JSON console output
│   │   │   ├── file-channel.ts     # File writing with rotation
│   │   │   └── sentry-channel.ts   # Sentry structured logging
│   │   ├── formatters/           # Format structured data for output
│   │   │   ├── pretty-formatter.ts # Human-readable format
│   │   │   ├── json-formatter.ts   # Structured JSON format
│   │   │   └── compact-formatter.ts # Single-line format
│   │   ├── logger.ts             # Main StructuredLogger implementation
│   │   ├── factory.ts            # createLogger(config) factory
│   │   └── index.ts              # Public exports
│   └── validation/               # Shared validation utilities

flow-service/
├── src/
│   ├── config/                   # Keep existing config system
│   ├── utils/
│   │   └── logger.ts            # Flow-service specific logger setup
│   └── app.ts
```

### Channel Architecture

```typescript
// Base channel interface
interface LogChannel {
  name: string;
  enabled: boolean;
  level: LogLevel;
  format: LogFormat;

  log(level: LogLevel, message: string, context: LogContext, error?: Error): Promise<void>;
  close?(): Promise<void>;
}

// Console channel with formatting
class ConsoleChannel implements LogChannel {
  constructor(
    private config: LogChannelConfig,
    private formatter: LogFormatter
  ) {}

  async log(level: LogLevel, message: string, context: LogContext): Promise<void> {
    const formatted = this.formatter.format(level, message, context);
    console[level](formatted);
  }
}

// File channel with rotation
class FileChannel implements LogChannel {
  private rotationManager: LogRotationManager;

  async log(level: LogLevel, message: string, context: LogContext): Promise<void> {
    await this.rotationManager.rotateIfNeeded();
    const formatted = this.formatter.format(level, message, context);
    await this.writeToFile(formatted);
  }
}

// Sentry channel with structured logging
class SentryChannel implements LogChannel {
  async log(level: LogLevel, message: string, context: LogContext, error?: Error): Promise<void> {
    if (error) {
      Sentry.captureException(error, {
        level: this.mapLevel(level),
        extra: context,
        tags: { component: context.component, operation: context.operation }
      });
    } else {
      // Use Sentry's new structured logging
      Sentry[level](message, context);
    }
  }
}
```

### Formatting System

```typescript
interface LogFormatter {
  format(level: LogLevel, message: string, context: LogContext, error?: Error): string;
}

// Pretty formatter for console (development)
class PrettyFormatter implements LogFormatter {
  format(level: LogLevel, message: string, context: LogContext): string {
    const timestamp = new Date().toISOString();
    const contextStr = this.formatContext(context);
    return `${timestamp} [${level.toUpperCase()}] ${message}${contextStr}`;
  }

  private formatContext(context: LogContext): string {
    if (!context || Object.keys(context).length === 0) return '';

    const parts = [];
    if (context.operation) parts.push(`op=${context.operation}`);
    if (context.meshPath) parts.push(`mesh=${context.meshPath}`);
    if (context.duration) parts.push(`${context.duration}ms`);
    if (context.nodeCount) parts.push(`${context.nodeCount} nodes`);

    return parts.length > 0 ? ` (${parts.join(', ')})` : '';
  }
}

// JSON formatter for file/analysis
class JsonFormatter implements LogFormatter {
  format(level: LogLevel, message: string, context: LogContext, error?: Error): string {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      service: 'flow-service',
      version: Deno.env.get('FLOW_VERSION'),
      environment: Deno.env.get('DENO_ENV'),
      ...context
    };

    if (error) {
      logEntry.error = {
        name: error.name,
        message: error.message,
        stack: error.stack
      };
    }

    return JSON.stringify(logEntry);
  }
}
```

## Implementation Phases

### Phase 1: Core Structured Infrastructure (flow-core)
**Priority: High** - Foundation for all applications

#### 1.1 Create Core Types and Interfaces
```typescript
// flow-core/src/logging/types.ts
export interface LogContext { /* ... */ }
export interface StructuredLogger { /* ... */ }
export interface LogChannel { /* ... */ }
export type LogLevel = 'debug' | 'info' | 'warn' | 'error';
export type LogFormat = 'json' | 'pretty' | 'compact';
```

#### 1.2 Implement Formatters
- **PrettyFormatter**: Human-readable for development
- **JsonFormatter**: Structured for analysis
- **CompactFormatter**: Single-line for production

#### 1.3 Implement Base Channels
- **ConsoleChannel**: Output to stdout/stderr with formatting
- **FileChannel**: Write to files with rotation support
- **SentryChannel**: Integrate with Sentry structured logging

#### 1.4 Create Logger Factory
```typescript
export function createLogger(config: LoggingConfig): StructuredLogger;
export function createConsoleLogger(level?: LogLevel): StructuredLogger;
```

### Phase 2: File Logging Implementation
**Priority: High** - Complete the missing file logging functionality

#### 2.1 Log Rotation Manager
```typescript
class LogRotationManager {
  constructor(private config: FileRotationConfig) {}

  async rotateIfNeeded(filePath: string): Promise<void> {
    const shouldRotate = await this.checkRotationConditions(filePath);
    if (shouldRotate) {
      await this.performRotation(filePath);
      await this.cleanupOldLogs(filePath);
    }
  }

  private async checkRotationConditions(filePath: string): Promise<boolean> {
    // Check file size, time-based rotation
  }

  private async performRotation(filePath: string): Promise<void> {
    // Archive current log file
  }

  private async cleanupOldLogs(filePath: string): Promise<void> {
    // Remove logs older than retention period
  }
}
```

#### 2.2 Async File Writer
```typescript
class AsyncFileWriter {
  private writeQueue: LogEntry[] = [];
  private isProcessing = false;

  async queueWrite(entry: LogEntry): Promise<void> {
    this.writeQueue.push(entry);
    if (!this.isProcessing) {
      this.processQueue();
    }
  }

  private async processQueue(): Promise<void> {
    // Batch write log entries for performance
  }
}
```

### Phase 3: Enhanced Service Integration
**Priority: Medium** - Upgrade flow-service to use structured logging

#### 3.1 Update Service Logger Setup
```typescript
// flow-service/src/utils/logger.ts
import { createLogger } from '@flow/core/logging';
import { getServiceConfig } from '../config/index.ts';

// Create structured logger from service config
const config = getServiceConfig();
const logger = createLogger(config.logging);

// Create contextual loggers for different components
export const meshLogger = logger.withContext({ component: 'mesh-scanner' });
export const apiLogger = logger.withContext({ component: 'api-handler' });
export const configLogger = logger.withContext({ component: 'config-resolver' });

export { logger };
```

#### 3.2 Update All Log Calls
Convert existing log calls to structured format:
```typescript
// Before
logger.info(`Mesh scan completed for ${meshPath}`);

// After
await logger.info('Mesh scan completed', {
  operation: 'scan',
  meshPath,
  duration: performance.now() - startTime,
  nodeCount
});
```

#### 3.3 Enhanced Error Handling
```typescript
export async function handleCaughtError(
  error: unknown,
  context: Partial<LogContext> = {},
  level: 'warn' | 'error' = 'error'
): Promise<void> {
  const errorContext: LogContext = {
    timestamp: new Date().toISOString(),
    service: 'flow-service',
    environment: Deno.env.get('DENO_ENV'),
    ...context
  };

  if (error instanceof Error) {
    await logger[level](`Error in ${context.component || 'unknown'}`, error, errorContext);
  } else {
    await logger[level]('Unknown error occurred', error, errorContext);
  }
}
```

### Phase 4: CLI and Web Integration
**Priority: Medium** - Add logging to other applications

#### 4.1 Flow-CLI Logging
```typescript
// flow-cli/src/logger.ts
import { createConsoleLogger } from '@flow/core/logging';

// Simple console-only logger for CLI
const logger = createConsoleLogger('info');

// CLI-specific context
const cliLogger = logger.withContext({ component: 'cli' });

// Usage in CLI commands
await cliLogger.info('Command executed', {
  operation: 'cli-command',
  command: 'flow init',
  args: process.argv.slice(2)
});
```

#### 4.2 Flow-Web Logging
```typescript
// flow-web/src/logger.ts
import { createLogger } from '@flow/core/logging';

// Browser-compatible logger (console + optional remote)
const logger = createLogger({
  console: { enabled: true, level: 'info', format: 'compact' },
  // No file logging in browser
  // Optional: remote logging endpoint
});

const webLogger = logger.withContext({ component: 'web-ui' });
```

## Advanced Features

### Performance Monitoring
```typescript
// Built-in performance tracking
class PerformanceLogger {
  static async time<T>(
    operation: string,
    fn: () => Promise<T>,
    context?: LogContext
  ): Promise<T> {
    const start = performance.now();
    try {
      const result = await fn();
      const duration = performance.now() - start;

      await logger.info(`${operation} completed`, {
        operation,
        duration,
        status: 'success',
        ...context
      });

      return result;
    } catch (error) {
      const duration = performance.now() - start;

      await logger.error(`${operation} failed`, error, {
        operation,
        duration,
        status: 'error',
        ...context
      });

      throw error;
    }
  }
}

// Usage
const result = await PerformanceLogger.time(
  'mesh-scan',
  () => scanMesh(meshPath),
  { meshPath, component: 'mesh-scanner' }
);
```

### Request Tracking
```typescript
// Middleware for request correlation
export function requestLoggingMiddleware() {
  return async (c: Context, next: Next) => {
    const requestId = crypto.randomUUID();
    const start = performance.now();

    // Add request context to all subsequent logs
    const requestLogger = apiLogger.withContext({
      requestId,
      endpoint: c.req.path,
      method: c.req.method,
      userAgent: c.req.header('user-agent')
    });

    c.set('logger', requestLogger);

    try {
      await next();

      const duration = performance.now() - start;
      await requestLogger.info('Request completed', {
        statusCode: c.res.status,
        duration
      });
    } catch (error) {
      const duration = performance.now() - start;
      await requestLogger.error('Request failed', error, {
        statusCode: c.res.status || 500,
        duration
      });
      throw error;
    }
  };
}
```

## Migration Strategy

### Backward Compatibility
- Keep existing logger interface working during transition
- Gradual migration of log calls to structured format
- No breaking changes to configuration system

### Rollout Plan
1. **Phase 1**: Implement flow-core logging infrastructure
2. **Phase 2**: Add file logging to existing flow-service
3. **Phase 3**: Migrate flow-service to structured logging
4. **Phase 4**: Add logging to flow-cli and flow-web
5. **Phase 5**: Advanced features (performance monitoring, etc.)

## Testing Strategy

### Unit Tests
- Formatter output validation
- Channel behavior testing
- Configuration parsing
- Error handling paths

### Integration Tests
- End-to-end logging pipeline
- File rotation and cleanup
- Sentry integration
- Performance under load

### Log Analysis
- Structured log queries
- Performance monitoring
- Error correlation
- Usage pattern analysis

## Success Criteria

### Functional Requirements
- ✅ All three channels (console, file, sentry) working with structured data
- ✅ File rotation and retention policies functional
- ✅ Shared logging infrastructure across all applications
- ✅ Rich context for mesh operations, API requests, and errors
- ✅ Performance monitoring capabilities

### Non-Functional Requirements
- ✅ No performance degradation from existing system
- ✅ Memory usage within acceptable limits
- ✅ Backward compatibility maintained
- ✅ Configuration system unchanged
- ✅ Easy to use and understand for developers

### Quality Metrics
- ✅ 90%+ test coverage for logging infrastructure
- ✅ No logging-related performance bottlenecks
- ✅ Structured logs enable effective debugging and monitoring
- ✅ Documentation and examples for all logging patterns
