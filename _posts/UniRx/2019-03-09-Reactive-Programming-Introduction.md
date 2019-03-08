---
layout: single
title: "Reactive Programming: An Introduction for Game Developers"
related: true
categories: 
  - 프로그래밍
tags:
  - Reactive Programming
link: https://medium.com/@urschanselmann/reactive-programming-an-introduction-for-game-developers-f7da00edb424
---

# What is Reactive Programming?
지금, 나는 반응 프로그래밍이 무엇인지, 그것이 어떻게 왔는지, 왜 그렇게 많은 의미가 있는지에 대한 높은 수준의 설명으로 당신을 지루하게하지 않을 것입니다. 훨씬 실용적인 것으로 시작하겠습니다.

Unity3D와 같이 잘 설계된 게임 엔진으로 작업 할 때 프로그래머로서의 삶의 대부분은 재미 있습니다. 당신은 당신 자신의 스크립트 컴포넌트를 작성하고, 그 빌딩 블록으로부터 객체를 생성합니다 - 그리고 대개 모든 것이 정말 잘 어울립니다. 그리고 그렇게하면서 대부분의 소프트웨어 디자인 질문에 대한 답은 자연스럽게 여러분에게옵니다.

> 내 plater hp를 저장하려면? 물론 Player 구성 요소 내에 저장해야지.
> 로켓이 플레이어를 때리면 발생하는 피해량은 어디에 저장해야합니까? 로켓 구성 요소 내부!

당신은 당신 자신의 스크립트 컴포넌트를 작성하고, 그 빌딩 블록으로부터 객체를 생성합니다 - 그리고 대개 모든 것이 정말 잘 어울립니다. 그리고 그렇게하면서 대부분의 소프트웨어 디자인 질문에 대한 답은 자연스럽게 여러분에게옵니다.

# Glue Code — when it starts getting sticky… (끈적거리는 코드)
이제 우리는 비 개발자가 수행 할 게임을 만들고 싶어하기 때문에 플레이어는 자신의 건강 가치가 내부적으로 얼마나 깔끔하고 정돈되어 있는지 신경 쓰지 않습니다. 그들이 관심을 갖는 것은 게임 UI에 표시되는 것입니다.

UI 구성 요소 내부에서 UI 레이블에 대한 참조를 배치하는 방법과 상태가 변경 될 때마다 UI를 업데이트하는 방법은 무엇입니까?

계속해서. 레벨을 달성할 남은 시간은 어때?

왜 똑같이하지 않니? 레벨 구성 요소 안에는 다른 UI 레이블에 대한 참조가 있으며 게임 시간을 줄이면 업데이트됩니다.

그러나 이제는 그것이 전부가 아닙니다. 우리의 디자이너는 더욱 화려 해지기를 선택했기 때문에 플레이어의 건강이 감소 할 때마다 깜박이는 멋진 애니메이션이 있습니다.

좋습니다. 그럼 Player 클래스로 돌아가서 애니메이션에 대한 또 다른 참조를 추가하고 모든 hp 감소와 함께 애니메이션을 시작하십시오.

잠깐, 뭐라구? 수집 할 수있는 파워 업으로 일정 기간 동안 다른 플레이어를 제어하고 자신의 건강 대신 플레이어의 건강을 보여줄 수 있습니까?

좋아, 잠깐만 ... 아 ...

# Reactive Programming to the Rescue!
반응 형 프로그래밍의 진정한 이점을 이해하기 위해 우리는 약간 이론적 인 것을 얻어야 할 것입니다. 그러나 걱정할 필요는 없으며 가능한 한 간단하게 유지할 것입니다.

게임을 작성하는 것은 대부분 명령형 언어로 이루어지며, C #을 사용하여 Unity3D 용 스크립트를 작성하는 것도 예외는 아닙니다. 이제 우리가 명령형 언어에 대해 이야기 할 때, 우리가 일반적으로 비교하는 상대방은 함수형 프로그래밍 언어입니다. 그러나 우리는 오늘 그 길을 걷지 않고 대신 다른 규모에 대해 이야기하고 있습니다.

