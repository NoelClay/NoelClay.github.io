---
layout: single
tilte: "1.0 VR 개요"
categories: Life Unity VRAR TextBook
tag: [study, Unity]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

#  VR 플랫폼별 대응을 위한 원 소스 멀티 유즈

</br>


</br>

>목표

    VR 플랫폼별 대응을 위한 모듈 작업

>순서

    1. PC 환경에서 작업할 수 있도록 구성
    2. 오큘러스에서 작업할 수 있도록 구성


</br>

# 1.1 PC 환경에서 작업할 수 있도록 구성

</br>

```c#
#define PC
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public static class ARAVRInput
{
    ...
}
```
</br>
이렇게 전처리기 매크로로 PC를 설정한 뒤에 #if PC ... #endif 구문으로 PC환경에서 작동할 코드를 짜주면 된다.

</br>


## 1.1.1 프로퍼티 정의

</br>

컨트롤러에 대한 정보들을 얻어오고 세팅하고 반환하는 프로퍼티들을 작성.
왼쪽 컨트롤러, 오른쪽 컨트롤러의 트랜스폼을 세팅하고 반환하는 프로퍼티.
위치, 방향을 얻어서 반환하는 프로퍼티
</br>

양쪽 컨트롤러를 세팅하는 변수와 프로퍼티를 작성한다.
```c#
    // 씬에 등록된 왼쪽 컨트롤러 찾아 반환
    // LHand에 관련된 모든 값들을 이 스태틱변수에 저장하게 될것이고, 그러기 위해 get 프로퍼티가 사용될 것이다.
//왼쪽 컨트롤러가 저장될 변수
static Transform lHand;
//씬에 등록되어있는 왼쪽 컨트롤러의 transform을 찾아 반환
public static Transfrom LHand
{
    //프로퍼티에 접근하고자 할때 get이 발동하는데
    get
    {
        //만약 lHand 변수가 비어있다면? PC환경에선 무조건 비어있겠지.
        if (lHand == null)
        {
#if PC
            //LHand라는 게임오브젝트를 만들어서 handObj에 할당
            GameObject handObj = new GameObject("LHand");
            //만들어진 객체의 트랜스폼을 lHand 변수에 할당
            lHand = handObj.transform;
            //컨트롤러를 카메라의 자식 객체로 등록
            lHand.parent = Camera.main.transform;
#endif
        }
        return lHand;    
    }
}
```
</br>

```c#
    // 씬에 등록된 오른쪽 컨트롤러 찾아 반환
    // RHand에 관련된 모든 값들을 이 스태틱변수에 저장하게 될것이고, 그러기 위해 get 프로퍼티가 사용될 것이다.
    static Transform rHand;
    public static Transform RHand
    {
        get
        {
            if (rHand == null)
            {
#if PC
                //rHand라는 이름으로 게임 오브젝트를 만든다.
                GameObject handObj = new GameObject("RHand");
                //만들어진 객체의 트랜스폼을 rHand 변수에 할당
                rHand = handObj.transform;
                //컨트롤러를 카메라의 자식 객체로 등록
                rHand.parent = Camera.main.transform;
#endif
            }
            return rHand;
        }
    }
```

</br>

