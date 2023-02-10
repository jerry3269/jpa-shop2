# 컬렉션 조회 - 엔티티 직접 노출

- 지금까지 OrderSimpleApiController를 작성하면서 간단한 OrderApiController를 만들어 보았다.

<br>

- Order에는 List<OrderItem> 필드가 있는데 해당 컬렉션 필드는 ToOne관계가 아니라 ToMany 관계이기 때문에 다루지 않았었다.

<br>

- 지금부터는 Order에 컬렉션 필드를 추가하여 조회하는 방법을 알아보고자 한다.


# 1. 코드

```java
@GetMapping("/api/v1/orders")
    public List<Order> ordersv1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getAddress();
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.stream().forEach(o -> o.getItem().getName());
        }
        return all;
    }
```

- 다음과 같이 작성하였다.

<br>

- Jackson이 프록시 객체를 JSON으로 바꾸지 못하기 때문에 Hibernate5Module을 등록하여 프록시 객체를 JSON으로 읽을 수 있게 만들어 주었다.

- 원래 참조를 LAZY로 설정하고, Hibernate5Module을 등록하게 되면 `LAZY = null` 로 처리 된다.

- 실제 객체데이터가 들어가도록 하기 위해서는 LAZY를 강제 초기화 해야 하는데 강제초기화 방법에는 2가지 방법이 있다.
    - `order.getMember().getName();` 와 같이 메소드를 직접 호출하여 강제 초기화
    - `Hibernate5Module`등록시 `hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);` 코드 추가.

<br>

- orderItem, item 관례를 코드와 같이 직접 초기화 하면 Hibernate5Module 설정에 의해 엔티티를 JSON으로 생성한다. 따라서 엔티티에 실제 데이터가 들어가게 된다.

- 양방향 연관관계라면 무한 루프에 걸리지 않게 한 곳에 @JsonIgnore 어노테이션을 추가해야 된다.

# 2. 정리

- 엔티티를 직접 노출하는 방법은 좋은 방법이 아니다.
- 엔티티가 변하면 API스펙이 변한다.
- 양방향 연관관계로 인해서 JsonIgnore 어노테이션을 추가해야 한다.

