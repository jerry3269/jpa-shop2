# 조회용 샘플

```java
package jpabook.jpashop;


import jpabook.jpashop.domain.*;
import jpabook.jpashop.domain.item.Book;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManager;

@Component
@RequiredArgsConstructor
public class InitDb {

    private final InitService initService;

    @PostConstruct
    public void init(){
        initService.dbInit1();
        initService.dbInit2();
    }


    @Component
    @RequiredArgsConstructor
    @Transactional
    static class InitService{

        private final EntityManager em;

        public void dbInit1(){
            Member member = createMember("userA", "서울", "1", "1111");
            em.persist(member);

            Book book1 = createBook("JPA1 BOOK", 10000, 100);
            em.persist(book1);

            Book book2 = createBook("JPA2 BOOK", 20000, 200);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 10000, 1);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 20000, 2);

            Delivery delivery = createDelivery(member);
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        }
        public void dbInit2(){
            Member member = createMember("userB", "경기", "2", "2");
            em.persist(member);

            Book book1 = createBook("SPRING1 BOOK", 20000, 300);
            em.persist(book1);

            Book book2 = createBook("SPRING2 BOOK", 40000, 400);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 20000, 3);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 40000, 4);

            Delivery delivery = createDelivery(member);
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        }


        private static Delivery createDelivery(Member member) {
            Delivery delivery = new Delivery();
            delivery.setAddress(member.getAddress());
            return delivery;
        }

        private static Book createBook(String JPA1_BOOK, int price, int stockQuantity) {
            Book book1 = new Book();
            book1.setName(JPA1_BOOK);
            book1.setPrice(price);
            book1.setStockQuantity(stockQuantity);
            return book1;
        }

        private static Member createMember(String userA, String 서울, String street, String zipcode) {
            Member member = new Member();
            member.setName(userA);
            member.setAddress(new Address(서울, street, zipcode));
            return member;
        }
    }
}

```

다음과 같이 조회용 샘플 코드를 만들었다.

<Br><Br>

@PostConstruct는 스프링 컨테이너에 컴포넌트가 모두 등록되고 나면 실행되도록 설계되었기 때문에

<Br><Br>

애플리케이션을 실행시킬때마다 해당 샘플이 들어가도록 설정하였다.

이때 application.xml파일에 JPA설정을 다음과 같이 변경하였다.

```xml
      ddl-auto: create
```
<Br><Br>

update로 하면 샘플 데이터가 지속적으로 들어가기 때문에 애플리케이션 실행 시점에 모든 테이블을 드랍하고 샘플 데이터가 들어가도록 설정 하였다.

해당 데이터에 대한 테이블은 다음과 같다.


<Br><Br>


`SELECT * FROM MEMBER;`
|MAMBER_ID|  	CITY | 	STREET | 	ZIPCODE | 	NAME  |
|:--:|:--:|:--:|:--:|:--:|
|1	|서울|	1|	1111|	userA|
|8	|경기|	2|	2|	userB|
(2 행, 0 ms)

<Br><Br>

`SELECT * FROM ORDERS ;`
|ORDER_ID|  	ORDER_DATE|  	STATUS|  	DELIVERY_ID  	|MEMBER_ID|  
|:--:|:--:|:--:|:--:|:--:|
|4|	2023-02-07| 19:07:27.355221|	ORDER|	5|	1|
|11|	2023-02-07| 19:07:27.390179|	ORDER|	12|	8|
(2 행, 0 ms)

<Br><Br>

`SELECT * FROM ITEM ;`
|DTYPE| 	ITEM_ID|  	NAME|  	PRICE|  	STOCK_QUANTITY|  	ARTIST|  	ETC|  	AUTHOR|  	ISBN|  	ACTOR|  	DIRECTOR|  
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|B|	2|	JPA1 BOOK|	10000|	99|	null|	null|	null|	null|	null|	null|
|B|	3|	JPA2 BOOK|	20000|	198|	null|	null|	null|	null|	null|	null|
|B|	9|	SPRING1 BOOK|	20000|	297|	null|	null|	null|	null|	null|	null|
|B|	10|	SPRING2 BOOK|	40000|	396|	null|	null|	null|	null|	null|	null|
(4 행, 0 ms)

<Br><Br>

`SELECT * FROM ORDER_ITEM ;`
|ORDER_ITEM_ID|  	COUNT|  	ORDER_PRICE|  	ITEM_ID|  	ORDER_ID|  
|:--:|:--:|:--:|:--:|:--:|
|6|	1|	10000|	2|	4|
|7|	2|	20000|	3|	4|
|13|	3|	20000|	9|	11|
|14|	4|	40000|	10|	11|
(4 행, 5 ms)

<Br><Br>

`SELECT * FROM DELIVERY ;`
|DELIVERY_ID|  	CITY|  	STREET|  	ZIPCODE|  	STATUS|  
|:--:|:--:|:--:|:--:|:--:|
|5|	서울|	1|	1111|	null|
|12|	경기|	2|	2|	null|
(2 행, 0 ms)

<Br><Br>