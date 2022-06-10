---
layout: post
title: "[OS] Condition Variables"
description: 보다 나은 스레드 운용을 위한 Condition Variables에 대해 알아보자
img: /title/c_condition_variables.jpeg
tags: [OS]
---

이번에는 **Condition Variables**에 대해 알아보도록 합시다.

# Condition Variabels란 무엇일까?

```c
void *child(void *arg) {
    printf("child\n");
    done = 1;
    return NULL;
}
...
Pthread_create(&c, NULL, child, NULL);
while (done == 0)
; // spin
```

위의 예시에서는 child가 해야할 일을 마무리하면 done이라는 공유 변수로 종료를 알리고 있습니다. 여기서는 우리가 이전에 lock에서 공부했던 spin 방법으로 이 스레드의 신호를 기다리고 있습니다. 결국 부모 스레드, 즉 이 상황에서 메인 스레드는 의미 없는 반복문(spin)을 실행시키며 기다리고 있는 것이죠. 이는 엄청난 CPU의 낭비가 아닐 수 없습니다!:triumph: Single-CPU라면 낭비의 효과가 더욱 체감되겠죠?? 따라서 이 spin을 수행하는 것이 아니라 기다리고 있는 스레드를 `Sleeping State` 전환하기 위해 우리는 `Condition Variable`을 사용해야 하는 겁니다.

```c
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c);
```

Condition Variable은 하나의 명시적인 `Queue`입니다. `wait()` 함수는 위에서 설명한대로 기다려야 할 스레드가 Sleeping State, 정확히는 스레드를 Queue에 넣습니다. 'signal()' 함수는 wait()으로 기다리고 있는 스레드에게 신호를 줌으로써 `Ready State`, 즉 스레드를 깨우는 것이죠. 일반적인 사용법의 예시는 아래와 같습니다.

![lock_condition](/assets/img/os/os-condition-variable/lock_condition.png){: width="60%" height="60%"}

Pthread_cond_wait함수의 가장 큰 특징은 기다리는 상태에 들어가면서 **일시적으로 Lock을 release 한 뒤 signal을 통해 깼을 때 다시 lock을 가지는 것입니다.** 왜냐하면 Lock을 건 상태로 기다리면 다른 스레드가 signal을 보낼 여지주차 줄 수 없기 때문이죠. 추가적으로 의문점을 가질 수 있는 질문은 다음과 같습니다.

*Q. 왜 wait/signal 전에 `mutex_lock`을 걸어야 하는가?*

A. `Race Condition`을 방지하기 위해서입니다. 같은 cond_t를 공유하는 wait과 signal이 함께 일어나면 될까요?:joy:

*Q. 무엇 때문에 done이라는 static variable을 사용해야 하는가?*

A. 만약에 wait전에 signal이 호출 되었다고 생각한다면, wait은 오지 않는 signal을 무한하게 기다려야 할 겁니다.

*Q. 그렇다면 왜 done을 확인하는데 while문으로 반복적으로 확인하는가?*

A. 여러 스레드가 wait하기 때문에 벌어지는 문제를 해결하기 위해서입니다. 아래에서 자세히 설명합니다.

# Producer/Consumer(Bounded Buffer) Problem

앞으로를 공부하기 위해 유명한 `Producer/Consumer Problem`을 알아봅시다. 매우 간단합다. Producer은 buffer에 아이템을 가져다 놓고 consumer은 이 item을 가져가는 것입니다.(읽기 및 삭제) 당연하지만 이 buffer은 모든 스레드의 shared resource겠죠?. 먼저 buffer가 하나인 경우의 코드는 아래와 같습니다. 

![consumer-producer](/assets/img/os/os-condition-variable/consumer_producer.png){: width="60%" height="60%"}

알고 넘어가야 할 것은 이 문제에서 producer의 put과 producer의 get이 동시에 일어나는 것을 막기 위해 `lock`으로 감싸줘야 한다는 것입니다. 또 하나는, producer과 아이템을 넣어주고(put) 나서야 consumer가 가져갈 수(get) 있겠죠? 그래서 static variable인 `count`가 필요한 것입니다.

먼저 첫 번째로, **Single Condition Variable**과 **If statement**을 이용하여 구현해봅시다. 

![sc-if](/assets/img/os/os-condition-variable/sc_if.png){: width="60%" height="60%"}

만약의 한 명의 producer, 한 명의 consumer라면 완벽히 작동할 것입니다. 하지만 만약 여러 명의 consumer라면 어떻게 될까요? 두 명의 consumer가 있다고 해봅시다. producer가 아이템을 넣어 wait하고 있는 consumer에게 signal을 주었습니다. 그러나 그 순간에 `Context Switching`으로 다른 consumer가 실행되어 아이템을 가져가버리는 일이 생길 수 있는 겁니다! 거기에 모자라 그 consumer는 원래  기다리고 있었던 consumer에 signal을 보내는 잔인한 짓을 저지르게 되는 거죠...:cold_sweat: `cond variable`이 하나이기 때문에 이와 같은 일이 벌어지는 것입니다. 그림으로 표현하면 아래와 같습니다.

