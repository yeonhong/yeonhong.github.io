---
layout: single
title: "AssetBundle fundamentals"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Assetbundle
link: https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-fundamentals?playlist=30089
---
이 장에서는 AssetBundles에 대해 설명합니다. AssetBundles가 구축되는 기본 시스템과 AssetBundles와 상호 작용하는 데 사용되는 핵심 API를 소개합니다. 특히, AssetBundles 자체의로드 및 언로드와 AssetBundle에서 특정 Asset 및 Objects로드 및 언로드에 대해 설명합니다.

## 3.1. Overview
AssetBundle 시스템은 Unity가 인덱싱하고 직렬화 할 수있는 보관 형식으로 하나 이상의 파일을 저장하는 방법을 제공합니다. AssetBundles는 설치 후 비 코드 컨텐츠를 전달하고 업데이트하기위한 Unity의 기본 도구입니다. 이를 통해 개발자는 더 작은 응용 프로그램 패키지를 제출하고 런타임 메모리 압력을 최소화하며 최종 사용자의 장치에 맞게 최적화 된 내용을 선택적으로로드 할 수 있습니다.

AssetBundles의 작동 방식을 이해하는 것은 모바일 장치 용 Unity 프로젝트를 성공적으로 구축하는 데 필수적입니다. AssetBundle 내용에 대한 전체 설명을 보려면 [AssetBundle 설명서](https://docs.unity3d.com/Manual/AssetBundlesIntro.html?_ga=2.225461661.60866194.1551365663-678518112.1480121168)를 검토하십시오.

## 3.2. AssetBundle layout
요약하면 AssetBundle은 헤더와 데이터 세그먼트의 두 부분으로 구성됩니다.

헤더에는 식별자, 압축 유형 및 매니페스트와 같은 AssetBundle에 대한 정보가 들어 있습니다. 매니페스트는 개체의 이름으로 키순 조회 테이블입니다. 각 엔트리는 AssetBundle의 데이터 세그먼트 내에서 주어진 객체가 어디에서 발견 될 수 있는지를 나타내는 바이트 인덱스를 제공합니다. 대부분의 플랫폼에서이 찾아보기 테이블은 균형 잡힌 검색 트리로 구현됩니다. 특히 Windows 및 OSX에서 파생 된 플랫폼 (iOS 포함)은 red-black 트리를 사용합니다. 따라서 **AssetBundle 내의 Assets 수가 증가하면 매니페스트를 구성하는 데 필요한 시간이 선형 적으로 증가합니다.**

데이터 세그먼트에는 AssetBundle에서 Assets을 serialize하여 생성 된 원시 데이터가 포함됩니다. LZMA가 압축 스키마로 지정된 경우 모든 일련 화 된 에셋의 전체 바이트 배열이 압축됩니다. LZ4가 대신 지정되면 별도의 Assets에 대한 바이트가 개별적으로 압축됩니다. 압축을 사용하지 않으면 데이터 세그먼트는 원시 바이트 스트림으로 유지됩니다.

일반적으로 Unity는 **동일한 AssetBundle에서 후속로드 요청에 대한 로딩 성능을 개선하기 위해 AssetBundle의 압축되지 않은 복사본을 캐싱합니다.**

## 3.3. Loading AssetBundles
AssetBundles은 4 개의 서로 다른 API를 통해로드 할 수 있습니다. 이 네 가지 API의 동작은 두 가지 기준에 따라 다릅니다.
1. AssetBundle이 LZMA 압축인지, LZ4 압축인지 또는 압축 해제인지 여부
2. AssetBundle이로드되는 플랫폼

APIs
* AssetBundle.LoadFromMemory (비동기방식을 선택할 수 있음)
* AssetBundle.LoadFromFile (비동기방식을 선택할 수 있음)
* UnityWebRequest의 DownloadHandlerAssetBundle
* WWW.LoadFromCacheOrDownload (on Unity 5.6 or older)

### 3.3.1 AssetBundle.LoadFromMemory(Async)
_Unity's recommendation is **not** to use this API._

[AssetBundle.LoadFromMemoryAsync](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadFromMemoryAsync.html?_ga=2.192382253.60866194.1551365663-678518112.1480121168)는 관리 코드 바이트 배열 (C#의 byte[])에서 AssetBundle을로드합니다. 항상 관리되는 코드 바이트 배열의 소스 데이터를 새로 할당 된 인접한 기본 메모리 블록으로 복사합니다. AssetBundle이 LZMA 압축이면 복사하는 동안 AssetBundle을 압축 해제합니다. 비압축 및 LZ4 압축 AssetBundle은 그대로 복사됩니다.

이 API가 소비하는 **최대 메모리 양은 AssetBundle의 두 배가됩니다.** 하나는 API에 의해 생성 된 기본 메모리에있는 사본이고, 다른 하나는 관리되는 바이트 배열에있는 사본이 API에 전달 된 것입니다. 따라서이 API를 통해 생성 된 AssetBundle에서로드 된 애셋은 관리 코드 바이트 배열에서 한 번, AssetBundle의 원시 메모리 복사본에서 한 번, 애셋 자체의 GPU 또는 시스템 메모리에서 세 번째로 메모리에서 세 번 복제됩니다 .

### 3.3.2. AssetBundle.LoadFromFile(Async)
[AssetBundle.LoadFromFile](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadFromFile.html?_ga=2.226452765.60866194.1551365663-678518112.1480121168)은 **압축되지 않은 또는 LZ4 압축** AssetBundle을 하드 디스크 나 SD 카드와 같은 로컬 저장소에서로드하기위한 고효율 API입니다.

데스크톱 독립형, 콘솔 및 모바일 플랫폼에서 API는 AssetBundle의 **헤더만 로드**하고 나머지 데이터는 디스크에 남겨 둡니다. AssetBundle의 객체는 로딩 메소드 (예 : AssetBundle.Load)가 호출되거나 InstanceID가 참조 해제 될 때 주문형으로 로드됩니다. **이 시나리오에서는 초과 메모리가 소모되지 않습니다.** 유니티 에디터에서, API는 AssetBundle 전체를 디스크에로드하고 AssetBundle.LoadFromMemoryAsync가 사용 된 것처럼 메모리에로드합니다. 이 API는 프로젝트가 Unity Editor에서 프로파일 링 된 경우 AssetBundle 로딩 중에 메모리 스파이크가 나타날 수 있습니다. 이는 기기의 성능에 영향을주지 않아야하며 이러한 스파이크는 치료 조치를 취하기 전에 기기에서 다시 테스트해야합니다.

### 3.3.3. AssetBundleDownloadHandler
[UnityWebRequest API](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html?_ga=2.224420253.60866194.1551365663-678518112.1480121168)를 통해 개발자는 Unity가 다운로드 한 데이터를 처리하는 방법을 정확하게 지정할 수 있으며 개발자는 불필요한 메모리 사용을 제거 할 수 있습니다. UnityWebRequest를 사용하여 AssetBundle을 다운로드하는 가장 간단한 방법은 [UnityWebRequest.GetAssetBundle](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.GetAssetBundle.html?_ga=2.224420253.60866194.1551365663-678518112.1480121168)입니다.

이 가이드의 목적에 따라 관심 클래스는 DownloadHandlerAssetBundle입니다. 작업자 스레드를 사용하여 다운로드 된 데이터를 고정 크기 버퍼로 스트리밍 한 다음 다운로드 핸들러가 구성된 방식에 따라 버퍼링 된 데이터를 임시 저장소 또는 AssetBundle 캐시로 스풀링합니다. 이러한 모든 작업은 원시 코드에서 발생하므로 관리 힙을 확장 할 위험이 없습니다. 또한이 다운로드 핸들러는 다운로드 한 모든 바이트의 원시 코드 복사본을 유지하지 않으므로 AssetBundle을 다운로드 할 때의 메모리 오버 헤드가 줄어 듭니다.

LZMA 압축 AssetBundles는 다운로드 중에 압축 해제되며 LZ4 압축을 사용하여 캐시됩니다. 이 동작은 [Caching.CompressionEnabled](https://docs.unity3d.com/ScriptReference/Caching-compressionEnabled.html?_ga=2.187672239.60866194.1551365663-678518112.1480121168)를 설정하여 변경할 수 있습니다.

다운로드가 완료되면 다운로드 핸들러의 [assetBundle 속성](https://docs.unity3d.com/ScriptReference/Networking.DownloadHandlerAssetBundle-assetBundle.html?_ga=2.187672239.60866194.1551365663-678518112.1480121168)은 다운로드 된 AssetBundle에서 AssetBundle.LoadFromFile이 호출 된 것처럼 다운로드 된 AssetBundle에 대한 액세스를 제공합니다.

캐싱 정보가 UnityWebRequest 객체에 제공되고 요청 된 AssetBundle이 Unity의 캐시에 이미 존재하면 AssetBundle을 즉시 사용할 수있게되며이 API는 AssetBundle.LoadFromFile과 동일하게 작동합니다.

### 3.3.4. WWW.LoadFromCacheOrDownload
* 참고 : Unity 2017.1부터 WWW.LoadFromCacheOrDownload는 UnityWebRequest를 간단하게 둘러 쌉니다. 따라서 Unity 2017.1 이상을 사용하는 개발자는 UnityWebRequest로 마이그레이션해야합니다. **WWW.LoadFromCacheOrDownload는 향후 릴리스에서 더 이상 사용되지 않습니다.**

### 3.3.5. Recommendations
가능한 경우 일반적으로 **AssetBundle.LoadFromFile을 사용해야합니다.** 이 API는 속도, 디스크 사용 및 런타임 메모리 사용 측면에서 가장 효율적입니다.

AssetBundles를 다운로드하거나 패치해야하는 프로젝트의 경우 Unity 5.3 이상을 사용하는 프로젝트에는 UnityWebRequest를 사용하고 Unity 5.2 이상을 사용하는 프로젝트에는 WWW.LoadFromCacheOrDownload를 사용하는 것이 좋습니다. 다음 장의 배포 섹션에서 자세히 설명했듯이 프로젝트 설치 프로그램에 포함 된 번들로 AssetBundle 캐시를 준비 할 수 있습니다.

고유 한 특정 캐싱 또는 다운로드 요구 사항이 필요한 상당량의 엔지니어링 팀이있는 프로젝트의 경우 사용자 정의 다운로더를 고려할 수 있습니다. 커스텀 다운로더를 작성하는 것은 간단한 엔지니어링 작업이며 커스텀 다운로더는 AssetBundle.LoadFromFile과 호환 가능해야합니다. 자세한 내용은 다음 장의 배포 섹션을 참조하십시오.

## 3.4. Loading Assets From AssetBundles
UnityEngine.Objects는 AssetBundle에서로드 할 수 있습니다.이 API는 모두 AssetBundle 객체에 연결됩니다.이 클래스에는 동기 및 비동기 변형이 모두 있습니다.

* [LoadAsset](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAsset.html?_ga=2.259017229.60866194.1551365663-678518112.1480121168) ([LoadAssetAsync](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAssetAsync.html?_ga=2.259017229.60866194.1551365663-678518112.1480121168))
* [LoadAllAssets](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAllAssets.html?_ga=2.56461868.60866194.1551365663-678518112.1480121168) ([LoadAllAssetsAsync](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAllAssetsAsync.html?_ga=2.56461868.60866194.1551365663-678518112.1480121168))
* [LoadAssetWithSubAssets](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAssetWithSubAssets.html?_ga=2.56461868.60866194.1551365663-678518112.1480121168) ([LoadAssetWithSubAssetsAsync](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAssetWithSubAssetsAsync.html?_ga=2.56461868.60866194.1551365663-678518112.1480121168))

이러한 API의 Async 버전은 항상 적어도 하나의 프레임만큼 sync 버전보다 빠릅니다.

Async방식의 Load는 타임 슬라이스 한도까지 프레임당 여러객체를 로드합니다. 이 동작에 대한 근본적인 기술적 이유에 대해서는 아래에 나오는 Low-level loading details 섹션을 참조하십시오. 

LoadAllAssets는 여러 개의 독립적 인 UnityEngine.Objects를 로드 할 때 사용해야합니다. AssetBundle 내의 대부분 또는 모든 객체를 로드해야하는 경우에만 사용해야합니다. LoadAllAssets는 다른 두 API와 비교하여 LoadAssets에 대한 개별 호출보다 약간 빠릅니다. 따라서 로드 할 자산의 수가 많지만 AssetBundle의 66% 미만을 한 번에 로드해야하는 경우 AssetBundle을 여러 개의 작은 번들로 분할하고 LoadAllAssets를 사용하는 것이 좋습니다.

LoadAssetWithSubAssets는 임베디드 된 애니메이션이 포함 된 FBX 모델 또는 내부에 여러 스프라이트가 포함 된 스프라이트 아틀라스와 같이 여러 내장 객체가 포함 된 복합 애셋을 로드 할 때 사용해야합니다. 모두 로드 해야하는 객체가 동일한 애셋에서 왔지만 다른 관련없는 많은 객체가있는 AssetBundle에 저장되어있는 경우이 API를 사용하십시오.

그외 경우에는 LoadAsset 또는 LoadAssetAsync를 사용하십시오.

### 3.4.1. Low-level loading details
UnityEngine.Object 로딩은 주 스레드에서 수행됩니다. 개체의 데이터는 작업자 스레드의 저장소에서 읽습니다. Unity 시스템의 스레드에 민감한 부분 (스크립팅, 그래픽)을 만지지 않는 항목은 작업자 스레드에서 변환됩니다. 예를 들어, VBO는 메쉬에서 생성되고 텍스처는 압축 해제됩니다.

Unity 5.3부터는 객체로드가 병렬처리 되었습니다. 여러 개체가 deserialized, 처리 및 작업자 스레드에 통합됩니다. 객체로드가 끝나면 Awake 콜백이 호출되고 객체는 다음 프레임 동안 나머지 Unity 엔진에서 사용할 수있게됩니다.

동기 적 AssetBundle.Load 메서드는 객체로드가 완료 될 때까지 주 스레드를 일시 중지합니다. 또한 객체로드가 타임 슬라이싱되어 객체 통합이 특정 시간 (밀리 초)의 프레임 시간을 초과하지 않도록합니다. 밀리 초 수는 속성에 의해 설정됩니다.

Application.backgroundLoadingPriority :
* ThreadPriority.High : 프레임 당 최대 50 밀리 초
* ThreadPriority.Normal : 프레임 당 최대 10 밀리 초
* ThreadPriority.BelowNormal : 프레임 당 최대 4 밀리 초
* ThreadPriority.Low : 프레임 당 최대 2 밀리 초입니다.

Unity 5.2 이후부터는 객체 로딩을위한 프레임 시간 제한에 도달 할 때까지 여러 객체가로드됩니다. 다른 모든 요소가 같다고 가정하면 자산로드 API의 비동기 변형은 비동기 호출 발행과 객체가 엔진에서 사용 가능하게되는 사이의 최소 한 프레임 지연으로 인해 동등한 동기 버전보다 항상 완료하는 데 더 오래 걸립니다.

### 3.4.2. AssetBundle dependencies
AssetBundle 사이의 의존성은 런타임 환경에 따라 두 개의 서로 다른 API를 사용하여 자동으로 추적됩니다. 유니티 편집기에서 AssetBundle 의존성은 [AssetDatabase API](https://docs.unity3d.com/ScriptReference/AssetDatabase.html?_ga=2.225412509.60866194.1551365663-678518112.1480121168)를 통해 조회 할 수 있습니다. AssetBundle 할당 및 종속성은 [AssetImporter](https://docs.unity3d.com/ScriptReference/AssetImporter.html?_ga=2.225412509.60866194.1551365663-678518112.1480121168) API를 통해 액세스하고 변경할 수 있습니다. 런타임시 Unity는 ScriptableObject 기반 [AssetBundleManifest API](https://docs.unity3d.com/ScriptReference/AssetBundleManifest.html?_ga=2.225412509.60866194.1551365663-678518112.1480121168)를 통해 AssetBundle 빌드 중에 생성 된 종속성 정보를 로드하는 선택적 API를 제공합니다.

AssetBundle은 부모 AssetBundle의 UnityEngine.Objects 중 하나 이상이 다른 AssetBundle의 UnityEngine.Objects 중 하나 이상을 가리킬 때 다른 AssetBundle에 종속됩니다. 개체 간 참조에 대한 자세한 내용은 Assets, Objects 및 Serialization 문서의 개체 간 참조 섹션을 참조하십시오.

AssetBundles는 해당 아티클의 Serialization 및 instances 섹션에서 설명한대로 AssetBundle에 포함 된 각 Object의 FileGUID 및 LocalID로 식별되는 소스 데이터의 소스 역할을합니다.

인스턴스 ID가 처음으로 역 참조 될 때 객체가로드되고 AssetBundle이로드 될 때 객체에 유효한 인스턴스 ID가 할당되므로 AssetBundles이로드되는 순서는 중요하지 않습니다. 대신 **Object 자체를 로드하기 전에 Object의 종속성이 포함 된 모든 AssetBundle을 로드하는 것이 중요합니다. 상위 AssetBundle이 로드 될 때 Unity는 하위 AssetBundles을 자동으로 로드하지 않습니다.**

예:

Material A가 Texture B를 참조한다고 가정합니다. Material A는 AssetBundle1에 패키지화되고 Texture B는 AssetBundle2에 패키지화됩니다.

![no-alignment](https://unity3d.com/sites/default/files/learn/ab1.jpg)

이 경우 AssetBundle1에서 Material A를로드하기 전에 AssetBundle2를 로드해야합니다.

이것은 AssetBundle2가 AssetBundle1보다 먼저 로드되어야 함을 의미하지 않으며, AssetBundle2에서 명시적으로로드되어야 함을 의미하지는 않습니다. AssetBundle1에서 Material A를로드하기 전에 AssetBundle2를 로드하면 충분합니다.

**그러나 Unity는 AssetBundle 1이로드 될 때 AssetBundle 2를 자동으로 로드하지 않습니다. 이 작업은 스크립트 코드에서 수동으로 수행해야합니다.**

AssetBundle 종속성에 대한 자세한 내용은 [매뉴얼 페이지](https://docs.unity3d.com/Manual/AssetBundles-Dependencies.html?_ga=2.220563103.60866194.1551365663-678518112.1480121168)를 참조하십시오.

### 3.4.3. AssetBundle manifests
BuildPipeline.BuildAssetBundles API를 사용하여 AssetBundle 빌드 파이프 라인을 실행할 때 Unity는 각 AssetBundle의 의존성 정보를 포함하는 Object를 직렬화합니다. 이 데이터는 AssetBundleManifest 유형의 단일 Object를 포함하는 별도의 AssetBundle에 저장됩니다.

이 AssetBundle이 빌드되는 상위 디렉토리와 동일한 이름을 가진 AssetBundle에 Asset이 저장됩니다. 프로젝트가 (projectroot) / build / Client /에있는 폴더로 AssetBundles를 빌드하면 매니페스트가 포함 된 AssetBundle은 (projectroot) /build/Client/Client.manifest로 저장됩니다.

매니페스트가 포함 된 AssetBundle은 다른 AssetBundle처럼 로드,캐싱 및 언로드 할 수 있습니다.

AssetBundleManifest 객체 자체는 [GetAllAssetBundles API](https://docs.unity3d.com/ScriptReference/AssetBundleManifest.GetAllAssetBundles.html?_ga=2.25128509.60866194.1551365663-678518112.1480121168)를 제공하여 매니페스트와 동시에 빌드 된 모든 AssetBundle과 특정 AssetBundle의 종속성을 쿼리하는 두 가지 메소드를 나열합니다.
* [AssetBundleManifest.GetAllDependencies](https://docs.unity3d.com/ScriptReference/AssetBundleManifest.GetAllDependencies.html?_ga=2.25128509.60866194.1551365663-678518112.1480121168)는 AssetBundle의 직계 하위 요소, 자식 요소 등의 종속성을 포함하는 AssetBundle의 모든 계층 적 종속성을 반환합니다.
* [AssetBundleManifest.GetDirectDependencies](https://docs.unity3d.com/ScriptReference/AssetBundleManifest.GetDirectDependencies.html?_ga=2.25128509.60866194.1551365663-678518112.1480121168)는 AssetBundle의 직접 자식 만 반환합니다.

이 두 API는 모두 문자열 배열을 할당합니다. 따라서 응용 프로그램 수명의 성능에 민감한 부분에 사용하지 말고 아껴서 사용해야합니다.

### 3.4.4. Recommendations
**대부분의 경우 플레이어가 주요 게임 레벨이나 세계와 같이 응용 프로그램의 성능이 중요한 영역에 들어가기 전에 가능한 많은 객체를 로드하는 것이 좋습니다.** 이는 특히 로컬 스토리지에 대한 액세스가 느리고 재생 시간에 오브젝트로드 및 언로드의 메모리 변동으로 인해 가비지 수집기가 트리거 될 수있는 모바일 플랫폼에서 특히 중요합니다.

자주 객체를 로드/언로드 해야하는 프로젝트의 경우, 객체 및 AssetBundle 언로드에 대한 자세한 내용은 AssetBundle 사용 패턴의 로드된 자산 관리 단원을 참조하십시오.
