# 회원 조회 API


<br><Br>

# 1. V1

<br><Br>

회원 조회 API의 제일 단순한 코드이다

```java
@GetMapping("/api/v1/members")
    public List<Member> membersV1(){
        return memberService.findMembers();
    }
```

무슨 문제가 있을까?

<br><Br>

## 1) 문제점
- 엔티티가 외부에 직접 노출된다.
- 엔티티의 모든 값이 노출된다.
- 엔티티에서 특정 필드의 값을 숨기고 싶을때는 필드에 @JsonIgnore 어노테이션을 추가해야 한다.
- 위와 같은 응답 스펙을 맞추기 위한 로직이 추가된다.
- 프레젠테이션 로직이 엔티티에 추가된다.
- 실무에서는 같은 엔티티에 대해 API의 용도에 따라 수많은 API 스펙이 요구된다. 여러 스펙에 대한 API를 엔티티 하나에 담기는 매우 어렵다
- 엔티티가 변경되면 API스펙이 변한다.
- 추가로 컬렉션을 반환하게 되면 API스펙을 변경하기 매우 어려워 진다.

<br><Br>

## 2) 결론
앞서 말했듯이 엔티티 자체를 외부에 노출하거나 요청 값으로 받는것은 현명하지 못한 일이다.

<br><Br>

가장 중요한 것은 엔티티가 변경되면 API 스펙이 변한다는 것이다.

따라서 API응답 스펙에 맞추어 별도의 DTO를 반환해야 한다.

<br><Br>


# 2. V2

```java
    @GetMapping("/api/v2/members")
    public Result membersV2(){
        List<Member> findMembers = memberService.findMembers();
        List<MemberDto> collect = findMembers.stream()
                .map(m -> new MemberDto(m.getName()))
                .collect(Collectors.toList());

        return new Result(collect);
    }

    @Data
    @AllArgsConstructor
    static class Result<T> {
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDto{
        private String name;
    }
```
다음과 같은 코드를 작성하였다.

<br><Br>

- 엔티티를 DTO로 변환해서 반환하였다.

따라서 엔티티가 변해도 API 스펙이 변경되지 않는다.

<br><Br>

여기서 주의깊게 봐야할 점은 2가지이다.
- 엔티티 -> DTO 변환
- 컬렉션을 직접 반환 하지 않고 `Result`클래스로 컬렉션을 감싸서 반환

<br><Br>

## 1) 엔티티 -> DTO 변환
엔티티를 외부에 노출시키지 않기 위해서 DTO 클래스를 만들었다.

<br><Br>

해당 클래스는 `name`변수만을 가지고 있어서 회원의 이름만을 출력하도록 하였다.


<br><Br>

멤버 변수에서 DTO로는 자바 8이상부터 제공되는 steam기능을 이용하였다.

스트림을 이용하여 `List`에 들어있던 모든 멤버 변수를 `DTO`로 변경하여 저장하였다.

<br><Br>

## 2) Result 클래스로 컬렉션은 감싼후 반환
엔티티 -> DTO 변환후 list 컬렉션을 바로 반환하지 않고, Result라는 클래스로 한번 감싼후 반환하였다.

<br><Br>

그 이유는 바로 향후 필드를 추가하기 위해서이다.

<br><Br>

예를들어, 현재는 회원의 이름반을 출력하고 있지만, 향후 API 스펙을 변경하여서 총 회원의 수도 API스펙에 추가한다고 가정해보자.

컬렉션을 반환하면 해당 DTO내부에 회원의 수를 세는 필드가 없을 뿐더러 만든다고 해도 멤버변수에 해당 필드가 없기 때문에 패밍하는 것에 문제가 발생한다.

<br><Br>

이러한 이유로 `Result`라는 클래스를 생성하여 DTO를 한번 감싸준뒤 반환하였다.

이렇게 하면 후에 `Result`클래스에 

```java
private int count;
```

필드만 추가해주고 return 할때에 

```java
return new Result(collect.size(), collect);
```
만 해주면 Json 데이터로 총 회원의 수 까지 스펙에 추가할 수 있게 된다.

<br><Br>






