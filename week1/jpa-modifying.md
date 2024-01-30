# [Spring Data JPA] @Modifying 어노테이션(feat. 1차 캐시)

> Spring Data JPA에서 제공하는 @Modifying 어노테이션에 대해 작성하였습니다.

<br>

## @Modifying 어노테이션이란?

- Spring Data JPA에서 제공하는 어노테이션으로, @Query 어노테이션과 함께 사용된다.
- 말그대로 ‘수정’이 일어날 수 있는 쿼리를 뜻하는 것으로, INSERT/UPDATE/DELETE 쿼리 작성 시에 같이 사용해주어야한다.

<br>

> cf. @Query 어노테이션
> : Spring Data JPA의 JpaRepository에서 기본적으로 제공하는 메서드들 외에 추가로 필요한 쿼리가 있다면 개발자가 직접 JPQL 혹은 native 쿼리로 작성할 수 있도록 해주는 어노테이션

<br>

## clearAutomatically 설정 옵션

@Modifying 어노테이션은 아래와 같이 2가지 설정 옵션을 제공한다.

- clearAutomatically : 쿼리 실행 후 영속성 컨텍스트를 비우는 옵션
- flushAutomatically : 쿼리 실행 전 쓰기 지연 저장소의 쿼리들을 flush하는 옵션

<br>

이 중 clearAutomatically 설정에 대해 알아보려한다. 

해당 설정 내용에 대해 이해하기 위해, JPA 영속성 컨텍스트의 1차 캐시 개념에 대해 먼저 알 필요가 있다.

<br>

### 1차 캐시

JPA에서는 엔티티 조회 시 영속성 컨텍스트의 1차 캐시를 먼저 조회 후, 1차 캐시에 데이터가 존재한다면 1차 캐시에 있는 엔티티를 반환한다.
![image](https://github.com/geulsseundang/season1/assets/78673570/803c4cf7-762c-44b7-83b6-f5fc4d3ea5f4)

하지만 @Query를 사용할 경우, 영속성 컨텍스트를 거치지 않고 바로 쿼리가 실행되어, 영속성 컨텍스트는 데이터 변경을 알 수 없다! 즉, 업데이트되지 않은 1차 캐시의 값이 조회되어 실제 DB의 값과 다른 데이터가 조회되는 문제가 발생할 수 있다.

이 때에 clearAutomatically 옵션을 true로 설정하면 자동으로 영속성 컨텍스트를 초기화할 수 있다. 조회 시 1차 캐시가 이미 clear된 상태이므로, 업데이트되지 않은 1차 캐시로 인한 문제를 해결할 수 있다.

<br>

## 사용법

위의 내용들을 종합하여 실제로는 다음과 같이 사용할 수 있다.
```java
@Modifying(clearAutomatically = true) -- clear 옵션 활성화
@Query("DELETE FROM User u WHERE u.active = false")
void deleteInactiveUsers();
```
