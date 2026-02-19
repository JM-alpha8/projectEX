# 07. Test Strategy

본 프로젝트는 핵심 비즈니스 로직의 안정성과 데이터 정합성을 보장하기 위해
단위 테스트(Unit Test)와 통합 테스트(Integration Test)를 구분하여 작성하였다.

---

## 1. 테스트 환경

- JUnit 5
- Mockito
- SpringBootTest
- Testcontainers (선택)
- H2 (또는 MySQL Test DB)
- @Transactional 기반 롤백 전략

---

## 2. 단위 테스트 (Unit Test)

### 목적

- 비즈니스 로직의 정확성 검증
- 외부 의존성(DB, Redis, 외부 API) 없이 테스트
- 빠른 실행 속도

---

### 예시: 주문 생성 서비스 테스트

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private ProductRepository productRepository;

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void 재고가_충분하면_주문이_성공한다() {
        // given
        Product product = new Product(1L, "맥북", 2000000, 10);
        when(productRepository.findById(1L)).thenReturn(Optional.of(product));

        // when
        orderService.createOrder(1L, 2);

        // then
        assertEquals(8, product.getStockQty());
        verify(orderRepository).save(any(Order.class));
    }
}
```

---

## 3. 통합 테스트 (Integration Test)

### 목적

- DB와 실제 연동 확인
- 트랜잭션 동작 검증
- Repository + Service 계층 통합 검증

---

### 예시: 주문 생성 통합 테스트

```java
@SpringBootTest
@Transactional
class OrderIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Autowired
    private ProductRepository productRepository;

    @Test
    void 주문_생성_후_DB에_저장된다() {
        // given
        Product product = productRepository.save(
            new Product("맥북", 2000000, 10)
        );

        // when
        orderService.createOrder(product.getId(), 1);

        // then
        Product updated = productRepository.findById(product.getId()).get();
        assertEquals(9, updated.getStockQty());
    }
}
```

---

## 4. 예외 테스트

이커머스 시스템에서 가장 중요한 예외는 다음과 같다:

- 재고 부족
- 결제 실패
- 인증 실패

### 예시: 재고 부족 테스트

```java
@Test
void 재고가_부족하면_예외를_던진다() {
    Product product = new Product(1L, "맥북", 2000000, 1);
    when(productRepository.findById(1L)).thenReturn(Optional.of(product));

    assertThrows(OutOfStockException.class,
        () -> orderService.createOrder(1L, 2));
}
```

---

## 5. 동시성 테스트 전략

재고 동시성 문제는 이커머스의 핵심 이슈이다.

테스트 전략:

- 멀티스레드 환경에서 동시에 주문 요청
- 재고가 음수가 되지 않는지 확인
- Redis Lock 적용 여부 검증

### 개념적 예시

```java
@Test
void 동시_주문_시_재고가_음수가_되지_않는다() throws InterruptedException {
    int threadCount = 10;
    ExecutorService executor = Executors.newFixedThreadPool(threadCount);

    for (int i = 0; i < threadCount; i++) {
        executor.submit(() -> {
            orderService.createOrder(1L, 1);
        });
    }

    executor.shutdown();
    executor.awaitTermination(5, TimeUnit.SECONDS);

    Product product = productRepository.findById(1L).get();
    assertTrue(product.getStockQty() >= 0);
}
```

---

## 6. 테스트 설계 의도

1. 단위 테스트는 빠른 피드백을 제공한다.
2. 통합 테스트는 실제 환경과의 정합성을 보장한다.
3. 예외 테스트는 시스템 안정성을 확보한다.
4. 동시성 테스트는 실무 환경에서 발생 가능한 문제를 사전에 검증한다.

---

## 7. 향후 개선 방향

- Testcontainers를 활용한 실제 MySQL 테스트
- Redis Embedded 환경 구성
- API 단위 MockMvc 테스트 추가
- CI 환경에서 자동 테스트 실행 (GitHub Actions)
