## chapter5. 연관관계 매핑기초
### 연관관계 매핑 관련 용어
| | 객체 | 테이블 |
| :---: | :---: | :---: |
| 연관관계 | 참조 | 외래키 |
| 연관관계 탐색 | 그래프 탐색 | 조인|
| 방향 | 항상 단반향 | 항상 양방향|

객체는 참조로, 테이블은 외래키로 연관관계를 맺는다. 객체의 참조와 테이블의 외래키를 매핑하는 것이 중요한데 관련 용어는 다음과 같다.
- 방향
    - 단방향 : 객체 간 관계
    - 양방향 : 테이블 간 관계
- 다중성
    - 1:N(일대다) ex. 회원 - 팀
    - N:1(다대일) ex. 팀 - 회원
    - N:N(다대다) ex. 주문 - 주문상품
- 연관관계 주인


### 객체의 단방향 연관관계
- 객체 : 참조로 다른 객체와 연관관계를 맺음
    - 항상 단방향
    - 매핑 어노테이션
        - @ManyToOne
            - 참고 관계의 엔티티를 매핑할 때 사용
            - 다중성 어노테이션은 필수
        - @JoinColumn
            - 외래키를 매핑할 때 사용
            - name 속성에 외래키를 매핑

```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ",
    initialValue = 1, allocationSize = 1
)
public class Member{
    
    @Id
    @Column(name ="MEMBER_ID")
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private int id;
    @Column(name ="NAME")
    private String username;

    public Member(String username){
        this.username = username;
    }

    // 연관관계 맹핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public void setTeam(Team team){
        this.team = team;
    }
}

@Entity
@SequenceGenerator(
    name = "TEAM_SEQ_GENERATOR",
    sequenceName = "TEAM_SEQ",
    initialValue = 1, allocationSize = 1
)
public class Team{
    
    @Id
    @Column(name ="TEAM_ID")
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "TEAM_SEQ_GENERATOR")
    private int id;
    private String name;

    public Team(String name){
        this.name = name;
    }
}
```
- 테이블 : 외래키로 다른 테이블과 연관관계를 맺음
    - 항상 양방향 -> MEMBER JOIN TEAM / TEAM JOIN MEMBER

- 저장 
    - JPA는 Member 엔티티와 Team 엔티티가 연관관계에 있는 걸 인식하며, Member 테이블의 외래키가 TEAM_ID도 인식하며 적절한 등록 쿼리를 생성한다.
    ```java
    public void saveMemberWithTeam(EntityManager em){
        Team team = new Team("팀1");
        em.persist(team);
    
        Member member1 = new Member("회원1");
        member1.setTeam(team);
        em.persist(member1);

        Member member2 = new Member("회원2");
        member2.setTeam(team);
        em.persist(member2);    
    }  
    ```
    ```sql
    INSERT INTO TEAM (TEAM_ID, NAME) VALUES (1, '팀1');

    INSERT INTO MEMBER (MEMBER_ID, NAME, TEAM_ID) VALUES (1, '회원1');
    INSERT INTO MEMBER (MEMBER_ID, NAME, TEAM_ID) VALUES (2, '회원2');
    ```

- 조회
    - 객체 그래프 탐색
    ```java
    Member member = em.find(Member.class, 2);
    // 객체 그래프 탐색
    Team team = member.getTeam();
    ```
- 수정
    - 연관관계 있는 참조 엔티티 변경 시, JPA는 외래키를 변경하는 적절한 쿼리를 생성한다..
    ```java
    public void updateMemberWithTeam(EntityManager em){
        Team team = new Team("팀2");
        em.persist(team);
    
        Member member = em.find(Member.class, "1");
        member.setTeam(team);
    }  
    ```
    ```sql
    UPDATE MEMBER
    SET TEAM_ID = '팀2'
    WHERE MEMBER_ID = '1'
    ```
- 삭제
    - 연관관계에 있는 엔티티를 삭제하려면, 먼저 연관관계부터 제거해야한다. 그렇지 않으면, 외래 키 제약조건으로 데이터베이스 오류가 발생한다.
    ```java
    public void deleteTeam(EntityManager em){
       member1.setTeam(null);
       member2.setTeam(null);
       em.remove(team)l
    }  
    ```
    ```sql
    UPDATE MEMBER
    SET TEAM_ID = null
    WHERE MEMBER_ID = 1;

    UPDATE MEMBER
    SET TEAM_ID = null
    WHERE MEMBER_ID = 2;

    DELETE FROM TEAM
    WHERE TEAM_ID = 1;
    ```

