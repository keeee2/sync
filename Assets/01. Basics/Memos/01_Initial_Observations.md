---
tags:
  - Basic
  - Unity
date: 2025-02-22
---

---
## 머리말
셰이더란.. 다음과 같이 표현할 수 있다.
- 폴리곤 오브젝트의 속성
- 렌더 파이프라인의 구조
- 행렬, 좌표 시스템

## 1.0.1. 폴리곤 오브젝트의 속성
*Properties of a polygonal object*

### 폴리곤에 대해서..
폴리곤이라는 말은 그리스어 *polugonos*에서 왔다. 여기서 Poly는 Many를 의미하고, Gnow는 Angle을 의미한다.
- Triangle (삼각형)
- Quadrilateral (사각형)
- Pentagon (오각형)
- Hexagon (육각형)

### 프리미티브에 대해..
프리미티브는 3차원 폴리곤으로 구성된 지오매트릭 오브젝트이다. 이는 여러 소프트웨어 프로그램에 이미 정의되어 있다. (Unity, Maya, Blender, ...)
- Spheres
- Boxes
- Quads
- Cylinders
- Capsules

이 오브젝트들은 전부 다르게 생겼지만.. 같은 속성을 공유한다.
- Vertices
- Tangents
- Normals
- UV coordinates
- Color

이들은 모두 **Mesh**라 불리는 데이터 타입에 저장된다!!

이 모든 프로퍼티는 셰이더 내에서 독립적이게 사용할 수 있다 (벡터는 유지한 채)..
이를 잘 활용한다면 멋진 이펙트를 제작할 수 있게 된다
- Vertices (점)
- Edges (선)
- Polygons (면)

---
## 1.0.2. Vertices.
정점은 2D나 3D 공간에서, 오브젝트의 표면을 정의할 수 있는 점과 대응되는 개념이다.
이해하기 쉽게, 다음 두 내용을 살펴보자.
1. 이들은 `Transform` 컴포넌트의 자식들이다.
2. 오브젝트의 전체 볼륨의 중심에 따라 정의된 `Position`을 갖는다.

Unity에서는 Transform 노드에 `position`, `rotation`, `scale`이 있고, 이들은 해당 오브젝트의 `pivot`과 연관된다.

Shape 노드는 Transform 노드의 자식이다. 이는 Geometry attribute (속성..)을 포함하고, 이는 오브젝트의 Vertices 포지션이 그들의 "볼륨"에 연관된다는 뜻이다.

이게... 무슨 뜻이냐면... Vertex들은 Move, Rotate, Scale을 전부 할 수 있다는 뜻이다. 또한 동시에 특정 Vertex의 위치 값을 변경할 수도 있다.

HLSL에서의 `POSITION` 시맨틱은 특별히 Vertices (Volume과 연관된)의 Position에 접근할 수 있도록 해준다.

정리하자면, Transform에는 Position, Rotation, Scale이 있고... Vertex position에는 position이 있다.

---
## 1.0.3. Normals.
노멀은 폴리곤의 면을 기준으로, 돌출된 방향을 가리키는 수직 벡터이다. 우리는 이를 통해 어떠한 면이 앞을 바라보고 있는지 알 수 있다.

후디니에서도... 노멀 벡터를 명시적으로 보여줬었다. 앞면을 제대로 맞출 수 있었다.

---
## 1.0.4. Tangents.
탄젠트는 U 좌표를 따라가는, 정규화된 벡터를 의미한다 (UV 좌표계 내에서). 이건 Main 함수가 Tangent-Space라는 공간에서 생성된다는 뜻이다.

기본적으로, 셰이더에서는 Binormal에 접근할 수 없다. 대신, Normals과 Tangents를 통해 계산하여 구할 수는 있다.

- Normal (Primitive 기준으로 위를 가리키는 벡터)
- Tangent (Primitive 공간에서, U를 따라가는 벡터)
- Binormal (Primitive 공간에서, V를 따라가는 벡터)

더 자세한 건 뒤에서 더 다룰 것이다.

---
## 1.0.5. UV coordinates.
UV 좌표는 3d 물체의 표면에 2d 텍스쳐를 "positioning" 할 수 있는 좌표를 의미한다. 이 좌표는 Reference points로 동작하며, 메쉬의 각 버텍스를 조종할 수도 있다. (블렌더랑.. 후디니에서 다 해본 거지..?)

UV 좌표계 위에서 Vertices를 포지셔닝하는 작업은 "**UV Mapping**"이라 부른다. 오브젝트의 메쉬를 2d 공간에 펼쳐놓고 작업하다. 

