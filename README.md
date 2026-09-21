JDBC & Hibernate — 사용자 관리 (DAO 패턴)

학습용 Java 애플리케이션입니다. MySQL에 사용자 테이블을 만들고 사용자 데이터를 저장·조회·삭제하는 동일한 요구사항을 순수 JDBC와 Hibernate 두 가지 방식으로 구현해 비교했습니다. (Habsida 교육 과정 과제)
설명
`UserDao` 인터페이스 하나에 두 가지 구현체를 두고, Service 계층에서 구현체만 바꾸면 JDBC ↔ Hibernate로 전환됩니다.
> Hibernate 구현은 [`hibernate-impl` 브랜치](https://github.com/Pan4ife/JDBC-Task/tree/hibernate-impl)에 있습니다. 기본 브랜치에는 JDBC 구현까지 포함되어 있습니다.
스택
JDBC — `DriverManager`, `PreparedStatement`, `try-with-resources`
Hibernate 5.6 — `SessionFactory`, `Session`, `Transaction`, HQL
MySQL 8 — 데이터베이스 (Connector/J 8.0.25)
Maven — 빌드
JUnit 4 — 테스트 (과제 제공 코드)
Java 17
아키텍처
```
Main
  ↓
Service (UserServiceImpl)            ← DAO 호출
  ↓
UserDao (interface)
  ├── UserDaoJDBCImpl                ← 순수 JDBC
  └── UserDaoHibernateImpl           ← Hibernate (Session/Transaction)
  ↓
MySQL
```
기능 (UserDao)
메서드	설명
`createUsersTable()`	`users` 테이블 생성
`dropUsersTable()`	`users` 테이블 삭제
`saveUser(name, lastName, age)`	사용자 저장
`removeUserById(id)`	ID로 사용자 삭제
`getAllUsers()`	전체 사용자 조회
`cleanUsersTable()`	테이블 데이터 전체 삭제
JDBC와 Hibernate 구현 비교
항목	JDBC (`UserDaoJDBCImpl`)	Hibernate (`UserDaoHibernateImpl`)
연결	`Util`에서 `DriverManager`로 직접 연결	`HibernateUtil`이 `SessionFactory`를 지연 초기화 싱글톤으로 제공
저장	`INSERT` + `PreparedStatement` 파라미터 바인딩	`session.save(user)`
삭제	`DELETE ... WHERE id = ?`	`session.get` 후 `session.delete`
조회	`ResultSet`을 `User`로 직접 매핑	HQL `from User`
자원 관리	`try-with-resources` (Connection, Statement)	`try-with-resources` (Session) + `Transaction` 관리
실행
MySQL 8에 사용할 데이터베이스를 생성합니다.
DB 접속 정보를 본인 환경에 맞게 수정합니다.
JDBC: `util/Util.java`
Hibernate: `util/HibernateUtil.java`
사용할 구현체를 `UserServiceImpl`에서 선택합니다.
```java
   private final UserDao userDao = new UserDaoHibernateImpl(); // 또는 UserDaoJDBCImpl
```
`Main`을 실행합니다. 테이블 생성 → 사용자 4명 저장 → 전체 조회 출력 → 데이터 삭제 → 테이블 삭제 순서로 동작합니다.
프로젝트 구조
```
src/main/java/jm/task/core/jdbc/
├── Main.java
├── dao/        — UserDao, UserDaoJDBCImpl, UserDaoHibernateImpl
├── model/      — User (JPA 엔티티)
├── service/    — UserService, UserServiceImpl
└── util/       — Util (JDBC 연결), HibernateUtil (SessionFactory)
src/test/java/
└── UserServiceTest.java   — 과제에서 제공된 테스트 코드
```
