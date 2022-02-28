---
layout: single
tilte: "4.10 UI 디자인 레이아웃"
categories: Life Unity TextBook
tag: [study, Unity]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

</br>

UI란 사용자와 컨텐츠가 상호작용하기 위한 시스템을 말한다.

</br>

# 4.10.1 유니티 UI 시스템 개념 정리

</br>

>목표

    UI 시스템의 기본 개념을 안다.

>순서

    1. 유니티 UI 시스템 구조 알기
    2. 캔버스의 속성 알아보기
    3. 앵커와 피벗에 대해 알아보기

과거 유니티의 UI 시스템은 성능이 좋지 못해서 NGUI 같은 서드파티 플러그인을 사용.

지금은 개선되어 좋아짐.

</br>

## 4.10.1.1 유니티 UI 시스템 구조


</br>

create 오브젝트를 할때 UI 오브젝트를 생성하면 항상 Canvas와 EventSystem이 자동으로 생성됨.

Canvas는 UI 오브젝트들을 담고 있는 도화지와 같은 녀석이라 한 씬에 여러개 생성가능.

EventSystem은 최상위 객체로 존재해야하며, 하나만 존재 가능.

UI 오브젝트들 처럼 캔버스 안에 자식으로 존재해야 활용이 가능한 방식을

컨테이너 방식이라고함. 

만약 캔버스 안이 아닌 바깥에 나오게 되면 UI 오브젝트들은 그냥 게임오브젝트화 된다.

가장 큰 차이점은 트랜스폼이다. 게임오브젝트는 트랜스폼 컴포넌트를 가지고 있지만,

UI오브젝트는 렉트 트랜스폼 컴포넌트를 가지고 있다. 

그래서 일반 UI 오브젝트는 월드상의 게임오브젝트처럼 사용 될 수 없는 것이다.

렉트 트랜스폼과 일반 트랜스폼은 공존할 수 없다.

</br>

## 4.9.1.2 배경 이미지 배치

</br>


해상도 비율과 유사한 이미지를 선택해 다운 받고 Materials 폴더에 복사

UI - Raw Image 오브젝트 생성 

Raw Image는 Sprite 타입뿐만 아니라 일반  텍스쳐도 사용가능한 범용적인 컴포넌트

</br>

## 4.9.1.3 입력란 배치
</br>

입력란은 각각 아이디라는 텍스트와 실제 텍스트를 입력할 입력란 2가지가 필요하다.

텍스트 오브젝트를 생성하고 아이디가 나오게 입력.

그리고 옆에 배치할 입력란은 Input Field 오브젝트를 선택하여 만든다.

인풋 필드는 입력 영역 박스 이미지 외에 자식 오브젝트로 

사용자가 입력하기를 대기하고 있는 플레이스 홀더와 

사용자가 입력하면 그것을 보여주는 텍스트를 갖고 있다.

플레이스 홀더 안에 Text 컴포넌트를 '아이디 입력'으로 변경.

인풋 필드 컴포넌트에는 사용자 입력 조건에 관한 설정항목들이 존재.

|항목|설명|
|-----|---|
|Text Component| 사용자의 텍스트 입력설정을 위한 오브젝트를 지정|
|Text| 사용자의 입력 내요이 저장될 변수|
|Character Limit|입력 가능한 문자열의 개수(영문, 숫자는 한 문자당 1씩 소모,</br>한글의 경우 한문자당 2를 소모), 이 항목이 0이면 제한없음.|
|Content Type|Standard: 모든 숫자, 모든 문자열</br>Integer: 정수만</br>Decimal: 실수만</br>Alphanumeric: 양의 정수만</br>Name: 문자열만</br>Email: 이메일 형식만</br>Password: 입력된  ***형태로 가려져서 표시. 모든 숫자, 모든 문자열</br>PIN: 입력된 데이터가 ***형태로 가려져서 표시. 양의 정수만</br>Custom: 사용자 설정으로 입력 제한|
|Line Type|Single Line: 한 줄로만 입력 가능</br> Multi Line Submit: 여러 주로 입력 가능. 자동 개행</br> Multi Line New Line: 여러 줄로 입력 가능. 사용자가 Enter 키를 누르면 개행|
|Placeholder| 미입력 시 표시될 텍스트 오브젝트를 지정할 수 있는 항목|
|Caret Blink Rate|커서의 깜빡임 속도|
|Caret Width|커서의 두께|
|Custom Caret Color|커서의 색상|
|Selection Color|모두 선택된 텍스트의 선택 영역 색상|
|Hide Mobile Input|모바일 장치에서 화면 키보드에 연결된 기본 입력 필드를 숨김(IOS일 때만)|

</br>

## 4.9.1.4 새로 생성 버튼, 로그인 버튼, 에러 텍스트 배치

</br>

버튼 오브젝트를 밑에 두고, 에러 텍스트를 위에 배치한다.

</br>

# 4.9.2 로그인 기능 구성

</br>

