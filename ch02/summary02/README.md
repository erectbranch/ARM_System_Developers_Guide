# 2 ARM Processor Fundamentals

---

## 2.3 Pipeline

다음은 명령어를 병렬로 처리하는 **pipelining** 과정을 나타낸 그림이다. 각 stage를 나누는 파이프라이닝을 통해, 높은 주파수로 동작이 가능하다.(1 stage = 1 cycle)

![pipeline](images/pipeline_example.png)

> 파이프라인 명령은 **Execute** 단계까지 처리되지 않는다.

> 주의할 점은, steady state까지 latency가 발생하고, instruction dependency를 고려해야 한다.

> 분기 명령이 있으면, 파이프라인에 채워놓은 다음 명령을 실행하면 안 되므로 flush된다.

단, ARM 패밀리에 따라서도 파이프라인 설계 방식이 다르다.

| | |
| :---: | --- |
| ARM9 | ![ARM9 pipeline](images/ARM9_pipeline.png) |
| ARM10 | ![ARM10 pipeline](images/ARM10_pipeline.png) |

---

### 2.3.1 Pipeline Executing Characteristics

파이프라인 실행 특성을 살펴보자.

![ARM instruction sequence](images/pipeline_example_2.png)

> `MSR`(Mode Status Register): I bit를 0으로 설정(clear). ADD 명령이 execute 단계에 진입 시 인터럽트(IRQ)가 활성화된다.

Fetch 단계 이후 Execute 단계에서는, `pc` 값이 최종적으로 8 bytes 만큼 증가한다.(0x8000 $\rightarrow$ 0x8008)

- 단계별로 `pc` 값이 자동으로 4 bytes씩 증가한다. 

  > 단, Thumb mode에서는 2 bytes씩 증가한다.

![pc = address + 9](images/pipeline_example_3.png)

> `LDR`: load to register, `NOP`: no operation, `DC*`: 데이터 할당(`DCD`: 워드 데이터)

> `[pc + \#0]`: pc 가리키는 주소 + 0. 즉, 본래 pc가 가리키는 주소를 의미한다.

---

## 2.4 Exceptions, Interrupts, Vector Table

인터럽트 발생 시, `pc`는 벡터 테이블 내 특정 주소를 가리키도록 설정된다.

- **Vector Table**: 각 entry는, 예외 혹은 인터럽트 처리 루틴으로 분기하는 주소를 가리킨다.

- **Software Interrupt vector**: SWI 명령 실행 시, OS 루틴을 호출하면서 주로 사용

> 일부 프로세서는 high address로부터 벡터 테이블을 시작한다.

| Exception/interrupt | Shorthand | Address | High address |
| --- | --- | --- | --- |
| Reset | RESET | 0x00000000 | 0xffff0000 |
| Undefined instruction | UNDEF | 0x00000004 | 0xffff0004 |
| Software interrupt | SWI | 0x00000008 | 0xffff0008 |
| Prefetch abort | PABT | 0x0000000c | 0xffff000c |
| Data abort | DABT | 0x00000010 | 0xffff0010 |
| Reserved | - | 0x00000014 | 0xffff0014 |
| Interrupt request | IRQ | 0x00000018 | 0xffff0018 |
| Fast interrupt request | FIQ | 0x0000001c | 0xffff001c |

---

## 2.5 Core Extensions

ARM 코어는 다양한 확장 기능을 지원한다.

---

### 2.5.1 Cache, TCM(Tightly Coupled Memory)

cache는 main memory와 core 사이에 위치한 fast memory block이다.

> ARM 기반 임베디드 시스템은, 프로세스 내부에서 단일 레벨 캐시를 주로 사용한다.

실시간 시스템은 코드 실행이 **deterministic**해야 하고, 명령이나 데이터를 load/store하는 데 드는 시간을 예측할 수 있어야 한다. 이를 해결하기 위해 **TCM**(Tightly Coupled Memory)라는 메모리 형태를 사용한다. 

> TCM: core 가까이 위치한 고속 SRAM 메모리로, 명령어 및 데이터를 가져오는 clock period를 deterministic하게 보장한다.

| 폰 노이만 구조 | 하버드 구조 |
| :---: | :---: |
| ![Von Neumann](images/Von_Neumann_architecture.png) | ![Harvard](images/Harvard_architecture.png) |
| memory, instruction: unified cache로 결합 | memory TCM, instruction TCM 별도 구현 |
| ARM 7 | ARM 9, 10, 11 |

혹은 두 구조를 결합하는 방식으로, 각각의 성능 및 deterministic한 장점을 확보할 수 있다.

![hybrid](images/hybrid_architecture.jpg)

---