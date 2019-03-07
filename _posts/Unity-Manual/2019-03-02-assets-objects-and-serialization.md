---
layout: single
title: "Assets, Objects and serialization"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Assetbundle
  - Asset 관리
link: https://unity3d.com/learn/tutorials/topics/best-practices/assets-objects-and-serialization?playlist=30089
---
이 장에서는 Unity의 직렬화 시스템의 깊은 내부 구조와 Unity가 Unity Editor와 런타임에서 서로 다른 객체간에 견고한 참조를 유지하는 방법에 대해 설명합니다. 또한 객체와 자산 간의 기술적 인 차이점에 대해서도 설명합니다. 여기에서 다루는 주제는 **Unity에서 자산을 효율적으로로드하고 언로드하는 방법을 이해하는 기본적인 사항입니다.** 적절한 자산 관리는 로딩 시간을 짧게하고 메모리 사용을 낮추는 데 중요합니다.

## 1.1. Inside Assets and Objects
Unity에서 데이터를 올바르게 관리하는 방법을 이해하려면 Unity가 데이터를 식별하고 직렬화하는 방법을 이해하는 것이 중요합니다. 첫 번째 요점은 **Assets**와 **UnityEngine.Object**의 차이입니다.

**Asset**은 Unity 프로젝트의 Assets 폴더에 저장된 디스크상의 파일입니다. 텍스처, 3D 모델 또는 오디오 클립은 일반적인 유형의 자산입니다. 일부 자산에는 자료와 같이 Unity 고유의 형식으로 된 데이터가 포함됩니다. 다른 애셋은 FBX 파일과 같은 기본 형식으로 처리해야합니다.

**UnityEngine.Object** 또는 대문자 'O'가있는 Object는 자원의 특정 인스턴스를 집합적으로 설명하는 일련화 된 데이터 집합입니다. 메시, 스프라이트, AudioClip 또는 AnimationClip과 같이 Unity Engine이 사용하는 모든 유형의 리소스가 될 수 있습니다. 모든 객체는 UnityEngine.Object 기본 클래스의 하위 클래스입니다.

