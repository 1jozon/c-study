## 预处理

1. 去掉注释
2. 预处理指令替换 处理宏定义#ifndef、#define
3. 头文件替换、导入include文件
4. 不检查语法错误

```
g++ -E test.cpp -o test.i
```

## 编译阶段

```
g++ -S test.i -0 test.s
```

1. 把c++翻译成汇编语言
2. 检查代码报错

## 汇编阶段

```
g++ -c test.s -o test.o
```

1. 将.s文件汇编成二进制文件

## 链接阶段

```
gcc test.o -o test
```

1. 将目标文件.o于库文件链接



