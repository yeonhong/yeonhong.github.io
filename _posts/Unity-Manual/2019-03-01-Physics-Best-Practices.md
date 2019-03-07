---
layout: single
title: "Physics Best Practices"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Physics
  - Optimaze
link: https://unity3d.com/learn/tutorials/topics/physics/physics-best-practices?playlist=30089
---
이 레슨에서는 게임에서 Physics을 사용할때의 모범사례와 왜 사용해야 하는지를 보여주는 몇 가지 단서를 살펴볼 것입니다.

## Layers and Collision Matrix

구성되지 않은 경우 모든 게임 개체는 기본적으로 모든 항목과 모든 항목이 충돌하는 기본 항목에 만들어집니다. 이것은 매우 비효율적입니다. **무엇과 충돌해야하는지 설정하십시오.** 이를 위해서는 **각 유형의 객체에 대해 서로 다른 레이어를 정의해야합니다.** 각각의 새 레이어에 대해 새 행과 열이 충돌 행렬에 추가됩니다. 이 행렬은 레이어 간의 상호 작용을 정의합니다. 기본적으로 새 레이어를 추가 할 때 Collision Matrix는 새로운 레이어가 기존의 모든 레이어와 충돌하도록 설정되므로 개발자가 액세스하고 상호 작용을 설정하는 것은 개발자의 책임입니다. 레이어를 올바르게 설정하고 Collision Matrix를 설정하면 불필요한 충돌을 피하고 충돌 청취자를 테스트 할 수 있습니다. 데모 용으로 상자 컨테이너 안에 2000 개의 객체 (1000 빨간색과 1000 녹색)를 인스턴스화하는 간단한 데모를 만들었습니다. 녹색 물체는 자신과 컨테이너 벽과 만 상호 작용해야합니다. 빨간색 물체도 마찬가지입니다. 테스트 중 하나에서 모든 인스턴스는 기본 레이어에 속하며 상호 작용은 충돌 수신기의 게임 개체 태그를 비교하는 문자열로 수행됩니다. 다른 테스트에서는 각 객체 유형이 자체 레이어에 설정되고 충돌 매트릭스를 통해 각 레이어의 상호 작용을 구성합니다. 이 경우 오른쪽 충돌 만 발생하기 때문에 문자열 테스트가 필요하지 않습니다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices01.png?itok=_dqpXRM7)

그림1 : 충돌 매트릭스 구성

위 이미지는 데모 자체에서 가져온 것입니다. 그것은 충돌의 양을 계산하고 5초 후에 자동으로 일시 중지하는 간단한 관리자가 있습니다. 공통 레이어를 사용할 때 발생하는 불필요한 여분의 충돌의 양은 상당히 인상적입니다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices02.png?itok=VdK0kAF8)

그림2 : 5 초 동안의 충돌 양

보다 구체적인 데이터를 얻으려면 필자는 물리엔진에서 프로파일러 데이터도 캡처했습니다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices03.png?itok=IHogM3gg)

그림 3 : 일반 레이어 vs 분리된 레이어의 물리 프로파일러 데이터

물리층에 소비 된 CPU의 양은 꽤 다릅니다. 프로파일러 데이터에서 알 수 있듯이 단일 레이어 (평균 27.7ms)에서 레이어 분리 (평균 17.6ms)까지 볼 수 있습니다.

## Raycasts

Raycasting은 물리 엔진에서 사용할 수있는 매우 유용하고 강력한 도구입니다. 그것은 우리가 어떤 길이의 어떤 방향으로 광선을 발사 할 수있게 해주 며, 그것이 뭔가와 부딪히면 우리에게 알려줄 것입니다. 그러나 이것은 비싼 작업입니다. 그 성능은 광선의 길이와 scene의 colider의 유형에 의해 크게 영향을받습니다.

다음은 사용법에 도움이되는 몇 가지 힌트입니다.

* 이것은 분명하지만 작업을 완료하는 광선의 양을 최소화하십시오.
* 광선의 길이를 필요 이상으로 연장하지 마십시오. 광선이 클수록 더 많은 물체를 테스트해야합니다.
* FixedUpdate() 함수 내에서 Raycast를 사용하지 마십시오. Update()도 안됩니다.
* 사용중인 충돌기 유형을 조심하십시오. 메쉬 콜 라이더에 대한 레이 캐스팅은 비용이 많이 듭니다.
  * 좋은 해결책은 기본 colider로 자식을 만들고 메시 모양을 근사화하는 것입니다. 부모 리지드 바디 아래의 모든 자식 콜리더는 복합 콜라이더로 작동합니다.
  * 메쉬 콜리더를 사용해야 할 필요가 있다면, 최소한 볼록하게 만듭니다.
