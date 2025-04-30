#### 文章目录

- [参考](#_4)
- [描述](#_13)
- [隐式形参 this 指针](#_this__22)
- - - [冲突](#_76)
    - [this 指针](#this__113)
- [常量形参](#_151)
- - - [按值传递与按引用传递](#_153)
    - [常量形参](#_171)
    - [承诺（常量形参的传递问题）](#_209)
- [常量成员函数](#_291)



## 参考

| 项目               |                                                              |
| ------------------ | ------------------------------------------------------------ |
| 精通C++ （第九版） | 托尼·加迪斯、朱迪·沃尔特斯、戈德弗雷·穆甘达 **（著）** **/** 黄刚 等 **（译）** |
| 搜索引擎           | **[Bing](https://cn.bing.com/?mkt=zh-CN)**                   |

## 描述

| 项目     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 编译器   | gcc version 8.1.0 (x86_64-win32-seh-rev0, Built by MinGW-W64 project) |
| 操作系统 | Windows 10 专业版（64 位）                                   |

## 隐式形参 this 指针

> 默认情况下，编译器为类的每个成员函数提供了一个隐式形参，该形参指向被调用的成员函数所在的对象。该隐式形参称为 **this** 。
>
> ------
>
> **上述内容引用自 精通 C++ （第九版）**

隐式形参 **this** 为一个指针，指向函数所在的对象。你可以在成员函数中直接使用隐式形参 **this** 。实际上，当你在成员函数中直接访问类中的成员时，就已经隐式的使用了 **this** 指针。

```
举个栗子
#include <iostream>
#include <string>
using namespace std;


class Notebook 
{
    public: 
    string content = "Hello World";
    void echo(){
        // 输出所属类的地址（以十进制展示）
        cout << (long long)this << endl;
        // 输出所属类的 content 成员属性
        cout << this -> content << endl;
    };
};

int main(){
    // 实例化
    Notebook notebook;

    notebook.echo();

    system("pause");
}
12345678910111213141516171819202122232425AI写代码
执行效果
6422000
Hello World
请按任意键继续. . .
123AI写代码
注：
```

**this** 指针仅能指向由所属类实例化的对象，将实参赋予形参的操作将在创建类的实例对象的过程中自动完成。

#### 冲突

当成员函数的形参与成员属性同名时，在成员函数的内部将发生冲突，成员属性将被隐藏。对此，请参考如下示例：

```cpp
#include <iostream>
#include <string>
using namespace std;


class Notebook
{
    public:
    string content = "TwoMoons";
    void read(string content){
        // 输出 content 
        cout << content << endl;
    };
};

int main(){
    Notebook notebook;
    notebook.read("RedHEart");

    system("pause");
}
123456789101112131415161718192021AI写代码
执行结果
RedHEart
请按任意键继续. . .
12AI写代码
```

在成员函数中向控制台输出 **content** 中保存的数据，结果是将 **形参 content** 中的内容进行输出，而 **成员属性 content** 被忽略。

#### this 指针

通过 **this 指针**，我们可以在发生冲突的成员函数内部冲破同名形参的封锁，访问同名的成员属性。对此，请参考如下示例：

```cpp
#include <iostream>
#include <string>
using namespace std;


class Notebook
{
    public:
    string content = "TwoMoons";
    void read(string content){
        // 输出 content 
        cout << content << endl;
        // 输出成员属性 content 
        cout << this -> content << endl;
    };
};

int main(){
    Notebook notebook;
    notebook.read("RedHEart");

    system("pause");
}
1234567891011121314151617181920212223AI写代码
执行结果
RedHEart
TwoMoons
请按任意键继续. . .
123AI写代码
```

## 常量形参

#### 按值传递与按[引用传递](https://so.csdn.net/so/search?q=引用传递&spm=1001.2101.3001.7020)

向函数传递参数时，我们有两种选择，即按值传递或按引用传递。

```
按值传递与按引用传递
```

1. 按值传递将实参复制后的副本传递给相关的形参。
2. 按引用传递将实参所处的地址传递给形参。

```
比较
```

按值传递在实参较复杂（数据量大，执行复制操作将消耗一定的性能）时将对程序的性能造成较大的影响。按引用传递传递的是地址，通过形参中存储的地址我们可以直接访问实参所指向的内存空间。由于形参与实参指向同一内存空间，因此加大了无意修改内存空间中的数据的可能。

```
解决
```

为了避免形参修改实参指向的内存空间中的数据，我们可以将形参设定为常量。尝试修改常量形参，C++ 将抛出错误。

#### 常量形参

```
举个栗子
#include <iostream>
#include <string>
using namespace std;


void fraud(const int &num1, int &num2){
    // num1 由于被设定为常量，因此不能被
    // 函数所修改。故执行
    // num1 += 100;
    // 将导致 C++ 抛出错误。

    num2 += 1;
    cout << (num1 + num2) << endl;
};


int main(){
    int num1 = 1;
    int num2 = 1;
    fraud(num1, num2);
    
    system("pause");
}
1234567891011121314151617181920212223AI写代码
执行效果
3
请按任意键继续. . .
12AI写代码
```

#### 承诺（常量形参的传递问题）

> 承诺不修改 **X** 的函数不能将 **X** 传递给另一个函数，除非第二个函数也承诺不修改 **X**。
>
> ------
>
> **上述内容引用自 精通 C++ （第九版）**

对此，请参考如下示例：

```cpp
#include <string>
#include <iostream>
using namespace std;


void underling(int &num){
    // underling 函数虽然没有承诺修改形参，
    // 但该函数并没有因此而修改形参。但即便如此，
    // C++ 也将抛出错误。
}

// fraud 函数通过关键字 const 承诺
// 不修改形参 num
void fraud(const int &num){
    // fraud 将承诺不修改的形参传递给
    // 没有承诺不修改形参 num 的函数。
    // 这将引发错误。即使 underling 没有
    // 修改形参 num 。
    underling(num);
}

int main(){
    int num = 666;
    fraud(num);

    cout << num << endl;
    
    system("pause");
}
1234567891011121314151617181920212223242526272829AI写代码
```

为了使得函数 **underling** 能够正常接收实参，我们必须承诺不修改形参。对此，请参考如下示例：

```cpp
#include <string>
#include <iostream>
using namespace std;


// underling 承诺不修改形参，于是
// underling 成功接收了 fraud 函数承诺
// 不修改的形参。程序由此正常运行。
void underling(const int &num){
    
}

// fraud 函数通过关键字 const 承诺
// 不修改形参 num
void fraud(const int &num){
    underling(num);
}

int main(){
    int num = 666;
    fraud(num);

    cout << num << endl;
    
    system("pause");
}
1234567891011121314151617181920212223242526AI写代码
执行效果
666
请按任意键继续. . .
12AI写代码
```

## 常量成员函数

常来成员函数即不能对所属对象进行修改的成员函数。要在类中声明一个常量成员函数仅需在该函数的函数头后添加关键字 **const** 即可。

```
举个栗子
#include <iostream>
#include <string>
using namespace std;


class Notebook
{
    public:
    string content = "Hello World";
    void visitor() const 
    {
        // 由于该函数已经被声明为常量成员函数，
        // 故使用如下两条语句都将导致 C++ 抛出错误。
        // this.content = "TwoMoons";
        // content = "TwoMoons";
    };
};

int main(){
    Notebook notebook;
    cout << notebook.content << endl;

    notebook.visitor();

    cout << notebook.content << endl;

    system("pause");
}
12345678910111213141516171819202122232425262728AI写代码
执行效果
```

在常量成员函数中，无论你是否通过隐式形参修改所属对象都将引发异常。

```shell
Hello World
Hello World
请按任意键继续. . .
123AI写代码
```