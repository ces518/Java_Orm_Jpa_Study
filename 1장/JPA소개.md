## 1장 JPA 소개



#### SQL 을 직접 다룰때 발생하는 문제



현대에 와서 대부분의 자바 애플리케이션은 **관계형 데이터베이스** 를 데이터 저장소로 사용한다.

데이터베이스에 데이터를 관리하려면, **SQL**을 사용해야 한다.



##### 반복, 그리고 SQL 의존적인 개발



`일반적으로 회원 관련 기능을 만든다고 가정했을때의 예제`

```java
class Member {
  private String memberId;
  private String name;
}
```



 ```java
class MemberDAO {
  private Member find(String memberId) {...}
  private void save(Member member) {...}
}
 ```



 ```sql
INSERT INTO MEMBER VALIES (...)
UPDATE MEMBER SET ... WHERE ...
 ```



`문제점`

1. 회원 <기능명> SQL 을 작성한다.
2. JDBC API 를 사용해서 SQL을 실행한다.
3. 결과를 Mapping 한다.





> 만약 회원 테이블의 컬럼이 추가/수정 될경우 모든 SQL에 수정이 필요하다.
>
> 개발자도 사람이기 때문에 휴먼에러(오타 등) 에 매우 취약하다.
>
> 컴파일 타임에 에러를 찾을 수 없다.



가장 큰 문제는, Layered Architechture의 의미가 사라진다.

- Layered Architechture의 의미는 해당 계층을 신뢰하고 사용할 수 있어야 하는데, 해당 계층을 신뢰할 수 없게 된다.



> Member -> Team 의 관계에서 MemberDAO.find() 를 했을때 Team 까지 함께 가져오는지 알수 없다.
>
> 결국에는 SQL 까지 열어봐야 알수 있다..



#### 패러다임의 불일치 해결



객체지향 프로그래밍은 추상화, 캡슐화, 정보은닉, 상속, 다형석 등 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공한다.

현대의 복잡한 애플리케이션은 대부분 객체지향 언어로 개발을 한다.



문제는 비즈니스 모델을 객체로 모델링한 도메인 모델을 저장할 때 발생한다.

결국 관계형 데이터베이스에 저장을 해야하는데 여기서 발생하는 패러다임의 불일치를 해결하기 위해, 너무 많은 시간과 코드를 소비한다.



##### 상속



객체는 상속이라는 기능을 가지고 있지만, 테이블을 상속이라는 기능이 없다.

그나마 유사한 모델이 슈퍼타입, 서브타입의 관계를 사용 하는 것이다.

JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.

마치 자바 컬렉션이 객체를 저장하듯이 JPA 에 객체를 저장하면 된다.



```java
class Item {
  Long id;
  String name;
  int price;
}

class Album extends Item {
  String artist;
}
  
jpa.persist(album); // 저장

Album album = jpa.find(Album.class, "id0001"); // 조회
```



```sql
INSERT INTO ITEM ...
INSERT INTO ALBUM ... // 저장

SELECT I.*, A.*
FROM ITEM I
JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID // 조회
```





##### 연관관계



객체는 **참조**를 통해 연관객체에 접근하지만, 테이블은 외래키를 이용한 **조인**을 사용해 연관테이블을 조회한다.

> 가장 큰 차이점은 객체는 근본적으로 **방향** 이 존재하지만, 테이블은 방향이 존재하지 않는다.



##### 객체를 테이블에 맞추어 모델링



```java
class Member {
  String id;
  Long teamId;
  String username;
}

class Team {
  Long id;
  String name;
}
```



> 테이블의 컬럼을 객체의 타입에 그대로 맞추어 매핑한 설계 방식이다.
>
> 하지만 객체지향의 장점을 살릴려면 객체 참조를 활용해야 한다.



##### 객체지향 모델링



```java
class Member {
  String id;
  String username;
  Team team;
}

class Team {
  Long id;
  String name;
}

Member member = jpa.find(Member.class, 1L);
member.getTeam(); // 참조를 통한 팀의 조회가 가능해진다.
```





> JPA는 연관관계와 관련된 패러다임의 불일치를 해결해준다.
>
> Member -> Team 의 관계를 설정하고 Member 객체를 저장하면, 참조를 외래키로 변환하여 SQL을 전달한다.



#### 객체 그래프 탐색



SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.



```java
class MemberService {
  public void process() {
    Member member = memberDAO.find(1L);
    member.getTeam(); // ??
    member.getOrder().getDelivery(); // ??
  }
}
```



> JPA 를 사용하면 객체 그래프를 마음껏 탐색할 수 있다.
>
> JPA는 실제 객체를 사용하는 시점에 **지연로딩** 을 사용해 데이터베이스에서 조회를 한다.



#### 비교



데이터베이스는 기본 키의 값으로 각 로우를 구분한다.

객체는 동일성 비교와 동등성 비교라는 두가지 방법이 있다.



- 동일성 비교는 == , 객체의 인스턴스 주소값을 비교한다.
- 동등성 비교는 equals() 메소드를 사용하여 객체 내부의 값을 비교한다.



**JPA 는 같은 트랜잭션 일때 같은 객체가 조회되는것을 보장한다.**

```java
String memberId = "USER_001";

Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2; // true
```



#### JPA란 무엇인가 ?



JPA는 java persistence api 의 약자로, 자바진영의 **ORM** 표준 기술이다.

애플리이션과 JDBC 사이에서 동작한다.



##### ORM이란 ?

- Object Relation Mapping 의 약자
- 객체와 관계형 데이터베이스를 매핑한다는 의미
- 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다.



##### JPA 소개

과거에 자바 빈즈 EJB라는 기술 표준을 만들었는데, 그 안에는 엔티티 빈이라는 ORM 기술도 포함되어 있었다.

하지만 너무 복잡하고 기술 성숙도도 떨어졌으며 J2EE 서버에서만 동작했고, 이때 하이버네이트 라는 오픈소스가 등장하게 된다.

자바진영에서 하이버네이트 개발자를 앉혀놓고 만든것이 JPA, 새로운 자바 ORM 기술 표준이다.



`버전별 특성`

- JPA 1.0 (JSR220) : 2006년 초기버전, 복합키와 연관관계 기능이 부족
- JPA 2.0 (JSR 317) : 2009년 대부분의 ORM기능을 포함하고 JPA Criteria가 추가됨
- JPA 2.1 (JSR 338) : 2013년 스토어드 프로시저 접근, 컨버터, 엔티티 그래프 기능이 추가됨



#### JPA를 사용해야하는 이유 ?



1. 생산성
   - JPA를 사용하면 자바 컬렉션에 객체를 관리하듯이 저장, 조회를 할 수 있다.
   - 반복적인 일은 JPA가 대신 처리해준다.
2. 유지 보수
   - 엔티티에 필드 하나만 추가해도 관련된 등록, 수정, 조회 SQL과 결과 매핑을 위한 처리를 JPA가 처리해준다.
   - 따라서 수정해야 할 코드가 줄어든다.
3. 패러다임의 불일치 해결
   - 상속, 연관관계, 객체 그래프탐색, 비교하기와 같은 패러다임 불일치 문제를 해결해준다.
4. 성능
   - 관계형 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.
5. 데이터 접근 추상화와 벤더 독립성
   - 관계형 데이터베이스는 벤더마다 사용법이 다른 경우가 많은데, JPA는 애플리케이션과 데이터베이스 사이에 추상화된 데이터 접근 계층을 제공하여 특정 기술에 종속되지 않도록 한다.

