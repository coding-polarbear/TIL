# FBI
## 문제
5명의 요원 중 FBI 요원을 찾는 프로그램을 작성하시오.

FBI요원은 요원의 첩보원명에 FBI가 들어있다. 

## 입력
5개 줄에 요원의 첩보원명이 주어진다. 첩보원명은 알파벳 대문자, 숫자 0~9, 대시 (-)로만 이루어져 있으며, 최대 10글자이다.

## 출력
첫째 줄에 FBI 요원을 출력한다. 이때, 해당하는 요원이 몇 번째 입력인지를 공백으로 구분하여 출력해야 하며, 오름차순으로 출력해야 한다. 만약 FBI 요원이 없다면 "HE GOT AWAY!"를 출력한다.

### 예제 입력 1
```
47-FBI
BOND-007
RF-FBI18
MARICA-13
13A-FBILL
```

### 예제 출력 1
```
1 3 5
```

## 풀이
`contains()` 메소드를 이용해서 FBI가 해당 문자열에 있는지 검토 한 후, result에 index와 빈칸 하나를 더한다.
맨 마지막 빈칸을 제거하기 위해 `trim()`을 이용하고, 비어있는 경우 "HE GOT AWAY"를, 그렇지 않을 경우 `result`를 바로 출력시킨다.

```java
import java.util.Scanner;

public class FBI {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        String value = "";
        String temp;
        for(int i = 0; i < 5; i++) {
            temp = in.nextLine();
            if(temp.contains("FBI"))
                value = value + ((i+1) + " ");

        }
        value = value.trim();
        if(value.equals("")) {
            System.out.println("HE GOT AWAY!");
        } else {
            System.out.println(value);
        }
    }
}
```