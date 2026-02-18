# 04 - End-to-End Testing

[Back to Lesson 09](../README.md)

---

## Overview

End-to-end (E2E) tests validate complete user journeys through the system. They test the entire application stack, including APIs, services, databases, and external integrations. While expensive and slow, E2E tests are essential for verifying critical business workflows.

---

## API E2E Tests

### Quote Creation and Premium Breakdown

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class QuoteApiE2ETest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("Should create quote and return premium breakdown")
    void shouldCreateQuoteWithPremiumBreakdown() throws Exception {
        // Given
        CreateQuoteRequest request = CreateQuoteRequest.builder()
            .productCode("MOTOR_COMP")
            .customer(CustomerDto.builder()
                .firstName("John")
                .lastName("Doe")
                .dateOfBirth(LocalDate.of(1985, 3, 15))
                .email("john.doe@email.com")
                .build())
            .riskDetails(MotorRiskDetailsDto.builder()
                .vehicleMake("Toyota")
                .vehicleModel("Camry")
                .vehicleYear(2022)
                .engineCapacity(2500)
                .usageType("PRIVATE")
                .build())
            .coverageOptions(CoverageOptionsDto.builder()
                .sumInsured(50000)
                .deductible(500)
                .build())
            .effectiveDate(LocalDate.now().plusDays(7))
            .build();

        // When/Then
        mockMvc.perform(post("/api/v1/quotes")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.quote_id").exists())
            .andExpect(jsonPath("$.status").value("ACTIVE"))
            .andExpect(jsonPath("$.premium_breakdown.base_premium").isNumber())
            .andExpect(jsonPath("$.premium_breakdown.total_premium").isNumber())
            .andExpect(jsonPath("$.payment_options").isArray())
            .andExpect(jsonPath("$.payment_options[0].frequency").value("ANNUAL"));
    }

    @Test
    @DisplayName("Should convert quote to policy")
    void shouldConvertQuoteToPolicy() throws Exception {
        // Given - Create quote first
        String quoteId = createQuote();

        ConvertQuoteRequest request = ConvertQuoteRequest.builder()
            .paymentMethod("CREDIT_CARD")
            .paymentFrequency("ANNUAL")
            .build();

        // When/Then
        mockMvc.perform(post("/api/v1/quotes/{id}/convert", quoteId)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.policy_id").exists())
            .andExpect(jsonPath("$.policy_number").exists())
            .andExpect(jsonPath("$.status").value("ISSUED"));
    }
}
```

---

## Claims Journey E2E Test

### Complete Claims Workflow

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class ClaimsJourneyE2ETest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    @DisplayName("Should complete full claims journey from FNOL to settlement")
    void shouldCompleteFullClaimsJourney() {
        // Step 1: Register Claim (FNOL)
        RegisterClaimRequest fnolRequest = createFnolRequest();
        ResponseEntity<ClaimResponse> fnolResponse = restTemplate.postForEntity(
            "/api/v1/claims", fnolRequest, ClaimResponse.class);

        assertThat(fnolResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        String claimId = fnolResponse.getBody().getClaimId();
        assertThat(fnolResponse.getBody().getStatus()).isEqualTo("OPEN");

        // Step 2: Upload Documents
        uploadClaimDocuments(claimId);

        // Step 3: Submit Assessment
        AssessmentRequest assessmentRequest = AssessmentRequest.builder()
            .assessedAmount(new Money("4500", "USD"))
            .assessmentNotes("Damage confirmed, repair estimate attached")
            .recommendation("APPROVE")
            .build();

        restTemplate.postForEntity(
            "/api/v1/claims/{id}/assess", assessmentRequest, Void.class, claimId);

        // Step 4: Approve Claim
        ApprovalRequest approvalRequest = ApprovalRequest.builder()
            .approvedAmount(new Money("4500", "USD"))
            .deductible(new Money("500", "USD"))
            .settlementType("REPAIR")
            .build();

        ResponseEntity<ClaimResponse> approvalResponse = restTemplate.postForEntity(
            "/api/v1/claims/{id}/approve", approvalRequest, ClaimResponse.class, claimId);

        assertThat(approvalResponse.getBody().getStatus()).isEqualTo("APPROVED");

        // Step 5: Settle Claim
        SettlementRequest settlementRequest = SettlementRequest.builder()
            .paymentMethod("BANK_TRANSFER")
            .bankAccount("1234567890")
            .build();

        ResponseEntity<ClaimResponse> settlementResponse = restTemplate.postForEntity(
            "/api/v1/claims/{id}/settle", settlementRequest, ClaimResponse.class, claimId);

        assertThat(settlementResponse.getBody().getStatus()).isEqualTo("SETTLED");
        assertThat(settlementResponse.getBody().getSettledAmount().getAmount())
            .isEqualByComparingTo("4000"); // 4500 - 500 deductible
    }
}
```

---

## Critical E2E Test Scenarios for Insurance

| Journey | Steps to Test |
|---------|---------------|
| **Quote to Policy** | Create quote -> Select coverage -> Apply discounts -> Convert to policy |
| **Policy Renewal** | Identify expiring -> Calculate new premium -> Generate renewal -> Accept/Decline |
| **Claims Journey** | FNOL -> Document upload -> Assessment -> Approval -> Settlement |
| **Policy Cancellation** | Request cancellation -> Calculate refund -> Process refund -> Update status |
| **Endorsement** | Request change -> Calculate adjustment -> Issue endorsement -> Update policy |

---

## Best Practices for E2E Testing

1. **Test happy paths** for critical business journeys
2. **Use realistic test data** that mirrors production scenarios
3. **Clean up test data** after each test run
4. **Run E2E tests in isolated environments**
5. **Keep E2E tests minimal** - focus on critical paths only
6. **Use meaningful assertions** that validate business outcomes

---

## Navigation

[Previous: Integration Testing](../03-integration-testing/README.md) | [Back to Lesson 09](../README.md) | [Next: Test Data Management](../05-test-data-management/README.md)
