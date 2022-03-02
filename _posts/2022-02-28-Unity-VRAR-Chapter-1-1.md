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

# 1.1.3 CamRotate.cs 스크립트 작성

</br>


</br>

>목표

    마우스 입력에 따라 카메라 회전

>순서

    1. 마우스 입력을 얻어 와야 한다.
    2. 방향이 필요하다.
    3. 회전을 시킨다.


</br>

하고자하는 최종 행위를 하기위한 순서를 역으로 구성한다.

회전을 시키고 싶다 -> 회전하려면 방향이 필요하다 -> 방향은 마우스 입력으로 정한다.

</br>

## 1.1.3.1 CamRotate를 만들기 위한 사전 지식

</br>

카메라를 마우스 포인터 입력에 따라 회전을 시키고자 한다면

마우스 포인터의 포지션은 2D 픽셀이기 때문에 

x입력은 좌우 회전이고, y입력은 상하 회전일 것이다.

그런데 좌우 회전은 메인카메라의 y축 기준 Rotate고,

상하 회전은 메인카메라의 x축 기준 회전이다. 

마우스 입력을 받는 방법을 알아본다.
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class CamRotate : MonoBehavior
{
    //회전 속도 변수
    public float rotSpeed = 200f;

    void Update()
    {
        //마우스 입력을 받는 방법
        float mouse_X = Input.GetAxis("Mouse X");
        float mouse_Y = Input.GetAxis("Mouse Y");

        //y 입력으로 x축을 회전시키되 y가 양수 입력이면 x축을 -방향으로 회전하여야함.
        //x 입력으로 y축을 회전시킬때에는 그냥 시계방향과 동일한 회전이 나간다.
        Vector3 dir = new Vector3(- mouse_Y, mouse_X, 0);

        //오일러 앵글에 Vector3를 누적시킨다.
        transform.eulerAngles += dir *rotSpeed * Time.deltaTime;

        //y축 회전은 360도 회전해도 문제 없지만, x축 회전은 -90~ 90도 사이로 막아놔야 한다.
        Vector3 rot = transform.eulerAngles;
        rot.x = Mathf.Clamp(rot.x, -90f, 90f);
        transform.eulerAngles = rot;
        //하지만 문제가 발생하게 된다. 이유는 eulerAngles가 반환하는 Vector3를 저장하는
        //rot의 x값을 Clamp로 초기화하는 문제이다.
    }
}
```

</br>

저렇게 코드를 짜게 될 경우 문제가 발생한다. 정면을 바라볼때 자꾸 90f 값으로 각도가 설정되는 문제이다.

이는 Transform.eulerAngles 의 연산 방식때문이다. 

transform의 rotation은 -360도든 -900도든 float값이면 각 xyz 값을 가질 수 있는데

내부적으로 eulerAngles가 연산하는 방식은 0~360도를 취급하기 때문이다. 따라서

-1도는 359도로 표현이 되고, 아주 조금만 올려서 - float x축 회전을 하게되면 

359도로 연산되는 eulerAngles를 가지고 rot에 저장하고, 

그 rot는 Mathf.Clamp()가 90f로 초기화를 하게 되니 결과적으로 

transform.eulerAngles엔 90도 회전이 적용되는 것이다.

이를 방지하는 코드는 마우스입력으로 회전을 만들고나서 범위를 제한하는 방식이 아니라,

애초에 마우스 입력이 발생할때 들어가는 float값을 

얼마까지만 누적시킬지를 선제한하는 방식으로 막는다.

</br>

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class CamRotate : MonoBehavior
{
    //회전 속도 변수
    public float rotSpeed = 200f;

    //transform.eulerAngles에 전달할 Vector3의 float값
    float mx = 0;
    float my= 0;

    void Update()
    {
        //마우스 입력을 받는 방법
        float mouse_X = Input.GetAxis("Mouse X");
        float mouse_Y = Input.GetAxis("Mouse Y");

        //x축 회전의 정도값인 float 자체를 Clamp로 제한할 것이다.
        //회전의 정도값은 마우스 입력값을 누적시킬것이다.
        mx = mouse_X * rotSpeed * Time.deltaTime;
        my = mouse_Y * rotSpeed * Time.deltaTime;

        my = Mathf.Clamp(my, -90f, 90f);

        //상하 입력이 x축을 회전시키고, 좌우입력이 y축을 회전시킨다.
        transform.eulerAngles = new Vector3(-my, mx, 0);
    }
}

```

## 1.1.3.2 VR에서도 적용할 CamRotate.cs

