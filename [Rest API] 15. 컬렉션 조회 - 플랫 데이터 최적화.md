# 1. 컬렉션 조회 - 플랫 데이터 최적화

<br><Br>

- V5는 괜찮게 최적화 되었지만 여전히 쿼리가 2번 나간다. 이를 1번만 나가게 하는 방법을 알아보겠다.

- 방법은 간단하다. 필요한 모든 데이터를 담은 DTO를 새로 만들고, 해당 데이터를 모두 가져와서 우리가 원래 반환 하려는 DTO로 변환해주면 된다.

<br><Br>

## 1. DTO 생성
- 컬렉션 필드의 값을 채워 넣는 과정을 없애기 위해서 새로운 DTO가 필요하다. 

```java
@Data
public class OrderFlatDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    // 컬렉션 필드 대신 컬렉션 필드의 속성으로 구성
    private String itemName;
    private int orderPrice;
    private int count;
    
    // 생성자 생략
}
```

- 컬렉션 필드를 사용하지 않고, 컬렉션 필드의 속성을 개별 필드로 생성하였다.

- 이와 같은 형태로 구성하는 이유는 일대다 JOIN으로 발생하는 중복 데이터를 모두 받아오기 위해서다. 

 
<br><Br>
 

## 2. JPQL
```java
 public List<OrderFlatDto> findAllByDtoFlat() {
    return em.createQuery(
        "select new jpabook.jpabook.dto.OrderFlatDto(o.id, m.name, o.orderDate, o.status, " +
            "d.adderss, i.name, oi.orderPrice, oi.count) " +
            "from Order o " +
            "join o.member m " +
            "join o.delivery d " +
            "join o.orderItems oi " +
            "join oi.item i", OrderFlatDto.class)
            .getResultList();
}
```

- 위의 JPQL에서 알 수 있듯이, o.orderItems를 JOIN 하였다. 

- 일대다 JOIN을 하였으므로 데이터 중복(= 데이터 뻥튀기)이 발생할 수밖에 없는 상황이다.



<br><Br>


## 3. 중복데이터 제거(grouping by)

- 위의 결과 데이터는 PK(= id) 값이 동일한 객체가 2개씩 있으므로, 영속성 Context의 측면에서 중복 객체다.

- 즉, 해당 JPQL은 중복 데이터를 반환하는 것이다. (이는 일대다 JOIN으로 필수 불가결하다.)

- 그러므로 동일한 PK(= id) 값을 가진 객체끼리 묶어주는 작업을 수행해야 한다. 

 
- 중복 객체를 묶어줄 뿐만 아니라, 기존의 API스펙에 맞는 DTO를 반환해야 한다.(OrderQueryDto)

