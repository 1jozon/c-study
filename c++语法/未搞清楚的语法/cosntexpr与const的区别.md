`constexpr` 和 `const` 都用于定义常量，但它们之间存在一些关键的区别，主要用于不同的上下文和目的。以下是它们之间的主要区别：

### `constexpr`

1. **编译期确定性**：`constexpr` 变量或函数的值必须在编译时就能确定，这意味着它们可以在任何需要常量表达式的上下文中使用（例如数组大小、枚举值等）。
   
2. **性能优化**：由于其值在编译时已知，因此可以允许编译器进行更多的优化。

3. **可用于函数和变量**：不仅可以用来声明变量，还可以用来声明函数。一个被声明为 `constexpr` 的函数可以在编译期计算结果（如果传入的参数也是编译期可知的），也可以像普通函数一样在运行时使用。

4. **更严格的限制**：对于 `constexpr` 函数，C++ 标准对其有一些额外的限制，比如不能包含复杂的控制流结构（尽管 C++14 之后这些限制有所放宽）。

### `const`

1. **运行时常量**：`const` 关键字表示该变量是只读的，但是它的初始值可以在运行时确定。也就是说，虽然你不能修改这个变量的值，但它不一定要在编译时就知道具体的值。

2. **灵活性更高**：因为 `const` 变量的初始化时间更加灵活，所以它适用于更多场景，尤其是那些在运行时才能确定值的情况。

3. **仅限于变量**：与 `constexpr` 不同，`const` 不能用于修饰函数，它只能用来声明变量为只读。

### 示例

```cpp
// constexpr 变量
constexpr int size = 50; // 编译时常量
int arr[size]; // 正确，因为 size 是在编译时已知的

// const 变量
const int runtimeSize = getSize(); // 假设 getSize() 在运行时返回一个值
// int arr2[runtimeSize]; // 错误，在C++中数组大小必须是编译时常量

// constexpr 函数
constexpr int square(int x) { return x * x; }
constexpr int a = square(5); // 编译时计算
int b = square(10); // 运行时计算
```

综上所述，`constexpr` 主要用于那些需要编译时期确定的值或操作，而 `const` 则更为通用，用于表明一个变量是不可变的，无论其值是在编译时还是运行时确定。选择哪一个取决于你的具体需求。