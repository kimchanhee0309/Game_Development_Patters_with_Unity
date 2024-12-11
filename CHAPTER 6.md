# 이벤트 버스로 게임 이벤트 관리하기

<details>
<summary>이벤트 버스 패턴이란?</summary>
<div markdown="1">    

* **이벤트 버스(event bus)**
  * 객체가 구독하거나 게시할 수 있는 특정한 전역 이벤트의 목록을 관리하는 중앙 허브 역할을 함
  * 빠른 결과가 필요할 때 유용함
</div>
</details>

___

<details>
<summary>6.1 기술적 요구 사항</summary>
<div markdown="1">    
  
* 스태틱
* UnityEvents
* UnityActions
</div>
</details>

___

<details>
<summary>6.2 이벤트 버스 패턴 이해하기</summary>
<div markdown="1">    

### 개요
* 객체(게시자)가 이벤트를 발생하게 하면 다른 객체(구독자)가 받을 수 있는 신호를 보냄
* 신호는 작업이 생겼다는 것을 알리는 알림 형식임
* 오브젝트는 이벤트 시스템에서 이벤트를 브로드캐스트함
* 구독하는 오브젝트만 알림을 받으며 어떻게 처리할지 결정함
* **이벤트 버스 패턴(event bus pattern)** : 메시징 시스템과 발행/구독 패턴과 가까운 관계임
  * 컴퓨터 용어인 버스는 구성 요소 간 연결을 의미함(이벤트 버스는 게시자와 구독자 사이의 중개자 역할을 함)
* 이벤트에서 발행/구독 모델을 사용하여 오브젝트를 연결하는 방법을 이벤트 버스라고 함
* 장점
  * 단 한 줄의 코드로 게시자나 구독자 역할을 할당하는 과정을 줄일 수 있다는 점
* 세 가지 주요 구성 요소
  * **게시자(publisher)** : 이벤트 버스에서 선언한 특정 종류의 이벤트를 구독자에게 게시할 수 있음
  * **이벤트 버스(event bus)** : 구독자-게시자 사이의 이벤트 전송을 조정하는 역할을 함
  * **구독자(subscriber)** : 이벤트 버스를 통해 특정 이벤트의 구독자로 자신을 등록함
 
### 6.2.1 이벤트 버스 패턴의 장단점
* 장점
  * **분리** : 오브젝트를 분리함. 오브젝트는 직접 서로를 참조하는 대신 이벤트로 통신할 수 있음
  * **단순성** : 이벤트의 구독 혹은 게시 메커니즘을 추상화하여 단순성을 제공함
 
* 단점
  * **성능** : 모든 이벤트 시스템의 내부에는 오브젝트 간 메시지를 관리하는 저수준 메커니즘이 있음. So, 이벤트 시스템을 사용할 때 약간의 성능 비용이 발생할 수 있음
  * **전역(global)**
    * static 메서드와 변수로 이벤트 버스를 구현하여 코드 어디서나 쉽게 접근할 수 있도록 함
    * 전역적 접근으로 디버깅과 유닛 테스트를 어렵게 해 프로젝트 관리 또한 어렵게 만듦.

### 6.2.2 이벤트 버스를 사용하는 시기
* **빠른 프로토타이핑**
* **프로덕션 코드**

### 6.2.3 전역 레이스 이벤트 관리하기
```C#
namespace EventBus
{
  public enum RaceEventType
  {
    COUNTDOWN, START, RESTART, PAUSE, STOP, FINISH, QUIT
  }
}
```
* 초반 열거형 값은 간략하게 레이스의 첫 단계부터 마지막 단계까지 설명하는 특정 이벤트임.
* 전역 범위로 이벤트를 처리하도록 제한함

```C#
using UnityEngine.Events;
using System.Collections.Generic;

namespace RaceEventBus
{
  private static readonly IDictionary<RaceEventType, UnityEvent>
  Events = new Dictionary<RaceEventType, UnityEvent>();

  public static void Subscribe(RaceEventType eventType, UnityAction listener)
  {
    UnityEvent thisEvent;

    if (Events.TryGetalue(eventType, out thisEvent))
    {
      thisEvent.AddListener(listener);
    }
    else
    {
      thisEvent = new UnityEvent();
      thisEvent.AddListener(listener);
      Events.Add(eventType, thisEvent);
    }
  }

  public static void Unsubscribe(RaceEventType type, UnityAction listener)
  {
    UnityEvent thisEvent;

    if (Events.TryGetValue(type, out thisEvent))
    {
      thisEvent.RemoveListener(listener);
    }
  }

  public static void Publish(RaceEventType type)
  {
    UnityEvent thisEvent;

    if (Events.TryGetValue(type, out thisEvent))
    {
      thisEvent.Invoke();
    }
  }
}
```

</div>
</details>

___

<details>
<summary>6.3 레이스 이벤트 버스 구현하기</summary>
<div markdown="1">    

</div>
</details>

___

<details>
<summary>6.4 대안 살펴보기</summary>
<div markdown="1">    

</div>
</details>

___

<details>
<summary>6.5 정리</summary>
<div markdown="1">    

</div>
</details>
