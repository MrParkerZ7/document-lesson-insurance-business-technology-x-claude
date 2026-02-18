# Lesson 09: Testing Strategies

## Learning Objectives

By the end of this lesson, you will be able to:
- Design comprehensive test strategies for insurance systems
- Implement unit, integration, and E2E tests
- Manage test data effectively
- Apply testing best practices for insurance domain

---

## 9.1 Testing Pyramid

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          TESTING PYRAMID                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │       E2E         │  ← Few tests                  │
│                         │      Tests        │    Slow, expensive            │
│                         │    (UI/API)       │    Real dependencies          │
│                         └───────────────────┘                               │
│                                                                              │
│                    ┌─────────────────────────────┐                          │
│                    │       Integration           │  ← Some tests            │
│                    │         Tests               │    Medium speed           │
│                    │  (Service + Dependencies)   │    Test containers        │
│                    └─────────────────────────────┘                          │
│                                                                              │
│              ┌───────────────────────────────────────────┐                  │
│              │              Unit Tests                    │  ← Many tests   │
│              │     (Single class/function in isolation)   │    Fast, cheap  │
│              │                                            │    Mocked deps  │
│              └───────────────────────────────────────────┘                  │
│                                                                              │
│  RATIO: Unit (70%) : Integration (20%) : E2E (10%)                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9.2 Unit Testing

### Testing Premium Calculation

```java
@ExtendWith(MockitoExtension.class)
class MotorPremiumCalculatorTest {

    @Mock
    private RateTableRepository rateTableRepository;

    @InjectMocks
    private MotorPremiumCalculator calculator;

    @Test
    @DisplayName("Should calculate base premium correctly for standard vehicle")
    void shouldCalculateBasePremiumForStandardVehicle() {
        // Given
        MotorRiskDetails risk = MotorRiskDetails.builder()
            .vehicleValue(new Money("50000", "USD"))
            .vehicleAge(2)
            .engineCapacity(2500)
            .usageType(UsageType.PRIVATE)
            .build();

        when(rateTableRepository.getBaseRate("MOTOR", "PRIVATE"))
            .thenReturn(new BigDecimal("0.025"));

        // When
        Money basePremium = calculator.calculateBasePremium(risk);

        // Then
        assertThat(basePremium.getAmount()).isEqualByComparingTo("1250.00");
        assertThat(basePremium.getCurrency()).isEqualTo("USD");
    }

    @Test
    @DisplayName("Should apply young driver loading for drivers under 25")
    void shouldApplyYoungDriverLoading() {
        // Given
        MotorRiskDetails risk = MotorRiskDetails.builder()
            .driverAge(22)
            .yearsLicensed(2)
            .build();

        when(rateTableRepository.getLoadingFactor("YOUNG_DRIVER", 22))
            .thenReturn(new BigDecimal("0.15"));

        // When
        LoadingResult loading = calculator.calculateYoungDriverLoading(risk);

        // Then
        assertThat(loading.getPercentage()).isEqualByComparingTo("15.00");
        assertThat(loading.getType()).isEqualTo(LoadingType.YOUNG_DRIVER);
    }

    @Test
    @DisplayName("Should apply no claims bonus discount")
    void shouldApplyNoClaimsBonusDiscount() {
        // Given
        MotorRiskDetails risk = MotorRiskDetails.builder()
            .claimFreeYears(5)
            .build();

        when(rateTableRepository.getDiscountFactor("NCB", 5))
            .thenReturn(new BigDecimal("0.25"));

        // When
        DiscountResult discount = calculator.calculateNoClaimsBonus(risk);

        // Then
        assertThat(discount.getPercentage()).isEqualByComparingTo("25.00");
        assertThat(discount.getType()).isEqualTo(DiscountType.NO_CLAIMS_BONUS);
    }

    @ParameterizedTest
    @CsvSource({
        "1, 0.00",    // 1 year = no discount
        "2, 10.00",   // 2 years = 10%
        "3, 15.00",   // 3 years = 15%
        "4, 20.00",   // 4 years = 20%
        "5, 25.00"    // 5+ years = 25% (max)
    })
    @DisplayName("Should calculate correct NCB discount for different years")
    void shouldCalculateCorrectNcbDiscount(int claimFreeYears, String expectedDiscount) {
        // Given
        MotorRiskDetails risk = MotorRiskDetails.builder()
            .claimFreeYears(claimFreeYears)
            .build();

        mockNcbRates();

        // When
        DiscountResult discount = calculator.calculateNoClaimsBonus(risk);

        // Then
        assertThat(discount.getPercentage())
            .isEqualByComparingTo(new BigDecimal(expectedDiscount));
    }
}
```

