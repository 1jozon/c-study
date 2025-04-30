# C++ 引入了四种功能不同的强制类型转换运算符以进行强制类型转换：static_cast、reinterpret_cast、const_cast 和 dynamic_cast。

强制类型转换是有一定风险的，有的转换并不一定安全，如把整型数值转换成[指针](https://c.biancheng.net/c/80/)，把基类指针转换成派生类指针，把一种函数指针转换成另一种函数指针，把[常量指针](https://c.biancheng.net/view/1ltv8ex.html)转换成非常量指针等。C++ 引入新的强制类型转换机制，主要是为了克服C语言强制类型转换的以下三个缺点。

1) 没有从形式上体现转换功能和风险的不同。


例如，将 int 强制转换成 double 是没有风险的，而将常量指针转换成非常量指针，将基类指针转换成派生类指针都是高风险的，而且后两者带来的风险不同（即可能引发不同种类的错误），C语言的强制类型转换形式对这些不同并不加以区分。

2) 将多态基类指针转换成派生类指针时不检查安全性，即无法判断转换后的指针是否确实指向一个派生类对象。



3) 难以在程序中寻找到底什么地方进行了强制类型转换。


强制类型转换是引发程序运行时错误的一个原因，因此在程序出错时，可能就会想到是不是有哪些强制类型转换出了问题。

如果采用C语言的老式做法，要在程序中找出所有进行了强制类型转换的地方，显然是很麻烦的，因为这些转换没有统一的格式。

而用 C++ 的方式，则只需要查找`_cast`字符串就可以了。甚至可以根据错误的类型，有针对性地专门查找某一种强制类型转换。例如，怀疑一个错误可能是由于使用了 reinterpret_cast 导致的，就可以只查找`reinterpret_cast`字符串。

C++ 强制类型转换运算符的用法如下：

强制类型转换运算符 <要转换到的类型> (待转换的表达式)

例如：

double d = static_cast <double> (3*5); //将 3*5 的值转换成实数

下面分别介绍四种强制类型转换运算符。

## static_cast

static_cast 用于进行比较“自然”和低风险的转换，如整型和浮点型、字符型之间的互相转换。另外，如果对象所属的类重载了强制类型转换运算符 T（如 T 是 int、int* 或其他类型名），则 static_cast 也能用来进行对象到 T 类型的转换。

static_cast 不能用于在不同类型的指针之间互相转换，也不能用于整型和指针之间的互相转换，当然也不能用于不同类型的引用之间的转换。因为这些属于风险比较高的转换。

static_cast 用法示例如下：

```
#include <iostream>
using namespace std;
class A
{
public:
    operator int() { return 1; }
    operator char*() { return NULL; }
};
int main()
{
    A a;
    int n;
    char* p = "New Dragon Inn";
    n = static_cast <int> (3.14);  // n 的值变为 3
    n = static_cast <int> (a);  //调用 a.operator int，n 的值变为 1
    p = static_cast <char*> (a);  //调用 a.operator char*，p 的值变为 NULL
    n = static_cast <int> (p);  //编译错误，static_cast不能将指针转换成整型
    p = static_cast <char*> (n);  //编译错误，static_cast 不能将整型转换成指针
    return 0;
}
```

## reinterpret_cast

reinterpret_cast 用于进行各种不同类型的指针之间、不同类型的引用之间以及指针和能容纳指针的整数类型之间的转换。转换时，执行的是逐个比特复制的操作。

这种转换提供了很强的灵活性，但转换的安全性只能由程序员的细心来保证了。例如，程序员执意要把一个 int* 指针、函数指针或其他类型的指针转换成 string* 类型的指针也是可以的，至于以后用转换后的指针调用 string 类的成员函数引发错误，程序员也只能自行承担查找错误的烦琐工作：（C++ 标准不允许将函数指针转换成对象指针，但有些编译器，如 Visual Studio 2010，则支持这种转换）。

reinterpret_cast 用法示例如下：

```
#include <iostream>
using namespace std;
class A
{
public:
    int i;
    int j;
    A(int n):i(n),j(n) { }
};
int main()
{
    A a(100);
    int &r = reinterpret_cast<int&>(a); //强行让 r 引用 a
    r = 200;  //把 a.i 变成了 200
    cout << a.i << "," << a.j << endl;  // 输出 200,100
    int n = 300;
    A *pa = reinterpret_cast<A*> ( & n); //强行让 pa 指向 n
    pa->i = 400;  // n 变成 400
    pa->j = 500;  //此条语句不安全，很可能导致程序崩溃
    cout << n << endl;  // 输出 400
    long long la = 0x12345678abcdLL;
    pa = reinterpret_cast<A*>(la); //la太长，只取低32位0x5678abcd拷贝给pa
    unsigned int u = reinterpret_cast<unsigned int>(pa);//pa逐个比特拷贝到u
    cout << hex << u << endl;  //输出 5678abcd
    typedef void (* PF1) (int);
    typedef int (* PF2) (int,char *);
    PF1 pf1;  PF2 pf2;
    pf2 = reinterpret_cast<PF2>(pf1); //两个不同类型的函数指针之间可以互相转换
}
```

程序的输出结果是：
200, 100
400
5678abed

第 19 行的代码不安全，因为在编译器看来，pa->j 的存放位置就是 n 后面的 4 个字节。 本条语句会向这 4 个字节中写入 500。但这 4 个字节不知道是用来存放什么的，贸然向其中写入可能会导致程序错误甚至崩溃。

上面程序中的各种转换都没有实际意义，只是为了演示 reinteipret_cast 的用法而已。在编写黑客程序、病毒或反病毒程序时，也许会用到这样怪异的转换。

