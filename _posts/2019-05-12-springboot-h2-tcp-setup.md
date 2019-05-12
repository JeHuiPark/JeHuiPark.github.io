---
layout: posts
title:  "Springboot H2 RDBMS 사용하기"
date:   2019-05-12 16:26:00 +0900
comments: true
categories: java
tags:
  - spring
  - h2
---

## Springboot + 데모수준 앱 + 관계형 데이터베이스(RDBMS) + 인텔리제이(IntelliJ)
**이 4가지가 충족 된다면 H2 데이터베이스를 추천합니다.**
인텔리제이의 경우에 H2를 선택한다면 무설치에 가까운 행위로 손쉽게 데이타소스 설정이 가능합니다.

> [위키][h2]에 따르면 H2는 JAVA로 작성된 RDBMS로 JAVA앱에 임베디드 되거나 클라이언트-서버모드로 구동이 가능하다고 합니다.

H2를 임메디드모드로 사용하는 방법은 다른 블로그에서도 설명이 잘 되어있으니 생략하고,

여기에서는 H2를 서버(TCP)모드로 구동시키는 법을 소개할 것이며, 서버모드의 장점은 다른 DB처럼 데이터베이스 클라이언트를 활용하여 커넥션 할 수 있다는 장점이 있습니다.

전체 소스는 [깃허브][git]에서 확인하실 수 있습니다.

1. **프로젝트에 의존성 주입**

    ``` groovy
    implementation 'com.h2database:h2'
    implementation 'org.apache.tomcat:tomcat-jdbc:9.0.10'
    // implementation 'org.bgee.log4jdbc-log4j2:log4jdbc-log4j2-jdbc4.1:1.16' 드라이버스파이
    ```

