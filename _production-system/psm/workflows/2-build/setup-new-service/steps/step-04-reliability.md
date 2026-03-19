---
step_number: 4
step_name: "Reliability, Observability & Error Handling"
next_step_file: "steps/step-05-security-infra.md"
estimated_time: "2.5-3 hours"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang"]
skill_loads:
  - "references/5.07 - RUN - Reliability & Resilience/SKILL.md"
  - "references/5.08 - RUN - Observability & Monitoring/SKILL.md"
  - "references/5.09 - RUN - Error Handling/SKILL.md"
is_final_step: false
outputFile: "{project-root}/psm/outputs/setup-new-service/{service_name}/reliability-plan.md"
workflowType: "setup-new-service"
---

# STEP 4: Reliability, Observability & Error Handling

## STEP GOAL

Build resilient service with comprehensive observability and error handling:

- **Resilience patterns**: Circuit breaker, retry logic, timeout, fallback
- **Structured logging**: JSON logs with correlation IDs, log levels
- **Metrics & alerting**: Prometheus, Grafana setup, key metrics
- **Distributed tracing**: OpenTelemetry instrumentation
- **Error handling strategy**: Categorization, recovery mechanisms

**Output**: `reliability-plan.md` + `observability-config.md` + `error-handling.md` ready for step-05

## Skill Files to Load

```
Load: references/5.07 - RUN - Reliability & Resilience/SKILL.md
      → Resilience patterns, circuit breaker, bulkhead

Load: references/5.08 - RUN - Observability & Monitoring/SKILL.md
      → Logging, metrics, tracing, alerting strategies

Load: references/5.09 - RUN - Error Handling/SKILL.md
      → Error categorization, retry strategies, fallback mechanisms
```

## MANDATORY RULES

1. **Observable by design** — Instrument from day 1, not retrofitted
2. **Fail gracefully** — Always have fallback behavior
3. **Structured logging** — JSON format, consistent fields
4. **Correlation IDs** — Trace requests across services
5. **Alert on outcomes** — Alert on business metrics, not just infrastructure

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-architecture.md", "step-02-api-database.md", "step-03-build-deploy.md"]
lastStep: "step-04-reliability.md"
lastSaved: "{current-timestamp}"
workflowType: "setup-new-service"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## EXECUTION PROTOCOL

### Phase 1: Resilience Patterns (25 min)

**Pattern 1: Circuit Breaker**

Purpose: Prevent cascading failures by stopping calls to failing service

```
States: [CLOSED] → [OPEN] → [HALF_OPEN] → [CLOSED]

CLOSED: Normal operation
  → 5 failures in 1min → transition to OPEN

OPEN: Reject all requests immediately
  → After 30 seconds → transition to HALF_OPEN

HALF_OPEN: Allow test request
  → Success → transition to CLOSED
  → Failure → transition to OPEN
```

**Implementation**:

```python
from pybreaker import CircuitBreaker

breaker = CircuitBreaker(
    fail_max=5,
    reset_timeout=30,
    exclude=[Exception],  # Don't count these
    listeners=[LogListener()]  # Log state changes
)

@breaker
def call_external_service():
    return requests.get('https://api.example.com/data')

try:
    response = call_external_service()
except CircuitBreakerListener:
    return fallback_response()
```

**Pattern 2: Retry with Exponential Backoff**

```
Attempt 1: Immediate
Attempt 2: Wait 1s + random jitter
Attempt 3: Wait 2s + random jitter
Attempt 4: Wait 4s + random jitter
Attempt 5: Wait 8s + random jitter
```

**Configuration**:

```yaml
retry_policy:
  max_attempts: 5
  initial_delay_ms: 100
  max_delay_ms: 30000
  exponential_base: 2.0
  jitter_ratio: 0.1
  retryable_status_codes: [408, 429, 500, 502, 503, 504]
```

**Pattern 3: Timeout**

```
Request → Service (10s timeout)
  → Success (< 10s) → Return response
  → Timeout (> 10s) → Trigger fallback
  → Circuit Open → Immediate fallback
```

**Pattern 4: Fallback / Graceful Degradation**

```
Primary path unavailable → Cached data
Primary path slow (> 1s) → Show cached version + refresh in background
Complete failure → Return empty response with 503 Service Unavailable
```

**Document**: Create `resilience-patterns.md`

### Phase 2: Structured Logging (20 min)

**JSON Logging Format**:

