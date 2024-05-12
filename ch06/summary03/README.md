# 6 Writing and Optimizing ARM Assembly Code

`CMP`나 `TST` 등 비교 명령어는 언제나 flag를 업데이트한다. (레지스터 값은 변화하지 않는다)

> `CMP{<cond>}{S} Rn, N`: `Rn-N`을 계산하고 조건 플래그(`N`, `Z`, `C`, `V`) 업데이트
>
> **Remark** (주의: 뺄셈의 경우, Carry 업데이트가 반대로 동작한다.)
> | flag |  | | 1 | 0 | 
> | :---: | --- |--- | --- |--- |
> | N | Negative || 결과의 bit 31이 1(음수) | 결과의 bit 31이 0(양수) |
> | Z | Zero || 결과 = 0 | 결과 $\neq$ 0 |
> | C | Carry | + | Carry 발생 | No carry |
> |   |       | - | No carry | carry 발생 |
> | V | oVerflow || overflow 발생 | overflow 없음 |

다음은 unsigned integer 값을 갖는 `r1`을 다양한 immediate 값과 비교하는 예시다.

<table>
<tr>
<td> Assembly </td> <td> Flag Update </td> 
</tr>
<tr>
<td>

```assembly
; Unsigned comparison
MOV    r1, #11
CMP    r1, #10         ; C=1, Z=0 (CMP는 뺄셈이므로, C=1이면 >임을 의미한다)

MOV    r1, #10
CMP    r1, #10         ; C=0, Z=1

MOV    r1, #9
CMP    r1, #10         ; C=0, Z=0

```

</td>
<td> 

> | Example | flags | Mnemonic |
> | --- | --- | --- |
> | r1 > 10 | C=1, Z=0 | `HI`gher |
> | r1 $\ge$ 10 | C=1 | `H`igher or `S`ame |
> | r1 = 10 | Z=1 | `EQ`ual |
> | r1 < 10 | C=0 | `LO`wer |
> | r1 $\le$ 10 | C=0 혹은 Z=1 | `L`ower or `S`ame |

</td>
</tr>
</table>

반면, signed comparision 경우, 조건에 따라 확인해야 하는 플래그가 다르므로 주의해야 한다.

```assembly
; Signed comparison
MOV    r1, #0         
CMP    r1, #-1          ; C=0, Z=0, V=0, N=0
                        ; signed에서는, (Z=0, N=V) 플래그로 >임을 확인할 수 있다. 
                        ; (unsigned의 (C=1, Z=0) 플래그로는 >임을 확인할 수 없다)
```

---

## 6.5 Conditional Execution

조건 플래그와 조건부 실행을 활용하여, if 문을 최적화할 수 있다. (많은 cycle을 소모하는 분기 명령을 제거할 뿐만 아니라, 코드 크기까지 줄일 수 있다.) 

다음은 unsigned integer 변수 i ( $0 \le i \le 15$ )를 hex 문자로 변경하는 코드다.

<table>
<tr>
<td> C code </td> <td> Assembly </td> 
</tr>
<tr>
<td>

```c
// unsigned integer i (in [0,15])
if (i<10)
{
  c = i + ‘0’;
}
else 
{
  c = i + ‘A’-10;
}
```

</td>
<td> 

```assembly
CMP      i, #10          
ADDLO    c, i, #‘0’       ; i<10.  즉, (C=0)이면 실행
ADDHS    c, i, #‘A’-10    ; i>=10. 즉, (C=1)이면 실행






```

</td>
</tr>
</table>

---

### 6.5.1 Conditional Execution of Data Processing Instructions

조건부 실행은 특히 cascading conditions에서 더욱 강력하다. 다음 코드는 조건문에서 변수 $c$ 가 모음인지 확인한다.

> `TEQ{<cond>}{S} Rn, N`: `Rn^N`(exclusive or)을 계산하고 조건 플래그(`N`, `Z`, `C`, `V`) 업데이트

<table>
<tr>
<td> C code </td> <td> Assembly </td> 
</tr>
<tr>
<td>

```c
if (c==‘a’||c==‘e’||c==‘i’||c==‘o’||c==‘u’)
{
  vowel++; 
}



```

</td>
<td> 

