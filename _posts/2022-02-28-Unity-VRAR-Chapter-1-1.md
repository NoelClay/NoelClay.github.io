---
layout: single
tilte: "1.1 매직복셀 제작"
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


#define PC 를 주석처리하여 #if PC ... #endif 전처리기 블록을 읽지 않도록 한다.

#define Oculus를 추가하여 전처리기 Oculus를 참으로 만들어 #if Oculus ... #endif 를 읽게 만든다.

오큘러스 VR 환경에서 제일 다른 점은 컨트롤러와 헤드셋을 이용하여 입력을 컨트롤하고, 게임과 상호작용 한다는 점이다.

센서의 정보들, 버튼과 스틱, 터치 등등을 가져와서 제어하기 위해서는 OVRCameraRig의 TrackingSpace 객체를 가져와서 사용해야 한다.

TrackingSpacd를 static 변수에 넣어서 숨기되, 동적으로 저장하고 불러올 수 있도록 하는 코드를 작성한다.

```c#
#if Oculus
    //오큘러스 환경에서 컨트롤러를 제어하기 위해서는 OVRCameraRig의 TrackinSpace 객체를 가져와서 하위 정보들을 활용해야 한다.
    static Transform rootTransform;
#endif

#if Oculus
    static Transform GetTransform()
    {
        if(rootTransform == null)
        {
            rootTransform = GameObject.Find("TrackingSpace").transform;
        }
        return rootTransform;
    }
#endif
```

</br>

오큘러스에 대응하는 HandPosition을 구현한다.

오큘러스에 내장된 OVRInput 클래스는 GetLocalControllerPosition()이라는 함수를 지원한다.

이 함수는 컨트롤러의 로컬포지션을 반환하는데 HandPosition 프로퍼티는 월드포지션을 반환해야한다.

월드포지션은 Transform.TransformPoint() 함수를 이용해 구하면 된다.

이때 사용하고자 하는 공간이 static이므로 static Transform을 가져와서 사용해야하는데

GetTransform() 을 이용하여 static rootTransform을 사용하여 월드로 만든다.

```c#
    public static Vector3 RHandPosition
    {
        get
        {
            ...(생략)...

            //RHandPosition은 진짜 게임 월드상의 오른손 위치를 반환하는 프로퍼티이다.
            //오큘러스에 내장된 OVRInput 클래스의 GetLocalControllerPosition() 함수를 이용하여 
            //컨트롤러의 로컬 포지션을 가져올 수 있는데
            //로컬 포지션을 월드포지션으로 변경하여 반환하면 적절하다.
            //Transform.TransfromPoint()를 이용하면 된다.
#if Oculus
            Vector3 pos = OVRInput.GetLocalControllerPosition(OVRInput.Controller.RTouch);
            //정적으로 설정된 공간에서 TransformPoint(Vector3 position)을 사용하기 위해서는
            //정적으로 선언된 트랜스폼 인스턴스를 가져와서 사용해야 쓸 수 있다.
            pos = GetTransform().TransformPoint(pos);
            return pos;
#endif
```

오큘러스에 대응하는 RHandDirection 만들기

위와 비슷한 방식으로 OVRInput.GetLocalControllerRotation(OVRInput.Controller.RTouch)로

로컬 로테이션을 가져올 수 있다. 그런데 이것이 반환하는 값은 쿼터니언 값이다.

쿼터니언 값으로 원하는 방향벡터를 얻고 싶으면 그 방향 벡터를 곱해서 얻는다.

그렇게 얻은 로컬 방향 벡터를 월드 벡터로 바꿔주면 된다.

```c#
    public static Vector3 RHandDirection
    {
        get
        {
            ...(생략)...
#if Oculus
            Vector3 direction = OVRInput.GetLocalControllerRotation(OVRInput.Controller.RTouch) * Vector3.forward;
            direction = GetTransform().TransformDirection(direction);
            return direction;
#endif
        }
    }
```

LHandPosition과 LHandDirection은 같은 방식으로 Controller 인자를 LTouch로 바꿔주기만하면 된다.

RHand와 LHand 트랜스폼객체를 등록하는 것은 더 간단하다. 

TrackingSpace 안에는 오큘러스 하드웨어가 인체의 동작을 추적하여 각 객체에 정보를 업데이트 하는 자식객체들이 많다.

그중 왼쪽 오른쪽 손의 정보를 저장하는 객체들은 각각 LeftHandAnchor, RightHandAnchor 이다. 이것들을 각각의 프로퍼티에서 세팅하고 반환하면 된다.