양쪽 컨트롤러의 위치와 방향을 세팅하면서 동시에 반환하는 프로퍼티
```c#
    // 오른쪽 컨트롤러의 위치 얻어오기
    public static Vector3 RHandPosition
    {
        get
        {
#if PC
            //Input.mousePosition은 마우스의 스크린 좌표(2D)를 얻어옵니다.
            Vector3 pos = Input.mousePosition;

            //2D인 마우스포지션값을 3D인 pos에 넣어놓고 z값을 0.7로 설정하여
            //진짜 손이 있을법한 가상의 위치를 고정으로 세팅
            pos.z = 0.7f;

            //위의 스크린 x,y와 가상의 0.7z값을 설정해봤자. 결국 pos.transform.position 값은 스크린의 위치값이지.
            //실제 월드상의 좌표는 아니다. Camera.main.ScreenToWorldPoint( 스크린좌표값 ) 의 결과는
            //메인카메라의 월드 위치를 기반으로 스크린좌표값을 실제 게임 월드좌표값으로 반환하는 메소드
            pos = Camera.main.ScreenToWorldPoint(pos);

            //RHand 프로퍼티를 참조하여 position값을 세팅.
            RHand.position = pos;
            return pos;
#endif
        }
    }
    // 오른쪽 컨트롤러의 방향 얻어오기
    public static Vector3 RHandDirection
    {
        get
        {
#if PC
            //벡터의 뺄셈: 뒤 벡터의 종점에서 앞 벡터의 종점으로 가는 벡터.
            //메인 카메라 트랜스폼의 중점에서 마우스 포인터의 위치에 따라 손의 포워드 방향이 회전하게 될 것이다.
            Vector3 direction = RHandPosition - Camera.main.transform.position;
            //디렉션이 구해졌으면 트랜스폰 프로퍼티 참조해서 forward 세팅.
            RHand.forward = direction;
            return direction;
#endif
        }
    }
    // 왼쪽 컨트롤러의 위치 얻어오기
    public static Vector3 LHandPosition
    {
        get
        {
#if PC
            Vector3 pos = Input.mousePosition;

            pos.z = 0.7f;

            pos = Camera.main.ScreenToWorldPoint(pos);

            RHand.position = pos;
            return pos;
#endif
        }
    }
    // 왼쪽 컨트롤러의 방향 얻어오기
    public static Vector3 LHandDirection
    {
        get
        {
#if PC
            Vector3 direction = RHandPosition - Camera.main.transform.position;
            RHand.forward = direction;
            return direction;
#endif
        }
    }
```
</br>

ARAVRInput.cs 클래스에서 사용하는 열거형 자료형을 추가한다.

버튼과 왼쪽 오른쪽 컨트롤러를 구별하는 용도이다.

ButtonTarget 열거형은 오큘러스에서 사용하는 이름을 동일하게 사용한다.

PC와 vive 환경에서 사용할 것이므로 선언 자체를 #if...#endif 안에 한다.

Button 열거형은 각 열거 상태를 추가하고, ButtonTarget 열거형 상수값을 할당하는 형태이다.

Controller 열거형 타입은 두가지 타입 LTouch, RTouch가 있음

```c#
    //PC에서도 OculusAPI에서 사용하는 버튼과 같은 방식으로 작동하게 만들기위해
    //PC라면 사용할 ButtonTarget enum을 선언, 다른 컨트롤러들은 각자 버튼 이벤트시
    //사용할 타겟들이 이미 정해져있기에 PC용 ButtonTarget을 세팅한다.
#if PC
    public enum ButtonTarget
    {
        Fire1, Fire2, Fire3, Jump
    }
#endif
    public enum Button
    {
        //각 버튼을 클릭할시 ButtonTarget의 상수값들이 호출되도록하는 enum
#if PC
        One = ButtonTarget.Fire1, Two = ButtonTarget.Jump,
        Thumbstick = ButtonTarget.Fire1, IndexTrigger = ButtonTarget.Fire3,
        HandTrigger = ButtonTarget.Fire2
#endif
    }
    //Controller의 좌우를 구분하는 열거형 추가
    public enum Controller
    {
#if PC
        LTouch, RTouch
#endif
    }

```

</br>

## 1.1.2 Get, GetDown, GetUp 함수 구현하기

</br>

Input 클래스에 있는 GetButton(), GetButtonDown(), GetButtonUp() 함수들을 호출하는데 

각 환경에 맞게 인수를 전달하며 호출 할 수 있도록 세팅하는 과정

각 함수들은 전달인자로 Button 열거형과 Controller 열거형을 가진다. 

실제 Input 클래스에 있는 버튼 함수들은 그런 인자를 가지고 있지 않지만,

여기서 세팅하고자하는 건 컨트롤러로 제어할때 이용할 함수이기에 그렇다.