3d 메쉬를 펼쳐서 보면.. 가로가 U고, 세로가 V가 된다. 여기서 UV 좌표는 `0.0 ~ 1.0`사이의 범위를 갖는다. 

---
## 1.0.6. Vertex Color.
우리가 3d 오브젝트를 Export하면, 색상과 라이팅 등을 작업해야 한다.
여기서 기본 색상은 "White"이다. 그리고, RGBA 채널로 보면, 1.0f이다.

---
## 1.0.7. 렌더 파이프라인 아키텍쳐
유니티의 렌더 파이프라인은 다음과 같다.
- Universal RP (Lightweight 버전..)
- High-Definition RP

렌더링 파이프라인은 우리의 폴리곤 오브젝트가 컴퓨터 화면에 보이기까지의 과정을 의미한다.
또한, 각각의 렌더링 파이프라인은 각자의 특성을 갖는다. 이는 각 오브젝트가 어떻게 보여지고, 어떻게 최적화되는지에 영향을 준다. 
- Material properties
- Light sources
- Textures
- 모든 셰이더에 포함된 함수

이제 이 프로세스가 일어나는 과정에 대해 알아보자. 그러기 위해서는, 기본 아키텍쳐를 배워야 한다.
유니티에서는 이 아키텍쳐를 4가지로 나눈다.
- Application stage.
- Geometry processing phase.
- Rasterication stage.
- Pixel processing stage.

### 렌더링 파이프라인 로직
리얼타임 렌더링 엔진에 사용되는 렌더링 파이프라인의 기본 모델을.. 꼭 알아두자.
1. Input Assembler
2. Vertex Shader
3. Tessellation
4. Geometry Shader
5. Rasterizer
6. Pixel Shader
7. Color Blending

---
## 1.0.8. Application stage.
애플리케이션 단계는 CPU에서 시작한다. 또한 씬 내에서 발생하는 많은 일들을 담당한다.
- Collision detection
- Texture animation
- Keyboard input
- Mouse input, ...

이 함수들은 프리미티브들(Triangles, Lines, Vertices)을 생성하기 위해, 저장된 메모리를 읽는다.
애플리케이션 단계가 끝날 때, 이 모든 정보는 "**지오매트리 처리 단계**"로 넘어가서 Vertices의 좌표 행렬들을 생성한다.

### 정리
Quad가 있다. 이는 두 개의 Triangle로 나눌 수 있다.
이렇게..

```c
## 삼각형 1
vertex p0 = (0, 0)
vertex p1 = (0, 1)
vertex p2 = (1, 1)

## 삼각형 2
vertex p3 = (0, 0)
vertex p4 = (1, 1)
vertex p5 = (1, 0)
```

---
## 1.0.9. 지오매트리 처리 단계
*Geometry processing phase*

우리가 컴퓨터 화면을 통해 보는 모든 이미지는 CPU를 위해 GPU에 요청된 것이다. 이는 두 가지 과정을 거쳐 발생한다.
1. 지오매트리 처리 단계부터, 픽셀 처리 단계를 거칠 때까지의 렌더 스테이트를 설정한다.
2. 그리고, 오브젝트는 화면에 그려진다.

지오매트리 처리 단계는 GPU에서 발생한다. 또한 오브젝트의 버텍스 처리를 담당한다. 다음 네 가지 하위 프로세스로도 나뉜다.
1. Vertex Shading
2. Projection
3. Clipping.
4. Screen mapping.

애플리케이션 스테이지에서 Primitive가 조립(Assembled)되면, 버텍스 셰이딩 (**Vertex Shader Stage**)에서는 다음 두 작업을 진행한다.
1. 오브젝트의 Vertices position을 계산한다.
2. 그리고, 컴퓨터 화면에 Projection (투사)할 수 있도록, 좌표값 변환을 진행한다.

또한, 이 하위 프로세스 내에서, 다음 단계에 전달할 프로퍼티를 선택할 수 있다 (Normals, Tangents, UV 좌표 등).

투영(Projection)과 클리핑은 이 과정 내에서 발생한다. 이는 카메라의 속성에 따라 달라진다! (Perspective, Orthographic (parallel))
**전체 렌더링 프로세스를 뷰 스페이스라도로도 하는 카메라 프러스럼 (뷰 스페이스)에 있는 요소에 대해서만 발생한다!!!**

