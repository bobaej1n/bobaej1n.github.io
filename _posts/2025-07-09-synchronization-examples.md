---
title: "[운영체제 공룡책 10th] Ch.7 동기화 예제"
description: >-
  Synchronization Examples
date: 2025-07-10 13:00:00 +0900
categories: [OS]
tags: [OS, Operating System]
pin: false
image:
  path: /assets/img/resources/os_cover.png
  alt: OS cover img
---

## 1. 고전적인 동기화 문제들

### 1) 유한 버퍼 문제 (The Bounded-Buffer Problem)

n개의 버퍼로 구성된 풀(pool)이 있으며 각 버퍼는 한 항목(item)을 저장할 수 있다고 가정한다. mutex 이진 세마포는 버퍼 풀에 접근하기 위한 상호 배제 기능을 제공하며로 초기화된다. empty와 full 세마포들은 각각 비어 있는 버퍼의 수와 꽉 찬 버퍼의 수를 기록한다. 세마포 empty는 n값으로 초기화되고, 세마포 full은 으로 초기화된다.

- 소비자 생산자 자료구조
  ```c
  int n; // 버퍼 크기

  semaphore mutex = 1;
  semaphore empty = n;
  semaphore full = 0;
  ```

- 생산자 프로세스 구조
  ```c
  while (true)
  {
      . . .
    /* produce an item in next_produced */
      . . .
    wait(empty);
    wait(mutex);
      . . .
    /* add next_produced to the buffer */
      . . .
    signal(mutex);
    signal(full);
  }
  ```

- 소비자 프로세스 구조
  ```c
  while (true)
  {
    wait(full);
    wait(mutex);
      . . .
    /* remove an item from buffer to next_consumed */
      . . .
    signal(mutex);
    signal(empty);
      . . .
    /* consume the item in next_consumed */
  }
  ```

- 특징
  - 생산자와 소비자 코드는 대칭적임
  - 생산자가 소비자를 위해 꽉 찬 버퍼를 생산하고, 소비자는 생산자를 위해 비어 있는 버퍼를 생산함

---

### 2) Readers-Writers 문제 (The Readers-Writers Problem)

하나의 데이터베이스에 여러 프로세스(Reader/Writer)가 동시 접근할 수 있는 상황에서, Reader는 동시 접근 허용, Writer는 배타적 접근 필요. 이를 적절히 조정하지 않으면 데이터 불일치, 기아(starvation), 교착(deadlock) 등이 발생할 수 있다.

- Readers-writers 변형
  - Reader 우선
    - Reader가 하나라도 있으면 Writer는 접근할 수 없음
    - Writer는 기아 상태에 빠질 수 있음
  - Writer 우선
    - Writer가 대기 중이면 새로운 Reader는 접근 불가
    - Reader가 기아 상태에 빠질 수 있음

- (Reader 우선) Readers 프로세스 자료구조
  ```c
  semaphore rw_mutex = 1;
  semaphore mutex = 1;
  semaphore read_count = 0; // 개체를 읽고 있는 프로세스 수
  ```

- Writers 프로세스 구조
  ```c
  while (true)
  {
    wait(rw_mutex);
      . . .
    /* writing is performed */
      . . .
      signal(rw_mutex);
  }
  ```

- Readers 프로세스 구조
  ```c
  while (true)
  {
    wait(mutex);
    read_count++;
    if (read_count == 1)
      wait(rw_mutex);
    signal(mutex);
      . . .
    /* reading is performed */
      . . .
    wait(mutex);
    read_count--;
    if (read_count == 0)
        signal(rw_mutex);
    signal(mutex);
  }
  ```

- Reader-Writer Lock 사용이 유용한 경우
  - 공유 데이터를 읽기만 하는 프로세스가 다수일 경우
  - Reader가 Writer보다 많을 경우 병행성을 높이는 데 유리

---

### 3) 식사하는 철학자들 문제 (The Dininng-Philosophers Problem)
![dining_philosophers](/assets/img/resources/250710/dining_philosophers.png){: w-50 .left}
5명의 철학자가 원형 테이블에 앉아 있고, 포크가 각 철학자 사이에 놓여있다. 철학자가 생각할 때 다른 동료들과 상호 작용 하지 않는다. 두 개의 포크를 모두 얻어야 식사가 가능하며 포크는 자신과 자신의 왼쪽, 오른쪽 철학자 사이에 있는 젓가락을 잡을 수 있다. 다른 사람의 손에 들어간 젓가락을 집을 수 없고, 배고픈 철학자가 동시에 젓가락 두 개를 집으면 식사를 한다. 식사를 마치면 젓가락 두 개를 모두 놓고 다시 생각을 시작한다.

