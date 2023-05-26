---
emoji: 🔮
title: 정적 메소드 팩토리 사용
date: '2021-11-10 00:00:00'
author: wooobo
tags: DI
categories: 프로그래밍
---


<br> 

# 정적 메소드 팩토리

정적 메소드 팩토리를 사용하면, 의미있는 이름의 생성자를 제공 할 수 있는 장점이 있습니다.  

> 참조 글     
[baeldung.com/java-constructors-vs-static-factory-methods](https://www.baeldung.com/java-constructors-vs-static-factory-methods)
> 
>1.생성자에는 의미 있는 이름 이 없으므로 항상 언어에서 부과하는 표준 명명 규칙으로 제한됩니다. 정적 팩토리 메소드는 의미 있는 이름을 가질 수 있으므로 그들이 하는 일을 명시적으로 전달합니다.  
2.정적 팩토리 메서드는 메서드, 하위 유형 및 기본 형식을 구현하는 동일한 유형을 반환할 수 있으므로 보다 유연한 반환 유형 범위를 제공합니다.  
3.정적 팩토리 메서드는 완전히 초기화된 인스턴스를 미리 구성하는 데 필요한 모든 논리를 캡슐화 할 수 있으므로 생성자에서 이 추가 논리를 이동하는 데 사용할 수 있습니다. 이것은 생성자가 필드를 초기화하는 것 이외의 추가 작업 을 수행하는 것을 방지 합니다.  
4.정적 팩토리 메소드는 제어된 인스턴스 메소드 일 수 있으며 싱글톤 패턴 이 이 기능의 가장 눈에 띄는 예입니다.  

## 정적 메소드 팩토리 사용하지 않은경우

`Student` 클래스로 구현해보겠습니다.

```java
class Student {
    private final String name;
    private final String className;
    private final String grade;

    public Student(String name, String className, String grade) {
        this.name = name;
        this.className = className;
        this.grade = grade;
    }

    public Student(String name, String className) {
        this.name = name;
        this.className = className;
        this.grade = "1학년";
    }
}
```
`Student` 클래스는 생성자가 두개 있습니다.  
1. public Student(String name, String className, String grade) `이름`, `반`, `학년` 인스턴스 변수 모두 받습니다.
2. public Student(String name, String className) `이름`, `반` 두개만 받고 `grade` 는 "1학년" 으로 고정으로 생성됩니다.

이때, `new Student(arg... )` 는 인자 값만 달라지고 생성자 이름은 클래스 이름으로 모두 동일합니다. 다양한 생성 인자 케이스가 많아진다면 의미를 
알기 힘들어지는 단점이 있습니다.  

정적 메소드 팩토리로 다시 살펴 보겠습니다.

## 정적 메소드 팩토리로 리팩토링

```java

class Student {
    public static final String GRADE = "1학년";

    private final String name;
    private final String className;
    private final String grade;
  
    private Student(String name, String className, String grade) {
        this.name = name;
        this.className = className;
        this.grade = grade;
    }

    public static Student of(String name, String className, String grade) {
        return new Student(name, className, grade);
    }

    public static Student createWithDefaultGrade(String name, String className) {
        return new Student(name, className, GRADE);
    }
}
```

`정적 메소드 팩토리` 를 사용하지 않았을때 보다 의미있는 생성자이름 이라는 것을 확인 할 수 있습니다.  
`public Student(String name, String className)` 보다  
`public static Student2 createWithDefaultGrade(String name, String className) {` 를 사용하면 "아! `Student` 인스턴스를 만드는데 `grade`는 기본값이 있구나" 
라고 의미를 해석 할 수 있게됩니다.  


### 참조 글
[baeldung.com/java-constructors-vs-static-factory-methods](https://www.baeldung.com/java-constructors-vs-static-factory-methods)