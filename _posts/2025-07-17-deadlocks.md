---
title: "[운영체제 공룡책 10th] Ch.8 교착 상태"
description: >-
  deadlocks
date: 2025-07-17 13:00:00 +0900
categories: [OS]
tags: [OS, Operating System]
pin: false
image:
  path: /assets/img/resources/os_cover.png
---

## 1. 시스템 모델

- 자원(Resource): CPU, 메모리, 파일, 프린터 등
- 자원은 하나의 프로세스만 사용할 수 있음
- 자원 사용 단계:
  1. **Request** (요청): 자원이 없다면 대기
  2. **Allocation** (할당): 자원 획득
  3. **Release** (반환): 자원 반환


---

## 2. 다중 스레드 응용에서의 교착 상태

- 두 스레드가 두 개의 mutex 락을 **다른 순서**로 획득하려고 할 때 교착상태 발생
- mutex락 생성과 초기화
	```c
	pthread_mutex_t first_mutex;
	pthread_mutex_t second_mutex;

	pthread_mutex_init(&first_mutex, NULL) ;
	pthread_mutex-init(&second_mutex, NULL) ;
	```
- 교착 상태 예제
	```c
	/* thread_one은 이 함수를 실행한다 */
	void *do_work_one (void *param)
	{
		pthread_mutex_lock(&first_mutex);
		pthread-mutex_lock(&second_mutex);
		/**
		* Do some work
		*/
		pthreadJiiutex_unlock(&second_mutex);
		pthreadjuutex.unlock(&first_mutex);

		pthread_exit(0);
	}

	/* thread_two는 이 함수를 실행한다 */
	void *do_work_two (void *param)
	{
		pthread_mutex_lock(&second_mutex);
		pthread_mutex_lock(&first_mutex);
		/**
		* Do some work
		*/
		pthread_mutex_unlock(&first_first);
		pthread_mutex_unlock(&second_mutex);

		pthread_exit(0);
	}
	```
	- thread_one: `first_mutex` → `second_mutex`
	- thread_two: `second_mutex` → `first_mutex`
	- 서로 상대 자원을 기다리며 무한 대기 = 교착상태

- **라이브락 (Livelock)**
	- 스레드가 실패한 행동을 **계속해서 시도**할 때 발생
	- 계속 양보만 하며 아무 일도 진행되지 않음
	- 각 스레드가 실패한 행동을 재시도하는 시간을 무작위로 정하면 회피할 수 있음

---

## 3. 교착 상태 특성

- 교착상태 발생을 위한 4가지 필요 조건
	1. **Mutual Exclusion** (상호배제): 자원은 한 번에 하나의 프로세스만 사용 가능
	2. **Hold and Wait** (점유 대기): 자원을 가진 채 다른 자원을 요청
	3. **No Preemption** (비선점): 자원을 강제로 빼앗을 수 없음(선점 불가)
	4. **Circular Wait** (환형 대기): 프로세스들이 원형으로 자원을 기다림

- **Resource Allocation Graph** (RAG, 자원할당그래프)
	![RAG_1](/assets/img/resources/250717/RAG_1.png){: width="600" }
	- 교착상태를 시각적으로 판단하기 위한 그래프 모델
	- 구성 요소
		- **정점(Vertex)**
			- 원형: 프로세스 (`P1`, `P2`, ...)
			- 사각형: 자원 (`R1`, `R2`, ...)
		- **간선(Edge)**
			- 요청 간선: `P → R`
			- 할당 간선: `R → P`
	- 사이클 여부와 교착상태

		| 자원 형태 | 사이클 의미 |
		|-----------|--------------|
		| 단일 인스턴스 | 사이클 → 교착상태 발생 |
		| 복수 인스턴스 | 사이클 존재해도 교착상태가 아닐 수 있음 |

		- 교착 상태를 갖는 RAG

			![RAG_2](/assets/img/resources/250717/RAG_2.png){: width="300" .normal }

		- 사이클이 있지만 교착 상태가 아닌 RAG

			![RAG_3](/assets/img/resources/250717/RAG_3.png){: width="300" .normal}

			- 왜 교착 상태가 아닌가? → 프로세스 T4가 R2를 방출할 수 있어, 해당 자원이 T3에 할당될 수 있게되어 사이클이 없어짐