</br>

## 1.2.2 열거형 타입 정의

</br>

PC와 달리 ButtonTarget을 설정할 필요가 없다. 각 버튼의 쓰임새에 맞는 명칭들이 이미 존재한다.

```c#
    public enum Button
    {
#elif Oculus
        One = OVRInput.Button.One, Two = OVRInput.Button.Two,
        Thumbstick = OVRInput.Button.PrimaryThumbstick, 
        IndexTrigger = OVRInput.Button.PrimaryIndexTrigger, 
        HandTrigger = OVRInput.Button.PrimaryHandTrigger
#endif
    }
    public enum Controller
    {
#elif Oculus
        LTouch = OVRInput.Controller.LTouch,
        RTouch = OVRInput.Controller.RTouch
#endif
    }
```

</br>

## 1.2.3 Get, GetDown, GetUp

</br>

오큘러스 환경에서 사용가능한 OVRInput 클래스의 각 함수들을 적절하게 반환하는 형식으로 사용한다.

```c#
    public static bool Get(Button virtualMask, Controller hand = Controller.RTouch)
    {
#elif Oculus
        return OVRInput.Get((OVRInput.Button)virtualMask, (OVRInput.Controller)hand);
#endif
    }

    //전달인자인 매개변수는 위의 Get함수와 동일하며
    //위의 Get함수는 GetButton메소드를 사용하기 위한 함수였다면,
    //GetDown함수는 각 플랫폼별 GetButtonDown을 사용하기 위한 함수다.
    public static bool GetDown(Button virtualMask, Controller hand = Controller.RTouch)
    {
#elif Oculus
        return OVRInput.GetDown((OVRInput.Button)virtualMask, (OVRInput.Controller)hand);
#endif
    }
    public static bool GetUp(Button virtualMask, Controller hand = Controller.RTouch)
    {
#elif Oculus
        return OVRInput.GetUp((OVRInput.Button)virtualMask, (OVRInput.Controller)hand);
#endif
```

</br>

## 1.2.4 GetAxis 함수 구현하기

</br>

PC 환경에서의 Input.GetAxis(axis) 는 float을 반환하는 함수입니다. 연속체크가 가능함.

그것을 오큘러스 환경에서는 OVRInput.Get() 함수가 대체합니다. OVRInput.Get() 함수의 원형은

public static Vector2 Get(Axis2D virtualMask, Controller controllerMask = Controller.Active)

Vector2 타입을 리턴하며 가로축 입력은 x에 저장되고, 세로축 입력은 y에 저장됩니다.

float 값을 반환하는 함수로 대체를 해주기 위해서는 조건에따라서 x를 반환할지 y를 반환할지 결정하면 됩니다.

```c#
    public static float GetAxis(string axis, Controller hand = Controller.LTouch)
    {
        //PC 환경에서는 GetAxis(축문자열)의 반환값을 반환한다.
#if PC
        return Input.GetAxis(axis);
#elif Oculus
        if(axis == "Horizontal")
        {
            return OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick, (OVRInput.Controller)hand).x;
        }
        else
        {
            return OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick, (OVRInput.Controller)hand).y;
        }
#endif
    }
```

</br>

## 1.2.5 진동 함수 PlayVibration

</br>

오큘러스 API 내부에 있는 OVRInput 클래스의 SetControllerVibration() 함수를 이용해서 진동가능

위의 함수는 2초 진동하고 끝나는 방식으로 단순하기에 섬세하게 제어하려면 코루틴을 사용해야함.

코루틴은 MonoBehaviour 를 상속받아 씬에서 계속 등록되어 있어야 사용할 수 있다.

하지만 ARAVRInput 클래스는 static 클래스로 결이 다른 클래스이다.

그래서 ARAVRInput.cs 파일 안에 MonoBehaviour를 상속받으면서 씬이 넘어가더라도 

사라지지 않고 계속 존재하는 클래스를 만들어 준다.(싱글톤 스타일)

MonoBehaviour를 상속받는 클래스를 이용해서 코루틴을 사용할 것이다.

```c#
//ARAVRInput 클래스에서 사용할 코루틴 객체
class CoroutineInstance : MonoBehaviour
{
    public static CoroutineInstance coroutineInstance = null;
    private void Awake()
    {
        if(coroutineInstance == null)
        {
            coroutineInstance = this;
        }
        DontDestroyOnLoad(gameObject);
    }
}
```

진동을 제어할 코루틴 함수를 만들기. VibrationCoroutine() 함수를 만든다.