```c#
    // Get 함수는 버튼 열거형을 매개변수로 받으면
    // 각 플랫폼에 맞게 ButtonTarget 열거형을 스트링화하여
    // Input클래스의 GetButton 메소드의 매개변수로 전달해주는 메소드
    public static bool Get(Button virtualMask, Controller hand = Controller.RTouch)
    {
#if PC
        return Input.GetButton(((ButtonTarget)virtualMask).ToString());
#endif
    }

    //전달인자인 매개변수는 위의 Get함수와 동일하며
    //위의 Get함수는 GetButton메소드를 사용하기 위한 함수였다면,
    //GetDown함수는 각 플랫폼별 GetButtonDown을 사용하기 위한 함수다.
    public static bool GetDown(Button virtualMask, Controller hand = Controller.RTouch)
    {
#if PC
        return Input.GetButtonDown(((ButtonTarget)virtualMask).ToString());
#endif
    }
    public static bool GetUP(Button virtualMask, Controller hand = Controller.RTouch)
    {
#if PC
        return Input.GetButtonUp(((ButtonTarget)virtualMask).ToString());
#endif
    }
```

</br>

## 1.1.3 GetAxis 함수 구현하기

</br>

위의 버튼 함수들의 경우엔 눌릴때 혹은 때질때 이벤트가 발생하거나 눌리고 있는지 여부를 검출한다. 

이런 함수들을 Action 함수라고 한다. 하지만 조이스틱이나 썸스틱 같이 

어느정도 변화되고 있는지 여부를 알기 위해선 Axis 함수를 사용해야 한다. 

Input 클래스의 GetAxis 함수를 호출하고 싶은데 컨트롤러를 사용하는 환경에서 사용하기 위해 세팅한다.

```c#
    // 컨트롤러의 Axis 입력을 반환 Input 클래스의 GetAxis()를 적절하게 사용가능하도록 세팅
    // axis: Horizontal, Vertical InputManager에 정의되어 있는 축문자열
    public static float GetAxis(string axis, Controller hand = Controller.LTouch)
    {
        //PC 환경에서는 GetAxis(축문자열)의 반환값을 반환한다.
#if PC
        return Input.GetAxis(axis);
#endif
    }
```

</br>

## 1.1.4 진동 함수 PlayVibration 구현하기

</br>

```c#
    //진동을 호출할 함수
    public static void PlayVibration(Controller hand)
    {
        //PC 환경에선 기능이 없다.
    }
```

</br>

## 1.1.5 원하는 방향으로 중심을 재설정하는 Recenter 함수 구현하기

</br>

키보드 마우스를 이용해 플레이하는 1인칭 게임에서는 플레이어가 월드에서

고정된 위치를 바라보고 있으므로 게임 시작시 비교적 원하는 위치를 바라보며 시작할 수 있다.

하지만 VR에서는 플레이어가 헤드셋을 끼고 회전 기본 세팅이 변경될 수 있으므로 

재설정이 필요할 수 가 있다. 

```c#
    //카메라가 바라보는 방향으로 월드 센터를 제설정하는 함수이다. 
    //VR에서는 헤드셋이 항상 같은 방향으로 게임이 시작되는게 아니므로
    //잘 활용해야 초기 정면 세팅에 유용하다.
    public static void Recenter()
    {

    }

    //Recenter 함수의 오버로딩으로 매개변수는 타겟 트랜스폼과 원하는방향이다.
    //게임에서 뒤돌기 기능을 호출하고 싶다면 이 함수를 이용하여 적용할 수 있다. 
    //어떤 이벤트나 혹은 버튼을 누를때 target은 플레이어에 direction은 Vector3.back이라면
    //그 즉시 뒤로 회전하는 함수가 완성됨.
    public static void Recenter(Transform target, Vector3 direction)
    {
        target.forward = target.rotation * direction;
    }
```

</br>

## 1.1.6 조준점 크로스헤어를 표현하는 DrawCrosshair 함수 구현하기

</br>

조준점 크로스헤어를 표현하는 DrawCrosshair 함수 구현하기

일반 PC게임과 달리 VR게임은 표현해야되는 화면이 2개라서 UI처럼 그냥 크로스헤어를 박아넣는 형식으로

