---
layout: single
tilte: "배열과 컬렉션 그리고 인덱서"
categories: C# study
tag: [C#, study]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 10.0 이 장의 핵심 개념

명함에 비유. 한 사람이 다른 누군가에게 명함을 돌린다. 그 사람도 다른 누군가에게 명함을 받는다. 이렇게 명함들이 돌고 돌다 보면 명함의 종류가 엄청 많아진다. 이때 필요한것이 명함집이다. 
우리가 다루는 데이터가 명함이고, 배열이나 컬렉션은 명합집이다.

1. 배열이 무엇인지 이해
2. 배열을 선언하고 초기화하는 방법을 익히기
3. System.Array 클래스를 이용해 배열을 다루는 사용 방법을 익힙니다.
4. 여러 종류의 배열을 사용하는 방법을 익힙니다.
5. 다양한 컬렉션 클래스를 이해합니다.
6. 인덱서를 선언하고 사용하는 방법을 익힙니다.

# 10.1 All for one, one for all

같은 성격을 띤 다수의 데이터를 한번에 처리하기 좋은 수단이 배열이다. 배열선언 방법

> 데이터형식[ ] 배열이름 = new 데이터 형식 [ 용량 ];

예를 들어 용량이 5인 int형 배열은

> int[] scores = new int [5];

일정 크기의 배열을 선언하는 방법은 위와 같고, 각 배열의 요소에 접근하여 할당하는 방법은 다음과 같다.
```c#
int[] scores = new int [5];
scores[0] = 80;
scores[1] = 74;
scores[2] = 81;
scores[3] = 90;
scores[4] = 34;
```

배열의 활용 출력
>foreach( int score in socres ) Console.WriteLine( score );
배열의 활용 계산
```c#
int sum = 0;
foreach( int score in scores )  sum += score;
int average = sum/scores.Length;//Length는 배열 객체의 길이를 알려주는 프로퍼티
```
배열의 첫번째 인덱스는 0입니다. 5개를 저장하는 배열의 마지막 인덱스는 4일 것 이다.

항상 마지막 인덱스는 [배열이름.Length-1]로 접근한다.

C# 8.0에서는 이런 불편을 없앤 **System.Index** 형식과 **^** 연산자가 생겼다. 

^ 연산자는 **컬렉션**의 마지막부터 역순으로 인덱스를 지정하는 기능을 가진다.

^ 연산자의 연산 결과는 System.Index 형식의 인스턴스로 나타낸다.
```C#
System.Index last = ^1;
scores[last] = 34; // scores[scores.Length - 1] = 34; 와 동일
```
일일히 인덱스형식 인스턴스를 인덱스형식 변수에 할당하여 사용하지 않아도 알아서 인덱스인스턴스를 바로 인덱스로 사용 가능하다.
```c#
scores[^1] = 34; // scores[scores.Length - 1] = 34; 와 동일
```

# 10.2 배열을 초기화하는 방법 세 가지

첫 번째, 데이터형식[ ] 배열이름 = new 데이터 형식 [ 용량 ] { //배열에 들어갈 요소들 콤마로 구분하여 작성};

위 처럼 배열 객체를 초기화하는 {} 블록을 컬렉션 초기자 Collection Initializer라고 부른다.

<br/>
두 번째

