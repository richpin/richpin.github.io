---
layout: post
title: "[OS] Thread(스레드)"
description: Concurrency를 위한 Thread에 대해 알아보자
img: /title/thread_wiki.png
tags: [OS]
---

이번에는 **Thread(스레드)**에 대해 알아보도록 합시다.

# Thread란 무엇일까?

`Thread`의 기본적인 정의는 어떠한 프로그램 내에서, 특히 프로세스 내에서 실행되는 흐름의 단위를 뜻합니다. 흔히 우리가 아는 프로레스는 사실 하나의 프로그램이고 그 프로세스는 적어도 하나의 이상의 스레드를 가지게 되는 것이죠. 같은 코드를 실행하는 각기 다른 `Program Counter`가 있다고 생각하시면 쉬울 겁니다. 스레드는 `address space`를 공유합니다. 즉 같은 코드와 데이터를 가지고 각기 다른 부분을 실행하는 것이지요. 다만 각각의 스레드는 자신만의 `stack`과 `register(including PC)`을 가지게 됩니다. 병렬성을 위한 존재하는 만큼 context switching이 가능하며 그 단위로 `TCB(Thread Control Block)`을 가집니다. 요즘 대부분의 프로그램들은 `Mult-threaded`로 만들어지고 있죠.

왜 우리는 스레드를 사용해야 할까요?

- Parallelism - 여러 개의 processor가 장착된 컴퓨터에서 여러 스레드를 돌린다면 빨라지겠죠?
- I/O overraping - 하나의 프로세서라도 어떤 스레드가 I/O를 기다리고 있는 동안 다른 스레드가 돌아간다면 매우 효율적일 것입니다.
- light-weight - 프로세스보다 스레드를 만드는 것이 훨씬 가벼운 작업입니다.

> **스레드는 데이터를 공유한다**

다만, 이와 같은 특징으로 인해 `logically separate`한 task에서는 스레드보다 프로세스가 더 나을 수 있습니다. 그러나 같은 코드를 나누어 동시적으로 실행한다는 장점 때문에 어플리케이션들에서는 `Update display`, `Fetch data`, `Answer a network request`와 같이 해야하는 일을 동시다발적으로 돌려 빠르게 효율성을 극대화하고 있습니다.
(+ 커널 또한 Multithreaded입니다.)

다시 정리하자면,

- Responsiveness - 프레세스의 어느 부분이 잠시 막히더라도, 더 `user interface`와 같이 더 중요한 작업들을 계속해서 해나갈 수 있습니다.
- Resource Sharing - 스레드의 가장 큰 특징으로 스레드들끼리는 리소스를 공유합니다.
- Economy - 프로세스보다 훨씬 가볍습니다.
- Scalability - 멀티 프레세스 시스템에서 프로세스들이 이득을 볼 수 있는 것은 다 스레드가 있기에 가능한 일입니다.

# Thread의 치명적 문제점

데이터를 공유한다는 스레드의 가장 큰 특징은 사실 스레드의 가장 큰 약점이 되기도 합니다.
아와 같은 상황을 생각해 봅시다.

![uncontrolled_scheduling](/assets/img/os/os_thread/uncontrolled_scheduling.png){: width="60%" height="60%"}

`counter = counter + 1`이라는 코드를 수행하는 과정입니다. 내부적으로 위와 같이 fetch하고 add연산한 뒤 store하는 일련의 어셈블리 과정을 내포하고 있죠. 그런데 만약 한 스레드가 fetch하고 add를 하는데 store를 하기 전에 다른 스레드로 context switching이 일어났다고 생각해 봅시다. 그렇다면 이 스레드는 전 스레드가 업데이트를 하기 전의 값을 fetch하게 됩니다. 결과적으로 두 스레드가 모두 store하는 값은 51로 같은 값이 되기 때문에 두 번의 연산으로 예상되는 결과가 벌어지지 않는 것입니다. 이를 `indeterminate`하다라고 표현하며 이와 같은 상황을 **Race Condition**이라고 합니다.

- **Race Condition** - 위와 같이 실행 시점의 타이밍이 맞지 않는 문제로 indeterminate하게 이루어지는 결과를 뜻합니다
  - **Critical Section** - `shared variable`에 접근하는 구역으로, Race Condition을 방지하기 위해 두 개 이상의 스레드가 실행되서는 안되는 구역입니다.
  - **Mutual Exclusion** - 한 스레드가 Critical Section을 실행하고 있을 때 다른 스레드가 해당 Critical Section을 실행하지 못하게 보장하는 것입니다.

따라서 우리는 한 번에 끝내야 하는 코드 section을 보장해야합니다. 이를 위해 해당 코드 section 수행 중 다른 스레드로 context switching이 일어나는 것을 방지해야 한다는 것을 의미하기도 합니다. 이를 **atomic**해야 한다고 말하기도 합니다. atomic은 중간에 멈추는 것 없이 아예 다 실행이 안되거나 아예 다 실행히 한 번에 이루어져야 함을 뜻합니다.