---

## 4. 교착 상태 처리 방법

1. **Ignore** (무시): 문제를 무시하고, 교착 상태가 시스템에서 절대 발생하지 않는 척하기 (예: Linux, 발생 시 재부팅)
2. **Prevention** (예방): 4가지 조건 중 하나 이상을 사전에 제거
3. **Avoidance** (회피): 교착상태를 사전에 피하는 알고리즘 사용 (예: 은행원 알고리즘)
4. **Detection and Recovery** (탐지 및 복구): 발생 후 탐지하여 해결

---

## 5. 교착 상태 예방

교착 상태가 발생하기 위한 네 가지 필요 조건 중 최소한 하나가 성립하지 않도록 보장하여 교착 상태 발생을 예방한다.

1. **상호배제 제거**: 자원을 공유 가능하게 (현실적으로 어려움)
2. **점유 대기 제거**: 모든 자원을 한 번에 요청
3. **비선점 허용**: 자원을 강제로 회수 가능하게 설계
4. **환형 대기 제거**: 자원에 순서를 지정하고 그 순서대로 요청

---

## 6. 교착 상태 회피

#### 1) **Safe State**
![safe_unsafe_deadlock_space](/assets/img/resources/250717/safe_unsafe_deadlock_space.png){: width="250" .normal }
- 안전 상태 (safe state): 시스템이 safe state에 있다면, 모든 프로세스가 교착상태 없이 자원을 확보하고 종료될 수 있음
- 안전 순서 (safe sequence): 프로세스들이 자원을 받고 실행될 수 있는 순서
- 불안전 상태(unsafe state): 반드시 deadlock을 의미하지는 않지만, 발생할 가능성이 있는 상태

#### 2) **Resource-Allocation Graph Algorithm**

- 각 프로세스와 자원을 노드로 표시
- 간선 두 가지:
	- T → R : 프로세스가 자원 요청 중
	- R → T : 자원이 프로세스에 할당됨
- claim edge (예약 간선): 미래에 요청할 수 있음을 나타냄
- 그래프에 사이클이 생기면 deadlock 발생 가능
	- 단일 인스턴스 자원에만 적용됨
	- 불안전 상태의 자원 할당 그래프

		![safe_unsafe_deadlock_space](/assets/img/resources/250717/unsafe_RAG.png){: width="300" height= "300" .normal}


#### 3) **Banker's Algorithm**

- 스레드가 시작할 때 스레드가 가지고 있어야 할 자원의 최대 개수를 자원 종류마다 미리 신고해야 함
- 자료구조
	- Available: 현재 이용 가능한 자원 수
	- Max: 각 프로세스가 필요로 하는 최대 자원 수
	- Allocation: 현재 할당된 자원
	- Need = Max - Allocation

- **Safety Algorithm**
	- 시스템이 safe한지 판단

	1. `Work`와 `Finish`를 각각 길이 `m`, `n`인 벡터로 설정한다. `Work = Available`, 모든 i에 대해 `Finish[i] = false`로 초기화한다. (i = 0, 1, ..., n−1)
	2. 다음 조건을 모두 만족하는 인덱스 i를 찾는다. (그런 i가 없다면, 4단계로 이동)
		- `Finish[i] == false`
		- `Needᵢ ≤ Work`
	3. `Work = Work + Allocationᵢ`, `Finish[i] = true` 후 2단계로 되돌아간다.
	4. 모든 i에 대해 `Finish[i] == true`이면, 시스템은 **안전한 상태(safe state)**