```assembly
TEQ    c, #‘a’             ; 일치할 경우, Z=1
TEQNE  c, #‘e’             ; Z=1이 아닐 때만 실행 (일치 시, Z=1 set)
TEQNE  c, #‘i’
TEQNE  c, #‘o’
TEQNE  c, #‘u’
ADDEQ  vowel, vowel, #1    ; Z=1일 때만 실행
                           ; (c==‘a’ or ‘e’ or ‘i’ or ‘o’ or ‘u’)
```

</td>
</tr>
</table>

> 위 구현은, 조건문의 모든 비교 연산자가 동일한 유형이면 적용할 수 있다.

비슷한 예시를 추가로 살펴보자. 다음은 c가 알파벳 문자인지 확인하는 코드다.

- 덧셈 혹은 뺄셈을 사용하여, $c$ 값의 범위를 $0 \le c \le$ limit 로 한정시킬 수 있다.

  > unsigned comparision을 통해 효율적으로 비교할 수 있다.

<table>
<tr>
<td> C code </td> <td> Assembly </td> 
</tr>
<tr>
<td>

```c
if ((c>=‘A’ && c<=‘Z’)||(c>=‘a’ && c<=‘z’))
{
  letter++; 
}


```

</td>
<td> 

```assembly
SUB    temp, c, #‘A’        
CMP    temp, #‘Z’-‘A’       ;  c-'A'와 'Z'-'A' 비교
SUBHI  temp, c, #‘a’        ;  대문자 범위보다 컸다면, c-'a' 실행(C=1, Z=0)
CMPHI  temp, #‘z’-‘a’       ;  마찬가지로 대문자 범위보다 컸다면, 
                            ;  c-'a'와 'z'-'a' 비교
ADDLS  letter, letter, #1   ;  c-'a'가 'z'-'a' 이하라면 실행(C=0 or Z=1)
```

</td>
</tr>
</table>

> 참고로, AND와 OR 논리 연산은 다음과 같이 서로 반전시킬 수 있다.
>
> | Inverted expression | Equivalent |
> | --- | --- |
> | `!(a && b)` | `(!a) \|\| (!b)` |
> | `!(a \|\| b)` | `(!a) && (!b)` |

---

## 6.6 Looping Constructs

성능에 중요한 대부분의 루틴에는 루프가 포함된다. 루프문을 최적화하는 방법을 살펴보자.

---

### 6.6.1 Decremented Counted Loops

앞서 5장에서, ARM 루프는 다운카운터 방식이 가장 빠르다는 사실을 확인했다.

> 0과 비교 작업에 비용이 없으며, 추가 레지스터 할당이 필요하지 않다.

<table>
<tr>
<td> N to 1 </td> <td> N-1 to 0 (배열에 적합) </td> 
</tr>
<tr>
<td>

```assembly
        MOV  i, N
loop
        SUBS i, i, #1    ; i = N, N-1, ..., 1
        BGT  loop
```

</td>
<td> 

```assembly
        SUBS i, N, #1
loop
        SUBS i, i, #1    ; i = N-1, N-2, ..., 0
        BGE  loop
```

</td>
</tr>
</table>

경우에 따라 1씩 감소하는 것이 아닌, 다른 step 크기를 갖는 루프 카운터가 필요할 수 있다. 다음 코드는 N/3 루프 카운터를 사용하는 예시다.

- 매번 N/3을 계산하기보다, 반복마다 3을 빼는 카운터가 더 효율적이다.

```assembly
        MOV  i, N         ; i = N
loop
        SUBS i, i, #3     ; iterates (round up)(N/3) times
        BGT  loop         ; i > 3이면 loop
```

---

### 6.6.2 Unrolled Counted Loops

**loop unrolling**이란, 한 루프에서 루프 본문을 여러 개 실행하여 루프 오버헤드를 줄이는 방안이다. 

- (1) 성능에 중요한 영향을 끼치는 코드만 Unroll

- (2) 루프 카운터는 unroll한 크기의 배수에 맞춘다.

> 하지만, 2번째 주의사항은 꼭 만족할 수 없는 경우가 발생한다.
>
> - 루프 횟수가 unroll한 크기의 배수가 아닌 경우
>
> - 루프 횟수가 unroll한 크기보다 작은 경우

---

#### 6.6.2.1 Example: memset()

