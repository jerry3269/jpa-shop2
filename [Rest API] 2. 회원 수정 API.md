# 회원 수정 API

<br><Br>

회원 수정은 @PutMapping을 사용한다.
여기서 정리하자면

보통 get방식은 select
<br><Br>
post방식은 insert
<br><Br>
put방식은 update 을 사용한다.
<br><Br>
REST API에서는 put 보통 전체 업데이트를 할때 사용한다.

부분업데이트는 patch나 post를 사용하는 것이 REST API스타일에 맞다.


<br><Br>

```java

 @PutMapping("/api/v2/members/{id}")
    public UpdateMemberResponse updateMemberV2(
            @PathVariable("id") Long id,
            @RequestBody @Valid UpdateMemberRequest request) {
        memberService.update(id, request.getName());
        Member findMember = memberService.findOne(id);
        return new UpdateMemberResponse(findMember.getId(), findMember.getName());
    }

    @Data
    @AllArgsConstructor
    static class UpdateMemberResponse {
        private Long id;
        private String name;
    }

    @Data
    static class UpdateMemberRequest {
        @NotEmpty
        private String name;
    }

```

다음과 같이 작성 하였다.
<br><Br>

코드를 살펴 보자면, 회원 수정API도 DTO로 요청 파라미터를 매핑하였다.
<br><Br>
또한, memberService에 다음과 같은 업데이트 메서드를 추가해주었다.

```java
@Transactional
 public void update(Long id, String name) {
    Member member = memberRepository.findOne(id);
    member.setName(name);
 }
 ```
데이터 수정을 변경감지로 하기 위해서 트랜잭션 내부에서 데이터를 변경하였다.

- 앞선 내용을 다 따라왔다면 해당 코드는 매우 이해가 쉬울 것이다.

<br><Br>

여기서 중요한점은 update메서드에서 반환타입을 Member를 반환하지 않고 void로 설정한 것이다.

`Member findMember = memberService.findOne(id);`
<br><Br>
Member를 반환한다면 다음 코드가 없어도 될텐데 왜 void로 선언했을까?

<br><Br>

이유는 간단하다.

바로 로직과 쿼리를 분리하기 위해서이다.
update라는 함수의 목적은 순전히 Member엔티티를 업데이트 하는 것이 목적이다.

<br><Br>

해당 함수에서 Member객체를 반환한다면, 그것은 Member를 업데이트 하는 로직과 멤버를 조회하는 쿼리의 기능을 가지게 되는 것이다
이는 후에 유지보수를 할때 개발자가 혼란이 올 수 있다.

<br><Br>

따라서 로직과 쿼리는 분리하는 것이 좋다. 이 경우에서는  

`Member findMember = memberService.findOne(id);`
을 넣으면 쿼리가 한번 더 날아가게 된다.

그럼에도 로직과 쿼리를 분리하는 것은, 유지보수에 매우 중요하기 때문이다.

