# 싱글턴으로 게임 매니저 구현하기
<details>
<summary>4.2 싱글턴 패턴 이해하기</summary>
<div markdown="1">       
  
* **싱글턴 패턴의 주요 목표는 유일성을 보장하는 것**
  * 클래스가 싱글턴 패턴을 제대로 구현했다면 초기화된 후에는 런타임 동안 메모리에 오직 하나의 인스턴스만 존재한다는 것을 의미함
  * 이 메커니즘은 전역적으로 접근할 수 있는 시스템을 관리하는 클래스가 있을 때 도움이 됨

* **싱글턴 디자인의 특징**
  * **오직 한 번만** 메모리 안에 생성되어야 함
  * 자기 자신과 같은 유형의 개체 인스턴스를 발견하면 **즉시 없앰**

* **싱글턴 패턴의 장단점**
  * 장점
    * 전역 접근 가능
    * 동시성 제어 : 공유 자원에 동시 접근을 제한하고자 사용할 수 있음

  * 단점
    * 유닛 테스트, 잘못된 습관이 생길 수 있다는 점 
</div>
</details>

___

<details>
<summary>4.3 게임 매니저 디자인하기</summary>
<div markdown="1">       

* 게임 매니저는 게임의 전체 수명 동안 살아있어야 한다는 점을 명심해야 함
* 메모리 내에서 유일하지만 지속적인 인스턴스가 되어야 함
</div>
</details>

___

<details>
<summary>4.4 게임 매니저 구현하기</summary>
<div markdown="1">       

* **Singleton 클래스 구현**
```C#
using UnityEngine;

namespace Chapter.Singleton
{
    public class Singleton<T> : MonoBehaviour where T: Component
    {
        private static T _instance;
        
        public static T Instance
        {
            get
            {
                if(_instance == null)
                {
                    _instance = FindObjectOfType<T>();

                    if(_instance == null)
                    {
                        GameObject obj = new GameObject();
                        obj.name = typeof(T).Name;
                        _instance = obj.AddComponent<T>();
                    }
                }
                return _instance;
            }
        }

        public virtual void Awake()
        {
            if (_instance == null)
            {
                _instance = this as T;
                DontDestroyOnLoad(gameObject);
            }
            else
            {
                Destroy(gameObject);
            }
        }
    }
}
```
* 엔진이 Awake() 함수를 호출했을 때 싱글턴 컴포넌트는 메모리에 초기화된 자신의 인스턴스가 이미 있는지 확인한다는 것을 반드시 알고 있어야 함
* 만약 인스턴스가 엇다면 싱글턴 컴포넌트는 자신이 현재 인스턴스가 됨
* 이미 있다면 복제를 막기 위해 **스스로를 제거함**
* So, 씬에는 **한 번에 하나의 특정한 싱글턴 인스턴스가 존재함**

* **GameManager 클래스 구현**
```C#
using System;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace Chapter.Singleton
{
    public class GameManager : Singleton<GameManager> //싱글턴으로 사용하고 싶을 때 필요!
    {
        private DateTime _sessionStartTime;
        private DateTime _sessionEndTime;

        void Start()
        {
            //TODO:
            // - 플레이어 세이브 로드
            // - 세이브가 없으면 플레이어를 등록 씬으로 리다이렉션한다.
            // - 백엔드를 호출하고 일일 챌린지와 보상을 얻는다.

            _sessionStartTime = DateTime.Now;
            Debug.Log("Game session start @: " + DateTime.Now);
        }
        void OnApplicationQuit()
        {
            _sessionEndTime = DateTime.Now;
            TimeSpan timeDifference = 
                -_sessionEndTime.Subtract(_sessionStartTime);

            Debug.Log("Game session ended @: " + DateTime.Now);
            Debug.Log("Game session lasted: " + timeDifference);
        }

        void OnGUI()
        {
            if(GUILayout.Button("Next Scene"))
            {
                SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
            }
        }
    }
}
```
  
</div>
</details>

___

<details>
<summary>4.5 정리</summary>
<div markdown="1">       

</div>
</details>
