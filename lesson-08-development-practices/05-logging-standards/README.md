# 8.5 Logging Standards

## Overview

Effective logging is essential for monitoring, debugging, and auditing insurance systems. This sub-lesson covers log levels, structured logging practices, and implementation patterns.

---

## Log Levels

| Level | Usage | Example |
|-------|-------|---------|
| **ERROR** | System errors requiring attention | Database connection failed |
| **WARN** | Potential issues, degraded state | External service timeout, retrying |
| **INFO** | Business events, milestones | Policy issued, claim settled |
| **DEBUG** | Detailed technical information | Request/response payloads |
| **TRACE** | Very detailed debugging | Method entry/exit |

---

## Log Level Guidelines

### ERROR
- System failures that prevent normal operation
- Exceptions that require immediate attention
- Data corruption or loss scenarios

### WARN
- Recoverable errors with retry logic
- Approaching resource limits
- Deprecated feature usage
- Unusual but valid business scenarios

### INFO
- Business transaction completions
- Service startup/shutdown
- Configuration changes
- User-initiated actions

### DEBUG
- Detailed flow information
- Variable values during processing
- External service request/response details

### TRACE
- Method entry/exit
- Loop iterations
- Very detailed diagnostic information

---

## Structured Logging

```java
@Service
@Slf4j
public class PolicyService {

    public Policy issuePolicy(IssuePolicyRequest request) {
        log.info("Issuing policy",
            kv("customerId", request.getCustomerId()),
            kv("productCode", request.getProductCode()),
            kv("sumInsured", request.getSumInsured()));

        try {
            Policy policy = createPolicy(request);

            log.info("Policy issued successfully",
                kv("policyId", policy.getId()),
                kv("policyNumber", policy.getPolicyNumber()),
                kv("premium", policy.getTotalPremium()));

            return policy;

        } catch (UnderwritingRejectedException e) {
            log.warn("Policy issuance rejected by underwriting",
                kv("customerId", request.getCustomerId()),
                kv("reason", e.getReason()));
            throw e;

        } catch (Exception e) {
            log.error("Failed to issue policy",
                kv("customerId", request.getCustomerId()),
                kv("error", e.getMessage()),
                e);
            throw e;
        }
    }
}
```

---

## Logging Best Practices

### What to Log

| Always Log | Context to Include |
|------------|-------------------|
| Business transactions | Transaction ID, entity IDs |
| External service calls | Service name, response time |
| Authentication events | User ID (never passwords) |
| Error conditions | Error code, context, stack trace |
| Configuration changes | What changed, who changed it |

### What NOT to Log

| Never Log | Reason |
|-----------|--------|
| Passwords | Security risk |
| Credit card numbers | PCI compliance |
| Personal health data | HIPAA compliance |
| Full social security numbers | Privacy regulation |
| API keys or secrets | Security risk |

---

## Log Format Example

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "logger": "com.insurance.PolicyService",
  "message": "Policy issued successfully",
  "context": {
    "policyId": "POL-123456",
    "policyNumber": "INS-2024-001234",
    "premium": 1500.00,
    "correlationId": "corr_abc123",
    "traceId": "trace_xyz789"
  }
}
```

---

## Correlation and Tracing

```java
// MDC (Mapped Diagnostic Context) for correlation
public class CorrelationFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        try {
            String correlationId = extractOrGenerateCorrelationId(request);
            MDC.put("correlationId", correlationId);
            MDC.put("traceId", generateTraceId());

            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

---

## Navigation

[Previous: 8.4 Error Handling](../04-error-handling/README.md) | [Back to Lesson 08](../README.md) | [Next: 8.6 Code Quality](../06-code-quality/README.md)
