# 주문조회 V4 - JPA에서 DTO로 바로 조회

<bR><Br>

- V4: JPA에서 DTO로 바로 조회하여 Select절에서 원하는 데이터만 선택하여 조회하는 방법을 알아보겠다.

```java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```
Controller에 다음 코드를 추가한다.

<bR><Br>

이전 V3와 다른점이 보일 것이다.

바로 기존 DTO가 아니라 새로운 DTO로 반환한다.

<bR><Br>

또한, 참조하는 레포지토리도 변경되었다.

<bR><Br>

또, 엔티티를 DTO로 변환하는 작업도 없어진 것을 볼 수 있다.

지금부터 하나하나 설명 하겠다.

우선 새로 만든 레포지토리 이다.

```java

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderSimpleQueryDto.class)
                        .getResultList();
}

```

그리고 새로 만든 DTO이다.

```java
@Data
public class OrderSimpleQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}

```
해당 DTO는 `OrderSimpleQueryRepository`에서 쓰이기 때문에 같은 패키지에 두는 것이 좋다.

 <br><Br>

새로운 점은 기존에는 DTO를 Controller 쪽에 두었다면 이번에는 레포지토리가 있는 패키지 쪽에 두었다는 점이다.

<br><Br>

그 이유는 보는 봐와 같이 `OrderSimpleQueryRepository`에서 DTO를 직접 new해서 사용하기 때문이다. 

<br><Br>

DTO의 스펙으로 직접 쿼리문을 보낼 때는 직접 select문에서 new를 해서 DTO를 생성해 주어야 한다. 

쿼리의 반환타입도 `OrderSimpleQueryDto.class`로 설정해 주었다.

이때 JPQL문을 보면 fetch join이 아니라 그냥 join을 사용한 것을 볼 수 있다.


DTO를 가져올 때는 fetch타입(LAZY, EAGER)과 상관없이 해당테이블에서 필요한 데이터만을 찝어서 가져오기 때문에 쿼리가 한방만 나가게된다.

<br><Br>

그런데 왜 기존 OrderRepository에다 만들지 않고 새로운 레포지토리에 넣었을까?


<br><Br>


바로 다른 DAO와는 성질이 다르기 때문이다.

기존의 DAO는 DB에 쿼리를 보내는 것 밖에 몰랐다.


<br><Br>

하지만 V4의 DAO는 DTO를 알고있다. 즉, DTO를 참조한다.

그 의미는 API스펙이 DAO에 담겨있다는 소리이다.


<br><Br>

보통 API스펙은 Controller쪽에서 담당한다.

하지만 이경우, API 스펙을 DAO가 알고 있기 때문에, 유지보수가 쉽도록 이러한 DAO만을 따로 관리하기 위해서 새로운 레포지토리를 생성한 것이다.


<br><Br>

나가는 sql문을 봐도 굉장히 짧아졌다는 것을 볼 수 있다.

```sql
    select
        order0_.order_id as col_0_0_,
        member1_.name as col_1_0_,
        order0_.order_date as col_2_0_,
        order0_.status as col_3_0_,
        delivery2_.city as col_4_0_,
        delivery2_.street as col_4_1_,
        delivery2_.zipcode as col_4_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.mamber_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id
```

fetch join을 사용한 sql은 다음과 같이 가져오는 필드가 많다.

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

이와 같이 좀더 최적화하여 필드를 가져올수 있다.

<br><Br>


DTO에 맞는 데이터를 DTO클래스로 가져오기 때문에 엔티티를 따로 DTO를 반환하는 작업이 없어졌다.

컨트롤러쪽의 코드가 단순해 졌다.




<br><Br>

## 1) Trade off

하지만 장점만 있을까?


<br><Br>

JPA에서 DTO로 직접 조회하는 것은 Trade off가 발생한다.

DTO로 바로 조회하는 것은 페치조인을 이용하는 것보다 조금 더 최적화 된다.(네트워크 용량 최적화)

또한, 엔티티를 변환하지 않아도 되므로 Controller의 코드가 단순해진다.

<br><Br>

하지만 네트워크 용량 최적화가 되긴 하지만 요새는 네트워크 성능이 매우 좋기 때문에 저정도 필드의 차이는 매우 미비한 수준이다.

또한, sql에서 성능을 좌지우지 하는것은 보통 join문의 수이다.

<br><Br>

위의 두 sql문을 보면 join된 Table의 수는 같은 것을 볼 수 있다.

즉, 별반 차이가 나지 않는 다는 것이다.

<br><Br>

뿐만 아니라, 레포지토리 재사용성이 떨어진다.

API스펙에 맞게 코드를 작성했기 때문에 해당 API스펙과는 다른 곳에서는 사용되지 않는다. 반면에 fetch join으로 가져오는 방식은 모든 필드가 있기 때문에 API스펙에 맞게 DTO로 변환하여 사용할수 있어서 재사용성이 매우 높다.

또한, 다른 DAO와는 성질이 다르기때문에 따로 관리해야 유지보수가 편하다.

이러한 양면성을 가지고 있다.

<br><Br>

따라서, 엔티티를 fetch join으로 가져와서 DTO로 변환하거나, DTO로 바로 조회하는 두 방법은 각각 장단점이 있다. 이는 개발자가 상황에 맞게 더 나은 방법을 선택해야 한다. 

정말 유저들의 조회가 매우 빈번하게 일어나서 해당 API스펙에 맞추어서 좀더 네트웤 용량을 최적화 해야 될 때만 사용하는 것이 좋다.

<Br><br>

## 2) 쿼리 방식 권장 순서


1. 엔티티를 DTO로 변환하는 방법 선택

2. 필요하면 페치 조인을 이용하여 성능 최적화 -> 대부분의 성능 이슈가 해결됨.

3. 그래도 안된다면 DTO로 직접 조회

4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용하여 SQL을 직접 사용

