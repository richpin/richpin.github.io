---
layout: post
title: "[Android] 더 나은 architecture을 위한 필수템, ViewModel"
description: "이제는 알아야 하는 ViewModel!!!"
img: MVVM_Pattern.png
tags: [Android]
---

Android 개발자로 추후 진로를 결정할 생각은 없지만, 어찌다 보니 지금 하고 있는 프로젝트가 안드로이드 어플리케이션이네요. ~~(그래도 아직은 생각 없습니다)~~
나름 `Google Playstore`에도 올려서 서비스를 진행할 어플리케이션이라 안정성을 신경써야 하기 때문에 **ViewModel** 이란 개념을 도입해서 개발하기로 했습니다.

새로운 개념이니만큼 이해하기 **쉽게** 그리고 개발을 위해 최소한으로 필요한 만큼만 설명을 하는 방향으로 잡겠습니다.:smile:

# MVVM이란?

`ViewModel`을 알기 전 `MVVM`이란 개념을 알아야하는데, `ViewModel`이 여기서 파생되었기 때문입니다.

![MVVMPattern](/assets/img/view_model/MVVM_handmade.png)
~~(발그림 죄송합니다ㅎㅎ)~~

**MVVM(Model-View-ViewModel)** 패턴은 하나의 코드에서 사용자에게 보여지는 GUI 부분, 즉 **View** 를 비즈니스 로직(모델)로부터 분리하여 독립적인 역할로써 존재하게 하는 `Software Architecture` 패턴입니다. 쉽게 말해서 `GUI`와 관련된 코드들을 관련없는 다른 코드들과 분리시켜서 **독립적** 으로 존재하게 하고 싶다는 겁니다.
기술이 발전할수록 개발되는 프로그램들은 점점 거대하져 가기에 기능에 있어 역할들을 세분하여 독립적이게 하는 것도 점점 중요한 주제가 되어가고 있죠? `MVVM`은 이 `View`에 있어 그런 니즈를 충족시키위해 태어났다고 보시면 됩니다.

1. `View`에서는 사용자가 스크린에서 보이는 **UI** 적인 요소만 독립적으로 정의합니다. 오직 어떻게 보일지에 대한 것만 구현한다는 뜻이죠. `ViewModel`의 변화에 맞춰 사용자에게 보이는 부분에도 변화를 주는 것이지요.
2. `Model`에서는 해당 프로그램에서 사용할 **Data** 와 그 `Data`에 대한 처리들을 정의합니다. 네트워크 통신과 관련되어 있는 부분이라 생각하면 편할겁니다. 서버와 커뮤니케이션 하여 `data`들을 보내고 가져오고 계산하는 등의 과정을 **독립적** 으로 맞게 되는 것입니다.
3. `ViewModel`에서는 위의 그림과 마찬가지로 이 `View`와 `Model`의 **다리** 가 되어준다고 생각하면 됩니다. `Model`과 커뮤니케이션하여 데이터들을 가져와 이를 알맞게 가공하여 `View`가 오직 이 데이터들을 **표현** 하는데에만 집중할 수 있도록 조력자 역할을 합니다.

이렇게 분리해 놓으면 작업이 병렬적으로 진행될 수 있게 되어 테스트와 유지 및 보수 등이 한결 효율적이게 진행되므로 전체적인 안정성에 크게 기여하게 되는 것입니다.

# Android의 ViewModel

Android에서 `View`는 UI 컨트롤러라고 부릅니다. 저희가 이미 아는 `Activity`, `Fragment`가 이 UI 컨트롤러에 포함되는 것이죠. 앞선 `ViewModel`의 장점과 더불어 Android내에서 `ViewModel`은 추가적으로 한 가지의 장점을 더 가지게 되는데 바로 **Life cycle(생명주기)** 입니다.

UI 컨트롤러(View)에서 관련 데이터를 저장하고 있게 되면 당연하게도 해당 UI 컨트롤러가 제거될 때 관련 데이터 또한 제거되게 되겠죠?? 이렇게 되면 UI 컨트롤러가 재생성 되게 될 때 관련 데이터 또한 다시 불러와야 하는 비효율적 문제가 생기게 됩니다. 대표적인 예시가 회전인데 Android에서 화면을 회전하게 될 때 UI 컨트롤러가 삭제 후 재생성이 되는데 사용하는 관점에서는 화면만 달라지는 것이 전부여야 하지만 UI가 재생성 되면서 데이터들까지 달라지는 참사가 생기게 되는 것이죠. (기존에는 `Bundle`을 사용해서 해결했지만, 이제는 성능의 한계에 부딪히게 되버렸다고 합니다.:sweat_smile:)

![viewmodel-lifecycle](/assets/img/view_model/viewmodel-lifecycle.png)

