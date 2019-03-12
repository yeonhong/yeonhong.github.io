---
layout: single
title: "ReactiveX and Unity3D: Part 2"
related: true
categories: 
  - 프로그래밍
tags:
  - Reactive Programming
  - UniRX
  - Unity
link: "https://ornithoptergames.com/reactivex-and-unity3d-part-2/"
---

## My little runaway
"Shift 키를 누른 상태로 실행"은 1 인칭 게임에서 실제 표준 연습이므로이 기능을 지원하고자합니다. 물론 Observables를 사용하여이를 수행 할 수있는 많은 방법이 있지만 Reactive Properties를 소개 할 수있는 좋은 기회라고 생각합니다. 이 기능은 ReactiveX가 아닌 UniRx에 추가 된 특별한 기능입니다. Reactive Properties는 우리에게 두 가지 장점을 모두 제공합니다. 평범한 속성의 유연성과 Observables의 힘입니다. 속성과 마찬가지로 리 액티브 속성에서 값을 설정하고 가져올 수 있지만 업데이트 알림을 받으려면 등록 할 수도 있습니다. 실행 입력은 사용자가 키를 눌렀거나 놓을 때 정확하게 반응 할 필요가 없기 때문에이를위한 훌륭한 후보입니다. 운동을 계산하는 동안 키가 눌러 졌는지 알고 싶을뿐입니다. 이 코드를 입력 스크립트에 추가해 보겠습니다 (이미 살펴본 내용은 생략합니다).

```
public ReadOnlyReactiveProperty<bool> Run { get; private set; }
// ...

private void Awake() {
  // ...
  Run = this.UpdateAsObservable()
    .Select(_ => Input.GetButton("Fire3"))
    .ToReadOnlyReactiveProperty();
}
```
먼저, ReadOnlyReactiveProperty를 사용하고 있습니다. 일반 ReactiveProperty를 사용하면 액세스 할 수있는 누구나 해당 값을 설정할 수 있습니다. 가능할 때마다 쓰기 권한을 제한하는 것이 가장 좋습니다. 코드의 연결을 해제하는 것입니다. 어쨌든, Run Observable에서 알아서 값을 설정합니다. 그리고 그것이 Awake에서하는 것입니다 : 모든 업데이트, 버튼 "Fire3"의 상태를 선택하고이를 우리의 자산으로 변환하십시오. ( "Fire3"은 새로운 Unity 프로젝트에서 기본적으로 정의 된 입력 축이며 Shift 키에 편리하게 매핑됩니다.)

우리의 새로운 입력을 사용하는 것은 간단합니다. RunSpeed 필드를 PlayerController에 추가 한 후 적절한 속도를 선택하기 위해 움직임을 계산할 때마다 Run의 값을 폴링합니다.

```
// In PlayerController.Start()
inputs.Movement
  .Where(v => v != Vector2.zero)
  .Subscribe(inputMovement => {
    var inputVelocity = inputMovement * (inputs.Run.Value ? runSpeed : walkSpeed);
    // ... etc.
```
이것은 아마도이 기능을 구현하는 가장 간단한 방법 일 것입니다. 그러나 완벽합니까? 옵서버 블 (Observables) (또는 비동기 코드)을 사용하는 데 드는 대가는 여기에 약간의 미묘함이 있습니다. **기본적으로 신호를 서로 종속적으로 만듦을 제외하고는 기본적으로 실행 순서를 보장 할 수 없습니다.** 다른 말로하면 : 나는 그 값에 접근 할 때 Run이 업데이트되었음을 확실히 알지 못할 수도 있습니다. 이 코드를 사용하면 이동 계산의 관점에서 Run의 값이 한 프레임 뒤 떨어질 수 있습니다. 적절한 실행 순서를 보장 할 수있는 방법이 있습니다 (3 부에서 살펴 보겠습니다). 그러나 코드가 조금 복잡해집니다. 비용이 그만한 가치가 있는지 자문 해보십시오. 우리가 한 프레임 늦게 시작하는게 중요할까요? 아마도 그렇지 않을 수도 있습니다.

