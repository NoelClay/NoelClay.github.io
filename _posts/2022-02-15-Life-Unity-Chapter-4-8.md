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