* 광선이 닿는 부분을 구체적으로 밝히고 항상 광선 차단 기능에 레이어 마스크를 지정하십시오.
  * 이것은 공식 문서에서 잘 설명되어 있지만 레이 캐스트 기능에서 지정하는 것은 레이어 ID가 아니라 **비트 마스크**입니다.
  * 그러므로 광선이 10 인 층에있는 물체에 부딪 치길 원한다면, 당신이 지정해야하는 것은 1 << 10 (왼쪽 10 배에 '1'을 비트 시프트)이며 10이 아닙니다.
  * 광선이 레이어 10에있는 것을 제외한 모든 것을 명중하려면 비트 마스크의 각 비트를 반전시키는 비트 보수 연산자 (~)를 사용하면됩니다.

나는 물체가 녹색 상자들과 충돌하는 광선을 쏘는 간단한 데모를 개발했습니다.
![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices04.png?itok=MmAXktdT)

그림 4 : 간단한 Raycast 데모 장면

거기에서 필자가 이전에 작성한 것을 백업 할 프로파일 러 데이터를 얻기 위해 광선의 수와 길이를 조작합니다. 아래의 그래픽에서 광선의 수와 길이가 가질 수있는 성능에 미치는 영향을 확인할 수 있습니다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices05.png?itok=1rsbKL_o)

그림 5 : 광선의 갯수가 성능에 미치는 영향

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices06.png?itok=zDYceKTk)

그림 6 : 광선의 길이가 성능에 미치는 영향

데모 목적으로, 나는 정상적인 기본 충돌기에서 메시 충돌기로 전환 할 수 있도록하기로 결정했다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices07.png?itok=8xWfcU2B)

그림 7 : 메쉬 콜리더 (colliders) 장면 (콜라이더 당 110 verts)

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices08.png?itok=whLx5PFx)

그림 8 : Primitive 대 Mesh colliders 물리 프로파일 러 데이터

프로필 그래프에서 볼 수 있듯이, 메쉬 콜 레이더에 대한 레이 캐스팅은 물리 엔진이 프레임 당 더 많은 작업을 수행하게합니다.

## Physics 2D vs 3D
프로젝트에 가장 적합한 물리 엔진을 선택하십시오. 2D 게임이나 2.5D (2D 평면에서 3D 게임)를 개발하는 경우 3D Physics 엔진을 사용하는 것은 과도한 행동입니다. 이 추가 차원은 프로젝트에 불필요한 CPU 비용을 발생시킵니다. 이 주제에 대해 특별히 쓴 이전 기사에서 두 엔진의 성능 차이를 확인할 수 있습니다.

## Rigidbody
Rigidbody 구성 요소는 객체간에 물리적 상호 작용을 추가 할 때 필수적인 구성 요소입니다. 콜리더를 트리거로 사용하는 경우에도 OnTrigger 이벤트가 제대로 작동하려면 게임 객체에 콜리더를 추가해야합니다. **RigidBody 구성 요소가없는 게임 개체는 정적 충돌 장치로 간주됩니다. 이것은 물리적 인 엔진이 물리적 인 세계를 다시 계산하도록 강제하기 때문에 static collider를 움직이는 것은 극도로 비효율적이기 때문에 주의해야합니다.** 다행히도 프로파일 러는 CPU 프로파일 러의 경고 탭에 경고를 추가하여 정적 충돌기를 움직이는 지 알려줍니다. 정적 충돌자를 움직일 때의 충격을보다 잘 보여주기 위해 필자가 제시 한 첫 번째 데모에서 모든 움직이는 물체의 RigidBody를 제거하고 그 위에 새로운 프로파일 러 데이터를 캡처했습니다.

![no-alignment](https://unity3d.com/sites/default/files/styles/original/public/learn/physicsbestpractices09.png?itok=K_7OzfYr)

그림 9 : Moving static colliders warning

그림에서 알 수 있듯이, 움직이는 게임 객체마다 하나씩 총 2000 건의 경고가 생성됩니다. 또한 Physics에서 소비 한 평균 CPU 양은 ~ 17.6ms에서 ~ 35.85ms로 증가했습니다. **게임 객체를 움직일 때 RigidBody를 추가해야합니다. 움직임을 직접 제어하려면 rigidbody 속성에서 kinematic으로 표시하기만 하면됩니다.**

## Fixed Timestep
Time Manager에서 Fixed Timestep 값을 조정하면 FixedUpdate() 및 Physics 업데이트 속도에 직접 영향을 미칩니다. 이 값을 변경하면 정확도와 Physics에서 소비 한 CPU 시간 사이에 좋은 절충안을 얻을 수 있습니다.

## Wrap-up
토론 된 모든 주제는 실제로 구성 / 구현하기 쉽고 충돌 감지에만 사용되는 경우에도 개발할 거의 모든 게임에서 물리 엔진을 사용하므로 프로젝트 성능에 분명히 차이가 있습니다.

