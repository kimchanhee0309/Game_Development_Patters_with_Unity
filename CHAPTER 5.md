# 상태 패턴으로 캐릭터 상태 관리하기

<details>
<summary>5.1 기술적 요구 사항</summary>
<div markdown="1">       

* Entity는 플레이어 입력이나 이벤트에 따라 하나의 상태에서 다른 상태로 끊임없이 전환함
  * 5장에서는 Entity의 개별 상태와 상태 동작을 정의할 수 있는 패턴을 살펴볼 예정임
 
* 캐릭터의 유한 상태를 관리하고자 상태 패턴을 사용함
  * But, 다양한 기계적 동작과 애니메이션을 상태 패턴으로 구현하다보면 상태 패턴의 한계를 경험하게 됨
  * 이것을 **유한 상태 기계(finite state machine, FSM)** 개념을 도입해 해결  
</div>
</details>

___

<details>
<summary>5.2 상태 패턴의 개요</summary>
<div markdown="1">       

* 상태 패턴으로 객체가 내부 상태를 기반으로 움직일 수 있도록 하는 시스템을 구현
  * 상태의 변화가 동작의 변화를 가져오게 됨
 
* 상태 패턴 구조의 3가지 핵심 요소
  * **Context 클래스**
    * 클라이언트가 객체의 내부 상태를 변경할 수 있도록 요청하는 인터페이스를 정의함, 현재 상태에 대한 포인터 보유
  * **IState 인터페이스**
    * 구체적인 상태 클래스로 연결할 수 있도록 설정
  * **ConcreteState 클래스**
    * IState 인터페이스를 구현하고 Context 오브젝트가 상태의 동작을 트리거하기 위해 호출하는 퍼블릭 메서드 handle()를 노출함
   
* 클라이언트는 객체의 상태를 업데이트하고자 Context 객체를 활용해 원하는 상태로 설정하거나 새로운 상태로 전환할 것을 요청함
  * 즉, Context는 언제나 관리하는 객체의 현재 상태를 인식       
</div>
</details>

___

<details>
<summary>5.3 캐릭터 상태 정의하기</summary>
<div markdown="1">       

* 플레이어는 같은 상태를 오래 지속하지 않음
* 상태를 정의할 때는 행동에 대한 설명과 애니메이션이 포함되어야 함
* 이러한 요구 사항은 상태 패턴을 구현하고 검토할 때 반드시 고려해야 
</div>
</details>

___

<details>
<summary>5.4 상태 패턴 구현하기</summary>
<div markdown="1">       

* 예상할 수 있는 동작을 캡슐화한다는 목표로 상태 패턴을 구현함
  * 간결함과 명확성을 위해 최소한의 스켈레톤 코드를 작성함
* 상태 패턴 구현하기
  * 구체적인 상태 클래스가 구현할 **기본 인터페이스** 작성
```C#
namespace Chapter.State
{
  public interface IBikeState
  {
    void Handle (BikeController controller);
  }
}

//Handle() 메서드에 BikeController 인스턴스를 전달한다는 점 유의!
//상태 클래스가 BikeController의 public 속석에 접근할 수 있도록 함
```
    
  * **context class** 구현
```C#
namespace Chapter.State
{
  public class BikeStateContext
  {
    public IBikeState CurrentState
    {
      get; set;
    }
    private readonly BikeController _bikeController;

    public BikeStateContext(BikeController bikeController)
    {
      _bikeController = bikeController;
    }

    public void Transition()
    {
      CurrentState.Handle(_bikeController);
    }

    public void Transition(IBikeState state)
    {
      CurrentState = state;
      CurrentState.Handle(_bikeController);
    }
  }
}

//오토바이의 현재 상태를 가리키는 public 속성을 노출하여 모든 상태 변경을 인식함
//개체 속성을 통해 현재 상태를 업데이트하고 Transition() 함수를 호출하여 업데이트한 상태로 전환할 수 있음
```
 
  * **BikeController class** 구현(Context 오브젝트와 상태 초기화, 상태 변경 트리거)
