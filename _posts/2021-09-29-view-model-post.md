---
title: "[Android] 더 나은 architecture을 위한 필수템, ViewModel"
excerpt: "이제는 알아야 하는 ViewModel!!!"
---

Android 개발자로 추후 진로를 결정할 생각은 없지만, 어찌다 보니 지금 하고 있는 프로젝트가 안드로이드 어플리케이션이다. ~~(그래도 아직은 생각 없다)~~
나름 Google Playstore에도 올려서 서비스를 진행할 어플리케이션이라 안정성을 신경써야 하기 때문에 __ViewModel__ 이란 개념을 도입해서 개발하기로 했다.

새로운 개념이니만큼 이해하기 __쉽게__ 그리고 개발을 위해 최소한으로 필요한 만큼만 설명을 하는 방향으로 잡았다.:smile:

## MVVM이란?

ViewModel을 알기 전 MVVM이란 개념을 알아야하는데, ViewModel이 여기서 파생되었기 떄문이다.

![MVVMPattern](/assets/images/MVVM_Pattern.png)

__MVVM(Model-View-ViewModel)__ 패턴은 하나의  코드에서 사용자에게 보여지는 GUI 부분, 즉 __View__ 를 비즈니스 로직(모델)로부터 분리하여 독립적인 역할로써 존재하게 하는 Software Architecture 패턴이다. 기술이 발전할수록 개발되는 프로그램들은 점점 거대하져 가기에 기능에 있어 역할들을 세분하여 독립적이게 하는 것도 점점 중요한 주제가 되어가고 있다. MVVM은 View에 있어 그런 니즈를 충족시키위해 태어났다고 보면 된다.

1. View에서는 사용자가 스크린에서 보이는 __UI__ 적인 요소만 독립적으로 정의한다. 오직 어떻게 보일지에 대한 것만 구현한다는 뜻이다. 또한 사용자와의 상호작용을 수신하여 이에 대한 처리를 `Data binding`을 통해 ViewModel에게 전달한다.
2. Model에서는 해당 프로그램에서 사용할 __Data__ 와 그 Data에 대한 처리들을 정의한다. 네트워크 통신과 관련되어 있는 부분이라 생각하면 편하다.
3. ViewModel에서는 위의 그림과 마찬가지로 이 View와 Model의 __다리__ 가 되어준다고 생각하면 쉽다. Model과 커뮤니케이션하여 데이터들을 가져와 이를 알맞게 가공하여 View가 오직 이 데이터들을 __표현__ 하는데에만 집중할 수 있도록 조력자 역할을 한다.

이렇게 분리해 놓으면 작업이 병렬적으로 진행될 수 있게 된다. 그로 인해 테스트와 유지 및 보수 등이 한결 효율적이게 진행되므로 전체적인 안정성에 크게 기여하게 되는 것이다.

## Android의 ViewModel

Android에서 View는 'UI 컨트롤러'라고 부른다. 우리가 아는 Activity, Fragment가 포함된다. 앞선 ViewModel의 장점과 더불어 Android내에서 ViewModel은 추가적으로 한 가지의 장점을 더 가지게 되는데 바로 __Life cycle__ 이다.  

UI 컨트롤러(View)에서 관련 데이터를 저장하고 있게 되면 당연하게도 해당 UI 컨트롤러가 제거될 때 관련 데이터 또한 제거되게 된다. 이렇게 되면 UI 컨트롤러가 재생성 되게 될 때 관련 데이터 또한 다시 불러와야 하는 비효율적 문제가 생기게 된다. 대표적인 예시가 회전인데 Android에서 화면을 회전하게 될 때 UI 컨트롤러가 삭제 후 재생성이 되는데 사용하는 관점에서는 화면만 달라지는 것이 전부여야 하지만 표시되는 데이터들까지 달라지는 경우가 생기게 되는 것이다. (기존에는 `Bundle`을 사용해서 해결했지만, 이제는 성능의 한계에 부딪히게 되버렸다.:sweat_smile:)

![viewmodel-lifecycle](/assets/images/viewmodel-lifecycle.png)

이와 같이 UI 컨트롤러와 상관없이 ViewModel이 UI 컨트롤러의 LifeCycle을 모두 포함하는 독립적인 Life Cycle을 갖게 되면서 UI 컨트롤러의 변동에도 데이터는 유지할 수 있게 되는 가장 큰 장점이 있다. 스마트폰의 성능이 향상되고 사용자의 활용이 복잡해지면서(어플리케이션 실행 중에 이것 저것 다른 활동을 하는 경우) ViewModel은 선택이 아닌 필수가 되었다고 할 수 있다. 

Android에서 구현되는 MVVM 패턴은 다음과 같다.

![final-architecture](/assets/images/final-architecture.png)

MVVM 개념에서 Activity/ Fragment가 View임은 위에서 설명했고, 추가적으로 Repository가 Model이 되는 것이다. Repository는 또 데이터 베이스와 웹 서비스 관련 Model로 각각 나누어질수도 있음을 보여준다. 위에서 생소한 것이 LiveData일 것이다. 

__LiveData__ 는 ViewModel에서 데이터를 담고 있게 해주는 Holder 클래스이다. LiveData는 UI 컨트롤러가 Data의 변경을 __관찰(모니터링)__ 할 수 있는 클래스이다. LiveData가 다른 클래스와 다른 가장 큰 특징은 LifeCycle을 인식한다는 점이다. UI 컨트롤러의 LifeCycle에 따라 Active한 상태에서만 Model과 커뮤니케이션하여 Data를 업데이트함으로써 동기화, 메모리, 비정상 종료, 자동화 등의 많은 측면에서 엄청난 장점을 가지게 된다.

## Android에서 ViewModel 구현 (준비 중)
