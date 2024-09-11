---
layout: post
title: "CPU 스케줄링 1"
date: 2024-08-30
categories:
  - study
  - CS
description: >
  이화여대 '반효경'교수님의 Operating Systems강의 중, 'CPU Scheduling 1'강의를 수강하고 학습합니다.
---

# CPU Scheduling

긴 프로세스가 CPU를 잡고 놓아주지 않으면, CPU를 잠깐만 필요로하는 I/O같은 경우에는 방치되는 문제점이 있습니다. 이러한 문제로 CPU Scheduling이 필요합니다.

<aside>
💡

현대에서는 선점형 스케줄링을 주로 사용하고있습니다.

</aside>

## 스케줄링 성능 척도 (Scheduling Criteria)

- **CPU 활용도 (CPU Utilization)**
  CPU는 고가의 자원이기 때문에 항상 일을 하도록 유지하는 것이 중요합니다. 이를 통해 CPU를 최대한 활용하는 것이 목표입니다.
- **처리량 (Throughput)**
  일정 시간 동안 완료된 프로세스의 수를 의미합니다. CPU 스케줄링의 목표는 이 처리량을 최대화하는 것입니다.
- **반환 시간 (Turnaround Time)**
  특정 프로세스가 실행을 완료하는 데 걸리는 전체 시간을 의미합니다.
- **대기 시간 (Waiting Time)**
  프로세스가 준비 큐에서 대기하는 시간입니다. 대기 시간이 적을수록 CPU 사용 효율이 높습니다.
- **응답 시간 (Response Time)**
  요청이 제출된 후 첫 번째 응답이 생성되기까지 걸리는 시간을 의미합니다. 이 응답 시간은 타임 공유 환경에서 특히 중요한 성능 척도입니다.

## CPU 스케줄링 알고리즘

### 1. FCFS (First-Come-First-Served)

<img src='/assets/img/공부/CS/scheduling1.png' alt='FCFS를 표현' />

FCFS는 먼저 도착한 프로세스를 먼저 처리하는 **비선점형** 스케줄링입니다. 프로세스가 도착한 순서대로 CPU를 할당받으며, 효율성이 낮은 경우가 많습니다. 예를 들어, 긴 프로세스가 먼저 도착하면 짧은 프로세스들이 오랜 시간 대기해야 하는 문제가 발생할 수 있습니다. 이를 **Convoy Effect**라고 하며, 전체 시스템의 응답성을 저하시킵니다.

<aside>
💡 Convoy effect? ⇒ CPU스케줄링 입장에서는 앞의 P1으로 인하여 긴 시간을 기다려야 하는 상황을 말한다.
</aside>

### 2. SJF (Shortest-Job-First)

<img src='/assets/img/공부/CS/scheduling2.png' alt='Non-Preemptive' />

Non-Preemptive 예제

<img src='/assets/img/공부/CS/scheduling3.png' alt='Premptive' />

Premptive 예제

SJF는 **비선점형** 또는 **선점형**으로 작동할 수 있는 스케줄링 알고리즘입니다. CPU burst time이 가장 짧은 프로세스에게 먼저 CPU를 할당하며, 평균 대기 시간을 최소화할 수 있습니다.

- **비선점형 SJF**: 한 번 CPU를 할당받으면 CPU burst가 끝날 때까지 CPU를 유지합니다.
- **선점형 SJF (Shortest-Remaining-Time-First)**: 더 짧은 CPU burst를 가진 새로운 프로세스가 도착하면 현재 프로세스는 CPU를 잃고 대기 상태로 전환됩니다.

SJF는 최적의 대기 시간을 보장하지만, **기아 현상 (Starvation)** 문제가 발생할 수 있습니다. 긴 CPU burst를 가진 프로세스는 영원히 CPU를 할당받지 못할 가능성이 있으며, CPU burst 시간을 정확히 예측하기 어렵다는 문제도 있습니다.

### 3. 우선순위 스케줄링 (Priority Scheduling)

각 프로세스는 우선순위 번호를 가집니다. 우선순위가 높은 프로세스(숫자가 작을수록 우선순위가 높음)일수록 먼저 CPU를 할당받습니다.

- **선점형**: 우선순위가 높은 프로세스가 도착하면 현재 프로세스는 CPU에서 밀려납니다.
- **비선점형**: 현재 프로세스가 끝날 때까지 CPU를 유지합니다.

SJF는 우선순위 스케줄링의 일종이며, 우선순위가 낮은 프로세스는 영원히 실행되지 않는 **기아 현상**이 발생할 수 있습니다. 이를 해결하기 위한 방법으로 시간이 지남에 따라 우선순위를 높이는 **Aging** 기법이 사용됩니다.

### 4. 라운드 로빈 (Round Robin, RR)

라운드 로빈은 **현대 운영체제에서 가장 많이 사용**되는 스케줄링 방식입니다. 각 프로세스는 동일한 크기의 시간 할당량(time quantum)을 부여받으며, 할당된 시간이 지나면 프로세스는 대기 상태로 전환되고 큐의 뒤로 이동합니다.

- **응답 시간이 매우 짧으며**, 각 프로세스가 공정하게 CPU 시간을 얻을 수 있습니다.
- n개의 프로세스가 있을 때, 각 프로세스는 최대 q 시간 단위마다 CPU의 1/n만큼을 할당받습니다.
- **할당 시간이 크면 FCFS처럼 작동**하고, **할당 시간이 작으면 문맥 교환 (context switch) 오버헤드가 커질** 수 있습니다.

라운드 로빈 방식은 모든 프로세스가 적절한 응답 시간을 보장받도록 설계되어 있으며, 특히 **타임 공유 시스템**에서 매우 효과적입니다.
