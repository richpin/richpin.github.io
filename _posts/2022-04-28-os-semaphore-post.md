---
layout: post
title: "[OS] Semaphore"
description: 또 다른 공유 자원 제한방법인 Semaphore에 대해 알아보자.
img: semaphore_svg.png
tags: [OS]
---

이번에는 **Semaphore**에 대해 알아보도록 합시다.

# Semaphore란 무엇일까?

우리가 흔히 `Dijkstra Algorithm`으로 잘 알고 있는 Dijkstra는 공유 자원에 대한 접근을, 즉 동기화 문제를 위해 `Semaphore`라는 것을 만들었습니다. 우리는 이전에 배웠던 `Locks`와 `Condition Variables` 대신 이 하나의 Semaphore라는 것을 이용할 수 있는 거지요. 단순하게 생각하면 Semaphore는 그저 두 개의 `atomic operation`으로 관리되는 하나의 정적 변수에 불과한 개념입니다. 내용은 아래와 같습니다. 

```c
#include <semaphore.h> 
sem_t s;
sem_init(&s, 0, 1);

int sem_wait(sem_t *s) { //performed atomically
       decrement the value of semaphore s by one
       wait if value of semaphore s is negative
}

int sem_post(sem_t *s) { //performed atomically
       increment the value of semaphore s by one
       if there are one or more threads waiting, wake one
}
```
init의 마지막 parameter에서 Semaphore의 값을 초기화 합니다. `sem_wait`은 단순히 값을 1 줄입니다. 그리고 그 값이 음수가 된다면 `wait 상태(Sleeping State)`에 들어가게 됩니다. `sem_post`는 단순히 값을 1 증가합니다. 이 때 wait 상태에 있는 스레드들이 존재한다면 그 중 하나를 깨웁니다. 기본적인 사용법은 아래와 같이 `Critical Section`을 wait과 post로 감싸줍니다.

![semaphore-basic](/assets/img/os-semaphore/semaphore_basic.png){: width="30%" height="30%"}

간단히 정리하자면 이와 같습니다. 아래의 그림과 함께 보시죠.

![semaphore-exmaple](/assets/img/os-semaphore/semaphore_example.png){: width="80%" height="80%"}

먼저 Semaphore값을 처음 1로 설정하는 이유는 **Critical Section에 들어갈 수 있는 스레드**가 하나여야 하기 때문입니다. 값이 1인 상태에서 어떠한 스레드가 Critical Section을 수행하기 위해 wait을 호출하면 값은 0이 됩니다. 이 상황에서 그 스레드가 post를 호출하여 다시 값을 1로 올릴 수도 있습니다. 그러나, 그 전에 다른 스레드가 Critical Section에 접근하기 위해 wait을 실행하면 값이 -1이 되고 Sleeping State에 들어가게 됩니다. 이런 식으로 어떤 스레드가 Critical Section을 수행 중일 때 다른 스레드가 접근해봤자 계속해서 값이 `-2, -3, -4 ...`식으로 줄어들게 됩니다. 이를 통해, 우리는 값의 음수 절대값이 **waiting 상태의 스레드 수**와 같다는 점을 알 수 있습니다. 이 상황에서 post를 하게 된다면 waiting인 중인 한 스레드가 깨서 Critical Section을 수행하게 되며 값을 1 증가시키기 때문에 음수의 절대값은 1 줄어들어 waiting 상태의 스레드 수를 정확히 반영하는 것을 확인할 수 있습니다. Critical Section의 수행을 제한하기 때문에 `Locks`의 기능, 스레드를 wait시키고 wake하기 때문에 `Condition Variables`의 기능을 모두 수행할 수 있다고 말할 수 있는 것입니다. 

위와 같은 방식 덕분에 wait 전에 post가 수행이 되는 등의 Context Switching에 의해 야기될 수 있는 문제도 일어나지 않을 수 있습니다.

이렇게 Semaphore가 `Locks`와 `Condition Variables`의 기능 모두 가지고 있기 때문에 구현에 있어서도 꼭 정의대로가 아닌 오른쪽 아래와 같이 그 둘을 이용해서도 구현이 가능합니다.

