# 예외처리 (Exception Handling)
## 예외상황과 예외처리의 이해

### 예외상황을 처리하지 않았을 때의 결과
* 예외 상황은 프로그램 실행 중 발생하는 문제의 상황을 의미한다.
* 예외상황의 예
  * 나이를 입력하라고 했는데, 0보다 작은 값이 입력 됨
  * 나눗셈을 위해서 두 개의 숫자를 입력받았는데, 제수로 0이 입력됨
  * 주민등록번호 13자리만 입력하라고 했는데 중간에 -가 삽입됨
* 이렇듯 예외는 문법적 오류가 아닌, 프로그램 논리에 맞지 않는 오류를 뜻한다.

### if문을 이용한 예외의 처리
* if문을 이용해서 예외를 발견하고 처리하면, 예외처리 부분과 일반적인 프로그램의 흐름을 쉽게 구분할 수 없다.
* if문은 일반적인 프로그램의 논리를 구현하는데 주로 사용된다.
* 그래서 C++은 별도의 예외처리 메커니즘을 제공한다.

## C++의 예외 처리 메커니즘

### C++의 예외 처리 메커니즙의 이해 : try, catch, throw

```c++
try {
    if (예외가 발생한다면)
        throw expn;
}
catch (type expn)
{
    //예외 처리
}
```

* try 블록은 예외발생에 대한 검사범위를 지정하는데 사용된다.
* catch 블록은 try 블록에서 발생한 예외를 처리하는 영역으로 그 형태가 마치 반환형 없는 함수와 같다.
* throw는 예외의 발생을 알리는 역할을 한다.
* try ~ catch는 하나의 문장이므로 그 사이에 다른 문장이 삽입 될 수 없다.

### 예외처리 메커니즘의 적용
```c++
int main(void)
{
    int num1, num2;
    cout << "두 개의 숫자 입력 : ";
    cin >> num1 >> num2;

    try {
        if(num2 == 0) 
            throw num2;
        cout << "나눗셈의 몫 : " << num1 / num2 << endl;
        cout << "나눗셈의 나머지 : " << num1 % num2 << endl;
    }
    catch (int expn) {
        cout << "제수는 " << expn << "이 될 수 없습니다." << endl;
        cout << "프로그램을 다시 실행하세요." << endl;
    }

    cout << "end of main" << endl;
    return 0;
}
```
> 예외의 발생으로 인해서 try 블록 내에서 throw절이 실행되면, try 부분의 나머지 부분은 실행되지 않는다.

### try 블록을 묶는 기준
> 예외와 연관이 있는 부분을 모두 하나의 try 블록으로 묶어야 한다.

## Stack Unwinding (스택 풀기)
### 예외의 발생위치와 처리 위치가 달라야 하는 경우
```c++
int StoI(char * str) 
{
    int len = strlen(str);
    int num = 0;
    
    for(int i = 0; i < len; i++) 
    {
        if(str[i] < '0' || str[i] > '9')
            throw str[i];
        num += (int) (pow((double 10, (len - 1) - i) * str[i] + (7 - '7)));
    }
    return num;
}

int main(void) 
{
    char str1[100];
    char str2[100];

    while(1)
    {
        cout << "두 개의 숫자 입력 : ";
        cin >> str1 >> str2;
        try {
            cout << str1 << " + " << str2 << " = " << StoI(str1) + StoI(str2) << endl;
            break;
        }
        catch(char ch)
        {
            cout << "문자 " << ch <<"가 입력되었습니다." << endl;
            cout << "재입력 진행합니다. " << endl << endl;
        }
    }

    cout << "프로그램을 종료합니다. " << endl;
    return 0;
}
```

