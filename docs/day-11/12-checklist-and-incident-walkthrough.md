# Checklist and Incident Walkthrough

[← Observability in Microservices](./11-observability-in-microservices.md) | [Day 11 Index](./README.md)

## Production Observability Checklist

### Logging

- [ ] Structured JSON logs to stdout
- [ ] `request_id` and `trace_id` on every log line
- [ ] No secrets or unnecessary PII
- [ ] Central log aggregation with search
- [ ] Retention policy defined (30–90 days typical)

### Metrics

- [ ] RED metrics on every service and gateway route
- [ ] USE metrics on DB, cache, queues
- [ ] Low-cardinality labels only
- [ ] Grafana (or equivalent) dashboards per service + system overview
- [ ] Deploy annotations on charts

### Tracing

- [ ] OpenTelemetry (or equivalent) in all services
- [ ] Trace context through gateway → services → external APIs
- [ ] `trace_id` in async messages
- [ ] Sampling strategy (keep errors and slow traces)

### Alerting

- [ ] Alerts on symptoms (error rate, latency, SLO burn)
- [ ] P1/P2 runbooks linked
- [ ] Alert fatigue review quarterly
- [ ] On-call rotation and escalation path

### SLOs

- [ ] SLIs defined on user journeys (checkout, login)
- [ ] SLO targets and error budget dashboard
- [ ] Burn-rate alerts configured
- [ ] Deploy policy tied to budget remaining

---

## Incident Walkthrough: Checkout Slow

**10:40 UTC** — P2 alert: checkout p99 > 3s (SLO burn rate elevated)

### Step 1 — Confirm symptom

```
Dashboard: order-service p99 jumped 200ms → 2.8s at 10:38
Error rate still low (0.2%) — latency not failures
```

### Step 2 — Check recent changes

```
Deploy marker: payment-service v1.8.2 at 10:35
Hypothesis: deploy regression
```

### Step 3 — Open trace

```
Sample slow trace (trace_id: abc123):
  gateway 15ms
  order-service 2700ms
    payment-service 2650ms
      stripe.api 2600ms  ← bottleneck
```

### Step 4 — Correlate logs

```
filter: trace_id="abc123"
payment-service: "stripe_timeout" duration_ms=30000
```

### Step 5 — Check dependency status

```
Stripe status page: elevated latency us-east-1
Not our deploy — external dependency
```

### Step 6 — Mitigate

```
Enable circuit breaker fallback message (Day 9)
Reduce payment client timeout from 30s → 5s (Day 9)
Communicate status to support
```

### Step 7 — Post-incident

```
Document timeline
Add alert: payment dependency latency SLO
Consider secondary payment provider (Day 9 graceful degradation)
```

**MTTR without observability:** hours guessing. **With traces + logs:** ~15 minutes to external root cause.

---

## Quick Reference

| Question | Tool |
|----------|------|
| Is the system up? | Health checks, uptime metrics |
| Are users impacted? | SLI dashboard, error rate |
| Which service is slow? | RED metrics, trace waterfall |
| Why did one request fail? | Logs filtered by trace_id |
| What changed? | Deploy markers, config diffs |
| Is budget exhausted? | Error budget panel |

---

## Summary

Use the **checklist** before launch and audit quarterly. During incidents: confirm **symptom** → check **deploys** → follow **traces** → search **logs** → mitigate → postmortem. Observability turns chaos into a repeatable workflow.

---

[← Observability in Microservices](./11-observability-in-microservices.md) | [Day 11 Index](./README.md)
