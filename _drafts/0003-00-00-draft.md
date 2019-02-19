---
layout: single
title: "UniRX (infallible Code)"
categories: 
  - 프로그래밍
tags:
  - UniRX
  - Unity
---

infallible Code의 UniRX 관련 동영상을 보고 정리한 내용. (in youtube)
> 비동기적으로 로직은 운용할때 유용한 framework로 이해가 된다. 일반적인 ui, user input, event를 제어할때 사용하면 좋을 것 같다.

### Theory of Reactive Programming | Learning UniRx [1]
[링크](https://youtu.be/9If7GUD0Dao)

* react programming 는 async data stream programming이다.
  * data stream을 섞거나 엮거나 조합하게 하는 프로그래밍.
  * imperative code (명령형 코드)
    * 명령에 따라 상태를 바꾸는 코드.
    * 프로그램이 어떻게 운용되는지 설명한다.
  * declarative code (선언적인 코드) >> ** rx의 방식. **
    * 운용되는 플로우를 설명하지 않고 로직을 표현한다.
    * 프로그램이 달성해야하는 것을 설명한다.

### Observables: Asynchronous Data Streams | Learning UniRx [2]
[링크](https://youtu.be/FHdDHlaETCw)

* Observable (Data Stream)
  * 이벤트의 순서를 표현하는 것.
  * Observers subcribe to Observables, and react to the value that they emit.
    * 옵저버는 데이터스트림을 구독하고, 그것의 값을 내놓을때 반응한다.

* The Observable Contract
  * OnNext : 항목이 어떤 데이터를 노티함. (OnNext(1), 1이라는 데이터를 노티하기)
  * OnCompleted : 성공적으로 종료했을때.
  * OnError : 에러로 인해 종료되었을때.
  * Subscribe : 노티를 받을 준비가 됨.
  * Unsubscribe : 더이상 노티를 안받음.

* UniRX는 대부분의 Mono 이벤트에 AsObservable로 함수를 만들어 둠. (ex. OnUpdateAsObservable)

### The Power of Operators | Learning UniRx [3]
[링크](https://youtu.be/or2IsE0zKs0)
*
* Rx가 좋은건 연산자를 연결하여 사용가능하다는 것.
  * 노티된 데이터를 변경하고 조합하여 사용할 수 있다.
  * 하나의 observable의 기능을 수행하고, 다른 observable을 반환할 수 있다.*
    * 각 observable의 데이터를 수정하지 않고, 결과물을 새로운 observable을 생성함
    * 일종의 쿼리문처럼 데이터를 다룬다.
  * 연결하는 종류
    * Creation : Observable을 생성하기.
    * Transform : Observable에서 노티된 데이터를 변환하기.
    * Filter : 노티된 데이터에서 조건에 맞는 녀석만 가져오기.
    * Combine : 두 Observable의 값을 조합하여 하나의 Observable로 만들기.

### How to Create Data Streams in Unity | Learning UniRx [4]
[링크](https://youtu.be/qcX2ghT8jtc)

유니티에서 데이터 스트림을 만드는 방식들

* Creater Operator
  * var datastream = observable.create<int>(observer => {return Dispose.empty});
  * datastream.Subscribe ()...
  * 커스텀하게 데이터 스트림을 생성하기.
* Monobehaviour Triggers.
  * OnEnableAsObservable().Subscribe()..
  * UpdateAsObservable().Subscribe()..
  * 위와 같이 모노의 기본 이벤트 트리거에 대한 구독 처리.
  * 또한 커스텀하게 Trigger를 만들 수 있다. (ObservableTriggerBase를 상속 받아서)
* Observable Coroutines
  * 코루틴을 구독하기. 코루틴을 데이터 스트림처럼 사용.
  * Observable.FromCoroutine(...).Subscribe()...
  * 코루틴함수().ToObservable().Subscribe()...
* Yield To Observables
  * Yield에 rx를 사용하여 반환까지 다양한 처리를 추가할 수 있다.
* Observable Network Operations
  * 네트워크를 rx식으로 구현할 수 있다.
  * var requestStream = ObservableWWW.GET(...).Subscribe()...
* FrameCount-Based Time Observable
  * 모든 업데이트나, 특정 프레임마다, 몇 프레임 이후에 노티를 발생시키는 방식.
  * Observable.EveryUpdate().Subscribe..
  * Observable.TimerFrame(60)..
  * Observable.IntervalFrame(60)..
