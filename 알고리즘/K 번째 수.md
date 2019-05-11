# K 번째 수

## 문제
수 N개 A1, A2, ..., AN이 주어진다. A를 오름차순 정렬했을 때, 앞에서부터 K번째 있는 수를 구하는 프로그램을 작성하시오.

## 입력
첫째 줄에 N(1 ≤ N ≤ 5,000,000)과 K (1 ≤ K ≤ N)이 주어진다.

둘째에는 A1, A2, ..., AN이 주어진다. (-109 ≤ Ai ≤ 109)

## 출력
A를 정렬했을 때, 앞에서부터 K번째 있는 수를 출력한다.

## 예제 입력 1
```
5 2
4 1 2 3 5
```

## 예제 출력 1
```
2
```

## 풀이

### 첫 번째 시도
소프트웨어 마에스트로 10기 코딩테스트에 나왔던 문제이고, 실제로 풀었던 문제이기 때문에 쉽게 풀려고 했었다.
그러나,  백준에는 시간 제한이 있었다.
이전에 풀때에는 java의 `Collections.sort()`나 `Arrays.sort()` 메소드를 사용했었는데, `2초`라는 시간제한에는 충분하지 않았던 것 같다.

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int k = in.nextInt();
        int n = in.nextInt();
        in.nextLine();
        int[] arr = new int[k];

        for(int i = 0; i < k; i++) {
            arr[i] = in.nextInt();
        }

        Arrays.parallelSort(arr);
        System.out.println(arr[n - 1]);
    }
}
```

### 두 번째 시도

좀 더 빠른 방법이 없을까 생각하다 stl에서 퀵소트를 기본적으로 지원한다는 것이 생각났다. C++로 언어를 바꾸어서 풀었다.

또한 `std::cin`과 `std::cout`의 속도를 좀 더 빠르게 해주기 위해서 일부 최적화 코드를 넣어주었다.

stl의 qsort()함수는 `compare(const void * , const void *)` 형태의 함수포인터를 요구한다. 오름차순으로 정렬하기 위해, `compare`함수를 아래와 같이 구현하였다.
```c++
#include <iostream>
#include <algorithm>

int compare(const void* first, const void* last) {
	if (*(int*)first > *(int*)last)
		return 1;
	else if (*(int*)first < * (int*)last)
		return -1;
	else
		return 0;
}
int main()
{
	int n, k;
	std::ios::sync_with_stdio(false); //cin, cout 입력 최적화
	std::cin >> k;
	std::cin >> n;

	int* arr = new int[k];
	for (int i = 0; i < k; i++)
		std::cin >> arr[i];

	qsort(arr, k, sizeof(int), compare);
	std::cout << arr[n - 1] << std::endl;
}
```

> 그 결과 2초라는 시간 제한을 맞출 수 있었다.