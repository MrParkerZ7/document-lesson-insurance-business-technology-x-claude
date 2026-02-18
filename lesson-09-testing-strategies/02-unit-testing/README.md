# 02 - Unit Testing

[Back to Lesson 09](../README.md)

---

## Overview

Unit tests verify individual components in isolation, using mocks for dependencies. In insurance systems, unit tests are essential for validating business logic such as premium calculations, state transitions, and validation rules.

---

## Testing Premium Calculation

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

---

## Testing Policy State Machine

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

## Best Practices for Unit Testing

1. **Use descriptive test names** with `@DisplayName`
2. **Follow the Given-When-Then pattern** for clarity
3. **Use parameterized tests** for testing multiple scenarios
4. **Mock external dependencies** to ensure isolation
5. **Test edge cases** and error conditions

---

## Navigation

[Previous: Testing Pyramid](../01-testing-pyramid/README.md) | [Back to Lesson 09](../README.md) | [Next: Integration Testing](../03-integration-testing/README.md)