![sc-if-problem](/assets/img/os/os-condition-variable/sc_if_problem.png){: width="40%" height="40%"}

signal을 보낸다고 바로 `Running State`로 스레드가 실행되는 것이 아닌 **Ready State**로 되기 때문에 다른 스레드가 새치기를 할 수 있는 것입니다. 이와 같이 깨어난 스레드가 Run을 하기 전에 어떠한 상태가 바뀌는 것을 `mesa-style`이라고 합니다.

## Mesa-style VS Hoare-style

![mesa-hoare](/assets/img/os/os-condition-variable/mesa_hoare.png){: width="50%" height="50%"}

**Mesa-style** - 대부분의 실제 OS에도 작동하는 방식입니다. 시그널을 보내는 스레드가 `lock/processor`를 가지고 있는 것입니다. 즉 위 그림과 같이 시그널을 보내는 스레드가 lock까지 처리하기 위해 깨운 스레드도 Running이 아닌 Ready State로 유지(`Ready Queue에 들어가는 것`)하는 것입니다. 그렇기 때문에 Ready 상태에 있는 스레드들 중 가장 먼저 실행되는 스레드가 자신이 아닐 경우, 이 스레드는 깨어난 이유가 없어질 수 있는 것입니다. 이 예시에서는 가져갈 값이 이미 없는 거지요.

**Hoare-style** - 이론에서 말하는 방식입니다. 시그널을 보내는 스레드가 `lock/processor`까지 깨운 스레드에게 넘겨주기 때문에 바로 Running하는 것이 가능합니다.

이와 같은 Mesa-style에 문제를 해결하기 위해 두 번쨰 방법으로 `if`대신 **while**을 사용하여 계속해서 condition을 `re-check`해줍니다.

![sc-while](/assets/img/os/os-condition-variable/sc_while.png){: width="50%" height="50%"}

다시 한번 시그널을 받아 깨어났더니 이미 다른 consumer가 아이템을 가져간 스레드가 있다고 가정해봅시다. while이 아니라 전처럼 if라면 바로 의미없는 get과 signal을 실행하게 됩니다. 이런 예기치 못한 경우를 위해 반복적으로 condition을 확인해야 합니다.

그러나 이 경우에도 결국 위에서 발생하였던 하나의 cond variable를 쓰기 때문에 발생하는 문제는 막지 못합니다. consumer가 다른 consumer에게 불필요하게 시그널을 보내게 되기 때문입니다. producer가 consumer의 signal만을 기다리고 있는데 consumer는 다른 consumer에게 signal을 보냅니다. 그 consumer는 깨어나봤자 값이 없기에 다시 wait 상태로 돌아가게 됩니다. 그렇게 된다면 모두가 wait하고 있는 대참사가 벌어지게 되는 거지요!:dizzy_face: 

producer와 consumer **두** 가지의 관계를 확실히 보장하기 위해, 즉 producer는 consumer에게만 consumer는 producer에게만 signal을 보내기 위해 **두** 가지의 cond variable을 가져야 합니다. 그 세 번째 해결책의 코드는 아래와 같습니다. 

![tc-while](/assets/img/os/os-condition-variable/tc_while.png){: width="50%" height="50%"}

이렇게 해결했지만 여전히 buffer는 하나입니다. 더 좋은 `Concurrency`와 더 적은 `signal/wait 빈도`를 위해서 아래와 같이 producer가 buffer를 쭈욱 다 채우고 consumer들이 쭈욱 다 가져가는 아래와 같은 코드를 짤 수 있을 겁니다.

![concurrency-efficiency](/assets/img/os/os-condition-variable/concurrency_efficiency.png){: width="50%" height="50%"}

# 생각해볼 수 있는 또 다른 문제

예를 들어 다음과 같은 시나리오가 있다고 생각해 봅시다.

1. 할당할 수 있는(free) 메모리 공간 X
2. Ta가 allocate(100bytes) 호출 -> 여유 메모리 공간 없어서 wait
3. Tb가 allocate(10bytes) 호출 -> 여유 메모리 공간 없어서 wait
4. Tc가 free(50bytes) 호출

Tc가 50bytes의 여유 공간을 만들어줬으니 Tb에게 signal을 보내줘야 하는 것이 맞지만, 어떻게 할까? 이러한 상황을 위해 `Pthread_cond_broadcast`와 같은 함수가 존재한다. 이는 모든 스레드를 `wake`한다. Ta와 Tb 모두 깨운다면 Tb는 실행되고 Ta는 어차피 while문에 의해 다시 wait모드에 들어갈 것이기 때문에 문제없다. 다만 모든 스레드를 wake하면서 벌어질 수 있는 `negative performance`는 감수를 해야할 것이다. 



