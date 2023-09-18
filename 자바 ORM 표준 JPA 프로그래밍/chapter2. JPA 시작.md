## Chapter2. JPA 시작
### 객체 매핑
- @Entity
- @Table
    - 엔티티 클래스에 테이블 정보를 알려줌
- @Id
    - 엔티티 클래스의 필드를 테이블의 기본 키에 매핑
    - @Id가 사용된 필드 → 식별자 필드
- @Column
    - 필드를 컬럼에 매핑

### persistence.xml 설정
JPA는 기본적으로 META-INF/persistence.xml 경로의 xml 파일을 읽어 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <persistence-unit name="jpabook">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

        </properties>
    </persistence-unit>

</persistence>
```

- 영속성 유닛(persistence-unit)
    - 연결할 데이터베이스 당 하나의 영속성 유닛 등
- 필수속성
    - javax.persistence.jdbc.driver : JDBC 드라이
    - javax.persistence.jdbc.user : 데이터베이스 접속 아이디
    - javax.persistence.jdbc.password : 데이터베이스 접속 비밀버호
    - javax.persistence.jdbc.url : 데이터베이서 접속 url
    - hibernate.dialect : 데이터베이스 방언 설정
- 옵션
- 데이터베이스 방언
    - 의미 :  특정 데이터베이스만의 고유한 기능에 맞게 SQL을 변환해주는 매개체
    - JPA 사용자는 JPA 표준 문법에 따라 개발을 하면 되고,  데이터베이스 방언이 사용하는 데이터베이스 특성에 맞는 SQL을 생성해준다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/581571e0-0f3c-43bd-812b-7545511a6ad4/4275ac5a-e384-4f53-924b-70b79a6bd5fd/Untitled.png)
    
### 애플리케이션 개발
JPA를 이용한 로직 수행과정
```java
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook"); //1. 엔티티 매니저 팩토리 생성
        EntityManager em = emf.createEntityManager(); //2. 엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {

            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }
```
1. EntityManagerFactory  생성
    - JPA를 사용할 수 있도록 해주는 팩토리 클래스
    - persistence.xml에 설정 영속성 유닛과 관련된 필수 속성에 따른 팩토리 클래스 생성
2. EntityManager 생성
    - EntityManagerFactory 클래스를 통해 EntityManger 생성
    - JPA의 대부분의 기능을 제공 → EntityManager를 통해 데이터베이스에 쓰기 작업 및 읽기 작업 수행 가능
    - 데이터베이스 커넥션을 유지하면서 데이터베이스와 통신
3. 트랜잭션관리
    - JPA를 사용하여 쓰기 작업을 할 경우, 항상 트랜잭션 안에서 수행해야한다.
    - 트랜잭션을 시작하고, 비즈니스 로직이 정상적으로 수행되면 commit이 발생하여 데이터베이스에 변경사항이 반영되고 그렇지 않으면 rollback하여 반영하지 않는다.
4. 비즈니스 로직
    - 등록
        
        ```java
        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("유미");
        member.setAge(25);
        
        //등록        
        em.persist(member);
        ```
        
        JPA는 Member 엔티티의 매핑 정보를 분석하여 다음과 같은 SQL을 만들어, 데이터베이스에 전달한다.
        
        ```sql
        INSERT INTO MEMBER(ID, NAME, AGE) VALUES (’id1’,’유미’,25) 
        ```
        
    - 수정
    - 조회
    - 삭제