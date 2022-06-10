---
layout: post
title: "[OS] Limited Direct Execution"
description: OS가 프로세스를 관리하는 법에 대해 알아보자
img: /title/cpu_virtualization.webp
tags: [OS]
---

![direct_execution](/assets/img/os/os-limited-direct-execution/direct_execution.png){: width="50%" height="50%"}

가장 먼저, 도대체 `Direct Execution`가 무엇이고, 또 문제는 무엇인지 부터 살펴봅시다. Direct Execution은 OS가 프로그램 실행에 대한 준비를 해놓은 다음 프로그램 코드는 CPU가 **직접** 돌리는 것입니다. 직관적으로 이야기 하면, 프로그램이 돌아갈 때 만큼은 프로그램이 OS의 손을 완전히 떠나 CPU의 품에 있다는 것입니다. CPU는 정말 **실행만** 시키는 놈이기 떄문에 다음과 같은 두 가지 문제가 생길 수 있지요.

1. Restricted Operations : 프로그램이 디스크에게 I/O request를 발생하거나, CPU나 메모리 같은 시스템 자원에 더 접근하는 등 OS가 원치않는 행동을 해버릴 수가 있습니다. 
2. Switching Between Processes : 여러 프로그램들을 효율적으로 실행시키기 위해 돌아가고 있는 프로그램들 사이사이에 교대를 종종 해줘야 하는데, 그러지를 못합니다.

결국 OS의 통제로 프로그램들을 조금 제한할 필요가 있습니다. OS가 무언가를 통제하지 않는다면 그냥 하나의 라이브러리에 지나지 않겠지요. 그렇게 해서 나온 것이 `Limited Direct Execution`입니다.

# Restricted Operations

첫 번째 문제를 해결한 방법에 대해 알아봅시다. 먼저 다음과 같은 두 가지 프로세서 모드를 도입하였는데, 이를 `Dual-Mode`라 합니다.

- user mode : 기본적인 모드로, 코드들이 할 수 있는 것이 조금 제한된 환경입니다. 예를 들어 user mode에서 프로세스가 I/O request를 발생시킨다면 프로세서가 `exception`을 발생시키고 수행을 막습니다.
- kernel mode : OS, 즉 kernel이 돌아가는 모드로, user mode에서 제한된 `Priviledged`명령어들을 포함한 모든 코드들이 실행 가능한 환겨입니다.

![system_call](/assets/img/os/os-limited-direct-execution/system_call.png){: width="60%" height="60%"}

그래서 user 프로세스가 priviledged operation을 동작하고 싶을 때는 `System Call`을 사용해야 합니다. 시스템 콜이 작동하면 프로그램은 `Trap Instruction`을 동작하게 되는데요. 이 Trap은 caller의 레지스터를 저장한 뒤, kernel 구역으로 점프한 뒤 모드를 kernel mode로 올립니다. 그곳에서 프로세스가 요구했던 user mode에서는 할 수 없었던 모든 privileged operatione들을 실행합니다. 마치고 나면 OS가 `return-from-trap Instruction`을 동작시켜 caller로 돌아오고 다시 모드를 user mode로 낮춥니다.

![system_api](/assets/img/os/os-limited-direct-execution/system_api.png){: width="60%" height="60%"}

조금 더 세부적으로 알아보자면 위와 같습니다. 일반적으로 우리는 direct한 시스템 콜이 아닌 `high-level Application Programming Interface(API)`를 통해 시스템 콜을 사용합니다. read, write, scanf, printf등이 모두 포함되지요. 그러면 그 API 내에 Trap을 발생시키는 시스템 콜이 위치해있습니다. 어떤 시스템 콜을 동작하고 싶은지 명시적인 `System-call-number`를 함께보내면 `Trap Handler`가 `Trap Table`에서 해당 번호에 해당하는 `Jump Address`를 받아 커널로 이동하여 실질적인 코드를 수행하는 것입니다.

또한, 기억해야 할 점은 모드를 바꾸는 과정을 `Hardware`가 담당한다는 것입니다. 하드웨어에서 `Mode bit`을 통해 현재 모드를 저장하고 Trap이나 Interrupt가 발생하였을 때 그 모드를 바꿔줍니다. OS가 프로세스를 통제하는 데에 하드웨어의 서포트를 받는 셈이죠.

![syscall_process](/assets/img/os/os-limited-direct-execution/syscall_process.png){: width="60%" height="60%"}

아까와 달리 이제 OS와 Program 사이에 Hardware가 개입해 있습니다. Hardware는 `Kernel Stack`을 이용해서 프로세스의 레지스터를 저장하고 모드를 옮겨 프로세스 사이에 OS가 개입하도록 해주고 있습니다.

## Interrupt

![interrupt](/assets/img/os/os-limited-direct-execution/interrupt.png){: width="60%" height="60%"}

`Interrupt`란 간단히 얘기해서 예상치 못한 모든 외부 이벤트를 의미합니다. Hardware와 Software 모두에서 발생될 수 있으나, 대게 Interrupt라 하면 H/W를 의미합니다. 먼저, H/W는 CPU에게 신호를 보냄으로써 Interrupt를 발생시킵니다. 예로는 I/O가 완료되었다는 I/O Interrupt, Clock Interrupt, Console Interrupt 등이 있습니다. 

그리고 그 Interrupt 중 Software에 의해 발생한 것을 우리는 `Trap`이라 부르는 것입니다. Trap에는 이 전에 다뤘던 System Call 말고도 돌아가고 있는 프로그램에 에러(ex. div by zero)가 났을 때 발생하는 `Exception`이 있습니다.

![interrupt_handling](/assets/img/os/os-limited-direct-execution/interrupt_handling.png){: width="60%" height="60%"}

