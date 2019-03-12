---
layout: single
title: "ReactiveX and Unity3D: Part 3"
related: true
categories: 
  - 프로그래밍
tags:
  - Reactive Programming
  - UniRX
  - Unity
link: "https://ornithoptergames.com/reactivex-and-unity3d-part-3/"
---

## Jumping to a conclusion
WASD- 이동과 점프를위한 코드를 분리하려고 시도하는 것에 유혹 당할 수 있다. 그러나 이것은 프레임 당 한 번 이상 CharacterController에서 Move를 호출하는 것을 포함합니다. 적어도 이것은 낭비 적이기 때문에 잠재적으로 버그가 발생하기 쉽습니다. 또한 출력 신호에 대한 실행 순서 및 책임에 대한 질문을 제기합니다. 대신 이러한 신호를 결합하여 모든 움직임을 한 번에 계산하는 방법을 찾아 보겠습니다. 신호를 결합하는 것이 일반적인 필요 사항이므로 우리를 도울 수있는 도구가 있습니다.

궁극적으로 우리는 Zip을 사용합니다. Zip은 두 개의 Observable을 결합하여 각각의 값을 가져 와서 제공하는 함수를 통해 실행합니다. ([interactive marble diagrams을 확인하기 좋다.](http://reactivex.io/documentation/operators/zip.html))
```
observableA.Zip(observableB, (a, b) => /* combine a and b here */);
```
이것은 Observable을 제공합니다. Observable의 형식은 우리의 조합 함수의 반환 유형입니다. 간단한 구조체를 사용하여 입력 값을 패키지화합시다.
```
public struct MoveInputs {
  public readonly Vector2 movement;
  public readonly bool jump;

  public MoveInputs(Vector2 movement, bool jump) {
    this.movement = movement;
    this.jump = jump;
  }
}
```
구조체 정의에 익숙하지 않은 경우보다 자세한 정보 일 수 있습니다. 구조체 필드를 변경 가능하게두고 생성자를 버리는 것이 일반적입니다. 나는 그들이 읽기 전용이되도록 노력해야 할 가치가 있다고 생각합니다. 당신은 실제로 사람들이 Observables의 가치를 우연히 또는 의도적으로 망치는 것을 원하지 않습니다. 불변의 것은 미덕입니다.

이제는 어떻게 결합 할 것인지 알았으므로 점프 신호를 어떻게 정의해야합니까? 다음과 같이 할 수 있습니다.
```
this.UpdateAsObservable().Select(_ => Input.GetButton("Jump"));
```

이것을 movement signal과 결합한다고 상상해보십시오. 버그를 발견 할 수 있습니까? 우리의 이동 신호는 모든 고정 업데이트 값을 생성한다는 것을 상기하십시오. 업데이트와 고정 업데이트는 동일한 비율로 실행되지 않습니다 (우연히 만 가능). 그러나 Zip은 신호의 의도 된 주파수에 대해 아무 것도 알지 못하며, 하나씩 항목을 가져 와서 느린 신호를 참을성있게 기다리는 반면 값은 빠른 신호에 "쌓습니다". 내가 이것을 테스트했을 때, 점프 명령이 뒤처졌고, 게임이 실행되면 점차 악화되었습니다. Zip의 내부 버퍼가 결국 오버플로되고 추락했는지, 아니면 값을 삭제하기 시작했는지 확실하지 않습니다.

```
this.FixedUpdateAsObservable().Select(_ => Input.GetButton("Jump"));
```

불행하게도. 입력 샘플링은 update()에 연결되어 있으므로 fixedupdate() 중에 샘플을 샘플링하면 키 누름이 쉽게 누락 될 수 있습니다 (또는 프레임 속도가 실제로 불안정한 경우 키 누름 횟수를 여러 번 계산할 수 있음).

따라서 update() 도중 입력을 샘플링해야하지만 fixedupdate()가 발생할 때까지 키를 계속 누른 다음 clean 하십시오. 나는 디지털 회로 설계에서 영감을 얻어서 이것을 래치 (Latch) 라 부른다. 우리는 Custom Observable을 가지고 Latch를 구현할 것입니다. 우리의 입력은 데이터 신호 - 우리의 핵심 프레스 - 및 출력을 생성 할시기를 알려주는 클록 신호 - fixedupdate입니다.

첫째, Observable을 처음부터 어떻게 만들 수 있습니까? 지금까지는 기존 Observables 만 변형했습니다. 가장 기본적인 방법은 다음과 같은 Create 메소드를 사용하는 것입니다.

```
Observable.Create<T>(observer => {
  // Now use the observer instance to implement the Observable's behavior.
  // To send a value: observer.OnNext(value)
  // To terminate with an error: observer.OnError(exception)
  // To complete: observer.OnCompleted()

  // Return an IDisposable, which is used to clean up whatever we create.
  return diposable;
});
```

그것이 정말로 자유 형식으로 보인다면 그것은 그렇기 때문입니다! 이 인터페이스는 모든 종류의 동작을 신호로 구현할 수있는 많은 자유를줍니다. 당신은 다른 Observables를 사용할 수 있고 파일에서 웹 사이트 나 리소스를로드 할 수 있습니다. 모두 너에게 달렸어. 더 자세히 설명하지는 않겠지 만 Observables를 직접 만들려면 중요합니다. [Rx 소개는 그것에 대해 더 많은 것을 배울 수있는 훌륭한 자료이며, 모든 것이 자세히 자세히 설명되어 있습니다.](http://introtorx.com/Content/v1.0.10621.0/04_CreatingObservableSequences.html#CreationOfObservables) 심지어 C #으로 작성되었습니다.

그래서, Here Here Be Dragons가 제대로 작동하는지 보도록 경고했습니다. Latch Observable은 내부적으로 데이터 및 클럭 신호를 구독하고 래치 상태를 유지합니다. 코드로 들어가기 전에 여기 marble diagram이 있습니다.

![no-alignment](https://ornithoptergames.com/content/images/2016/08/Observable-Latch-Marble-Diagram.png)

```
public static IObservable<bool> Latch(
    IObservable<Unit> tick,
    IObservable<Unit> latchTrue,
    bool initialValue) {

  // Create a custom Observable.
  return Observable.Create<bool>(observer => {
    // Our state value.
    var value = initialValue;

    // Create an inner subscription to latch:
    // Whenever latch fires, store true.
    var latchSub = latchTrue.Subscribe(_ => value = true);

    // Create an inner subscription to tick:
    var tickSub = tick.Subscribe(
      // Whenever tick fires, send the current value and reset state.
      _ => {
        observer.OnNext(value);
        value = false;
      },
      observer.OnError, // pass through tick's errors (if any)
      observer.OnCompleted); // complete when tick completes

    // If we're disposed, dispose inner subscriptions too.
    return Disposable.Create(() => {
      latchSub.Dispose();
      tickSub.Dispose();
    });
  });
}
```

2 개의 내부 구독이 처리를 위해 올바르게 준비되어 있음을 알 수 있습니다. 그렇지 않으면 그들은 필요 이상으로 계속해서 살아남아 메모리 누수와 CPU 낭비를 초래할 수 있습니다.

Tick에 대한 subscription에는 OnError 및 OnCompleted 처리기 함수에 대한 아직 다루지 않은 세부 정보가 포함되어 있습니다. 지금까지 우리는 오류를 던지거나 종료 할 Observable을 실제로 보지 못했습니다. 좋은 예가 웹에서 자원을로드하는 Observable입니다. 연결이 끊어지면 오류가 발생하고 성공하면 단일 값을 발생시킨 다음 종료됩니다. Subscribe의 2 및 3 인수 변종은 이러한 사례를 처리 할 수있는 함수를 제공합니다. 이 경우 오류 또는 완성 된 신호로 영리하게하려고하지 않으므로 관찰자에게 전달합니다.

두 개의 서로 다른 비동기 프로세스에서 값에 액세스하는 정확성에 의문을 제기 할 수 있습니다. 당신은 그것을 자세히 조사하는 것이 옳지 만 잠재적 인 경쟁 조건의 영향은 여기에서 무시할 수 있다고 생각합니다. (당신이 생각한대로 실행 입력과 동일하게 손을 흔드는 것).

이와 같이 정의 된 래치를 사용하여이 모든 것을 함께 모아 IObservable <MoveInputs>를 생성 할 수 있습니다.

```
Jump = this.UpdateAsObservable().Where(_ => Input.GetButtonDown("Jump"));
var jumpLatch = CustomObservables.Latch(this.FixedUpdateAsObservable(), Jump, false);
MoveInputs = Movement.Zip(jumpLatch, (m, j) => new MoveInputs(m, j));
```

우리는 이제 두 개의 매우 다른 입력 신호를 성공적으로 조정했습니다. 하나는 연속 (WASD 키를 누르고 있음)이고 다른 하나는 순간 누적 (Space pressed and released)입니다.

PlayerController에서 inputs.Movement.Subscribe (...)를 inputs.MoveInputs.Subscribe (...)로 변경하고 나머지는 간단합니다 (아래 전체 코드).

컨트롤러의 출력 신호와 유사합니다. Walked :이 계산에서 점프 및 착륙에 대한 신호를 추가 할 수도 있습니다. Walked와 별도로 구독하고 수학을 조금만 더 사용하면 발걸음을 시뮬레이트하기 위해 단계별 신호를 만들 수 있습니다. 사운드 효과를 추가하려면 이러한 요소가 필요합니다.

## Sounds to astound
우리는 간단한 Unity 스크립트 인 PlayerAudio를 사용하여 AudioClips 및 AudioSource를 구성하고 간단한 구독을 다음과 같이 만듭니다.
```
player.Jumped
  .Subscribe(_ => audioSource.PlayOneShot(jump))
  .AddTo(this);
```
나는 이것이 우리 셋업에 대한 정말로 우아한 결과라고 생각한다. 그리고 그것은 음향 효과에만 국한되지 않습니다. 이러한 이벤트에 대한 응답으로 시각 효과 또는 AI 트리거를 추가하는 것도 이제는 간단합니다. 우리는 선수가 뛰어 내린 횟수를 쉽게 추적 할 수 있습니다. 예를 들어 업적 시스템. 무엇보다 우리는 정상적이고 예측 가능하며 분리 된 코드로이 모든 것을 할 수 있습니다.

