---
layout: single
tilte: "프로퍼티"
categories: C# study
tag: [C#, study]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 9.0 이 장의 핵심 개념

1. 프로퍼티가 무엇인지 이해
2. 매소드, 필드, 프로퍼티 간의 차이 이해
3. 프로퍼티를 이용하여 객체를 초기화하는 방법을 익힌다.
4. 무명 형식을 이해
5. 인터페이스와 추상 클래스에서의 프로퍼티 선언 방법을 익힌다.

# 9.1 public 필드의 유혹

클래스를 작성하다 보면 속성인 필드들을 public으로 선언하고 제약없이 마구잡이로 쓰고 싶다는 충동이 들 수 있다. 이것은 언제든지 수정시에 데이터가 원치 않는 오염당할 위험을 잠재적으로 냅두는 행동이지만, 은닉성을 지켜가며 일일히 짜기엔 편의성이 너무 떨어지기 때문이다.

그 동안 알고 있던 지식으로는 은닉성을 유지하기위해 private으로 필드를 선언하고 GetXXX 메소드로 외부에서 호출하고, SetXXX 메소드로 외부에서 할당하는 방식을 이용했다.

```C#
class TestClass
{
    private int field;
    public int GetField(){ return field; };
    public int SetField( int value ){ field = value; };
}
```

모든 필드마다 이런식으로 은닉성을 지키고자하면 편의성이 삭제당하기에 이런 방식으로 짤려고하면 선뜻 손이 안가는 것. 게다가 또 클래스명쓰고 메소드명 일일히 외워서 쓰는것도 한계가 있다.. 하지만 C#에선 두 마리 토끼를 잡기 위한 녀석이 존재하는데 그것이 프로퍼티이다. 프로퍼티를 무조건 알아보고 싶어진다.

#9.2 메소드보다 프로퍼티

```c#
class 클래스명
{
    데이터형식 필드명;
    접근한정자 데이터형식 프로퍼티명
    {
        get
        {
            return 필드명;
        }
        set
        {
            필드명 = value;
        }
    }
}
```
이것이 프로퍼티 선언 규칙이다. 필드의 은닉성을 비교적 편하게 지킬수 있을 것 처럼 보인다.

프로퍼티 문법에서 get{} 과 set{} 을 일컬어 접근자라고 한다. get{} 접근자는 필드의 값을 가져오고, set{} 접근자는 필드에 값을 저장한다. set{} 접근자 안의 value는 키워드이며, 암묵적으로 매개변수처럼 간주한다. 

두 개의 접근자를 가진 프로퍼티 사용법은 각각의 메소드명을 외워서 쳐야되는 것보다 훨씬 간단하다. 그냥 프로퍼티명만 알고 있으면 알아서 get set 을 사용한다.
```c#
class TestClass
{
    private int myField;
    public int MyField
    {
        get { return myField; }
        set { myField = value; }
    }
}
class MainApp
{
    static void Main()
    {
        TestClass test = new TestClass();
        test.MyField = 3; 
//프로퍼티명에 할당하니 프로퍼티의 set{}접근자를 이용하고
        Console.WriteLine( test.MyField );
//변수처럼 프로퍼티명을 사용하니 get{}접근자를 이용하더라.
    }
}
```
만일 외부에서 변경되는 것은 원치 않는다면, set{} 접근자부분만 작성하지 않아도 된다. 외부에서 할당하려고 하면 에러를 송출할 것이다.

>쓰기전용 set 프로퍼티를 만들 생각이라면 주의해야한다. 은닉성을 지키는 필드에 값을 할당만 하는 프로퍼티라면 왜 그렇게 하는지 의도, 용도, 동작 결과 확인방법 등을 프로퍼티 작성자가 알려줄 의무가 있다. 이 의무를 다하지 않는다면, 코드 관리가 어려워 질 수 있다.


# 9.3 자동 구현 프로퍼티

앞서 작성한 프로퍼티는 마치 메소드를 대체하는 획기적인 편의모델 같다는 느낌을 준다. 여기서 더 훨씬 편하게 해주는 방법이 있다. 프로퍼티만 선언하면 알아서 프로퍼티가 필드의 역할도 한다면? 그리고 어차피 get{return} set{value} 단순한 로직을 생략 할 수 있다면?

```c#
public class NameCard
{
    public string Name { get; set; }
    public string PhonNumber{ get; set; }
//변수처럼 선언하되 앞글자는 대문자이고 
//{get;set;}만 붙이면 알아서 프로퍼티라 인식한다.
}
```
C#7.0 부터는 프로퍼티 필드 초기화도 추가되었다.
```c#
public class NameCard
{
    public string Name { get; set; } = "none"
    public string PhonNumber{ get; set; } = "000-0000-0000"
}
```
## 9.3.1 자동 구현 프로퍼티 뒤에서 일어나는 일

일반적인 필드 선언 프로퍼티 선언 후에 일어나는 일과 자동 구현 프로퍼티 선언 후에 일어나는 일을 비교하기 위해서 .NET 디어셈블리 도구인 ildasm.exe 를 열어서 직접 확인해보자.

둘다 결국 필드가 필요하다. 전자는 직접 입력한 필드를 참조하고, 후자는 컴파일러가 자동으로 생성한 필드를 이용한다.


# 9.4 프로퍼티와 생성자

8장에서 클래스 생성자를 다룰때 매개변수를 이용해서 필드를 초기화하는 방법을 다루었다. 프로퍼티가 선언되어 있다면, 훨씬 간단한 방법으로 초기화가 가능하다.

```c#
클래스이름 인스턴스 = new 클래스이름()
{
    프로퍼티1 = 값, 프로퍼티2 = 값,
    프로퍼티3 = 값  //세미콜론이 아닌 콤마임에 주의
}
```
매개변수로 초기화를 하려면 생성자 선언 단계에서부터 어떤 매개변수로 어떤 필드를 초기화를 할지를 미리 구현할 고민을 해야 한다.
하지만 프로퍼티의 set{}접근자와 필드가 연결되어있다면(자동구현 프로퍼티는 자동으로 연결되어 있을거니까) 그냥 <프로퍼티=값> 목록으로 초기화가 된다.
모든 프로퍼티를 다 쓸 필요는 없다. 원하는 것만 순서 상관없이 초기화하면 된다.

# 9.5 초기화 전용(Init-Only) 자동 구현 프로퍼티

데이터의 오염을 막기 위해 읽기 전용 필드를 만드는 일이 종종 있습니다. 접근한정자, readonly 등등이 그 예이다.

프로퍼티를 읽기 전용으로 만든다면 어떨까?
get접근자로 필드를 연결하고, set접근자는 적지 않는 방법이 있을 것이다.

이렇게 만들어진 프로퍼티의 초기화는 어떻게 해야되지?
set접근자가 존재한다면 9.4에서 알아본 것 처럼 <프로퍼티 = 값> 목록으로 초기화를 할 것인데, set 접근자가 없다면 class생성자 오버로딩을 통하여 매개변수로 필드를 초기화하는 방법밖에 없다.

하지만 c# 9.0에 와서 읽기 전용 프로퍼티의 초기화가 상당히 간편해졌다. set 접근자를 대체할 init 접근자를 제공하기 때문이다.
```c#
public class Transaction
{
    public string From { get; init; }
    public string To { get; init; }
    public int Amount { get; init; }
}
```
이렇게 되면 set을 사용했을때처럼 <프로퍼티 = 값> 목록으로 클래스 디폴트 생성자 뒤에 붙어 쉽게 초기화가 가능해진다. set을 사용했을때와 달리 초기화만 가능하고, 쓰기는 불가능하다.

#9.6 레코드 형식으로 만드는 불변 객체

불변 객체란 안의 내용물을 변경 할 수 없는 객체. 불변 객체를 이용해 데이터 **복사**와 **비교**연산을 주로 한다. 불변 객체를 복사하고 약간 수정한 새로운 객체를 생성한다든지, 상태확인을 위해 불변객체와 비교연산을 한다든지가 그 예이다.

레코드 형식이란 C#9.0에 새로 도입된 형식으로 복사 비교 연산을 효율적이고 편리하게 수행할 수 있게 만든 형식이다.

불변 형식은 기존에 어떻게 만들었을까?
참조형식의 대표주자 class와 값형식의 대표주자 struct를 예로 들자면 둘다 모든 필드를 readonly로 강제하면 바꿀 수 없는 class와 struct가 된다.

자 이제 얘네들에게 복사와 비교 상황극을 대입해보자. 

값 형식 객체는 다른 객체에 할당을 할때 늘 깊은 복사를 하게 됩니다. 필드 몇 개일때는 상관없지만, 무수히 커져버린 값 형식을 복사할때 그 속도가 굉장히 비효율적일 것입니다. 반면 참조형식은 깔끔하죠. 할당할때 어디까지 읽을것인지만 형식만 깔끔하게 정해놓으면, 복사하는 내용은 메모리 주소 한줄일테니까. 하지만 단점은 참조형식의 깊은복사를 구현하기 위해선 각 객체에 맞는 코드를 새로짜야된다는 것.

비교를 할때도 비슷한 상황. 값 형식 객체와 다른 객체를 비교할땐 필드 하나 하나 메모리상에 일치하는지 다 비교해야지 비로소 비교완료가 된다. 참조형식은? 빠르지만, 힙영역에서 어떻게 비교할지 코드를 다 짜야 된다는 것.

레코드 형식은 이런 두 녀석의 장점을 섞었다. 참조만큼 빠르고, 값만큼 간편하게 복사 비교를 한다.
## 9.6.1 레코드 선언하기

레코드는 class와 선언방법이 유사하다. 안에 들어갈 필드는 "초기화 전용 자동구현 프로퍼티"로 채워 넣는다. 이러면 참조형식이면서 값형식만큼 편리하게 사용가능 하면서 불변객체로 사용 가능해진다. 일반적인 "자동구현 프로퍼티"로 채워 넣어도 문법적으로 이상은 없지만, 불변성은 잃게 된다. 사용하게 된다면 레코드를 더 학습하고나서 써야할 일이 있을거 같다. 지금은 불변객체 방법을 위해 공부중이다.

```c#
record RTransaction
{
    public string From { get; init; }
    public string To { get; init; }
    public int Amount { get; init; }
}
```

## 9.6.2 with를 이용한 레코드 복사

C# 컴파일러는 레코드 형식을 위한 복사 생성자를 자동으로 작성합니다. 참조 형식임에도 불구하고 어떤 규칙만 따른다면 자동으로 깊은 복사가 편리하게 이루어진단는 말이다. 복사 생성자는 protected로 선언되어 있어 명시적 호출은 불가함.
```c#
RTransaction tr1 = new RTransaction{From="Alice", To="Bob", Amount = "100"};
RTransaction tr2 = with tr1{To = "Charlie"}; 
//with tr1에서 깊은 복사가 이루어지고
//{}부분에서 일부 프로퍼티필드값을 초기화
```
## 9.6.3 레코드 객체 비교하기

클래스와 레코드의 비교연산차이점은 Equals() 메소드 구현에 있습니다. 레코드 형식에서의 Equals()메소드는 자동으로 구현해주므로 그냥 호출만 하면 되지만, 클래스에서는 직접 구현해놓아야 합니다.
```c#
class CTransaction
{
    public string From { get; init; }
    public string To { get; init; }
    public int Amount { get; init; }

    public override bool Equals(Object obj)
    {
        CTransaction target = (CTransaction)obj;
        if(this.From == target.From && this.To == target.To && this.Amount == target.Amount) return true;
        else return false;
    }
}
```
이처럼 클래스 선언시에 클래스 필드와 맞게 Equals() 메소드를 오버라이딩 해줘야 합니다. 오버라이딩 하는 걸로 보아 원래 메소드가 있는 것이 분명하다. 원래 메소드는 딱 인스턴스가 가진 값인 주소값을 비교하는 메소드이다.
```c#
using System;
namespace Recordcompare
{
    class CTransaction
    {
        public string From { get; init; }
        public string To { get; init; }
        public int Amount { get; init; }

        public override string ToString()
        {
            return $"{From, -10} -> {To, -10} : ${Amount}";
        }
    }

    record RTransaction
    {
        public string From { get; init; }
        public string To { get; init; }
        public int Amount { get; init; }

        public override string ToString()
        {
            return $"{From, -10} -> {To, -10} : ${Amount}";
        }       
    }

    class MainApp
    {
        static void Main(string[] args)
        {
            CTransaction trA = new CTransaction{From="Alice", To="Bob", Amount=100};
            CTrnasaction trB = with trA;
            Console.WriteLine(trA);
            Console.WriteLine(trB);
            Console.WriteLine($"trA와 trB의 주소값은 다르니까 : {trA.Equals(trB)}");

            RTransaction tr1 = new RTransaction{From="Alice", To="Bob", Amount=100};
            RTransaction tr2 = with tr1;
            Console.WriteLine(tr1);
            Console.WriteLine(tr2);
            Console.WriteLine($"trA와 trB의 안에 내용물 비교 : {tr1.Equals(tr2)}");
        }
    }
}
```
# 9.7 무명 형식
위의 레코드내용에서 빠져나와 다시 프로퍼티에대해 알아본다. 무명형식이란 형식을 지정하지 않은 객체이다. 앞서 배웠던 var 변수가 떠오르면 다행이다. 프로퍼티를 이용한 무명형식에대해 알아본다.

무명형식은 선언과 동시에 인스턴스가 할당된다. 즉 class나 record처럼 미리 구현해놓고 사용하고자하는 곳에서 인스턴스를 생성하는 것이 아닌, 구현부도 없이 사용하고자하는 자리에서 그 즉시 구현과 인스턴스 생성을 동시에 한다. 단순하게 만들고 다시는 사용하지 않을때 무명형식이 요긴하게 쓰인다. 특히 나중에 배우는 LINQ 라는 개념에서 요긴하게 쓰인다고 한다. 나도 지금은 이런걸 왜 쓰지? 싶다. 어차피 튜플 하나 만들어 쓰면 그만 아닌가 싶은데 나중에 다시 참고하자.

무명 형식 선언 방법과 사용방법은 다음과 같다.
```c#
var myInstance = new {Name="Noel", Age = "1", Location="Korea"};
//var 객체명 = new {프로퍼티=값,...} 프로퍼티의 자료형은 컴파일러가 알아서 처리해준다.
Console.WriteLine(myInstance.Name, myInstance.Age);
//클래스의 프로퍼티 get접근자 사용하듯이 사용한다.
```
# 9.8 인터페이스의 프로퍼티

인터페이스는 8장에서 배웠듯이 그냥 설계도 규칙 규정입니다. 메소드 뿐만 아니라 프로퍼티, 인덱서들을 규정할 수 있습니다. 당연히 인터페이스를 상속하는 클래스는 인터페이스에 적혀있는 프로퍼티를 전부 구현해야하는 의무를 가집니다. 인터페이스에 프로퍼티를 작성하는 규칙은 자동 구현 프로퍼티와 동일하지만, 메모리의 어떠한 부분에도 필드를 차지하는 일은 없습니다. 그냥 겉모습만 자동구현 프로퍼티입니다.
```c#
interface 인터페이스명
{
    public 자료형 프로퍼티명 { get; set; }
}
```
# 9.9 추상 클래스의 프로퍼티

추상클래스는 인터페이스의 **강제성**을 가지면서도 클래스처럼 구현과 메모리를 이용하는 클래스입니다. 앞서 추상클래스의 추상메소드가 상속하는 클래스들에게 강제적으로 구현부를 작성하도록 강요한다고 배웠는데 프로퍼티도 마찬가지고 추상프로퍼티를 작성할 수 있습니다. 인터페이스처럼 자동구현 프로퍼티 형태로 작성하면 될까? 그렇지 않다. 이녀석은 인터페이스와 달리 클래스의 한 종류이기때문에 한가지 키워드를 더 붙이면된다. 바로 맨 앞에 abstract 키워드를 붙이는 방법이다.
<div class = "notice--success">
<h4>공지사항입니다.</h4>
<ul>
    <li> 공지사항 순서 1 </li>
    <li> 공지사항 순서 2 </li>
    <li> 공지사항 순서 3 </li>
</ul>
</div>

**[공지사항]** [지킬블로그신규 업데이트 안내 드립니다.](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/){: .notice--danger}

<div class = "notice--success">
<h4>공지사항입니다.</h4>
<ul>
    <li> 공지사항 순서 1 </li>
    <li> 공지사항 순서 2 </li>
    <li> 공지사항 순서 3 </li>
</ul>
</div>

[Google](https://google.com){: btn.btn--danger}

{% include video id="oqrn64PaJjE" provider="youtube" %}

앞으로 열심히


![ice_640](../assets/images/ice_640.jpg)