> 목표

    사용자 데이터를 새로 작성하거나 저장된 데이터를 읽어 사용자 입력과 일치하는지 검사

> 순서

    1. 인풋 필드의 값을 읽기 위한 변수 만들기
    2. 사용자의 아이디는 Key로 패스워드를 Value로 설정해 저장하는 기능 만들기
    3. 사용자의 아이디를 키로 사용해 패스워드 값을 읽어오는 기능
    4. 사용자의 입력이 비어있다면 기능이 없도록 제어
    5. 로그인 화면 UI 버튼과 기능 연결


</br>

## 4.9.2.1 인풋 필드의 값을 읽기 위한 변수

</br>

LoginManager라는새로운 스크립트 생성.

InputField를 연결하여 접근하는 public InputField 변수를 선언

에러가 나든 일치를하든 현재 상태를 알려줄 텍스트를 public으로 연결

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class LoginManager : MonoBehaviour
{
    
    public InputField id;

    public InputField password;

    public Text notify;
    // Start is called before the first frame update
    void Start()
    {
        //처음엔 빈 텍스트로 초기화. 이후상태 메시지를 띄울 것이다.
        notify.text = "";
    }
}
```

</br>

## 4.9.2.2 사용자 아이디 = 키, 패스워드 = 값 으로 저장하는 기능

</br>

데이터를 게임 외 공간에 저장하고 불러오는 기능을 구현하려면 파일 입출력 기능을 만들어야 한다.

기본적으로 저장공간과 메모리공간을 연결다리 역할인 Stream을 제어하는 방식으로 사용한다. 

유니티에서는 보다 쉽게 데이터르를 관리하라고 PlayerPrefs라는 클래스를 제공한다.

정수형, 실수형, 문자열 데이터들을 Get/Set 함수들로 관리하는데 사용법은 다음과 같다.

```c#
PlayerPrefs.SetString(저장할 키값, 저장할 밸류값);

GetString(키값) == 저장된 밸류값;

HasKey(키값);//해당 키값을 가지고 있으면 참 반환 
```


그래서 저장기능은 다음과 같이 만든다.


```c#
    public void SaveUserData()
    {
        //HasKey는 키값이 존재하는지를 체크하는 함수
        if (!PlayerPrefs.HasKey(id.text))
        {
            //Key and Value로 데이터 저장
            PlayerPrefs.SetString(id.text, password.text);
            notify.text = "아이디 생성 완료";
        }
        else 
        {
            notify.text = "이미 존재하는 아이디 입니다.";
        }     
    }
```



</br>

## 4.9.2.3 키를 이용해 아이디 패스워드 데이터를 읽는 기능

</br>

로그인을 하기 위해서는 입력하는 아이디와 패스워드가 

저장된 데이터와 일치하는지를 검사해야함.

그럴려면 읽어와야 함.


```c#
    public void CheckUserData()
    {
        string tempPassword = PlayerPrefs.GetString(id.text);

        if(tempPassword == password.text)
        {
            SceneManager.LoadScene(1);
        }
        else
        {
            notify.text = "입력한 정보와 일치하는 데이터가 존재 하지 않습니다.";
        }
    }
```

</br>


## 4.9.2.4 사용자의 입력이 비어있는 상태에서는 기능 잠금

</br>

사용자 입력이 빈칸이 있다면, 위에 짠 함수를 호출할때 예외가 발생하며 에러를 발생시킨다.

InputField의 텍스트값에 접근하는데 값이 없을 것이기 때문.

그렇다면 조건 검사하고 아무것도 안하고 끝내는 함수를 구현한 뒤에 

위의 각 기능들 앞에 호출한다.

```c#
    bool CheckInput(string id, string pwd)
    {
        if(id == "" || pwd == "")
        {
            notify.text = "아이디 또는 패스워드를 입력하세요.";
            return false;
        }
        else
        {
            return true;
        }
    }
```

</br>

## 4.9.2.5 기능 연결

</br>

빈 게임오브젝트 객체 LoginManager 생성. 각 값에 public 연결하고

버튼마다 OnClick 이벤트 추가. 한 뒤에 빌드세팅 씬넘버 추가.

</br>

# 4.9.3 비동기 씬 로딩하기

</br>

로딩이 오래 걸릴때 그 씬을 숨겨놓고 씬이 로딩이 다 되고나서 보여주는 

방식을 비동기적인 방식이라 한다.

> 목표

    다음 씬을 비동기 방식으로 로드하고 싶다. 또한 현재 씬에는 로딩 진행률을 시각적으로 표현하고 싶다.

> 순서

    1. 로딩 화면을 위한 씬을 만들고 로딩 화면 UI 구성하기
    2. 비동기 로드 과정을 구현한 코루틴 함수 구현하기
    3. 빌드 세팅에서 빌드 씬에 로드 씬 추가하기

</br>

# 4.9.3.1 로딩 씬을 만들고 UI 구성

</br>

배경과 로딩슬라이더, 간단한 텍스트 정도 넣으면 로딩 씬 UI 끝.