1. **application 설정**

    ``` yml
    spring:
      datasource:
        continue-on-error: true
        #url: jdbc:log4jdbc:h2:tcp://localhost:9093/mem:management_db_9093 드라이버스파이 적용할 경우 사용
        #driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy 드라이버스파이 적용할 경우 사용
        url: jdbc:h2:tcp://localhost:9093/mem:management_db_9093
        driver-class-name: org.h2.Driver
        username:
        password:

        jpa:
          database-platform: H2
          show-sql: false
          hibernate:
            ddl-auto: update
    ```

    여기서 `jdbc:h2:tcp://localhost:9093/mem:management_db_9093` **이 라인을 이해해야셔야 다음 단계에서 수월하게 이해할 수 있습니다.**
    - `jdbc:h2:tcp://` 여기까지는 프로토콜 즉 약속입니다. 해석하자면 h2데이터베이스를 사용할 것이고 tcp(클라이언트)모드로 사용할 것이다. 이 정도의 의미가 됩니다.
    - `localhost:9093` 아시다시피 커넥션 할 목적지입니다.
    - `mem:management_db_9093` 이 부분은 모두가 알다시피 관리할 db 이름을 명시하는 곳인데 H2 DB를 TCP모드로 실행할 경우 `management_db_{tcpPort}`라는 데이터베이스가 자동으로 생성됩니다. database서버를 9093 포트로 서비스할 것이기 때문에 management_db_9093으로 명시해주었으며, `mem:`은 H2 드라이버에서 정의한 dbname 프로토콜로 보입니다.

    해당 내용은 org.h2.Driver패키지에서 확인가능하며, org.h2.server.TcpServer 클래스에서 정의되어 있습니다.
    ![h2 db prefix](https://user-images.githubusercontent.com/25237661/57580201-a2d76700-74e1-11e9-8914-cc7d81b65c22.png)
    ![h2 dbname](https://user-images.githubusercontent.com/25237661/57580223-f47ff180-74e1-11e9-86c0-29251e4517a9.png)





1. **테스트용 도메인**
    ``` Java
    @Builder
    @Data
    @Entity
    public class User {

      @Id
      @GeneratedValue
      private long id;
      @Column(unique = true)
      private String userId;
      private String userName;
      private int age;
    }

    @Repository
    public interface UserRepository extends CrudRepository<User, Long> {
    }

    @RequiredArgsConstructor
    @Service
    public class UserRegister {

      private final UserRepository userRepository;

      /**
       * 랜덤유저생성
       * @return 생성된 유저
       */
      public User randUser(){
        User rand = User.builder()
            .userId(System.currentTimeMillis() + "")
            .age(new Random().nextInt(31)+1)
            .userName(new String[]{"김철수", "김영희", "짱구", "맹구", "슛돌이", "도꺠비", "김삿갓"}[new Random().nextInt(7)])
            .build();
        return userRepository.save(rand);
      }
    }
    ```

1. **Java 설정**
    ``` Java
    @Slf4j
    @Configuration
    @ComponentScan(includeFilters = @Filter(OptionalComponent.class))
    @Profile("local")
    public class H2ServerConfiguration {


    	/**
    	 * @see org.h2.server.TcpServer
    	 * @return
    	 * @throws SQLException
    	 */
    	@Bean
    	@ConfigurationProperties("spring.datasource")
    	public DataSource dataSource() throws SQLException {
    		//Server server = adviceRun(9093, "external_db_name", "dbname", FilePath.absolute);
    		Server server = defaultRun(9093);
    		if(server.isRunning(true)){
    			log.info("server run success");
    		}
    		log.info("h2 server url = {}", server.getURL());

    		return new org.apache.tomcat.jdbc.pool.DataSource();
    	}

    	private Server adviceRun(int port, String externalDbName, String dbname, FilePath db_store) throws SQLException {
    		return Server.createTcpServer(
    				"-tcp",
    				"-tcpAllowOthers",
    				"-ifNotExists",
    				"-tcpPort", port+"", "-key", externalDbName, db_store.value2(dbname)).start();
    	}

    	private Server defaultRun(int port) throws SQLException {
    		return Server.createTcpServer(
    				"-tcp",
    				"-tcpAllowOthers",
    				"-ifNotExists",
    				"-tcpPort", port+"").start();
    	}

    	enum FilePath {
    		absolute("~/"),
    		relative("./");
    		String prefix;
    		FilePath(String prefix){
    			this.prefix = prefix;
    		}
    		public String value2(String dbname){
    			return prefix + dbname;
    		}
    	}
    }
    ```

    H2 DB도 [공식사이트][h2_official]를 살펴보면 많은 옵션을 지원하는거 같은데요 저에게 필요한 기능만 가져와 보았습니다.

    **기본옵션으로 TCP모드를 실행할 경우 데이터베이스를 인메모리로 서비스**하기 때문에 서버종료시 데이터를 유지할 수 없습니다. 하지만 저는 일반적인 DB처럼 **영속가능한 DB**를 원했기 때문에 사진에 보이는것 처럼 -key라는 옵션을 추가하였습니다.

    ![image](https://user-images.githubusercontent.com/25237661/57582566-3ec49b00-7501-11e9-81ab-37c7a0ac64aa.png)

    - key의 첫번째 인자값은 **외부용 dbname으로 externalDbName은 db커넥션시 명시해줘야할 dbname을 말합니다.
      > 해당옵션을 사용하였다면 datasource설정에서 url속성도 변경이 필요합니다.<br>
      ex) `spring.datasource.url: jdbc:h2:tcp://{address}:{port}/{externalDbName}`

    - key의 두번째 인자값은 내부적으로 관리될 dbname으로 **prefix로 ./또는 ~/을 붙여주어야 합니다.**

      - 두번째 인자값이 ./{dbname}일 경우에는 상대경로에 **{dbname}.mv.db**라는 파일이 생기며 해당 파일에 데이터영속화가 수행됩니다.
      *상대경로의 기본경로는 모듈경로입니다. h2.iml파일에 명시되어있네요.*

      - 두번째 인자값이 ~/{dbname}일 경우에는 절대경로에 **{dbname}.mv.db** 파일이 생기며 해당 파일에 데이터 영속화가 수행됩니다.
      *절대경로의 기본경로는 -baseDir 옵션을 따로 지정하지 않았으니 OS에 로그인된 계정의 root경로 입니다.*

1. **서버실행 후 DB 클라이언트로 접속하기**

    ![서버실행로그](https://user-images.githubusercontent.com/25237661/57582905-995ff600-7505-11e9-97f5-6cba6d0091e7.png)

    도깨비라는 이름을 가진 USER가 생성되었네요

    ![IntelliJ database](https://user-images.githubusercontent.com/25237661/57582949-1f7c3c80-7506-11e9-940e-7c694cdfd71d.png)

    **사진에 보이는 메뉴가 없으면 인텔리제이의 액션기능(Ctrl+Shift+A)에 Database를 입력하시면 됩니다.**

    ![IntelliJ data source](https://user-images.githubusercontent.com/25237661/57583012-19d32680-7507-11e9-85ab-47fb93aaf411.png)

    ![IntelliJ data source properties](https://user-images.githubusercontent.com/25237661/57583047-759daf80-7507-11e9-82b8-31e0a09eaea1.png)

    사진처럼 커넥션 타입을 URL only로 설정하신후 URL에 db서버 주소를 명시해주세요. (remote 선택하셔도 무방합니다.)

    ![image](https://user-images.githubusercontent.com/25237661/57583099-06748b00-7508-11e9-9ada-15794a04f686.png)

    ![image](https://user-images.githubusercontent.com/25237661/57583103-0eccc600-7508-11e9-91b3-720b62a3ad19.png)

    DB콘솔 활성화후 쿼리를 날려보니 아까전에 서버 실행로그에서 확인했던 도깨비라는 유저의 정보를 확인할 수 있네요!

개인공부용으로 쓰기 위해 제가 분석한 H2의 내용은 여기까지 입니다. 내용상에 오류가 있다면 지적 부탁드리며, 여기까지 따라오느라 수고하셨습니다!



[h2]:https://ko.wikipedia.org/wiki/H2 "위키로 이동"
[git]:https://github.com/JeHuiPark/spring-study/tree/master/h2 "깃허브로 이동"
[h2_official]:http://www.h2database.com/html/advanced.html "h2 공식사이트로 이동"