- **Resource-Request Algorithm**
	- 프로세스의 자원 요청을 처리해도 safe한 상태인지 판단

	1. Requestᵢ ≤ Needᵢ 인지 확인한다.
		- 만족하면 2단계로 이동
		- 만족하지 않으면 reject (해당 스레드가 자신이 선언한 최대 요구량을 초과함)
	2. Requestᵢ ≤ Available 인지 확인한다.
		- 만족하면 3단계로 이동
		- 만족하지 않으면 reject (프로세스 Tᵢ는 자원이 부족하므로 대기해야 함)
	3. 시스템이 요청된 자원을 Tᵢ에게 할당했다고 가정하고 상태를 아래와 같이 수정
		```
		Available = Available − Requestᵢ  
		Allocationᵢ = Allocationᵢ + Requestᵢ  
		Needᵢ = Needᵢ − Requestᵢ
		```
		- 이 변경된 상태에서 다시 Safety Algorithm을 실행해 시스템이 여전히 안전 상태인지 확인해야함. **안전하면 요청 grant**, 불안전하면 요청 reject

---

### 7. 교착 상태 탐지

- 각 자원 유형이 한 개씩 있는 경우 (Single Instance)
	- **Wait-for Graph** (WFG, 대기그래프) 사용
		- 자원 할당 그래프로부터 자원 유형의 노드를 제거하고 적절한 간선들을 결합함으로써 대기 그래프(WFG)를 얻음
	- 대기 그래프가 사이클을 포함하는 경우에만 시스템에 교착 상태가 존재함

- 각 유형의 자원을 여러 개 가진 경우 (Multiple Instances)
	- 대기 그래프 사용 불가 → Banker’s Algorithm 변형하여 이용

- 탐지 알고리즘 활용
	- 탐지 시기 결정 중요:
		- 자원 요청할 때마다 검사: 교착 상태를 야기한 스레드와 연루된 스레드들을 알아낼 수 있으나 오버헤드가 너무 큼
		- 주기적으로 검사: 여러 개의 사이클을 포함할 수 있어 오버헤드를 줄일 수 있지만 교착 상태를 야기한 스레드를 알아낼 수 없음
	- 교착이 발생해도 탐지만으로 끝나는 것이 아니라 해결책이 필요함

---

### 8. 교착 상태 회복

- 방법 1. **프로세스 종료**
	- 교착된 프로세스 모두 중지
		- 확실하게 교착 상태의 사이클을 깨뜨리지만 비용이 큼
		- 프로세스들이 오랫동안 연산했을 가능성이 있으며, 이들 부분 계산 결과 폐기해야하므로 다시 계산해야함
	- 교착 상태가 제거될 때까지 한 프로세스씩 중지
		- 각 프로세스가 중지될 때마다 교착 상태 탐지 알고리즘을 호출해 프로세스들이 교착 상태에 있는지 확인하므로 상당한 오버헤드를 유발함
	- 종료 방식 사용 시, 어느 프로세스를 중지해야 할지를 반드시 결정해야 함
		- 중지시켰을 때 유발되는 비용이 최소인 프로세스들을 중지시켜야 함
		

- 방법 2. **자원 선점**
	- 특정 자원을 회수하여 다른 프로세스에 할당
	- 고려사항:
		1. **희생자 선택** (selection of a victim): 비용 적은 프로세스부터 회수
		2. **후퇴** (Rollback): 프로세스를 안전한 상태로 롤백시킴. 종료된 프로세스가 작업 재개할 수 있도록 상태 저장 필요
		3. **기아 상태** (Starvation)
			- 같은 프로세스만 반복적으로 희생되면 문제 발생
			- 희생자 선택이 주로 비용 요인에 근거한 시스템에서는 동일한 프로세스가 항상 희상자로 선택될 수 있음. 해당 프로세스는 기아 상태에 있게 되므로 **한정된 시간 동안만** 희생자로 선정된다는 것을 반드시 보장해야함 → 비용 요소에 후퇴 횟수 포함하기
