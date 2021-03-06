---
layout: single
title: "AssetBundle usage patterns"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Assetbundle
link: https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-usage-patterns?playlist=30089
---

이 시리즈의 이전 장에서는 다양한 로딩 API의 저수준 동작을 포함하는 AssetBundles의 기본 사항에 대해 설명했습니다. 이 장에서는 실제로 AssetBundles를 사용하는 다양한 측면에 대한 문제와 잠재적 인 솔루션에 대해 설명합니다.

## Managing loaded Assets
메모리에 민감한 환경에서는, 로드 된 객체의 크기와 수를 신중하게 제어하는 것이 중요합니다. 유니티는 활성 장면에서 객체를 제거 할 때 객체를 자동으로 언로드하지 않습니다. 자산 cleanup은 특정 시간에 트리거되며 수동으로 트리거 될 수도 있습니다.

AssetBundles 자체는 신중하게 관리해야합니다. AssetBundle은 로컬 스토리지에있는 파일 (Unity 캐시 또는 AssetBundle.LoadFromFile을 통해로드 된 파일)에 의해 지원되므로 최소한의 메모리 오버 헤드를 가지므로 수십 킬로바이트 이상을 소비하지 않습니다. 그러나 많은 수의 AssetBundle이 있으면이 오버 헤드가 여전히 문제가 될 수 있습니다.

대부분의 프로젝트에서 사용자는 콘텐츠 재생 (예: 레벨 재생)을 할 수 있으므로 AssetBundle을 로드하거나 언로드 할시기를 알아야합니다. **AssetBundle이 부적절하게 언로드되면 메모리에 Object 복제가 발생할 수 있습니다.** 애셋 번들을 부적절하게 언로드하면 텍스처가 누락되는 등의 특정 상황에서 바람직하지 않은 동작이 발생할 수도 있습니다.

**자산 및 AssetBundle을 관리 할 때 이해해야 할 가장 중요한 점은 unloadAllLoadedObjects 매개 변수에 대해 AssetBundle.Unload를 true 또는 false로 호출 할 때 동작의 차이입니다.**

이 API는 호출중인 AssetBundle의 헤더 정보를 언로드합니다. unloadAllLoadedObjects 매개 변수는이 AssetBundle에서 인스턴스화 된 모든 객체를 언로드할지 여부를 결정합니다. true로 설정하면 AssetBundle에서 비롯된 모든 객체가 현재 활성 장면에서 사용되고있는 경우에도 즉시 언로드됩니다.

예를 들어, M이 AssetBundle AB에서로드되었고 M이 현재 활성 장면에 있다고 가정합니다.