</br>

위에서 알아본 CamRotate에서 전부 배웠다. 하지만 고려하지 않은게 있다. 

일반 3D 게임은 초기 카메라 위치가 저장되어 시작되지만, VR은 그렇지 않기 때문이다. 

이를 고려한 코드를 짜본다.

</br>

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// 마우스 입력에 따라 카메라를 회전시키고 싶다.
// 필요 속성: 현재 각도, 마우스 감도
public class CamRotate : MonoBehaviour
{
    // 좌우 입력을 누적시킬 Vector3 자료형
    Vector3 angle;
    // 마우스 감도
    public float sensitivity = 200;

    void Start()
    {
        // 시작할 때 현재 카메라의 각도를 적용한다.
        // 3D 게임과 다른점이다. 그리고 x축 회전을 제한하지도 않을 것이다.
        // 마우스 입력은 실제 무한히 360도로 가능하지만, VR장비를 끼면
        // 불가능 할 것이기 때문이다.

        //Vector3 자료형에 현재 카메라의 eulerAngles 요소값을 저장하여 초기화
        //angle.y값에 -x값을 담는 이유는 angle.y에 누적시킨 float 값을 나중에
        //x축 회전값에 대입할때 -를 곱해서 대입할 것이기 때문에 초기화 단계에서 
        //-를 미리 곱해서 대입하는 것이다.
        angle.y = -Camera.main.transform.eulerAngles.x;
        angle.x = Camera.main.transform.eulerAngles.y;
        angle.z = Camera.main.transform.eulerAngles.z;
    }

    void Update()
    {
        // 마우스 입력에 따라 카메라를 회전시키고 싶다.
        // 1. 사용자의 마우스 입력을 얻어와야 한다.
        // 마우스의 좌우 입력을 받는다.
        float x = Input.GetAxis("Mouse X");
        float y = Input.GetAxis("Mouse Y");

        // 2. 방향이 필요하다.
        // 이동 공식에 대입해 각 속성별로 회전 값을 누적시킨다.
        //Vector3 angle.x에 x입력값을 누적시키고
        //angle.y에 y 입력값을 누적 시킨다.
        angle.x += x * sensitivity * Time.deltaTime;
        angle.y += y * sensitivity * Time.deltaTime;

        // 3. 회전시키고 싶다.
        // 카메라의 회전 값에 새로 만들어진 회전 값을 할당한다.
        transform.eulerAngles = new Vector3(-angle.y, angle.x, transform.eulerAngles.z);
    }
}
```

</br>

# 1.1.5 복셀 만들기

</br>

복셀이란 큐브만으로 디자인된 물체를 의미. 마인크래프트를 생각하면된다.

2D에선 도트 디자인이라고 불리는 것이 3D에선 복셀이라고 부른다 생각하면 된다.

VR에서는 성능을 많이 잡아 먹기 때문에 최적화를 위해 잘 짜여진 

로우 폴리 아트를 활용하는 것이 유리하다.

</br>

>목표
    
    복셀을 생성해 랜덤한 방향으로 튀게 한다.

> 순서

    1. 복셀 게임 오브젝트 생성
    2. 복셀 스크립트를 작성
    
</br>

## 1.1.5.2 복셀 스크립트 작성

1. 복셀은 랜덤한 방향으로 날아가는 운동을 한다.
2. 일정 시간이 지나면 복셀을 제거한다.
3. 메모리 파편화 및 로딩 시간 등을 관리하기 위해 오브젝트 풀을 사용해 복셀을 관리한다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// 1. 복셀은 랜덤한 방향으로 날아가는 운동을 한다.
// 필요 속성: 날아갈 속도
// 2. 일정 시간이 지나면 복셀을 제거하고 싶다.
// 필요 속성: 복셀을 제거할 시간, 경과 시간

public class Voxel : MonoBehaviour
{
    // 1. 복셀이 날아갈 속도 구하기
    public float speed = 5;
    // 복셀을 제거할 시간
    public float destoryTime = 3.0f;
    // 경과 시간
    float currentTime;

    void Start()
    {
        //현재 시간 초기화
        currentTime = 0;
        // 2. 랜덤한 방향을 찾는다.
        //Random.insideUnitSphere 반경 1을 갖는 구 안의 임의의 지점을 반환합니다.
        //이 vector3를 고스란히 방향벡터로 사용할 것이다.
        Vector3 direction = Random.insideUnitSphere;
        // 3. 랜덤한 방향으로 날아가는 속도를 준다.
        Rigidbody rb = gameObject.GetComponent<Rigidbody>();
        rb.velocity = direction * speed;
    }

    void Update()
    {
        // 일정 시간이 지나면 복셀을 제거하고 싶다.
        // 1. 시간이 흘러야 한다.
        currentTime += Time.deltaTime;
        // 2. 제거 시간이 됐으니까.
        // 만약 경과 시간이 제거 시간을 초과했다면
        if (currentTime > destoryTime)
        {
            Destroy(gameObject);
        }
    }
}
```

