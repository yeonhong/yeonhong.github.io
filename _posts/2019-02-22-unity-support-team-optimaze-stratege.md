---
layout: single
title: "Unity-Support-Team-Optimaze-Strategy-2019.2"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Optimaze
---

유니티 기술팀에서 공유해준 최적화 전략관련 내용 정리. (2019.2)

## 성능프로파일러
### 쓰로틀링 (과열현상으로 인한)
성능이 높아 연산을 처리하다보니 과열현상이 생길 수 밖에 없다.
하여 과열을 방지하기 위해 성능제한을 함 (쓰로틀링)
최신기기일수록 더 빠르게 온다.

### Application.targetframerate
Application.targetframerate 의 적정한 사항. 일반적으로 40.
기기마다 초기값이 다름. 세팅을 안하면 기본값으로 잡히게됨. 명시적으로 잡아야.

### Tire 세팅
Tier세팅은 기기의 환경에 따라 자동으로 설정함.
Tier1 아이폰5 이하, 
Tier2 opengl es 3.0 이상, 아이폰5s이상
(그래픽퀄리티는 사람이 다루는것)

### Custom profiler tags.
Profiler.beginsample("name");
Profiler.endsample();

Unity proriler는 cpu위주. Gpu profiler는 디바이스에서는 잘 안됨.

### Fillrate병목 
해상도/프래그먼트쉐이더 복잡도/오버드로우 발생시. 
> player에 resolution세팅에서 resolution sacling mode에서 fixed dpi로 완화가능.

### 기타
오버드로우 문제의 경우.
> 파티클의 경우가 많음. 완화하도록 노력해야함.
Postprocessing.
> 블룸등.. 사용시 유의.
	
업스케일링 샘플링. 
> 씬내녀석만 줄이고 ui는 정규로 보이도록. 하는방식.
	
Texture 병목
> 퀄리티 > 렌더링 > texture 퀄리티에서 resolution을 조정해서 판단가능.

메모리 프로파일러. (기본 프로파일러 메뉴 말고)
> 내장 안되어 있으면 Github혹은 패키지매니저에서 받도록.

## 성능최적화 방안 (유니티 프로젝트 리뷰를 통한)

### Project settings
time에서 fixed timestep의 설정을 맞춰줘야.
* 물리엔진업데이트를 때문.
* Rigidbody가 하나라도 있으면 동작함.
Physics
* 컬리전 메트릭스로 레이어로 지정하여 좀더 향상가능.
Player.
* Static여부를 고려해야. 화면에 보이는 영역에 따라.
* dynamic batching(은 끄는게 맞음)
* Gpu skinning = false
  * Cpu와 속도차이가 없음. 굳이 이점이 없음
Quality
* AA는 끄도록.
  * V-sync count는 ios는 off해도 동작하지만 안드로이드는 동작하고 있어서
    * 프레임에 맞춰서 조정이 필요.
Lighting
* 디폴트 스카이박스의 경우 밤낮이 있어서 쓸데없이 동작하고 있어 바꿔야.
* Realtime lighting 의 경우도 조정이 필요.


### Asset imports 관련
#### Texture.
* Mipmap 끄는것.
* Read/write enable 끄기.
* Format.
  * ETC2 체크시 미지원기기에서는 비압축상태로 변환됨.

#### Model
* Read/write enable 끄기.
* Mesh compress.
  * 디스크사이즈 조정에만,
  * 정밀도를 조정함. 메쉬의 이음새가 이상해질수 있음.
  * 단독메쉬에서만 사용하기.
* Animation type도 챙겨서 안쓰는건 none으로.

####Audio
* Force To Mono로 사용해야.
* 혹은 모노로 어셋을 제작하도록.
* 압축포멧은 mp3는 지양하도록. Vorbis가 약간 낫다?
* Loadtype
  * 200kb이하는 무압축
  * 1mb이하 압축
  * 1mb이상 스트리밍.

유니티에서 mp3를 돌릴때 ios에서는 cpu로 돌리고 있음.

### Memory 관련
* 단편화문제.
  * 게임데이터를 json으로 가져오는데. (string)
  * 재사용이 안되면 메모리 사용량이 늘어날 수 있다.
  * 씬사이에 빈씬 넣기. 
    * 메모리풀사이즈를 관리해야. (최대메모리와 관련)
    * 유니티는 자체 메모리풀을 사용함…….
* iOS 메모리
  * 통합메모리 구조를 가지고 있음. 
  * Dirty size :: 앱이 실행되서 메모리에 로드됬을때 모자르면 반환시키는데 안되는 항목, 하여 크래쉬 할때 셧다운 되는 원인. 이것을 줄이는게 중요하다.
  * 아트어셋들은 100% 더티메모리에 들어감..
  * 아래는 최대 한계점.
    512 >> 180mb
    1g >> 360mb
    2g >> 1.2gb


