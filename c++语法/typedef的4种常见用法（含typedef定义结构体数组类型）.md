# [typedef的4种常见用法（含typedef定义结构体数组类型）](https://www.cnblogs.com/guanghui-hua/p/17426155.html)

typedef的4种常见用法：

一、给已定义的变量类型起个别名
二、定义函数指针类型
三、定义数组指针类型
四、定义数组类型

总结一句话：“加不加typedef，类型是一样的“，这句话可以这样理解：
没加typedef之前如果是个数组，那么加typedef之后就是数组类型；
没加typedef之前如果是个函数指针，那么加typedef之后就是函数指针类型；
没加typedef之前如果是个指针数组，那么加typedef之后就是指针数组类型；

```C++
typedef char TA[5];//定义数组类型
typedef char *TB[5];//定义指针数组类型,PA定义的变量为含5个char*指针元素的数组(指针数组类型)
typedef char *(TC[5]);//指针数组类型，因为[]的结合优先级最高，所以加不加()没啥区别，TC等价于TB
typedef char (*TD)[5];//数组指针类型
```

## 一、给已定义的变量类型起个别名

```C++
typedef unsigned char uin8_t;       //uint8_t就是unsigned char的别名，这是最基础的用法

//举例
struct __person
{
    char    name[20];
    uint8_t age;
    uint8_t height;
}
typedef __person person_t;
//以上两段代码也可合并为一段，如下：
typedef struct __person
{
    char    name[20];
    uint8_t age;
    uint8_t height;
}person_t;
```

作用是给`struct __person`起了个别名`person_t`，这种这种用法也很基础

## 二、定义函数指针类型

我们首先来看一下如何定义函数指针变量，然后再看如何定义函数指针类型

### 1. 定义函数指针变量

- `int (*pFunc)(char *frame, int len);`
  定义了一个函数指针变量pFunc，它可以指向这样的函数：返回值为int，形参为char*、int
- `int *(*pFunc[5])(int len);`
  定义了5个函数指针变量：pFunc[0]、pFunc[1]···，它们都可以指向这样的函数：返回值为int*，形参为int

### 2. 定义函数指针类型

定义函数指针类型，必须使用typedef，方法就是，在“定义函数指针变量”加上typedef

```
typedef int (*pFunc_t)(char *frame, int len);//定义了一个类型pFunc_t
```

举例：

```C++
typedef  int (*pFunc_t)(char *frame, int len);//定义了一个类型pFunc_t

int read_voltage(char *data, int len)
{
    int voltage = 0;
    ···//其他功能代码
    return voltage;
}
int main(void)
{
    pFunc_t   pHandler = read_voltage;//使用类型pFunc_t来定义函数指针变量
    ···//其他功能代码
}
```

## 三、定义数组指针类型

这个问题还是分两步，先看如何定义数组指针变量，再看如何定义数组指针类型

### 1、定义数组指针变量

```C++
int(*pArr)[5];//定义了一个数组指针变量pArr，pArr可以指向一个int [5]的一维数组

char(*pArr)[4][5];///定义了一个数组指针变量pArr，pArr可以指向一个char[4][5]的二维数组

int(*pArr)[5];//pArr是一个指向含5个int元素的一维数组的指针变量
int a[5] = {1,2,3,4,5};
int b[6] = {1,2,3,4,5,6};
pArr = &a;//完全合法，无警告
pArr = a;//发生编译警告，赋值时类型不匹配：a的类型为int(*)，而pArr的类型为int(*)[5]
pArr = &a[0];//发生编译警告，赋值时类型不匹配：a的类型为int(*)，而pArr的类型为int(*)[5]
pArr = &b;//发生编译警告，赋值时类型不匹配：&b的类型为int(*)[6]，而pArr的类型为int(*)[5]
pArr = (int(*)[5])&b;//类型强制转换为int(*)[5]，完全合法，无警告
```

上面这个例子中，使用类型转换时，代码的样式略显复杂，试想，我们如果强转为一个结构体数组的指针，那这个强转的括号里的内容得多长！这就直接影响了代码的可读性，因此，强转后的类型应该定义出来。

### 2、定义数组指针类型

如同上面定义函数指针类型的方法，直接在前面加typedef即可，例如

```C++
typedef int(*pArr_t)[5];//定义一个指针类型，该类型的指针可以指向含5个int元素的一维数组

int main(void)
{
    int a[5] = {1,2,3,4,5};
    int b[6] = {1,2,3,4,5,6};
    pArr_t pA;//定义数组指针变量pA
    pA= &a;//完全合法，无警告    
    pA= (pArr_t)&b;//类型强制转换为pArr_t，完全合法，无警告
}
```

## 四、定义数组类型

如果我们想声明一个含5个int元素的一维数组，一般会这么写：`int a[5];`

如果我们想声明多个含5个int元素的一维数组，一般会这么写：`int a1[5], a2[5], a3[5]`···，或者 `a[N][5]`

可见，对于定义多个一维数组，写起来略显复杂，这时，我们就应该把数组定义为一个类型，例如：

```C++
typedef int arr_t[5];//定义了一个数组类型arr_t，该类型的变量是个数组

typedef int arr_t[5];
int main(void)
{
    arr_t d;        //d是个数组，这一行等价于:  int d[5];
    arr_t b1, b2, b3;//b1, b2, b3都是数组

    d[0] = 1;
    d[1] = 2;
    d[4] = 134;
    d[5] = 253;//编译警告：下标越界
}
```

看一个应用：

```C++
typedef struct _jmp_buf
{
    int _jb[_JBLEN + 1]; 
} jmp_buf[1];
```

上面这一段定义来自C库函数 <setjmp.h>，为了理解这个定义，我们把它分解为下面这个样子：

```C++
typedef struct _jmp_buf
{
    int _jb[_JBLEN + 1];
} _jmp_buf_t;

typedef _jmp_buf_t jmp_buf[1];
```

jmp_buf是一个含一个元素的数组类型，数组的元素为_jmp_buf_t类型，也即struct _jmp_buf类型。

这种定义有什么作用呢？看个例子：

```C++
jmp_buf buf;//这一行等价于：struct _jmp_buf buf[1];
buf->_jb[5] = 34;//这一行等价于： (&buf[0])->_jb[5] = 34;
handle(buf);//这一行等价于：handle(&buf[0])
```

本例中，buf变量其实是个含一元素的数组，由于数组名本身就是第一个一级成员的指针，而该数组的成员又是个结构体，因此，我们可以直接用数组名指向结构体的成员__jb。这种技巧有什么好处：例子中，1、定义buf变量，就相当于定义了一个结构体变量，而且buf就是这个结构体实体的指针，2、想把结构体的地址传给某个函数时，不必写取地址符了，buf本身就是地址。总结起来就一句话，jmp_buf这种类型，在定义变量实体的同时，也获得了该变量的地址。