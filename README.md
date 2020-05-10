# JPA Study

#### 스터디 멤버
- 본인
- 쿄잉 (유쾌한 스프링)

#### 진행 방식
- 리볼링 스터디
- 주 2회진행, 1장씩 돌아가면서 발표



### JPA를 학습해야 하는 이유 ?

#### SQL 개발의 문제점

- CRUD
  - 무의미한 반복적인 노가다..
  - 자바 객체를 SQL로, SQL을 자바객체로 매핑해주어야 한다..



> *사실상 개발자는 SQL Mapper의 역할을 하게 된다.*



```java
class Member {
  private String memberId;
  private String name;
  private int age;
}
```



```sql
INSERT INTO MEMBER VALUES (...)
```





개발 도중 혹은 추후에 Member 테이블에 필드가 추가되면, 모든 쿼리에 수정이 필요하다.



> SQL 의존적인 개발을 할 수 밖에 없다.





`패러다임의 불일치 해결`



객체 vs 관계형 데이터베이스



> *객체지향 프로그래밍은 추상화, 캡슐화, 은닉, 상속, 다형성 등 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공한다.*



객체를 영구 보관하는 다양한 저장소

- RDB, NoSQL, File등이 있지만 현실적인 대안은 관계형 데이터베이스이다.



객체를 관계형 데이터베이스에 저장하려면 ? 

- 객체 -> SQL 변환 -> DB



> 개발자가 SQL Mapper의 일을 하고 있다..



`객체와 관계형 데이터베이스의 차이와 해결 방법들`



- 1.밀도 문제

| 객체                              | 릴레이션         |
| --------------------------------- | ---------------- |
| 다양한 크기의 객체를 만들기 쉽다. | 테이블           |
| 커스텀한 타입을 가질 수 있다.     | 기본 데이터 타입 |

> * 커스텀한 데이터타입을 생성했을때 JPA가 릴레이션과 객체간의 매핑 문제를 해결해 준다.





- 2.상속 (서브타입) 문제
  - 객체는 상속구조를 만들기가 쉽지만, 테이블을 상속이 없다.
  - 다형성
  - 관계형 데이터베이스는 다형성을 표현할 방법이 없다.



- 3.데이터 타입
  - 커스텀한 데이터타입을 만들었을때 JPA가 릴레이션과의 매핑 문제를 해결해 준다.



- 4.연관관계
  - 객체는 참조를 사용한다.
    - member.getTeam();
  - 테이블을 외래 키를 사용한다.
    - MEMBER JOIN ON M.TEAM_ID = T.TEAM_ID

- 객체에는 근본적으로 ********방향 이 존재하지만, 데이터베이스 에서는 방향*****\***이 의미가 없다.
  - join을 활용해 아무 방향이나 다 가져올 수 있다.
  - 데이터베이스에는 다대다 관계가 없으며, 조인테이블이나 링크테이블이 필요하다.



> *테이블은 Member <-> Team이 가능하지만, 객체는 Member -> Team 만 가능하다*

> 이러한 매핑문제를 해결해준다. (링크테이블 자동생성 등)



`객체를 테이블에 맞춰 모델링`

```java
class Member {
	private String memberId;
  private Long teamId;
  private String username;
}

class Team {
  private Long id;
  private String name;
}
```



`객체다운 모델링`

```java
class Member {
  private String memberId;
  private Team team;
  private String username;
}

class Team {
  private Long id;
  private String name;
}
```



> 연관관계를 객체 참조관계로 표현한다.





`객체 모델링, 자바 컬렉션을 통한 관리`

```java
List<Member> members = new ArrayList<>();
members.add(new Member("memberA"));

Member memeber = members.get(memberId); // memberId를 통해 가져올수 있다고 가정
Team team = member.getTeam();
```



>  객체로 관리하면 매우 단순하고 깔끔한 문제이지민, 데이터베이스 넣고 빼는 순간 문제가 발생한다.





- 5.객체 그래프 탐색
  - 객체는 자유롭게 객체 그래프 탐색을 할 수 있어야 한다.

>  Delevery -> Order -> Member -> Team ...



***처음 작성한 SQL에 따라 그래프 탐색이 제한된다.***

> 엔티티에 대한 신뢰성 문제 발생



```java
class MemberService {
  public void find() {
    Member member = memberDAO.find(memberId);
    member.getTeam();
    member.getOrder(); // ??
    member.getOrder().getDelivery(); // ??
  }
}
```



Layered 아키텍쳐 에서 특정 레이어를 신뢰하고 사용할 수 있어야 하지만 불가능..



> *진정한 의미의 계층 분할이 어렵다. => 논리적으로 매우 강결합 되어있다.*





- 6.데이터 식별 방법
  - 객체는 레퍼런스의 동일성
  - 데이터베이스는 PK
  - 인스턴스의 동일성은 equals



```java
String memberId = "USER_001";

Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2; // true
```



> 식별성 문제를 JPA가 해결해준다.



**객체를 자바 컬렉션에 저장하듯이 관리할 수는 없을까 ?**



1980년대 부터 고민해 왔고 그 고민의 결과가 바로 JPA

> *JPA (Java Persistence API)
