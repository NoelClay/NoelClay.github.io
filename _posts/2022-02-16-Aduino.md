---
layout: single
tilte: "4.8 옵션 UI 제작"
categories: Life Unity TextBook
tag: [study, Unity]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 4.8.1 옵션 UI 레이아웃 구성

</br>
옵션 기능과 로그인 기능같은 로직적인 부분을 들어가기 이전에 게임 외적인 부분을 보강하는 법을 배웠다.

</br>

>목표

>>옵션 버튼과 화면 레이아웃을 구성한다.

>순서

1. 옵션 버튼 UI를 만들고 배치하기
2. 옵션 화면 배경과 버튼을 만들고 배치하기

</br>


## 4.8.1.1 옵션 버튼

</br>


이미지를 UI - Button 의 소스이미지로 사용하기 위해서는 이미지의 Texture Type을 Sprite로 바꿔야 한다.

Sprites 는 2D 그래픽 오브젝트입니다. 이미지(Image) 컨트롤은 사용자에게 상호작용하지 않는 이미지를 표시합니다. 

장식, 아이콘 등에 사용할 수 있으며, 스크립트를 통해 다른 컨트롤에 변경점을 반영하도록, 이미지를 변화시킬 수도 있습니다. 

이 컨트롤은 로우 이미지 컨트롤과 유사하지만, 이미지를 애니메이션화하고 컨트롤 사각형을 정확하게 채울 수 있도록 더 많은 옵션을 제공합니다. 

하지만, 이 컨트롤을 사용하려면 텍스처가 스프라이트여야 하는 반면, 로우 이미지 컨트롤은 모든 텍스처를 사용할 수 있습니다.

위의 규칙을 따르기에 UI - Button의 Image Component 의 Property인 Source Image는 Sprite Texture Type만을 취급하는 겁니다.

</br>

우측 상단에 옵션 버튼을 위치시키고 싶다면, Anchor 값을 조정합니다. Anchor 계산을 간단하게 할 수 있도록 Anchor Presets 를 제공합니다.

</br>

## 4.8.1.2 옵션 화면

</br>

옵션 버튼을 클릭했을 때 표시될 옵션 화면을 구성.

1_진한 바탕화면 이미지, 2_옵션 화면임을 알리는 텍스트, 3_계속하기 버튼, 4_다시하기 버튼, 5_나가기 버튼

위 다섯가지를 관리하기위한 빈 게임오브젝트. 빈 게임오브젝트를 Canvas 안에 추가하고, 위 5가지를 자식객체로 만든다.

</br>

# 4.8.2 옵션 기능 구현

</br>

> 목표

>> 옵션 버튼의 각 기능 구현

> 순서

1. 옵션 설정 상태 추가 ( Enum State )
2. 옵션의 각 버튼 기능 구현
3. 옵션 버튼마다 기능 함수들을 연결시키기
4. 구현해놓은 기능을 옵션창 뿐만 아니라 다른 곳에서도 호출하기

</br>

## 4.8.2.1 옵션 설정 상태 추가

</br>

옵션 기능은 게임의 룰과 관련이 있으므로 게임 매니저 클래스에서 정의.

옵션 버튼을 누를때 활성화해야할 인스턴스를 선언하고 public으로 할당할 것이다.

일시정지 상태를 추가하고, 일시정지 상태에서 시행할 함수를 선언할 것이다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    // 싱글톤 변수
    public static GameManager gm;
    
    private void Awake()
    {
        if(gm == null)
        {
            gm = this;
        }
    }

    // 게임 상태 상수
    public enum GameState
    {
        Ready,
        Run,
        Pause,//일시정지 상태를 나타내는 GameState.Pause
        GameOver
    }

    // 현재의 게임 상태 변수
    public GameState gState;

    // 게임 상태 UI 오브젝트 변수
    public GameObject gameLabel;

    //옵션 화면 UI 오브젝트 변수 여기서는 OptionPanel이라고 이름지은 빈게임오브젝트를 할당할 것.
    public GameObject gameOption;
```
 
 게임 중에 옵션 버튼을 누르면 일시정지 시키고 싶다. => 프로젝트의 Time 클래스에 있는 timeScale 값을 조절해 적용할 수 있다.

 timeScale은 게임 전체의 프레임 진행 시간의 배율이다. timeScale이 0.5라면 50%배속이고, 2라면 2배속, 1이면 정상배속, 0 이면 멈춤이다.

 버튼을 클릭할때 함수를호출하는  방식으로하고 싶다면 함수의 접근한정자를 public으로 선언하고, 각 버튼마다 있는 On Click()에 연결한다.


</br>


 ## 4.8.2.2 각 버튼의 기능 구현하기

</br>

### 1. 옵션 버튼

</br>

```c#
    // 옵션 화면 키는 함수
    public void OpenOptionWindow()
    {
        gameOption.SetActive(true);
        //게임속도 0배속으로 전환
        Time.timeScale = 0f;
        //게임상태를 일시정지 상태로 변경
        gState = GameState.Pause;
    }