# Imperative vs Declarative (명령형 vs 선언적)
**명령형 언어와 선언 형 언어의 차이점은 아주 쉽게 구분할 수 있습니다. 명령형 언어는 무언가를 어떻게 처리 할 것인지를 설명하지만 선언적 언어는 수행 할 내용을 설명합니다.**

이제 우리는 C#이 명령형 언어라는 사실을 바꿀 수 없습니다. 그리고 핵심에서 우리는 그것을 사용하여 명령 할 수 있습니다. 그러나 내부적으로 우리 모두를 위해 어려운 작업을 수행하는 특정 프레임 워크를 사용하여 특정 선언 패러다임을 여전히 사용할 수 있습니다.

이 패러다임 중 하나는 리액티브 프로그래밍 (Reactive Programming)으로 Reactive Extensions for Unity (UniRx) 형식으로 Unity 및 C #에 제공됩니다.

## Show me some code!
우리는 앞에서 말한 Player 클래스 예제를 살펴봄으로써 이것을 시작합니다.

```
 using UnityEngine;
 using UnityEngine.UI;
 
 public class Player : MonoBehaviour {
   public int health = 100;
   public Text scoreLabel;
 
   public void ApplyDamage(int damage) {
     health -= damage;
     scoreLabel.text = health.ToString();
   }
 }
```
scorelabel과 health는 서로 연관되어 있습니다. 이제 그것을 제거하고 Reactive Extensions 마법으로 대체하십시오.

```
 using UnityEngine;
 using UnityEngine.UI;
 using UniRx;
 
 public class Player : MonoBehaviour {
   public ReactiveProperty<int> health 
     = new ReactiveProperty<int>(100);
 
   public void ApplyDamage(int damage) {
     health.Value -= damage;
   }
 }
```

이제 레이블 참조는 어디로 갔습니까? 우리는 그것을 다른 곳으로 옮기고 있습니다. 먼저 멋진 **ReactiveProperty 클래스**에 대해 이야기 해 보겠습니다.

이제는 특별한 종류의 컨테이너라고 가정 해 보겠습니다.이 컨테이너는 우리가 일반적으로 변수 안에 저장하는 모든 것을 저장할 수 있습니다 (이 경우에는 정수이지만 다른 간단한 유형이나 참조 일 수도 있습니다). 시작 값을 할당 할 수 있으며, 언제든지 그 안에 저장된 값을 액세스하고 변경할 수 있습니다.

## The Reactive superpowers of ReactiveProperty
UiPresenter 클래스에서 새로운 플레이어를 소개합니다.

```
 using UnityEngine;
 using UnityEngine.UI;
 using UniRx;
 
 public class UiPresenter : MonoBehaviour {
   public Player player;
 
   public Text scoreLabel;
 
   void Start() {
     player.health.SubscribeToText(scoreLabel);
   }
 }
```

우리 점수 레이블이 또 있습니다. 그러나 지금 기다려라, 그 밖의 무엇이 여기에서 일어나고 있냐?

우리는 선언적 대 명령론에 대해 어떻게 이야기했는지, 그리고 Reactive Extensions를 통해 선언적으로 확장 할 수있는 확장 성을 제공함으로써 어떻게 선언적이 될 수 있었는지 기억하십니까? 지금 더 혼란 스럽습니까?

나는 그것을 목적에 따라했다! 왜냐하면 당신이 지금 혼란 스러우면 곧 모든 점들을 곧 연결하는 것이 옳은 마음입니다. 걱정 마세요. 단계별로 단계별로 수행 할 것입니다.

우리가 실제로 여기에서하는 일부터 시작하거나, 컴퓨터가해야 할 일 (즉, 첫 번째 점)부터 시작합시다. 우리는 ReactiveProperty에 "구독"하라고 말하고 있습니다. 이것이 내부적으로 어떻게 수행되는지는 우리가 사용하고있는 Reactive Extensions에 의해 정의됩니다 (그래서 우리는 그것이 작동하는 방법에 대해 걱정할 필요가 없습니다 - 휴, 그렇지?).

