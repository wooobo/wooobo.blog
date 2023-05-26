---
emoji: 🔮
title: method naming 에서 of 와 from 의 차이는 뭘까?
date: '2021-11-09 00:00:00'
author: wooobo
tags: DI
categories: 프로그래밍
---


# of 와 from 의미가 뭘까?

문득 코딩을 하다 궁금해졌습니다.  
메소드의 이름으로 사용되는 of 와 from 의 의미에 대해서 알고 싶어졌습니다.

구글 검색결과  
[oracle - java documentation](https://docs.oracle.com/javase/tutorial/datetime/overview/naming.html)  
[stackoverflow](https://stackoverflow.com/questions/67680011/naming-convention-what-is-the-difference-between-from-vs-of-methods)

등등 에서 정보를 찾을 수 있었습니다.

오라글 문서에서 정의하는 메소드 네이밍 컨벤션은 아래와 같습니다.

|Prefix|Method Type|Use|
|------|---|---|
|of|static factory|팩토리가 주로 입력 매개변수의 유효성을 검사하고 변환하지 않는 인스턴스를 만듭니다.|
|from|static factory|입력 매개변수를 대상 클래스의 인스턴스로 변환합니다. 이 경우 입력에서 정보가 손실될 수 있습니다.|
|parse|static factory|입력 문자열을 구문 분석하여 대상 클래스의 인스턴스를 생성합니다.|
|format|instance|지정된 포맷터를 사용하여 임시 개체의 값을 형식화하여 문자열을 생성합니다.|
|get|instance|대상 객체 상태의 일부를 반환합니다.|
|is|instance|대상 개체의 상태를 쿼리합니다.|
|with|instance|한 요소가 변경된 대상 개체의 복사본을 반환합니다. 이것은 JavaBean 의 set 메소드와 동일한 불변입니다.|
|plus|instance|시간이 추가된 대상 개체의 복사본을 반환합니다.|
|minus|instance|시간을 뺀 대상 개체의 복사본을 반환합니다.|
|to|instance|이 개체를 다른 유형으로 변환합니다.|
|at|instance|이 개체를 다른 개체와 결합합니다.|

of, from 뿐만 아니라 다른 네이밍 컨벤션도 궁금했는데 갓 오라클이네요 ^^
(구글 번역기로 나온 번역이라.. 정확한 정보는 위 링크 참고해주세요.)


## 이해한걸 바탕으로 예제로 알아보기

<summary>예시로 사용될 클래스</summary>

  ```java 
  // 예시로 사용될 클래스 입니다.
  class StringArray {
      private final List<String> stringArray;
  
      private StringArray(List<String> array) {
          this.stringArray = array;
      }
  
      public static StringArray of(int... ints) {
          return new StringArray(Arrays.stream(ints).mapToObj(String::valueOf).collect(Collectors.toList()));
      }
  
      public static StringArray from(String str, String delimiter) {
          return new StringArray(List.of(str.split(delimiter)));
      }
  
      @Override
      public String toString() {
          return stringArray.toString();
      }
  }
  ``` 

예시로 좀 더 탐구해봤습니다. 

```java
@Test
void method_naming_conventions() {
    // of (static factory)
    // str1.{index}Of({arg}) -> str1 의 {arg} 값의 index 를 줘
    String str1 = "123456";
    System.out.println(str1.indexOf("1")); // 0
    System.out.println(str1.indexOf("2")); // 1
    
    // {StringArray}.of({arg}); -> {arg} 를 검증해서 {StringArray} 인스턴스를 줘
    StringArray of = StringArray.of(1, 2, 3, 4, 5, 6);
    System.out.println(of.toString()); // [1, 2, 3, 4, 5, 6]

    // from (static factory)
    // {StringArray}.from({arg}) -> {arg} 를 변경해서 {StringArray} 인스턴스 를 줘
    StringArray from = StringArray.from("1,2,3,4,5,6", ":");
    System.out.println(from.toString()); // [1,2,3,4,5,6]

    // parse (static factory)
    // {Integer}.parseInt({arg}) -> {arg} 를 Int 로 parse 해줘
    int parseInt = Integer.parseInt("1");
    System.out.println(parseInt); // 1


    // format (instance)
    // {String}.format({arg}) -> {arg}를 포맷팅해서 {String} 으로 줘
    String stringFormat = String.format("%s 포맷팅합니다.", "문자열");
    System.out.println(stringFormat); // "문자열 포맷팅합니다."

    // get (instance)
    // {instance}.get({arg}) -> {instance} 의 {arg} 번째 데이터를 줘
    List<String> stringList = new ArrayList<>();
    stringList.add("첫번째 String");
    stringList.add("두번째 String");
    System.out.println(stringList.get(0)); // "안녕 String"


    // is (instance)
    // {instance}.is{Status}() -> {instance} 가 {Status} 이냐? 맞으면 true , 틀리면 false
    String empty = "";
    System.out.println(empty.isEmpty()); // true
    
    // to (instance)
    // {number}.to{String} -> {number} 를 {String} 으로 변환해줘
    Integer number = 1234;
    System.out.println(number.toString()); // "1234"


    // at (instance)
    // str.{char}At(arg) -> str 의 {arg} index 의 값을 {char} 로 반환해줘
    String str = "adasd";
    System.out.println(str.charAt(1));
}

```


