공식문서 링크<br/> 
[https://docs.flutter.dev/resources/architectural-overview](https://docs.flutter.dev/resources/architectural-overview)

<br/>

## 플러터 아키텍쳐

<br/>

### Architectural Layers

<img width="400" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/77ce1d39-45c7-4d93-9303-34ce202c4aaa">

<br/>
<br/>

#### Embbeder
🔹 Embedder가 수행하는 역할은 크게 앱의 진입점, 플랫폼별 운영체제와 상호작용, 플러터 애플리케이션 이벤트 루프 처리 3가지가 있다.<br/>
🔹 진입점 역할: 각 플랫폼별 언어로 작성되어 특정 플랫폼에서 실행될 수 있도록 한다.

<br/>

#### Engine
🔹 엔진은 프레임을 **rasterize**하는 역할을 수행한다. 여기서 rasterize란 벡터 형식의 그래픽 데이터를 픽셀 단위로 변환하여 디스플레이 장치에 표시할 수 있도록 만드는 과정이다.<br/>
🔹 엔진은 `dart:ui`(C++코드를 Dart로 포장) 를 통해 플러터 프레임워크에 노출되어 있다. 따라서 엔진을 이용한 세부작업이 필요하면 `dart:ui`를 이용하자

<br/>

#### Framework
🔹 Foundation: Flutter 프레임워크의 다른 모든 계층에서 사용되는 가장 기본적인 유틸리티 클래스와 함수들로 구성<br/>
🔹 Animation/Painting/Gestures: 사용자 경험 향상을 위한 애니메이션, 페인팅, 제스처를 다루기 위한 위젯 제공 <br/>
🔹 Rendering: Render Tree가 만들어지는 영역으로 실제 렌더링이 가능한 객체가 만들어지며 이 객체들을 "동적"으로 다룰 수 있다.<br/>
🔹 Widgets: Widget Tree가 만들어지는 곳으로 각 위젯은 Rendering layer의 위젯들과 대응된다. 또한 반응형 모델이 도입된 영역이기도 하다<br/>
🔹 Material/Cupertino: 각각의 디자인 언어(Material Design과 iOS 디자인)를 구현하는 데 필요한 위젯들을 제공<br/>

<br/>

#

### Anatomy of an App

<img width="357" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/bdd7caca-3799-47f5-9066-1a35fe303078"><br/>

<br/>

flutter 애플리케이션이 처음만들어질때 `flutter create`가 실행되고 위와같은 스택구조가 쌓인다. <br/>

🔹 **Dart 애플리케이션** <br/>
앱 개발자가 비즈니스 로직과 UI를 구현<br/>
위젯을 조합하여 앱의 화면을 구성<br/>

🔹**프레임워크** <br/>
개발자가 작성한 위젯 트리를 받아 고수준의 API를 통해 다양한 기능 제공 <br/>
위젯 트리를 씬으로 합성하여 엔진에 전달 <br/>

🔹**엔진** <br/>
합성된 씬을 래스터화(벡터->픽셀)하여 실제 화면에 그린다<br/>
`dart:ui` API를 통해 프레임워크와 상호작용합니다.<br/>
저수준의 그래픽, 텍스트 레이아웃, Dart 런타임 기능을 제공<br/>

🔹**임베더** <br/>
운영 체제와 상호작용하여 렌더링 표면, 접근성, 입력 등의 서비스를 제공 <br/>
(사용자/IO)이벤트 루프를 관리하여 애플리케이션이 원활하게 실행되도록 관리 <br/>
플랫폼별 API를 통해 엔진과 플랫폼을 통합 <br/>

🔹**러너** <br/>
임베더의 API를 사용하여 앱 패키지를 생성하고, 이를 대상 플랫폼에서 실행할 수 있도록 만듬 <br/>
앱 개발자가 flutter create 명령어를 통해 생성한 템플릿의 일부라 할 수 있음 <br/>

<br/>

#

### Reactive user interfaces

<img width="400" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/afd4897d-edeb-4462-bce9-cc624d8af17e">

<br/>
<br/>


🔸 전통적인 UI인터페이스는 한 번만 그려지고 참조형식으로 이벤트에 의해 바뀌는 부분만 업데이트 되었었다. <br/>
🔸 하지만 이렇게 각각의 UI 조각들이 상태를 참조하면 나중에 상태가 많아질경우 관리하기가 매우 복잡해진다. 위 UI에서만 봐도 칼러박스하고 슬라이더가 변경됨에 따라 바뀌어야할 녀석들이 한두친구가 아니다 <br/>
🔸 이를 보완하기 위해 MVC패턴이 생겼다. 컨트롤러를 통해 이벤트를 받아 모델의 데이터가 바뀌면 모델은 뷰에게 새로운 상태를 제공하는 형식이다. 하지만 UI요소를 만들고 업데이트한다는 스텝이 분리되어 있어 참조해야하는 데이터 동기화(sync)하는데 어려움을 겪을 수 있다. <br/>
🔸 그래서 플러터는 React-style 방식을 따랐다. 개발자는 UI의 형태만 선언하고 프레임워크에서 UI생성과 업데이트를 책임진다.<br/>
🔸 플러터의 위젯은 불변객체로, 트리 구조로 선언된다. 여기서 트리는 레이아웃과 각 위젯들을 컴포지셔닝하는 역할을 맡는 트리가 나눠진다 <br/>

<br/>

<img src="https://github.com/suojae3/flutter_docs_study/assets/126137760/a4041c07-1461-403e-a2dc-a9780ec766f5" width="400">

<br/>

🔸 플러터의 `build()` 메서드는 이러한 reactive 형식을 따라 state 파라미터를 가지고 불변UI를 그려낸다. 이를통해 state의 사이드이펙트를 방지할 수 있다.<br/>
🔸 이러한 방식에서 고려할 점이 상태 변화에 따라 런타임에서 불변UI를 계속해서 그려내야해서 속도가 중요한데 다행히 다트는 이점에서 최적화되어있다고 한다 ^^ <br/>

<br/>

#

### Widgets

Flutter는 UI 요소들을 각 플랫폼(iOS/AOS..)에서 제공하는 컨트롤을 사용하는 대신, 자체적으로 순수 Dart로 구현된 통합된 UI요소를 제공해준다. 이를통해 얻을 수 있는 이점은 아래 3가지가 있다. <br/>

🔸 Flutter는 시스템이 제공하는 확장 포인트에 제한되지 않고, 개발자가 커스터마이징 할 수 있도록 만들어놨다. <br/>
🔸 전체 UI요소를 한 번에 합성(composite)할 수 있다. 이를통해 플랫폼 코드 간의 전환을 반복하지 않아서 성능 병목을 피할 수 있다<br/>
🔸 어떤 운영 체제 버전에서도 동일한 UI와 동작을 유지하여 플랫폼에 종속되지 않고 일관성을 유지할 수 있다.  <br/>
🔸 위젯은 기본적으로 하나의 작고, 하나의 목적만을 가지게된다. 이러한 단일 목적을 가진 위젯들이 합성되어 전체 앱을 만들어낸다. <br/>
🔸 컴포지셔닝 특성상 다양한 조합을 만들어내기 위해 플러터는 클래스 계층을 최대한 얇고 넓게 만들려 노력했다. 이러한 노력의 일환으로 padding 같은 여백도 따로 클래스로 선언되어 있다. <br/>

<br/>

#

### Widget State

🔸 Stateful Widget에서 State를 따로 분리함으로써 Stateful Widget또한 Stateless Widget과 같은 방식으로 동작할 수 있도록 만들었다.<br/>
🔸 즉 위젯트리 자체는 Statless 위젯 동작으로 퉁치고 State는 Element Tree가 들고 관리하는 구조가 된다. -> State를 프레임워크에서 관리해줌<br/>
🔸 이해를 못한 관계로 추후 플러터 이해도 높아졌을시 작성

<br/>

#

### Rendering and layout

<img width="450" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/c8be6825-7acd-432d-bb0e-eeb24bb8e364">

<br/>
<br/>

🔹 **Build** <br/>
build 단계에서는 Widget Tree/Element Tree/Render Tree가 만들어진다.<br/>

<img width="300" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/99483b02-cfae-4263-99ee-5be52da5c046">

<br/>

이 단계에서 Element는 위젯의 타입과 키를 확인하고 상태와 생명주기를 관리한다. 그리고 각 element들은 자신이 사용할 Render object를 만든다.

<br/>

🔹 **Layout** <br/>

<img width="400" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/d55446ed-67e6-47f6-8d61-e2e0daf8bc8a">

<br/>

Layout phase에서는 RenderTree가 상위위젯부터 하위위젯까지 constraints를 전달하고 다시 하위위젯부터 상위위젯까지 size를 전달한다. 당연한 이야기이지만 상위부터 최소/최대 제약조건이 마지막 하위 위젯 제약까지 계산이 되고나서야 사이즈가 결정되고 결정된 사이즈는 하위부터 상위까지 전달한다.

<br/>

🔹 **Paint** <br/>

<img width="400" alt="image" src="https://github.com/suojae3/flutter_docs_study/assets/126137760/d41489b6-c557-4696-a995-f35db9451b42">

- Layout이 끝나고 Renderobject의 최종결과물, 즉 플러터 프레임워크단의 최종 결과물인 **Layer Object**가 만들어지면 이를 GPU가 그릴 수 있게 엔진으로 보낸다 <br/>
  

<br/>

🔹 **Compositioning Phase** <br/>
- 프레임워크로부터 Layer Object를 받은 프레임엔진은 GPU를 통해 각레이어를 결합하여 최종이미지를 생성한다. <br/>

<br/>

🔹 **Rasterize Phase** <br/>
- 생성한 최종이미지를 GPU는 픽셀로 바꿔 화면에 UI를 띄운다. 만약 바뀐게 없다면 픽셀을 재사용한다.

<br/>


<img src="https://github.com/suojae3/flutter_docs_study/assets/126137760/0d1108b4-d99b-44e7-ae03-9ea0babcd6fe" width="500">