```json
{
  "timestamp": "2026-03-17T10:30:45.123Z",
  "level": "ERROR",
  "logger_name": "OrderService",
  "message": "Failed to process payment",
  "correlation_id": "req-abc123def456",
  "trace_id": "trace-xyz789",
  "service_name": "order-api",
  "environment": "production",
  "user_id": "user-12345",
  "duration_ms": 2500,
  "error": {
    "type": "PaymentGatewayError",
    "message": "Connection timeout",
    "stack": "..."
  },
  "context": {
    "order_id": "order-54321",
    "amount": 99.99,
    "retry_count": 2
  }
}
```

**Log Levels Strategy**:

```
ERROR: Unexpected failures, requires immediate action
WARNING: Degraded conditions, should investigate
INFO: Important business events (payment, order)
DEBUG: Detailed execution flow (queries, API calls)
TRACE: Most detailed, variable values
```

**Logging Configuration**:

```python
import logging
import json
import sys
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'correlation_id': getattr(record, 'correlation_id', ''),
        }
        if record.exc_info:
            log_data['error'] = self.formatException(record.exc_info)
        return json.dumps(log_data)

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(JSONFormatter())
logging.getLogger().addHandler(handler)
```

### Phase 3: Metrics & Alerting (25 min)

**Key Metrics to Track**:

**RED Method** (Request-based):

```
Rate: Requests per second (RPS)
  prometheus_counter: http_requests_total

Errors: Failed requests percentage
  prometheus_counter: http_requests_failed_total
  Alert: > 1% error rate for 5 minutes

Duration: Request latency
  prometheus_histogram: http_request_duration_ms
  Alert: p99 latency > 500ms for 10 minutes
```

**Example Prometheus Configuration**:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - '/etc/prometheus/rules.yml'

scrape_configs:
  - job_name: 'api-service'
    static_configs:
      - targets: ['localhost:8080']
```

**Alert Rules**:

```yaml
groups:
  - name: api_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (sum(rate(http_requests_failed_total[5m])) /
           sum(rate(http_requests_total[5m]))) > 0.01
        for: 5m
        annotations:
          severity: critical
          summary: "High error rate detected"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, http_request_duration_ms) > 500
        for: 10m
        annotations:
          severity: warning
          summary: "High latency detected"

      - alert: CircuitBreakerOpen
        expr: |
          circuit_breaker_state{state="open"} > 0
        for: 1m
        annotations:
          severity: warning
          summary: "Circuit breaker opened"
```

### Phase 4: Distributed Tracing (20 min)

**OpenTelemetry Setup**:

```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

jaeger_exporter = JaegerExporter(
    agent_host_name='localhost',
    agent_port=6831,
)

trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("process_order") as span:
    span.set_attribute("order_id", order_id)
    span.set_attribute("user_id", user_id)

    # Automatic child spans for database calls
    process_order(order_id)
```

**Key Trace Attributes**:

```
Span name: Describes operation (e.g., "db.query", "http.request")
Trace ID: Unique identifier for entire request flow
Span ID: Unique identifier for this operation
Parent Span ID: Links to parent operation
Duration: How long operation took
Attributes: Key-value context (user_id, order_id, etc.)
Events: Markers within span (retry, fallback used)
Status: OK, ERROR, or UNSET
```

### Phase 5: Error Handling Strategy (20 min)

**Error Categorization**:

```
Transient Errors (retry with backoff):
- Connection timeouts
- Temporary 503 Service Unavailable
- Database connection pool exhausted

Permanent Errors (fail fast):
- 400 Bad Request
- 401 Unauthorized
- 404 Not Found
- Validation errors

Degradation Errors (fallback):
- External API slow (> 1s)
- Cache miss (use stale data)
- Non-critical dependency down
```

**Error Response Format**:

```json
{
  "error_code": "PAYMENT_FAILED",
  "message": "Payment processing failed, please retry",
  "details": {
    "reason": "Gateway timeout",
    "retry_after_seconds": 5,
    "support_ticket": "ticket-123456"
  },
  "timestamp": "2026-03-17T10:30:45.123Z",
  "request_id": "req-abc123"
}
```

**Document**: Create `error-handling-strategy.md`

## MENU - Choose Your Path

### [A] Accept reliability and observability design
- Confirms resilience patterns
- Validates monitoring strategy
- Proceeds to step-05

### [C] Continue refining
- Adjusts alerting thresholds
- Revisits error categorization
- Refines resilience patterns

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-05-security-infra.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-05-security-infra.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
