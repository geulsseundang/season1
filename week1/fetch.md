# Fetch Type

## Lazy loading

- 연관된 Entity를 proxy로 조회
- proxy를 실제로 사용할 때 초기화하면서 데이터베이스 조회

## Eager loading

- 연관된 Entity를 즉시 조회
- Hibernate는 가능하면 SQL join으로 한 번에 조회함

## Default Fetch Stategy

@ManyToOne, @OneToOne : Eager
@OneToMany, @ManyToMany : Lazy

- 기본적으로 안전하려면 Lazy 사용해야겠죠!! 왜????

## 겪었던 FetchType.EAGER 중첩으로 인한 Duplicated Key 문제

### 현상

JPA 사용하여 chaining으로 조회한 객체가 중복 키를 return 함

### 원인

날아가는 SQL 확인해보니, 중첩된 Eager로 2번의 Outer join으로 쿼리가 날아가 발생한 것으로 확인

### 기존 Entity 구조

- 예시

```
BoardEntity
ㄴ List<PostEntity> (EAGER)
        PostEntity
         ㄴ List<TagEntity> (EAGER)
```

=> 이로 인해 PostEntity \* TagEntity 갯수로 PostEntity가 나오게 됨

### 예시 코드 및 결과

- 예시 코드

```java
 @Override
    public void test(){
        BoardEntity board = boardEntityRepository.findById(38L).get();
        board.getPosts().forEach(post->log.info("Post Id :"+post.getId()));
    }
```

- 실행 결과

```
Hibernate:
    select
       ...
    from
        board
    left outer join
        post on board.id = post.board_id
    left outer join
        tag  on post.id = tag.post_id
    where
        board.id=?
2024-01-05 16:03:06,355 INFO [TestServiceImpl.java : 64] Post Id :38
2024-01-05 16:03:06,355 INFO [TestServiceImpl.java : 64] Post Id :38
2024-01-05 16:03:06,355 INFO [TestServiceImpl.java : 64] Post Id :38
```

## 개선 방향

!! Best Practice

- ~ ToOne : EAGER 사용 가능
- ~ ToMany : LAZY 사용, EAGER 사용 지양
