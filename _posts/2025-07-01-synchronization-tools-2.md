---
title: "[운영체제 공룡책 10th] Ch.6 동기화 도구들 - 2"
description: >-
  Synhronization Tools - 2
date: 2025-07-01 13:00:00 +0900
categories: [OS]
tags: [OS, Operating System]
pin: false
image:
  path: /assets/img/resources/os_cover.png
---

## 1. Mutex Locks

- Mutex (Mutual Exclusion)란 ?
	- 여러 스레드, 프로세스가 동시에 공유 자원에 접근하는 것을 막아주는 동기화 기법
	- 임계구역 문제에 대한 하드웨어 기반 해결책은 복잡하며 응용프로그래머는 사용할 수 없음
		- 운영체제 설계자들은 임계구역 문제를 해결하기 위한 상위 수준 소프트웨어 도구 개발 → **Mutex Lock** (가장 간단한 도구)
	- 프로세스는 임계구역에 들어가기 전에 반드시 락을 획득, 임계구역을 빠져 나올 때 락을 반환해야함
    ```java
    while (true)
    {
      acquire lock
        /* critical section */
      release lck
        /* remainder section */
    }
    ```
      - **acquire(): 락 획득**
        ```java
        acquire()
        {
            while (!available)
              ; /* busy wait */
            available = false;
        }
        ```
      - **release(): 락 반환**
        ```java
        release()
        {
          available = true;
        }
        ```

- **바쁜 대기(busy waiting)**
  - 프로세스가 임계구역에 있는 동안 임계구역에 들어가기를 원하는 다른 프로세스들은 acquire() 함수를 호출하는 반복문을 계속 실행해야 함

    → 하나의 CPU 코어가 여러 프로세스에서 공유되는 실제 다중 프로그래밍 시스템에서 문제가 됨. 다른 프로세스가 생산적으로 사용할 수 있는 CPU 주기를 낭비함
        

- 락 경합
  - **경합 상태**: 락을 획득하려고 시도하는 동안 스레드가 봉쇄된 상태
      - 높은 경합 상태: 비교적 많은 수의 스레드가 락을 획득하려고 시도
      - 낮은 경합 상태: 비교적 적은 수의 스레드가 락을 획득하려고 시도
  - **비경합 상태**: 스레드가 락을 얻으려고 시도할 때 락을 사용할 수 있는 상태

- **스핀락(spinlock)**
  - 락을 사용할 수 있을 때까지 프로세스가 **회전**해서, mutex 락 유형을 스핀락이라고도 함
  - 락이 유지되는 기간이 문맥 교환을 두 번 하는 시간보다 짧은 경우 사용
    - 락을 기다리는 스레드는 1) 스레드를 대기 상태로 옮기기 위한 문맥 교환, 2) 락이 사용 가능해지면 대기 중인 스레드를 복원하기 위한 문맥 교환, 총 2번의 문맥 교환이 필요함
  - 장점: 프로세스가 락을 기다려야하고 문맥 교환에 상당한 시간이 소요될 때 문맥 교환이 필요하지 않음. 잠깐 동안 락을 유지해야 하는 경우 다른 스레드가 하나의 코어에서 임계구역을 실행하는 동안 스레드는 다른 처리 코어에서 스핀하고 있을 수 있음

---

## 2. Semaphore

- 세마포란 ?
  - mutex와 유사하게 동작하지만 프로세스 간 동기화 및 자원 접근 제어를 보다 정교하게 수행할 수 있음
  - 초기화를 제외하고, wait()와 signal() 로만 접근 가능, 이 두 연산은 반드시 원자적으로 수행되어야 함
    - **wait()** 
      ```java
      While(S)
      {
        while (S <= 0)
          ; // busy wait
        S--;
      }
      ```
        
    - **signal()**
      ```java
      signal(S)
      {
        S++;
      }
      ```
        - 자원이 없으면 (S ≤ 0) 계속 기다림 (바쁜 대기 → CPU 낭비)
        - 자원이 있으면(S > 0) 세마포 값을 감소시키고 진입
        - signal은 자원을 사용한 후 세마포 값을 증가시켜, 다른 프로세스가 접근 가능하도록 함

