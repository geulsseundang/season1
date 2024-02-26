# MapStruct

- Request와 Response DTO가 수정되는 것에 Dependency가 생기는 것을 방지하기 위해 Internal DTO를 사용하고 싶은데, converter를 만드는 것이 번거로움 -> MapStruct를 사용하자!

## Setting

Gradle
```
dependencies {
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
}
```
[주의] Lombok보다 MapStruct가 뒤에 선언되어야 함
-> MapStruct는 Lombok의 getter, setter, builder를 이용하여 생성됨!

## 기존 Mapper library와의 차이점

**기존 Mapper library**
- 대부분 리플렉션과 Setter를 사용을 강제 -> 디버깅 어려움, 런타임 성능에 영향

**MapStruct**
- 리플렉션을 사용 X, 컴파일 타임(어노테이션 프로세서)에 처리 -> 앱 성능에 영향 X
- 생성된 코드를 직접 확인 가능 -> 매핑 도중에 오류가 생겼을 때 디버깅이 용이
- 처리속도 빠름 -> 컴파일 시간에도 거의 영향 X

## 기본 예시

### Incoming Request DTO
```java
public class RequestMemberDto {
    private String name;
    private int age;
    private String email;
}
```

### (Application) Internal DTO
```java
public class MemberDto {
    private String name;
    private int age;
    private String email;
}
```

### Mapper Interface
- 직접 Interface를 생성 -> MapStruct가 자동으로 Impl을 생성
```java
@Mapper
public interface MemberMapper {
    MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);
    // RequestMemberDto -> MemberDto (Mappping)
    MemberDto toMemberDto(RequestMemberDto requestMemberDto);
}
```

### MapperImpl
- MapStruct가 자동으로 생성
```java
public class MemberMapperImpl implements MemberMapper {
    @Override
    public MemberDto toMemberDto(RequestMemberDto requestMemberDto) {
        if ( requestMemberDto == null ) {
            return null;
        }

        MemberDto memberDto = new MemberDto();

        memberDto.setName( requestMemberDto.getName() );
        memberDto.setAge( requestMemberDto.getAge() );
        memberDto.setEmail( requestMemberDto.getEmail() );

        return memberDto;
    }
}
```

## 응용
1. 2개 이상 Dto -> Dto로 매핑 가능
    - `@Mapping` 어노테이션 사용 : `@Mapping(source = "source", target = "target")`
2. Dto 외 다른 parameter를 받아서 Dto로 매핑 가능
    - Interface에 필요 parameter 추가해서 method 생성
3. 기타 파라미터
    - defalutExpression : 기본값으로 넣을 코드 작성
    - defaultValue : 기본값 설정
    - ignore : 매핑 무시
    - qualifiedByName : 특정 메소드로 매핑
        -> @Named 어노테이션 사용
4. Enum 매핑
    - `@ValueMapping`, `@EnumMapping`

## 설정 가능 옵션
1. Component Model
    - `@Mapper(componentModel = "spring")` : Spring Component로 사용 -> mapper를 Bean으로 등록 가능(singleTon)
2. unmappedTargetPolicy : target 필드 존재, source 필드 없을 때 설정
    - `@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)` : 매핑되지 않은 필드 오류
    - `@Mapper(unmappedTargetPolicy = ReportingPolicy.WARN)` : 매핑되지 않은 필드 경고
    - `@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE)` : 매핑되지 않은 필드 무시
3. Null 값 처리
    - `@Mapper(nullValueMappingStrategy = NullValueMappingStrategy.RETURN_DEFAULT)` : null일 때 기본값으로 매핑
    - `@Mapper(nullValueMappingStrategy = NullValueMappingStrategy.RETURN_NULL)` : null일 때 null로 매핑
    - nullValueIterableMappingStrategy : Iterable 타입의 null 값 처리

### Reference
https://januaryman.tistory.com/494?category=961027
https://medium.com/naver-cloud-platform/%EA%B8%B0%EC%88%A0-%EC%BB%A8%ED%85%90%EC%B8%A0-%EB%AC%B8%EC%9E%90-%EC%95%8C%EB%A6%BC-%EB%B0%9C%EC%86%A1-%EC%84%9C%EB%B9%84%EC%8A%A4-sens%EC%9D%98-mapstruct-%EC%A0%81%EC%9A%A9%EA%B8%B0-8fd2bc2bc33b