대부분의 Object 유형은 내장되어 있지만 두가지 특수 유형이 있습니다.
1. [ScriptableObject](https://docs.unity3d.com/ScriptReference/ScriptableObject.html?_ga=2.22957628.60866194.1551365663-678518112.1480121168)는 **개발자가 자신의 데이터 유형을 정의** 할 수있는 편리한 시스템을 제공합니다. 이러한 유형은 Unity에서 기본적으로 직렬화 및 직렬화 해제 할 수 있으며 Unity Editor의 Inspector 창에서 조작 할 수 있습니다.
2. [MonoBehaviour](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html?_ga=2.22957628.60866194.1551365663-678518112.1480121168)는 [MonoScript](https://docs.unity3d.com/ScriptReference/MonoScript.html?_ga=2.22957628.60866194.1551365663-678518112.1480121168)에 링크되는 래퍼를 제공합니다. MonoScript는 Unity가 특정 어셈블리 및 네임 스페이스 내에서 특정 스크립팅 클래스에 대한 참조를 유지하는 데 사용하는 내부 데이터 유형입니다. MonoScript에는 실제 실행 코드가 없습니다.

Asset과 Object 간에는 일대-다관계가 있습니다. 즉, **주어진 자산 파일에는 하나 이상의 Object**가 들어 있습니다.

## 1.2. Inter-Object references
모든 UnityEngine.Objects는 다른 UnityEngine.Objects에 대한 참조를 가질 수 있습니다. 이러한 다른 객체는 동일한 Asset 파일 내에 있거나 다른 Asset 파일에서 가져올 수 있습니다. 예를 들어, 머티리얼 객체는 일반적으로 텍스처 객체에 대한 하나 이상의 참조를가집니다. 이러한 텍스처 오브젝트는 일반적으로 하나 이상의 텍스처 애셋 파일 (예 : PNG 또는 JPG)에서 가져옵니다.

이 참조는 직렬화 될 때 두 개의 개별 데이터 즉 파일 GUID와 로컬 ID로 구성됩니다. 파일 GUID는 대상 리소스가 저장된 Asset 파일을 식별합니다. 자산 파일에 여러 객체가 포함될 수 있으므로 유니크한 로컬 ID는 자산 파일 내의 각 객체를 식별합니다.

파일 GUID는 .meta 파일에 저장됩니다. 이러한 .meta 파일은 Unity가 처음으로 자산을 가져 오면 자산과 동일한 디렉토리에 저장됩니다.

위의 식별 및 참조 시스템은 텍스트 편집기에서 볼 수 있습니다. 새로운 Unity 프로젝트를 만들고 편집기 설정을 변경하여 보이는 메타 파일을 표시하고 에셋을 텍스트로 직렬화합니다. 머티리얼을 생성하고 텍스처를 프로젝트로 가져옵니다. 장면의 큐브에 재질을 지정하고 장면을 저장합니다.

텍스트 편집기를 사용하여 재질과 관련된 .meta 파일을 엽니 다. "guid"라고 표시된 줄이 파일의 맨 위에 나타납니다. 이 행은 Material Asset의 File GUID를 정의합니다. 로컬 ID를 찾으려면 텍스트 편집기에서 재질 파일을 엽니 다. Material 객체의 
정의는 다음과 같습니다.

```
--- !u!21 &2100000
Material:
 serializedVersion: 3
 ... more data ...
```
위 예제에서 '&'가 앞에 오는 숫자는 머티리얼의 로컬 ID입니다. 이 Material 객체가 File GUID "abcdefg"로 식별되는 자산 내에있는 경우 Material 객체는 File GUID "abcdefg"와 로컬 ID "2100000"의 조합으로 고유하게 식별 될 수 있습니다.

## 1.3. Why File GUIDs and Local IDs?

Unity의 파일 GUID와 로컬 ID 시스템이 필요한 이유는 무엇입니까? 그 답은 견고성이며 플랫폼에 독립적 인 유연한 워크 플로우를 제공합니다.

파일 GUID는 파일의 특정 위치를 추상화합니다. 특정 파일 GUID가 특정 파일과 연관 될 수있는 한 디스크상의 해당 파일의 위치는 관련성이 없어집니다. 파일을 참조하는 모든 객체를 업데이트하지 않고도 파일을 자유롭게 이동할 수 있습니다.

주어진 Asset 파일에는 여러 개의 UnityEngine.Object 자원이 포함될 수 있기 때문에 (또는 가져 오기를 통해 생성) 각 개별 Object를 모호하지 않게 식별하려면 Local ID가 필요합니다.

자산 파일과 연관된 파일 GUID가 손실되면 해당 자산 파일의 모든 오브젝트에 대한 참조도 손실됩니다. 따라서 .meta 파일은 연결된 파일과 동일한 파일 이름 및 같은 폴더에 저장되어 있어야합니다. Unity는 삭제되거나 잘못 배치 된 .meta 파일을 재생성합니다.

유니티 에디터에는 알려진 파일 GUID에 대한 특정 파일 경로의 맵이 있습니다. 자산이로드되거나 임포트 될 때마다 맵 항목이 기록됩니다. 지도 항목은 자산의 특정 경로를 자산의 파일 GUID에 연결합니다. .meta 파일이 누락되어 Asset 경로가 변경되지 않을 때 Unity Editor가 열려 있으면 Editor는 자산이 동일한 File GUID를 유지하는지 확인할 수 있습니다.

**Unity Editor가 닫힌 상태에서 .meta 파일이 손실되거나 .meta 파일이 자산과 함께 이동하지 않고 Asset 경로가 변경되면 해당 Asset 내의 Objects에 대한 모든 참조가 깨집니다.**

## 1.4. Composite Assets and importers

내부 자산 및 객체 섹션에서 언급했듯이 비 고유 자산 유형은 Unity로 가져와야합니다. 이 작업은 자산 가져 오기 도구를 통해 수행됩니다. 이러한 임포터는 대개 자동으로 호출되지만 [AssetImporter API](https://docs.unity3d.com/ScriptReference/AssetImporter.html?_ga=2.31345208.60866194.1551365663-678518112.1480121168)를 통해 스크립트에 노출됩니다. 예를 들어, [TextureImporter AP](https://docs.unity3d.com/ScriptReference/TextureImporter.html?_ga=2.31345208.60866194.1551365663-678518112.1480121168)I는 PNG 파일과 같은 개별 텍스처 Assets을 가져올 때 사용되는 설정에 대한 액세스를 제공합니다.

가져오기 프로세스의 결과는 하나 이상의 UnityEngine.Objects입니다. 이것들은 Unity Editor에서 상위 애셋 내의 여러 하위 애셋으로 볼 수 있습니다. 예를 들어 스프라이트 아틀라스로 가져온 텍스처 애셋 아래에 중첩 된 여러 스프라이트가 있습니다. 소스 데이터가 동일한 Asset 파일에 저장되므로 이러한 각 객체는 파일 GUID를 공유합니다. 로컬 ID로 가져온 텍스처 애셋 내에서 구분됩니다.

가져오기 프로세스는 원본 자산을 Unity Editor에서 선택한 대상 플랫폼에 적합한 형식으로 변환합니다. 가져오기 프로세스에는 텍스처 압축과 같은 많은 중량 작업이 포함될 수 있습니다. 이것은 종종 시간이 많이 소요되는 프로세스이기 때문에 임포트 된 애셋은 라이브러리 폴더에 캐시되므로 다음 에디터를 시작할 때 애셋을 다시 가져올 필요가 없습니다.

특히 가져 오기 프로세스의 결과는 자산의 파일 GUID의 처음 두 자리에 이름이 지정된 폴더에 저장됩니다. 이 폴더는 Library / metadata / 폴더 안에 저장됩니다. 자산의 개별 개체는 자산의 파일 GUID와 동일한 이름을 가진 단일 이진 파일로 직렬화됩니다.

이 프로세스는 기본 자산이 아닌 모든 자산에 적용됩니다. 네이티브 애셋은 오래걸리는 변환 프로세스 필요 없으며, 재직렬화하지 않습니다.

## 1.5. Serialization and instances

파일 GUID 및 로컬 ID는 견고하지만 GUID 비교는 느리고 실행 시간에 더 많은 성능이 필요한 시스템이 필요합니다. Unity는 내부적으로 파일 GUID와 로컬 ID를 단순한 세션 고유의 정수로 변환하는 캐시를 유지합니다. 이를 **인스턴스 ID**라고하며, 새로운 객체가 캐시에 등록 될 때 단순하고 단조롭게 증가하는 순서로 할당됩니다.

캐시는 객체의 원본 데이터 위치를 정의하는 지정된 인스턴스 ID, 파일 GUID 및 로컬 ID와 메모리에있는 객체의 인스턴스(있는 경우) 간의 매핑을 유지 관리합니다. 이를 통해 UnityEngine.Objects는 서로에 대한 참조를 견고하게 유지할 수 있습니다. 인스턴스 ID 참조를 해석하면 인스턴스 ID가 나타내는로드 된 객체를 신속하게 반환 할 수 있습니다. 대상 객체가 아직로드되지 않은 경우 파일 GUID 및 로컬 ID를 객체의 소스 데이터로 확인할 수 있으므로 Unity가 객체를 적시에로드 할 수 있습니다.

시작할때, 인스턴스 ID 캐시는 리소스 폴더에 포함 된 모든 객체뿐만 아니라 프로젝트에서 즉시 요구되는 모든 객체(즉, 내장 된 장면에서 참조)에 대한 데이터로 초기화됩니다. 런타임에 새 에셋을 가져오거나 AssetBundles에서 객체를 로드 할 때 추가 항목이 캐시에 추가됩니다. 인스턴스 ID 항목은 특정 파일 GUID 및 로컬 ID에 대한 액세스를 제공하는 AssetBundle이 언로드 될 때 캐시에서 제거됩니다. 이 경우 인스턴스 ID, 파일 GUID 및 로컬 ID 간의 매핑이 메모리를 절약하기 위해 삭제됩니다. AssetBundle이 다시로드되면 다시로드 된 AssetBundle에서로드 된 각 객체에 대해 새 인스턴스 ID가 생성됩니다.
> 런타임에 새 Asset을 생성하는 예. a Texture2D Object created in script, like so: var myTexture = new Texture2D(1024, 768);

AssetBundle 언로드의 의미에 대한 더 자세한 설명은 [AssetBundle Usage Patterns](https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-usage-patterns)의 _Loaded Assets 관리 섹션_ 을 참조하십시오.

특정 플랫폼에서 특정 이벤트로 인해 개체에 메모리가 부족할 수 있습니다. 예를 들어 그래픽 애셋은 앱이 일시 중지되면 iOS의 그래픽 메모리에서 언로드 될 수 있습니다. 이러한 객체가 언로드 된 AssetBundle에서 비롯된 경우 Unity는 객체에 대한 소스 데이터를 다시로드 할 수 없습니다. 이 오브젝트에 대한 현존하는 참조는 유효하지 않습니다. 앞의 예제에서 장면은 보이지 않는 메시 또는 자홍색 텍스처를 갖는 것처럼 보일 수 있습니다.

_구현 참고 사항_ : 런타임에 위의 제어 흐름은 말 그대로 정확하지 않습니다. 런타임에 파일 GUID와 로컬 ID를 비교하면 로드가 많은 작업 중에는 성능이 충분하지 않을 수 있습니다. 유니티 프로젝트를 만들 때, 파일 GUID와 로컬 ID는 결정론적으로 간단한 형식으로 매핑됩니다. 그러나 개념은 동일하게 유지되며 파일 GUID 및 로컬 ID에 대한 생각은 런타임 중에 유추 할 수 있습니다. 또한 자산 파일 GUID를 런타임에 쿼리 할 수 없는 이유이기도합니다.

## 1.6. MonoScripts
MonoBehaviour에는 MonoScript에 대한 참조가 있고 MonoScript에는 특정 스크립트 클래스를 찾는 데 필요한 정보가 포함되어 있음을 이해하는 것이 중요합니다. Object 유형에는 스크립트 클래스의 실행 가능 코드가 들어 있지 않습니다.

MonoScript에는 어셈블리 이름, 클래스 이름 및 네임 스페이스라는 세 개의 문자열이 있습니다.

프로젝트를 빌드하는 동안 Unity는 Assets 폴더에있는 느슨한 모든 스크립트 파일을 Mono 어셈블리로 컴파일합니다. Plugins 하위 폴더 외부에있는 C# 스크립트는 Assembly-CSharp.dll에 저장됩니다. Plugins 하위 폴더 내의 스크립트는 Assembly-CSharp-firstpass.dll에 배치됩니다. 또한 Unity 2017.3에는 [사용자 지정 관리되는 어셈블리](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html?_ga=2.266749577.60866194.1551365663-678518112.1480121168)를 정의 할 수있는 기능이 도입되었습니다.

이 어셈블리는 사전 빌드 된 어셈블리 DLL 파일과 함께 Unity 응용 프로그램의 최종 빌드에 포함됩니다. 또한 MonoScript가 참조하는 어셈블리입니다. 다른 리소스와 달리 Unity 애플리케이션에 포함 된 모든 어셈블리는 애플리케이션 시작시로드됩니다.

이 MonoScript Object는 AssetBundle (또는 Scene 또는 프리 팹)이 실제로 AssetBundle, Scene 또는 프리 팹의 MonoBehaviour 구성 요소에 실행 가능 코드를 포함하지 않는 이유입니다. MonoBehaviours가 다른 AssetBundle에 있더라도 다른 MonoBehaviours가 특정 공유 클래스를 참조 할 수 있습니다.

## 1.7. Resource lifecycle
로딩 시간을 줄이고 응용 프로그램의 메모리 사용 공간을 관리하려면 UnityEngine.Objects의 자원 수명주기를 이해하는 것이 중요합니다. 객체는 특정 시간 및 정의 된 시간에 메모리에 로드 / 언로드됩니다.

다음과 같은 경우에 객체가 자동으로 로드됩니다.
1. 해당 객체에 매핑 된 인스턴스 ID가 역 참조됩니다.
2. 개체가 현재 메모리에 로드되지 않았습니다.
3. 객체의 소스 데이터를 찾을 수 있습니다.

객체를 생성하거나 리소스로드 API (예 : AssetBundle.LoadAsset)를 호출하여 객체를 스크립트에 명시적으로로드 할 수도 있습니다. 객체가로드되면 Unity는 각 참조의 파일 GUID와 로컬 ID를 인스턴스 ID로 변환하여 모든 참조를 해결하려고 시도합니다. 

다음 두 가지 조건에 해당되면 인스턴스 ID가 처음으로 참조 될 때 객체가 필요할 때 로드됩니다.
1. 인스턴스 ID가 현재 로드되어 있지 않은 Object를 참조합니다.
2. 인스턴스 ID에 유효한 파일 GUID와 로컬 ID가 캐시에 등록되어 있습니다.

이는 일반적으로 참조 자체가 로드되고 해결 된 직후에 발생합니다.

파일 GUID 및 로컬 ID에 인스턴스 ID가 없거나 언로드 된 객체가 있는 인스턴스 ID가 잘못된 파일 GUID 및 로컬 ID를 참조하는 경우 참조는 유지되지만 실제 객체는로드되지 않습니다. 이는 Unity Editor에서 "(Missing)"참조로 나타납니다. 실행중인 응용 프로그램이나 장면 뷰에서 "(Missing)"객체는 유형에 따라 다른 방법으로 표시됩니다. 예를 들어, 메시는 보이지 않는 것처럼 보이지만 텍스처는 마젠타처럼 보일 수 있습니다.
> 객체가 언로드되지 않고 런타임에 메모리에서 제거되는 가장 일반적인 경우는 Unity가 그래픽 컨텍스트의 제어권을 잃을 때 발생합니다. **모바일 앱이 일시 중지되어 앱이 백그라운드로 강제 실행되는 경우 이러한 현상이 발생할 수 있습니다.** 이 경우 모바일 OS는 일반적으로 GPU 메모리에서 모든 그래픽 리소스를 제거합니다. 응용 프로그램이 포 그라운드로 돌아 오면 유니티는 장면 렌더링을 다시 시작하기 전에 필요한 모든 텍스처, 쉐이더 및 메쉬를 GPU에 다시로드해야합니다.

Object는 세 가지 특정 시나리오에서 언로드 됩니다.

* unused Asset cleanup이 발생하면 Object가 자동으로 언로드됩니다. 이 프로세스는 장면이 파괴적으로 변경되면 (즉, [SceneManager.LoadScene](https://docs.unity3d.com/ScriptReference/SceneManagement.SceneManager.html?_ga=2.263236875.60866194.1551365663-678518112.1480121168)이 non-additively하게 호출 될 때) 또는 스크립트가 [Resources.UnloadUnusedAssets API](https://docs.unity3d.com/ScriptReference/Resources.UnloadUnusedAssets.html?_ga=2.263236875.60866194.1551365663-678518112.1480121168)를 호출 할 때 자동으로 트리거됩니다. 이 프로세스는 참조되지 않은 객체 만 언로드합니다. Mono 변수가 Object에 대한 참조를 보유하지 않고 Object에 대한 참조를 보유하는 다른 활성 객체가없는 경우에만 객체가 언로드됩니다. 또한 [HideFlags.DontUnloadUnusedAsset](https://docs.unity3d.com/ScriptReference/HideFlags.DontUnloadUnusedAsset.html?_ga=2.263236875.60866194.1551365663-678518112.1480121168) 및 [HideFlags.HideAndDontSave](https://docs.unity3d.com/ScriptReference/HideFlags.HideAndDontSave.html?_ga=2.253774479.60866194.1551365663-678518112.1480121168)로 표시된 항목은 언로드되지 않습니다.
* Resources 폴더에서 가져온 객체는 Resources.UnloadAsset API를 호출하여 명시 적으로 언로드 할 수 있습니다. 이러한 개체의 인스턴스 ID는 유효하며 유효한 파일 GUID 및 LocalID 항목을 계속 포함합니다. **Mono 변수 나 다른 Object가 Resources.UnloadAsset으로 언로드 된 Object에 대한 참조를 보유하고 있으면 해당 라이브 참조 중 하나가 참조 해제되는 즉시 해당 객체가 다시로드됩니다.**
* AssetBundle에서 가져온 객체는 AssetBundle.Unload (true) API를 호출 할 때 자동으로 즉시 언로드됩니다. 이렇게하면 개체의 인스턴스 ID의 파일 GUID와 로컬 ID가 무효화되고 언로드 된 개체에 대한 실제 참조는 "(누락 됨)"참조가됩니다. C# 스크립트에서 언로드 된 객체의 메서드 나 속성에 액세스하려고하면 NullReferenceException이 발생합니다.

AssetBundle.Unload (false)가 호출되면 언로드 된 AssetBundle에서 가져온 라이브 객체는 소멸되지 않지만 Unity는 해당 인스턴스 ID의 파일 GUID 및 로컬 ID 참조를 무효화합니다. 나중에 메모리에서 언로드되고 언로드 된 객체에 대한 실제 참조가 남아있는 경우 Unity가 이러한 객체를 (자동으로)다시로드하는 것은 불가능합니다.

## 1.8. Loading large hierarchies

**Prefab 직렬화와 같이 Unity GameObjects의 계층 구조를 직렬화 할 때, 전체 계층 구조가 완전히 직렬화된다는 것을 기억하는 것이 중요합니다.** 즉, 계층 구조의 모든 GameObject 및 Component는 일련 화 된 데이터에서 개별적으로 표현됩니다. 이것은 GameObjects의 계층 구조를 로드하고 인스턴스화하는 데 필요한 시간에 흥미로운 영향을 미칩니다.

GameObject 계층을 만들 때 CPU 시간은 여러 가지 방법으로 사용됩니다.
* 소스 데이터 읽기 (저장소, AssetBundle, 다른 GameObject 등)
* 새로운 Transforms 사이의 부모 - 자식 관계 설정하기
* 새 GameObjects 및 구성 요소 인스턴스화
* 메인 스레드에서 새 GameObjects 및 구성 요소 깨우기

후자의 세가지의 시간 비용은 계층 구조가 기존 계층 구조에서 복제되는지 또는 저장 영역에서로드되는지 여부에 관계없이 일반적으로 변하지 않습니다. 그러나 **소스 데이터를 읽는 시간은 계층 구조에 직렬화 된 구성 요소 및 게임 객체 수가 선형 적으로 증가하며 데이터 소스의 속도도 곱해집니다.**

현재 모든 플랫폼에서 스토리지장치에서 데이터를 로드하는 대신 메모리의 다른 위치에서 데이터를 읽는 것이 훨씬 빠릅니다. 또한, 사용 가능한 저장 매체의 성능 특성은 플랫폼에 따라 크게 다릅니다. 따라서 스토리지가 느린 플랫폼에서 프리팹을 로드 할 때 스토리지에서 프리팹의 직렬화 된 데이터를 읽는 데 소요되는 시간이 프리 팹을 인스턴스화하는 데 소요되는 시간을 빠르게 초과 할 수 있습니다. 즉, **로딩 작업의 비용은 저장소 I/O 시간에 바인딩됩니다.**

앞에서 언급했듯이, **하나로 크게만든 프리팹을 직렬화하면 모든 GameObject 및 구성 요소의 데이터가 별도로 직렬화되므로 데이터가 중복 될 수 있습니다.** 예를 들어 30 개의 동일한 요소가 포함 된 UI 화면에는 동일한 요소가 30 번 일련 화되어 2진수 데이터의 큰 얼룩이 생성됩니다. 로드 할 때, 30 개의 복제 요소 각각에있는 모든 GameObjects 및 Components의 데이터는 새로 인스턴스화 된 Object로 전송되기 전에 디스크에서 읽혀 져야합니다. 이 파일 읽기 시간은 대형 조립식을 인스턴스화하는 전체 비용의 중요한 요인입니다. 대규모 계층 구조는 모듈러 청크로 인스턴스화 한 다음 런타임에 함께 결합해야합니다.