- 세마포 사용법
  - 세마포의 종류
    1. **카운팅 세마포** (counting semaphore)
      - 정수 값을 가지며, 자원의 개수를 나타냄. 유한한 개수를 가진 자원에 대한 접근을 제어하는 데 사용될 수 있음 (무한 개 가능)
      - 동작 방식: 각 자원을 사용하려는 프로세스는 세마포에 wait() 연산 수행, 세마포 값 감소함. 프로세스가 자원 방출 할 때 signal() 연산 수행, 세마포 값 증가함. 세마포 값이 0이면 모든 자원 사용 중, 이후 자원을 사용하려는 프로세스는 세마포 값이 0보다 커질 때까지 봉쇄됨
    2. **이진 세마포** (binary semaphore)
      - 값이 0 또는 1만 가질 수 있음 → mutex 락과 유사하게 동작
      - 상호 배제를 위한 목적으로 사용됨 (mutex의 특수한 형태)

  - 세마포 동기화 예시: 두 프로세스 P1과 P2가 순차적으로 실행되어야 할 때
      ```java
      // P1
      S1;
      signal(synch);  // 시그널 보냄
      
      // P2
      wait(synch);    // 시그널 기다림
      S2;
      ```
      - P1과 P2가 세마포 synch를 공유, synch는 0으로 초기화
      - P2는 wait(synch)에서 블록됨
      - P1이 signal(synch)을 호출한 후에야 P2가 실행 가능 → 순서 제어(S1 이후 S2 실행)가 가능해짐

- 세마포 구현
  - 문제점: 바쁜 대기 방식의 비효율
    - 단순한 wait()/signal() 구현은 CPU를 낭비함.
    - 해결책: 프로세스를 대기 큐에 넣고 sleep / wakeup을 통해 제어

  - 구조체 기반의 세마포 정의
    ```java
    typedef struct
    {
      int value;             // 세마포 값
      struct process *list;  // 대기 중인 프로세스 리스트
    } semaphore;
    ```
      

  - 개선된 wait() 구현
    ```java
    wait(semaphore *S)
    {
      S->value--;

      if (S->value < 0)
      {
        add this process to S->list;
        sleep(); // 자기 자신을 멈춤
      }
    }
    ```
    - 세마포 값을 감소하고, 자원이 없으면 대기 리스트에 등록 후 블록됨

  - 개선된 signal() 구현
    ```java
    signal(semaphore *S)
    {
      S->value++;
      if (S->value <= 0)
      {
        remove a process P from S->list;
        wakeup(P); // P를 깨움
      }
    }
    ```
    - 대기 중인 프로세스가 있다면, 하나를 깨워서 실행 재개

  - sleep() & wakeup()
    - **sleep()**: 프로세스를 일시 중지 상태로 전환 (CPU에서 제외됨)
    - **wakeup(P)**: 특정 프로세스 P를 실행 가능 상태(ready)로 전환시킴

      → 이 두 연산은 운영체제에서 제공하는 기본 시스템 호출이며, 대기 기반 세마포 구현의 핵심
          
  - 추가 고려 사항
    - wait()와 signal() 연산은 반드시 원자적으로(atomic) 수행되어야 함
    - 다중 CPU 시스템(SMP)에서는 인터럽트 비활성화, spinlock, compare_and_swap() 등의 기법을 사용해 race condition 방지
    - 커널에서 구현 시, 세마포 값이나 리스트 조작 중 문맥 전환(context switch) 발생하지 않도록 보호해야 함

---

## 3. Moniters

- 모니터란?
  - 세마포는 프로세스 간 동기화에 강력하지만 사용이 까다롭고 오류 발생 가능성이 높음. 이러한 오류를 줄이기 위한 고급 추상화 구조

    - 공유 자원 보호 + 동기화 기능을 함께 제공
    - 명시적 wait(), signal() 로 동기화 수행
    - 자동 mutual exclusion 보장