> 다행히도 많은 복잡한 atomic instruction 필요 없이, 하드웨어가 synchronization primitives를 위한 몇 가지 명령어들을 제공합니다.

# Race Condition을 해결해보자

Thread를 만들고 관리하기 위해 Thread 라이브러리는 프로그래머들에게 API를 제공합니다. 우리는 그 중 **Pthreads**라는 라이브러리를 이용할 것입니다.

## Pthreads

Thread creation과 synchronization을 위해 만들어진 POSIX standard API입니다. 눈여겨볼 점은 이 라이브러리는 user-level과 kernel-level 모두 동작한다는 것입니다. 사용방법은 아래와 같이 기존 컴파일 명령어에 `-Wall -pthread`를 덧붙이기만 하면 됩니다.

```c
#include <pthread.h>
int pthread_create(pthread_t * thread,
                   const pthread_attr_t * attr,
                   void* (*start_routine)(void*),
                   void* arg);

int pthread_join(pthread_t thread, void **value_ptr);
```

`pthread_create`로 스레드를 생성하면 스레드가 함수 포인터로 전달된 start_routine 함수를 실행합니다. `pthread_join`은 해당 스레드가 끝날 때까지 기다린 후, return 값의 포인터를 value_ptr에 받습니다. 다만 여기서 , 중요한 점은 위에서 설명했듯이 스레드는 각각의 **Stack**을 가지기 때문에 함 수 내에서 선언된 지역 변수에 대한 포인터를 return해봤자 의미 없다는 것입니다.

## Locks

또한 pthread에서 critical section에 대한 mutual exclusion을 제공하기 위해 `Locks`를 제공합니다. 한 스레드가 어떤 `pthread_mutex_t` 변수에 lock을 걸었다고 해봅시다. 그러면 unlock을 할 때까지 다른 스레드는 해당 변수에 lock을 걸지 못합니다. Lock에 대해서는 다음 포스트에서 제대로 다룰 예정입니다.:smile:

아래와 같이 Lock의 intilaization에는 다음과 같은 정적 및 동적 초기화가 있다는 것만 알아둡시다.

```c
pthread_mutex_lock = PTHREAD_MUTEX_INITIALIZER;
int rc = pthread_mutex_init(&lock, NULL); assert(rc == 0);
```

계속 lock이 되기만을 기다리기는 비효율적입니다. 그래서 lock을 시도해본 뒤 걸려있으면 failure을 반환하는 `trylock`이나 lock을 요청하고 설정한 시간이 지나면 failure을 반환하는 `timedlock`등이 있습니다. 이것들은 추후에 설명할 **deadlock**을 방지하기 위한 함수들입니다.

## Condition Variables

어떠한 스레드가 다른 스레드와 관계성을 가지기 위해, 예를 들어 다른 스레드가 특정 조건을 만족할 때까지 기다려야 할 때 우리는 가장 먼저 아래와 같이 `Shared Variable`을 이용하는 방법을 생각해볼 수 있을 겁니다.

![spin_exampel](/assets/img/os/os_thread/spin_example.png){: width="70%" height="70%"}

ready라는 공유 변수를 이용해서 한 스레드는 다른 스레드가 `ready = 1`을 수행할 때까지 실행이 되어서는 안되는 것입니다. `while(ready == 0);`과 같이 기다리는 상황을 **Spin**이라고 표현합니다. 다만 이는 스레드가 `Running`일 때 계속해서 반복문을 수행하면서 CPU 사이클을 잡아먹고 에러까지 도출될 수가 있습니다. 그래서 pthread에서는 아래와 같은 Condition Variable을 제공합니다. 이 또한, 이후 포스트에서 제대로 다룰 예정이니 이쯤에서 넘어가도록 하겠습니다.:smile:

# 번외

스레드에 대해 나머지 여러가지 사실 들을 알아봅시다.

1. 스레드는 `User thread`s와 `Kernel threads`로 나뉩니다.
2. 스레드라는 개념이 북상하면서, 프로그래머가 직접 생성 및 관리를 하는 Explicit이 아닌 컴파일러나 라이브러리가 생성 및 관리를 하는 `Implicit Threading`의 개념이 있습니다.
3. 애초에 여러 스레드를 생성하여 큰 스레드의 집합을 만든 뒤 필요한 request를 Queue하는 `Thread Pool`이라는 개념이 있습니다.
4. 굳이 thread를 생성하지 않아도 `OpenMP`에서는 `#pragma omp parallel`을 사용하면 해당 구역에 대해 코어 수에 따라 가능한 많은 스레드를 알아서 돌려서 처리해 줍니다.
5. `Linux`에서는 threads라는 표현보다는 `tasks`라는 표현을 주로 사용합니다. `clone()` 명령어는 parent task의 주소 공간을 공유하는 child task를 만들어 줍니다.
