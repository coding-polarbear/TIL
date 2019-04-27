# C언어 기반의 C++

### C에서 const의 의미
```c
const int num = 10;
```
> 변수 num을 상수화

```c
const int * ptr = & val;
```
> 포인터 ptr을 이용해서 val1의 값을 변경할 수 없음

```c
int * const ptr2 = &val
```
> 포인터 ptr2가 상수화 됨

```c
const int * const ptr3 = &val3;
```
> 포인터 ptr3가 상수화되었으며, ptr3를 이용해서 val3의 값을 변경할 수 없음


### 실행중인 프로그램의 메모리 공간
* 데이터 : 전역변수가 저장되는 영역
* 스택 : 지역변수 및 매개변수가 저장되는 영역
* 힙 : malloc 함수호출에 의해 프로그램이 실행되는 과정에서 동적으로 할당이 이뤄지는 영역
* malloc & free : malloc 함수호출에 의해 할당된 메모리 공간은 free 함수호출을 통해서 소멸하지 않으면 해제되지 않는다.

### 새로운 자료형 bool
* true와 false는 bool 형 데이터이다.
* true와 false 정보를 저장할 수 있는 변수는 bool형 변수이다.

```c++

bool IsPositive(int num)
{
    if(num < 0)
        return false;
    else
        return true;
}
int main(void)
{
    bool isPos;
    int num;
    cout << "Input number : ";
    cin >> num;
    isPos = IsPositive(num);
    if(isPos)
        cout << "Positive number" << endl;
    else
        cout << "Negative number" << endl;
    return 0;
}
```

### 참조자의 이해
> 기존에 선언된 변수에 붙이는 `별칭`

참조자가 만들어지면 변수의 이름과 사실상 차이가 없다.

## 예제
```c++
int main(void)
{
    int num1 = 1020;
    int &num2 = num;

    num2 = 3047;

    cout << "VAL : " << num1 << endl;
    cout << "REF : " << num2 << endl;

    cout << "VAL : " << &num1 << endl;
    cout << "REF : " << &num2 << endl;

    return 0;
}
```

**출력 결과**
```
VAL : 3047
REF : 3047
VAL : 0012FF60
REF : 0012FF60
```

### 참조자의 선언 가능 범위
* 상수 대상으로의 참조자 선언은 불가능하다.
* 참조자는 생성과 동시에 누군가를 참조해야 한다.
* 포인터처럼 NULL로 초기화하는 것도 불가능하다.
* 기본적으로 참조의 대상읜 변수여야 한다.
* 참조자는 참조의 대상을 변경할 수 없다.
* 배열 역시 변수의 성향을 지니기 때문에 참조자의 선언이 가능하다.

### 참조자 기반의 Call-by-Reference
> 매개변수는 함수가 호출될 때 선언이 되는 변수이므로, 함수 호출의 과정에서 선언과 동시에 초기화된다.

```c++
void SwapByRef(int &ref1. int &ref2)
{
    int temp = ref1;
    ref1 = ref2;
    ref2 = temp;
}

int main(void)
{
    int val1 = 10;
    int val2 = 20;
    SwapByRef2(val1, val2);
    cout << "val1 : " << val1 << endl;
    cout << "val2 : " << val2 << endl;
    return 0;
}
```

### const 참조자
```c++
void HappyFunc(const int &ref)  { }
```
> 함수 HappyFunc 내에서 참조자 ref를 이용한 값의 변경은 허용하지 않겠다! 라는 의미

#### const 참조자의 장점
* 함수의 원형 선언만 봐도 값의 변경이 일어나지 않음을 판단할 수 있다.
* 실수로 인한 값의 변경이 일어나지 않는다.

### 반환형이 참조이고 반환도 참조로 받는 경우
* 함수의 반환형이 참조형이면 반환되는 대상을 참조자로, 또 변수로 받을 수 있다.
* 반환형이 값의 형태라면, 참조자로 그 값을 받을 수 없다.
* 참조하는 대상이 `소멸` 되지 않도록 유의하자

### const 참조자의 또 다른 특징
> const 선언이 들어간 변수는 const 참조자를 이용해야 한다.

### 참조자의 상수 참조
**const 참조자**를 이용하면 상수를 참조할 수 있다.
> 상수를 메모리공간에 임시적으로 저장하기 때문에, 행을 바꿔도 소멸하지 않는다.

### malloc 과 free를 대신하는 new와 delete
#### new
> malloc를 대신하는 메모리 동적 할당 방법. 크기를 바이트 단위로 계산하는 일을 거치지 않아도 된다.

###### int형 변수의 할당

```c++
int * ptr = new int;
```
###### double형 변수의 할당
```c++
double * ptr2 = new double;
```
###### 길이가 3인 int형 배열의 할당
```c++
int arr1 = new int[3];
```
###### 길이가 7인 double형 배열의 할당
```c++
double * arr2 = new double[7];
```

#### delete
> free를 대신하는 메모리의 해제 방법!

###### int 변수의 소멸
```c++
delete ptr1
```
###### double형 배열의 소멸
```c++
delete ptr2;
```
###### int형 배열의 소멸
```c++
delete []arr1;
```
##### double형 배열의 소멸
```c++
delete []arr2;
```

### C++의 표준 헤더 : c를 더하고 .h를 빼라
* `#include <Stdio.h>`  : `#include <cstdio>`
* `#include <stdlib.h>` : `#include <cstdlib>`
* `#include <math.h>` : `#include <cmath>`
* `#include <string.h>` : `#include <cstring>`

> 표준 C에 대응하는 표준 C++ 함수는 C++ 문법을 기반으로 변경 및 확장되었다. 따라서 가급적이면 C++의 헤더파일을 포함하여, C++의 표준함수를 호출해야한다.