- 세마포 해결안 (교착 상태 해결 안 됨)

  ```c
  semaphore chopstick[5];

  while (true)
  {
    wait(chopstick[i]);
    wait(chopstick[(i+1) % 5]);
      . . .
    /* eat for a while */
      . . .
    signal(chopstick[i]);
    signal(chopstick[(i+1) % 5]);
      . . .
    /* think for awhile */
      . . .
  }
  ```

- 개선된 해결책들
  - 최대 4명만 동시에 식탁에 앉도록 제한
  - 한 철학자가 두 포크를 모두 획득할 수 있을 때만 포크 집기 허용
  - 비대칭 접근: 홀수는 왼쪽 먼저, 짝수는 오른쪽 먼저 포크 집음

- 모니터 해결안
  - 철학자의 상태를 `THINKING`, `HUNGRY`, `EATING`으로 구분
  - 조건 변수(condition variable)를 사용해 포크 사용 제어
  ```java
  monitor DiningPhilosophers
  {
    enum {THINKING, HUNGRY, EATING} state[5];
    condition self[5];

    void pickup(int i)
    {
      state[i] = HUNGRY;
      test(i);
      if (state[i] != EATING)
          self[i].wait();
    }

    void putdown(int i)
    {
      state[i] = THINKING;
      test((i + 4) % 5);
      test((i + 1) % 5);
    }

    void test(int i)
    {
      if (state[(i + 4) % 5] != EATING &&
        state[i] == HUNGRY &&
        state[(i + 1) % 5] != EATING)
        {
          state[i] = EATING;
          self[i].signal();
      }
    }

    void initialization_code()
    {
      for (int i = 0; i < 5; i++)
        state[i] = THINKING;
    }
  }
  ```

- 특징
  - 두 철학자가 동시에 식사하지 않음 보장
  - 교착 상태는 방지하지만, 기아 상태까지 항상 보장하는 것은 아님

---

## 2. 커널 안에서의 동기화

### 1) Windows의 동기화

- 스레드 동기화 방식
  - Windows 커널은 다중 스레드를 지원하며, 스레드 간 동기화를 위해 **dispatcher 객체**를 제공함
  - 대표적인 dispatcher 객체: `mutex`, `semaphore`, `event`, `timer`
  - event: 조건변수와 유사함. 기다리는 조건이 만족하면 기다리고 있는 스레드에 통지할 수 있음
  - timer: 지정한 시간이 만료되면 하나 또는 둘 이상의 스레드에 통지함

- dispatcher 객체의 동작
  - `signaled`: 객체가 사용 가능함 → 스레드가 진행됨
  - `nonsignaled`: 객체가 사용 불가함 → 스레드는 block됨 (대기 큐에 들어감)
  - 커널은 여러 스레드 중 하나를 선택하여 dispatcher 객체를 사용할 수 있게 함

- Mutex 예시
  ![mutex_dispatcher](/assets/img/resources/250710/mutex_dispatcher.png)
  - **signaled** 상태: 스레드는 mutex를 얻고 잠금을 수행
  - **nonsignaled** 상태: 스레드는 대기 큐로 이동
  - mutex는 오직 하나의 스레드만 소유할 수 있음

- **Critical Section 객체**
  - 유저 모드에서 실행되며 커널 개입 없이 동기화 가능
  - 속도 면에서 매우 빠름
  - 동일 프로세스 내의 스레드들만 접근 가능
  - 교착 상태를 방지하려면 회전(lock-free) 대기 등을 사용

---

### 2) Linux의 동기화

- 커널 스케줄링 방식
  - 2.6 이전: 비선점형 커널 → 실행 중인 프로세스를 강제로 중단할 수 없음
  - 현재: 선점형 커널 → 커널 모드에서도 선점 가능

- 동기화 기법
  - 원자적 정수
    - 변수의 값을 한 번에 읽고 쓰는 연산 → `atomic_t` 자료형 사용
    - 예: `atomic_set()`, `atomic_add()`, `atomic_inc()`
  - Mutex 락
    - 커널 안의 임계구역을 보호하기 위해 사용
    - `mutex_lock()` → 락 획득
    - `mutex_unlock()` → 락 해제
    - **재귀적이지 않음**: 같은 스레드가 중복 획득 불가