### Testing Policy State Machine

```java
class PolicyStateMachineTest {

    private PolicyStateMachine stateMachine;

    @BeforeEach
    void setUp() {
        stateMachine = new PolicyStateMachine();
    }

    @Test
    @DisplayName("Should transition from ISSUED to ACTIVE when activated")
    void shouldTransitionToActiveWhenActivated() {
        // Given
        Policy policy = Policy.builder()
            .status(PolicyStatus.ISSUED)
            .effectiveDate(LocalDate.now())
            .build();

        // When
        PolicyStatus newStatus = stateMachine.activate(policy);

        // Then
        assertThat(newStatus).isEqualTo(PolicyStatus.ACTIVE);
    }

    @Test
    @DisplayName("Should not allow cancellation of already cancelled policy")
    void shouldNotAllowDoubleCancellation() {
        // Given
        Policy policy = Policy.builder()
            .status(PolicyStatus.CANCELLED)
            .build();

        // When/Then
        assertThatThrownBy(() -> stateMachine.cancel(policy))
            .isInstanceOf(InvalidStateTransitionException.class)
            .hasMessageContaining("Cannot cancel policy in CANCELLED state");
    }

    @Test
    @DisplayName("Should allow reinstatement within reinstatement period")
    void shouldAllowReinstatementWithinPeriod() {
        // Given
        Policy policy = Policy.builder()
            .status(PolicyStatus.LAPSED)
            .lapseDate(LocalDate.now().minusDays(15))
            .reinstatementPeriodDays(30)
            .build();

        // When
        PolicyStatus newStatus = stateMachine.reinstate(policy);

        // Then
        assertThat(newStatus).isEqualTo(PolicyStatus.ACTIVE);
    }

    @Test
    @DisplayName("Should reject reinstatement after reinstatement period")
    void shouldRejectReinstatementAfterPeriod() {
        // Given
        Policy policy = Policy.builder()
            .status(PolicyStatus.LAPSED)
            .lapseDate(LocalDate.now().minusDays(45))
            .reinstatementPeriodDays(30)
            .build();

        // When/Then
        assertThatThrownBy(() -> stateMachine.reinstate(policy))
            .isInstanceOf(ReinstatementPeriodExpiredException.class);
    }
}
```

---

## 9.3 Integration Testing

### Testing Policy Service with Database

