최근에는 Controller에서 HTML 뷰를 렌더링하지 않고 JSON과 같은 데이터를 메시지 바디에 직접 넣어서 데이터를 반환하는데 이러한 방식을 API방식이라고 한다. 이때 Controller에 @Controller가 아닌 @RestController를 사용하여 Rest API라고도 한다.

<br><br>

# 1. 회원등록 API
회원등록 API를 만들어 보려고 한다.

## 1) V1
```java
    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(
        @RequestBody @Valid Member member) {

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
```
다음과 같이 작성하였다.

<br><br>

- @RequestBody로 request메시지 바디에 있는 데이터를 Member 엔티티로 바로 매핑 받아서 사용한다.

- @Valid: request메시지 바디의 해당 변수의 제약조건을 만족하는지 확인한다.

- 여기서는 요청 값은 Member엔티티로 받았고 response메시지 바디에 들어가는 것은 CreateMemberResponse객체를 하나 생성하여 DTO로 만들어 반환해 주었다.

<br><br>

### (1) V1 문제점
- 엔티티 자체를 값으로 받아서 사용하는것이 문제가 된다.
- 엔티티에 API 검증을 위한 로직이 들어간다(@NotEmpty 등)
- 엔티티가 변경되면 API의 스펙이 변하게 된다.

<br><br>

### (2) V1 해결책
- API요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.

<br><br>

# 2. V2

```java
    @PostMapping("api/v2/members")
    public CreateMemberResponse saveMemberV2(
        @RequestBody 
        @Valid CreateMemberRequest request) {

        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest{
        @NotEmpty
        private String name;
    }
```

다음과 같이 작성한다.

<br><Br>

- @Valid: DTO에 정해준 @NotEmpty 제약조건에 맞게 데이터가 들어왔는지 판별한다.

- CreateMemberRequest라는 DTO를 만들어 엔티티 대신 RequestBody와 매핑한다.

- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.

- 엔티티와 API 스펙을 분리 함으로써 엔티티가 변해도 API 스펙이 변하지 않는다.