Interrupt가 발생했을 때의 과정은 위와 같습니다. 먼저 Interrupt에 대한 프로세스를 실행시키기 위해 현재의 프로세스를 저장해 둡니다. 이 프로세스를 저장해두는 형태, 즉 프로세스의 중요한 정보를 담고 있는 메모리의 단위를 `Process Control Block(PCB)`라고 합니다. 그 후에는, OS의 `Interrupt Handler`가 실행되어 Interrupt에 대한 원인을 분석하고 거기에 맞는 `Interrupt Service Routine(ISR)`을 선택합니다. 그렇게 선택된 ISR이 처리를 한 뒤, 다시 기존이나 다른 프로세스가 실행되는 것입니다.

# Switching Between Processes

이제 두 번째 문제르 해결할 방법에 대해 알아보도록 합시다. 하나의 프로세스가 끝날 때까지 CPU를 점령하는 것을 막기 위해 OS가 적절히 교대해주면서 이를 조절해야 합니다. 그런데 그러려면 프로세스가 실행된는 중간중간 CPU는 **OS의 Code**를 실행해야 하는데, 도대체 언제가 되어야 할까요?

첫 번째 방법은 그냥 무작정 시스템 콜을 기다리는 것입니다. OS가 프로세스를 믿고 얘가 알아서 나 불러주겠지~하고 믿어버리는 거죠. 프로세스가 협렵적인 태도를 취하는 것이 전제되어야 하기에 `Cooperative Approach`입니다. 하지만 프로세스가 악의가 있거나, 버그가 많거나, 무한 루프에 빠지는 등 시스템 콜을 안보내는 상황이 즐비할 수 있게 되지요. 정말 좋지 못한 방법임이 자명합니다.:joy:

그래서 우리는 `Non-Cooperative Approach`를 지향해야 합니다. 그래서 나온 두 번째 방법인 `Timer Interrupt`를 활용하는 방식입니다. 정해진 시간마다 Interrupt를 발생시켜서 그 때 CPU가 OS를 실행, 즉 OS가 통제를 쥐게 하는 것이지요. 

![timer_interrupt](/assets/img/os/os-limited-direct-execution/timer_interrupt.png){: width="60%" height="60%"}

그렇게 통제를 쥔 OS는 현재 프로세스를 잠시 멈추고, 다른 프로세스를 실행시킵니다. 

### Context Switch

그렇다면 OS는 어떻게 이와 같이 현 프로세스를 멈추고 다른 프로세스를 실행시키는 과정을 할까요? 바로 OS가 `Context Swtich`라는 low-level한 코드를 실행함으로써 이루어집니다. Context Switching의 과정은 아래와 같습니다.

![context_switching](/assets/img/os/os-limited-direct-execution/context_switching.png){: width="60%" height="60%"}

이전 Direct Execution에서 어떤 프로세스를 **실행시키는데 필요한 정보**를 담기 위해 하드웨어가 `Kernel Stack`이라는 이름으로 저장했었죠? 이제는 OS 또한 여러 프로세스에 대한 정보를 알고 있어야 Switching이 가능할 거 아니겠습니까? 그래서 이를 `process-structure`, 즉 `PCB`의 형태로 저장하는 것입니다. 그렇게 OS는 PCB를 이용하여 실행시킬 프로세스의 Kernel Stack으로 `Stack Pointer`를 옮김으로써 Context Switching을 하는 것이지요.

그렇게 PCB를 저장하는 것은 `Context Saving`, PCB의 값을 가져오는 것을 `Context Restoring`, 그리고 이렇게 어떤 프로세스를 Context Saving하고 다른 프로세르르 Context Restoring하는 과정을 `Context Switching`이라 명명한 것입니다. Context Switch를 하는 데 드는 시간은 당연하게도 OS나 PCB가 복잡할수록 늘어납니다. 그리고 중간 H/W Support에도 굉장히 큰 영향을 받지요. 

![simple_switching](/assets/img/os/os-limited-direct-execution/simple_switching.png){: width="60%" height="60%"}

보통 이후로 Context Switching을 얘기할 때 위와 같이 중간 과정인 Kernel Stack은 제외한 채로 표현하곤 합니다. 또한 Context Switching을 하는 과정에 있어 OS가 실행되기 때문에 결과적으로 프로세스들이 모두 `idle`한 상태가 필수적으로 생기게 되는데요. Context Switching의 어마어마한 장점을 위해서 이는 어쩔 수 없는 부분이며, 오늘날에는 멀티 프로세서가 너무 발달해서 사실 의미가 없어졌습니다.

> 또한 이처럼 하나의 CPU에서 여러 프로세스들이 돌아가게 하는 것은 `Time-Sharing` 기법이라고 하며, 이는 마치 OS가 하나의 CPU를 여러 개의 CPU가 돌아가는 것처럼 **보이게** 해주는 `CPU Virtualization` 중에 하나입니다.

### 생각해볼 점

그렇다면 이럴 수도 있지 않을까요? 시스템 콜이 수행되는 과정에서 Interrupt가 발생한다면요? 아니면 그 Interrupt를 처리하는 과정에서 또 Interrupt가 발생한다면요? 가장 단순한 해결책은 Interrupt 과정이 일어나고 있을 때는 아예 Interrupt를 `disable` 시켜버리는 것입니다. 그러나 이렇게 되면 일어나야 할 Interrupt를 놓치는 사고가 일어나겠죠. 그래서 조금 더 세련된 방법인 바로 `Locking`입니다. Locking에 대한 개념은 OS의 Locking 포스트에서 확인하실 수 있습니다.

그렇다면 통제를 쥔 OS가 다음에 실행시킬 프로세스를 어떻게 **정할까요?** 이 때 필요한 것이 바로 `Scheduler`로, 이후 포스트에서 다루도록 하겠습니다. 