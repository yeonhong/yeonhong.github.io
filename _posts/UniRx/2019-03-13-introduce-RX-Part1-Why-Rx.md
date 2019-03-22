---
layout: single
title: "PART 1 - Getting started : Why Rx?"
related: true
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx"
---

사용자는 실시간 데이터를 기대합니다. 결과를 기다리는 것을 막을 필요가 없습니다. 준비가되었을 때 결과를 당신에게 푸시하고 싶습니다. 결과 세트로 작업 할 때, 준비가 되면 개별 결과를 받기 원합니다. 첫 번째 행을보기 전에 전체 집합이 처리 될 때까지 기다리지 않으려 고합니다. 세상은 밀어 붙이기 시작했습니다. 사용자는 우릴 따라잡으려 합니다개발자는 데이터를 푸시 할 수 있는 도구를 가지고 있습니다. 개발자는 데이터를 푸시하기 위해 반응하는 도구가 필요합니다.

[Reactive Extensions for .NET (Rx)](https://docs.microsoft.com/en-us/previous-versions/dotnet/reactive-extensions/hh242985(v=vs.103))에 오신 것을 환영합니다. 이 책은 .NET 4에서 팝업 된 IObservable <T> 및 IObserver <T> 인터페이스에 대해 궁금한 모든 .NET 개발자를 대상으로합니다. Microsoft의 Reactive Extensions 라이브러리는 이러한 인터페이스를 구현하여 신속하게 견인력을 끌어 내고 있습니다. 서버, 클라이언트 및 웹 개발자 모두. Rx는 강력하게 생산적인 개발 도구입니다. Rx를 사용하면 개발자가 문제를 우아하고 친숙하며 선언적으로 해결할 수 있습니다. 종종 Rx없이 가능했던 것보다 적은 코드로 결정적으로. LINQ를 활용함으로써 Rx는 LINQ 구현의 표준 이점을 자랑합니다.

* 통합
  * LINQ는 C # 언어에 통합되어 있습니다.
* 균등
  * LINQ를 사용하면 기존 스킬을 활용하여 데이터를 쿼리 (LINQ to SQL, LINQ to XML 또는 LINQ to objects)하여 모션중인 데이터를 쿼리 할 수 ​​있습니다. Rx를 LINQ 이벤트로 생각할 수 있습니다. LINQ를 사용하면 다른 패러다임에서 일반적인 패러다임으로 전환 할 수 있습니다. 예를 들어 표준 .NET 이벤트, 비동기 메서드 호출, 태스크 또는 제 3 자 미들웨어 API를 하나의 공통 Rx 패러다임으로 전환 할 수 있습니다. 기존의 선택 언어를 활용하고 Select, Where, GroupBy 등의 익숙한 연산자를 사용하여 개발자는 일반적인 형식으로 디자인이나 코드를 합리화하고 전달할 수 있습니다.
* 확장 가능
  * 사용자 정의 쿼리 연산자 (확장 메서드)를 사용하여 Rx를 확장 할 수 있습니다.
* 선언적
  * LINQ는 코드가 코드에서 수행하는 작업의 선언으로 코드를 읽을 수있게하고 연산자를 구현하는 방법을 유지합니다.
* 작성 가능
  * 확장 메서드, 람다 구문 및 쿼리 이해 구문과 같은 LINQ 기능은 개발자가 소비 할 수있는 유창한 API를 제공합니다. 수많은 연산자로 쿼리를 구성 할 수 있습니다. 그런 다음 쿼리를 함께 구성하여 복합 쿼리를 더 생성 할 수 있습니다.
* 변형 가능
  * 검색어는 한 유형에서 다른 유형으로 데이터를 변형 할 수 있습니다. 쿼리는 단일 값을 다른 값으로 변환하거나 값 시퀀스에서 단일 평균 값으로 집계하거나 단일 데이터 값을 값 시퀀스로 확장 할 수 있습니다.

## When is Rx appropriate? (Rx가 적절한 때는?)
Rx는 일련의 이벤트를 다루기위한 자연스러운 패러다임을 제공합니다. 시퀀스에는 0 개 이상의 이벤트가 포함될 수 있습니다. Rx는 일련의 이벤트를 구성 할 때 가장 가치가 있습니다.

### Should use Rx
Rx가 구축 한 이벤트는 다음과 같습니다.
* 마우스 움직임, 버튼 클릭과 같은 UI 이벤트
* 속성 변경, 컬렉션 업데이트, 주문 완료, 등록 승인 등의 도메인 이벤트
* 파일 감시자, 시스템 및 WMI 이벤트와 같은 인프라 이벤트
* 메시지 버스의 브로드 캐스트 또는 WebSockets API 또는 Nirvana와 같은 기타 대기 시간이 짧은 미들웨어의 푸시 이벤트와 같은 통합 이벤트
* StreamInsight 또는 StreamBase와 같은 CEP 엔진과의 통합

Rx는 오프 로딩을 목적으로 동시성을 도입하고 관리하는 데에도 매우 적합합니다. 즉, 주어진 스레드를 동시에 수행하여 현재 스레드를 비울 수 있습니다. 이것의 매우 **보편적 인 사용은 응답성있는 UI를 유지하는 것입니다.**

모션에서 데이터를 모델링하려고 시도하는 기존 IEnumerable <T>을 가지고 있다면 Rx 사용을 고려해야합니다. IEnumerable <T>은 수익률 반환과 같은 지연 평가를 사용하여 움직이는 데이터를 모델링 할 수 있지만 실제로는 확장되지 않습니다. IEnumerable <T> 반복은 스레드를 소비 / 차단합니다. IObservable <T>를 통해 Rx의 비 차단 속성을 선호하거나 .NET 4.5에서 비동기 기능을 고려해야합니다.

### Could use Rx
Rx는 비동기 호출에도 사용할 수 있습니다. 이것들은 사실상 하나의 사건의 연속이다.
* Task 또는 Task<T>의 결과 
* FileStream BeginRead / EndRead와 같은 APM 메소드 호출 결과

TPL, Dataflow 또는 async 키워드 (.NET 4.5)를 사용하는 것이 비동기 메서드를 구성하는보다 자연스러운 방법임을 알 수 있습니다. Rx가 이러한 시나리오를 확실히 도와 줄 수 있지만,보다 적절한 다른 프레임 워크가 있으면 먼저 고려해야합니다.

Rx를 사용할 수는 있지만 병렬 계산을 수행하거나 스케일링 할 목적으로 병행 성을 도입하고 관리하는 데 적합하지 않습니다. TPL (Task Parallel Library) 또는 C ++ AMP와 같은 다른 전용 프레임 워크는 병렬 계산 집약적 인 작업을 수행하는 데 더 적합합니다.

Microsoft의 동시성 (Concurrency) 홈페이지에서 TPL, Dataflow, async 및 C ++ AMP에 대해 자세히 알아보십시오.

### Won't use Rx
Rx와 IObservable <T>는 IEnumerable <T>를 대체하지 않습니다. 자연스럽게 끌어 당겨서 푸시 기반이되도록 무언가를 취하려고하는 것은 좋지 않습니다.
* 코드베이스가 "더 많은 Rx"가되도록 기존 IEnumerable <T> 값을 IObservable <T>로 변환하는 짓.
* 메시지 큐. MSMQ 또는 JMS 구현과 같은 대기열은 일반적으로 트랜잭션 성을 가지며 정의 상 순차적입니다. 나는 IEnumerable <T>가 자연스럽게 여기 있다고 생각한다.

작업에 가장 적합한 도구를 선택하면 코드 유지 관리가 쉬워지고 성능이 향상되며 더 나은 지원을받을 수 있습니다.

## Rx in action
Rx를 채택하고 학습하면 인프라와 도메인에 느리게 적용 할 수있는 반복적 인 접근 방식이 될 수 있습니다. 짧은 시간에 간단한 연산자로 구성된 쿼리에 코드를 생성하거나 기존 코드를 줄일 수있는 기술을 갖추어야합니다. 예를들어 이 간단한 ViewModel은 사용자 유형으로 실행될 검색을 통합하기 위해 코딩해야하는 모든 것입니다.
``` csharp
public class MemberSearchViewModel : INotifyPropertyChanged
{
  //Fields removed...
  public MemberSearchViewModel(IMemberSearchModel memberSearchModel,
  ISchedulerProvider schedulerProvider)
  {
    _memberSearchModel = memberSearchModel;
    //Run search when SearchText property changes
    this.PropertyChanges(vm => vm.SearchText)
    .Subscribe(Search);
  }

  //Assume INotifyPropertyChanged implementations of properties...
  public string SearchText { get; set; }
  public bool IsSearching { get; set; }
  public string Error { get; set; }
  public ObservableCollection<string> Results { get; }
  //Search on background thread and return result on dispatcher.
  private void Search(string searchText)
  {
    using (_currentSearch) { }
    IsSearching = true;
    Results.Clear();
    Error = null;
    _currentSearch = _memberSearchModel.SearchMembers(searchText)
      .Timeout(TimeSpan.FromSeconds(2))
      .SubscribeOn(_schedulerProvider.TaskPool)
      .ObserveOn(_schedulerProvider.Dispatcher)
      .Subscribe(
        Results.Add,
        ex =>
        {
          IsSearching = false;
          Error = ex.Message;
        },
        () => { IsSearching = false; });
  }
  ...
}
```
이 코드 스니펫은 매우 작지만 다음 요구 사항을 지원합니다.
* 반응형 UI를 지속함
* timeout 지원
* 검색이 완료되면 알 수 있습니다.
* 결과를 한 번에 하나씩 되돌릴 수 있습니다.
* 오류 처리
* 동시성에 대한 우려가있는 경우에도 단위를 테스트 할 수 있다
* 사용자가 검색을 변경하면 현재 검색을 취소하고 새 텍스트로 새 검색을 실행합니다.

이 샘플을 생성하는 것은 거의 요구 사항을 단일 쿼리로 일치시키는 연산자를 작성하는 경우입니다. 쿼리는 작고, 유지 보수가 용이하며, 선언적이며 코드가 "자신 만의 것"보다 훨씬 적습니다. 잘 테스트 된 API를 재사용 할 때 얻을 수있는 이점이 있습니다. 작성해야 할 코드가 적을수록 테스트, 디버그 및 유지 관리해야하는 코드가 적습니다. 다음과 같은 다른 쿼리를 만드는 것은 간단합니다.

* 일련의 값들의 이동 평균을 계산하는 단계; 평균 대기 시간 또는 가동 중지 시간에 대한 서비스 수준 계약.
* Bing, Google 및 Yahoo의 검색 결과 또는 Accelerometer, Gyro, Magnetometer 또는 온도의 센서 데이터 등 여러 소스의 이벤트 데이터 결합
* 데이터 그룹화
  * 예 : 주제 또는 사용자 별 트윗 또는 델타 또는 유동성 별 주가
* 데이터 필터링 등. 지역 내의 온라인 게임 서버, 특정 게임 또는 최소한의 참가자 수.

push는 여기에 있습니다. Rx로 무장하는 것은 푸시 세계에 대한 사용자의 기대를 충족시키는 강력한 방법입니다. Rx의 구성 부분을 이해하고 구성함으로써 들어오는 이벤트를 처리하는 복잡성에 대한 짧은 작업을 수행 할 수 있습니다.