# MIPS 기본 명령어

add a, b, c `# a = b + c`
sub a, b, c `# a = b - c`

### Memory Operands
* 배열과 구조
    * 데이터가 너무 많다.
    * 레지스터에 저장하는 것이 아니라 메모리에 저장한다.

### 데이터 이전 명령어
* lw (load word) : 메모리로 부터 값을 불러옴
* sw (store word) : 메모리에 값을 저장시킴

**예제**
A[10] 에서 A의 base 주소가 $S3라고 해보자
C에서의
```c
int g, h, A[10];
g = h + A[10]
```
을 구현 한다고 해보자. 단, h의 값은 `$S1`에, g의 값은 `$S2`에 저장되어있음.

```mips
lw $t0, 32($s3)
add $s1, $s2, $t0
```
`lw $t0, 32($s3)`는 $s3를 base로 하여 32를 더한 곳에 있는 값을 가져온다.
8을 더하는 것이 아니라 32를 더하는 이유는, lw 명령은 워드 단위로 주소를 처리하기 때문이다.

### 빅 에디안과 리틀에디안
#### Little Endian
* 주소의 가장 오른쪽 (`little end`) 바이트를 워드 주소로 사용한다.
* Intel IA-32, DEC PDP 11, VAX-11, Alpha, Atmel AVR

#### Big Endian
* 주소의 가장 왼쪽 (`big end`) 바이트를 워드 주소로 사용한다.
* MIPS, IBM S/360 and S/370, Motorola 680x0, SPARC, PA-RISC