```java
@SpringBootTest
@Testcontainers
@AutoConfigureTestDatabase(replace = Replace.NONE)
class PolicyServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("insurance_test")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private PolicyService policyService;

    @Autowired
    private PolicyRepository policyRepository;

    @Autowired
    private CustomerRepository customerRepository;

    @BeforeEach
    void setUp() {
        // Setup test data
        Customer customer = CustomerFixture.createValidCustomer();
        customerRepository.save(customer);
    }

    @Test
    @DisplayName("Should issue policy and persist to database")
    void shouldIssuePolicyAndPersist() {
        // Given
        IssuePolicyRequest request = IssuePolicyRequest.builder()
            .customerId("CUST-001")
            .productCode("MOTOR_COMP")
            .sumInsured(new Money("50000", "USD"))
            .effectiveDate(LocalDate.now().plusDays(1))
            .build();

        // When
        Policy policy = policyService.issuePolicy(request);

        // Then
        assertThat(policy.getPolicyNumber()).isNotNull();
        assertThat(policy.getStatus()).isEqualTo(PolicyStatus.ISSUED);

        // Verify persistence
        Optional<Policy> persisted = policyRepository.findById(policy.getId());
        assertThat(persisted).isPresent();
        assertThat(persisted.get().getPolicyNumber()).isEqualTo(policy.getPolicyNumber());
    }

    @Test
    @DisplayName("Should create endorsement and update policy")
    void shouldCreateEndorsementAndUpdatePolicy() {
        // Given
        Policy existingPolicy = createActivePolicy();

        EndorsementRequest request = EndorsementRequest.builder()
            .policyId(existingPolicy.getId())
            .type(EndorsementType.SUM_INSURED_CHANGE)
            .newSumInsured(new Money("75000", "USD"))
            .effectiveDate(LocalDate.now())
            .build();

        // When
        Endorsement endorsement = policyService.createEndorsement(request);

        // Then
        Policy updatedPolicy = policyRepository.findById(existingPolicy.getId()).get();
        assertThat(updatedPolicy.getSumInsured().getAmount())
            .isEqualByComparingTo("75000");
        assertThat(updatedPolicy.getVersion()).isEqualTo(2);
    }
}
```

### Testing Event Publishing

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"policy.events"})
class PolicyEventPublisherIntegrationTest {

    @Autowired
    private PolicyService policyService;

    @Autowired
    private EmbeddedKafkaBroker embeddedKafka;

    private Consumer<String, String> consumer;

    @BeforeEach
    void setUp() {
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps(
            "test-group", "true", embeddedKafka);
        consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        consumer = new DefaultKafkaConsumerFactory<>(consumerProps,
            new StringDeserializer(), new StringDeserializer())
            .createConsumer();
        consumer.subscribe(Collections.singletonList("policy.events"));
    }

    @Test
    @DisplayName("Should publish PolicyIssued event when policy is issued")
    void shouldPublishPolicyIssuedEvent() {
        // Given
        IssuePolicyRequest request = createValidIssuePolicyRequest();

        // When
        Policy policy = policyService.issuePolicy(request);

        // Then
        ConsumerRecords<String, String> records = KafkaTestUtils.getRecords(consumer);
        assertThat(records.count()).isGreaterThan(0);

        ConsumerRecord<String, String> record = records.iterator().next();
        PolicyIssuedEvent event = objectMapper.readValue(record.value(), PolicyIssuedEvent.class);

        assertThat(event.getEventType()).isEqualTo("policy.issued");
        assertThat(event.getData().getPolicyId()).isEqualTo(policy.getId().toString());
        assertThat(event.getData().getPolicyNumber()).isEqualTo(policy.getPolicyNumber());
    }
}
```

---

## 9.4 End-to-End Testing

### API E2E Tests

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

### Claims Journey E2E Test

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

## 9.5 Test Data Management

### Test Fixtures

```java
public class PolicyFixture {

    public static Policy createActivePolicy() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber("POL-2024-000001")
            .customerId(CustomerId.of("CUST-001"))
            .productCode("MOTOR_COMP")
            .status(PolicyStatus.ACTIVE)
            .effectiveDate(LocalDate.now().minusMonths(6))
            .expiryDate(LocalDate.now().plusMonths(6))
            .sumInsured(new Money("50000", "USD"))
            .totalPremium(new Money("1200", "USD"))
            .build();
    }

    public static Policy createExpiredPolicy() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber("POL-2023-000001")
            .status(PolicyStatus.EXPIRED)
            .effectiveDate(LocalDate.now().minusYears(2))
            .expiryDate(LocalDate.now().minusYears(1))
            .build();
    }

    public static Policy.PolicyBuilder policyBuilder() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber(generatePolicyNumber())
            .status(PolicyStatus.DRAFT);
    }
}

public class ClaimFixture {

