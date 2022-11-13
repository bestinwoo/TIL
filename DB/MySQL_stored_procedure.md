# MySQL 저장 프로시저 (Stored Procedure)

DB 내부에 저장된 일련의 SQL 명령문을 하나의 함수처럼 실행하기 위한 쿼리의 집합

## CREATE STORED PROCEDURE

```sql
DELIMITER $$
CREATE PROCEDURE StoredProcedureName (IN | OUT | INOUT Parameter_name datatype[(length])
BEGIN
	SQL CODING...
END $$
DELIMITER ;

```

**IN 파라미터**

- 기본 파라미터 모드로 저장 프로시저를 호출할 때 인자로 전달받는다.
- 프로시저 내부에서 값을 수정하더라도 프로시저가 종료된 후 원래 값은 변경되지 않는다. 즉 저장 프로시저의 IN 매개변수는 원본의 복사본이다.

**OUT 파라미터**

- 매개변수 값은 저장 프로시저 내에서 변경할 수 있으며 새 값은 호출자에게 다시 전달된다.
- 초기값은 NULL로 프로시저는 OUT 파라미터의 초기값에 액세스할 수 없다.

**INOUT 파라미터**

- IN과 OUT을 합친 매개변수로 호출자에 의해 인수가 전달되고 저장 프로시저 내에서 매개변수를 수정하고 새 값을 호출자에게 다시 전달한다.

## WHILE문 예시

```sql
DECLARE i INT DEFAULT 0;
DECLARE sum INT DEFAULT 0;

WHILE i <= 10 DO
	SET sum = sum + i;
	SET i = i + 1;
END WHILE;
```

## IF문 예시

```sql
IF score >= 90 AND score <= 100 THEN
	SET grade = 'A+';
ELSE
	IF score >= 80 THEN
		SET grade = 'A';
	END IF;
END IF;	
```

## CASE문 예시

```sql
CASE GRADE
	WHEN 'VVIP' THEN
		SET 'discount' = 20;
	WHEN 'VIP' THEN
		SET 'discount' = 10;
	ELSE
		SET 'discount' = 5;
END CASE;
```

## CALL PROCEDURE

```sql
CALL StoredProcedureName();
```

- Stored Procedure를 처음 호출하면 MySQL은 DB의 catalog에서 이름을 찾아 프로시저의 코드를 컴파일하여 메모리 영역에 캐시로 저장한 후 프로시저를 실행한다.
- 동일한 세션에서 같은 Stored Procedure를 재호출하면 MySQL은 컴파일을 거치지 않고 기존에 있는 캐시로 실행한다.
- Stored Procedure에서 매개변수가 있을 경우 값을 전달하고 결과를 다시 반환받을 수 있다.
- Stored Procedure에는 IF, CASE, LOOP 등 control flow statements가 포함될 수 있다.
- Stored Procedure는 다른 Stored Procedure 또는 Stored Function을 호출하여 코드를 만들 수 있다.

### 프로시저 추가 문법

```sql
SHOW PROCEDURE STATUS; # 목록 확인
SHOW CREATE PROCEDURE 프로시저 이름; # 내용 확인
DROP PROCEDURE 프로시저 이름; # 프로시저 삭제
DROP PROCEDURE IF EXISTS 프로시저 이름; # 프로시저가 정의되어 있다면 삭제
```

## 프로시저의 장점

### Reduce network traffic

- Application과 MySQL Server 사이에 Network traffic을 줄이는데 도움이 된다.
- 긴 SQL문을 여러 개 전송하지 않아도 application이 저장 프로시저의 이름과 매개변수만 보내면 된다.

### Centralize business logic in the database

- 여러 Application에서 재사용 가능한 비즈니스 로직을 만들 수 있기 때문에 동일한 로직을 복제하려는 노력을 줄일 수 있고 DB의 일관성을 향상시킬 수 있다.

### Make database more secure

- 특정 Stored Procedure에 접근할 수 있는 권한만을 부여하여 기본 테이블을 더 안전하게 관리할 수 있다.

## 프로시저의 단점

### Resource usages

- Stored Procedure를 많이 사용할 경우 메모리 사용량이 증가할 수 있다.
- 또한 MySQL이 logical operations를 잘 할 수 있게 설계되지 않았기 때문에, 저장 프로시저에서 과도하게 많은 연산을 할 경우 CPU 사용량이 증가할 수 있다.

### Troubleshooting

- Stored Procedure를 디버깅 하기 어렵다.
- MySQL은 Stored Procedure 디버깅 기능을 제공하지 않는다. (Oracle 같은 Enterprise DB는 제공)

### Maintenances

- Stored Procedure를 개발하고 유지, 관리하는 데 개발 리소스가 많이 필요하다. 그렇기 때문에 유지보수의 문제가 발생할 수 있다.

### 참고

[https://www.mysqltutorial.org/introduction-to-sql-stored-procedures.aspx](https://www.mysqltutorial.org/introduction-to-sql-stored-procedures.aspx)

[https://fourierdev.medium.com/mysql-stored-procedure-알아보기-in-mysql-1fdd342b661a](https://fourierdev.medium.com/mysql-stored-procedure-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-in-mysql-1fdd342b661a)

[https://spiderwebcoding.tistory.com/7](https://spiderwebcoding.tistory.com/7)