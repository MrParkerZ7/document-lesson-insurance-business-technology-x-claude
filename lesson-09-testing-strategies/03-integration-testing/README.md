# 03 - Integration Testing

[Back to Lesson 09](../README.md)

---

## Overview

Integration tests verify that components work correctly together. They test the interactions between services and their dependencies such as databases, message queues, and external APIs using test containers or embedded services.

---

## Testing Policy Service with Database

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

---

## Testing Event Publishing

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

## Key Integration Testing Patterns

### Test Containers
Use Docker containers to run real databases, message brokers, and other services:
- PostgreSQL, MySQL for database testing
- Kafka, RabbitMQ for messaging testing
- Redis for caching testing

### Embedded Services
Use embedded alternatives for faster tests:
- H2 for database testing (when possible)
- EmbeddedKafka for Kafka testing
- WireMock for external API mocking

---

## Best Practices

1. **Use real dependencies** when testing integration points
2. **Clean up test data** between tests with `@BeforeEach`
3. **Use dynamic property sources** for container-based tests
4. **Test both success and failure scenarios**
5. **Verify side effects** (database changes, events published)

---

## Navigation

[Previous: Unit Testing](../02-unit-testing/README.md) | [Back to Lesson 09](../README.md) | [Next: End-to-End Testing](../04-end-to-end-testing/README.md)
