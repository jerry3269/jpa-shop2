# 컬렉션 조회 - 페이징

<br>

- V3버전은 페치 조인으로 1 + N 문제를 해결하였지만 페이징이 불가능 하다는 단점이 있었다.

<br>

- 페이징이 가능한 V3.1 버전을 만들어 보도록 하겠다.

# 1. 코드

`Controller`
```java
@GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(
        @RequestParam(value = "offset", defaultValue = "0") int offset,
        @RequestParam(value = "limit", defaultValue = "100") int limit) {

        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());

        return result;
    }
```

`OrderRepository`
```java
    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
```

`application.yml`
```java
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

<br>

- 컬렉션을 페치 조하면 일대다 조인이 발생하여 데이터가 예측할 수없이 증가한다.

<br>

- 일대다에서 `일`을 기준으로 페이징하는 것이 목적인데, 데이터는 `다` 기준으로 row가 증가하게된다.

<br>

- Order를 기준으로 페이징하고 싶은데, `다`인 OrderItem을 조인하게 되면 OrderItem 기준이 되어 버린다.

- 이 경우 하이버네이트는 경로 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. -> 서버 장애로 이어질 수 있음

<br>

- V3.1 에서는 ToOne 관계는 모두 페치조인으로 조회하였다. 해당 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.

<br>

- 컬렉션은 지연 로딩으로 조회하도록 하였다.

<br>

- 지연로딩으로 하게 되면 1 + N 문제가 발생하게 되는데 성능 최적화를 위해 `hibernate.default_batch_fetch_size` , `@BatchSize`를 사용하였다.
    - hibernate.default_batch_fetch_size: 글로벌 설정
    - @BatchSize: 개별 최적화
    - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.

<br>

- 개별로 설정하려면 @BatchSize 를 적용하면 된다. (컬렉션은 컬렉션 필드에 엔티티는 엔티티 클래스에 적용)

<br>

# 2. 정리

- 1 + N 에서 1 + 1로 최적화 된다.

<br>

- 조인보다 DB데이터 전송량이 최적화 된다.(Order와 OrderItem을 조인하면 Order가 OrderItem만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)

<br>

- 페이징이 가능하다.

- ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 

<br>

ToOne관계는 페치조인으로 쿼리 수를 줄이고, 나머지는 배치 사이츠로 최적화

<br><br><br><br>


> __참고__ <br>
참고: default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를
선택하는 것을 권장한다. 이 전략을 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를
1000으로 제한하기도 한다. 1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB
에 순간 부하가 증가할 수 있다. 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야
하므로 메모리 사용량이 같다. 1000으로 설정하는 것이 성능상 가장 좋지만, 결국 DB든 애플리케이션이든
순간 부하를 어디까지 견딜 수 있는지로 결정하면 된다.


