---
emoji: 🔮
title: 빌더 패턴
date: '2021-11-12 00:00:00'
author: wooobo
tags: 개발
categories: 프로그래밍
---


<br>

> 참조블로그 : https://asfirstalways.tistory.com/350


# 빌더 패턴



한번에 인스턴스를 생성하고 싶은데 생성자 인자가 너무 많을때 혹은 기본 생성자로 인스턴스를 만들고 setter 로 점진적으로 값을 주입시킬때 이럴때 빌더 패턴을 사용하면 장점이 있습니다.

## 빌더 패턴을 사용하지 않은 예

```java

class User {
    String name;
    String phoneNumber;
    String address;
    int zipCode;
    String age;
    String email;

    public User(String name, String phoneNumber, String address, int zipCode, int age, String email) {
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.address = address;
        this.zipCode = zipCode;
        this.age = age;
        this.email = email;
    }
}

```

- 생성자

```java 

@Test
void test() {
    new User("이름", "010-1234-5678", "집주소", 12345, 100, "email@email.com");
}
 
```

생성 인자가 너무 많다보니, 순차적으로 뭐가 들어가는지도 헷갈리기 시작합니다. 혹여나 실수로 위치를 다르게 넣을 수 도 있을것 같습니다.

## 빌더 패턴 적용 예

```java 


class User {
    String name;
    String phoneNumber;
    String address;
    int zipCode;
    int age;
    String email;

    private User(String name, String phoneNumber, String address, int zipCode, int age, String email) {
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.address = address;
        this.zipCode = zipCode;
        this.age = age;
        this.email = email;
    }

    public static class Builder {
        String name;
        int age;
        String phoneNumber;
        String email;
        String address;
        int zipCode;

        public Builder() {
        }

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder contact(String phoneNumber, String email) {
            this.phoneNumber = phoneNumber;
            this.email = email;
            return this;
        }

        public Builder house(String address, int zipCode) {
            this.address = address;
            this.zipCode = zipCode;
            return this;
        }

        public User build() {
            return new User(name, phoneNumber, address, zipCode, age, email);
        }
    }
}
```

빌더 패턴을 사용하면 많은 생성자 인자를 명시적으로 맵핑 할 수 있는 장점이 있고, 기본 생성자를 만들고 `setter`로 순차적으로 맵핑하는 것보다
한번에 인스턴스를 생성 할 수있습니다.

```java 
new User.Builder()
    .name("이름")
    .age(100)
    .contact("010-1234", "email@email.com")
    .house("집주소", 12345)
    .build();
```

한번에 그리고 명시적으로 파라미터를 맵핑 할 수 있는 장점이 있는 빌더 패턴을 알아보았습니다. 
