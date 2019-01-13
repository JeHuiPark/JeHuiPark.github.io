---
layout: posts
title:  "GoogleAnalytics Report API 연계"
date:   2018-09-02 17:50:21 +0900
comments: true
categories: study
tags:
  - googleanalytics
redirect_from:
  - /googleanalytics/2018/09/02/ga_report_api_guide/
  - /googleanalytics/ga_report_api_guide/

---

GoogleAnalytics를 들어보셨나요.
GoogleAnalytics는 개발자가 약간의 노력만으로 특정 서비스를 이용하는 사용자들의 활동을 추적할 수 있게 도와주는 아주 유용한 도구입니다.
물론 추적 데이터 또한 구글 자체서버에 저장되고 있으니 기본적인 기능만 이용할꺼라면 개발자는 정말 아무것도 할 게 없게 만들어주는 강력한 도구이죠.
이러한 추적데이터를 [GoogleAnalytics 공홈](https://analytics.google.com/analytics)에서 확인 할 수도 있지만 구글은 고맙게도 추적데이터를 개발자가 직접 서비스 할 수 있게 **GoogleAnalytics Report API** 서비스를 제공하고 있습니다.

이번 글에서는 바로 java로 개발된 서버에서 **GoogleAnalytics Report API** 를 어떻게 연계하는지에 대해 공유해보겠습니다.

<br>
1. 우선 Google Play Console페이지로 이동하여 아래와 같은 순서로 프로젝트를 생성.
(*GoogleAnalytics서비스와 GoogleAnalytics Report API는 다른 서비스이며 GoogleAnalytics Report API는 GoogleAnalytics의 데이터를 조회하기 위한 인터페이스*)
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958518-6d26bb00-af1c-11e8-8c9d-86c173c277e6.png)

2. 좌측에 라이브러리 메뉴에 들어가서 **Google Analytics Reporting API** 를 찾아서 사용함으로 설정.

3. GoogleAnalytics Reporting API 사용을 위한 API 인증정보 생성. (**추후 Google Analytics 콘솔에서 데이터조회 권한을 부여할 때 사용됨**)
아래와 같이 서비스 계정관리를 클릭하여 프로젝트 서비스계정을 발급하는 페이지로 이동하여 서비스 계정 발급
(_발급받는 과정중 마지막에 **json파일을 제공해주는데 반드시! 잘! 갖고 있어야 합니다.**_)
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958519-6d26bb00-af1c-11e8-9bff-b4844d6fffd9.png)

4. 서비스 계정을 발급받으면 생성되는 이메일 주소를 복사합니다.
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958520-6d26bb00-af1c-11e8-8419-ed4c76bc35f4.png)

5. 아래와 같이 GoogleAnalytics콘솔로 돌아가서 데이터 조회 **권한을 서비스 계정에 부여**합니다.
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958522-6dbf5180-af1c-11e8-9c06-08fc9ba05087.png)

    이렇게 **GoogleAnalytics** 와 **GoogleAnalytics Report API** 연계는 마무리가 됩니다.

