### View & ViewGroup
- 개요
  + 안드로이드 UI를 구성하는 가장 기본적인 빌딩 블록은 View와 ViewGroup입니다. 이 둘의 관계와 역할을 이해하는 것은 Android UI 개발의 핵심입니다.

- View
  + 개요
    * View클래스는 사용자 인터페이스의 기본 구성요소입니다.
    * 화면에서 사용자와 상호작용할 수 있는 모든 것의 기본 클래스입니다.
    * View는 화면의 특정 직사각형 영역을 차지하며, 해당 영역의 그리기와 이벤트 처리를 담당합니다.
  + 주요 특징 및 역할
    * 화면의 직사각형 영역 차지: 모든 View는 화면에 자신만의 공간을 가집니다. 이 공간의 크기와 위치는 레이아웃 과정에서 결정됩니다.
    * 그리기 (Drawing): View는 자신의 모양과 내용을 화면에 그립니다. onDraw(Canvas canvas) 메서드를 오버라이드하여 커스텀 그리기를 구현할 수 있습니다. 시스템은 필요할 때 이 메서드를 호출하여 View를 다시 그리도록 요청합니다.
    * 이벤트 처리 (Event Handling): 사용자의 터치, 키 입력 등과 같은 이벤트를 감지하고 반응합니다. onTouchEvent(MotionEvent event), onClickListener, onKeyListener 등을 통해 이벤트를 처리합니다.
    * 속성 (Properties): View는 다양한 속성을 가집니다. (예: ID, 배경색, 패딩, 가시성 등). 이러한 속성은 XML 레이아웃 파일이나 Kotlin/Java 코드에서 설정할 수 있습니다.
    * 측정 (Measuring): onMeasure(int widthMeasureSpec, int heightMeasureSpec) 메서드를 통해 자신의 크기를 결정합니다. 부모 ViewGroup으로부터 크기 제약 조건을 전달받아 자신의 콘텐츠와 제약 조건에 맞는 크기를 계산합니다.
    * 레이아웃 (Layout): onLayout(boolean changed, int left, int top, int right, int bottom) 메서드를 통해 화면 내 자신의 위치와 크기를 최종적으로 할당받습니다. 이 과정은 부모 ViewGroup에 의해 관리됩니다.

- ViewGroup
  + 개요
    * ViewGroup은 View 클래스와 특별한 서브클래스입니다.
    * ViewGroup의 주요 역할은 다른 View 객체들을 담는 컨테이너 역할을 하는 것입니다.
    * 즉, ViewGroup은 UI 요소들을 그룹화하고 이들의 레이아웃을 관리합니다.
  + 주요 특징 및 역할
    * 자식 View 포함 (Container): 여러 개의 View 또는 다른 ViewGroup을 자식으로 가질 수 있습니다. 이를 통해 복잡한 UI 계층 구조를 만들 수 있습니다.
    * 레이아웃 관리 (Layout Management): ViewGroup은 자식 View들의 크기를 측정하고 화면에 배치하는 방법을 결정합니다. 각 ViewGroup의 서브클래스는 자신만의 레이아웃 정책을 가집니다.
    * 이벤트 전달 (Event Dispatching): ViewGroup은 터치 이벤트와 같은 사용자 이벤트를 받으면, 해당 이벤트가 발생한 위치에 있는 적절한 자식 View에게 이벤트를 전달할지, 아니면 자신이 직접 처리할지를 결정합니다. (onInterceptTouchEvent(), dispatchTouchEvent())
    * 계층 구조 (Hierarchy): ViewGroup은 다른 ViewGroup을 자식으로 가질 수 있으므로, UI 요소들은 트리(tree)와 같은 계층 구조를 형성합니다. 이 구조의 최상위에는 일반적으로 DecorView라는 특별한 ViewGroup이 있으며, 그 아래에 개발자가 정의한 레이아웃이 배치됩니다.

- View와 ViewGroup의 관계 요약
  + View는 개별 UI 요소입니다.
  + ViewGroup은 이러한 View들을 담는 컨테이너이며, 이들의 배치 방식을 결정합니다.
  + ViewGroup은 View를 상속받으므로, ViewGroup 자체도 하나의 View로 취급될 수 있습니다. 
  + UI는 View와 ViewGroup의 계층적인 트리 구조로 구성됩니다. ViewGroup은 다른 View나 ViewGroup을 자식으로 가질 수 있습니다.

- 면접 질문
  + View와 ViewGroup의 차이점은 무엇인가요?
    * View는 화면에 그려지고 사용자와 상호작용하는 기본적인 UI 구성요소 입니다. 예를들어 TextView, Button 등이 있습니다. 
      반면 ViewGroup은 다른 View들을 담는 컨테이너 역할을 하는 특별한 종류의 View 입니다. 예를들어 LinearLayout, RelativeLayout 등이 있습니다.
  + onMeasure() 메서드에서 MeasureSpec은 무엇이며 어떻게 사용되나요?
    * MeasureSpec은 부모 ViewGroup이 자식 View에게 전달하는 크기 제약 조건입니다. 정수 값으로 인코딩되며, 모드(Mode)와 크기(Size) 두 가지 정보를 담고 있습니다.


---