## Bob and un-weave
camera bob 구현은 좀 더 복잡해 지겠지만 Observables가 코드를 잘 분해 할 수 있다는 약속이 몇 가지 있습니다. 우리는 걷기를 에뮬레이션하기 위해 카메라를 약간 위아래로 돌려 놓기를 원하기 때문에 플레이어가 각 프레임을 걷는 거리를 알아야합니다. Standard Assets에서는 플레이어 컨트롤러가 camera bob 효과를 담당하는 클래스를 직접 업데이트하도록함으로써 구현됩니다. 당연히 이는 플레이어 컨트롤러 코드를 카메라 밥 코드에 연결합니다. 대신 Observables를 사용하여이 두 클래스 간의 인터페이스를 구성합니다. 그리고 실제 인터페이스로 시작합니다. (실제로는 추상 클래스입니다.하지만 Unity의 인스펙터와의 호환성을 위해서입니다.)

```
public abstract class PlayerSignals : MonoBehaviour {
  public abstract float StrideLength { get; }
  public abstract IObservable<Vector3> Walked { get; }
}
```
PlayerController는이 인터페이스를 확장하고 구현하므로 camera bob 스크립트는 직접적인 종속성을 필요로하지 않습니다. StrideLength는 간단한 구성 가능 필드이지만 Walked Observable은 어떻게 생성합니까? (Subject로 생성하기.)

유니티의 CharacterController 컴포넌트는 실제로 (이 벽 충돌 등을 차지 한 후 이동 한 실제 거리이기 때문에) 우리를 위해 이 값을 계산해줍니다.

```
inputs.Movement
  .Where(v => v != Vector2.zero)
  .Subscribe(inputMovement => {
    // ...
    var distance = playerVelocity * Time.fixedDeltaTime;
    character.Move(distance);
    var distanceActuallyWalked = character.velocity * Time.fixedDeltaTime;
}).AddTo(this);
```
Observable에 distanceActuallyWalked를 넣고 싶습니다. 하지만 이미 외부로부터 Observable에 가치를 주입 할 수는 없다고 말했죠?

하나의 옵션은 또 다른 개념, Subject를 소개합니다. ReactiveX의 Subject는 Observer 및 Observable 인터페이스를 결합하지만 본질적으로 읽기 / 쓰기 Observable이라고 생각할 수 있습니다.  Reactive Property과 다른 점은 무엇입니까? Reactive Property가하는 것처럼 주제에는 "현재 값"의 개념이 없습니다. Subject가 변경되면 알림을 받을 수 있지만 현재 값을 요청할 수는 없습니다. 따라서 주제는 Reactive Property보다 조금 덜 강력하다고 생각할 수 있으며 어떤 상황에서도 작동 할 수있는 가장 강력하지 않은 옵션을 선호해야합니다. 실제로 신호를 의도적으로 사용하는 방법에 따라 이 결정을 내릴 수 있습니다.

그래서 우리는 필드로 PlayerController 스크립트에 Subject <Vector3>를 추가합니다. 이것이 "우리"신호이기 때문에, 우리는 Awake에서 그것을 초기화 할 필요가있다.

```
walked = new Subject<Vector3>().AddTo(this);
```

이것은 새로운 신호이기 때문에 우리는 AddTo (this)를 통해 게임 객체에 생명주기를 제한해야합니다.

그리고 이제 우리는 위의 distanceActuallyWalked 행을 이것을 다음과 같이 대체 할 수 있습니다 :
```
walked.OnNext(character.velocity * Time.fixedDeltaTime);
```

OnNext메서드는 신호에 새 값을 제공하는 데 사용됩니다. 신호에 가입 한 모든 사람은 새 값으로 알림을 받게됩니다.

우리는 아직 camera bob 부분에 도달하지 못했지만, 우리는 놀라운 것을 해냈다고 생각합니다. 우리의 스크립트는 신호를 "소비"할뿐만 아니라 신호도 생성합니다. 이것은 Unity의 물리학 및 장면 그래프와 같이 Observables를 기반으로하지 않는 시스템을 통합 할 수있게 해주는 근본적인 접착제입니다.

지금까지 나는 사물에 값을 쓸 수있는 사람을 제한하는 데 매우 신중했습니다. Subject를 공개하면 누구나 잠재적으로 값을 쓰고 코드를 위반할 수 있습니다. 걱정하지 마세요. Subject는 Observable이므로, 다음과 같이 정의하여 공개 필드를 제한하기 만하면됩니다.

```
private Subject<Vector3> walked;
public override IObservable<Vector3> Walked {
  get { return walked;  }
}
```
이제 Subject가 표시되고 다른 모든 사람들은 IObservable을 봅니다. 멋지고 깨끗합니다!

