# OpenTelemetry Quick Start Guide

## ✅ What Was Done

1. **Installed OpenTelemetry packages**:
   - `@azure/monitor-opentelemetry` - Azure Monitor integration
   - `@opentelemetry/api` - Core API
   - `@opentelemetry/sdk-metrics` - Metrics
   - `@opentelemetry/resources` - Resource definitions
   - `@opentelemetry/semantic-conventions` - Standard conventions
   - `@opentelemetry/sdk-trace-base` - Tracing

2. **Created `instrumentation.ts`** - Auto-instruments your Next.js app

3. **Updated `next.config.ts`** - Enabled instrumentation hook

4. **Added environment variable** to `.env.local`

## 🚀 Quick Setup (3 Steps)

### Step 1: Get Connection String
1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to **Application Insights** resource (or create one)
3. Click **Overview** → Copy **Connection String**

### Step 2: Add to `.env.local`
```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...;IngestionEndpoint=https://...
```

### Step 3: Restart App
```bash
npm run dev
```

✅ Look for this in console:
```
[OpenTelemetry] Azure Monitor OpenTelemetry initialized successfully
```

## 📊 What Gets Tracked Automatically

- ✅ **HTTP Requests**: Every API call to `/api/*`
- ✅ **External APIs**: Calls to Microsoft Graph, Power Platform, Azure
- ✅ **Performance**: Request duration, status codes
- ✅ **Errors**: Exceptions with stack traces
- ✅ **Dependencies**: External service calls

## 🔍 View Your Data

**Azure Portal > Application Insights > Your Resource**

- **Application Map**: See service dependencies
- **Performance**: Request durations & bottlenecks
- **Failures**: Errors and exceptions
- **Live Metrics**: Real-time data stream
- **Logs**: Query with KQL

### Example Query (Logs):
```kql
requests
| where url contains "/api/"
| project timestamp, name, duration, resultCode
| order by timestamp desc
| take 100
```

## 🎯 Cost Estimate

- **Free tier**: 5 GB/month
- **Typical usage**: 1-15 GB/month
- **Cost**: $0-25/month (depending on traffic)

## 📚 Full Documentation

See `docs/OPENTELEMETRY-SETUP.md` for:
- Detailed architecture
- Custom instrumentation
- Advanced configuration
- Troubleshooting
- Cost optimization

## ❓ Troubleshooting

**No data appearing?**
1. Check connection string is set: `echo $APPLICATIONINSIGHTS_CONNECTION_STRING`
2. Restart the app: `npm run dev`
3. Wait 2-5 minutes for data to appear
4. Check console for initialization message

**Wrong service name?**
- Edit `instrumentation.ts` → change `ATTR_SERVICE_NAME`

**Too much data?**
- Enable sampling in `instrumentation.ts` (sample 10-20% of requests)
