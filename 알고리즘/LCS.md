# LCS
## 문제
LCS(Longest Common Subsequence, 최장 공통 부분 수열)문제는 두 수열이 주어졌을 때, 모두의 부분 수열이 되는 수열 중 가장 긴 것을 찾는 문제이다.

예를 들어, ACAYKP와 CAPCAK의 LCS는 ACAK가 된다.

## 입력
첫째 줄과 둘째 줄에 두 문자열이 주어진다. 문자열은 알파벳 대문자로만 이루어져 있으며, 최대 1000글자로 이루어져 있다.

## 출력
첫째 줄에 입력으로 주어진 두 문자열의 LCS의 길이를 출력한다.

## 풀이
```java
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        char[] s1 = sc.nextLine().toCharArray();
        char[] s2 = sc.nextLine().toCharArray();
        
        int[][] dp = new int[1001][1001];
        
        for(int i = 1; i <= s1.length; i++) {
            for(int j = 1; j <= s2.length; j++) {
                if(s1[i-1] == s2[j-1]) {
                    dp[i][j] = dp[i-1][j-1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        
        System.out.println(dp[s1.length][s2.length]);
    }
}
```

* 두개의 스트링을 s1, s2라고 할때,
* `s1[i-1]`과 `s2[j-1]`이 같으면 dp[i][j] = dp[i-1][j-1] + 1이다.
* 그렇지 않을 경우, dp[i][j]는 s1의 이전값과 현재값의 LCS와, s1과  s2 이전값의 LCS중 최대값이다.