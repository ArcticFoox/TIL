# N + 1 Problem
DB에서 1번만 가져오면 될 데이터를 여러 번에 걸쳐 조회하게 되는 현상

JPA는 객체지향적으로 데이터를 가져오고자 지연 로딩(LAZY)을 기본으로 함
연관된 엔티는 처음에 바로 가져오지 않고, 실제로 접근할 때 추가 쿼리 발생
```java
List<Post> posts = postRepository.findAll();
for (Post post : posts) {
    System.out.println(post.getAuthor().getNickname()); // 이 시점에서 author 조회 쿼리 발생
}
``` 
게시글은 한 번의 쿼리로 가져오지만, 작성자(author)는 접근 시점마다 따로 가져오기 때문에 결국 게시글 수만큼 추가 쿼리 발생

## 해결법
### fetch join
```sql
@Query("SELECT p FROM Post p JOIN FETCH p.author") 
List<Post> findAllWithAuthor();
```
주의점
컬렉션을 fetch join 할 경우 중복 발생
페이징 불가능

```sql
@Query("SELECT new com.example.dto.PostDto(p.id, a.nickname) FROM Post p JOIN p.author a") 
List<PostDto> findAllDtos();
```
주의점
엔티티가 아니므로 영속성 컨텍스트 미적용
쿼리 재사용이 어려울 수 있음