### 스택 풀기 (Stack Unwinding)
![](http://purplebeen.kr/images/%EA%B7%B8%EB%A6%BC1.png)

### 자료형이 일치하지 않아도 예외 데이터는 전달
```c++
int SimpleFunc(void)
{
    try {
        if()
            throw -1; //int형 예외 데이터 발생
    }
    catch(char expn) {
        //char형 예외 데이터를 전달하라!
    }
}
```
> 형 변환을 발생하지 않아서 예외데이터는 SimpleFunc 함수를 호출한 영역으로 전달된다.

### 하나의 try 블록과 다수의 catch 블록
```c++
try
{
    cout << str1 << " + " << str2 << " = " << StoI(str1) + StoI(str2) << endl;
    break;
} catch (char ch)
{
    cout << "문자 " << ch << "가 입력되었습니다." << endl;
    cout << "재입력 진행합니다." << endl << endl;
}
catch (int expn)
{
    if(expn == 0)
        cout << "0으로 입력하는 숫자는 입력 불가." << endl;
    else
        cout << "비정상적 입력이 이루어졌습니다." << endl;
    cout << "재입력 진행합니다. " << endl << endl;
}
```

> 하나의 try 영역 내에서 종류가 다른 둘 이상의 예외가 발생할 수 있기 때문에, 하나의 try블록에 다수의 catch 블록을 추가할 수 있다.

### 전달되는 예외의 명시
```c++
int ThrowFunc(int num) throw (in, char)
{

}
```

> 함수 내에서 예외상황의 발생으로 인해서 int형 예외 데이터와 char형 예외 데이터가 전달될 수 있음을 명시한 선언

## 예외상황을 표현하는 예외 클래스의 설계
### 예외 클래스와 예외 객체
###### 예외 클래스
> 예외발생을 알리는데 사용되는 객체
###### 예외 클래스
> 예외객체의 생성을 위해 정의된 클래스

객체를 이용해서 예외상황을 알리면 예외가 발생한 원인에 대한 정보를 보다 자세히 담을 수 있다.

```c++
class DepositException
{
    private:
        int reqDep; //요청 입금액
    public:
    DepositException(int money) : reqDep(money){}
    void ShowExceptionReason() {
        cout << "[예외메시지 : " << reqDep << "는 입금 불가]" << endl;
    }
};
```

```c++
class WithDrawException
{
private:
    int balance; //잔고
public:
    WithdrawException(int money) : balance(money) {}
    void showExceptionReason() {
        cout << "[예외메시지 : " << balance << ", 잔액부족]" << endl;
    }
}
```

```c++
class Account
{
private:
    char accNum[50];
    int balance;
public:
    Accout(char * acc, int money) : balance(money) {
        strcpy(accNum, acc);
    }
    void Deposit(int money) throw (DepositException) {
        if(money < 0) {
            DepositException expn(money);
            throw expn;
        }
        balance += money;
    }

    void Withdraw(int money) throw (WithdrawException)
    {
        if (money > balance)
            throw WithDrawException(balance);
        balance -= money;
    }

    void ShowMyMoney()
    {
        cout << "잔고 : " << balance << endl << endl;
    }
}

int main(void)
{
    Account myAcc("56789-827120", 5000);

    try {
        myAcc.Deposit(2000);
        myAcc.Deposit(-300);
    }
    catch(DepositException &expn)
    {
        expn.ShowExceptionReason();
    }
    myAcc.ShowMyMoney();

    try
    {
        myAcc.Withdraw(3500);
        myAcc.Withdraw(4500);
    }
    catch(WithdrawException &expn)
    {
        expn.ShowExceptionReason();
    }
    myAcc.ShowMyMoney();
    return 0;
}
```

### 상속관계에 있는 예외 클래스
```c++
class AccoutException
{
public:
    virtual void ShowExceptionReason() = 0;
};
```

```c++
class DepositException : public AccountException
{
private:
    int reqDep;
public:
    DepositException(int money) : reqDep(money);
    {}
    void ShowExceptionReason()
    {
        cout << "[예외 메시지 : " << reqDep << "는 입금 불가]" << endl;
    }
};
```

```c++
class WithdrawException : public AccountException
{
private:
    int balance;
public:
    WithdrawException(int money) : balance(money);
    {}

    void ShowExceptionReason()
    {
        cout << "[예외메시지 : " << balance << ", 잔액부족]" << endl;
    }
};
```

```c++
try
{
    myAcc.Deposit(2000);
    myAcc.Deposit(-300);
}
catch(AccountException &expn)
{
    expn.ShowExceptionReason();
}

try 
{
    myAcc.Withdraw(3500);
    myAcc.Withdraw(4500);
}
catch(AccountException &expn)
{
    expn.ShowExceptionReason();
}
```

> `DepositException` 예외 클래스와  `WithdrawException` 예외클래스는 `AccountException` 클래스를 상속하므로 `AccountException`을 대상으로 정의된 catch 블록에 의해 처리될 수 있다.

### 예외 전달방식에 따른 주의사항
* 맨위에 있는 catch 블록부터 시작해서 아래로 적절한 catch 블록을 찾아 내려온다.

## 예외처리에 관련된 또 다른 특성들

### new 연산자에 의해 전달되는 예외

```c++
#include <iostream>
int main(void)
{
    int num = 0;
    try {
        while(1)
        {
            num++;
            cout << num << "번째 할당 시도" << endl;
            new int[10000][10000];
        }
    }
    catch (bad_alloc &bad) 
    {
        cout << bad.what() << endl;
        cout << "더 이상 할달 불가!" << endl;
    }
    return 0;
}
```
###### 실행결과
```
1번째 할당 시도
2번째 할당 시도
3번째 할당 시도
4번째 할당 시도
5번째 할당 시도
bad allocation
더이상 할당 불가!
```

> bad_alloc과 같이 프로그래머가 정의하지 않아도 발생하는 예외도 있다.


### 모든 예외를 처리하는 catch 블록
```c++
try
{
    . . .
}
catch(...) // ...은 전달되는 모든 예외를 다 받아주겠다는 선언 
{
    . . . . 
}
```

* 마지막 catch 블록에 덧붙여 지는 경우가 많음
* 대신 발생한 예외와 관련되서 그 어떠한 전달받을 수 없으며, 전달한 예외의 종류도 구분이 불가능하다.

### 예외 던지기
* catch 블록에 전달된 예외는 다시 더져질 수 있다.
* 그리고 이로 인해서 하나의 예외가 둘 이상의 catch 블록에의해서 처리되게할 수 있다.

```c++
void Divide(int num1, int num2)
{
    try
    {
        if(num2 == 0)
            throw 0;
        cout << "몫 : " << num1 / num2 >> endl;
        cout << "나머지 : " << num1 % num2  << endl;
    }
    catch (int expn) 
    {
        cout << "first catch" << endl;
        throw; //예외를 다시 던진다.
    }
}

int main(void)
{
    try 
    {
        Divide(9, 2);
        Divide(4.0);
    }
    catch(int expn)
    {
        cout << "second catch" << endl;
    }
    return 0;
}
```