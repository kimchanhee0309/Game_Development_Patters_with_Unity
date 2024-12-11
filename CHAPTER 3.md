# 유니티 프로그래밍 간략 설명


<details>
<summary>3.2 C# 기능</summary>
<div markdown="1">    

* **스태틱(static)**
  * 스태틱 키워드를 가진 메서드와 멤버는 인스턴스 초기화 없이도 이름으로 직접 접근할 수 있음
  * 스태틱 메서드와 멤버는 코드 어디서나 쉽게 접근할 수 있어 유용함
  * 예시 코드
```C#
using UnityEngine.Events;
using System.Collections.Generic;

namespace EventBus
{
  public class RaceEventBus
  {
    private static readonly IDictionary<RaceEventType, UnityEvent> Events = new Dictionary<RaceEventType, UnityEvent>();

    public static void Subscribe(RaceEventType eventType, UnityAction listener)
    {
      UnityEvent thisEvent;
      if (Events.TryGetValue(eventType, out thisEvent))
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

    public static void Unsubscribe(RaceEventType eventType, UnityAction listener)
    {
      UnityEvent thisEvent;
      if (Events.TryGetValue(eventType, out thisEvent))
      {
        thisEvent.RemoveListener(listener);
      }
    }

    public static void Publish(RaceEventType eventType)
    {
      UnityEvent thisEvent;
      if (Events.TryGetValue(eventType, out thieEvent))
      {
        thisEvent.Invoke();
      }
    }
  }
}
```

* **이벤트**
  * 게시자 역할을 하는 오브젝트가 다른 오브젝트가 수신할 신호의 발신을 허용함
  * 특정 이벤트를 수신하는 오브젝트를 **구독자** 라고 함
  * 예시 코드
```C#
using UnityEngine;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
  public delegate void TimerStarted();
  public static event TimerStarted OnTimerStarted;
  public delegate void TimerEnded();
  public static event TimerEnded OnTimerEnded;

  [SerializeField]
  private float duration = 5.0f;

  void Start()
  {
    StartCoroutine(StartCountdown());
  }

  private IEnumerator StartCountdown()
  {
    if (OnTimerStarted != null)
    {
      OnTimerStarted();
    }

    while (duration > 0)
    {
      yield return new WaitForSeconds(1f);
      duration--;
    }

    if (OnTimerEnded != null)
    {
      OnTimerEnded();
    }
  }

  void OnGUI()
  {
    GUI.color = Color.blue;
    GUI.Label(new Rect(125, 0, 100, 20), "COUNTDOWN: " + duration);
  }
}
```

* **델리게이트(delegate)**
  * 다른 함수의 메모리 주소를 가지는 함수 포인터를 의미함
  * 델리게이트를 함수 위치의 목록이 포함된 주소록으로 시각화할 수 있음 -> So, 여러 함수 포인터를 가지면서 한 번에 여러 개를 호출할 수 있는 이유임
  * 델리게이트는 주로 이벤트 핸들러와 콜백을 구현하는 데 사용할 때가 많음
  * 예시 코드
```C#
using UnityEngine;

public class Buzzer : MonoBehaviour
{
  void OnEnable()
  {
    //타이머 클래스에 선언한 델리게이트에 로컬 함수를 할당
    CountdownTimer.OnTimerStarted += PlayStartBuzzer;
    CountdownTimer.OnTimerEnded += PlayEndBuzzer;
  }

  void OnDisable()
  {
    CountdownTimer.OnTimerStarted -= PlayStartBuzzer;
    CountdownTimer.OnTimerEnded -= PlayEndBuzzer;
  }

  void PlayStartBuzzer()
  {
    Debug.Log("[BUZZER] : Play start buzzer!");
  }

  void PlayEndBuzzer()
  {
    Debug.Log("[BUZZER] : Play end buzzer!");
  }
}
```

* **제네릭(generic)**
  * 클라이언트가 클래스를 선언하고 인스턴스화할 때까지 타입 형식을 지연하는 것을 허용하는 C# 기능임
  * 클래스가 제네릭이라고 부를 떄는 정의된 오브젝트 타입을 가지지 않는다는 뜻임
  * 예시 코드
```C#
// <T>는 어떤 타입이라도 될 수 있음
public class Singleton<T> : MonoBehaviour where T : Component
{
  // ...
}
```

* **직렬화**
  * 오브젝트의 인스턴스를 바이너리 혹은 텍스트 형식으로 변환하는 과정
  * 오브젝트의 상태를 파일로 보존할 수 있음
  * 예시 코드
```C#
private void SerializePlayerData(PlayerData playerData)
{
  //PlayerData 인스턴스 직렬화
  BinaryFormatter bf = new BinaryFormatter();
  FileStream file = File.Create(Application.persistentDataPath + "/playerData.dat");
  bf.Serialize(file, playerData);
  file.Close();
}
```

</div>
</details>

___

<details>
<summary>3.3 유니티 엔진 기능</summary>
<div markdown="1">    

* **프리팹(prefab)**
  * 게임 오브젝트와 컴포넌트의 조합으로 만들어짐

* **ScriptableObject**
  * ScriptableObject를 상속하는 클래스임
  * MonoBehaviour를 상속한 클래스는 동작을 구현하고 ScriptableObject는 데이터 컨테이너 역할을 함
  * ScriptableObject의 인스턴스는 에셋으로 저장할 수 있으며, 종종 워크플로 작성에 사용함
  * 예시 코드
```C#
using UnityEngine

[CreateAssetMenu(fileName = "NewSword", menuName = "Weaponary/Sword")]
public class Sword : ScriptableObject
{
  public string swordName;
  public string swordPrefab;
}
```
* **코루틴(coroutine)**
  * 자신의 실행 과정을 대기, 시간 조절, 일시중지하는 추가 기능을 가진 함수
  * 예시 코드
```C#
using UnityEngine;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
  private float _duration = 10.0f;

  IEnumerator Start()
  {
    Debug.Log("Timer Started!");
    yield return StartCoroutine(WaitAndPrint(1.0f));
    Debug.Log("Timer Ended!");
  }

  IEnumerator WaitAndPrint(float waitTime)
  {
    while (Time.time < _duration)
    {
      yield return new WaitForSeconds(waitTime);
      Debug.Log("Seconds: " + Mathf.Round(Time.time));
    }
  }
}
```

</div>
</details>
