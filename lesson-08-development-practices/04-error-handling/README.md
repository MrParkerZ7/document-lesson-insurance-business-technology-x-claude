# 8.4 Error Handling

## Overview

Robust error handling is critical for insurance systems where accurate processing and clear feedback are essential. This sub-lesson covers exception hierarchies, global handlers, and standardized error responses.

---

## Exception Hierarchy

### Base Exception

```java
public abstract class InsuranceException extends RuntimeException {
    private final String errorCode;
    private final Map<String, Object> context;

    protected InsuranceException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }

    public InsuranceException withContext(String key, Object value) {
        this.context.put(key, value);
        return this;
    }
}
```

### Business Exceptions (4xx)

```java
// Validation Exception
public class ValidationException extends InsuranceException {
    private final List<FieldError> fieldErrors;

    public ValidationException(List<FieldError> errors) {
        super("VALIDATION_ERROR", "Request validation failed");
        this.fieldErrors = errors;
    }
}

// Not Found Exception
public class PolicyNotFoundException extends InsuranceException {
    public PolicyNotFoundException(String policyId) {
        super("POLICY_NOT_FOUND", "Policy not found: " + policyId);
        withContext("policyId", policyId);
    }
}

// State Transition Exception
public class InvalidStateTransitionException extends InsuranceException {
    public InvalidStateTransitionException(String currentState, String targetState) {
        super("INVALID_STATE_TRANSITION",
            String.format("Cannot transition from %s to %s", currentState, targetState));
        withContext("currentState", currentState);
        withContext("targetState", targetState);
    }
}
```

### Technical Exceptions (5xx)

```java
public class ExternalServiceException extends InsuranceException {
    public ExternalServiceException(String service, String message) {
        super("EXTERNAL_SERVICE_ERROR", message);
        withContext("service", service);
    }
}
```

---

## Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.warn("Validation failed: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .details(ex.getFieldErrors())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(PolicyNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(PolicyNotFoundException ex) {
        log.info("Resource not found: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }

    @ExceptionHandler(InvalidStateTransitionException.class)
    public ResponseEntity<ErrorResponse> handleStateTransition(InvalidStateTransitionException ex) {
        log.warn("Invalid state transition: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .context(ex.getContext())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);

        ErrorResponse response = ErrorResponse.builder()
            .code("INTERNAL_ERROR")
            .message("An unexpected error occurred")
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

---

## Error Response Structure

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "effectiveDate",
        "code": "INVALID_DATE",
        "message": "Effective date must be in the future"
      },
      {
        "field": "sumInsured",
        "code": "EXCEEDS_LIMIT",
        "message": "Sum insured exceeds product maximum of $1,000,000"
      }
    ],
    "correlation_id": "corr_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

---

## Exception Categories

| Category | HTTP Status | Use Case |
|----------|-------------|----------|
| Validation | 400 | Invalid input data |
| Not Found | 404 | Resource does not exist |
| State Transition | 422 | Invalid business operation |
| Authentication | 401 | Missing or invalid credentials |
| Authorization | 403 | Insufficient permissions |
| External Service | 502/503 | Third-party service failures |
| Internal Error | 500 | Unexpected system errors |

---

## Best Practices

1. **Provide Context**: Include relevant IDs and state information
2. **Use Error Codes**: Enable client-side error handling
3. **Log Appropriately**: ERROR for system issues, WARN for business violations
4. **Never Expose Internals**: Hide stack traces and system details from clients
5. **Correlation IDs**: Enable request tracing across services

---

## Navigation

[Previous: 8.3 Naming Conventions](../03-naming-conventions/README.md) | [Back to Lesson 08](../README.md) | [Next: 8.5 Logging Standards](../05-logging-standards/README.md)
