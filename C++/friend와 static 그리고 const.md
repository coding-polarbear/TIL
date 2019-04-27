# frinend와 static 그리고 const

## const와 곤련해서 아직 못다한 이야기

### const 함수 오버로딩
> const 객체 또는 참조자를 대상으로 멤버함수 호출시 const 선언된 멤버함수가 호출된다!
```c++
const SoSimple
{
private:
    int num;
public:
    SoSimple(int n) : num(n) { }
    SoSimple & AddNum(int n) 
    {
        num + = n;
        return *this;
    }

    void SimpleFunc()
    {
        cout << "SimpleFunc : " << num << endl;
    }

    void SimpleFunc() const
    {
        cout << "const Simplefunc : " << num1 << endl;
    }
};
```

> 함수의 const 선언 유무는 함수 오버로딩의 조건이 된다!

## 클래스와 함수에 대한 friend 선언
### 클래스의 friend 선언
* friend 선언은 private 멤버의 접근을 허용하는 선언이다.
* friend 선언은 정보은닉에 반하는 선언이기 때문에 매우 제한적으로 선언되어야 한다.
```c++
class Boy
{
private:
    int height;
    friend class Girl; //Girl 클래스에 대한 friend 선언
public:
    Boy(int len) : height(len);
    { }

};
```
```c++
class Girl
{
private:
    char phNum[20];
public:
    Girl(char * num) 
    {
        strcpy(phNum, num);
    }
    void ShowYourFriendInfo(Boy &frn)
    {
        cout << "His height : " << frn.height << endl;
    }
};
```

### 함수의 friend 선언
```c++
class Point
{
private:
    int x;
    int y;
public:
    Point(const int &xpos, const int &ypos) : x(xpos), y(ypos) 
    { }
    friend Point PointOP::PointAdd(const Point&, const Point&);
    friend Point PointOP::PointSub(const Point&, const Point&);
    friend void ShowPointPos(const Point &);
}
```
* 전역 함수 대상으로 friend 선언 가능
* 클래스의 특정 멤버 함수를 대상으로 friend 선언 가능

## C++에서의 static
### C언어 에서의 static
* 전역변수에 선언된 static의 의미
  * 선언된 파일 내에서만 참조를 허용하겠다는 의미
* 함수 내에 선언된 static의 의미
  * 한번만 초기화되고, 지역변수와 달리 함수를 빠져나가도 소멸되지 않는다.

### static 멤버변수 (클래스 변수)
> static 변수는 객체별로 존재하는 변수가 아닌, 프로그램 전체 영역에서 하나만 존재하는 변수로, 프로그램 실행과 동시에 초기화되어 메모리 공간에 할당된다.

```c++
class SoSimple
{
private:
    static int simObjCnt;
public:
    SoSimple()
    {
        simObjCnt++;
        cout << simObjcnt << "번째 SoSimple 객체" << endl;
    }
};

int SoSimple::simObjCnt = 0; // static 멤버변수의 초기화
```

### static 멤버변수의 접근 방법 
> static 변수를 외부에서 접근 가능하게 하려면, 해당 변수가 public으로 선언되어야 한다.

```c++
class SoSimple
{
public:
    static int simObjCnt;
public:
    SoSimple()
    {
        simObjCnt++; //접근 case 1
    }
};
int SoSimple::simObjCnt = 0;
```

```c++
int main(void) 
{
    cout << SoSimpe::simObjCnt << "번째 SoSimple 객체" << endl;
    SoSimple sim1;
    SoSimple sim2;

    cout << SoSimple::simObjCnt << "번째 SoSimple 객체" << endl; //접근 case 2
    cout << sim1.simObjCnt << "번째 SoSimple 객체" << endl; //접근 case 3
    cout << sim2.simObjCnt << "번째 SoSimple 객체" << endl;
    return 0;
}
```

### static 멤버 함수
* 선언된 클래스의 모든 객체가 공유한다.
* public으로 선언이 되면, 클래스의 이름을 이용해서 호출이 가능하다.
* 객체의 멤버로 존재하는 것이 아니다.
* 객체 내에 존재하는 함수가 아니기 때문에 멤버변수나 멤버함수에 접근이 불가능하다.
* static 함수는 static 변수에만 접근 가능하고, static 함수만 호출 가능하다.

```c++
class SoSimple
{
private:
    int num1;
    static int num2;
public:
    SoSimple(int n) : num1(n)
    { }
    static void Adder(int n)
    {
        num1 += n; //컴파일 에러 발생
        num2 += n;
    }
};
int SoSimple::num2 = 0;
```

### const static 멤버와 mutable
> `const static` 멤버변수는 클래스가 정의될 때 지정되는 값이 상수이기 때문에 초기화가 가능하도록 문법으로 정의

```c++
class CountryArea
{
public:
    const static int RUSSIA = 1707540;
    const static int CANADA = 998467;
    const static int CHINA = 957290;
    const static int SOUTH_KOREA = 9922;
};

int main(void)
{
    cout << "러시아 면적 : " << CountryArea::RUSSIA << "km" << endl;
    cout << "캐나다 면적 : " << CountryArea::CANADA << "km" << endl;
    cout << "중국 면적 : " << CountryArea::CHINA << "km" << endl;
    cout << "한국 면적 : " << CountryArea::SOUTH_KOREA << "km" << endl;
    return 0;
}
```

> `mutable`로 선언된 멤버변수는 const 함수 내에서 값의 변경이 가능하다.

```c++
class SoSimple
{
private:
    int num1;
    mutable int num2;
public:
    SoSimple(int n1, int n2)
        : num1(n1), num2(n2);
    { }

    void CopyToNum2() const 
    {
        num2 = num1;
    }
};
```