6. 소스레벨에서  Report API 사용하기

    **구글에서 제공하는 [API가이드](https://developers.google.com/analytics/devguides/reporting/core/v4/quickstart/service-java)를 간략하게 요약해볼게요**
    <br>

    1. **ReportRequest클래스를 이용하여 요청 쿼리 파라미터를 작성** (아래는 예시입니다.)

        [요청 파라미터 작성 예제 사이트](https://ga-dev-tools.appspot.com/query-explorer/)

        ```java
        ReportRequest request = new ReportRequest()
            .setViewId(GAHelper.VIEW_ID) // GoogleAnalytics View Id
            .setDateRanges(this.helper.createDateRange(startDate, endDate)) // 데이터 조회 기간
            .setMetrics(this.helper.createMetric()) // 어떤 통계를 뽑을지
            .setDimensions(this.helper.createDimension()) // 통계를 어떤 관점에서 바라볼지
            .setIncludeEmptyRows(true); // 데이터가 없는 행도 포함 시킬 것인지.
        ```

        *View ID는 아래처럼 GoogleAnalytics에서 확인 할 수 있어요.*
        ![view_id](https://user-images.githubusercontent.com/25237661/45264558-875f1c80-b479-11e8-8897-7343801e2a2f.png)

    2. **사전에 발급받은 json파일을 read하여 유효한 사용자인지 검증 후 구글로부터 해당 API에 대한 액세스 토큰을 발급 받습니다.**

        ```java
        HttpTransport httpTransport = GoogleNetHttpTransport.newTrustedTransport();
        		GoogleCredential credential = GoogleCredential.fromStream(new FileInputStream(file))
        				.createScoped(AnalyticsReportingScopes.all());
        AnalyticsReporting.Builder(httpTransport, GSON_FACTORY, credential)
        				.setApplicationName(APPLICATION_NAME).build();
        ```

    3. **요청 파라미터를 set하여 API를 호출하고 응답을 받습니다.**

        ```java        
        GetReportsRequest getReport = new GetReportsRequest().setReportRequests(requests);
        GetReportsResponse response = service.reports().batchGet(getReport).execute();
        ```

        ![ga_report_api_07](https://user-images.githubusercontent.com/25237661/44958517-6d26bb00-af1c-11e8-9495-b1ac8185c9af.png)

<br>

**필자의 경우는 특정기간에 사용자별, 브라우저별 등등 통계 타입이 사전에 정의가 되어 있었기에 추후 확장성을 고려하여 API 호출 파라미터를 작성하기 편하도록 아래와 같은 구조를 기반으로 작업을 진행하였습니다.**

![package](https://user-images.githubusercontent.com/25237661/45264404-f129f700-b476-11e8-8e48-608b52494258.PNG)

<br>

#### GAVo.java

```java
/**
 * GoogleAnalytics VO
 * @author JH
 * @since 2018.08.21
 */
public class GAVo{

  private String start;
  private String end;
  private int type;

  public String getStart() {
    return start;
  }
  public void setStart(String start) {
    this.start = start;
  }
  public String getEnd() {
    return end;
  }
  public void setEnd(String end) {
    this.end = end;
  }

  /**
   * set HelperEnumCode <p>
   * 1 : 방문자 <p>
   * 2 : 총 페이지뷰 <p>
   * 3 : 브라우저별 <p>
   * 4 : 모니터 해상도별 <p>
   * 5 : 이전 접속 사이트 <p>
   * 6 : 검색엔진 단어별 <p>
   * 7 : 가장 많이본 사이트 <p>
   * @param type
   */
  public void setType(int type) {
    this.type = type;
  }

  /**
   * @return type에 맞는 GAHelper.Type 반환
   */
  public GAHelper getHelper(){
    switch(this.type){
      case 1 : return GAHelper.VISIT_USER;
      case 2 : return GAHelper.TOTAL_VIEW_COUNT;
      case 3 : return GAHelper.BROWSER;
      case 4 : return GAHelper.RESOLUTION;
      case 5 : return GAHelper.REFFER;
      case 6 : return GAHelper.KEYWORD;
      case 7 : return GAHelper.PAGE_PATH;
      default : return null;
    }
  }
```

#### GARequester.java

```java
/**
 * GA Reporting API Request Creator
 * @author JH
 * @since 2018.08.21
 */
public class GARequester {
  private List<ReportRequest> requests;
  private File file;

  private GARequester(List<ReportRequest> requests, File file){
    this.requests = requests;
    this.file = file;
  }

  public GetReportsResponse request() throws GeneralSecurityException, IOException{
    AnalyticsReporting service = GAHelper.initializeAnalyticsReporting(file);
    GetReportsRequest getReport = new GetReportsRequest().setReportRequests(requests);
    GetReportsResponse response = service.reports().batchGet(getReport).execute();
    return response;
  }

  /**
   * GARequester Builder
   * @author JH
   * @since 2018.08.21
   */
  public static class Builder{

    String startDate;
    String endDate;
    GAHelper helper;
    File file;

    public Builder setStartDate(String startDate){
      this.startDate = startDate;
      return this;
    }
    public Builder setEndDate(String endDate){
      this.endDate = endDate;
      return this;
    }

    public Builder setHelper(GAHelper helper){
      this.helper = helper;
      return this;
    }

    public Builder setFile(File file){
      this.file = file;
      return this;
    }

    public GARequester build(){
      ReportRequest request = new ReportRequest()
          .setViewId(GAHelper.VIEW_ID)
          .setDateRanges(this.helper.createDateRange(startDate, endDate))
          .setMetrics(this.helper.createMetric())
          .setDimensions(this.helper.createDimension())
          .setIncludeEmptyRows(true);
      return new GARequester(Arrays.asList(request), file);
    }
  }
}
```

#### GAHelper.java

```java
/**
 * GA Helper <p>
 * 조회타입에 맞게 파라미터 미리 정의 <p>
 * {@link https://developers.google.com/analytics/devguides/reporting/core/v4}
 *
 * @author JH
 * @since 2018.08.21
 */
public enum GAHelper {

  VISIT_USER("ga:sessions", "ga:date", null, "user"),
  TOTAL_VIEW_COUNT("ga:pageviews", "ga:date", null, "totalViewCount"),
  BROWSER("ga:sessions", "ga:browser", null, "browser"),
  RESOLUTION("ga:sessions","ga:screenResolution", null, "resolution"),
  REFFER("ga:sessions", "ga:keyword", null, "reffer"),
  KEYWORD("ga:sessions", "ga:fullReferrer", null, "keyword"),
  PAGE_PATH("ga:users", "ga:pagePath", null, "pagePath");

  private String metrics;
  private String dimensions;
  private String filter;
  private String alias;

  static final String APPLICATION_NAME = "DEMO";
  static final GsonFactory GSON_FACTORY = GsonFactory.getDefaultInstance();
  static final String VIEW_ID = "DEMO";

  GAHelper(String metrics, String dimensions, String filter, String alias) {
    this.metrics = metrics;
    this.dimensions = dimensions;
    this.filter = filter;
    this.alias = alias;
  }

  /**
   * QueryParameter DateRange
   * @param start
   * @param end
   * @return dateRangeList
   */
  public List<DateRange> createDateRange(String start, String end) {
    //...
    return list;
  }

  /**
   * QueryParameter Metric 초기화
   * @return metricList
   */
  public List<Metric> createMetric() {
    //...
    return list;
  }

  /**
   * QueryParameter Dimension 초기화
   * @return dimenssionList
   */
  public List<Dimension> createDimension() {
    //...
    return list;
  }

  /**
   * GA Reporting API 사용권한 검증
   * @return AnalyticsReporting
   * @throws GeneralSecurityException
   * @throws IOException
   */
  static AnalyticsReporting initializeAnalyticsReporting(File file) throws GeneralSecurityException, IOException {

    HttpTransport httpTransport = GoogleNetHttpTransport.newTrustedTransport();
    GoogleCredential credential = GoogleCredential.fromStream(new FileInputStream(file))
    		.createScoped(AnalyticsReportingScopes.all());

    // Construct the Analytics Reporting service object.
    return new AnalyticsReporting.Builder(httpTransport, GSON_FACTORY, credential)
    		.setApplicationName(APPLICATION_NAME).build();
  }

}
```

**이번글은 여기서 마무리 하겠습니다 ~**
