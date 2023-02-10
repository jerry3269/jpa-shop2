# 컬렉션 조회 - DTO로 변환

<br>

- 이전 V1의 엔티티 노출 문제를 해결하기 위해 DTO로 변환하여 반환해야 한다.
- 이전과 같은 방식으로 변환하되 DTO 내부에 엔티티를 두어서는 안된다.


<br>

# 1. 코드 

```java
@GetMapping("/api/v2/orders")
    public List<OrderDto> ordersV2(){
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());

        return result;
    }

//OrderDto
@Data
    static class OrderDto {

        private Long orderId;
        private String name;
        private LocalDateTime orderDate; //주문시간
        private OrderStatus orderStatus;
        private Address address;

        private List<wnd> orderItems;

        public OrderDto(Order order) {
            this.orderId = order.getId();
            this.name = order.getMember().getName();
            this.orderDate = order.getOrderDate();
            this.orderStatus = order.getStatus();
            this.address = order.getDelivery().getAddress();
            this.orderItems = order.getOrderItems().stream()
                    .map(o -> new OrderItemDto(o))
                    .collect(toList());
        }
    }

    //OrderItemDto
    @Data
    static class OrderItemDto {
        private String itemName;
        private int orderPrice;
        private int count;
        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
    }
```

- 이전과 같은 방식으로 DTO를 작성하였다.

<Br>

- 여기서 주의 할 점은 OrderDto 내부에는 OrderItem컬렉션을 가지고 있어야 하는데, 이 컬렉션 또한 OrderItem 엔티티를 직접 노출하면 안된다는 것이다.

<Br>

- 내부에 OrderItemDto을 하나 더 만들어서 OrderItem 대신 DTO를 사용하도록 설계하였다. 이때 생성자로, LAZY를 강제 초기화 해주었다.

<br>

# 2. 정리

- LAZY방식이기 때문에 지연로딩으로 인해 너무 많은 SQL이 실행된다.

- SQL 실행 수
    - order 1번
    - member , address N번(order 조회 수 만큼)
    - orderItem N번(order 조회 수 만큼)
    - item N번(orderItem 조회 수 만큼)