표현하게 될 경우 착시현상이 발생한다. 그 착시를 막기위한 연산은 최적화를 떨어뜨린다. 

그래서 VR에선 크로스헤어를 카메라에서 레이가 오브젝트 표면에 닿을때 

그 오브젝트 위에 크로스헤어를 그리는 방식으로 할것이다.

</br>

DrawCrosshair 함수의 매개변수들은 3가지

그릴려는 오브젝트의 위치정보인 Transform crosshair

손위치와 방향을 기반으로 레이를 쏴서 크로스헤어를 그릴건지 여부를 세팅할 bool isHand

오른손컨트롤러인지 왼쪽컨트롤러인지의 정보를 전달할 Controller hand

</br>

isHand의 기본값은 true. 만약 false를 전달하면 컨트롤러를 기반으로 쏘지 않고 카메라 중앙을 기점으로 레이가 나간다.

</br>

>목표

    광선 레이가 닿는 곳에 크로스헤어 위치시키기

>순서

    1. 광선 레이 만들기
    2. 레이 쏘기
    3. 부딪힌 점이 있다면 크로스헤어 위치시키기
    4. 부딪힌 점이 없다면 허공에 크로스헤어 위치시키기



</br>

### 1.1.6.1 광선 레이 만들기

</br>

```c#
    public static void DrawCrosshair(Transform crosshair, bool isHand =true, Controller hand = Controller.RTouch)
    {
        Ray ray;
        //컨트롤러의 위치와 방향을 이용해 레이 제작
        if (isHand)
        {
#if PC
            ray = Camera.main.ScreenPointToRay(Input.mousePosition);
#endif
        }
        else
        {
            //카메라를 기준으로 화면의 정중앙으로 레이를제작한다.
            ray = new Ray(Camera.main.transform.position, Camera.main.transform.forward);
        }
    }
```

</br>

### 1.1.6.2 레이쏘기

</br>

레이를 생성했으면 레이를 쏴야 크로스 헤어를 띄울 위치를 찾을 수 있다. 

그냥 쏘지 않고 Plane 클래스를 이용하는 방법이 있다. Plane은 월드 공간에 존재하는 

보이지 않는 무한의 평면을 말한다. 어떤 지점을 지나갈 수 있고 면의 방향이 존재한다. 

</br>

#### Plane 생성자

</br>
이런 Plane을 생성하는 생성자의 파라미터를 보면

public Plane(Vector3 inNormal, Vector3 inPoint)

public Plane(Vector3 inNormal, float d)

public Plane(Vector3 a, Vector3 b, Vector3 c)

inNormal: Plane 면이 향하는 방향.

inPoint: plane 면이 지나가야되는 Vector3 지점.

d: 월드공간 원점으로부터의 수직 거리

a, b, c: 지나가야되는 세점 좌표

</br>

#### Plane.Raycast

</br>

Physics.Raycast 를 안쓰고, Plane.Raycast 를 쓴다. 고려해야될 것을 알아본다.

</br></br>
일단 첫 번째, 화면에 일정한 크기의 크로스헤어를 띄울 생각이기 때문이다.

실제 FPS에서 레이에 물체가 맞았는지는 여기서 중요하지 않다. 

우리는 양안 착시 없이 크로스헤어를 띄우는게 목적이기 때문이다.
</br></br>
두 번째 Canvas camera screen 옵션이 아닌 월드 좌표에 크로스헤어를 크기 조절하여 띄울 건데,

저 멀리 있는 크로스헤어가 물체에 가려지면 무슨 소용일까? 하지만 걱정할 필요가 없다. 

나중에 셰이더 작업을 통해 크로스헤어가 보이도록 할 것이다.
</br></br>
세 번째 Plane.Raycast란 무엇인가. Plane 클래스에 정의되어 있는 함수이며,

특정 Plane과 Ray가 만나는지를 검출하는 함수이다. 만났다면, Ray.Origin 으로부터 맞은 지점까지의

