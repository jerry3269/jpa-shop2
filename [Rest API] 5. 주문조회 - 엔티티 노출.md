# 주문조회 - 엔티티 직접 노출

<br><br>

```java
@GetMapping("/api/v1/simple-orders")
    public List<Order> orderV1(){
        List<Order> orders = orderRepository
                                .findAllByString(new OrderSearch());
        return orders;
    }
```
다음과 같이 엔티티를 직접 반환 하였다.

<br><br>

위의 코드를 그냥 호출하면 오류가 발생한다.

그 이유는 무한 루프에 빠지기 때문이다..

<br><br>

여기서 왜 무한 루프에 빠지게 될까?

바로 order가 Member, Delivery, Order_Item과 양방향 연관관계를 맺고 있기 때문이다.

Order에서 Member, Delivery, Order_Item를 조회하는데 3개의 객체에서도 Order를 참조하고 있기 때문에 무한 루프가 발생하는 것이다.

<br><br>

사실 프록시라는 것을 공부해보았다면 더 의문이 생길 것이다.

우리는 지금까지 모든 연관관계를 LAZY로 설정하였기 때문이다.

Order엔티티를 반환해도 LAZY 전략이므로 실제 Member객체가 아닌 프록시 객체가 들어가게 된다.

그렇다면 양방향 관계가 맺어지지 않고, 무한루프에 빠지지 않는다라는 오해를 하게 된다.

<br><br>

나도 실제로는 그렇게 동작해서 무한루프에 빠지지 않는다고 생각했다.

하지만 실상은 Jackson 라이브러리가 엔티티를 Json데이터로 보내기위해 해당 엔티티에 접근을 하게 될때 초기화를 자동으로 실행한다는 것이다. 

따라서 무한루프가 돌게 된다.

<br><br>


# 1. @JsonIgnore

무한루프를 막기 위해서 Member, Delivery, Order_Item클래스에 있는 Order에 대한 참조에 `@JsonIgnore` 어노테이션을 추가하였다. 이러면 해당 필드는 Json데이터로 반환되지 않기 때문에 무한루프에서 빠질수 있다.

<br><br>

하지만 그렇게 변경하여도 실행하여 보면 

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer ....
```

다음과 같은 오류가 발생하는 것을 볼 수있다.


오류문에 `bytebuddy`라는 것을 볼 수 있는데, 이는 프록시 객체로 인하여 오류가 발생하였다는 뜻이다.

<br><br>

나는 여기서 한번더 의문이 들었다.

위에서 분명 `Jackson 라이브러리가 엔티티를 Json데이터로 보내기위해 해당 엔티티에 접근을 하게 될때 초기화를 자동으로 실행한다는 것이다. `라고 말을 하였다.

즉 초기화가 되었는데 왜 프록시 객체로 인하여 오류가 발생했을까?

<br><br>

이유는 간단하다.

초기화를 하더라도 해당 객체는 실제객체가 아니라 프록시 객체이기 때문이다. 

프록시 객체가 초기화 되어있는가 아닌가의 차이 일뿐, 결국 프록시 객체라는 것은 변함이 없다.

<br><br>

아마 JSON으로 프록시 객체는 넘어가지 못하는 것 같다.

<br><br>

# 2. hibernate5Module

이럴때 hibernate5Module을 사용한다.

해당 하이버네이트 모듈은 프록시 객체를 JSON으로 읽을수 있도록 도와준다.

`build.gradle`에 다음을 추가하자.

```java
	implementation 'com.fasterxml.jackson datatype:jackson-datatype-hibernate5'
```

그리고 애플리케이션 코드 메인메서드 밑에 다음을 추가하자.

```java
@Bean
	Hibernate5Module hibernate5Module(){
		Hibernate5Module hibernate5Module = new Hibernate5Module();
		return hibernate5Module;
	}
```
그러면 더이상 프록시 객체로인한 에러가 발생하지 않고, 프록시 객체에는 전부 null값이 나오는 것을 볼 수 있다.