<br><Br>
 
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

  private final OrderRepository orderRepository;
  private final OrderQueryRepository orderQueryRepository;
  
  
  @GetMapping("/api/v6/orders")
    public List<OrderQueryDto> orderV6 () {
    
    	List<OrderFlatDto> flats = orderQueryRepository.findAllByDtoFlat();
        
        return flats.stream()
            // stream()을 이용하여 OrderFlatDto의 일부를 OrderQueryDto로 변환
        	.collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(), o.getName(),
                        o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                // stream()을 이용하여 OrderFlatDto의 일부를 OrderItemQueryDto로 변환
                mapping(o -> new OrderItemQueryDto(o.getOrderId(), o.getItemName(),
                            o.getOrderPrice(), o.getCount()), toList())
            )).entrySet().stream()
            // 최종적으로 OrderQueryDto와 OrderItemQueryDto를 이용하여 OrderQueryDto 생성
            .map(e -> new OrderQueryDto(e.getKey().getOrderId(),
                e.getKey().getName(), e.getKey().getOrderDate(),
                e.getKey().getOrderStatus(), e.getKey().getAddress(), e.getValue()))
            .collect(toList());
   }
}
```

 
- OrderFlatDto는 컬렉션 필드 대신 컬렉션 필드의 속성으로 구성되어있다.

- findAllByDtoFlat() 메서드에서 OrderFlatDto 객체를 반환하면, 이를 이용하여 후처리 작업을 수행한다.

- 우선 OrderFlatDto의 일부 데이터를 이용하여 OrderQueryDto를 생성한다. 

- 그리고 기존의 컬렉션 필드를 대신했던 컬렉션 필드의 속성 값을 이용하여 OrderItemQueryDto를 생성한다.

- 비어있는 OrderQueryDto의 컬렉션 필드를 OrderItemQueryDto로 채워 넣는다. 

 <br><Br>

- 위의 코드의 핵심은 groupingBy()를 사용한 것이다.

- DB의 입장에서는 중복 데이터가 반환되는 것은 동일하다.

- 하지만, JPA는 groupingBy()를 통해 영속성 Context 상에서 동일한 객체의 값을 하나로 묶는다. 

- 이는 다음과 같은 결과를 JSON으로 반환한다. 

 
```json
[
    {
        "orderId": 11,
        "name": "userB",
        "orderDate": "2022-06-04T16:04:57.769566",
        "orderStatus": "ORDER",
        "address": {
            "city": "Busan",
            "street": "Haeundae",
            "zipcode": "22222"
        },
        "orderItems": [
            {
                "itemName": "Spring1 Book",
                "orderPrice": 20000,
                "count": 3
            },
            {
                "itemName": "Spring2 Book",
                "orderPrice": 40000,
                "count": 4
            }
        ]
    },
    {
        "orderId": 4,
        "name": "userA",
        "orderDate": "2022-06-04T16:04:57.65529",
        "orderStatus": "ORDER",
        "address": {
            "city": "Seoul",
            "street": "Gangnam",
            "zipcode": "11111"
        },
        "orderItems": [
            {
                "itemName": "JPA1 Book",
                "orderPrice": 10000,
                "count": 1
            },
            {
                "itemName": "JPA2 Book",
                "orderPrice": 20000,
                "count": 2
            }
        ]
    }
]
```
 

- 이처럼 동일한 객체를 하나로 묶어주고, 동일 객체에 한해서 다른 데이터는 배열로 묶어서 표현한다.

 
<br><Br>
 

## 4. groupingBy()의 기준 설정 - @EqualsAndHashCode 
- groupingBy() 메서드는 SQL의 Group By와 동일한 기능을 수행한다.

- 다만, 객체 중점적이라는 차이만 있을 뿐이다. 

 

- 그렇다면 groupingBy()는 어떤 값을 기준으로 객체를 묶는 것일까?

- groupingBy()가 적용되는 대상을 살펴보면 OrderQueryDto 객체인 것을 확인할 수 있다. 

그러므로 groupingBy()의 기준은 OrderQueryDto 클래스에 다음과 같이 설정해준다.

 
```java
@Data
@EqualsAndHashCode(of = "orderId") // v6 메서드의 groupingBy 에서 사용하는 묶음 기준
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;
    
    // 생성자 생략
}
```

- 위의 코드를 보면 @EqualsAndHashCode라는 어노테이션이 사용되었다.

- 그리고 해당 어노테이션의 매개변수로 "orderId"가 들어있는 것을 확인할 수 있다.

- 이는 OrderQueryDto 객체 간의 비교를 수행할 때, orderId 필드의 값을 기준으로 비교를 수행한다는 의미다.

- 즉, orderId 값이 동일한 객체끼리 groupingBy()를 통해 하나의 객체로 묶이는 것이다. 

 
<br><Br>
 

# 2.정리
- 장점
    - 쿼리 발행 회수가 1회다. 
    - 데이터가 많지 않으면 작업 수행 속도가 빠르다. 
- 단점
    - 일대다 JOIN으로 인하여 조회 결과에 중복 데이터가 발생된다. 

    - 그러므로 Order를 기준으로 페이징이 불가능하다. (= 데이터 뻥튀기, 중복 데이터 발생)

    - 애플리케이션 측면세어 보면 후처리 작업이 크다.

<br><Br>

# 3. 딜레마

사실 V3과 V6은 뚜렷한 성능 차이는 볼 수없다. V3으로 페치조인으로 개선을 하였을 때, 성능이 잘 나오지 않는다면, 캐시서버 증설을 생각해보아야 한다. 그 이유는 V3로 거의 모든 조회 성능이 개선되기 때문이다.

이미 V3로 성능이 잘 나오지 않는다면, 캐시서버 증설을 고려해야 될 단계이다.

또한 가장 큰 문제가, 바로 애플리케이션의 후처리 작업이다. V3(페치조인 최적화) 까지는 코드가 매우 단순하고 유연성이 매우 높았다. 반면에 DTO로 조회하는 V4, V5, V6는 가면 갈수록 코드가 매우 복잡해지고 나중에는 DTO를 DTO로 변환하는 복잡한 후처리 작업까지 애플리케이션 단계에서 수행해야 한다. 이렇게 되면 유지보수가 어려워지는 문제가 있다.

실무에서는 보통 페치조인으로 해결되면 V3를 사용하고, 페이징을 해야된다면 V5버전을 사용한다.

V6는 후처리 작업이 너무 많고, 유지보수가 어렵다.


<br><Br>



 