이와 같이 UI 컨트롤러와 상관없이 `ViewModel`이 UI 컨트롤러의 `LifeCycle`을 모두 포함하는 독립적인 `Life Cycle`을 갖게 되면서 UI 컨트롤러의 변동에도 데이터는 유지할 수 있게 되는 가장 큰 장점이 생기게 됩니다. 스마트폰의 성능이 향상되고 사용자의 활용이 복잡해지면서(어플리케이션 실행 중에 이것 저것 다른 활동을 하는 경우) `ViewModel`은 선택이 아닌 필수가 되었다고 할 수 있겠죠???:woozy_face:

Android에서 구현되는 `MVVM` 패턴은 다음과 같습니다.

![final-architecture](/assets/img/view_model/final-architecture.png)

`MVVM` 개념에서 `Activity/Fragment`가 `View`임은 위에서 설명했고, 추가적으로 `Repository`가 `Model`이 되는 것입니다. `Repository`에서 또 데이터 베이스와 웹 서비스 관련 `Model`들과 소통이 이루어짐을 보여줍니다. 위에서 생소한 것이 `LiveData`일 것일 텐데요.

**LiveData** 는 `ViewModel`에서 데이터를 담고 있게 해주는 `Holder` 클래스입니다. `LiveData`는 UI 컨트롤러가 `Data`의 변경을 **관찰(모니터링)** 할 수 있는 클래스입니다. 따라서 `ViewModel`이 이 `LiveData`라는 것을 통해 UI 컨트롤러에게 '야! 데이터 바뀌었어! 일해!!!"라고 해주는 것이지요. LiveData가 다른 클래스와 다른 가장 큰 특징은 `LifeCycle`을 인식한다는 점입니다. UI 컨트롤러의 `LifeCycle`에 따라 `Active`한 상태에서만 `Model`과 커뮤니케이션하여 `Data`를 업데이트함으로써 동기화, 메모리, 비정상 종료, 자동화 등의 많은 측면에서 엄청난 장점을 가지게 됩니다.

# Android에서 ViewModel(MVVM) 구현

제가 찾기로 한국 블로그에 설명은 많은데 코드를 보여주며 설명해주는 곳이 많지 않았습니다. 그래서 저와 함께 간단히 살펴볼텐데요. 외국 블로그에서 여러 능력자분들이 쓰신 여러가지 방식의 코드들이 있습니다만, **저는 제가 직접 프로젝트에 사용하였던 방식으로 예시를 들어보겠습니다.(따라서 다소 코드에 편향성이 있을 수 있으니 하나의 예시 그 이하로만 생각해주셨으면 합니다.)** 이 때 배웠단 간간한 다른 지식들까지 말씀드릴테니 천천히 따라와 보시죠! 간단하게 사용자 정보를 다루는 `MVVM`을 구현해보도록 하겠습니다.

가장 먼저, `ViewModel` 작성을 해봅시다. 사용자 정보에 대한 `ViewModel`이니 `class`이름을 `UserViewModel`이라 칭하겠습니다.

```kotlin
class UserViewModel : ViewModel() {
    private val users = MutableLiveData<ArrayList<User>>()
}
```

사용자 정보를 담은 User라는 데이터 클래스를 담은 `ArrayList`을 `LiveData`로 감싸고 있는 모습입니다. `ArrayList<User>`대신 어떤 타입이든 사용자가 필요한 자료형을 그때그때 넣어서 감싸주시면 됩니다.

이제 함수를 사용자 정보를 가져오는 함수를 만들어볼까요. `Repository`에 데이터 베이스로부터 사용자 정보를 가져오는 코드를 작성합니다.

```kotlin
object Repository {
    fun loadUsers() = callbackFlow {
        ...
        trySend(user)
    }
}
```

갑자기 **callbackFlow** 가 뭐냐 하실수도 있는 저와같은 분이 계실겁니다. 이 `getUsers`함수처럼 네트워크와 통신을 한다면 통신에 의한 시간이 걸리겠죠? 평소에는 괜찮다 하더라도 만약 이 시간이 내가 작성한 코드를 컴퓨터가 진행하는 속도보다 오래 걸린다면 문제가 생기는 것입니다. 이 통신은 코드 진행과는 별개로 ~~(정확한 표현은 아니지만...)~~ 일어나고 있기 때문에 `user`을 가져오기도 전에 프로그램은 이미 `user`을 가져온 이후의 코드들을 실행시키고 있는 것이지요. 그것을 막아주기 위해 `kotlin`에서는 이와 같은 방식으로 좀 그때그때마다 기다려달라는 의사를 전달하고 있는 것입니다. user라는 사용자 정보를 가져오는 것이 끝났다면 `trySend`로 데이터를 쏴줌으로써 이어서 진행하라는 의사까지 전달하는 것입니다. 이와 같은 방식을 **callback** 이라고 하는데 더 알아보고 싶으신 분들은 정말 많은 좋은 글들과 자료들을 참고하여 공부하시면 좋을 것 같습니다.

자 이제 `UserViewModel`와 `Repository`사이에 다리를 놓아주어야 할 것입니다. `Repository`에서 가져온 정보를 UI 컨트롤러에게 전달하는 함수를 작성해봅시다.

