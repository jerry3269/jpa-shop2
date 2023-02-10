# V1. 

__엔티티 직접 노출__


- 엔티티가 변하면 API 스펙이 변한다.
- 트랜잭션 안에서 지연 로딩 필요(1 + N Query)
- 양방향 연관관계 문제

<br><br>

# V2. 

__엔티티를 조회해서 DTO로 변환(fetch join 사용X)__

- 트랜잭션 안에서 지연 로딩 필요(1 + N Query)

<br><br>

# V3. 

__엔티티를 조회해서 DTO로 변환(fetch join 사용O)__

- 페이징 시에는 N 부분을 포기해야함(대신에 batch fetch size? 옵션 주면 N -> 1 쿼리로 변경
 가능)

 <br><br>

# V4.

__JPA에서 DTO로 바로 조회, 컬렉션 N 조회__

- (1 + N Query)
- 페이징 가능

<br><br>

# V5. 

__JPA에서 DTO로 바로 조회, 컬렉션 1 조회 최적화 버전__

- (1 + 1 Query)
- 페이징 가능

<br><br>

# V6. 

__JPA에서 DTO로 바로 조회, 플랫 데이터__

- (1 Query)
- 페이징 불가능
- 매우 복잡


<br><br>