- 모니터 사용법
  - 모니터 = 추상 자료형(ADT): 공유 자원과 그것을 조작하는 함수 포함
  - 내부적으로 공유 자원 보호를 위한 임계구역, 초기화 코드, 조건 변수(condition) 등이 있음
    ```java
      monitor monitor name
      {
        /* shared variable declarations */

        function P1 ( . . . )
        { . . . }
        
        function P2 ( . . . )
        { . . . }
        
        ...
        
        function Pn ( . . . )
        { . . . }
      }
    ```
  - **조건 변수**는 wait()와 signal() 두 개의 연산 제공
    - 조건 변수 연산
      - x.wait() : x 조건을 만족할 때까지 현재 프로세스 대기
      - x.signal() : x 조건을 기다리는 프로세스 중 하나를 깨움  
        ![monitor](/assets/img/resources/250701/monitor.png){: width="350" }          
    - 조건 변수의 signal 처리 방식에 따른 두 가지 전략
      1. **Signal and wait**
        - P가 Q를 깨우면 P는 일시 정지 상태가 됨
      2. **Signal and continue**
        - P가 Q를 깨우고 그대로 계속 실행함. Q는 대기
      - 일반적으로 signal-and-continue 방식이 더 널리 쓰이며, Java, C#, Python 등의 언어도 이 방식을 채택함

- 세마포를 이용한 모니터 구현

  - 각 모니터는 내부적으로 mutex 세마포로 보호됨
  - signal-and-wait 기법 기반
  - next 세마포어 및 next_count 변수 도입하여 정확한 프로세스 재개 순서 관리
  - 조건 변수 x에 대한 wait()와 signal()은 다음과 같이 구현됨:
    ```java
    // wait()

    x.wait()
    {
      x_count++;
      if (next_count > 0)
        signal(next);
      else
        signal(mutex);
      wait(x_sem);
      x_count--;
    }
    ```
    
    ```java
    // signal()

    x.signal()
    {
      if (x_count > 0)
      {
        next_count++;
        signal(x_sem);
        wait(next);
        next_count--;
      }
    }
    ```

- 모니터 내에서 프로세스 수행 재개
  - conditional-wait 구조
    - `x.wait(c);`
    - c는 우선순위 번호(priority number)를 나타내는 정수 수식(expression)
    - x.signal() 호출 시, 가장 우선순위가 높은 프로세스를 깨움
    - 명시적으로 재개 순서를 제어할 수 있음

  - 예제: 하나의 자원을 할당해 주는 모니터
    ```java
    monitor ResourceAllocator
    {
      boolean busy;
      condition x;
  
      void acquire(int time)
      {
        if (busy)
            x.wait(time);
        busy = true;
      }
  
      void release()
      {
        busy = false;
        x.signal();
      }
  
      initialization_code()
      {
        busy = false;
      }
    }
    ```
    - 자원이 사용 중이면 x.wait(time) 호출
    - release()가 호출되면 x.signal()로 대기 중 하나를 깨움

- conditional-wait 구조
    - `x.wait(c);`
    - c는 우선순위 번호(priority number)를 나타내는 정수 수식(expression).
    - x.signal() 호출 시, 가장 우선순위가 높은 프로세스를 깨움
    - 명시적으로 재개 순서를 제어할 수 있음

- 예제: 하나의 자원을 할당해 주는 모니터
    
  ```java
  monitor ResourceAllocator
  {
    boolean busy;
    condition x;

    void acquire(int time)
    {
      if (busy)
        x.wait(time);
      busy = true;
    }

    void release()
    {
      busy = false;
      x.signal();
    }

    initialization_code()
    {
      busy = false;
    }
  }
  ```
    - 자원이 사용 중이면 x.wait(time) 호출
    - release()가 호출되면 x.signal()로 대기 중 하나를 깨움