- Spinlock, Semaphore
  - Spinlock: 짧은 시간 동안 락을 소유해야 할 때 적합 (CPU 낭비 있음)
  - Semaphore: 긴 시간 동안의 락 유지에 적합

- **선점 제어**
  - `preempt_disable()` → 커널 선점 비활성화
  - `preempt_enable()` → 커널 선점 활성화
  - 커널 내부 구조체 `thread_info`의 `preempt_count` 값으로 선점 가능 여부를 판단
  - 락(또는 커널 선점 불가능)이 짧은 시간 동안만 유지될 때 사용

---

## 3. POSIX 동기화

### 1) POSIX mutex 락

Mutex 락은 Pthreads에서 사용할 수 있는 기본적인 동기화 기법이며 코드의 임계 구역을 보호하기 위해 사용된다. 임계 구역에 들어가기 전 mutex를 `lock()` 하고, 나올 때 `unlock()` 해야 한다.

- 주요 함수 및 사용 예
  ```c
  pthread_mutex_t mutex;
  pthread_mutex_init(&mutex, NULL);       // 뮤텍스 초기화
  pthread_mutex_lock(&mutex);             // 락 획득
  /* critical section */
  pthread_mutex_unlock(&mutex);           // 락 해제
  ```

---

### 2) POSIX 세마포

세마포어는 공유 자원의 접근 개수를 조절하는 데 사용된다. 크게 기명(named) 세마포어와 무명(unnamed) 세마포어로 나뉜다.

- **기명 세마포**
  - `sem_open()`을 이용해 생성
  - 이름을 통해 여러 프로세스 간에도 공유 가능
    ```c
    sem_t *sem;
    sem = sem_open("SEM", O_CREAT, 0666, 1);  // 세마포어 생성 및 초기화

    sem_wait(sem);    // 자원 획득
    /* critical section */
    sem_post(sem);    // 자원 반환
    ```

- **무명 세마포**
  - 같은 프로세스 내에서 주로 사용
  - `sem_init()` 사용, 세 개의 매개변수 전달(세마포를 가리키는 포인터, 공유 수준을 나타내는 플래그, 세마포 초기값)
    ```c
    sem_t sem;
    sem_init(&sem, 0, 1);   // 무명 세마포어 초기화

    sem_wait(&sem);         // 자원 획득
    /* critical section */
    sem_post(&sem);         // 자원 반환
    ```

---

### 3) POSIX 조건 변수

조건 변수는 특정 조건이 만족될 때까지 스레드를 기다리게 하거나, 다른 스레드에 신호를 보내어 깨어나게 하는 기능을 한다.

- 주요 함수 및 예시
  ```c
  pthread_mutex_t mutex;
  pthread_cond_t cond_var;

  pthread_mutex_init(&mutex, NULL);
  pthread_cond_init(&cond_var, NULL);

  // 조건 기다리기
  pthread_mutex_lock(&mutex);
  while (a != b)
    pthread_cond_wait(&cond_var, &mutex);
  pthread_mutex_unlock(&mutex);

  // 조건 만족 시 다른 스레드에게 알림
  pthread_mutex_lock(&mutex);
  a = b;
  pthread_cond_signal(&cond_var);
  pthread_mutex_unlock(&mutex);
  ```

- 특징
  - `pthread_cond_wait()`는 mutex를 자동으로 해제한 뒤 조건 만족될 때 다시 획득
  - `pthread_cond_signal()`은 대기 중인 스레드 하나를 깨움

---

## 4. Java에서의 동기화

### 1) Java 모니터

- 특징
  - `synchronized` 키워드를 사용하면 메서드나 블록 단위로 동기화할 수 있음
  - 한 스레드가 객체의 락(lock)을 획득하면, 다른 스레드는 해당 락이 해제될 때까지 대기해야 함