!(https://unity3d.com/sites/default/files/ab2a.jpg)

AssetBundle.Unload (true)가 호출되면 M은 장면에서 제거되고 파괴되고 언로드됩니다. 그러나 AssetBundle.Unload (false)가 호출되면 AB의 헤더 정보가 언로드되지만 M은 장면에 남아 있고 계속 작동합니다. AssetBundle.Unload (false)를 호출하면 M과 AB 사이의 링크가 끊어집니다. AB가 나중에 다시로드되면 AB에 포함 된 객체의 새로운 사본이 메모리에로드됩니다.

!(https://unity3d.com/sites/default/files/ab2b.jpg)

AB가 나중에 다시로드되면 AssetBundle의 헤더 정보의 새로운 사본이 다시로드됩니다. 그러나 M은 AB의 새로운 사본에서로드되지 않았습니다. 유니티는 AB와 M.의 새 복사본 사이의 링크를 설정하지 않습니다

!(https://unity3d.com/sites/default/files/ab2c.jpg)

AssetBundle.LoadAsset ()가 M을 다시로드하기 위해 호출 된 경우 Unity는 M의 이전 복사본을 AB의 데이터 인스턴스로 해석하지 않습니다. 그러므로 Unity는 M의 새로운 복사본을로드 할 것이고 M의 동일한 복사본 두 개가 장면에있게 될 것입니다.

!(https://unity3d.com/sites/default/files/ab2d.jpg)

대부분의 프로젝트에서이 동작은 바람직하지 않습니다. 대부분의 프로젝트는 AssetBundle.Unload (true)를 사용하고 객체가 중복되지 않도록 보장하는 메소드를 채택해야합니다. 일반적인 두 가지 방법은 다음과 같습니다.
1. 레벨 간 또는 로딩 화면과 같이 일시적인 AssetBundles이 언로드되는 애플리케이션의 수명 동안 잘 정의 된 포인트가 있습니다. 이것은 간단하고 가장 일반적인 옵션입니다.
2. 개별 Object에 대한 참조 카운트 유지 및 모든 구성 객체가 사용되지 않은 경우에만 AssetBundle 언로드. 이렇게하면 응용 프로그램이 메모리를 복제하지 않고 개별 객체를 언로드하고 다시로드 할 수 있습니다.

응용 프로그램에서 AssetBundle.Unload (false)를 사용해야만 하는 경우, 개별 객체는 다음 두 가지 방법으로만 언로드 할 수 있습니다.
1. 장면과 코드에서 원하지 않는 객체에 대한 모든 참조를 제거하십시오. 이 작업이 완료되면 Resources.UnloadUnusedAssets를 호출하십시오.
2. 장면을 비가 상적으로로드하십시오. 그러면 현재 장면의 모든 객체가 삭제되고 Resources.UnloadUnusedAssets가 자동으로 호출됩니다.

프로젝트가 게임 모드 나 레벨 사이에서와 같이 객체로드 및 언로드를 기다릴 수있는 잘 정의 된 지점이있는 경우 이러한 점을 사용하여 필요한 만큼의 객체를 언로드하고 새 객체를로드해야합니다.

가장 간단한 방법은 프로젝트의 각 덩어리를 씬에 패키지화 한 다음 해당 씬을 모든 종속성과 함께 AssetBundles로 빌드하는 것입니다. 그런 다음 응용 프로그램은 "로드 중" 씬을 입력하고 이전 장면이 포함 된 AssetBundle을 완전히 언로드 한 다음 새 씬에 포함 된 AssetBundle을 로드 할 수 있습니다.

이것이 가장 간단한 흐름이지만 일부 프로젝트는보다 복잡한 AssetBundle 관리가 필요합니다. 모든 프로젝트가 다르기 때문에 보편적 인 AssetBundle 디자인 패턴이 없습니다.

객체를 AssetBundle로 그룹화하는 방법을 결정할 때, 객체를 동시에로드하거나 업데이트해야하는 경우 객체를 AssetBundle에 번들링하는 것이 일반적으로 가장 좋습니다. 예를 들어, 롤 플레잉 게임을 생각해보십시오. 개별 맵과 컷씬은 장면별로 AssetBundles로 그룹화 할 수 있지만 대부분의 장면에서는 일부 오브젝트가 필요할 것입니다. AssetBundles는 인물 게임, 인게임 UI 및 다양한 캐릭터 모델과 텍스처를 제공하도록 제작할 수 있습니다. 이 후자의 개체와 자산은 시작시로드되고 앱의 수명 동안로드 된 상태로 유지되는 두 번째 AssetBundle 집합으로 그룹화 할 수 있습니다.

AssetBundle이 언로드 된 후에 Unity가 AssetBundle에서 객체를 다시로드 해야하는 경우 또 다른 문제가 발생할 수 있습니다. 이 경우 리로드가 실패하고 Object가 Unity Editor의 계층 구조에 (Missing) 객체로 나타납니다.

**이는 주로 Unity가 모바일 앱이 일시 중지되거나 사용자가 자신의 PC를 잠그는 등 그래픽 컨텍스트에 대한 제어권을 잃어 버리고 다시 회복 할 때 발생합니다. 이 경우 Unity는 텍스처와 쉐이더를 GPU에 다시 업로드해야합니다.** 이러한 애셋의 소스 AssetBundle을 사용할 수없는 경우 응용 프로그램은 장면의 객체를 자홍색으로 렌더링합니다.

## 4.2. Distribution (배포)
프로젝트의 AssetBundles를 클라이언트에 배포하는 두 가지 기본 방법은 프로젝트와 동시에 설치하거나 설치 후 다운로드하는 것입니다.

AssetBundle을 설치 중에 또는 이후에 제공할지 여부는 프로젝트가 실행될 플랫폼의 기능 및 제한 사항에 따라 결정됩니다. 모바일 프로젝트는 일반적으로 초기 설치 크기를 줄이고 무선 다운로드 크기 제한을 유지하기 위해 설치 후 다운로드를 선택합니다. 콘솔 및 PC 프로젝트는 일반적으로 초기 설치시 AssetBundles를 제공합니다.

적절한 아키텍처를 사용하면 처음에 AssetBundle을 전달하는 방법과 상관없이 프로젝트의 사후 설치에 새롭거나 수정 된 내용을 패치 할 수 있습니다. 이에 대한 더 자세한 정보는 Unity 매뉴얼의 [AssetBundles로 패치](https://docs.unity3d.com/Manual/AssetBundles-Patching.html?_ga=2.19443519.60866194.1551365663-678518112.1480121168)하기 섹션을 참조하십시오.

### 4.2.1. Shipped with project
프로젝트와 함께 AssetBundles를 포함하는 것이 추가 다운로드 관리 코드가 필요 없으므로 가장 간단하게 배포 할 수 있습니다. 프로젝트에 AssetBundles를 설치할 수 있는 주요 이유는 두 가지입니다.
* 프로젝트 구축 시간을 줄이고 반복 개발을 단순화합니다. 이러한 AssetBundle을 응용 프로그램과 별도로 업데이트 할 필요가없는 경우 AssetBundles를 Streaming Assets에 저장하여 응용 프로그램에 포함시킬 수 있습니다. 아래의 스트리밍 저작물 섹션을 참조하십시오.
* 업데이트 할 수 있는 콘텐츠의 초기 버전을 제공합니다. 이는 일반적으로 초기 설치 후 최종 사용자 시간을 절약하거나 나중에 패치하기 위한 기초로 사용하기 위해 수행됩니다. 스트리밍 애셋은 이 경우에 이상적이지 않습니다. 그러나 사용자 정의 다운로드 및 캐싱 시스템을 작성하는 것이 옵션이 아닌 경우, 업데이트 할 수 있는 컨텐츠의 초기 개정을 Streaming Assets의 Unity 캐시에로드 할 수 있습니다.

#### 4.2.1.1. Streaming Assets
설치시 Unity 어플리케이션 내에 AssetBundles를 포함한 모든 유형의 컨텐츠를 포함시키는 가장 쉬운 방법은 프로젝트를 빌드하기 전에 / Assets / StreamingAssets / 폴더에 컨텐츠를 빌드하는 것입니다. 빌드시에 StreamingAssets 폴더에 포함 된 모든 내용이 최종 응용 프로그램에 복사됩니다.

로컬 저장소의 StreamingAssets 폴더에 대한 전체 경로는 런타임에 [Application.streamingAssetsPath](https://docs.unity3d.com/ScriptReference/Application-streamingAssetsPath.html?_ga=2.228560410.60866194.1551365663-678518112.1480121168) 속성을 통해 액세스 할 수 있습니다. AssetBundle은 대부분의 플랫폼에서 AssetBundle.LoadFromFile을 통해로드 될 수 있습니다.

**Android 개발의 경우** : Android에서는 StreamingAssets 폴더의 자산이 APK에 저장되므로 APK에 저장된 파일이 다른 저장소 알고리즘을 사용할 수 있으므로 압축 된 경우로드하는 데 시간이 더 걸릴 수 있습니다. 사용 된 알고리즘은 Unity 버전마다 다를 수 있습니다. 7-zip과 같은 아카이버를 사용하여 APK를 열어 파일 압축 여부를 결정할 수 있습니다. 그렇다면 AssetBundle.LoadFromFile ()이 더 느리게 수행 될 것으로 기대할 수 있습니다. 이 경우 해결 방법으로 UnityWebRequest.GetAssetBundle을 사용하여 캐시 된 버전을 검색 할 수 있습니다. **UnityWebRequest를 사용하면 AssetBundle이 첫 번째 실행 중에 압축 해제되고 캐시되므로 실행 속도가 빨라집니다.** AssetBundle이 캐시에 복사되므로 더 많은 저장 공간이 필요합니다. 또는 빌드 타임에 Gradle 프로젝트를 내보내고 AssetBundle에 확장을 추가 할 수 있습니다. 그런 다음 build.gradle 파일을 편집하고 해당 확장을 noCompress 섹션에 추가 할 수 있습니다. 끝나면 압축 해제 성능 비용을 지불하지 않고도 AssetBundle.LoadFromFile ()을 사용할 수 있어야합니다.

_참고_ : 일부 플랫폼에서는 스트리밍 자산이 쓰기 가능한 위치가 아닙니다. 설치 후 프로젝트의 AssetBundle을 업데이트해야하는 경우 WWW.LoadFromCacheOrDownload를 사용하거나 사용자 정의 다운로더를 작성하십시오.

### 4.2.2. Downloaded post-install
AssetBundles를 휴대 기기에 전달하는 가장 좋은 방법은 앱 설치 후 앱 세트를 다운로드하는 것입니다. 또한 설치 후 사용자가 전체 응용 프로그램을 다시 다운로드하지 않고도 컨텐트를 업데이트 할 수 있습니다. 많은 플랫폼에서 애플리케이션 바이너리는 비싸고 오랜 시간이 걸리는 재 인증 프로세스를 거쳐야합니다. 따라서 설치 후 다운로드에 적합한 시스템을 개발하는 것이 중요합니다.

AssetBundle을 전달하는 가장 간단한 방법은 웹 서버에 배치하고 UnityWebRequest를 통해 전달하는 것입니다. Unity는 다운로드 한 AssetBundle을 로컬 스토리지에 자동으로 캐시합니다. 다운로드 한 AssetBundle이 LZMA로 압축 된 경우, AssetBundle은 향후 로딩 속도를 높이기 위해 LZ4 ([Caching.compressionEnabled](https://docs.unity3d.com/ScriptReference/Caching-compressionEnabled.html?_ga=2.234325273.60866194.1551365663-678518112.1480121168) 설정에 따라 다름)로 비 압축 또는 다시 압축 된 형태로 캐시에 저장됩니다. 다운로드 한 번들이 LZ4로 압축되어 있으면 AssetBundle이 압축되어 저장됩니다. 캐시가 가득 차면 Unity는 가장 최근에 사용하지 않은 AssetBundle을 캐시에서 제거합니다.

**가능한 경우 UnityWebRequest를 사용**하거나 Unity 5.2 또는 이전 버전을 사용하는 경우에만 WWW.LoadFromCacheOrDownload를 사용하여 시작하는 것이 좋습니다. 기본 제공 API의 메모리 소비, 캐싱 동작 또는 성능이 특정 프로젝트에 적합하지 않거나 요구 사항을 달성하기 위해 프로젝트가 플랫폼 별 코드를 실행해야하는 경우에만 맞춤형 다운로드 시스템에 투자하십시오.

UnityWebRequest 또는 WWW.LoadFromCacheOrDownload의 사용을 방해 할 수있는 상황의 예 :
* AssetBundle 캐시에 대한 세부적인 제어가 필요할 때
* 프로젝트가 커스텀 압축 전략을 구현해야 할 때
* 프로젝트가 비활성 상태에서 데이터를 스트리밍해야하는 것과 같이 특정 요구 사항을 충족시키기 위해 플랫폼 별 API를 사용하고자하는 경우.
  * 예 : 백그라운드에서 iOS의 백그라운드 작업 API를 사용하여 데이터를 다운로드합니다.
* AssetBundle이 Unity가 적절한 SSL 지원 (예 : PC)을 지원하지 않는 플랫폼에서 SSL을 통해 전달되어야하는 경우.

### 4.2.3. Built-in caching
Unity에는 AssetBundle 버전 번호를 인수로 받아들이는 오버로드를 가진 UnityWebRequest API를 통해 다운로드 된 AssetBundles를 캐시하는 데 사용할 수 있는 **내장 AssetBundle 캐싱 시스템이 있습니다.** 이 번호는 AssetBundle 내에 저장되지 않으며 AssetBundle 시스템에 의해 생성되지 않습니다.

캐싱 시스템은 UnityWebRequest에 전달 된 마지막 버전 번호를 추적합니다. 이 API를 버전 번호와 함께 호출하면 캐싱 시스템은 버전 번호를 비교하여 캐시 된 AssetBundle이 있는지 확인합니다. 이 숫자가 일치하면 시스템은 캐시 된 AssetBundle을 로드합니다. 숫자가 일치하지 않거나 캐시 된 AssetBundle이 없으면 Unity는 새 사본을 다운로드합니다. 이 새로운 사본은 새 버전 번호와 연결됩니다.

캐싱 시스템의 AssetBundles는 다운로드 된 전체 URL이 아니라 파일 이름으로 만 식별됩니다. 즉, 동일한 파일 이름을 가진 AssetBundle을 Content Delivery Network와 같이 여러 위치에 저장할 수 있습니다. 파일 이름이 동일하면 캐싱 시스템은 파일 이름을 동일한 AssetBundle로 인식합니다.

AssetBundle에 버전 번호를 할당하는 적절한 전략을 결정하고 이러한 번호를 UnityWebRequest에 전달하는 것은 각 개별 응용 프로그램에 달려 있습니다. 번호는 CRC 값과 같은 일종의 고유 한 식별자에서 나올 수 있습니다. AssetBundleManifest.GetAssetBundleHash()도 이 용도로 사용할 수 있지만 실제로는 해시 계산이 아닌 추정을 제공하므로 버전 관리에이 함수를 사용하지 않는 것이 좋습니다.

자세한 내용은 Unity 설명서의 [AssetBundles로 패치하기 섹션](https://docs.unity3d.com/Manual/AssetBundles-Patching.html?_ga=2.262563211.60866194.1551365663-678518112.1480121168)을 참조하십시오.

**Unity 2017.1 이후 버전에서는 개발자가 여러 캐시에서 활성 캐시를 선택할 수있게하여 [Caching API](https://docs.unity3d.com/ScriptReference/Caching.html?_ga=2.262038154.60866194.1551365663-678518112.1480121168)가 보다 세부적인 제어를 제공하도록 확장되었습니다.** 이전 버전의 Unity는 캐시 된 항목을 제거하기 위해서만 Caching.expirationDelay와 Caching.maximumAvailableDiskSpace를 수정할 수 있습니다 (이러한 속성은 Cache 클래스의 Unity 2017.1에 남아 있습니다).

expirationDelay는 AssetBundle이 자동으로 삭제되기 전에 경과해야하는 최소 시간(초)입니다. 이 시간 동안 AssetBundle에 액세스하지 않으면 자동으로 삭제됩니다.

maximumAvailableDiskSpace는 캐시가 expirationDelay보다 최근에 사용되지 않은 AssetBundle을 삭제하기 전에 사용할 수있는 로컬 저장소의 공간 (바이트)을 지정합니다. 한도에 도달하면 Unity는 가장 최근에 열어 본 (또는 Caching.MarkAsUsed를 통해 사용 된 것으로 표시 한) 캐시에서 AssetBundle을 삭제합니다. Unity는 새로운 다운로드를 완료하기에 충분한 공간이 생길 때까지 캐시 된 AssetBundle을 삭제할 것입니다.

#### 4.2.3.1. Cache Priming
AssetBundles은 파일 이름으로 식별되기 때문에 응용 프로그램과 함께 제공되는 AssetBundles로 캐시를 "초기화"할 수 있습니다. 이렇게하려면 각 AssetBundle의 초기 또는 기본 버전을 / Assets / StreamingAssets /에 저장하십시오.

응용 프로그램이 처음 실행될 때 Application.streamingAssetsPath에서 AssetBundles를 로드하여 캐시를 채울 수 있습니다. 그런 다음 응용 프로그램은 UnityWebRequest를 정상적으로 호출 할 수 있습니다 (UnityWebRequest는 초기에 StreamingAssets 경로에서 AssetBundles를로드하는데도 사용할 수 있습니다).

### 4.2.3. Custom downloaders
사용자 정의 다운로더를 작성하면 응용 프로그램이 AssetBundle의 다운로드, 압축 해제 및 저장 방법을 완벽하게 제어 할 수 있습니다. 관련 엔지니어링 작업이 중요하지 않으므로이 방법은 대규모 팀에만 권장됩니다. 사용자 정의 다운로더를 작성할 때 다음과 같은 4 가지 주요 고려 사항이 있습니다.
* 다운로드 메커니즘
* 저장 위치
* 압축 유형
* 패치

#### 4.2.3.1. Downloading
대부분의 응용 프로그램에서 HTTP는 AssetBundles를 다운로드하는 가장 간단한 방법입니다. 그러나 HTTP 기반 다운로더 구현은 가장 간단한 작업이 아닙니다. 사용자 다운로더는 과도한 메모리 할당, 과도한 스레드 사용 및 과도한 스레드 깨우기를 방지해야합니다. Unity의 WWW 클래스는 [여기](https://yeonhong.github.io/프로그래밍/assetbundle-fundamentals/)에 철저히 설명 된 이유로 부적합합니다.

사용자 정의 다운로더를 작성할 때 세 가지 옵션이 있습니다.
* C#'s HttpWebRequest and WebClient classes
* Custom native plugins
* Asset store packages

##### 4.2.3.1.1. C# classes
응용 프로그램에 HTTPS / SSL 지원이 필요하지 않은 경우 C #의 WebClient 클래스는 AssetBundles를 다운로드하는 가장 간단한 메커니즘을 제공합니다. 과도한 관리 메모리 할당없이 로컬 스토리지에 직접 모든 파일을 비동기 적으로 다운로드 할 수 있습니다.

WebClient를 사용하여 AssetBundle을 다운로드하려면 클래스의 인스턴스를 할당하고 다운로드 할 AssetBundle의 URL과 대상 경로를 전달합니다. 요청의 매개 변수를 통해 더 많은 제어가 필요한 경우 C #의 HttpWebRequest 클래스를 사용하여 다운로더를 작성할 수 있습니다.
1. HttpWebResponse.GetResponseStream로부터 바이트 스트림을 취득합니다.
2. 스택에 고정 크기 바이트 버퍼를 할당하십시오.
3. 응답 스트림에서 버퍼로 읽습니다.
4. C#의 File.IO API 또는 다른 스트리밍 IO 시스템을 사용하여 디스크에 버퍼를 작성하십시오.

##### 4.2.3.1.2. Asset Store Packages
여러 자산 저장소 패키지는 HTTP, HTTPS 및 기타 프로토콜을 통해 파일을 다운로드하기위한 원시 코드 구현을 제공합니다. Unity에 대한 사용자 정의 원시 코드 플러그인을 작성하기 전에 사용 가능한 자산 저장소 패키지(Asset Store packages)를 평가하는 것이 좋습니다.

##### 4.2.3.1.3. Custom Native Plugins
커스텀 네이티브 플러그인을 작성하는 것은 시간 집약적이지만 Unity에서 데이터를 다운로드하는 가장 유연한 방법입니다. 높은 프로그래밍 시간 요구 사항 및 높은 기술 위험으로 인해이 방법은 응용 프로그램의 요구 사항을 충족시킬 수있는 다른 방법이없는 경우에만 권장됩니다. 예를 들어, 응용 프로그램이 Unity에서 C # SSL 지원없이 플랫폼에서 SSL 통신을 사용해야하는 경우 사용자 지정 네이티브 플러그인이 필요할 수 있습니다.

맞춤 네이티브 플러그인은 일반적으로 대상 플랫폼의 기본 다운로드 API를 래핑합니다. 예로는 iOS에서는 NSURLConnection이, Android에서는 java.net.HttpURLConnection이 있습니다. 이러한 API 사용에 대한 자세한 내용은 각 플랫폼의 기본 설명서를 참조하십시오.

#### 4.2.3.2. Storage
**모든 플랫폼에서 Application.persistentDataPath는 응용 프로그램 실행간에 유지되어야하는 데이터를 저장하는 데 사용해야하는 쓰기 가능한 위치를 가리 킵니다.** 사용자 정의 다운로더를 작성할 때는 Application.persistentDataPath의 하위 디렉토리를 사용하여 다운로드 한 데이터를 저장하는 것이 좋습니다.

Application.streamingAssetPath는 쓰기가 가능하지 않으며 AssetBundle 캐시에 적합하지 않습니다. streamingAssetsPath의 위치는 다음과 같습니다.
* OSX : .app 패키지 내. 쓸 수 없습니다.
* Windows : 설치 디렉토리 (예 : 프로그램 파일); 대개 쓰기 불가
* iOS : .ipa 패키지 내. 쓸 수 없다
* Android : .apk 파일 내. 쓸 수 없다

## 4.3. Asset Assignment Strategies
프로젝트의 자산을 AssetBundle로 분할하는 방법을 결정하는 것은 간단하지 않습니다. 모든 객체를 자신의 AssetBundle에 배치하거나 단일 AssetBundle 만 사용하는 것과 같은 단순한 전략을 채택하는 것이 유혹적이지만 이러한 솔루션에는 다음과 같은 중요한 단점이 있습니다.

* AssetBundles가 너무 적을 경우
  * 런타임 메모리 사용량 증가
  * 로딩 시간 증가
  * 더 큰 다운로드 storage 필요
* AssetBundles가 너무 많을 경우
  * 빌드 시간 증가
  * 개발을 복잡하게 할 수있다.
  * 총 다운로드 시간 증가

주요 결정은 AssetBundles로 객체를 그룹화하는 방법입니다. 주요 전략은 다음과 같습니다.
* 논리적으로 관계가 가까운 것 끼리.
* Object Type
* Concurrent content (같이 있는 컨텐츠)

이러한 그룹 전략에 대한 자세한 내용은 [매뉴얼](https://docs.unity3d.com/Manual/AssetBundles-Preparing.html?_ga=2.25079229.60866194.1551365663-678518112.1480121168)에 나와 있습니다.

## 4.4. Common pitfalls (일반적인 함정)
이 섹션에서는 AssetBundles를 사용하여 프로젝트에 일반적으로 나타나는 몇 가지 문제에 대해 설명합니다.

### 4.5.1. Asset duplication
nity 5의 AssetBundle 시스템은 Object가 AssetBundle에 내장 될 때 Object의 모든 종속성을 발견합니다. 이 종속성 정보는 AssetBundle에 포함될 객체 세트를 결정하는 데 사용됩니다.

AssetBundle에 명시 적으로 할당 된 객체는 해당 AssetBundle에만 내장됩니다. Object의 AssetImporter의 assetBundleName 속성이 비어 있지 않은 문자열로 설정된 경우 객체는 "명시적으로"할당됩니다. 이는 Unity Insightor 또는 Editor 스크립트에서 AssetBundle을 선택하여 Unity Editor에서 수행 할 수 있습니다.

객체는 AssetBundle 빌딩 맵의 일부로 정의하여 AssetBundle에 할당 할 수 있습니다.이 맵은 [AssetBundleBuild](https://docs.unity3d.com/ScriptReference/AssetBundleBuild.html?_ga=2.23093693.60866194.1551365663-678518112.1480121168) 배열을 사용하는 오버로드 된 [BuildPipeline.BuildAssetBundles()](https://docs.unity3d.com/ScriptReference/BuildPipeline.BuildAssetBundles.html?_ga=2.23093693.60866194.1551365663-678518112.1480121168) 함수와 함께 사용됩니다.

**_AssetBundle에 명시 적으로 할당되지 않은 객체는 태그가없는 객체를 참조하는 하나 이상의 객체가 포함 된 모든 AssetBundle에 포함됩니다._**

예를 들어, 두 개의 서로 다른 객체가 두 개의 다른 AssetBundle에 할당되었지만 **둘 다 공통 종속 객체에 대한 참조가 있는 경우 해당 종속 객체가 두 개의 AssetBundle에 복사됩니다.** 중복 된 종속성도 인스턴스화됩니다. 즉, 종속성 객체의 두 사본이 다른 식별자를 가진 다른 객체로 간주됩니다. 그러면 응용 프로그램의 AssetBundle의 전체 크기가 증가합니다. 또한 응용 프로그램에서 부모를 모두 로드하는 경우 두 개의 Object 사본이 메모리에로드됩니다.

이 문제를 해결할 수 있는 몇 가지 방법이 있습니다.

1. 다른 AssetBundle에 내장 된 객체가 종속성을 공유하지 않도록하십시오. 종속성을 공유하는 객체는 종속성을 복제하지 않고 동일한 AssetBundle에 배치 할 수 있습니다.
  * 이 방법은 일반적으로 많은 공유 종속성이있는 프로젝트에서는 실행 가능하지 않습니다. 편리하고 효율적으로 만들기 위해 너무 자주 다시 빌드하고 다시 다운로드해야하는 모놀리식(하나로 뭉쳐져버린) AssetBundles를 생성합니다.
2. 종속성을 공유하는 두 개의 AssetBundle이 동시에로드되지 않도록 AssetBundles를 분할합니다.
  * 이 방법은 레벨 기반 게임과 같은 특정 유형의 프로젝트에서 작동 할 수 있습니다. 그러나 여전히 프로젝트의 AssetBundle 크기가 불필요하게 증가하고 빌드 시간과 로딩 시간이 모두 증가합니다.
3. 모든 종속성 자산이 자체 AssetBundle에 빌드 되었는지 확인하십시오. 이는 중복 자산의 위험을 완전히 없애지만 복잡성을 초래합니다. 응용 프로그램은 AssetBundle 간의 종속성을 추적하고 AssetBundle.LoadAsset API를 호출하기 전에 올바른 AssetBundle이 로드되는지 확인해야합니다.

객체 의존성은 UnityEditor 네임 스페이스에있는 AssetDatabase API를 통해 추적됩니다. 네임 스페이스가 암시 하듯이이 API는 Unity Editor에서만 사용할 수 있으며 런타임에는 사용할 수 없습니다. **AssetDatabase.GetDependencies는 특정 Object 또는 Asset의 모든 즉각적인 종속성을 찾는 데 사용할 수 있습니다. 이러한 종속성에는 고유 한 종속성이있을 수 있습니다. 또한 AssetImporter API를 사용하여 특정 Object가 할당 된 AssetBundle을 쿼리 할 수 있습니다.**

AssetDatabase 및 AssetImporter API를 결합하여 모든 AssetBundle의 직접 또는 간접 종속성이 AssetBundle에 할당되도록하거나 AssetBundle이 AssetBundle에 할당되지 않은 종속성을 공유하지 않도록하는 Editor 스크립트를 작성할 수 있습니다. 자산을 복제하는 데 드는 메모리 비용 때문에 모든 프로젝트에 그러한 스크립트가있는 것이 좋습니다.

### 4.5.2. Sprite atlas duplication
자동으로 생성 된 스프라이트 아틀라스는 스프라이트 아틀라스가 생성 된 스프라이트 객체가 포함 된 AssetBundle에 할당됩니다. 스프라이트 객체가 여러 개의 AssetBundle에 할당 된 경우 스프라이트 아틀라스는 AssetBundle에 할당되지 않고 복제됩니다. 스프라이트 객체가 AssetBundle에 할당되지 않은 경우 스프라이트 아틀라스는 AssetBundle에도 할당되지 않습니다.

스프라이트 아틀라스가 중복되지 않도록하려면 같은 스프라이트 아틀라스에 태그 된 모든 스프라이트가 동일한 AssetBundle에 할당되어 있는지 확인하십시오.

### 4.5.3. Android textures
Android 생태계에서 장치가 대량으로 조각화되어 있기 때문에 텍스처를 여러 가지 형식으로 압축해야하는 경우가 있습니다. 모든 Android 기기가 ETC1을 지원하지만 ETC1은 알파 채널이있는 텍스처를 지원하지 않습니다. 응용 프로그램이 OpenGL ES 2 지원을 필요로하지 않는 경우 문제를 해결하는 가장 깨끗한 방법은 모든 Android OpenGL ES 3 장치에서 지원되는 ETC2를 사용하는 것입니다.

대부분의 응용 프로그램은 ETC2 지원을 사용할 수없는 구형 장치에서 제공해야합니다. 이 문제를 해결하는 한 가지 방법은 Unity 5의 AssetBundle Variant (다른 옵션에 대한 자세한 내용은 Unity의 Android 최적화 가이드 참조)입니다.

AssetBundle Variant를 사용하려면 ETC1을 사용하여 깨끗하게 압축 할 수없는 모든 텍스처를 텍스처 전용 AssetBundle로 분리해야합니다. 다음으로 DXT5, PVRTC 및 ATITC와 같은 공급 업체별 텍스처 압축 형식을 사용하여 Android 생태계의 비 ETC2 가능 슬라이스를 지원하기 위해 이러한 AssetBundle의 변형을 만듭니다. 각 AssetBundle Variant에 대해 포함 된 텍스처의 TextureImporter 설정을 Variant에 적합한 압축 포맷으로 변경합니다.

런타임시 [SystemInfo.SupportsTextureFormat API](https://docs.unity3d.com/ScriptReference/SystemInfo.SupportsTextureFormat.html?_ga=2.263236875.60866194.1551365663-678518112.1480121168)를 사용하여 다양한 텍스처 압축 형식에 대한 지원을 감지 할 수 있습니다. 이 정보는 지원되는 형식으로 압축 된 텍스처가 포함 된 AssetBundle Variant를 선택하고로드하는 데 사용해야합니다.

Android 텍스처 압축 형식에 대한 자세한 내용은 [여기](https://developer.android.com/guide/topics/graphics/opengl.html#textures)를 참조하십시오.

### 4.5.4. iOS file handle overuse(과용)
현재 버전의 Unity는이 문제의 영향을받지 않습니다.

## 4.5. AssetBundle Variants(다양성, 변형)
AssetBundle 시스템의 핵심 기능은 AssetBundle Variant를 도입한 것입니다. Variants의 목적은 응용 프로그램이 런타임 환경에 더 잘 맞게 내용을 조정할 수 있게하는 것입니다. 변형은 객체를 로드하고 인스턴스 ID 참조를 해석 할 때 서로 다른 AssetBundle 파일의 서로 다른 UnityEngine.Object가 "동일한"객체로 나타날 수있게합니다. 개념적으로 두 개의 UnityEngine.Objects가 동일한 파일 GUID 및 로컬 ID를 공유하는 것처럼 보이게 하고 실제 UnityEngine.Object를 식별하여 문자열 Variant ID로 로드합니다.

이 시스템에는 두 가지 기본 사용 사례가 있습니다.
1. 변형은 주어진 플랫폼에 적합한 AssetBundle의 로드를 단순화합니다.
  * 예 : 빌드 시스템은 독립 실행 형 DirectX11 Windows 빌드에 적합한 고해상도 텍스처와 복잡한 쉐이더를 포함하는 AssetBundle을 만들고 안드로이드를위한 더 낮은 품질의 컨텐트를 가진 두 번째 AssetBundle을 만들 수 있습니다. 런타임에 프로젝트의 리소스 로딩 코드는 해당 플랫폼에 적합한 AssetBundle Variant를로드 할 수 있으며 AssetBundle.Load API에 전달 된 Object 이름은 변경할 필요가 없습니다.
2. 변형을 사용하면 응용 프로그램이 동일한 플랫폼에서 다른 하드웨어로 다른 내용을 로드 할 수 있습니다.
  * 이것은 광범위한 모바일 장치를 지원하는 데 핵심입니다. iPhone4는 모든 실제 응용 프로그램에서 최신 iPhone과 동일한 충실도의 콘텐츠를 표시 할 수 없습니다.
  * 안드로이드에서, AssetBundle Variants는 화면 종횡비와 DPI의 장치 간 엄청난 분열을 해결하는 데 사용할 수 있습니다.

### 4.5.1. Limitations (한계)
AssetBundle Variant 시스템의 핵심 한계는 Variants가 별개의 Assets으로 구축되어야한다는 것입니다. 이 제한은 해당 애셋 간의 변형 만 가져 오기 설정 인 경우에도 적용됩니다. 변형 A와 변형 B에 내장 된 텍스처 사이의 유일한 차이가 Unity 텍스처 가져 오기에서 선택한 특정 텍스처 압축 알고리즘 인 경우 변형 A와 변형 B는 여전히 완전히 다른 자산이어야합니다. 즉, 변형 A와 변형 B는 디스크의 개별 파일이어야합니다.

이 제한은 특정 프로젝트의 여러 사본을 소스 제어에 보관해야하므로 대규모 프로젝트 관리를 복잡하게 만듭니다. 개발자가 저작물의 내용을 변경하고자 할 때 모든 저작물 사본을 업데이트해야합니다. 이 문제점에 대한 기본 제공 조치가 없습니다.

대부분의 팀은 자신의 AssetBundle Variant 형식을 구현합니다. 이는 주어진 AssetBundle이 나타내는 특정 변형을 식별하기 위해 파일 이름에 잘 정의 된 접미사가 추가 된 AssetBundles를 작성하여 수행됩니다. 사용자 정의 코드는 이러한 AssetBundle을 빌드 할 때 포함 된 Assets의 임포터 설정을 프로그래밍 방식으로 변경합니다. 일부 개발자는 사용자 정의 시스템을 확장하여 프리팹에 부착 된 구성 요소의 매개 변수를 변경할 수도 있습니다.

## 4.6. Compressed or uncompressed?
AssetBundles를 압축할지 여부는 몇 가지 중요한 고려 사항이 필요합니다.
* 로딩 시간 : 비압축 AssetBundle은 로컬 스토리지 또는 로컬 캐시에서로드 할 때 압축 AssetBundle보다 로드가 빠릅니다.
* 빌드 시간 : 파일 압축시 LZMA 및 LZ4가 매우 느리며 유니티 에디터가 AssetBundles를 순차적으로 처리합니다. AssetBundle이 많은 프로젝트는 압축하는 데 많은 시간을 소비합니다.
* 응용 프로그램 크기 : 응용 프로그램에서 AssetBundle을 제공하는 경우,이를 압축하면 응용 프로그램의 전체 크기가 줄어 듭니다. 또는 AssetBundle을 설치 후 다운로드 할 수 있습니다.
* 메모리 사용 : Unity 5.3 이전에는 Unity의 모든 압축 해제 메커니즘이 압축 된 AssetBundle을 압축 해제하기 전에 메모리에 로드해야 했습니다. 메모리 사용이 중요한 경우 비 압축 또는 LZ4 압축 AssetBundles를 사용하십시오.
* 다운로드 시간 : AssetBundles가 크거나 사용자가 저속 또는 계량 연결을 통해 다운로드하는 것과 같이 대역폭이 제한된 환경에 있는 경우에만 압축이 필요할 수 있습니다. 고속 연결을 통해 수십 메가 바이트의 데이터 만 PC에 전달되는 경우 압축을 생략 할 수 있습니다.

### 4.6.1. Crunch Compression
크런치 압축 알고리즘을 사용하는 DXT 압축 텍스처로 주로 구성된 번들은 압축되지 않은 상태로 만들어야합니다.

## 4.7. AssetBundles and WebGL
WebGL 프로젝트의 모든 AssetBundle 압축 해제 및 로딩은 Unity의 WebGL 내보내기 옵션이 현재 작업자 스레드를 지원하지 않기 때문에 주 스레드에서 수행되어야합니다. AssetBundles의 다운로드는 XMLHttpRequest를 사용하여 브라우저에 위임됩니다. 일단 다운로드되면 압축 된 AssetBundles가 Unity의 메인 스레드에서 압축 해제 될 것이므로 번들의 크기에 따라 Unity 컨텐츠의 실행이 지연됩니다.

Unity는 개발자가 성능 문제를 피하기 위해 작은 자산 묶음을 선호한다고 권장합니다. 이 방법은 대규모 자산 번들을 사용하는 것보다 훨씬 효율적입니다. Unity WebGL은 LZ4 압축 및 압축 해제 자산 번들 만 지원하지만 Unity에 의해 생성 된 번들에는 gzip / brotli 압축을 적용 할 수 있습니다. 이 경우 브라우저에서 파일을 다운로드 할 때 압축 해제되도록 웹 서버를 적절하게 구성해야합니다. 자세한 내용은 여기를 참조하십시오.
