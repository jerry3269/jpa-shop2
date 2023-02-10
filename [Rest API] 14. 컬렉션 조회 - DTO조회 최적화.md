# 컬렉션 조회 - DTO 조회 최적화

- 기존 V4방식에서 1 + N번 쿼리가 나가는 것을 1 + 1번 쿼리가 나가도록 하는 것이다.

- In절을 이용한다.

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

- in 키워드를 사용하고 ordersIds라는 매개변수를 받는다. 

- ordersIds는 Long 타입의 List 자료구조로, 다수의 Id 값을 가지고 있다. 

- 이처럼 IN 키워드를 사용하여 N + 1 문제를 해결할 수 있다.

<br><Br>

- 위의 findAllByDto() 메서드는 총 2번의 쿼리를 발행한다.

- findOrders()에서 1번, orderItems를 찾아오는 JPQL에서 1번이 발행된다. 

<br><Br>

다음 V6에서는 단 한번의 쿼리로 Orders를 조회하는 방법을 알아 보겠다.ㄴㄴ