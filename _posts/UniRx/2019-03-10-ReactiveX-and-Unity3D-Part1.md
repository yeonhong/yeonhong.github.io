---
layout: single
title: "ReactiveX and Unity3D: Part 1"
related: true
categories: 
  - 프로그래밍
tags:
  - Reactive Programming
  - UniRX
  - Unity
link: "https://ornithoptergames.com/reactiverx-in-unity3d-part-1/"
---

단단히 결합 된 코드는 두통입니다. 중력과 같은 자연스러운 힘이 있어야 코드를 짜내고, 읽을 수 없으며, 허약 한 얽힘을 코드를 작성할 때 끌어 내릴 수 있습니다. 그리고 게임 개발의 어떤 이유로 이 힘은 두 배로 강력합니다! 의식적으로 저항하지 않는 한, 결국 게임은 임계 질량에 도달하고 블랙홀로 붕괴 될 것입니다.

좋은 코드는 우리가 사용하는 도구와 많은 관련이 있습니다 (반복 할 위험에 처해 있음). 게임 개발자는 커플 링을 제어하기 위해 Entity Component Systems와 같은 도구를 사용하는 것이 일반적입니다. 내 목표는 게임 개발 (지금까지)에서 덜 일반적으로 알려진 다른 도구 (내가 생각하기에)를 익히는 것입니다 : [ReactiveX](http://reactivex.io/)

## The tool
기본 레벨에서, ReactiveX를 사용하는 것은 이벤트 처리와 매우 흡사하다고 말할 수 있습니다. IObservable <T>. 우리는 그들을 변형시키고, 지연 시키며, 필터링하고, 결합하고, 다양한 방법으로 커스터마이징하여 이전에 사용했던 오래된 이벤트 처리보다 훨씬 더 강력해질 수 있습니다. (ReactiveX 사람들은 원할 경우 더 완벽한 소개를 제공 할 수 있습니다.)

## These boots are made for walking
우리 player가 움직 이도록하십시오. 고전적인 방법은 다음과 같은 스크립트를 사용하는 것입니다.

```csharp
public class ClassicPlayerController : MonoBehaviour {
  private void Update() {
    // Read keyboard inputs
    // Translate into movement velocity
    // Apply that velocity to the character
  }
}
```

이미 코드를 긴밀하게 연결하기 시작했습니다. 플레이어 컨트롤러가 입력을 읽는 법을 알아야하는 이유는 무엇입니까? 입력이 제공되는 방식을 변경하려면 어떻게해야합니까? 어쩌면 댄스 패드, VR 헤드셋, 또는 트 위치 (Twitch)의 명령 또는 일부 인공 지능 인공 지능으로부터 얻을 수 있습니다. 우리는 이들 각각에 대해 새로운 플레이어 컨트롤러를 작성할 필요가 없습니다. 게임이 항상 키보드로 구동된다는 것을 알고 있더라도 불필요한 커플 링은 피해야하며 지금부터 시작하는 것이 가장 좋습니다.

대신, 움직임 입력을 신호로 생각하십시오 : W 키를 누르면 신호가 "앞으로갑니다"라고 말합니다. 우리의 컨트롤러는 신호가 어떻게 만들어 졌는지 알지 못합니다. 신호를 보았을 때 무엇을 해야할지 간단합니다. 신호는 Observables입니다. 이동 (2D 방향)의 경우 IObservable <Vector2> 일 수 있습니다. 궁극적으로 플레이어 컨트롤러는 Observable에 가입하여 새로운 Vector2가 처리 될 때마다 알림을받을 Observer를 만듭니다.

따라서 입력을위한 스크립트 만 작성하여 관심를 분리합시다.

```csharp
public class Inputs : MonoBehaviour {

  public IObservable<Vector2> Movement { get; private set; }

  private void Awake() {
    Movement = this.FixedUpdateAsObservable()
      .Select(_ => {
        var x = Input.GetAxis("Horizontal");
        var y = Input.GetAxis("Vertical");
        return new Vector2(x, y).normalized;
      });
  }
}
```
먼저 우리는 Awake에서 이동 신호를 public 속성으로 선언하고 초기화합니다. Observables의 가장 좋은 점 중 하나는 외부로부터 값을 주입 할 수 없다는 것입니다. 하나를 만든 후에 할 수있는 것은 값을 얻기 위해 구독하는 것입니다. 실망스럽게 들릴지 모르겠지만, 실제로 코드를 멋지게 분리 할 수있는 매우 강력한 속성입니다. Observable을 만드는 가장 일반적인 방법은 기존의 것을 변환하는 것입니다. 원래의 Observable은 여전히 ​​남아 있기 때문에 "변형"은 느슨하게 사용됩니다. 즉, 변경 불가능한 것이므로 새로운 것을 만듭니다.

언제 입력을 읽고 싶습니까? 모든 고정 업데이트 (운동이 물리 시스템을 포함 할 것이므로 업데이트가 아님). 당신이 this.FixedUpdateAsObservable를 호출하여 액세스 () : UniRx는 고정 업데이트의 "tick"의 감시를 제공합니다. 이것은 우리에게 IObservable <Unit>을 줍니다. 유닛은 너무 자세하게 설명하지 않고 신호에 데이터 페이로드가 없음을 알립니다. 이 신호가 전혀 발사된다는 사실이 신호입니다. 이것에 우리는 각각의 입력 값에 대해, 우리는 전달 함수에 따른 새로운 값을 출력하는 새로운 관찰 가능한 만드는 특별한 방법 선택을 호출한다.이 기능은 보행 입력을 판독하고, 정규화 벡터로서 출력한다. 그래서 Movement는 Observable입니다. 모든 Fixed Update는 키보드 입력을 나타내는 벡터를 출력합니다.

어떻게 사용합니까? 다른 스크립트에서는이 신호를 구독하고 캐릭터에게 움직임을 적용합니다.

```csharp
public class PlayerController : MonoBehaviour {
  // ... fields omitted ...

  private void Start() {
    inputs.Movement
      .Where(v => v != Vector2.zero)
      .Subscribe(inputMovement => {
        var inputVelocity = inputMovement * walkSpeed;

        var playerVelocity =
          inputVelocity.x * transform.right +
          inputVelocity.y * transform.forward;

        var distance = playerVelocity * Time.fixedDeltaTime;
        character.Move(distance);
      }).AddTo(this);
  }
}
```
먼저, Awake를 사용하여 신호를 설정하고 이제 구독을 시작합니다. 이는 초기화 순서 문제를 피하기위한 것이며 일반적으로 따르는 패턴입니다. (내 Observables에 대한 깨우침, 당신을 위해 시작하십시오.)

Movement Observable에 대한 참조를 얻은 후에, 우리는 아주 단순한 최적화를했습니다. Where to call. 다른 변환 함수는 어디에 있는가?이 경우 우리는 0 인 이동 벡터를 무시합니다. 사용자가 아무 키도 누르지 않으면 우리는 일찍 깨어납니다.

그런 다음 Observable에서 새로운 가치가있을 때마다 호출 할 함수를 제공하여 구독합니다. 이 함수에서 단순히 플레이어의 보행 속도를 곱하고 2D 입력을 플레이어의 3D 좌표계로 변환하고 시간을 곱하여 거리를 구하며 CharacterController를 사용하여 운동을 적용합니다.

마지막으로, **UniRx 세부 정보를 AddTo(this)라고 부릅니다. Observables 및 구독 (Observers)이 살아있는 것임을 아는 것이 중요합니다.** 신호가 계속되는 한, 그들은 다 행복하게 계속 처리 할 것입니다. Observables는 오류를 완료하거나 생성 할 수 있지만, 현재로서는 다루지 않습니다. 메모리와 낭비되는 처리 능력을 누설하지 않으려면 게임 객체가 파괴 될 때 정리해야합니다. 그것이 AddTo (이)가 우리를 위해하는 것입니다. 기본적으로 PlayerController의 게임 객체가 파기되면 구독을 처리합니다. 이것은 Observable, Movement를 처리하지 않는다. Movement가 해당 객체의 Fixed Update Observable에서 시작 되었기 때문에 Inputs 게임 객체가 파괴 될 때 처리됩니다.

## 내 생각
react는 명령형에서 선언적 언어로 바꾸는 패러다임이다. 내가 원하는 동작을 작성하는 것으로 구현을 한다는 개념.
근데 대부분 이벤트 중심으로 동작하고 있다. 어쩌면 내가 작성하려는 코드도 어떤 입력(이벤트)를 받고 처리하는 것이기에 범용적으로 사용할 수 있겠지.