    public static Claim createOpenClaim(Policy policy) {
        return Claim.builder()
            .id(ClaimId.generate())
            .claimNumber("CLM-2024-000001")
            .policyId(policy.getId())
            .customerId(policy.getCustomerId())
            .status(ClaimStatus.OPEN)
            .lossDate(LocalDate.now().minusDays(3))
            .notificationDate(LocalDate.now())
            .claimedAmount(new Money("5000", "USD"))
            .lossDescription("Rear-end collision")
            .build();
    }
}
```

### Test Data Factory

```java
@Component
public class TestDataFactory {

    private final Faker faker = new Faker();

    public Customer randomCustomer() {
        return Customer.builder()
            .id(CustomerId.generate())
            .firstName(faker.name().firstName())
            .lastName(faker.name().lastName())
            .email(faker.internet().emailAddress())
            .phone(faker.phoneNumber().phoneNumber())
            .dateOfBirth(randomDateOfBirth(25, 65))
            .address(randomAddress())
            .build();
    }

    public MotorRiskDetails randomMotorRisk() {
        return MotorRiskDetails.builder()
            .vehicleMake(faker.options().option("Toyota", "Honda", "Ford", "BMW"))
            .vehicleModel(faker.options().option("Sedan", "SUV", "Coupe"))
            .vehicleYear(faker.number().numberBetween(2015, 2024))
            .engineCapacity(faker.options().option(1500, 2000, 2500, 3000))
            .vehicleValue(randomMoney(20000, 100000))
            .usageType(UsageType.PRIVATE)
            .build();
    }

    private LocalDate randomDateOfBirth(int minAge, int maxAge) {
        int age = faker.number().numberBetween(minAge, maxAge);
        return LocalDate.now().minusYears(age);
    }

    private Money randomMoney(int min, int max) {
        int amount = faker.number().numberBetween(min, max);
        return new Money(String.valueOf(amount), "USD");
    }
}
```

---

## 9.6 Testing Best Practices

### Insurance-Specific Test Scenarios

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INSURANCE TEST SCENARIOS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  QUOTE SERVICE:                                                              │
│  ├── Premium calculation accuracy                                            │
│  ├── Discount/loading application order                                      │
│  ├── Tax calculation by region                                               │
│  ├── Quote expiration handling                                               │
│  └── Product eligibility validation                                          │
│                                                                              │
│  POLICY SERVICE:                                                             │
│  ├── Policy status transitions (state machine)                               │
│  ├── Endorsement premium adjustments                                         │
│  ├── Pro-rata refund calculations                                            │
│  ├── Renewal premium recalculation                                           │
│  ├── Grace period handling                                                   │
│  └── Reinstatement eligibility                                               │
│                                                                              │
│  CLAIMS SERVICE:                                                             │
│  ├── Coverage verification                                                   │
│  ├── Deductible application                                                  │
│  ├── Claim limit checking (per occurrence, aggregate)                        │
│  ├── Waiting period validation                                               │
│  ├── Pre-existing condition exclusion                                        │
│  └── Fraud indicator detection                                               │
│                                                                              │
│  BILLING SERVICE:                                                            │
│  ├── Payment schedule generation                                             │
│  ├── Commission calculation                                                  │
│  ├── Clawback processing                                                     │
│  ├── Failed payment retry logic                                              │
│  └── Refund processing                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Test Coverage Guidelines

| Component | Min Coverage | Focus Areas |
|-----------|-------------|-------------|
| Domain Services | 90% | Business logic, calculations |
| Controllers | 80% | Request validation, response mapping |
| Repositories | 70% | CRUD operations, queries |
| Event Handlers | 85% | Event processing, state changes |
| External Clients | 75% | Error handling, retries |

---

## Key Takeaways

1. **Testing pyramid** guides test distribution (many unit, few E2E)
2. **Fixtures and factories** enable consistent test data
3. **Integration tests** verify component interactions
4. **E2E tests** validate complete user journeys
5. **Domain-specific scenarios** ensure business logic correctness

---

## Exercises

1. Write unit tests for claim reserve calculation
2. Create integration tests for policy renewal workflow
3. Design E2E test for quote-to-policy conversion with payment

---

[← Previous Lesson](../08-development-practices/README.md) | [Next Lesson: DevOps & Deployment →](../10-devops-deployment/README.md)