### 클리핑
만약 씬에 원이 하나 있고, 반 정도는 카메라 밖에 나갔다고 가정해보자.
그러면 카메라 씬 내에 있는 면들만 살아남을 것이다. (모서리에 있던 녀석은 절두체가 된다.)

잘린 오브젝트가 메모리에 남아있다면, 이를 Scene map에 보낸다. 이 스테이지는 씬의 3차원 공간이 2d 스크린 좌표에 위치할 수 있게 해준다. 이를 Screen, Window 좌표라고도 부른다.

---
## 1.1.0. Rasterization stage.
이쯤 오면 오브젝트는 2d 스크린 좌표를 갖는다. 그러므로 투영 공간 (Projection Area)의 픽셀들을 꼭 찾아야한다. 

화면 내 모든 픽셀을 찾는 과정을 바로 "**래스터화, Rasterization**" 라고 한다. 
이 과정은 씬의 오브젝트와 화면의 픽셀 간의 동기화 지점으로 볼 수 있다.

래스터화는 각 오브젝트에 대해 두 가지 프로세스를 수행한다.
1. Triangle Setup
2. Triangle Traversal

Triangle Setup은 Triangle Traversal로 전송할 데이터를 생성하는 과정이다. 여기에는 Screen의 오브젝트 Edge에 대한 방정식이 포함되어 있다!
그리고, Triangle Traversal에서는 폴리곤 오브젝트의 Area에 포함되는 픽셀을 나열한다. (Listing??)

이러한 방식으로 **Fragments**라 하는 픽셀 그룹을 생성한다. 개별 픽셀을 부를 때도 사용할 **Fragment Shader**라는 용어가 바로 여기서 탄생했다.

---
## 1.1.1. Pixel processing stage.
이전 프로세스를 거쳐오며 만들어진 보간된 값을 통해, 모든 픽셀이 화면에 투사될 준비가 되면, 이 마지막 단계가 시작된다.

이 때 **Pixel Shader Stage**라고도 부르는 **Fragment Shader Stage**가 시작되며, 각 픽셀의 가시성을 담당한다. 이 단계는 픽셀의 최종 색상을 처리한 다음, **Color Buffer**로 전송하는 역할을 한다.

(래스터라이저 단계와 달리... 좀 자글자글해진다. 어쩔 수 없다.)

---
## 1.1.2. Types of Render pipeline
Built-in RP는 소프트웨어에 포함된, 가장 오래된 엔진에 해당한다.
또한 Universal RP와 High Definition RP는 **Scriptable RP**(그래서 SRP)라고도 불린다. 이들은 더 최신이며, 그래픽 성능 향상을 위해 사전 최적화 (Pre-optimized) 되어 있다.

렌더 파이프라인에 관계 없이, 화면에 이미지를 생성하기 위해선 파이프라인을 통과해야 한다. 
또한, 파이프라인에는 "**Render Path**"라 해서, 다양한 처리 경로가 있을 수 있다.

### Render Path
Render Path는 오브젝트의 조명 및 셰이딩과 관련된 일련의 작업에 해당한다.
이를 통해 Illuminated 씬 (Directional light와 Sphere가 있는...)을 그래픽으로 처리할 수 있다.
기본적으로 아래 4가지를 확인할 수 있다.
- Forward Rendering.
- Deferred Shading.
- Legacy deferred.
- Legacy vertex lit.

유니티에서는 기본적으로 "**Forward Rendering**"이다. 이는 모든 렌더 파이프라인을 통틀어, 가장 첫 단계 (Initial path)이다. 이는 그래픽카드 호환성이 높고, 라이팅 계산에 제한이 있다. 즉, 프로세스가 많이 최적화된 버전이다.

### Lighting Model
씬에 Direct light와 오브젝트가 있다고 가정해보자. 이들 간의 상호 작용은 다음 두 가지 기본 개념을 기반으로 한다.
1. Lighting Characteristics (조명 특성)
2. Object material characteristics (물체 머테리얼 특성)

이런 상호작용을 **Lighting Model**이라 한다. 기본 라이팅 모델은 세 가지 속성의 합에 해당한다.
- Ambient color
- Diffuse reflection
- Specular reflection

라이팅 계산은 셰이더 내에서 수행되며, Vertex나 Fragment 별로 수행될 수 있다.
라이팅이 Vertex 별로 계산되는 경우를 "**Per-Vertex Lighting**"이라 하며, Vertex Shader Stage에서 수행된다.

