---
layout: posts
title:  Load Time Weaving 적용기 - Spring LTW
date:   2019-05-25 22:26:00 +0900
comments: true
categories: java
tags:
  - spring
  - ltw
---

스프링 환경에서 IOC 대상이 아닌 일반객체도 별도의 코드 작성없이 스프링 컨테이너로부터 DI 받을 수 있는 방법을 공유합니다.

전체소스는 **[깃허브][git]** 에 존재합니다.

## 의문점에서 시작

  **Spring환경에서 개발을 진행하던중 문득 일반객체도 스프링 컨테이너에 빈으로 등록된 객체들을 별도의 코드작성없이 Autowired와 같은 어노테이션을 이용하여 DI받을 수는 없을까란 의문점이 생겼습니다.**

  즉, 제가 하고싶었던 것은 객체에 대한 제어는 제가 직접하지만, 별도의 코드작성없이 스프링 컨테이너에 등록된 빈들을 DI받는게 목적이 였습니다.

## 왜 필요했는지?

  **결론은 도메인주도 개발을 하고싶었습니다.**

  서버앱을 개발하다보면 어떤 규칙에 의해서 DB에서 관리되는 도메인의 속성이 변경되는 패턴이 상당히 많은 비중을 차지하게 됩니다. 그렇기 때문에 소프트웨어 구조적으로 따졌을때 이런 기능들은 서비스 레이어에 정의 되기보다는 도메인 레이어에 정의하고 서비스레이어는 이러한 기능들을 상황에 알맞게 호출만 하는것이 더 좋은 패턴이라고 저는 생각하고, 해당 방법으로 개발을 진행하였습니다.
  하지만, 저는 이 방법이 아직은 완전하지 않은 방법이라고 생각했습니다.
  Spring에서 보통 DB엑세스는 IOC로부터 관리되는 객체들을 이용하여 수행될텐데 도메인 객체는 IOC대상에 포함되지 않아서 DI가 불가능하기 때문에 외부상황에 의존되는 상황이였기 때문이죠.

## 무엇이 필요한지?

  IOC 대상에는 포함되지 않는, 즉 런타임에 필요할 때 마다 생성되는 객체지만 추가 코드없이 스프링 컨테이너로부터 자동으로 DI받을 수 있는 기술이 필요합니다. 이게 가능하려면 aspectJ가 필요할 것입니다.
  클래스가 객체화되는 시점에 필요한 객체들이 DI되어야 하기 때문이죠

## 사전 지식
  스프링에서 제공하는 AOP 기능은 IOC 대상에 포함되는 객체에만 사용할 수 있는 기술입니다. 그 이유는 스프링프레임웍이 부팅되면서 IOC 대상에 포함시킬 모든 클래스들을 찾아서 컨테이너에 빈으로 등록하게 될텐데 이때 해당 빈들은 모두 프록시 객체로 감싸지게 되어 실제 어떤 기능이 실행되더라도 프록시 객체를 통해 실행된다는 점을 이용하여 구체화된 기술이기 때문입니다. 이것을 RTW(RunTimeWeaving)이라고 부릅니다.