> [MSRC: Building Faster AMD64 Memset Routines](https://msrc.microsoft.com/blog/2021/01/building-faster-amd64-memset-routines/)

C 라이브러리 함수 `memset`을 기반으로 구성한 코드를 최적화하는 방법을 살펴보자.

- `void my_memset(char *s, int c, unsigned int N);`: 메모리 주소 s에서, $N$ bytes 만큼을 byte value c로 설정한다.

문제는 바이트 크기 $N$ 및 메모리 주소 $s$ 의 정렬 여부에 따라서, 최적의 `memset` 알고리즘이 다르다.

| 최적화 문제 | | 임계치 |
| --- | --- | --- |
| 정렬 여부 | $N$ 이 충분히 클 경우에만 정렬할 가치가 있다. | 정렬 여부를 $T_1$ 으로 결정(최소 3 이상) |
| load-store 명령어 | $N$ 의 크기에 따라, 가장 효율적인 명령이 다르다.<br/>(`STRB`(1 byte), `STR`(4 byte), `STMIA`(128 bytes) 등) | `STMIA` 사용 여부를 $T_2$ 로 결정(최소 128 이상) |

다음 코드는 `memset` 함수를, $N$ 에 따라 세 가지 구간으로 나누어 구성한 `my_memset` 함수이다. ( $N =128N_h +4N_m +N_l$ )

> 최적 임계값 $T_1$ 및 $T_2$ 을 찾기 위해서는, 여러 $N$ 을 입력으로 한 결과 cycle 수를 분석해야 한다. 

> | N range | store instruction | label | 
> | --- | --- | --- |
> | $0 \le N < 4$ | `STRB`(1 byte) | memset_1ByteBlk | 
> | $4 \le N < 128$ | `STR`(4 byte) | memset_4ByteBlk | 
> | $N \ge T_2$ | `STMIA`(128 bytes) | aligned |

> - `BCC`(Branch if Carry Clear) : `C`=0 일 때 분기
>
> - `BEQ`(Branch if Equal) : `Z`=1 일 때 분기

```
s   RN 0    ; current string pointer
c   RN 1    ; the character to fill with
N   RN 2    ; the number of bytes to fill

c_1 RN 3    ; copies of c
c_2 RN 4
c_3 RN 5
c_4 RN 6
c_5 RN 7
c_6 RN 8
c_7 RN 12

        ; void my_memset(char *s, unsigned int c, unsigned int N)
my_memset
        ;----------------------------------------------- 
        ; First section aligns the array
        CMP    N, #T_1              ; T_1>=3
        BCC    memset_1ByteBlk      ; N<T_1일 경우, memset_1ByteBlk로 분기
        ANDS   c_1, s, #3           ; c_1 = 메모리 주소 s & 0x3 mask
        BEQ    aligned              ; BEQ: Z flag set 시 분기
                                    ; (s가 4의 배수, 즉 정렬일 경우에만 Z set)
        RSB    c_1, c_1, #4         ; c_1 = 4-c_1 (정렬할 바이트 수)
        SUB    N, N, c_1            ; N -= c_1    (정렬할 바이트 수 제외)
        CMP    c_1, #2              ; 2와 비교하여 c_1 값 특정
        STRB   c, [s], #1           ; s에 c 1 byte 작성 (이후 c += 1 증가)
        STRGEB c, [s], #1           ; c_1 >= 2 이면, 추가 1 byte 작성
        STRGTB c, [s], #1           ; c_1 >  3 이면, 추가 1 byte 작성

aligned                             ; 정렬 이후
        ORR    c, c, c, LSL#8       ; c [15:8]을 c[7:0]과 동일하게 설정
        ORR    c, c, c, LSL#16      ; c 전체를 기존 c x 4로 구성

        ;----------------------------------------------- 
        ; Second section writes blocks of 128 bytes
        CMP    N, #T_2              ; T_2 >= 128
        BCC    memset_4ByteBlk      ; N<T_2이면 memset_4ByteBlk로 분기
        STMFD  sp!, {c_2-c_6}       ; save scratch registers
        MOV    c_1, c               ; c_1-c_7에 c 값 복사
        MOV    c_2, c
        MOV    c_3, c
        MOV    c_4, c 
        MOV    c_5, c 
        MOV    c_6, c
        MOV    c_7, c
        SUB    N, N, #128            ; N -= 128
loop128                              ; 32 words 단위 작성(128 bytes)
        STMIA  s!, {c, c_1-c_6, c_7} ; s에 c, c_1-c_6, c_7 8 words 작성(이후 wb)
        STMIA  s!, {c, c_1-c_6, c_7} ;              〃
        STMIA  s!, {c, c_1-c_6, c_7} ;              〃
        STMIA  s!, {c, c_1-c_6, c_7} ;              〃
        SUBS   N, N, #128            ; N -= 128 (multiple store 조건 확인)
        BGE    loop128               ; N >= 128이면 루프 반복
        ADD    N, N, #128            ; N < 128일 경우, N += 128로 복구
        LDMFD  sp!, {c_2-c_6}        ;       〃     , 스크래치 레지스터 복구

        ;-------------------------------------------- 
        ; Third section deals with left over bytes
memset_4ByteBlk
        SUBS   N, N, #4              ; N -= 4
loop4                                ; 4 bytes 단위 작성
        STRGE  c, [s], #4            ; N > 4이면, s에 4 bytes 작성(이후 wb)
        SUBGES N, N, #4              ; N > 4이면, N -= 4
        BGE    loop4                 ; 위 명령에서 N > 4이면 루프 반복
        ADD    N, N, #4              ; N < 4이면, N += 4로 복구

memset_1ByteBlk
        SUBS   N, N, #1              ; N -= 1
loop1                                ; 1 byte 단위 작성
        STRGEB c, [s], #1            ; N > 1이면, s에 1 byte 작성(이후 wb)
        SUBGES N, N, #1              ; N > 1이면, N -= 1
        BGE    loop1                 ; 위 명령에서 N > 1이면 루프 반복
        MOV    pc, lr                ; return
```

프로파일링 시, 세 구간의 소모 \#cycle은 다음과 같다.

- 최적 임계치: $T_1 = 5$ , $T_2 = 128$ 

| N range | cycle(ARM9TDMI 기준) |
| --- | --- |
| $0 \le N < T_1$ | $640N_h +20N_m +5N_l +6$ |
| $T_1 \le N < T_2$ | $160N_h +5N_m +5N_l +17+5Z_l$ |
| $N \ge T_2$ | $36N_h +5N_m +5N_l +32+5Z_l +5Zm$ |

---

### 6.6.3 Multiple Nested Loops

중첩 루프문에서 각 루프 카운트의 bit width 합산이 32-bit를 초과하지 않는다면, 단 하나의 루프 카운터만으로 모든 루프를 수행할 수 있다.

다음은 matrix multiplication $A^{(R \times T)} = B^{(R \times S)} \cdot C^{(S \times T)}$  연산을 수행하는 코드다. (예제: $R=S=T=40$ 로 설정)

- 행렬 $A[i,j]$ byte address: `&A[i,j] = a + 4*(i*T+j)`

- 레지스터 카운터(`count`)는 다음과 같은 값을 갖게 된다.

  좌측부터 순서대로 `k` loop, `j` loop, `i` loop에 대응된다.

  > 예를 들어, `S-1-k`는 'S-1 to 0'를 의미한다. 

  ![multiple nested](https://github.com/erectbranch/ARM_System_Developers_Guide/blob/master/ch06/summary03/images/multiple_nested_loop.png)

> `BPL`(Branch if PLus) : `N`=0 일 때 분기

<table>
<tr>
<td> C code </td> <td> Assembly </td> 
</tr>
<tr>
<td>

```c
#define R 40
#define S 40
#define T 40
void ref_matrix_mul(int *a, int *b, int *c)
{
unsigned int i,j,k;
int sum;
for (i=0; i<R; i++)
{
  for (j=0; j<T; j++)
  {
    /* calculate a[i,j] */
    sum = 0;
    for (k=0; k<S; k++)
    {
     /* add b[i,k]*c[k,j] */
     sum += b[i*S+k]*c[k*T+j];
    }
    a[i*T+j] = sum;
  }
} }






















```

</td>
<td> 


```assembly
R     EQU 40
S     EQU 40
T     EQU 40

a     RN 0     ; R rows × T columns 행렬 포인터
b     RN 1     ; R rows × S columns 행렬 포인터
c     RN 2     ; S rows × T columns 행렬 포인터
sum   RN 3 
bval  RN 4 
cval  RN 12 
count RN 14

        ; void matrix_mul(int *a, int *b, int *c)
matrix_mul
        STMFD  sp!, {r4, lr}                 ; 스택에 r4, lr 저장
        MOV    count, #(R-1)                 ; (i 카운터) i=(R-1) 초기화

loop_i
        ADD    count, count, #(T-1) << 8     ; (j 카운터) j=(T-1) 초기화

loop_j
        ADD    count, count, #(S-1) << 16    ; (k 카운터) k = (S-1) 초기화
        MOV    sum, #0                       ; sum = 0 초기화

loop_k
        LDR    bval, [b], #4                 ; bval = B[i,k] (이후 b = &B[i, k+1])
        LDR    cval, [c], #4*T               ; cval = C[k,j] (이후 c = &C[k+1,j] )
        SUBS   count, count, #1 << 16        ; (k 카운터) S-1 to 0 다운 카운트
        MLA    sum, bval, cval, sum          ; MAC 연산 수행(sum += B[i, 1~S]*C[1~S, j])
        BPL    loop_k                        ; k 카운터 >= 0이면 loop_k 반복
        STR    sum, [a], #4                  ; A[i,j] = sum (누적 완료 시 저장)
         
        SUB    c, c, #4*S*T                  ; c = &C[0,j] 복구
        ADD    c, c, #4                      ; j++ 주소 이동
        ADDS   count, count, #(1<<16)-(1<<8) ; (k, j 카운터) 음수된 k 카운터 복구 및 j 다운 카운트
                                             ; ((1<<16)-(1<<8) = 0xf0)
        SUBPL  b, b, #4*S                    ; j 카운터 >= 0이면, b = &B[i,0] 복구
        BPL    loop_j                        ;      〃        , loop_j 반복
        SUB    c, c, #4*T                    ; c = &C[0,0] 복구
        ADDS   count, count, #(1<<8)-1       ; (j, i 카운터) 음수된 j 카운터 복구 및 i 다운 카운트
                                             ; ((1<<8)-1 = 0x0f)
        BPL    loop_i                        ; i 카운터 >= 0이면, loop_i 반복
        LDMFD  sp!, {r4, pc}
```

</td>
</tr>
</table>

---

### 6.6.4 Other Counted Loops

하지만, N to 1 또는 N-1 to 0 카운트다운이 아닌 방식이 필요할 수 있다.

> 예를 들어 iteration마다 레지스터 데이터의 1 bit만을 사용한다면, iteration마다 2배를 취하는 2의 거듭제곱 마스크가 필요할 수 있다. 

---

#### 6.6.4.1 Negative Indexing

다음은 특정 스탭 크기를 갖는 negative indexing을 구현한 코드다. 

> `RSB{<cond>}{S} Rd, Rn, N`: `Rd=N-Rn`로 동작한다.

> 다음과 같이 루프 카운터 값을, 루프 내 계산에서 사용할 수 있다. 

```assembly
        RSB     i, N, #0        ; i=-N

loop
        ; loop body goes here and i=-N,-N+STEP,...,
        ADDS    i, i, #STEP     
        BLT     loop            ; N=V이면 loop (i<0이 아니면 overflow 발생)
                                ; 조건에 0을 포함시킬지 여부에 따라 BLT나 BLE 중 선택
```

---

#### 6.6.4.2 Log Indexing

다음은 log indexing을 구현한 코드다.

<table>
<tr>
<td> 2^N to 1 </td> <td> N bit mask to 1 bit mask </td> 
</tr>
<tr>
<td>

```assembly
        MOV     i, #1
        MOV     i, i, LSL N   ; i=1<<N shift

loop
        ; loop body
        MOVS    i, i, LSR#1   ; i=i>>1 shift
        BNE     loop
```

</td>
<td> 

```assembly
        MOV     i, #1
        RSB     i, i, i, LSL N  ; i=(1<<N)-1

loop
        ; loop body
        MOVS    i, i, LSR#1     ; i=i>>1 shift
        BNE     loop
```

</td>
</tr>
</table>

---