```kotlin
class UserViewModel : ViewModel() {
    private val users = MutableLiveData<ArrayList<User>>()

    fun getUsers() = viewModelScope.launch {
        val list: ArrayList<User> = arrayListOf()

        Repository.loadUsers().collect { user ->
            user?.let { list.add(user) }
        }
        users.postValue(list)
    }
}
```

`getUsers`가 하는 일은 `Repository`의 미리 써놓은 `loadUsers`를 통해서 사용자 정보를 가져와 `LiveData`에 변화를 줌으로써 UI 컨트롤러 인식하게 하는 일 뿐입니다. 정말 다리 역할만 하는 것이죠. `loadUsers`옆의 `collect` 구문은 `trySend`를 쏴준 데이터를 받아들이겠다는 것입니다. 람다형식으로 쏴준 데이터를 `user`라는 이름으로 받아 처리를 해주고 있는 것이지요. 여기서 주의할 점은 **LiveData는 값 자체가 바뀌는 것만을 변화로 취급한다는 것입니다.** 배열을 `LiveData`로 감싸주었지만 배열의 요소가 추가되고 제거되고 등의 변화는 `LiveData`는 감지하지 못한다는 것입니다. 배열의 값, 즉 감싸고 있는 배열의 주솟값 자체가(쉽게 말하면 새로운 배열이 들어와야) 변화를 감지한다는 것이지요. 아직 `kotlin`의 `LiveData`가 그 정도로 똑똑하지는 않으니 부디 저처럼 당연하게 믿고 썼다가 낭패를 보시지 않았으면 좋겠습니다. 그래서 저는 이와 같이 임시로 새로운 배열을 만들어 그곳에 들어오는 사용자 정보들을 다 저장한 뒤 `postValue`로 `LiveData`의 값을 바꿔준 것입니다.

여기서 또 아까 `callbackFlow`마냥 써있는 `viewModelScope.launch`는 무엇일까요. `getUsers` 안에 `callback`함수로 쓴 `loadUsers`가 있죠? 방금 설명들였듯이 `load`할 때까지 기다린다는 뜻입니다. 그런데 말입니다~ 기다리는 건 좋은데 그 때 마냥 멈춰있기에는 너무 비효율적이지 않나요? 그래서 우리는 이 친구를 뒤쪽에 둔 채 계속 나아가야할 코드들은 진행을 계속합니다. 다른 **Thread** 에서 진행되게끔 해서 **Main Thread** 에는 멈춤을 주지 않는 이 방식을 **비동기 처리** 라고 합니다. 굉장히 중요한 개념이니 이 또한 무조건 공부하셔야 하는 내용입니다. 저는 이렇게 쉽게 이해의 포문을 열어드렸고 마찬가지로 굉장히 많은 좋은 글들과 자료들로 관심있게 찾아봅시다. 어쨌든 결론적으로 이 비동기 처리를 `kotlin`에서는 **coroutine** 이라는 이름으로 제공을 하는 것이고 이 `coroutine`을 `ViewModel`의 상태에 따라(`ViewModel`이 중지되면 `coroutine`도 중지되게끔 등등..) 진행을 하게끔 해주는 것이 `viewModelScope`의 `launch`메소드 랍니다.

이제 UI 컨트롤러로를 작성하여 봅시다. UI 컨트롤러 내에서 `ViewModel`과의 소통을 위해 `ViewModel` 변수를 생성하여 줍니다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: UserViewModel

        override fun onCreate(savedInstanceState: Bundle?) {
        ...
        viewModel = ViewModelProvider(this).get(UserViewModel::class.java)
        }
}
```

이제 UI 컨트롤러 내에서 `ViewModel`의 `LiveData`의 변화를 감지하기 위해 `observe`를 해줍시다.

```kotlin

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    viewModel.contents.observe(this, Observer { users ->
        ...
    })
}
```

(추가적으로 `Fragment`라면 이 `LiveData`가 `Fragmen`t의 `Lifecycle`을 따라갈 수 있도록 `this`대신 `viewLifecycleOwner`을 넣어주셔야 합니다. ) 이렇게 `LiveData`안에 담긴 `User` 배열에 변화가 생겼을 때 이 값을 람다 형식을 통해 `users`로 받아 이를 이용해 UI를 다루는 코드를 작성하시면 된답니다.

어땠나요? 천천히 따라오니 이제 Android에서 `MVVM`을 어떻게 적용할 지 흐름을 좀 느끼셨나요? 그랬다면 다행입니다. 저 역시 처음 봤을 때 많이 헷갈렸던 새로운 개념인지라, 저와 같은 분에게 조금이나마 도움이 되고자 글을 써보았습니다. 그래서 인지 너무 쉽게 설명했기에 조금 더 큰 이해를 위해서는 각종 글들과 자료로 충분히 공부도 해보시고 무엇보다 적용해보시면서 `MVVM`을 완전정복 하셨으면 좋겠습니다! :grin:
