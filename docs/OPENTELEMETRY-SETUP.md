# OpenTelemetry Integration with Azure Application Insights

This document explains how OpenTelemetry is configured in this application to send telemetry data to Azure Application Insights.

## Overview

OpenTelemetry provides vendor-neutral instrumentation for collecting traces, metrics, and logs. This application uses the Azure Monitor OpenTelemetry Distro to automatically instrument Node.js server code and send telemetry to Application Insights.

## Architecture

```
┌─────────────────────┐
│   Next.js App       │
│  (Server Runtime)   │
└──────────┬──────────┘
           │
           │ Auto-instrumentation
           │
┌──────────▼──────────┐
│  OpenTelemetry SDK  │
│  (@azure/monitor-   │
│   opentelemetry)    │
└──────────┬──────────┘
           │
           │ OTLP Protocol
           │
┌──────────▼──────────┐
│ Application Insights│
│   (Azure Monitor)   │
└─────────────────────┘
```

## What Gets Instrumented Automatically

The Azure Monitor OpenTelemetry Distro automatically instruments:

### 1. **HTTP/HTTPS Requests**
- Incoming requests to your Next.js API routes
- Outgoing HTTP/HTTPS calls to external APIs (Microsoft Graph, Azure, Power Platform, etc.)
- Request duration, status codes, URLs

### 2. **Database Queries**
- Any database calls (if you add a database)
- Query performance and errors

### 3. **Dependencies**
- External service calls
- Third-party API calls
- Redis, MongoDB, PostgreSQL, etc. (if configured)

### 4. **Exceptions**
- Unhandled exceptions
- Error stack traces
- Exception types and messages

### 5. **Performance Metrics**
- Request latency
- Memory usage
- CPU usage
- Custom metrics (if added)

## Files Added/Modified

### 1. `instrumentation.ts` (New)
- **Location**: Root directory
- **Purpose**: Configures OpenTelemetry before the application starts
- **When it runs**: Before any Next.js code executes (Next.js 13.4+ feature)
- **Key configuration**:
  - Reads `APPLICATIONINSIGHTS_CONNECTION_STRING` from environment
  - Sets service name to `pulse-360-portal`
  - Enables auto-instrumentation for HTTP and other modules

### 2. `next.config.ts` (Modified)
- **Added**: `experimental.instrumentationHook: true`
- **Purpose**: Enables Next.js to load the instrumentation file

### 3. `.env.local` (Modified)
- **Added**: `APPLICATIONINSIGHTS_CONNECTION_STRING` documentation
- **Purpose**: Stores the Application Insights connection string

### 4. `package.json` (Modified)
- **Added packages**:
  - `@azure/monitor-opentelemetry` - Azure Monitor integration
  - `@opentelemetry/api` - OpenTelemetry core API
  - `@opentelemetry/sdk-metrics` - Metrics SDK
  - `@opentelemetry/resources` - Resource definitions
  - `@opentelemetry/semantic-conventions` - Standard attributes
  - `@opentelemetry/sdk-trace-base` - Tracing SDK

## Setup Instructions

### 1. Get Your Connection String

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to your Application Insights resource (or create one)
3. Go to **Overview** tab
4. Copy the **Connection String** (not Instrumentation Key)
5. Format: `InstrumentationKey=<key>;IngestionEndpoint=https://<region>.in.applicationinsights.azure.com/;LiveEndpoint=https://<region>.livediagnostics.monitor.azure.com/`

### 2. Configure Environment Variable