</br>

# 1.1.6 복셀 제작자 만들기

</br>

위에서 랜덤 방향으로 속도를 유지하다가 파괴되는 복셀을 만들었다면,

이제는 그 복셀을 소환하는 오브젝트를 만들 것이다.

</br>

> 목표

    사용자 입력이 닿는 곳에 복셀을 제작

> 순서

    1. 복셀 제작자 만들기
    2. 마우스 포인터가 닿는 곳에 복셀을 만들기
    3. 시선이 닿는 곳에 복셀을 만들기

</br>

## 1.1.6.2 마우스 포인터가 닿는 곳에 복셀을 만들기

</br>

복셀공장(프리팹) 만들기 -> 마우스 위치가 바닥을 향한 뒤에 -> 생성입력을 한다면 -> 생성



</br>

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
// 사용자가 마우스를 클릭한 지점에 복셀을 1개 만들고 싶다.
// 필요 속성: 복셀 공장
public class VoxelMaker : MonoBehaviour
{
    // 복셀 공장
    public GameObject voxelFactory;
    // 오브젝트 풀의 크기
    public int voxelPoolSize = 20;
    // 오브젝트 풀
    public static List<GameObject> voxelPool = new List<GameObject>();
    // 생성 시간
    public float createTime = 0.1f;
    // 경과 시간
    float currentTime = 0;
    // 크로스헤어 변수
    public Transform crosshair;

    void Start()
    {
        // 오브젝트 풀에 비활성화된 복셀을 담고 싶다.
        for (int i = 0; i < voxelPoolSize; i++)
        {
            // 1. 복셀 공장에서 복셀 생성하기
            GameObject voxel = Instantiate(voxelFactory);
            // 2. 복셀 비활성화하기
            voxel.SetActive(false);
            // 3. 복셀을 오브젝트 풀에 담고 싶다.
            voxelPool.Add(voxel);
        }
    }

    void Update()
    {
        // 크로스헤어 그리기
        ARAVRInput.DrawCrosshair(crosshair);

        // 1) 마우스 클릭이 들어온다면 Fire1은 ctrl이나 왼쪽 클릭에 반응한다.
        if (Input.GetButtonDown("Fire1"))
        {
            //2. 마우스가 바닥 위에 위치해 있다면
            // 바닥 위에 위치해 있는지는 Ray에 닿는 것이 무엇인지를 계산할 필요가 있다.
            //ray를 생성하고 ray가 맞춘것이 바닥인지를 검사한다.
            Ray ray = camera.main.ScreenPointToRay()

            // 일정 시간마다 복셀을 만들고 싶다.
            // 1. 경과 시간이 흐른다.
            currentTime += Time.deltaTime;

            // 2. 경과 시간이 생성 시간을 초과했다면
            if (currentTime > createTime)
            {
                // Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
                // 2) 컨트롤러가 향하는 방향으로 시선 만들기
                Ray ray = new Ray(ARAVRInput.RHandPosition, ARAVRInput.RHandDirection);
                RaycastHit hitInfo = new RaycastHit();

                // 2. 마우스의 위치가 바닥 위에 위치해 있다면
                if (Physics.Raycast(ray, out hitInfo))
                {
                    // 복셀 오브젝트 풀 이용하기
                    // 1. 만약 오브젝트 풀에 복셀이 있다면
                    if (voxelPool.Count > 0)
                    {
                        // 복셀을 생성했을 때만 경과 시간을 초기화해준다.
                        currentTime = 0;
                        // 2. 오브젝트 풀에서 복셀을 하나 가져온다.
                        GameObject voxel = voxelPool[0];
                        // 3. 복셀을 활성화한다.
                        voxel.SetActive(true);
                        // 4. 복셀을 배치하고 싶다.
                        voxel.transform.position = hitInfo.point;
                        // 5. 오브젝트 풀에서 복셀을 제거한다.
                        voxelPool.RemoveAt(0);
                    }
                }
            }
        }
    }
}

```

</br></br></br></br></br></br></br></br>