좋아, 실제 camera bob 스크립트에! 이 스크립트는 카메라 게임 객체에 앉아 동작을 제어합니다. Walked 신호를 구독하고 플레이어의 보폭을 모듈로 거리로 누적합니다. 이를 사용하여 Unity AnimationCurve로 정의 된 사인 커브를 평가합니다. 그러면 카메라의 현재 위치를 최종적으로 조정할 수 있습니다.

```
public class CameraBob : MonoBehaviour {
  // IPlayerSignals reference configured in the Unity Inspector, since we can
  // reasonably expect these game objects to be in the same hierarchy
  public PlayerSignals player;
  public float walkBobMagnitude = 0.05f;
  public float runBobMagnitude = 0.10f;

  public AnimationCurve bob;

  private Camera view;
  private Vector3 initialPosition;

  private void Awake() {
    view = GetComponent<Camera>();
    initialPosition = view.transform.localPosition;
  }

  private void Start() {
    var distance = 0f;
    player.Walked.Subscribe(w => {
      // Accumulate distance walked (modulo stride length).
      distance += w.magnitude;
      distance %= player.StrideLength;
      // Use distance to evaluate the bob curve.
      var magnitude = InputsV2.Instance.Run.Value ? runBobMagnitude : walkBobMagnitude;
      var deltaPos = magnitude * bob.Evaluate(distance / player.StrideLength) * Vector3.up;
      // Adjust camera position.
      view.transform.localPosition = initialPosition + deltaPos;
    }).AddTo(this);
  }
}
```
distance 변수는 구독에 필요한 상태입니다. 여기에서 우리는 폐쇄를 통해 그 상태를 통합했습니다. 이전에 우리는 클래스 필드 (예를 들면, CharacterController 인스턴스)를 통해 상태를 통합했습니다.

인식 사실성을 높이기 위해, 우리는 또한 플레이어가 실행되고 있는지 여부에 따라 bob의 크기를 증가시킨다. 

## 추가
ReactiveX on Unity의 실제 시리즈에서 이전 파트에 보너스 자료를 조금 추가하고 싶었습니다. 우리는 Shift 키를 누른 상태에서 실행할 수 있는 기능을 추가했습니다. 일부 게임 대신 Shift 키를 눌러 실행을 전환하는 옵션을 제공합니다. Observables 컨트롤러를 사용하면 변경 작업이 매우 쉽습니다.

신호 기반 아키텍처에 대한 더 좋은 점 중 하나는 신호에 반응하는 코드를 변경하지 않고 신호가 생성되는 방식을 변경할 수 있다는 것입니다. 원래 실행 신호는 다음과 같이 정의했습니다.

```
Run = this.UpdateAsObservable()
  .Select(_ => Input.GetButton("Fire3"))
  .ToReadOnlyReactiveProperty();
```

모든 Update, Run은 Shift ( "Fire3"입력 축)를 누르면 true 값으로 업데이트되고, 그렇지 않으면 false로 업데이트됩니다. (다른 코드에서 사용하기 쉽도록 Reactive Property로 저장했습니다.)

이것을 토글 입력으로 변경하는 것은 다음과 같이 쉽습니다.

```
var runValue = false;
Run = this.UpdateAsObservable()
  .Where(_ => Input.GetButtonDown("Fire3"))
  .Do(_ => runValue = !runValue)
  .Select(_ => runValue)
  .ToReadOnlyReactiveProperty();
```

runValue는 state-by-closure입니다. Shift 버튼을 누른 Update는 runValue를 무효화 한 다음 선택합니다. 여기서는 Do 메서드가 반드시 필요한 것은 아닙니다. 우리는 상태를 무효화하고 Select에 전달하는 함수에서 새 값을 반환 할 수 있습니다. 그러나 Do는 일부 부작용이 일어나고 있음을 명시 적으로 밝혀 주며,이 경우에는 상태 조작이 이루어 지므로 동료 프로그래머에게 경고 플래그 역할을합니다 (미래 자신 포함).

물론 아무도 runValue를 방해하지 않도록주의해야합니다. 따라서 유틸리티 함수에서 어딘가에 숨겨서 보호해야 할 수 있습니다.

가장 중요한 것은 실행 신호를 생성하는 코드를 제외하고 코드 행을 변경하지 않았기 때문입니다. 그것을 사용하는 모든 코드는 변경되지 않고 예상대로 작동합니다!
