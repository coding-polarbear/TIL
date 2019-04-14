# IEEE 754

> 컴퓨터의 부동소수점을 표현하는 가장 많이 쓰이는 표준

2의 보수를 사용해서 양수와 음수를 구부하는 정수와 다르게, 부호비트를 두고 2의 지수를 이용하여 현재 크기를 나타낸다.
이러한 이유로 IEEE 754의 표기 방식을 `부호화 크기(sign and magnitude)` 표현 방식이라고 부르기도 한다.


![IEEE 754 구성도](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e8/IEEE_754_Single_Floating_Point_Format.svg/618px-IEEE_754_Single_Floating_Point_Format.svg.png)

위의 그림은 `단일 정밀도(Single Precision)` 일때의 예시이다. 이때 IEEE 754는 1개의 워드를 사용하며, 1개의 `부호비트`와 8개의 `지수 비트`, 23개의 `fraction bit`로 이루어져 있다.

이런식으로 지수를 이용해서 크기를 나타내면 많은 정보를 볼 수 있지만, 대소 관계를 구분할 때 문제가 생긴다.

대소관계를 비교할 시에 지수가 `음수` 일 경우, 실제로는 `양수`인 수가 더 크지만 `지수가 음수`인 수가 더 크게 보이는 문제점이 생긴다. 

이러한 문제를 해결하기 위하여, `IEEE 754`에서는 `Biased 표기법`을 사용한다.

## Biased 표기법
> 실제 값을 구하기 위해서는 부허없이 표현된 수에서 빼야하는 상수인 `Biase`를 이용하여 표기하는 표기법

### IEEE 754 Biased 값
* 단일정밀도 : 128
* 2배 정밀도 : 1023


## 단일정밀도와 2배 정밀도
* 단일 정밀도 : 32비트 워드를 이용하여 부동소수점을 표현하는 것

![단일정밀도](https://media.geeksforgeeks.org/wp-content/uploads/Single-Precision-IEEE-754-Floating-Point-Standard.jpg)

* 2배 정밀도 : 오버플로우와 언더플로우를 방지하기 위해서 지수 부분이 더 큰 다른 표현 형식을 사용할 수 있도록 만든 것. 1개 워드가 아닌 2개 워드를 사용하여 총 64비트를 사용한다.

![다중정밀도](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a9/IEEE_754_Double_Floating_Point_Format.svg/618px-IEEE_754_Double_Floating_Point_Format.svg.png)