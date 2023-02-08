# 주문조회 V3 - 페치 조인 최적화

<br><Br>

```java
    @GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> orderV3(){
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(Collectors.toList());

        return result;
    }
```
다음과 같이 작성하고 OrderRepository에 다음 코드를 추가 한다.

```java
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery("select o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d", Order.class)
                .getResultList();
    }
```

fetch join을 이용사여 연관된 테이블을 쿼리 한방에 조회한다.

<br><Br>

기존에는 최악의 경우 1 + N + N 번의 쿼리가 나갔는데 

fetch join을 사용하면 1번의 쿼리만이 나가게 된다.

```sql
    select
        order0_.order_id as order_id1_6_0_,
        member1_.mamber_id as mamber_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        order0_.delivery_id as delivery4_6_0_,
        order0_.member_id as member_i5_6_0_,
        order0_.order_date as order_da2_6_0_,
        order0_.status as status3_6_0_,
        member1_.city as city2_4_1_,
        member1_.street as street3_4_1_,
        member1_.zipcode as zipcode4_4_1_,
        member1_.name as name5_4_1_,
        delivery2_.city as city2_2_2_,
        delivery2_.street as street3_2_2_,
        delivery2_.zipcode as zipcode4_2_2_,
        delivery2_.status as status5_2_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.mamber_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id
```

다음과 같은 쿼리가 나가게 된다.

쿼리는 한번 나가게 되지만 DTO로 API 스펙을 살펴 보면

```java
    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;
```
다음과 같다.

<br><Br>

하지만 API의 스펙과는 상관없는 필드들의 정보까지 전부 가져오는 것을 볼 수 있다.

V4에서는 DTO의 스펙에 맞추어 원하는 필드만을 가져와보는 방법을 알아보도록 하겠다.



