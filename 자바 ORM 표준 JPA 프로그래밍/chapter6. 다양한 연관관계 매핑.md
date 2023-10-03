## 다양한 연관관계 매핑
엔티티 간 연관관계를 매핑할 때 고려해야할 사항은 다음과 같다.
- 방향
    - 단방향
        - 대상 객체만이 상대 객체에 대한 참조를 지님
        - A객체가 B객체 참조, B객체는 A객체 참조x
    - 양방향
        - 서로 참조를 지님
        - A객체가 B객체 참조, B객체도 A객체를 참조
        - 테이블의 경우, 외래키로 관련있는 두 테이블은 항상 양방향
- 다중성
    - 다대일(@ManyToOne)
    - 일대다(@OneToMany)
    - 일대일(@OneToOne)
    - 다대다(@ManyToMany)
- 연관관계 주인
    - 외래키를 관리하는 엔티티
    - 외래키를 가진 테이블과 매핑되는 엔티티에서 관리하는 것이 효율적

가능한 연관관계는 다음과 같다.
- 다대일 : 단방향, 양방향
- 일대다 : 단방향, 양방향
- 일대일 : 단방향, 양방향
- 다대다 : 단방향, 양방향

### 다대일
- 단방향
    - 의미 
        - 대상 객체와 상대 객체의 관계는 다대일
        - 대상 객체만이 상대 객체를 참조
    - 예시
        - 대상 엔티티 : Member, 상대 엔티티 : Team
        ```java
        @Entity
        public class Member{

            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private int id;

            @Column(name = "NAME")
            private String username;

            @ManyToOne
            @JoinColumn(name = "TEAM_ID")
            private Team team;

        }

        @Entity
        public class Team{

            @Id @GeneratedValue
            @Column(name = "TEAM_ID")
            private int id;

            @Column(name = "NAME")
            private String name;

        }
        ``` 
- 양방향
    - 의미
        - 대상 객체와 상대 객체의 관계는 다대일, 상대 객체와 대상 객체와의 관계는 다대일
        - 서로를 참조
    - 연관관계 주인 : 외래키를 가지고 있는 테이블과 매핑되는 엔티티
        - DB에서는 항상 다 쪽이 외래키를 관리 => 다쪽에 해당하는 엔티티가 연관관계의 주인
    - 예시
        - 대상 엔티티 : Member, 상대 엔티티 : Team
        ```java
        @Entity
        public class Member{

            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private int id;

            @Column(name = "NAME")
            private String username;

            @ManyToOne
            @JoinColumn(name = "TEAM_ID")
            private Team team;

        }

        @Entity
        public class Team{

            @Id @GeneratedValue
            @Column(name = "TEAM_ID")
            private int id;

            @Column(name = "NAME")
            private String name;

            @OneToMany(mappedBy = "team")
            private List<Member> memberList = new ArrayList<>();

        }
        ``` 
### 일대다
- 단방향
    - 의미
        - 대상 객체와 상대 객체의 관계는 일대다
        - 대상 객체만 상대 객체를 참조
    - 연관관계의 주인 : 대상 엔티티
    - 예시
        - 대상 엔티티 : Team, 상대 엔티티 : Member
        ```java
        @Entity
        public class Team{

            @Id @GeneratedValue
            @Column(name = "TEAM_ID")
            private int id;

            @Column(name = "NAME")
            private String name;

            @OneToMany
            @JoinColumn(name = "TEAM_ID")
            private List<Member> memberList = new ArrayList<>();

            public Team(int id, String name){
                this.id = id;
                this.name = name;
            }

        }

        @Entity
        public class Member{

            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private int id;

            @Column(name = "NAME")
            private String username;

            public Member(int id, String username){
                this.id = id;
                this.username = username;
            }

        }
        ``` 
    - 문제점
        - 다른 테이블에서 관리하는 외래키를 관리
        - 외래키를 관리하는 테이블에 대해, 외래키 값을 넣어주기 위한 UPDATE 문이 추가 실행
            - Member 엔티티는 외래키를 관리하지 않기에, insert 쿼리 생성 시 외래키에 해당하는 team_id 포함 x
            - Team 엔티티가 외래키를 관리하기에, 연관관계에 있는 Member들에 대해 외래 키 값을 넣어주는 update 문이 추가적으로 생성
        ```java
        public void testSave(){
            Member member1 = new Member(1, "member1");
            Member member2 = new Member(2, "member2");

            Team team = new Team(1, "teamA");
            team.getMemberList().add(member1);
            team.getMemberList().add(member2);

            em.persist(member1);
            em.persist(member2);
            em.persist(team);
        }
        ```

        ```sql
        INSERT INTO MEMBER (MEMBER_ID, NAME) VALUES (1, 'member1');
        INSERT INTO MEMBER (MEMBER_ID, NAME) VALUES (2, 'member2');

        INSERT INTO TEAM (TEAM_ID, NAME) VALUES (1, 'teamA');

        UPDATE MEMBER SET TEAM_ID = 1 WHERE MEMBER_ID = 1;
        UPDATE MEMBER SET TEAM_ID = 2 WHERE MEMBER_ID =2;
        ```
    - 결론 : 외래키를 가지고 있지 않은 테이블과 매핑되는 엔티티에서 외래키를 관리하는 것은 update 쿼리가 추가적으로 수행되어야 하는 성능 상 문제와 관리 차원의 문제를 내포하고 있다. 따라서 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하는 것이 성능, 관리 차원에서 바람직하다.
- 양방향
    - 의미
        - 대상 객체와 상대 객체의 관계는 일대다, 상대 객체와 대상 객체와의 관계를 다대일
        - 서로를 참조
    - 연관관계의 주인을 대상 엔티티로 할 경우, 일대다 단방향 매핑의 문제와 동일한 문제가 발생한다. 따라서 다대일 양방향 매핑을 사용해서, 외래키를 외래키를 가진 테이블과 매핑되는 엔티티에서 관리하도록 하는 것이 바람직하다.
### 일대일
- 단방향
    - 의미
        - 대상 객체와 상대 객체의 관계는 일대일
        - 대상 객체만 상대 객체를 참조
    - 연관관계의 주인
        - DB상에서는 일대일 관계에 있는 테이블의 경우, 주 테이블 또는 상대 테이블 모두 외래키를 지닐 수 있다. 
- 양방향
### 다대대
- 단방향
- 양방향