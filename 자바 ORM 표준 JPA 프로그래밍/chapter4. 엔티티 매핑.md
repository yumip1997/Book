## chapter4. 엔티티 매핑
### 객체와 테이블 매핑 : @Entity, @Table
### @Entity
- 의미 : 클래스와 테이블을 매핑하며, 해당 어노테이션을 붙이면 JPA가 관리하게 되며 클래스를 엔티티라고 부름
- 주의사항
    - 기본 생성자 필수
    - final 클래스, enum, interface, inner 클래스에서는 사용 불가
    - 필드에 final 사용불가
### @Table
- 의미 : 엔티티와 매핑할 테이블 지정
- 속성
    - name : 매핑할 테이블 이름을 지정하며 해당 속성을 사용하지 않을 경우 엔티티의 이름을 기본값으로 사용
    - catalog :  catalog 기능이 있는 db에서 catalog를 매핑
    - schema : schema 기능이 있는 db에서 schema를 매핑
### 기본 키 매핑
- 기본 키 직접 할당 전략
    - 기본 키에 해당하는 필드에 @Id를 명시
    - 가능 타입 : 자바 기본형, 자바 래퍼형, String, java.util.Date, java.sql.Date, java.math.BigDecimal, java.math.BigInteger
    ```java
    @Entity
    @Table(name="MEMBER")
    public class Member {

    @Id
    private Long id;

    private String userName;
    }
    ```
- IDENTITY 전략
    - 기본 키 생성을 데이터베이스에 위임 (EX. 데이터베이스의 AUTO_INCREMENT 속성)
    - 기본 키에 해당하는 필드에 @Id, @GeneratedValue(strategy = GenrationType.IDENTITY) 명시
    ```java
    @Entity
    @Table(name="MEMBER")
    public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String userName;
    }
    ```
    - 영속성 컨테이너 관리 하에 있기 위해서는 식별자가 반드시 필요하다. AUTO_INCREMENT와 같은 기본 키 생성 방식으로는 INSERT 후 기본 키 값을 알 수 있다. 따라서, persist 메서드 호출 직후 INSERT 쿼리가 수행되어야한다. (쓰기 지연 동작X)
    - persist 메서드 호출 후 영속상태에 돌입하는 과정
      - persist 메서드 호출 시, INSERT 쿼리가 즉시 데이터베이스에 전달    
      - 기본 키 값이 식별자에 할당
      - 영속상태
    - 쓰기 지연 동작X
- SEQUENCE 전략
    - 기본키 생성을 데이터베이스에 위임하는데, 데이터베이스는 시퀀스를 사용해 기본 키 생성
    ```java
    @Entity
    @Table(name="MEMBER")
    @SequenceGenerator(
        name = "BOARD_SEQ_GENERATOR",
        sequenceName = "BOARD_SEQ", // 매핑할 데이터베이스의 시퀀스 이름
        initialValue = 1, allocationSize = 1
    )
    public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;

    private String userName;
    }
    ```
    - @SequenceGenerator
      - 클래스 Level
      - name 속성 : Generator의 이름
      - sequenceName 속성 : 매핑할 데이터베이스의 시퀀스 이름
    - 식별자에 해당하는 필드
      - @Id
      - @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BOARD_SEQ_GENERATOR")
    - persist 메서드 호출 후 영속상태에 돌입하는 과정
      - DB의 시퀀스를 이용해 기본 키 값 조회
      - 기본 키 값이 식별자에 할당
      - 영속상태
      - INSERT 쿼리는 쓰기 지연 SQL 저장소에 저장 -> 트랜잭션 커밋 후 DB에 전달
### 필드와 컬럼 매핑 : @Column

### 데이터베이스 스키마 자동 생성
persistence.xml에 다음과 같은 속성을 추가하면, 애플리케이션 실행 시점 데이터베이스 테이블이 자동으로 생성된다. 이와 같은 속성은 운영 서버에서는 절대 사용하지 말아야하는 옵션이다. 운영 중인 테이블을 삭제할 수 있기 때문이다.
```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

### 