그럼 그게 무슨 뜻 이죠? ReactiveProperty에 "구독" 하시겠습니까? 이 질문에 답하기 위해 먼저 몇 가지 개념을 밝혀야합니다.

ReactiveProperty는 Observable의 특별한 종류이며 Reactive Properties라는 기본 클래스가 만들어집니다. Observable을 **값의 흐름(stream of values)**으로 생각할 수 있습니다.

!(https://cdn-images-1.medium.com/max/800/1*NHginaEr1rpQgm_5iwZLMA.png)

6개의 Observables가 보입니다.

관찰 대상이 무엇인지 이해하려면 지금해야 할 일은 그림에 시간을 더하는 것입니다. 게임이 한 장씩 진행되는 것을 상상해보십시오. 관측 대상이 독립적으로 공을 방출합니다. 다음 이미지에서 우리는 관측 가능한 각 홀에 대해 이것을 봅니다. 다이어그램의 X 축은 앞으로 나아갈 시간입니다.

!(https://cdn-images-1.medium.com/max/800/1*jKlUkPH4sf-5fjPJxQCkFw.png)

좋아, observables은 시간이 지남에 따라 값의 흐름이다. 이제는 선언적 프로그래밍에 관한 그 모든 것이 무엇 이었습니까?

일단 관측치가 무엇인지 확립하면, 우리는 관측 대상이 무엇인지를 결정한 후에 그 위에 일반 작업을 정의함으로써 다음 단계로 넘어갈 수 있습니다.

우리의 사례로 돌아가서 풀 공 중 하나에서 풀 공이 사라지면 어떻게되는지 생각해보십시오. 그들은 테이블 아래에있는 어떤 경로를 따라 굴러 가기 시작합니다. 그리고이 예제를 위해 - 하나의 연속 행으로 끝나야한다고 가정 해 봅시다. 그래서 우리가 원하는 것은 이것입니다 :

!(https://cdn-images-1.medium.com/max/800/1*BWDr4QR2YmS3SmwR7wkWag.png)

이제 이 동작을 설명하기 위해 Reactive Extensions에서 제공하는 **선언적 언어**를 다음과 같이 사용할 수 있습니다.

```
all_balls_in_order = Observable.Merge(h1, h2, h3, h4, h5, h6);
```

핵심은 우리가 아직 아무 것도하지 않고 있다는 것입니다. 이 코드를 단독으로 실행하면 전혀 효과가 없습니다. 우리는 여기에서 어떤 데이터도 처리하지 않고 있습니다. 대신에 우리는 주어진 스트림에 어떤 일이 일어나는지 설명하고 나중에 다른 스트림을 나타내기 위해 사용될 수있는 모델을 생성합니다.

# Now we can even travel through time! (시간을 넘어서 여행을 할 수 있어)
그래도 앞으로만 전달됩니다. (뒤로는 못감) 너를 실망 시켜서 미안해.

우리의 풀 볼 예제를 다시 한 번 살펴 보겠습니다. 볼을 넣는 구멍에 따라 맨 아래로 굴러 갈 때까지 특정 시간이 걸리는 사실을 시뮬레이션하려고 시도합니다.

```
all_balls_delayed = Observable.Merge(
  h1.Delay(
    TimeSpan.FromSeconds(1)
  ), 
  h2.Delay(
    TimeSpan.FromSeconds(2)
  ), 
  h3.Delay(
    TimeSpan.FromSeconds(3)
  ), 
  h4.Delay(
    TimeSpan.FromSeconds(3)
  ), 
  h5.Delay(
    TimeSpan.FromSeconds(2)
  ), 
  h6.Delay(
    TimeSpan.FromSeconds(1)
  )
);
```

[RxMarble](https://rxmarbles.com/)에서 RX 제어에 대해 눈으로 보자.

# Ungluing the UI example
실제로 우리는 단지 UI 글루 코드를 없애고 싶었고, 이제 어떻게 든 시간 여행에 관해 이야기하는 것을 끝 냈는지 기억하십니까? Reactive Programming의 흥미 진진한 세계에 오신 것을 환영합니다!

그러나 실제로 이전의 예제로 되돌아갑니다. 우리가 관찰 할 수있는 것들에 대해 정의 할 수있는 가장 기본적인 작업들과 그것을 선언 한 후에 실제로 사용하는 방법에 대해 이야기합시다. 이제 UiPresenter의 다른 버전을 살펴 보겠습니다.

```
 using UnityEngine;
 using UnityEngine.UI;
 using UniRx;
 
 public class UiPresenter : MonoBehaviour {
   public Player player;
 
   public Text scoreLabel;
 
   void Start() {
     player.health
       .Select(health => string.Format("Health: {0}", health))
       .Subscribe(text => scoreLabel.text = text);
   }
 }
```

먼저 Select 연산자 (rxmarbles 및 다른 Reactive Extension 구현에서는 map이라고 함)에 대해 설명합니다. 그것이하는 일은 스트림의 모든 요소에 함수를 적용하는 것입니다. 이 경우 모든 입력 요소 (정수)를 가져 와서 문자열로 변환합니다.

나중에 우리가하는 일은 이 결과 스트림을 구독하는 것입니다. 스트림을 선언하는 것만으로는 아직 아무 것도하지 않는다고 말한 것을 기억하십니까?

**Subscribe는 실제로 스트림 처리를 시작하여 스트림의 모든 값을 제공된 콜백에 전달합니다. 콜백에서이 값을 사용하여 라벨의 텍스트 속성을 변경합니다.** 이전에 사용한 SubscribeToText 명령은 바로가기 일 뿐이며 정확히 똑같습니다.

이제 이 예제를 한 번 더 확장하여 이전에 다루 려던 세 가지 요구 사항을 모두 구현해 보겠습니다. 먼저 애니메이션을 보겠습니다.

```
 using UnityEngine;
 using UnityEngine.UI;
 using UniRx;
 
 public class UiPresenter : MonoBehaviour {
   public Player player;
 
   public Text scoreLabel;
   public Animator animator;
 
   void Start() {
     player.health
       .Select(health => string.Format("Health: {0}", health))
       .Do(x => animator.Play("Flash"))
       .Subscribe(text => scoreLabel.text = text);
   }
 }
```

그게 쉽지 않았습니까? 아마도 Do 연산자는 스트림의 모든 값에 대해 함수를 변경하지 않고 단순히 함수를 실행한다고 추측 할 수 있습니다.

이제 다른 세 번째로 제어되는 플레이어를 지원하는 세 번째 일 :

```
using UnityEngine;
 using UnityEngine.UI;
 using UniRx;
 
 public class UiPresenter : MonoBehaviour {
   public ReactiveProperty<Player> player 
     = new ReactiveProperty<Player>();
 
   public Text scoreLabel;
   public Animator animator;
 
   void Start() {
     var playerHealth = this.player
       .Where(player => player != null)
       .Select(player => player.health);
 
     playerHealth
       .Select(health => string.Format("Health: {0}", health))
       .Do(x => animator.Play("Flash"))
       .Subscribe(text => scoreLabel.text = text);
   }
 }
 ```

쉬운 일이 아니 었나요? 방금 플레이어를 ReactiveProperty로 바꾸어서 거기에서 가져 왔습니다. 그래서 한 특정 플레이어를 다루는 코드를 작성하는 대신 활성 플레이어를 변수에서 스트림으로 전환합니다.

그리고 일단 반응 형 프로그래밍을 더 파헤 치면 곧 깨닫게 될 것입니다. **거의 모든 것이 스트림으로 바뀔 수 있습니다.**
