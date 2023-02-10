# 컬렉션 조회 - DTO로 직접 조회

<br>



# 1. 코드

- DTO 변환 조회 메서드의 코드는 다음과 같다.

```java
@GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o)).collect(Collectors.toList());
        return result;
}
```

<br><br>

## 1. DTO 생성

```java
@Data
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```
 

- 위의 DTO의 핵심은 생성자에 있다.

- 생성자를 살펴보면 컬렉션 필드인 orderItems가 생략되어 있다.

- 데이터 뻥튀기 문제를 해결하기 위해, 후처리 과정으로 직접 주입하는 방식을 사용할 예정이다. 

 
<br><br>
 

## 2. 컬렉션 필드 DTO 생성

```java
@Data
public class OrderItemQueryDto {

    @JsonIgnore
    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;
    
    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```
 
 <br> <br>

## 3. DTO를 이용한 조회 


DTO로 조회하는 별도의`Repository` 생성
```java
private List<OrderQueryDto> findOrders() {
    return em.createQuery(
        "select new jpabook.jpabook.dto.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.adderss) " +
        "from Order o " +
        "join o.member m " +
        "join o.delivery d", OrderQueryDto.class).getResultList();
}
```
 
-  컬렉션 필드인 OrderItems의 값이 빠져있음.

<br><br>
 
## 4. 컬렉션 필드 채워 넣기 

 
`Repository`
```java
public List<OrderQueryDto> findOrderQueryDtos() {
        
    // result는 컬렉션 필드(orderItems)가 비어있는 상태
    List<OrderQueryDto> result = findOrders();

    // 컬렉션 필드를 직접 채우는 과정
    result.forEach(o -> { 
        List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());            
        o.setOrderItems(orderItems);
    });

    return result;
}
```
 <br>

`Repository`
```java
private List<OrderItemQueryDto> findOrderItems(Long orderId) {
    return em.createQuery(
        "select new jpabook.jpabook.dto.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
            "from OrderItem oi " +
            "join oi.item i " +
            "where oi.order.id = :orderId", OrderItemQueryDto.class)
        .setParameter("orderId", orderId)
        .getResultList();
}
```
 <br>



-> OrderQueryDto 객체에는 컬렉션 필드인 OrderItems 값이 비어있음.

 
-> 반복문으로 OrderQueryDto 객체를 탐색하며 findOrderItems() 메서드를 호출

-> findOrderItems() 메서드는 OrderQueryDto 객체의 Id 값을 이용하여 DB로부터 OrderItem의 값을 조회한다.

 

  <br> <br>

## 5. Repository 전체 코드
 

 
```java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderQueryDto> findOrderQueryDtos() {

        List<OrderQueryDto> result = findOrders(); 
        result.forEach(o -> { 
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });
        return result;
    }

    private List<OrderQueryDto> findOrders() {
        return em.createQuery(
            "select new jpabook.jpabook.dto.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.adderss) " +
            "from Order o " +
            "join o.member m " +
            "join o.delivery d", OrderQueryDto.class).getResultList();
    }

    private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
            "select new jpabook.jpabook.dto.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                "from OrderItem oi " +
                "join oi.item i " +
                "where oi.order.id = :orderId", OrderItemQueryDto.class)
            .setParameter("orderId", orderId)
            .getResultList();

    }
}
```

# 2. 정리
 

- 위의 코드는 DB로부터 받아온 OrderQueryDto 객체에 비어있는 OrderItem 객체를 채워 넣는 과정을 거친다.

- findOrderQueryDtos() 메서드를 실행하면 총 몇 번의 쿼리가 발행될지 생각해보자. 

 
- 우선 findOrders() 메서드에서 쿼리가 1회 발행된다.

- 그리고 findOrderItems() 메서드에서 쿼리가 N번 발행된다.

- findOrderItems() 메서드는 반복문이 돌아가는 만큼 실행되기 때문이다.

 
- 즉, 결과적으로 N + 1 문제가 발생한다.


- 대부분 N + 1 문제는 WHERE 절에서 비교 또는 검색 조건이 하나씩 처리되기 때문에 발생한다.

<br> <br>

# 3. V5

- V4에서 발생한 1 + N 문제를 해결하기 위해 JPQL에 IN 절로 한번에 가져온다.

```java
public List<OrderQueryDto> findAllByDto() {
    List<OrderQueryDto> result = findOrders();
    List<Long> orderIds = result.stream().map(o -> o.getOrderId()).collect(Collectors.toList());

    List<OrderItemQueryDto> orderItems = em.createQuery(
            "select new jpabook.jpabook.dto.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                    "from OrderItem oi " +
                    "join oi.item i " +
                    "where oi.order.id " +
                    "in :orderIds", OrderItemQueryDto.class)
            .setParameter("orderIds", orderIds)
            .getResultList();

    // orderItems를 Map으로 변환 (사용하기 쉽게하기 위함)
    Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream()
            .collect(Collectors.groupingBy(orderItemQueryDto -> orderItemQueryDto.getOrderId()));

    // result의 orderItems 필드를 채우는 과정
    result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

    return result;
}
```

위의 코드는 SQL이 몇번 발생할까?

<br><br>

findOrders()에서 fetch join 쿼리 1번, 

orderItems를 조회하는 쿼리 1번으로 총 2번이다.

IN절을 사용하면 쿼리를 최적화 할 수 있다.

하지만 여전히 2번의 쿼리가 나간다. 이를 해결하기 위한 방법은 V5에서 설명하겠다.