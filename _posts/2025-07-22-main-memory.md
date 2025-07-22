---
title: "[운영체제 공룡책 10th] Ch.9 메인 메모리 - 1"
description: >-
  Main Memory - 1
date: 2025-07-22 13:00:00 +0900
categories: [OS]
tags: [OS, Operating System]
pin: false
image:
  path: /assets/img/resources/OS/os_cover.png
---

## 1. 배경

### 1) 기본 하드웨어 (Basic Hardware)

- **CPU는 명령어 실행 시 메모리 접근 필요**
  - 메모리에 저장된 명령어와 데이터를 불러와 실행

- Base & Limit Registers
  - **Base Register**: 프로세스가 접근 가능한 시작 주소
  - **Limit Register**: 접근 가능한 범위 (길이)
    - ex) Base = 1000, Limit = 500 → 접근 가능한 주소는 1000 ~ 1499
  - 범위를 벗어나는 접근 시 운영체제는 **트랩(trap)**을 발생시켜 예외 처리 수행
    ![9.2](/assets/img/resources/OS/ch9/9.2.png){: width="500" .normal}

---

### 2) 주소 바인딩 (Address Binding)

- **compile** 시간: 고정된 주소에서 실행, 절대 주소 사용
- **load** 시간: 실행 시 메모리 위치 변경 가능, 상대 주소 사용
- **execution** 시간: 실행 중에도 위치 변경 가능, MMU 필요

  ![9.3](/assets/img/resources/OS/ch9/9.3.png){: width="300" .normal }

---

### 3) 논리(Logical) vs 물리(Physical) 주소 공간

- 논리 주소 vs 물리 주소

  | **논리 주소** | CPU가 생성하는 주소. 사용자 입장에서 보이는 주소 |
  | **물리 주소** | 실제 메모리(RAM)의 주소. MMU를 통해 변환됨 |

- 논리 주소 공간 vs 물리 주소 공간

  | **논리 주소 공간** | 프로세스가 사용할 수 있는 주소 범위 (ex. 0~2³²-1) |
  | **물리 주소 공간** | 실제 RAM의 주소 범위 (ex. 0~2¹⁹-1) |

- **MMU (Memory Management Unit)** 역할: 논리 → 물리 변환

  ![9.4](/assets/img/resources/OS/ch9/9.4.png){: width="400" .normal }

- 변환 방식
  - 물리 주소 = 논리 주소 + base register
  - MMU가 base를 더하고, limit으로 범위 확인
  - 접근 허용 여부 판단

    ![9.5](/assets/img/resources/OS/ch9/9.5.png){: width="400" .normal }

---

### 4) 동적 적재 (Dynamic Loading)

- 프로그램 전체를 메모리에 적재하지 않고, **필요한 루틴만 동적으로 적재**
- 메모리 사용 효율성 향상 가능

---

### 5) 동적 연결 및 공유 라이브러리 (Dynamic Linking & Shared Libraries)

- 공통 라이브러리를 런타임에 연결하는 방식
  - ex) Windows의 .dll, Linux의 .so 라이브러리

- 장점
  - 메모리 절약 (여러 프로세스가 공유)
  - 프로그램 크기 감소

---

## 2. 연속 메모리 할당

### 1) 메모리 보호 (Protection)

- 잘못된 주소 접근 방지를 위해 MMU는 `base` / `limit` 방식 사용
  - 접근 주소가 `base` ~ `base + limit` 범위 밖이면 트랩(trap) 발생

  ![9.6](/assets/img/resources/OS/ch9/9.6.png){: width="400" .normal }

---

### 2) 메모리 할당 (Allocation)

- **가변 파티션 기법**: 메모리를 할당하는 가장 간단한 방법
  - 운영체제는 사용 가능한 메모리 부분과 사용 중인 부분을 나타내는 테이블을 유지함
  - 처음에는 모든 메모리가 사용자 프로세스에 사용 가능, 하나의 큰 사용 가능한 메모리 블록인 **hole**로 간주
  - 각 프로세스가 메모리를 얼마나 요구하며, 사용 가능한 메모리 공간이 어디에 얼마나 있는지 고려하여 공간 할당함

  ![9.7](/assets/img/resources/OS/ch9/9.7.png){: width="400" .normal }

- 메모리 할당 문제에 대한 해결책
  - **최초 적합 (First-fit)**: 가장 처음 맞는 공간에 배치. 빠르지만 단편화 유발
  - **최적 적합 (Best-fit)**: 가장 작은 맞는 공간에 배치. 공간 활용은 좋지만 느림
  - **최악 적합 (Worst-fit)**: 가장 큰 공간에 배치. 큰 공간을 나눠쓰기 유도

---

### 3) 단편화 (Fragmentation)

- 연속 할당에서의 문제: 단편화

- 종류
  - **외부 (External) 단편화**: 빈 공간이 연속되지 않아서 사용 불가
  - **내부 (Internal) 단편화**: 프로세스에 할당된 공간이 조금 남아 사용 불가

- **50% 규칙**: 첫-fit 전략의 경우 외부 단편화로 전체 메모리의 약 1/3이 낭비된다고 예측

- **압축 (compaction)**: 외부 단편화 해결책
  - 메모리 재배열로 연속된 빈 공간 확보
  - 단, CPU 오버헤드 큼

---