- **ResourceAllocator**
  - R은 ResourceAllocator 형태의 인스턴스, 이 구조는 자원을 하나만 다루는 간단한 형태의 모니터
  - 자원 요청 순서 보장 및 자원 상태 추적에 있어 한계가 있음
  - 다음과 같은 문제가 발생할 수 있음
    - 프로세스가 자원에 대한 허락을 받지 않고 자원을 액세스할 경우
    - 프로세스가 자원에 대한 허락을 받은 다음 그 자원을 방출하지 않을 경우
    - 프로세스가 자원에 대한 허락을 받지 않았는데도 그 자원을 방출할 경우
    - 프로세스가 자원에 대한 허락을 받은 다음 방출하지 않은 상태에서 또 그 자원을 요
    청할 경우
  - 해결방안 ? 자원 액세스 연산 자체를 ResourceAllocator 모니터 내부에 두는 것 → 모니터 자체의 스케줄러에게 맡기는 것
  - 프로세스들이 올바른 순서를 지키도록 보장하기 위해서, ResourceAllocator 모니터와 모니터가 관리하는 자원을 사용하는 모든 프로그램을 검사해야 함
    - 조건 1. 프로세스들이 모니터를 정확한 순서에 맞추어 호출하는지 검사해야함
    - 조건 2. 비협조적인 프로세스가 액세스 제어 프로토콜을 사용하지 않아서 모니터가 정한 상호 배제 규칙 경로를 무시하여 공유 자원을 직접 액세스하지 않는다는 것을 보장해야함

---

## 4. Liveness

- 라이브니스란 ?
  - 공룡책) 프로세스가 실행 수명주기 동안 진행되는 것을 보장하기 위해 시스템이 충족해야 하는 일련의 속성
  - 프로세스가 실행 수명 주기 동안 진전할 수 있는지, 즉 무기한 대기 상태에 빠지지 않고 결국에는 작업을 완료할 수 있는지를 보장하는 성질. 이는 교착 상태(Deadlock) 및 무한 대기(Starvation)와 관련이 있음
  - 핵심 개념
    - 라이브니스가 보장되지 않으면, 어떤 프로세스는 무한히 대기 상태에 빠질 수 있음
    - 예: 동기화 도구(Mutex, 세마포어 등)가 잘못 사용되면 발생
    - 운영체제는 이러한 상황을 방지해야 함

- 교착 상태(Deadlock)
  - 정의
    - 두 개 이상의 프로세스가 서로 자원을 기다리며 무한히 대기하는 상태
    - 이벤트(signal)가 발생하길 서로 기다리는 경우 발생
  - 예시 시스템 구성
      ```java
      P0:
      wait(S);
      wait(Q);
      ...
      signal(S);
      signal(Q);
      ```

      ```java
      P1:
      wait(Q);
      wait(S);
      ...
      signal(Q);
      signal(S);
      ```
      - P₀는 S를 기다리고, P₁은 Q를 기다리는 상황에서 서로가 가진 자원을 기다리면서 순환 대기 상태에 빠짐. (전형적인 교착 상태 예)

  - 교착 상태 해결
      - 교착 상태 발생을 방지하거나 회피하는 알고리즘 필요
      - 예: 세마포어 사용 시 자원 획득 순서를 명시하거나, 타임아웃 정책 등

- **우선순위 역전**(Priority Inversion)
  - 정의
    - 낮은 우선순위 프로세스가 가진 자원을 높은 우선순위 프로세스가 기다리는 상황
    - 낮은 우선순위 프로세스를 중간 우선순위 프로세스가 계속 선점하면, 높은 우선순위 프로세스는 무한 대기 → 우선순위 역전 현상
  
  - 예시
    - 세 프로세스: L (낮음), M (중간), H (높음)
    - L이 세마포어 자원을 획득
    - H는 해당 자원을 필요로 함 → L이 자원을 놓을 때까지 대기
    - 이때 M이 계속 실행되면, L은 CPU를 얻지 못하고 자원 해제도 불가 → H는 계속 기다림

  - 해결 방법: **우선순위 상속 프로토콜**(Priority Inheritance Protocol)
    - 낮은 우선순위 프로세스(L)가 높은 우선순위 프로세스(H)의 요청으로 인해 일시적으로 우선순위를 상속받아 자원을 해제할 수 있도록 보장
    - 자원 해제 후 다시 원래 우선순위로 복귀