## 찾아보고 적용해보기

  1. 우선 해당기능이 구현된 LoadTimeWeaver 의존성을 추가해줍니다.

      LoadTimeWeaver는 클래스로드 타임에 위빙하여 일반 클래스 또한 스프링에 의해 AOP처리가 가능하도록 하는 라이브러리입니다.

      ```gradle
      dependencies {
          // ...
          implementation 'org.springframework.boot:spring-boot-starter-aop'
          implementation 'org.springframework:spring-instrument'
      }

      test.doFirst {
          def instrumentLib = instrumentLibPath()
          jvmArgs "-javaagent:${instrumentLib}"
      }

      bootRun.doFirst {
          def instrumentLib = instrumentLibPath()
          jvmArgs "-javaagent:${instrumentLib}"
      }

      File instrumentLibPath(){
          return sourceSets.getByName("main").compileClasspath.find {
              cls -> return cls.getName().contains("spring-instrument")
          }
      }
      ```
      **로드타임에 위빙하기 위해서는 spring-agent를 통해 jvm에 로드되어야 합니다. 때문에 jvm agent옵션으로 spring-instrument라이브러리 경로를 넘겨주어야 합니다.**
      저같은 경우는 실행할때마다 라이브러리 경로를 직접 명시해주는게 ~~너무~~ 싫어서 위에처럼 라이브러리 경로를 자동으로 찾고 jvm옵션으로 전달하도록 하였습니다.
      <br><br>

  1. Application 최소설정     
      ```java
      @Configuration
      @EnableSpringConfigured
      @EnableLoadTimeWeaving
      public class Config {

      }
      ```
      - EnableLoadTimeWeaving 어노테이션은 로드타임위빙이 가능하게 합니다.
      - EnableSpringConfigured 어노테이션은 일반클래스 또한 스프링설정을 주입받는게 가능하게 합니다.
      <br><br>

  1. **Configurable 어노테이션을 이용하여 DI옵션 활성화**

      ```java
      @Configurable(autowire = Autowire.BY_TYPE)
      @Getter@Setter
      @Entity
      @Slf4j
      public class User {

        @Transient
        private  UserRepository userRepository;

        public void setUserRepository(UserRepository userRepository){
          this.userRepository = userRepository;
          log.info("Auto Dependency Injection");
        }

        @Id
        @GeneratedValue
        private long id;
        private String userId;
        private String userNm;
        private String status;

        /**
         * 스테이터스 값 변경
         * @param status
         */
        public void statusChange(String status){
          log.info("status origin = {}, status new = {} ", this.status, status);
          this.status = status;
          userRepository.saveAndFlush(this);
        }
      }
      ```
      Configurable 어노테이션은 EnableSpringConfigured 어노테이션이 활성화 되있다면 DI 받겠다는 설정입니다.
      **이 설정으로 User클래스는 IOC에게 관리되지 않지만, 스프링 컨테이너로부터 DI받는게 가능해집니다**
      <br><br>

  1. **AOP 로그 소스 작성**

      ```java
      @Aspect
      @Slf4j
      public class UserInitAspect {

        @Before("execution(com.example.configurable.user.domain.User.new())")
        public void userInitBefore(){
          log.info("com.example.configurable.user.domain.User 생성전 ");
        }

        @After("execution(com.example.configurable.user.domain.User.new())")
        public void userInitAfter(){
          log.info("com.example.configurable.user.domain.User 생성후 ");
        }
      }
      ```
      **User클래스는 IOC에게 관리되는 클래스가 아니기때문에 컴포넌트로 등록해선 안됩니다.**
      <br><br>


  1. **AOP 설정**

      aop 설정파일 추가가 필요합니다. spring-agent에서 로드합니다.

      ![aop.xml 경로](https://user-images.githubusercontent.com/25237661/58369042-ebeed880-7f2f-11e9-9221-d9079be389a1.png)

      aop.xml
      ```xml
      <aspectj>
        <weaver options="-Xset:weaveJavaxPackages=true" >
          <include within="com.example.configurable..*"/>
        </weaver>
        <aspect name="com.example.configurable.aop.UserInitAspect"/>
      </aspectj>
      ```
      <br><br>

  1. **Test 코드작성**

      ```java
      @RunWith(SpringRunner.class)
      @SpringBootTest
      public class ApplicationTest {

        @Autowired
        UserRepository userRepository;

        @Test
        public void contextLoadTest() {
          User user = new User();
          Assert.assertNotNull(user.getUserRepository());
        }

        @Test
        public void ltwTest(){
          final String userId = "아이디";
          User user  = new User();
          user.setUserId(userId);
          user.setUserNm("박제희");
          user.setStatus("9");
          userRepository.saveAndFlush(user);

          User persistUser = userRepository.findByUserId(userId);
          persistUser.statusChange("3");
          User test = userRepository.findByUserId(userId);
          assert "3".equals(test.getStatus());
        }
      }
      ```
      <br><br>

## 테스트 해보기

  앞서 알려드린 것 처럼 jvm agent 옵션으로 spring-agent경로를 넘겨주어야 하며 두가지 방법이 존재합니다.

  - IDE
    IDE를 이용하여 실행할 경우 Run/Debug Configuration에서 jvm옵션을 지정해주어야 합니다.

    ![idea-test](https://user-images.githubusercontent.com/25237661/58369101-c57d6d00-7f30-11e9-9acd-855ca36a1d24.png)

  - gradle task
    gradle을 이용할 경우 spring-agent경로를 알아서 찾을것이기 때문에 별도의 jvm옵션 지정은 필요하지 않습니다.

    ![gradle-task](https://user-images.githubusercontent.com/25237661/58369211-23f71b00-7f32-11e9-97dd-77cc0b3d0c3a.png)


  ![test-log](https://user-images.githubusercontent.com/25237661/58369148-72f08080-7f31-11e9-932f-899ff550c28b.png)

  테스트 결과입니다. User클래스는 bean이 아니지만, DI가 정상적으로 실행되었습니다. 또한, 테스트용으로 작성한 Aspect 코드도 동작하는 모습을 확인하실 수 있습니다.

---

  **마지막으로 DI받는 과정을 이미지로 첨부하며 글을 마치겠습니다!!!**

  ![di-handle](https://user-images.githubusercontent.com/25237661/58366604-cc47b800-7f0f-11e9-9f9c-d22e6a9846e8.png)


[git]:https://github.com/JeHuiPark/spring-study/tree/master/configurable "깃허브로 이동"