```C#
using UnityEngine;

namespace Chapter.State
{
  public class BikeController : MonoBehaviour
  {
    public float maxSpeed = 2.0f;
    public float turnDistance = 2.0f;

    public float CurrentSpeed { get; set; }

    public Direction CurrentTurnDirection {
      get; private set;
    }

    private IBikeState _startState, _stopState, _turnState;

    private BikeStateContext _bikeStateContext;

    private void Start() 
    {
      _bikeStateContext = new BikeStateContext(this);

      _startState = gameObject.Addcomponent<BikeStartState>();
      _stopState = gameObject.AddComponent<BikeStopState>();
      _turnState = gameObject.AddComponent<BikeTurnState>();

      _bikeStateContext.Transition(_stopState);
    }

    public void StartBike()
    {
      _bikeStateContext.Transition(_startState);
    }

    public void StopBike()
    {
      _bikeStateContext.Transition(_stopState);
    }

    public void Turn(Direction direction)
    {
      CurrentTurnDirection = direction;
      _bikeStateContext.Transition(_turnState);
    }
  }
}

//개별적인 상태 클래스 내부의 오토바이 동작을 캡슐화하지 않았다면 BikeController 내부에 구현했을 것 
//그렇게 되면 유지 및 관리가 힘든 장황한 controller 클래스를 사용해야 함
//상태 패턴은 클래스를 간소화하고 유지 및 관리가 쉽도록 만든다.
//그리고 오토바이의 핵심 컴포넌트를 관리하는 책임을 BikeController에 돌려줌
//이는 오토바이를 제어하고 설정할 수 있는 속성을 토출하고 구조적 종속성을 관리하기 위한 인터페이스를 
  제공하기 위함
```
    
  * 3가지 상태 클래스(각 클래스는 IBikeState 인터페이스를 구현한다는 점 명심)
```C#
//BikeStopState 클래스

using UnityEngine;

namespace Chapter.State
{
  public class BikeStopState : MonoBehaviour, IBikeState
  {
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
      if(!_bikeController)
        _bikeController = bikeController;

      _bikeController.CurrentSpeed = 0;
    }
  }
}
```
```C#
//BikeStartState 클래스

using UnityEngine;

namespace Chapter.State
{
  public class BikeStartState : MonoBehaviour, IBikeState
  {
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
      if(!_bikeController)
        _bikeController = bikeController;

      _bikeController.CurrentSpeed = _bikeController.maxSpeed;
    }

    void Update()
    {
      if(_bikeController)
      {
        if(_bikeController.CurrentSpeed > 0)
        {
          _bikeController.transform.Translate(
            Vector3.forward * (_bikeControlled.CurrentSpeed * Time.deltaTime));
        }
      }
    }
  }
}
```
```C#
//BikeTurnState 클래스

using UnityEngine;

namespace Chapter.State
{
  public class BikeTurnState : MonoBehaviour, IBikeState
  {
    private Vector3 _turnDirection;
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
      if(!_bikeController)
        _bikeController = bikeController;

      _turnDirection.x = 
        (float) _bikeController.CurrentTurnDirection;

      if(_bikeController.CurrentSpeed > 0)
      {
        transform.Translate(_turnDirection * _bikeController.turnDistance);
      }
    }
  }
}
```
    
  * **Direction** 구현
```C#
namespace Chapter.State
{
  public enum Direction
  {
    Left = -1,
    Right = 1
  }
}
```
 
  * **ClientState 클래스** 구현
```C#
using UnityEngine;

namespace Chapter.State
{
  public class ClientState : MonoBehaviour
  {
    private BikeController _bikeController;

    void Start()
    {
      _bikeController = (BikeController) FindObjectOfType(typeof(BikeController));
    }

    void OnGUI()
    {
      if(GUILayout.Button("Start Bike"))
        _bikeController.StartBike();
      if(GUILayout.Button("Turn Bike"))
        _bikeController.Turn(Direction.Left);
      if(GUILayout.Button("Turn Right"))
        _bikeController.Turn(Direction.Right);
      if(GUILayout.Button("Stop Bike"))
        _bikeController.StopBike();
    }
  }
}
```
</div>
</details>

___

<details>
<summary>5.5 상태 패턴의 장단점</summary>
<div markdown="1">       

* 장점
  * **캡슐화**
    * 상태 패턴은 상태가 변할 때 개체에 동적으로 할당할 수 있는 컴포넌트의 집합, 개체의 상태별 행동을 구현할 수 있음
  * **유지 및 관리**
    * 긴 조건문이나 방대해진 클래스의 수정 없이도 쉽게 새로운 상태를 구현할 수 있음

* 단점
  * **블렌딩**
    * 기본 형태에서 상태 패턴은 애니메이션 블렌드 방법을 제공하지 않음
  * **전환**
    * 관계와 조건에 따라 상태 간의 전환 transition을 정의하고 싶다면 더 많은 코드를 작성해야      
</div>
</details>