### 객체의 양방향 연관관계
엄일하게, 객체의 경우 양방향 관계는 없고, 각 엔티티에 단방향 관계를 각각 맺어주면 마치 양방향 관계처럼 보이게 할 수 있다. 연관관계에 있는 두 객체의 서로를 참조하는 단방향 관계 2개를 편의상 양방향 관계로 명명할 수 있다. 
- 객체
    - 단방향 관계를 연관관계에 놓인 두 엔티티 모두에 설정
    - 회원 <-> 팀
        - 회원 - 팀 : N:1 관계
        - 팀 - 회원 : 1:N 관계
    - 1:N 관계에 있는 경우 컬렉션을 사용
```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ",
    initialValue = 1, allocationSize = 1
)
public class Member{
    
    @Id
    @Column(name ="MEMBER_ID")
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private int id;
    @Column(name ="NAME")
    private String username;

    public Member(String username){
        this.username = username;
    }

    // 연관관계 맹핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public void setTeam(Team team){
        this.team = team;
    }
}

@Entity
@SequenceGenerator(
    name = "TEAM_SEQ_GENERATOR",
    sequenceName = "TEAM_SEQ",
    initialValue = 1, allocationSize = 1
)
public class Team{
    
    @Id
    @Column(name ="TEAM_ID")
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "TEAM_SEQ_GENERATOR")
    private int id;
    private String name;
    
    @OneToMany(mappedBy="team")
    private List<Member> memberList;

    public Team(String name){
        this.name = name;
    }
}
```  
- 테이블
    - 항상 양방향 관계
- 조회
```java
Team team = em.find(Team.class, "team1");
List<Member> memberList = team.getMemberList(); // 객체 그래프 탐색
```

### 연관관계의 주인
서로 단방향 관계에 있는 두 엔티티에 대해 외래키를 관리하는 주인을 정해야한다. 주인 엔티티만이 외래키 값을 등록, 수정, 삭제할 수 있다. 주인은 외래키를 지닌 테이블과 매핑되는 엔티티로 정하면 되며, 다중성 어노테이션에 mappedBy 속성을 지정해주지 않으면 된다.

### 양방향관계 설정 시 주의점
다음과 같은 상황에서 외래키 값이 제대로 등록되지 않는다.
- 주인 엔티티의 연관관계에 있는 참조변수 셋팅 x
    ```java
    public void notSettedRelEntityOfOwner(EntityManager em){
        Member member1 = new Member("회원1");
        em.persist(member1);

        Member member2 = new Member("회원2");
        em.persist(member2);

        Team team = new Team("팀1");
        team.getMemberList().add(member1);
        team.getMemberList().add(member2);
        em.persist(team);

    }  
    ```
    => Member 테이블의 Insert 된 Row에 대해 TEAM_ID의 값이 모두 null  

결론  
1) 주인 엔티티의 연관관에 있는 참조변수는 반드시 셋팅
2) 순수 자바 객체와 데이터베이스 데이터와의 정합성을 위해, 연관관계에 있는 참조는 주인이든 아니든 모두 셋팅 
    ```java
    public void setEntities(EntityManager em){
        Team team = new Team("팀1");
        em.persist(team);

        Member member1 = new Member("회원1");
        member1.setTeam(team);
        team.getMemberList().add(member1);
        em.persist(member1);

        Member member2 = new Member("회원2");
        team.getMemberList().add(member2);
        em.persist(member2);

    }  
    ```

### 연관관계 편의 메서드
연관관계에 놓인 두 엔티티의 필드를 설정하는 코드를 보면, 연관관계 셋팅 코드가 각각 2번 시행된다. 
- 주인 엔티티에서 연관관계 셋팅
     ```java
    member1.setTeam(team);
    ```
- 주인x 엔티티에서 연관관계 셋팅
    ```java
    team.getMemberList().add(member1);
    ```
    => 각각 연관관계를 셋팅하다 보면 어느 하나를 빠뜨리는 실수가 생길 가능성이 있다.
- 연관관계 편의 메서드 : 각각의 연관관계를 한번에 셋팅해주는 메서드
- 주의사항
    - 연관관계에 놓인 상대 엔티티에 자기자신을 설정해 줄 때, 기존 연관관계를 끊어주는 로직 필수
    - 기존 연관관계를 끊어주지 않으면, DB 상 데이터와 자바 객체 데이터 정합성 x
    ```java
    public class Member {
        ...

        private Team team;

        ...
        // 연관관계 편의 메서드
        public void setTeam(Team team){
            this.team = team;

            if(this.team != null){
                this.team.getMemberList().remove(this);
            }

            this.team.getMemberList().add(this);
        }
    }
    ```