Add to your `.env.local`:

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=<your-connection-string>
```

**Example**:
```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=12345678-1234-1234-1234-123456789012;IngestionEndpoint=https://eastus-8.in.applicationinsights.azure.com/;LiveEndpoint=https://eastus.livediagnostics.monitor.azure.com/
```

### 3. Restart Your Application

```bash
npm run dev
```

You should see in the console:
```
[OpenTelemetry] Azure Monitor OpenTelemetry initialized successfully
[OpenTelemetry] Service Name: pulse-360-portal
[OpenTelemetry] Connection String: InstrumentationKey=...
```

### 4. Verify Data in Application Insights

1. Wait 2-5 minutes for data to appear
2. Go to Azure Portal > Application Insights
3. Check:
   - **Application Map**: See service dependencies
   - **Performance**: View request durations
   - **Failures**: See exceptions and errors
   - **Live Metrics**: Real-time telemetry stream

## What Data is Collected

### Traces (Distributed Tracing)
Every API request creates a trace with:
- **Request URL**: `/api/power-platform/environments`
- **Duration**: How long it took
- **Status Code**: 200, 404, 500, etc.
- **Dependencies**: External API calls made during the request
- **Parent/Child relationships**: End-to-end request flow

**Example**: 
```
User Request → Next.js API Route → Microsoft Graph API → Response
     ↓              ↓                      ↓
   Trace        Span (child)          Dependency Span
```

### Metrics
- **Request rate**: Requests per second
- **Response time**: Average, P95, P99
- **Failure rate**: Percentage of failed requests
- **Memory usage**: Heap size, RSS
- **Custom metrics**: (If you add them)

### Logs
- Console logs (if configured)
- Exception logs
- Warning and error logs

### Exceptions
- **Type**: Error, TypeError, ReferenceError
- **Message**: Error description
- **Stack trace**: Full call stack
- **Context**: Request that caused the error

## Advanced Configuration

### Custom Spans (Manual Instrumentation)

Add custom spans to track specific operations:

```typescript
import { trace } from '@opentelemetry/api'

const tracer = trace.getTracer('my-service')

export async function myFunction() {
  const span = tracer.startSpan('myFunction')
  
  try {
    // Your code here
    const result = await someOperation()
    span.setStatus({ code: SpanStatusCode.OK })
    return result
  } catch (error) {
    span.setStatus({ 
      code: SpanStatusCode.ERROR, 
      message: error.message 
    })
    throw error
  } finally {
    span.end()
  }
}
```

### Custom Metrics

Track custom business metrics:

```typescript
import { metrics } from '@opentelemetry/api'

const meter = metrics.getMeter('my-service')
const userLoginCounter = meter.createCounter('user_logins', {
  description: 'Number of user logins'
})

export function trackLogin(userId: string) {
  userLoginCounter.add(1, { userId })
}
```

### Custom Attributes

Add custom properties to spans:

```typescript
import { trace } from '@opentelemetry/api'

const span = trace.getActiveSpan()
if (span) {
  span.setAttribute('user.id', userId)
  span.setAttribute('tenant.id', tenantId)
  span.setAttribute('environment.name', environmentName)
}
```

## Filtering Sensitive Data

To prevent logging sensitive data:

### 1. Filter HTTP Headers

Modify `instrumentation.ts`:

```typescript
useAzureMonitor({
  azureMonitorExporterOptions: {
    connectionString,
  },
  instrumentationOptions: {
    http: {
      enabled: true,
      // Don't capture these headers
      ignoreIncomingRequestHook: (request) => {
        // Ignore health check endpoints
        return request.url?.includes('/health')
      },
      // Filter sensitive headers
      requestHook: (span, request) => {
        // Remove Authorization header from telemetry
        if (request.headers) {
          delete request.headers['authorization']
          delete request.headers['x-api-key']
        }
      }
    }
  }
})
```

### 2. Use Telemetry Processors

Add a processor to filter data before export:

```typescript
import { Span } from '@opentelemetry/sdk-trace-base'