### Transform 관련
Instantiate할때 뒤에 부모를 넣도록. 안하면 root에 생겨서 오버헤드가 생김.
부모트랜스폼에 transform.hierarchycapacity부분을 설정해서 미리 설정해두는게 오버헤드가 적다.
> 유니티에서는 부모에 자식데이터를 갖게 하고 있음. (array로)
Optimize game objects를 체크하면 본을 제거하여 멀티 스레드로 스키닝을 처리할 수 있도록 해주고 있다. (rig메뉴에 있음)
> (근데 2017부터는 무조건 멀티로 돔)

### Assetbundle Load 관련
Async가 sync api보다 좀더 빠르다. 싱크로 해도 어싱크로 돈다 내부적으로
i/o 성능 특징
> Ios가 시퀀셜한 로드가 빠르다. (이전에 유니티에서 개선되서)
> 압축된경우가 비압축보다 빨라지는 경향

### 파티클시스템
일반적으로 많은 자식 노드가 생기는 구조.
* 자식 transform을 찾아서 동작하는 시스템. 즉 getcomponent호출함.
* Depth가 깊으면 깊을수록 성능저하.	
* 우회하려면 withchildren param을 false로 할수도 있지만.
* 하위 파티클시스템자체를 로딩타임에 직접 가지고 있다가 각각 api호출해서 사용하는것이 더 낫다.
	

### 쉐이더
로드되면서 딸꾹질 생길때.

쉐이더 preloading 항목에 save to asset으로 어셋으로 묶어주는데,, 쉐이더의 종류를 관리할 수 있음.

Shadervariantcollection.warmup 해서 다룰수 있다. 로딩타임에 해주면 게임중에 로딩시 딸꾹질이 덜해짐.

### Ugui optimize

#### 주요저하요인

1. Batch 작업을 위한 mesh생성
2. Batching 실패로 인한 drawcall 증가
3. Pixcel overdraw로 인한 gpu 증가.

대응방향
1. 일반적으로 캔버스 단위로 동적/정적 ui 요소를 분리해야.
2. 겹치는 영역을 제거애햐 
  * 비출력되는놈은 그냥 active를 끄자.
  * 정적 layer는 병합.
3. Layout group을 쓰는데,,
  * 이거를 anchor나 userscript로 대체..
  * 이걸설정하면 하위 녀석을 런타임에서 계속 찾음.

#### 점멸표현의 경우.
gameobject를 활성/비활성하면 배칭이 발생.

Canvas 분리및 canvase component를 활성비활성 시키는 방향으로 개선
	
#### Animator 사용시
동작이 없어도 매프레임 캔버스 갱신을 함.
> 가능하면 tween이나 스크립으로 제어해야….
	
#### 다이나믹폰트
텍스트요소가 늘어나면 런타임에 아틀라스가 커지거나 재구축. 동적으로 추가 text를 미리 계산하게됨.
> 가능하다면 미리 알수 있으면 font.texturerebuilt delegate에 생성될 폰트를 미리 구성가능.

Textmesh pro의 경우 다이나믹폰트 지원 예정.
	
#### 스크롤 뷰
캔버스를 분리해야. (pixel perfect off 해야.) 스크롤시는 의미가 없어서 끄는게..

화면바깥요소는 안보이는데,
> Rectmask2d componet를 사용하면 배칭에서 빠지게되어 향상가능.

Scroll rect 수정해서 velocity가 작을때 멈추도록 수정.

Element pool 사용시 계층구조상 순서변경은 canvas 배칭/리배칭 발생시킴
> 가능하면 positon조정으로 해결해야.

Graphic raycaster는 불필요하면 비활성.

World space camera의 event camera를 설정해주자. (none이면 main camera를 찾는 오버헤드)

Ui용 custom shader를 사용하는게.. Masking/clipping은 제거..

[Link](https://unity3d.com/learn/tutorials/topics/best-practices/optimizing-ui-controls)에 예제가 있음.
	
#### UIElements
나중엔 UIElements 에서 retained mode를 사용가능해진다. (변경된것만 업데이트)
> 현재는 에디터에서만 가능.

### 자잘한것.
게임오브젝트 null체크의 경우 네이티브 객체라 오버헤드가 생김. (싱클턴일때 체크하면)
> 개선 하려면 null 제거 (excute order로), bool 변수로 체크하기 

Async upload buffer. 
> 디폴트로 4mb로 잡혀있는데 (퀄리티 세팅에)
> 오버헤드가 생기기에, 최대 texture로 설정하도록 권장.
  
String key api는 문자열 비교하는 로직임..
> Animator.get/set xxx.. Int로 하는 쪽으로..

Array를 반환하는 api는 주의해야.
> Mesh.vertices. Input.touches, getcomponent 등은 내부적으로 alloc해서 사용할때 주희하도록 

?. 연산자는 batch instuction가 추가됨. (약간의 차이다.)

Multiple active camera 관리가 필요. 많이 안되도록.

Blend tree(메카님)은 가능하면 안써야.

서드파티 플러그인을 쓸때 샘플 스크립트들은 없애야. 로딩타임에 영향 (다올라감)