거리를 out 키워드와 함께 쓰인 float 매개변수에 반환한 뒤에 함수 자체는 참을 반환한다.

만나지 않았다면 거짓을 반환한다.
</br>

```c#
        //눈에 보이지 않는 Plane을 바닥에 쫙 깔아 놓는다.
        Plane plane = new Plane(Vector3.up, 0);
        float distance = 0;

        //만들어 놓은 Ray를 Plane 클래스의 Plane.Raycast로 검출한다.
        if(plane.Raycast(ray,out distance))
        {

        }
```
</br>

### 1.1.6.3 레이가 검출되면 크로스헤어 그리기
</br>

Plane이 Ray를 검출하게 되면 인자로 전달된 float에 ray.Origin 과 충돌 지점간의 거리가 반환된다는 것을 배웠다.</br>
이 distance를 ray.GetPoint()에 인자로 전달하면 레이가 distance가 찍힌 좌표를 찾아서 월드좌표로 반환한다.</br>
그 월드 좌표를 크로스헤어의 좌표로 한 뒤에 항상 메인카메라를 바라 볼수 있도록</br> 
Camera.main.transform.forward의 역벡터를 크로스헤어의 포워드로 세팅</br>
크로스 헤어의 크기를 최소 기본 크기서 거리에따라 더 커지게 한다.</br>
</br>

```c#
//PC 환경에서 크로스헤어의 originScale을 설정한다. 카메라에 일정하게 보이게 하기 위해서 
//나중에 멀어진거리만큼 곱해줄 것이다.
#if PC
    static Vector3 originScale = Vector3.one * 0.02f;
#endif
    public static void DrawCrosshair(Transform crosshair, bool isHand =true, Controller hand = Controller.RTouch)
    {
        Ray ray;
        //컨트롤러의 위치와 방향을 이용해 레이 제작
        if (isHand)
        {
#if PC
            ray = Camera.main.ScreenPointToRay(Input.mousePosition);
#endif
        }
        else
        {
            //카메라를 기준으로 화면의 정중앙으로 레이를제작한다.
            ray = new Ray(Camera.main.transform.position, Camera.main.transform.forward);
        }

        //눈에 보이지 않는 Plane을 바닥에 쫙 깔아 놓는다.
        Plane plane = new Plane(Vector3.up, 0);
        float distance = 0;

        //만들어 놓은 Ray를 Plane 클래스의 Plane.Raycast로 검출한다.
        if(plane.Raycast(ray, out distance))
        {
            //레이의 GetPoint 함수를 이용해 충돌 지점의 위치를 가져온다.
            crosshair.position = ray.GetPoint(distance);
            //메인카메라를 항상 바라보게 세팅한다.
            crosshair.forward = -Camera.main.transform.forward;
            //크로스헤어의 크기를 최소 기본 크기에서 일정 거리 이상은
            //더 커지게해서 눈에 보이는 외관의 크기를 유지한다.
            crosshair.localScale = originScale * Mathf.Max(1, distance);
        }
    }
```
</br>

### 1.1.6.4 레이가 검출되지 않을때(즉 바닥을 향하고 있지 않아서 Plane을 검출 못할때)
</br>

Plane을 전혀 검출할 수 없을때에는 

크로스헤어를 ray.origin으로 부터 레이의 방향으로 일정 거리의 허공에 위치시키고</br>
방향은 카메라를 바라보게 하고, distance값은 위의 일정거리가 될 것이다.</br>
구하는 방법은 크로스헤어의 월드좌표에서 레이의 원점 월드좌표를 뺀 뒤에 매그니튜드하면</br>
float 거리값을 반환해줄 것이다.</br>

```c#
        else
        {
            crosshair.position = ray.origin + ray.direction * 100;
            crosshair.forward = - Camera.main.transform.forward;
            distance = (crosshair.position - ray.origin).magnitude;
            crosshair.localScale = originScale * Mathf.Max(1, distance);
        }
```
</br>

</br>

# 1.2 오큘러스 세팅 만들기</br>


</br>

## 1.2.1 프로퍼티 정의
</br>

</br></br></br>

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


