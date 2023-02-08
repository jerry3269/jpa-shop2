# 주문조회 V2 - 엔티티를 DTO로 변환

<br><br>

엔티티를 외부에 노출하면 안된다. 

V2에서는 엔티티를 조회하여 DTO로 변환한다.

```java
    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDto> orderV2(){
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(Collectors.toList());

        return result;
    }

    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate; //주문시간
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto(Order order) {
            this.orderId = order.getId();
            this.name = order.getMember().getName();
            this.orderDate = order.getOrderDate();
            this.orderStatus = order.getStatus();
            this.address = order.getDelivery().getAddress();
        }
    }
```

- 엔티티를 V1과같이 조회하여 자바에서 제공하는 stream을 이용하여 엔티티를 DTO로 변환 하였다.

- 해당 요청에서는 최악의 경우 쿼리가 총 1 + N + N번 나가게 된다.(V1과 쿼리 수 동일)
    - order 조회 1번(order 조회 결과 수가 N이 됨)
    - order -> member 지연로딩 조회 N번
    - order -> delivery 지연로딩 N번
    - 지연로딩에서 영속성 컨텍스트에 있는 상태이면, 쿼리를 보내지 않는다.
    - 따라서 최악의 경우 1 + N + N번 쿼리가 나간다.

    

 