- **진입 집합** (Entry Set)
  - synchronized 메서드를 호출할 때, **다른 스레드가 이미 락을 보유 중이면 진입 집합(entry set)**에 추가됨
  - 락이 해제되면 진입 집합에서 스레드가 **FIFO** 순서로 선택됨(JVM 구현에 따라 다를 수 있음)

  ```java
  /* Producers call this method */
  public synchronized void insert(E item)
  {
    while (count == BUFFER_SIZE)
    {
      try
      {
        wait();
      }
      catch (InterruptedException ie) { }
    } 
    buffer[in] = item;
    in = (in + 1) % BUFFER_SIZE;
    count++;
    
    notify(); // 소비자 깨움
  }

  /* Consumers call this method */
  public synchronized E remove()
  {
    E item;
    while (count == 0)
    {
      try
      {
        wait();
      }
      catch (InterruptedException ie) { }
    }
    item = buffer[out];
    out = (out + 1) % BUFFER_SIZE;
    count--;

    notify(); // 생산자 깨움

    return item;
  }
  ```

---

### 2) 재진입 락 (Reetrant Locks)

- 특징
  - 명시적으로 락을 제어 가능 (`lock()` / `unlock()`)
  - **재진입 가능** (같은 스레드가 여러 번 lock 획득 가능)
  - `try-finally`를 통해 **락 해제 보장**

  ```java
  Lock key = new ReentrantLock();

  key.lock();
  try
  {
    // critical section
  }
  finally
  {
    key.unlock();
  }
  ```

---

### 3) 세마포

- 특징
  - 지정된 개수만큼 락을 허용하는 동기화 도구
  - `acquire()`로 락 획득, `release()`로 해제함

  ```java
  Semaphore sem = new Semaphore(1);

  try
  {
    sem.acquire();
    // critical section
  }
  catch (InterruptedException ie) { }
  finally
  {
    sem.release();
  }
  ```

---

### 4) 조건 변수

- 특징
  - `ReentrantLock`과 함께 사용
  - `wait()` / `notify()`와 유사하지만, 여러 조건 변수 사용이 가능함

  ```java
  Lock lock = new ReentrantLock();
  Condition[] condVars = new Condition[5];

  for (int i = 0; i < 5; i++)
      condVars[i] = lock.newCondition();

  public void doWork(int threadNumber)
  {
    lock.lock();
    try
    {
      if (threadNumber != turn)
        condVars[threadNumber].await();

      // 작업 수행 ...

      turn = (turn + 1) % 5;
      condVars[turn].signal(); // 다음 스레드 깨움
    }
    catch (InterruptedException ie) { }
    finally
    {
      lock.unlock();
    }
  }
  ```

---

## 5. 대체 방안들

### 1) 트랜잭션 메모리

- 특징
  - 공유 메모리에 대한 접근을 **트랜잭션처럼 처리**하여 병행성 문제를 자동으로 해결
  - 성공적으로 끝나면 `commit`, 실패하면 `rollback`
  - 명시적 락 없이도 공유 자원 보호 가능

- 전통적 락 방식과 비교
  ```c
  void update()
  {
    acquire();        // 락 획득
    /* 공유 데이터 변경 */
    release();        // 락 해제
  }
  ```

- 트랜잭션 메모리 방식
  ```c
  void update()
  {
    atomic
    {
        /* 공유 데이터 변경 */
    }
  }
  ```

- 분류
  - **소프트웨어 트랜잭션 메모리 (STM)**: 소프트웨어적으로 구현
  - **하드웨어 트랜잭션 메모리 (HTM)**: CPU 캐시 구조 이용, 성능 우수

---

### 2) OpenMP

- 특징
  - 병렬 프로그래밍을 위한 지시문 기반 API
  - `#pragma omp parallel`, `#pragma omp critical` 등을 통해 병행 실행 및 경쟁 조건 방지
    ```c
    void update(int value)
    {
      #pragma omp critical
      {
        counter += value;
      }
    }
    ```

- 한계
  - 명시적 경쟁 조건은 여전히 고려 필요
  - 중첩된 임계구역은 교착 상태 발생 가능

---

### 3) 함수형 프로그래밍 언어

- 특징
  - C, Java 등 명령형 언어는 상태 변경이 빈번해 병행성에 취약
  - 함수형 언어는 **불변성(immutability)** 을 기본 전제로 하여 병행 환경에 유리

- 장점
  - 상태 변경을 최소화해 공유 자원 충돌 가능성 감소
  - 경쟁 조건 자체를 구조적으로 제거

- 대표 언어
  - **Erlang**: 병렬 시스템에 최적화
  - **Scala**: Java 기반 함수형 언어
