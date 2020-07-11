---
layout: posts
title:  Springboot 2.2.x MockMvc 인코딩 이슈
date:   2020-07-11 19:22:17 +0900
comments: true
categories: spring
---

* TOC
{:toc}

## SpringBoot MockMvc 한글깨짐
SpringBoot 2.1.x 를 사용하다가 2.2.x 로 버전을 변경한 이후에 눈에 거슬리는게 생겼다.  
`MockMvc` 에는 요청과 응답에 관한 정보를 이쁘게 출력해주는 기능이 포함되어 있는데, 
버전을 변경한 이후 MockMvc 에서 출력시 이용하는 인코딩이 맞지 않아 글자가 박살난 것을 확인할 수 있다.  
![image](https://user-images.githubusercontent.com/25237661/87222377-1eff6000-c3ae-11ea-89bd-353cae7f4038.png){:width="300px"}

이 한글깨짐 현상은 SpringBoot 2.2 로 넘어오면서 적용된 변경사항에 대한 사이드 이펙트로 추측 (이거 버그 같은데)  
> *Spring 에서 `MediaType.APPLICATION_JSON_UTF8` 사용중단 결정이 내려지고, 해당 변경사항이 SpringFramework 5.2 버전에서 적용이 되었는데, SpringBoot 2.2.x 버전부터 SpringFramework 버전을 5.2 변경*

아무튼 실제 코드 동작에는 영향이 없기도 하고 귀찮아서 그냥 방치하고 있었는데, MockMvcFilter 에 UTF8 인코딩 필터를 추가하면 된다는 정보를 확인했다. 

## 해결법 찾기
좋은 정보를 얻었으니 직접 해결 해보자
  
우선 현재의 테스트 코드는 어플리케이션 컨텍스트에서 `MockMvc` 를 주입받고 있는 방식이니 `@AutoConfigureMockMvc` 의 주석을 확인 해보았다.  
![image](https://user-images.githubusercontent.com/25237661/87222728-46a3f780-c3b1-11ea-852f-9b3cf2a7171a.png)

주석에서 시키는대로 `MockMvcAutoConfiguration` 와 `SpringBootMockMvcBuilderCustomizer` 를 확인해 보았다.

`MockMvcAutoConfiguration` 의 일부분인데 자주 보이는 패턴이라서 어떤 느낌인지 알 것 같다.  
*자동 설정에 대하여 확장 포인트를 제공하기 위해 이런식으로 `hook` 을 제공하는 패턴이 많음*
![image](https://user-images.githubusercontent.com/25237661/87222798-042eea80-c3b2-11ea-9858-6cee042fb4a8.png)

그리고, `SpringBootMockMvcBuilderCustomizer` 를 살펴보자.   
- `SpringBootMockMvcBuilderCustomizer` 는 `MockMvcAutoConfiguration` 에서 확인했던 `MockMvcBuilderCustomizer` 인터페이스의 기본 구현체다.
- `builder` 를 이용하여 `MockMvc` 에 필터를 추가할 수 있겠다는 정보를 얻을 수 있다. 
![image](https://user-images.githubusercontent.com/25237661/87222819-3e988780-c3b2-11ea-987b-9706f6155af1.png)

## 해결

1. 커스텀 설정 어노테이션을 작성한다
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @AutoConfigureMockMvc
    @Import({
        SampleConfig.Sample.class,
    })
    @interface SampleConfig {
    
      class Sample {
        @Bean
        MockMvcBuilderCustomizer utf8Config() {
          return builder ->
              builder.addFilters(new CharacterEncodingFilter("UTF-8", true));
        }
      }
    }
    ```
1. 테스트에 커스텀 설정을 추가한다
    ```java
    @SampleConfig
    @SpringBootTest
    class SampleApplicationTest {
    
      @Autowired
      MockMvc mockMvc;
    
      @Autowired
      ObjectMapper objectMapper;
    
      @Test
      void test2() throws Exception {
        var json = objectMapper.writeValueAsString(Map.of("k", "한글"));
    
        mockMvc.perform(
            get("/sample")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json)
        ).andDo(print());
    
      }
    }
    ```

테스트 코드를 변경했으니 `MockMvc` 인코딩이 깨지나 안깨지나 로깅을 확인 해보자  
![image](https://user-images.githubusercontent.com/25237661/87223171-a9978d80-c3b5-11ea-96cb-c23f938cc016.png){:width="300px"}

안깨진다 👍👍👍 샘플 코드는 [여기에서](https://github.com/JeHuiPark/blog-sample/tree/master/boot2-2-x-mock-mvc-sample) 

## 정리
영어 능력이 부족해서 100% 맞는 내용이다 장담할 수는 없지만 나름대로 요약하면 
- RFC 7159 가 폐기되고, RFC 8259 로 대체 되었다.  
- RFC 8259 에 따르면 JSON 문자열은 UTF-8 을 이용하여 인코딩 하여야 한다.
- 메이저 브라우저인 크롬에는 이미 RFC 8259 스펙이 적용되어 있다.
- 스프링에서도 RFC 8259 스펙을 적용하기로 결정하였다.
- RFC 8259 스펙 적용은 SpringFrameWork 5.2 버전에서 이루어졌다.
- SpringBoot 2.2.x 버전은 SpringFrameWork 5.2 버전을 사용한다.
- SpringBoot 테스트 도구에는 RFC 8259 스펙 적용이 되지 않은듯 하다. (2.3.0 버전도 확인해 보았으나, 동일한 이슈 존재)

## 관련링크
- [샘플 코드](https://github.com/JeHuiPark/blog-sample/tree/master/boot2-2-x-mock-mvc-sample)
- [SpringBoot 2.2 Release Note](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes)
- [Upgrading to Version 5.2](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x#deprecation-of-mediatypeapplication_json_utf8-and-mediatypeapplication_problem_json_utf8)
- [미디어 타입 깃헙 이슈](https://github.com/spring-projects/spring-framework/issues/22788)
- [RFC 8259](https://tools.ietf.org/html/rfc8259)
- [RFC 7159](https://tools.ietf.org/html/rfc7159)
