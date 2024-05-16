# 5 Efficient C Programming

> [컴파일에 대한 이해](https://bradbury.tistory.com/226)

자주 실행되고 성능상 중요한 함수만을 최적화할 가치가 있다.  이때, 중요한 함수를 찾기 위해서는 프로파일링 툴을 사용해야 한다.

- 대표적으로 gcc에 내장된 **gprof** 프로그램을 통해 프로파일링할 수 있다. 

  > 단, gcc 컴파일러에 `-pg` 옵션을 추가해서 컴파일해야 한다.

  > Syntax: `gcc [options] [source files] [object files] [-o output file]`

다음은 gcc 컴파일러에서 자주 사용하는 option을 정리한 도표다.

| Option | Description | Detail |
| --- | --- | --- |
| `S` | compile source files to assembly files(`*.s`) | |
| `c` | compile source files to object files(`*.o`) without linking | |
| `o` | build output to an output file | `-O0`(default), `-O1`(or `-O`), `-O2`, `-O3`, `-Ofast` |
| `O{level}` | optimization level | `-O0`(default), `-O1`(or `-O`), `-O2`, `-O3`, `-Ofast` |
| `g{level}` | include debugging information | |

다음은 위 옵션을 사용해 컴파일을 수행한 예시다.

```bash
$ gcc -S samp.c                  # samp.s assembly file 생성

$ gcc -c myfile.c                # myfile.o object file 생성

# -o flag
$ gcc myfile.c -o myfile         # myfile output file 생성
$ ./myfile                       # Program run

# -O flag
$ gcc -O myfile.c -o myfile
$ ./myfile 

# -c, -o flag
$ gcc file1.c file2.c -o execfile # compile file1.c, file2.c and link to output file execfile
$ ./execfile

$ gcc -c file1.c file2.c          # compile file1.c, file2.c to object files file1.o, file2.o(without linking)

# -g flag
$ gcc -g myfile.c -o execfile     # compile myfile.c with debug information and link to output file execfile

# gprof
$ gcc -Wall -pg test.c -o test    # test output file, gmom.out 생성
$ gprof test gmon.out > result.txt
```

> object file 포맷은 윈도우에서 `PE`(Portable Executable), 리눅스에서는 `ELF`(Executable and Linkable Format)을 주로 사용한다. 

---

## 5.1 ARM C Compiler

다음은 ARM C compiler를 이용한 코드 컴파일 예시다.

- 대부분의 syntax는 gcc와 동일하다.

- fromelf를 통해 disassembly할 수 있다.

```bash
$ armcc  -Otime -c -o test.o test.c    # c 소스로부터 object 파일 생성
$ fromelf -text/c test.o > test.txt    # disassembly
```

---

### 5.1.1 Overview of C Compilers and Optimizations

컴파일러는 보수적으로 동작해야 하며, 따라서 컴파일러에 의존하는 최적화에는 한계가 존재한다. 

다음은 C 코드로 작성한, 주소 `data` 에서 $N$ byte만큼 메모리를 지우는 `memclr()` 함수이다. 

```c
// overhead (2)에 따라, *data의 타입으로 char 대신 int가 효율적
void memclr(char *data, int N)  
{
    for (; N>0; N--)
    {
        *data=0;
        data++;
    }
}
```

해당 함수를 컴파일할 때, 두 가지 overhead가 발생한다.

| 원인 | 발생 overhead | 
| --- | --- | 
| (1) condition $N$ > 0? | 최초 iteration에서, $N$ 이 조건을 만족하는지 테스트 필요 |
| (2) `data` is aligned? | (보수적으로) 모든 정렬 방식을 가정해서, 1 byte씩 load-store(`LDRB`, `STRB`) |

하지만 컴파일러가 몇 가지 정보를 가지면, 코드를 보다 효율적으로 수행할 수 있다.

- $N$ 이 4의 배수라는 정보가 있다면, loop unrolling을 통해 조건문 검사를 줄일 수 있다.


- data가 4 byte로 정렬되어 있다면, 명령어(`LDR`, `STR`) 한 번에 4 byte씩 load/store할 수 있다.

---

## 5.2 Basic C Data Types

ARM에서 8-bit, 16-bit load 명령어를 사용할 경우, 32-bit 확장을 거쳐 레지스터에 적재된다.

- **unsigned**: zero padding

- **signed**: sign-extension

값이 32-bit로 확장이 되었기 때문에, 적재 후 다음과 같은 장점을 갖게 된다.

- (+) 적재한 값을 `int` 타입으로 cast해도, 추가적인 명령어가 필요하지 않다. 

- (+) 적재한 8-bit(혹은 16-bit) 값 저장 시, LSB 위치에서 8(혹은 16)-bit를 저장하므로, 추가적인 명령어가 필요하지 않다.

다음은 ARM C 컴파일러에서 수행하는 각 데이터 타입의 매핑을 정리한 도표다.

> ARM 컴파일러에서 `char` 타입은 unsigned로 매핑되므로 주의해야 한다. 
>
> - 예를 들어 `char i`를, `i>=0` 같은  loop condition에서 사용하면 무한 루프가 발생한다.

| C Data Type | Implementation |
| --- | --- |
| `char` | unsigned 8-bit byte |
| `short` | signed 16-bit word |
| `int` | signed 32-bit word |
| `long` | signed 32-bit word |
| `long long` | signed 64-bit word |

---

### 5.2.1 Local Variable Types

지역 변수(local variable)는, 32-bit 데이터 타입(`int`, `long`)을 사용한 선언이 효율적이다.

> 이와 달리, 전역 변수(global variable)는 메모리에 저장되기 때문에, 작은 크기를 갖는 데이터 타입을 사용하는 편이 효율적이다.

---

#### 5.2.1.1 Checksum Example: Counter i Data Type

다음은 64개의 단어가 포함된 데이터 패킷의 값을 합산하는 `checksum()` C 함수이다. (checksum이 정답과 다른 값일 경우, 데이터 패킷에 오류가 있는 것으로 판단)

> TCP/IP와 같은 대부분의 통신 프로토콜에서는, checksum 혹은 cyclic redundancy check(CRC) 같은 루틴을 포함하여 오류를 검사한다.

다음과 같이, `i`의 데이터 타입을 어떻게 선언하는가에 따라서 컴파일 결과가 달라진다.

- `checksum_v1` 함수가, `checksum_v2` 함수에 비해 명령어 하나가 더 필요하다.

  > `AND r1,r1,#0xff`: `char`(0~255) 데이터 타입 값 범위로 제한

- `r0`: 함수의 인자를 전달하는 레지스터. (`*data`)

<table>
<tr>
<td> C code </td> <td> Assembly 1(char i) </td> <td> Assembly 2(unsigned int i) </td>
</tr>
<tr>
<td>

```c
int checksum(int *data)
{
  char i;              // checksum_v1
  // unsigned int i;   // checksum_v2
  int sum = 0;

  for (i=0 ; i < 64; i++)
  {
    sum += data[i];
  }
  return sum;
}
```

</td>
<td> 

```assembly
checksum_v1
        MOV r2,r0       ; r2 = data
        MOV r0,#0       ; sum = 0
        MOV r1,#0       ; i = 0
checksum_v1_loop
        LDR r3,[r2,r1,LSL #2]
        ADD r1,r1,#1    ; r1 = i+1
        AND r1,r1,#0xff ; (char)r1
        CMP r1,#0x40
        ADD r0,r3,r0
        BCC checksum_v1_loop 
        MOV pc,r14
```

</td>
<td> 

```assembly
checksum_v2
        MOV r2,r0
        MOV r0,#0
        MOV r1,#0 
checksum_v2_loop
        LDR r3,[r2,r1,LSL #2] 
        ADD r1,r1,#1    ; i++
        CMP r1,#0x40
        ADD r0,r3,r0
        BCC checksum_v2_loop 
        MOV pc,r14

```

</td>
</tr>
</table>

---

#### 5.2.1.2 Checksum Example: Checksum Data Type (16-bit data packet)

다음은 16bit 데이터 패킷(`*data`)을 받아 checksum을 계산하는 코드다. 먼저 sum을 동일하게 `short` 타입으로 둔 코드를 살펴보자.

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
short checksum_v3(short *data)
{
  unsigned int i; 
  short sum=0;
  
  for (i=0; i<64; i++) 
  {
    sum = (short)(sum + data[i]);
  }
  return sum; 
}



```

</td>
<td> 

```assembly
checksum_v3
        MOV     r2,r0
        MOV     r0,#0
        MOV     r1,#0
checksum_v3_loop
        ADD     r3,r2,r1,LSL #1   ; LDRH 명령 중 shift를 미지원하므로, ADD 명령 필요
        LDRH    r3,[r3,#0]
        ADD     r1,r1,#1
        CMP     r1,#0x40
        ADD     r0,r3,r0
        MOV     r0,r0,LSL #16
        MOV     r0,r0,ASR #16
        BCC     checksum_v3_loop
        MOV     pc,r14
```

</td>
</tr>
</table>

위 코드에서는 두 가지 overhead가 발생한다.

| 원인 | 발생 overhead | 
| --- | --- | 
| (1) `sum = (short)(sum+data[i]);` | `sum + data[i]`는 32bit이므로, 매번 short casting이 필요하다.<br/>(매번 `MOV` 명령어 두 번씩 수행 필요) |
| (2) `ADD r3,r2,r1,LSL #1` | `LDRH`은 피연산자 shift를 지원하지 않는다.<br/>(`ADD` 명령어를 통한 주소 계산이 먼저 필요하다.) |

위 overhead를 개선하는 방안은 다음과 같다.

- (overhead 1) 부분 합계 `sum`의 데이터 타입을 `int`로 두고, 함수의 리턴 타입을 `short`로 둔다.

  함수의 return 과정에서만 casting이 수행된다.

- (overhead 2) `data[i]` 같은 index 기반의 접근 대신, 배열 주소를 증가시킨다.

  `LDRSH` 명령을 활용해 load할 수 있다. (Load Register Signed Halfword)

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
short checksum_v4(short *data)
{
  unsigned int i;
  int sum=0;
  
  for (i=0; i<64; i++)
    {
      sum += *(data++);
    }
  return (short)sum;
}

```

</td>
<td> 

```assembly
checksum_v4
        MOV      r2,#0
        MOV      r1,#0
checksum_v4_loop
        LDRSH    r3,[r0],#2
        ADD      r1,r1,#1
        CMP      r1,#0x40
        ADD      r2,r3,r2
        BCC      checksum_v4_loop
        MOV      r0,r2,LSL #16
        MOV      r0,r0,ASR #16
        MOV      pc,r14
```

</td>
</tr>
</table>

---

### 5.2.2 Function Argument Types

더 나아가서 함수의 인자나 return 타입도, `int`로 선언한 코드가 더욱 효율적이다.

- `int` 타입이 아닌 함수 인자의 전달은, 컴파일러의 종류에 따라서 다르게 수행된다.

  | cast | description | compiler |
  | --- | --- | --- |
  | narrow | 인자가 갖는 타입 크기에 맞게 감소 | gcc |
  | wide | 감소 없이 전달 | armcc |

다음은 gcc, armcc 컴파일러의 인자 전달 과정에서의 차이를 보여주는 코드다.

<table>
<tr>
<td> C code </td> <td> Assembly(armcc) </td> <td> Assembly(gcc) </td>
</tr>
<tr>
<td>

```c
short add_v1(short a, short b)
{
  return a + (b>>1); 
}




```

</td>
<td> 

```assembly
add_v1
        ADD r0, r0, r1, ASR #1 
        MOV r0, r0, LSL #16 
        MOV r0, r0, ASR #16 
        MOV pc, r14



```

</td>
<td> 

```assembly
add_v1_gcc
        MOV r0, r0, LSL #16
        MOV r1, r1, LSL #16
        MOV r1, r1, ASR #17     ; narrow cast
        ADD r1, r1, r0, ASR #16 ; narrow cast
        MOV r1, r1, LSL #16
        MOV r0, r1, ASR #16
        MOV pc, lr
```

</td>
</tr>
</table>

---

### 5.2.3 Division: Signed vs Unsigned Types

덧셈, 뺄셈, 곱셈 연산과 달리, 나눗셈은 `unsigned` 데이터 타입 사이의 연산이 `signed` 타입보다 효율적이다.

- (`signed`) 양수와 음수의 나눗셈은 서로 다른 방식으로 동작한다.

  > 양수에서 1만큼 right shift는 나누기 2와 같지만, 음수는 그렇지 않다.
  > 
  > - 예를 들어 -3(`0x11111100`)을 오른쪽으로 1회 shift하면, -2(`0x11111110`)가 된다.
  > 
  > - 제대로 된 값을 얻으려면, 먼저 1을 더하고(`0x11111101`) shift(`0x11111110`)를 수행해야 한다.

- 정리하면, `signed`에서는 `(x<0) ? ((x+1)>>1) : (x>>1)` 방식으로 나눗셈을 수행한다.

다음은 `signed int`를 대상으로 한 나눗셈 연산 예시다.

- `ADD   r0,r0,r0,LSR #31`: 부호 비트인 1 bit만 남기고 right shift하므로, 음수에서만 +1로 동작한다.

   > 양수는 0을 더하게 되므로 변화하지 않는다.

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int average_v1(int a, int b)
{
  return (a+b)/2;
}

```

</td>
<td> 

```assembly
average_v1
        ADD   r0,r0,r1         ; r0 = a + b
        ADD   r0,r0,r0,LSR #31 ; if (r0<0) r0++
        MOV   r0,r0,ASR #1     ; r0 = r0>>1
        MOV   pc,r14           ; return r0        
```

</td>
</tr>
</table>

---

## 5.3 C Looping Structures

루프문의 반복 횟수에 따라서 코드 효율성이 달라진다.

---

### 5.3.1 Loops with a Fixed Number of Iterations

고정된 횟수 동안 루프할 경우, 다운카운터를 사용하는 구현이 효율적이다. 두 가지 방식을 비교해 보자.

먼저 루프 카운터가 점차 증가하는 다음 예시에서는, `i++` 후 매번 condition(`i<64`)을 비교해야 한다.

- `CMP`, `SUBS`: condition flag 업데이트

- `BCC`(Branch if Carry Clear) : `C`=0 일 때 분기

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int checksum_v5(int *data)
{
  unsigned int i;
  int sum=0;
  
  for (i=0; i<64; i++)
  {
    sum += *(data++);
  }
  return sum; 
}
```

</td>
<td> 

```assembly
checksum_v5
        MOV    r2,r0
        MOV    r0,#0
        MOV    r1,#0 
checksum_v5_loop
        LDR    r3,[r2],#4
        ADD    r1,r1,#1
        CMP    r1,#0x40
        ADD   r0,r3,r0
        BCC    checksum_v5_loop 
        MOV    pc,r14      
```

</td>
</tr>
</table>

하지만 다음과 같이 0을 향해 카운트 다운하도록 구현하여 `CMP` 명령을 제거할 수 있다.

- 매번 0과 비교할 필요 없이, 조건부 실행 명령(`SUBS`)을 통해서 조건을 검사한다.

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int checksum_v6(int *data)
{
  unsigned int i;
  int sum=0;
  for (i=64; i!=0; i--)
  {
    sum += *(data++);
  }
  return sum; 
}
```

</td>
<td> 

```assembly
checksum_v6
        MOV    r2,r0
        MOV    r0,#0
        MOV    r1,#0x40 
checksum_v6_loop
        LDR    r3,[r2],#4
        SUBS   r1,r1,#1
        ADD    r0,r3,r0
        BNE    checksum_v6_loop 
        MOV    pc,r14 
```

</td>
</tr>
</table>

단, 다운 카운터를 사용한 구현은 `i`의 데이터 타입에 주의해야 한다. 

- `(signed) int i`: `i!=0` 대신 `i>0`을 사용하면, underflow 문제로 종료가 불가능하다.(0x80000000)

  > -0x80000000 - 1 = 0x7fffffff > 0

- `unsigned int i` : `i!=0`, `i>0` 모두 정상적으로 동작한다.


참고로 `signed int i`는 해당 문제를 방지하기 위해, 컴파일러에서 명령어 하나를 늘려서 컴파일한다.

```assembly
; i > 0 (signed)
SUB r1,r1,#1  ; i--
CMP r1,#0     ; i!=0
BGT loop
```

> 따라서 signed/unsigned 모두 조건으로 `i!=0`이 바람직하다.

---

### 5.3.2 Loops Using a Variable Number of Iterations

고정되지 않은 임의의 값 $N$ 을 쓰는 루프 문에서, `i!=0`를 사용한 카운트 다운 방식을 적용해 보자.

- 최초 iteration에서 $N!=0$ 을 검사하는 overhead가 발생한다. (`CMP r1,#0`)

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int checksum_v7(int *data, unsigned int N)
{
  int sum=0;
  for (; N!=0; N--)
  {
    sum += *(data++);
  }
  return sum; 
}



```

</td>
<td> 


```assembly
checksum_v7
        MOV    r2,#0
        CMP    r1,#0
        BEQ    checksum_v7_end
checksum_v7_loop
        LDR     r3,[r0],#4    ; r3 = *(data++)
        SUBS    r1,r1,#1      ; N-- and set flags
        ADD     r2,r3,r2      ; sum += r3
        BNE     checksum_v7_loop
checksum_v7_end
        MOV     r0,r2         ; r0 = sum
        MOV     pc,r14        ; return r0
```

</td>
</tr>
</table>

do-while 반복문을 사용하면, 코드 집적도를 보다 높일 수 있다. (명령어 2개 감소)

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int checksum_v8(int *data, unsigned int N)
{
  int sum=0;
  do {
    sum += *(data++);
  } while (--N!=0);
  return sum;
}

```

</td>
<td> 


```assembly
checksum_v8
        MOV     r2,#0
checksum_v8_loop
        LDR     r3,[r0],#4
        SUBS    r1,r1,#1
        ADD     r2,r3,r2
        BNE     checksum_v8_loop
        MOV     r0,r2
        MOV     pc,r14
```

</td>
</tr>
</table>

---

### 5.3.3 Loop Unrolling

**loop unrolling**이란, 한 iteration에서 loop body를 여러 번 수행하여 반복 횟수를 줄이는 기법이다.

- 따라서, `SUBS    r1,r1,#1`, `BNE     checksum_v9_loop` 명령이 호출되는 총 횟수가 줄어든다.

<table>
<tr>
<td> C Code </td> <td> Assembly </td>
</tr>
<tr>
<td> 

```c
int checksum_v9(int *data, unsigned int N)
{
  int sum=0;
  do {
    sum += *(data++);
    sum += *(data++);
    sum += *(data++);
    sum += *(data++);
    N -= 4;
  } while (--N!=0);
  return sum;
}



```

</td>
<td> 


```assembly
checksum_v9
        MOV     r2,#0
checksum_v8_loop
        LDR     r3,[r0],#4
        SUBS    r1,r1,#1
        ADD     r2,r3,r2
        LDR     r3,[r0],#4
        ADD     r2,r3,r2
        LDR     r3,[r0],#4
        ADD     r2,r3,r2
        LDR     r3,[r0],#4
        ADD     r2,r3,r2
        BNE     checksum_v9_loop
        MOV     r0,r2
        MOV     pc,r14
```

</td>
</tr>
</table>

단, loop unrolling은 적절한 프로파일링을 병행했을 때 최고의 효율을 갖는다.

- unrolling size가 너무 클 경우, 성능 저하가 발생할 수 있다.

  cache를 벗어나는 크기로 설정 시, cache spill을 발생시킨다. (cache miss로 인해 memory access 증가)

- 루프 문의 코드 크기가 늘어나는 overhead에 주의해야 한다.

---