```

</br>

### 2. 계속하기 옵션

```c#
    // 옵션 화면 끄는함수 계속하기 버튼은 이것을 호출할 것이다.
    public void CloseOptionWindow()
    {
        gameOption.SetActive(false);
        //게임 배속 정상화
        Time.timeScale = 1.0f;
        gState = GameState.Run;
    }
```

</br>

### 3. 다시하기 옵션

```c#
    //다시하기 옵션
    public void RestartGame()
    {
        //게임 배속 정상화, 로직상 일시정지 상태를 풀어주고 넘겨야한다.
        Time.timeScale = 1.0f;

        //현재 씬 번호를 다시 로드 하는 방식으로 처리.
        //이때 GameManger는 다시 start()로 돌아가 Ready상태를 만들 것이다.
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }
```

</br>

### 4. 게임종료 옵션

```c#
    //종료 옵션
    public void QuitGame()
    {
        //프로그램 종료함수
        Application.Quit();
    }
```

</br></br>

## 4.8.2.3 각 버튼에 기능호출 연결하기

</br>

Button Component의 OnClick() 이벤트에 기능을 구현해둔 클래스인 GameManager 오브젝트를 추가한다.

이후 필요한 함수들을 연결합니다.

</br>

## 4.8.2.4 게임 오버 상태일때 다시하기 버튼과 게임종료 버튼이 활성화

</br>

플레이어의 체력이 바닥나면 게임 오버 상태가 되도록 GameManager에서 제어하고 있다. 

이때 GameOver 텍스트와 함께 다시하기, 게임종료 버튼이 활성화가 되도록 하고 싶다.

게임 상태를 출력하는 Text 객체의 자식으로 빈게임오브젝트를 만든뒤에 

그 자식으로 버튼 2개를 배치하고 OnClick() 함수에 각 기능을 연결한다.

이후 항상 가려져있도록 빈게임오브젝트를 비활성화시키고, 

게임오버가 되는 부분에서 상태텍스트의 자식객체를 가져오고, 활성화한다.

```c#
   void Update()
    {
        // 만일, 플레이어의 hp가 0 이하라면...
        if(player.hp <= 0)
        {
            // 플레이어의 애니메이션을 멈춘다.
            player.GetComponentInChildren<Animator>().SetFloat("MoveMotion", 0f);

            // 상태 텍스트를 활성화한다.
            gameLabel.SetActive(true);

            // 상태 텍스트의 내용을 "Game Over"로 한다.
            gameText.text = "Game Over";

            // 상태 텍스트의 색상을 붉은색으로 한다.
            gameText.color = new Color32(255, 0, 0, 255);

            // 상태 텍스트의 자식객체로 둔 버튼 오브젝트의 트랜스폼 컴포넌트를 가져와서
            Transform buttons = gameText.transform.GetChild(0);

            //활성화 시키면 자식으로 세팅된 오브젝트들이 보인다.
            buttons.gameObject.SetActive(true);

            // 상태를 "게임 오버" 상태로 변경한다.
            gState = GameState.GameOver;
        }
    }
```






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
# 9.10 연습 문제

## 연습 문제 1.

NameCard 클래스의 GetAge(), SetAge(), GetName(), SetName() 메소드들을 프로퍼티로변경해 작성하세요.
```c#
using System;

namespace Ex9_1
{
    class NameCard
    {
        public int Age { get; set; }
        public stirng Name { get; set; }
    }
    class MainApp
    {
        public static void Main()
        {
            NameCard MyCard = new NameCard();

            MyCard.Age = 24;
            myCard.Name= "Noel";

            Console.WriteLine("나이 : {0}", MyCard.Age);
            Console.WriteLine("이름 : {0}", Mycard.Name);
        }
    }
}
```
## 연습 문제 2.

프로그램을 완성해서 다음과 같은 결과를 출력하도록 하라. 무명형식을 이용해야 한다.
>이름:noel, 나이:17

>Real:3, Imaginary:-12

```c#
using System;
namespace Ex9_2
{
    class MainApp
    {
        static void Main(string[] args)
        {
            var nameCard = new { Name = "noel", Age = 17 }
            Console.WriteLine("이름:{0}, 나이:{1}", nameCard.Name, nameCard.Age);
            
            var complex = new { Real = 3, Imaginary = -12 }
            Console.WriteLine("Real:{0}, Imaginary:{1}", complex.Real, complex.Imaginary);
        }
    }
}
```