const filterProcessor = {
  onStart: (span: Span) => {
    // Modify span on start
  },
  onEnd: (span: Span) => {
    // Remove sensitive attributes
    span.attributes['http.request.header.authorization'] = '[REDACTED]'
  }
}
```

## Querying Telemetry Data

### In Azure Portal

**Application Insights > Logs (KQL)**

#### Query all requests to Power Platform API:
```kql
requests
| where url contains "/api/power-platform"
| project timestamp, name, url, duration, resultCode
| order by timestamp desc
| take 100
```

#### Query failed requests:
```kql
requests
| where success == false
| project timestamp, name, url, duration, resultCode, customDimensions
| order by timestamp desc
```

#### Query dependencies (external API calls):
```kql
dependencies
| where type == "Http"
| where target contains "graph.microsoft.com" or target contains "api.powerplatform.com"
| project timestamp, name, target, duration, resultCode
| summarize avg(duration), count() by target
```

#### Query exceptions:
```kql
exceptions
| project timestamp, type, outerMessage, innermostMessage, operation_Name
| order by timestamp desc
```

## Performance Impact

OpenTelemetry adds minimal overhead:
- **CPU**: < 1% increase
- **Memory**: ~10-20 MB
- **Latency**: < 1ms per request
- **Network**: Telemetry batched every 30 seconds

## Troubleshooting

### No Data Appearing

1. **Check connection string**:
   ```bash
   echo $APPLICATIONINSIGHTS_CONNECTION_STRING
   ```

2. **Check console for initialization**:
   ```
   [OpenTelemetry] Azure Monitor OpenTelemetry initialized successfully
   ```

3. **Verify environment**:
   - Only runs in Node.js runtime (not browser)
   - Check `process.env.NEXT_RUNTIME === 'nodejs'`

4. **Wait 2-5 minutes** for data to appear in portal

### Wrong Service Name

Modify `instrumentation.ts`:
```typescript
resource: new Resource({
  [ATTR_SERVICE_NAME]: 'your-custom-name',
})
```

### Too Much Data

Configure sampling in `instrumentation.ts`:
```typescript
import { TraceIdRatioBasedSampler } from '@opentelemetry/sdk-trace-base'

useAzureMonitor({
  // ... other config
  samplerConfig: {
    samplingRatio: 0.1 // Sample 10% of requests
  }
})
```

## Cost Considerations

Application Insights pricing is based on:
- **Data ingestion**: $2.30/GB (first 5 GB/month free)
- **Data retention**: 90 days included, then $0.10/GB/month

### Typical usage for this app:
- **10,000 requests/day**: ~50 MB/day = 1.5 GB/month
- **100,000 requests/day**: ~500 MB/day = 15 GB/month
- **Cost**: ~$2-25/month depending on traffic

### Reduce costs:
1. **Sampling**: Sample 10-20% of requests
2. **Filter**: Exclude health check endpoints
3. **Retention**: Reduce to 30 days
4. **Aggregation**: Use metrics instead of traces for high-volume APIs

## Integration with Existing Application Insights

If you already have Application Insights setup (for browser monitoring):
- **This adds**: Server-side telemetry (Node.js API routes)
- **Existing browser telemetry**: Still works via JavaScript SDK
- **Combined view**: See full end-to-end traces (browser → API → dependencies)

## References

- [OpenTelemetry Node.js Getting Started](https://opentelemetry.io/docs/languages/js/getting-started/nodejs/)
- [Azure Monitor OpenTelemetry for Node.js](https://learn.microsoft.com/en-us/azure/azure-monitor/app/opentelemetry-enable?tabs=nodejs)
- [Next.js Instrumentation](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation)
- [Azure Monitor OpenTelemetry npm](https://www.npmjs.com/package/@azure/monitor-opentelemetry)
- [OpenTelemetry Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/)

## Next Steps

1. ✅ Add connection string to `.env.local`
2. ✅ Restart application
3. ✅ Verify console logs show initialization
4. ✅ Wait 5 minutes and check Application Insights portal
5. ⬜ Add custom spans for critical operations
6. ⬜ Set up alerts for high error rates
7. ⬜ Create dashboards for key metrics
8. ⬜ Configure sampling if needed