![semaphore-implementation](/assets/img/os-semaphore/semaphore_implementation.png){: width="65%" height="65%"}

# Producer/Consumer Problem에서의 적용

Producer/Consumer Problem을 Seamphore를 통해 구현한 코드는 아래와 같습니다.

![pc_semaphroe](/assets/img/os-semaphore/pc_semaphore.png){: width="70%" height="70%"}

full, empty Semaphore을 마치 두 `Condition Variable`처럼 활용하는 모습을 볼 수 있습니다. 처음 상태를 반영하기 위해 empty를 1, full은 0으로 설정해 놓습니다. 그렇다면 Producer가 empty를 줄이고 full을 증가하고, Consumer가 반대로 full을 줄이고 empty를 증가하게 됩니다. 그렇기 때문에 Consumer가 empty를 증가하지(Item 가져가지) 않았는데 Producer가 empty를 줄이려(Item을 채우려)하면 wait상태에 들어가게 되는 것이죠. 또한 full도 이 반대로 마찬가지입니다. 그러나 이 코드의 문제는 put과 get이라는 `Critical Section`에 대한 `Race Condition`을 방지하지 못한다는 것입니다. 그 둘을 싸고 있는 empty와 full은 각각 다른 Semaphore이기 때문에 Lock의 기능을 수행해주지는 못하고 있습니. 그래서 우리는 아래와 같이 다른 Semaphore를 하나 더 만들어 Lock의 기능을 수행하게끔 해야하는 것입니다..

![semaphore-lock](/assets/img/os-semaphore/semaphore_lock.png){: width="70%" height="70%"}

전에 구현하였던 Lock대신 Producer와 Consumer의 과정 전체를 mutex라는 `Semaphore`로 감싸주었습니다. 그러나 Lock과 다르게 이 코드에서는 **Deadlock**이라는 치명적인 문제가 발생하게 됩니다.

우리가 전에 `Condition Variable`에서 wait은 Lock을 `release` 해주는 기능이 있음을 배웠습니다. 그러나 Semaphore에서는 그러한 기능이 없기 때문에 문제가 발생할 수 있습니다. 예를 들어 Consumer가 mutex를 통과하여 full을 wait하고 있지만, Producer가 mutex를 통과하지 못해(Consumer가 가지고 있으므로) 둘 다 계속해서 기다리고 있는 교착 상태에 도달하게 됩니다. 이와 같은 현상을 **Deadlock**이라고 하며 방금 설명했던 현상을 그림으로 나타내면 아래와 같습니다.

![pc-deadlock](/assets/img/os-semaphore/pc_deadlock.png){: width="40%" height="40%"}

## Fine-Grained Lock

그래서 우리는 전과는 다르게 Item을 넣고 가져가는 put, get만을 Semaphore로 감싸야 합니다. 이와 같이 lock의 `Scope`를 줄이는 것을 `Fine-Grained Lock`이라고 합니다.

![fg-lock](/assets/img/os-semaphore/fg_lock.png){: width="35%" height="35%"}

## + Reader-Writer Locks

추가적으로, Reader/Writer Problem에 대해서도 알아봅시다. 전에 Producer/Consumer은 Consumer가 Item을 아예 가져만 `Reader`는 값은 그대로 존재하고 읽기만 하기 때문에 약간의 차이가 있게 됩니다. Reader들이 읽는다고 Writer가 바로 값을 써야할 필요는 없기 때문이죠. 위의 코드는 여러 Reader들이 있을 때 첫 번째 Reader가 `writelock`, 즉 Writer가 쓰기를 하지 못하게 lock을 걸고 마지막 Reader가 그 writelock을 해지합니다. 이렇게 되면 Writer는 항상 모든 Reader가 읽기를 기다려야 하기 때문에 `Fairness Problem`을 유발할 수 있습니다. 하물며, 끝날만 하다 싶을 때 Reader가 추가적으로 더 들어와서 Writer가 예상보다도 더 기다리는 상황도 생기기 다반사 이겠죠? 이렇기 때문에 Writer가 `waiting`할 경우에는 추가적인 Reader를 받지 않는 등의 해결책이 필요할 것입니다.