duration: 지속시간, frequency: 빈도, Amplitude: 진동 크기, hand: RTouch또는 LTouch

OVRInput의 SetControllerVibration()함수를 세부제어하는 코루틴이다. 

```c#
#if Oculus
    static IEnumerator VibrationCoroutine(float duration, float frequency, float amplitude, Controller hand)
    {
        float currentTime = 0;

        //전달된 인수인 duration 동안 반복할려고 한다.
        //2초 이상 반복하기 위해 
        while (currentTime < duration)
        {
            //시간 누적으로 지속시간을 확인한다.
            currentTime += Time.deltaTime;

            //반복 누적호출되어도 상관 없음.
            OVRInput.SetControllerVibration(frequency, amplitude, (OVRInput.Controller)hand);
            
            //yield return null을 하게 되면 1프레임쉬고 넘어간다는 뜻(리턴한다는 뜻은 아니다.)
            //즉, update 함수가 실행되고 난 다음에 실행하란 뜻
            //반복문 안에서의 yield return null은 반복문의 주기를 Update()와 동일시 하기 위함이다.
            //yield return null을 안써주면 너무 빠른 반복으로 유니티가 크래쉬나게 된다.
            //사실상 코루틴이 호출되게 되면 duration동안 반복 호출된다.
            yield return null;
        }
        //0, 0 으로 설정하면 진동이 멈춘다. 반복문 빠져나가면 멈추게
        OVRInput.SetControllerVibration(0,0, (OVRInput.Controller)hand);
    }
#endif
```

코루틴을 작성했으니 코루틴을 사용하는 함수를 작성한다. 실질적인 오큘러스 환경의 PlayVibration()함수

모노비헤이비어를 상속받지 않는 스태틱 클래스에서 스타트 코루틴을 사용하기 위해

싱글톤 객체가 없다면 만드는 코드를 작성.

플레이중인 진동 코루틴은 항상 정지하고 코루틴을 시작한다.

```c#
    //진동을 호출할 함수
    public static void PlayVibration(float duration, float fequency, float amplitude, Controller hand)
    {
        //PC 환경에선 기능이 없다.

#if Oculus
        if(CoroutineInstance.coroutineInstance == null)
        {
            GameObject coroutineObj = new GameObject("coroutineInstance");
            coroutineObj.AddComponent<CoroutineInstance>();
        }
        CoroutineInstance.coroutineInstance.StopAllCoroutines();
        CoroutineInstance.coroutineInstance.StartCoroutine(VibrationCoroutine(duration, fequency, amplitude, hand));
#endif
    }
```

세부적인 조정값이 없이 기본 디폴트 진동을 재생하는 함수를 오버로딩으로 만든다.

```c#
    public static void PlayVibration(Controller hand)
    {
        //PC 환경에선 기능이 없다.

#if Oculus
        PlayVibration(0.06f, 1, 1, hand);
#endif
    }
```

</br>

## 1.2.6 원하는 방향으로 중심을 재설정하는 Recenter()

</br>

OVRManager의 display 객체가 가지고 있는 RecenterPos() 함수가 이 기능을 제공한다.

```c#
    public static void Recenter()
    {
        //카메라가 바라보는 방향으로 월드 센터를 초기화 한다.
#if Oculus
        OVRManager.display.RecenterPose();
#endif
    }
```

</br>

## 1.2.7 DrawCrosshair

</br>

PC 환경에서는 OriginScale을 비교적 크게 세팅하였다. 

비슷한 느낌을 주기 위해서 OriginScale을 줄이면 된다. 

로직은 동일하고, 레이가 마우스 포인터가 아닌 컨트롤러에서 나간다는 것을 고려하면 된다.

```c#
#if PC
    static Vector3 originScale = Vector3.one * 0.02f;
#else
    static Vector3 originScale = Vector3.one * 0.005f;
#endif
    public static void DrawCrosshair(Transform crosshair, bool isHand =true, Controller hand = Controller.RTouch)
    {
        Ray ray;
        //컨트롤러의 위치와 방향을 이용해 레이 제작
        if (isHand)
        {
#if PC
            ray = Camera.main.ScreenPointToRay(Input.mousePosition);
#else
            if(hand == Controller.RTouch)
            {
                ray = new Ray(RHandPosition, RHandDirection);
            }
            else
            {
                ray = new Ray(LHandPosition, LHandDirection);
            }
#endif
 ...(생략)...
```


</br></br></br></br></br></br>