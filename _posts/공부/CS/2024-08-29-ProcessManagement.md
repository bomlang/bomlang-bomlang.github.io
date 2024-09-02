---
layout: post
title: "프로세스 관리"
date: 2024-08-29
categories:
  - study
  - CS
description: >
  이화여대 '반효경'교수님의 Operating Systems강의 중, 'Process Management 1 & 2'강의를 수강하고 학습합니다.
---

# 프로세스 생성 (Process Creation)

[\*\*Copy-on-write (COW)](https://www.notion.so/4-Process-Management-e3adea29570744f5a1defc456d9323e3?pvs=21):\*\* 내용이 변경될 때 메모리를 복사합니다.

- \*부모 프로세스(Parent process)**는 **자식 프로세스(Child process)\*\*를 생성합니다.
- 자식이 생성되면, **프로세스 트리**가 형성됩니다.
- **프로세스**는 자원을 필요로 하며, 이를 운영체제로부터 받거나 부모와 공유할 수 있습니다.

### 자원의 공유 모델

1. **모든 자원을 공유하는 모델:** 부모와 자식이 모든 자원을 공유합니다.
2. **일부 자원을 공유하는 모델:** 일부 자원만 부모와 자식이 공유합니다.
3. **전혀 자원을 공유하지 않는 모델:** 일반적으로 사용되는 모델입니다.

### 수행 (Execution)

- 부모와 자식이 동시에 수행되는 경우
- 자식 프로세스가 종료될 때까지 부모 프로세스가 기다리는 경우

### 프로세스 생성 방법

**주소 공간 (Address Space):**

- **자식 프로세스**는 **부모의 공간을 복사**합니다. 이는 실행 중인 프로그램의 코드와 데이터를 포함합니다.
- 자식 프로세스는 기존 공간에 새로운 프로그램을 로드할 수 있습니다.

### 프로세스 종료 (Process Termination)

- **프로세스가 종료되는 경우:**
  - 마지막 명령어를 수행한 후 **운영체제에 이를 알립니다**. (exit 시스템 호출)
  - **자식 프로세스**가 **부모 프로세스**에게 **출력 데이터를 전송**합니다. (wait 시스템 호출을 통해)
  - 프로세스가 사용하던 자원은 운영체제에 반환됩니다.
- **부모 프로세스가 자식 프로세스를 종료시키는 경우:**
  - 자식이 할당된 자원의 한계를 초과한 경우
  - 자식에게 할당된 태스크가 더 이상 필요하지 않을 경우
  - 부모가 종료하는 경우: 이때 운영체제는 자식이 더 이상 수행되지 않도록 합니다.

## Copy-on-Write (COW) 메커니즘

**Copy-on-Write (COW):**

- COW는 효율적인 메모리 관리 기법으로, 프로세스가 `fork()`를 호출할 때 부모 프로세스의 메모리 공간을 자식 프로세스가 복사합니다. 그러나 메모리의 실제 복사는 자식 또는 부모 프로세스가 메모리 내용을 변경할 때까지 지연됩니다. 이 방식은 메모리 사용을 최소화하고 성능을 향상시키는 데 도움이 됩니다.

**COW의 장점:**

- 자식 프로세스가 부모 프로세스의 메모리 내용을 수정하지 않는다면, 메모리 공간이 불필요하게 복사되지 않으므로 자원을 효율적으로 사용합니다.
- 이는 특히 많은 자식을 생성하거나, 자식이 생성된 후 곧바로 새로운 프로그램을 로드하는 경우에 유리합니다. 예를 들어, `fork()`와 `exec()`를 조합하여 자식을 생성하고 새로운 프로그램을 실행할 때, COW는 메모리 사용을 최소화할 수 있습니다.

<br/>

# System call

### 시스템 콜: `fork()`, `exec()`, `wait()`, `exit()`

**fork() 시스템 콜:**

- `fork()`는 새로운 프로세스를 생성하며, 부모 프로세스의 주소 공간을 복사하여 자식 프로세스가 동일한 메모리 공간을 갖도록 합니다. 그러나 자식 프로세스의 PID는 다릅니다.

**exec() 시스템 콜:**

- `exec()`는 기존의 프로세스 메모리 이미지를 새로운 프로그램으로 대체하여 다른 프로그램을 실행할 수 있게 합니다. 이 시스템 호출이 실행되면, 이전에 실행되던 프로그램의 코드와 데이터는 메모리에서 제거되고 새로운 프로그램이 로드됩니다.

**wait() 시스템 콜:**

- 부모 프로세스는 `wait()` 시스템 호출을 사용하여 자식 프로세스가 종료될 때까지 기다릴 수 있습니다. 이 호출이 실행되면, 부모 프로세스는 자식 프로세스가 종료될 때까지 대기 상태로 전환됩니다.

**exit() 시스템 콜:**

- 프로세스는 마지막 명령을 수행한 후 `exit()` 시스템 호출을 통해 운영체제에 자신의 종료를 알립니다. 또한, 자발적 종료 외에도 부모 프로세스에 의해 비자발적으로 종료될 수 있습니다.

## fork() 시스템 콜

- A process is created by the `fork()` system call.
  - creates a new address space that is duplicate of the caller.

```c
int main()
{ int pid;
	print("\n Hello, I am child\n");
	pid = fork();
	if (pid == 0)  /*this is child*/
		print("\n Hello, I am child!\n");

	esle if (pid > 0)  /*this is parent*/
		print("\n Hello, I am parent!\n");
}
```

부모 함수는 `fork()` 를 만난 시점부터 자식을 생성하고, 자식은 `fork()` 를 만난 시점 이후부터 시작된다.

문제점

1. 자식이 부모프로세스를 복제본 취급할 수 있는 문제점
2. fork를 하면 똑같은 프로그램이 만들어지니까 fork로 인해 만들어진 복제본이 똑같은 제어흐름을 따라가야 할 거같은 문제점

⇒ `fork()` 를 실행하고 나면 부모 프로세스는 fork를 실행하면 도출되는 결과값이 자식과는 다르다. 부모 프로세스는 결과값으로 양수를 받고, 자식은 0을 받는다. 이 값으로 부모와 자식을 구분짓는다.

## exec() 시스템 콜

**`exec()` 시스템 호출을 통해 프로세스가 다른 프로그램을 실행할 수 있습니다.**

- 호출자의 메모리 이미지를 새로운 프로그램으로 대체합니다.

```c
int main()
{ int pid;
	pid=fork();
	if(pid == 0)  /*this is child*/
	{ print("\n Hello, I am child! Now I'll run date \n");
		execlp("/bin/date", "bin/date", (char*)0);
	}
		else if(pid > 0)  /*this is parent*/
			print("\n Hello, I am parent!\n");
	}
```

부모 프로세스가 자식을 만들었을때, 자식은 pid가 0을 가지고 if조건문의 첫번째 print를 출력후, `execlp()` 를 만나 새로운 데이터로 다시 태어납니다. 새로운 데이터로 뒤집어 씌워지면 다시 이전 상태로 돌아갈 수 없습니다.

- 만약 `execlp()` 이후에 추가코드가 있다 하더라도 실행을 하지 못합니다.

## wait() 시스템 콜

자식이 종료될 때 까지 실행하는 시스템 콜입니다.

![스크린샷 2024-09-02 오후 3.05.11.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/0b9d597d-ce43-4f8a-afdf-be467bf49fb4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.05.11.png)

- 프로세스 A가 wait() 시스템 콜을 호출하면
  - 커널은 child가 종료될 때까지 프로세스A를 sleep시킵니다. (block상태)
  - Child process가 종료되면 커널은 프로세스A를 깨웁니다. (ready상태)

## exit() 시스템 콜

- 프로세스의 종료
  - 자발적 종료
    - 마지막 statement 수행 후 exit() 시스템 콜을 통해
    - 프로그램에 명시적으로 적어주지 않아도 main함수가 리턴되는 위치에 컴파일러가 넣어줍니다
  - 비자발적 종료
    - 부모 프로세스가 자식 프로세스를 강제 종료시킴
      - 자식 프로세스가 한계치를 넘어서는 자원 요청
      - 자식에게 할당 된 태스크가 더 이상 필요하지 않음
    - 키보드로 kill break등을 친 경우
    - 부모가 종료하는 경우
      - 부모 프로세스가 종료하기 전에 자식을이 먼저 종료됩니다.

<br/>

# 프로세스 간 협력 (Inter-Process Communication, IPC)

**독립적 프로세스 (Independent Process):**

- 독립적 프로세스는 각자 고유한 주소 공간을 갖고 수행되므로 다른 프로세스의 수행에 영향을 미치지 않습니다.

**협력 프로세스 (Cooperating Process):**

- 협력 프로세스는 특정 메커니즘을 통해 서로의 수행에 영향을 미칠 수 있습니다.

**프로세스 간 협력 메커니즘 (IPC):**

![스크린샷 2024-09-02 오후 3.25.03.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/f067727a-8aab-4eb6-a2a7-834647bf23c7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.25.03.png)

- [\*\*메시지 전달 (Message Passing)](https://www.notion.so/4-Process-Management-e3adea29570744f5a1defc456d9323e3?pvs=21):\*\* 커널을 통해 메시지를 전달하는 방식으로, 프로세스 간에 공유 변수를 사용하지 않고 통신합니다.
- **공유 메모리 (Shared Memory):** 서로 다른 프로세스가 일부 주소 공간을 공유하는 방식으로, 효율적인 데이터 교환이 가능합니다.
- 메시지를 전달하는 방법
  - [message passing](https://www.notion.so/4-Process-Management-e3adea29570744f5a1defc456d9323e3?pvs=21): 커널을 통해 메세지 전달
- 주소 강간을 공유하는 방법
  - shared memory: 서로 다른 프로세스 간에도 일부 주소 공간을 공유하게 하는 shared memory 메커니즘이 있습니다.
  - thread: thread는 사실상 하나의 프로세스이므로 프로세스 간 협력으로 보기에는 어렵지만, 동일한 process를 구성하는 thread들 간에는 주소 공간을 공유하므로 협력이 가능합니다.

## Message Passing

- Message system
  - 프로세스 사이에 공유 변수(shared variable)를 일체 사용하지 않고 통신하는 시스템
- Direct Communication
  - 통신하려는 프로세스의 이름을 명시적으로 표시
    ![스크린샷 2024-09-02 오후 3.20.36.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/e4e4f259-aef1-40b3-8f6e-3cf42d547746/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.20.36.png)
- Indirect Communication
  - mailbox (또는 port)를 통해 메시지를 간접 전달
    ![스크린샷 2024-09-02 오후 3.21.37.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/389e2d3d-0ee9-42db-bd56-a05e8fa41986/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.21.37.png)

<br/>

# CPU and I/O Bursts in Program Execution

프로그램 실행 동안 CPU와 I/O 작업이 연속적으로 수행되며, CPU 작업(CPU bursts)과 I/O 작업(I/O bursts)의 길이와 빈도는 서로 다릅니다.

![스크린샷 2024-09-02 오후 3.28.59.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/c678a32b-7b17-4970-a2ef-951f5309f709/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.28.59.png)

### CPU-burst Time의 분포

![CPU-burst Time graph](https://prod-files-secure.s3.us-west-2.amazonaws.com/c220dc6c-8c78-4cae-ae02-81aef8ad3066/f27e1c4b-dac6-4975-8729-e61dc00523ca/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.32.35.png)

CPU-burst Time graph

**CPU-Burst Time의 분포:**

- 대부분의 CPU 시간이 **CPU-bound 작업**에 의해 사용

됩니다. 이는 긴 시간 동안 CPU를 집중적으로 사용하는 작업입니다. 반면, **I/O-bound 작업**은 빈도가 높지만 상대적으로 짧은 CPU 사용 시간을 가지며, 주로 I/O 장치의 응답을 기다리는 시간이 많습니다.

**CPU 스케줄링의 필요성:**

- 다양한 작업(job)이 섞여 있어, 효율적인 CPU 스케줄링이 필요합니다. 특히 I/O-bound 작업이 지나치게 오래 기다리지 않도록 하는 것이 중요합니다. 이를 통해 시스템 자원을 골고루 효율적으로 사용할 수 있으며, 사용자에게 적절한 응답 시간을 제공할 수 있습니다.

<br/>

# CPU 스케줄러와 디스패처 (CPU Scheduler & Dispatcher)

**CPU 스케줄러 (CPU Scheduler):**

- CPU 스케줄러는 **Ready 상태**에 있는 프로세스 중에서 다음에 CPU를 사용할 프로세스를 선택합니다.

**디스패처 (Dispatcher):**

- 디스패처는 CPU 스케줄러에 의해 선택된 프로세스에게 CPU 제어권을 넘깁니다. 이 과정은 **문맥 교환 (Context Switching)**이라 불리며, 프로세스 간의 전환을 의미합니다.

### CPU 스케줄링이 필요한 경우

1. **Running → Blocked:** 예를 들어, I/O 요청과 같은 시스템 호출을 통해 프로세스가 수행을 중단하고 대기 상태로 전환될 때.
2. **Running → Ready:** 할당된 CPU 시간이 만료되어 타이머 인터럽트가 발생했을 때.
3. **Blocked → Ready:** I/O 작업이 완료된 후 대기 중이던 프로세스가 다시 준비 상태로 전환될 때.
4. **Terminate:** 프로세스가 종료될 때.

**스케줄링 방식:**

- **Non-preemptive (비선점형) 스케줄링:** 프로세스가 자발적으로 CPU를 반납하는 경우로, 1번과 4번 상황에 해당합니다.
- **Preemptive (선점형) 스케줄링:** 프로세스가 강제로 CPU를 빼앗기는 경우로, 2번과 3번 상황이 이에 해당합니다.