마찬가지로, Fragment 별로 계산된다면, "**Per-Fragment, Per-Pixel Lighting**"이라 하며, 이는 Fragment Shader Stage에서 수행된다.

---
## 1.1.3. Forward Rendering.
Forward는 기본 렌더링 경로이다. 여기서 머테리얼의 모든 일반적인 기능 (노멀맵, 개별 픽셀 조명 (Illumination), 그림자 등)을 지원한다. 이 렌더링 경로에는 셰이더에서 사용할 수 있는 두 가지 경로 (코드 작성)가 있는데, 첫 번째 **base pass**와 두 번째 **additional pass**가 있다.

base pass에서 **ForwardBase Light Mode**를 정의할 수 있다. additional pass에서는 추가 라이팅 계산을 위한 "**ForwardAdd Light Mode**"를 정의할 수 있다. 두 가지 모두 라이팅 계산이 포함된 셰이더의 특별한 함수이다. base pass는 픽셀 단위로 Directional Light를 처리할 수 있으며, 씬에 Directional Light가 여러 개 있는 경우, 가장 밝은 Light에 우선 순위를 둔다! 또한 base pass는 Light Probes, Global Illumination, 그리고 Ambient Illumination (Sky light)를 처리할 수 있다.

이름에서 알 수 있듯이, Additional pass는 오브젝트에 영향을 주는 추가 조명 (Point light, Spotlight, Area Light) 또는 그림자를 처리할 수 있다.

-> 이것이 의미하는 바는 "**씬에 두 개의 조명이 있는 경우, 오브젝트는 그 중 하나의 조명에만 영향을 받는다**"라는 뜻이다. 여기서 이 Configuraion에 additioanl pass를 정의한 경우, 두 가지 조명에 모두 영향을 받게 할 수 있다.

하나 고려해야 할 사항으로는, 각 Illuminated pass가 **독립적인 Draw Call**을 발생시킨다는 점이다. 
정의상, Draw Call은 컴퓨터 화면에 요소가 그려질 때마다 GPU에서 생성되는 Call Graphic이다. 이러한 호출은 많은 양의 계산이 필요한 프로세스이기 때문에, 최소한으로 유지해야 하며, 모바일에서는 특히 조심해야 한다.

이해하기 쉽게... 씬에 네 개의 Sphere와 Directional Light가 있다고 해보자. 각 Sphere는 기본적으로 GPU에 대한 call을 생성한다. 즉, 독립적인 Draw Call을 생성하게 된다.

마찬가지로 Directional light는 씬에 있는 모든 Sphere에 영향을 미치므로 각 Sphere에 대한 추가 Draw Call을 생성한다.

이는 주로 그림자 투영 (Shadow projection)을 계산하기 위해 셰이더에 second pass가 포함되었기 때문에, 4개의 Spehre와 1개의 Directional Light는 총 8개의 Graphic Call을 생성한다....

base pass를 결정한 후, 셰이더에 다른 pass를 추가하면 각 오브젝트에 대해 다른 Draw Call이 추가되어 결과적으로 그래픽 부하가 각각 증가하게 된다....

---
## 1.1.4. Deferred Shading.
이 렌더링 패스는 Lighting Pass가 단 하나 존재한다. 그래서 해당 광원의 영향을 받는 픽셀에 대해서만 계산될 수 있도록 한다. (Geometry와 Lighiting을 분리한다)
이는 여러 오브젝트에 영향을 미치는 상당히 많은 양의 Lighting을 생성할 수 있어, 최종 렌더링의 충실도를 향상시키지만, GPU의 픽셀 당 계산양을 증가시킨다.

Deferred Shading은 여러 광원을 계산할 때 있어 Forward 방식보다 우수하지만, 몇 가지 제한 사항이 있다. 이는 여러 렌더 타겟, 3.0 이상의 Shader Model, 그리고 Depth render texture를 지원하는 그래픽 카드가 있어야 한다.

모바일 디바이스의 경우, 이 구성은 OpenGL ES 3.0 이상을 지원하는 디바이스에서만 작동한다.

또한 Perspective Camera에서만 동작한다. 직교 투영 (Orthographic projection)은 지원되지 않는다.

---
## 1.1.5. 추천 렌더 파이프라인
과거엔 Built-in RP만 있었기에, 프로젝트를 시작하기 쉬웠다. 하지만 지금은 좀 다르다.
1. 일반적으로 PC는 모바일 기기나 콘솔보다 컴퓨팅 성능이 뛰어나다. 즉, 하이엔드 기기를 대상으로 하는 경우에는 High
