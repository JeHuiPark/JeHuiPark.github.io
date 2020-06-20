---
layout: posts
title:  MS Excel csv 한글깨짐
date:   2020-06-20 17:36:09 +0900
comments: true
categories: note
---

* TOC
{:toc}

## 증상  
- 특정 편집기로는 CSV 파일을 열어볼 때는 글자 깨짐 현상 없음  
![image](https://user-images.githubusercontent.com/25237661/85197661-72d1d880-b31d-11ea-8c75-716a66090d9b.png){:width="300px"}

- MS Excel 편집기로 CSV 파일을 열어볼 때는 글자 깨짐 현상 발생
![image](https://user-images.githubusercontent.com/25237661/85197684-8f6e1080-b31d-11ea-8a8a-4600bb13ef68.png){:width="300px"}

## 원인
MS Excel 에 파일의 인코딩을 알리는 `BOM` 누락 (매직넘버라고 부른다.)

아래는 [위키피디아](https://ko.wikipedia.org/wiki/%EB%B0%94%EC%9D%B4%ED%8A%B8_%EC%88%9C%EC%84%9C_%ED%91%9C%EC%8B%9D)의 문서의 내용을 발췌
> 마이크로소프트의 컴파일러와 인터프리터 그리고 노트패드와 같은 마이크로소프트 윈도우의 많은 소프트웨어들은 BOM을 휴리스틱을 이용하지 않고 필수적인 매직 넘버처럼 처리합니다. 이 도구들은 UTF-8로 텍스트를 저장할 때 BOM을 추가하며, BOM이 나타나지 않거나 파일에 ASCII만 포함되어 있지 않다면 UTF-8을 해석할 수 없습니다. 또한 구글 독스는 문서를 다운로드를 위한 플레인 텍스트로 변환할 때 BOM을 추가합니다.

## 해결
`UTF8` `BOM`[-17, -69, -65] 추가  

## 예제코드
[예제코드 저장소](https://github.com/JeHuiPark/blog-sample/tree/master/ms-excel-utf8-sample)
```java
class Example {

  public static void main(String[] args) throws IOException {
    var noBomCsv = new File("ms-excel-no-bom-example.csv");
    var bomCsv = new File("ms-excel-bom-example.csv");

    try (var noBomCsvWriter = new FileOutputStream(noBomCsv);
         var bomCsvWriter = new FileOutputStream(bomCsv)) {

      var csvString = "이름,나이,성별\n"
          + "박제희,29,남\n"
          + "홍길동,30,남";

      noBomCsvWriter.write(csvString.getBytes());

      apply_MS_UTF8_Encoding_BOM(bomCsvWriter);
      bomCsvWriter.write(csvString.getBytes());
    }
  }

  private static void apply_MS_UTF8_Encoding_BOM(FileOutputStream bomCsvWriter) throws IOException {
    bomCsvWriter.write(new byte[]{-17, -69, -65});
  }
}
```

### 매직넘버 적용결과
![image](https://user-images.githubusercontent.com/25237661/85197943-77978c00-b31f-11ea-820e-248f425463b7.png){:width="300px"}

