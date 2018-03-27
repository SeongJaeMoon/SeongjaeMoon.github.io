---
layout: post
title:  "Oracle + Docker + OJDBC"
date:   2018-02-18 23:00:00
author: Seongjae Moon
categories: DB
tags:   Oracle 오라클 Docker OJDBC JAVA
---

[저번 포스팅](https://seongjaemoon.github.io/2018/02/18/database-oracle5/)을 통해 오라클의 간단한 쿼리문에 대해서 알아보았다. 오라클은 RDMBS 중에서도 진짜 잘 만들어진 DBMS로서 다양한 기능을 제공하고 있다. 검색 쿼리 최적화라던지 이런 저런 기능들은 너무 종류가 많아서 다 정리하기 벅차므로 API 문서 등을 열심히 찾아보는 걸로 하고, Oracle 11g와 OJDBC를 통해 JAVA 환경에서 실제 데이터베이스를 활용하는 방법에 대해서 정리하고자 한다.  

여기서 Docker라는 친구가 필요한데, 그 이유는 Mac OS에서는 오라클을 지원하지 않기 때문이다. 이러한 이유로 가상 머신을 통해 돌리거나 다른 DBMS를 활용해서 개발해야 한다. Windows나 Linux 등 다른 운영체제를 사용하는 사용자는 오라클을 다운로드 받고 오라클 서버를 바로 실행하면 된다.

또한, SQL 쿼리문을 테스트 하고 실제 오라클 데이터베이스에 튜플을 확인하기 위해 SQL developer라는 프로그램을 사용한다.

#### 1. 오라클 사용 준비.
##### 우선 오라클을 사용하기 위해서 오라클을 다운로드 받아야 한다.
Windows, Linux 운영체제에서 사용하기 위해선 [여기](http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html)에서 오라클 서버를 다운로드 받는다.

Mac OS에서 사용하기 위해선 Docker를 이용해 별도로 오라클 이미지를 사용해 연동을 진행해야 한다. 우선, Docker라는 친구를 [여기](https://www.docker.com/docker-mac)에서 다운로드 받는다. Docker 정보에 대해선 [여기](https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html)에 아주 잘 정리 되어 있으니 참고하자.

![Docker 실행](//assets/uploads/docker.png)
Docker를 다운로드 받고 실행을 하면, 상태바에 고래 모양을 띈 귀여운 친구가 나타난 것을 확인할 수 있다. 고래 모양을 클릭한 후 Docker is running 이라는 창이 확인되면 준비가 완료된다.

이제 터미널을 열고 아래 명령어를 작성해준다. 이렇게 명령어를 작성하면 Docker 컨테이너 Oracle 11g 이미지를 내려받는다.
```
docker pull wnameless/oracle-xe-11g
```
다음은 내려 받은 컨테이너를 실행시킨다. 다음 명령은 도커 컨테이너 1521 포트를 로컬호스트의 59161 포트에 연결한다는 명령이 된다. 포트 넘버의 변경으로 타 서버의 연결도 가능하다.
```
docker run -d -p 59160:22 -p 59161:1521 wnameless/oracle-xe-11g
```

##### 쿼리문 작성을 위해 SQL developer를 다운로드 받는다.
많은 사람들이 SQL 쿼리를 작성하고 DB를 관리하기 위해 애용하는 GUI 프로그램인 SQL developer를 [여기](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)에서 운영체제에 맞게 다운로드 하고 실행한다.

처음 SQL developer를 실행하면 아래와 같은 화면이 나타나게 된다. 현재 과정에선 관리자로 접속하는 것이니 초기 아이디는 system로 비밀번호는 oracle로 되어있다. 이 접속 아이디와 비밀번호로 접속이 이루어지게 된다.
![오라클 접속 설정](//assets/uploads/sqldev.png)

##### OJDBC를 다운로드 받는다.
오라클을 JAVA 개발 환경에서 사용하기 위한 인터페이스 라이브러리로 ojdbc6.jar 파일을 [여기](http://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html)에서 다운로드 한다.

ojdbc6.jar를 사용하기 위해서 빌드 경로 설정을 해주어야 한다. Project Explorer에서 해당 프로젝트를 우클릭해서 Build Path > Configure Build Path... > Libraries > Add External JARs...를 순서대로 누르고, 다운로드 받은 ojdbc6.jar를 선택한 후 Apply 해준다.

정상적으로 작업이 이루어지면 아래와 같이 ojdbc.jar가 외부 라이브러리에 추가된다.
![ojbc.jar연결](//assets/uploads/ojdbc6.png)

- 가끔 Git과 연동하는 도중에 이 빌드 경로 설정 문제 때문에 에러가 발생하곤 한다. 그럴 땐 기존의 ojdbc.jar 파일을 삭제하고 다시 설정 해주면 된다.

#### 2. 개발 진행.
위 단계에서 개발 준비가 완료 되었으니, 이제 실제로 개발을 진행 하면 된다. 아래는 테스트 이클립스에서 작성된 테스트 코드이다.

- 오라클 데이터베이스와 연결하기 위한 SQLConnection이라는 클래스를 작성한다. DB 연결 변동 사항이 발생할 경우를 대비하여 하나의 클래스에 DB 연결 관리 부분을 모두 작성하는게 정신 건강에 좋다고 할 수 있다.

```java
package com.sistmng;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class SQLConnection {
	//JDBC를 사용하기 위한 변수 선언.
	private static final String JDBC_DRIVER = "oracle.jdbc.driver.OracleDriver";
	private static final String DB_URL = "jdbc:oracle:thin:@127.0.0.1:59161:xe";
	//DB 접속 아이디, 비밀번호.
	private static final String USER = "system"; //아이디
	private static final String PASS = "oracle"; //비밀번호

	private static Connection conn;
	//DB 연결 메소드 선언.
	public static final Connection connect() throws ClassNotFoundException, SQLException{

		Class.forName(JDBC_DRIVER);

		conn = DriverManager.getConnection(DB_URL, USER, PASS);

		return conn;
	}
	//DB 연결 종료 메소드 선언.
	public static void close() throws SQLException{
		if(conn!=null) {
			conn.close();
		}
	}
}
```
- 아래 코드는 학생 정보 관리용 DAO 클래스에서 회원 정보를 가져오는 간단한 코드의 예이다.
```java
package com.sistmng.student;

import java.util.*;

import com.sistmng.Current;
import com.sistmng.SQLConnection;
import java.sql.*;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class StudentDAO {

	private LocalDate now = LocalDate.now();
	private DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd");

  //회원정보출력
  public Student menu_1() {
		//DTO 클래스인 Student 타입의 결과를 반환할 객체를 생성한다.
		Student student = new Student();
		/*
		 SELECT COUNT(sh.mid) AS count_, m.name_, m.ssn, m.phone
		 FROM studentHistory_ sh, member_ m, student_ s
		 WHERE sh.mid = s.mid
		 AND s.mid = m.mid
		 AND sh.mid = '?'
		 GROUP BY  m.name_, m.ssn, m.phone ;
		 */
		Connection conn = null;
		PreparedStatement pstmt = null;
		String sql = "SELECT COUNT(sh.mid) AS count_, m.name_, m.ssn, m.phone FROM studentHistory_ sh, member_ m, student_ s WHERE WHERE sh.mid = s.mid AND s.mid = m.mid AND sh.mid = ? GROUP BY  m.name_, m.ssn, m.phone";

		try {
			//앞서 설정한 DB를 연결하기 위한 변수를 선언한다.
			conn = SQLConnection.connect();
			//보안상의 이유로 prepareStatement 메소드를 사용해서 오라클에 쿼리를 한 번만 업로드 할 수 있도록 한다.
			pstmt = conn.prepareStatement(sql);
			//물음표 기호의 부분에 들어갈 값 설정. 각각 인자는 (인덱스, 값)
			pstmt.setString(1, Current.getInstance().getCurrent());

			//SELECT 쿼리의 결과를 반환한다.
			ResultSet rs = pstmt.executeQuery();
			while(rs.next()) {
				String name_ = rs.getString("name_");
				String ssn = rs.getString("ssn");
				String phone = rs.getString("phone");
				int courseNumber = rs.getInt("count");
				student.setName_(name_);
				student.setSsn(ssn);
				student.setPhone(phone);
				student.setCourseNumber(courseNumber);
			}
			//반복문을  통해 setter를 통해 student 객체의 값을 할당한다.
			rs.close();
		} catch (SQLException se) {
			se.printStackTrace();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (pstmt != null)
					pstmt.close();
			} catch (SQLException se) {

			}
			try {
				SQLConnection.close();
			} catch (SQLException se) {
				se.printStackTrace();
			}
		}
		return student;
  }
}
```
- 위와 같이 DAO 클래스에서 DB의 CRUD 작업을 구성하고, 서비스 클래스나 다른 클래스에서 DAO 클래스의 메소드를 호출해서 사용하면 된다.  
#### 3. 오라클 DB 확인.
SQL developer를 통해 오라클 DB의 CRUD 작업을 진행할 수 있으며, 쿼리문을 이리 저리 날려볼 수 있다. 이클립스에선 자바 코드만을 컨트롤하여 적절한 분업?이 가능해졌다!
![SQL developer SELECT 쿼리 테스트](//assets/uploads/sqldev2.png)

처음 대학에서 Oracle 데이터베이스를 접했을 때는 가상 머신 위에서 오라클을 구동하고, 윈도우 프롬프트 창에서 열심히 한줄 한줄 정성스레? 코드를 작성했었던 기억이 난다. 세상이 점점 좋아지고 있다는 걸 실감한다... ~~갑자기?~~

* 오타나 잘못된 부분을 지적 해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [튜토리얼 포인트 JDBC](https://www.tutorialspoint.com/jdbc/index.htm)
* [Docker for mac](https://docs.docker.com/docker-for-mac/)
* [Docker&Oracle 연동 참고](http://jojoldu.tistory.com/169)