reinterpret_cast体现了 C++ 语言的设计思想：用户可以做任何操作，但要为自己的行为负责。

## const_cast

const_cast 运算符仅用于进行去除 const 属性的转换，它也是四个强制类型转换运算符中唯一能够去除 const 属性的运算符。

将 const 引用转换为同类型的非 const 引用，将 const 指针转换为同类型的非 const 指针时可以使用 const_cast 运算符。例如：

```
const string s = "Inception";string& p = const_cast <string&> (s);string* ps = const_cast <string*> (&s);  // &s 的类型是 const string*
```

## dynamic_cast

用 reinterpret_cast 可以将多态基类（包含虚函数的基类）的指针强制转换为派生类的指针，但是这种转换不检查安全性，即不检查转换后的指针是否确实指向一个派生类对象。dynamic_cast专门用于将多态基类的指针或引用强制转换为派生类的指针或引用，而且能够检查转换的安全性。对于不安全的指针转换，转换结果返回 NULL 指针。

dynamic_cast 是通过“运行时类型检查”来保证安全性的。dynamic_cast 不能用于将非多态基类的指针或引用强制转换为派生类的指针或引用——这种转换没法保证安全性，只好用 reinterpret_cast 来完成。

dynamic_cast 示例程序如下：

```
#include <iostream>
#include <string>
using namespace std;
class Base
{  //有虚函数，因此是多态基类
public:
    virtual ~Base() {}
};
class Derived : public Base { };
int main()
{
    Base b;
    Derived d;
    Derived* pd;
    pd = reinterpret_cast <Derived*> (&b);
    if (pd == NULL)
        //此处pd不会为 NULL。reinterpret_cast不检查安全性，总是进行转换
        cout << "unsafe reinterpret_cast" << endl; //不会执行
    pd = dynamic_cast <Derived*> (&b);
    if (pd == NULL)  //结果会是NULL，因为 &b 不指向派生类对象，此转换不安全
        cout << "unsafe dynamic_cast1" << endl;  //会执行
    pd = dynamic_cast <Derived*> (&d);  //安全的转换
    if (pd == NULL)  //此处 pd 不会为 NULL
        cout << "unsafe dynamic_cast2" << endl;  //不会执行
    return 0;
}
```

程序的输出结果是：
unsafe dynamic_cast1

第 20 行，通过判断 pd 的值是否为 NULL，就能知道第 19 行进行的转换是否是安全的。第 23 行同理。

如果上面的程序中出现了下面的语句：

Derived & r = dynamic_cast <Derived &> (b);

那该如何判断该转换是否安全呢？不存在空引用，因此不能通过返回值来判断转换是否安全。C++ 的解决办法是：dynamic_cast 在进行引用的强制转换时，如果发现转换不安全，就会拋出一个异常，通过处理异常，就能发现不安全的转换。



确实，C++ 引入了四种不同的类型转换运算符，每种都有其特定的用途和适用场景。这些运算符提供了比C风格的类型转换（如 `(type)expression`）更安全、更明确的方式来进行类型转换。以下是这四种类型转换运算符的详细说明：

### 1. `static_cast`

- **用途**：用于基本数据类型的转换，例如将 `int` 转换为 `double`，或者进行具有继承关系的类之间的指针或引用转换。
- **特点**：
  - 不执行运行时类型检查，因此它比 `dynamic_cast` 更快。
  - 可以用于在相关类型之间进行转换，比如基类和派生类之间的转换（但不检查实际对象类型）。
  - 不能移除 `const` 或 `volatile` 属性。

```cpp
double d = 3.14;
int i = static_cast<int>(d); // 将 double 转换为 int

class Base {};
class Derived : public Base {};
Derived* dp = new Derived();
Base* bp = static_cast<Base*>(dp); // 安全向下转换
```

### 2. `reinterpret_cast`

- **用途**：进行低级别的重解释操作，将一种类型的数据当作另一种类型来处理。这种转换通常用于底层编程。
- **特点**：
  - 允许将任何指针类型转换为另一个指针类型，或将指针转换为整数类型（反之亦然），但这种转换依赖于具体的实现细节，可能不可移植。
  - 没有运行时检查，使用时需要非常小心。

```cpp
int data = 0x12345678;
void* p = reinterpret_cast<void*>(&data); // 将 int* 转换为 void*
int* ip = reinterpret_cast<int*>(p); // 再转回 int*
```

### 3. `const_cast`

- **用途**：唯一可以用来添加或移除 `const` 或 `volatile` 属性的类型转换。
- **特点**：
  - 如果尝试通过 `const_cast` 移除了 `const` 属性并修改了一个实际上声明为 `const` 的对象，结果是未定义行为。
  
```cpp
const int a = 10;
int* b = const_cast<int*>(&a); // 移除 const 属性
// 修改 *b 可能导致未定义行为
```

### 4. `dynamic_cast`

- **用途**：主要用于涉及继承层次结构的安全向下转换（从基类到派生类）。它可以在运行时验证转换是否合法。
- **特点**：
  - 只能在包含虚函数的类层次中工作，因为它依赖于RTTI（Run-Time Type Information）。
  - 如果转换不可能完成（即目标对象不是请求的目标类型），则返回 `nullptr`（对于指针转换）或抛出 `std::bad_cast` 异常（对于引用转换）。

```cpp
class Base { virtual ~Base() {} };
class Derived : public Base {};

Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr); // 如果 basePtr 实际上指向一个 Derived 对象，则成功
if (derivedPtr) {
    // 转换成功
} else {
    // 转换失败
}
```

总之，正确选择合适的类型转换运算符可以帮助提高代码的安全性和可读性。每种转换都有其特定的应用场景，理解它们的区别和最佳实践是编写健壮的C++代码的关键。