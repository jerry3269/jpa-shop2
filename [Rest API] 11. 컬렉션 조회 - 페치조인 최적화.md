# 컬렉션 조회 - 페치조인 최적화

<br>

- V2에서는 엔티티를 직접 노출하지는 않았지만 지연로딩으로 인해 SQL문이 너무 많이 나가는 문제(1 + N 쿼리)가 발생하였다.

<br>

- 이를 해결하기 위해 fetch join을 사용하여 V3를 만들어 보겠다.

<br>

# 1. 코드


`Controller`
```java
    @GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3(){
        List<Order> orders = orderRepository.findAllWithItem();

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return result;
    }
```

`orderRepository`
```java
 public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i" , Order.class)
                .getResultList();
    }
```

- Dto는 기존의 V2에서 사용된 것을 그대로 사용하였다.

<br>

# 정리

<br>

- 페치조인으로 인해 SQL이 1번 실행된다.

<br>

- `distinct`를 사용한 이유는 해당 조회를 하게되면 1대다 조인이 있으므로 데이터베이스의 row가 증가하게 된다. 따라서 같은 order 엔티티가 중복되어 나타나게 되는데 이를 방지하기 위한 distinct이다.

- JPA의 distinct는 SQL에 distinct기능 + 같은 엔티티가 조회되면 애플리케이션에서 중복을 걸러주는 기능을 한다.

<br>

- 하지만 페치조인이기 때문에 페이징이 불가능하다.

