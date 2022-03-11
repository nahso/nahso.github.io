---
layout: mypost
title: C++ primer 粗看备忘
categories: [C++, 备忘]
---

# 运算符

## 左值、右值、`decltype`
当一个对象被当做左值的时候，用的是这个对象的值（内存中的内容），当一个对象被当做右值时，用的是这个对象的身份（内存中的位置）

通常左值可以当成右值，实际使用的是它的内容（值）；不能把右值当成左值（也就是位置）

取地址符作用于一个左值对象，返回一个指向该对象的指针，这个指针是右值；解引用、下标运算符、迭代器解引用运算符、string和vector的下标运算符求值结果都是左值

如果表达式的求值结果是左值，`decltype` 用于该表达式得到的是一个引用类型，比如 

```c++
int *p;
// 解引用生成左值，因此下式的结果是 int&
decltype(*p)
// 取地址生成右值，因此下式的结果是 int**
decltype(&p) 
```

## 求值顺序
视具体实现而定，标准中没有明确规定

```c++
// f1 f2 哪个先算不确定
int i = f1() * f2();
// ub行为
int i = 0;
cout << i << "" << ++i << endl;
```

即如果改变了某个运算对象的值，在表达式中的其他地方就不要再使用这个对象

## 赋值
```c++
// 错误，初始化列表不能窄化转换
int k = { 3.14 };
```

## 自增自减
`*it++` 为提取当前值并把 `it` 移动到下一元素，`++` 的优先级要比 `*` 高

## 位运算符
关于符号位如何处理没有明确规定，建议仅将位运算符用于处理无符号类型

左移右移时，如果运算对象是小整型，则它的值会被自动提升

## const
把 `*` 放在 `const` 前面说明指针是一个常量，不变的是指针本身的值，而非指向的那个值。
```c++
int errno = 0;
// 指针的指向不可改，从右向左看，p1首先是一个const，也就是p1的内容不能改，然后p1又是一个 int* ，所以p1是一个指向int的指针
int *const p1 = &errno;
// 指针指向的内容不可改，从右向左看，p首先是一个*，是一个指针，然后这个指针指向的是const int，也就是指针的指向可以改，但是指向的那个内容不可改
const int *p = &errono;
// 指针的指向和指向的具体内容都不可改
const int *const p2 = &errno;
```
- 底层const：指针所指向的对象的是一个 const 
- 顶层const：变量本身是一个 const

## 类型转换
### `static_cast` 
只要不包含底层 `const` ，都可以使用

```c++
double d;
// 任何非常量对象的地址都能存入void*
void *p = &d;
double *p1 = static_cast<double*>(p);
```
### `const_cast`
只能改变运算对象的底层const，但是通过 p 写值是未定义行为。本接口一般用来让代码通过编译

```c++
const char *p;
char *q = const_cast<char*>(p);
```

### `reinterpret_cast`
相当于C语言中的强制类型转换

# 语句
## 语句作用域
每次迭代时都会创建并初始化 `i`
```c++
while (int i = get_num());
```

## switch-case
`case` 标签中的必须是整型常量表达式，以下是错误语句
```c++
int val = 3;
...
case 3.14:
case val:
```
每个 `case` 中间都是独立的作用域

## continue
终止循环中剩余的语句，并直接开始下一次循环条件判断

## goto
无条件跳转到同一函数中的另一条语句

## 异常处理
常见头文件：
- exception: 有最通用的异常类 `exception`
- stdexcept: 有常用的异常类
- new: bad_alloc
- type_info: bad_cast

# 函数
## 局部对象
- 自动对象：生命周期从变量声明开始，到函数块末尾结束
- 局部静态对象：生命周期从变量声明开始，直到程序结束时才销毁

## 重载
形参的顶层 const 会忽略掉，比如下面两个不构成重构，而是会直接报错

```c++
void func(const int i) {}
void func(int i) {}
```
### 数组形参
以下三者等价
```c++
void fun(const int*);
void fun(const int[]);
void fun(const int[10]); // 这里的10只是一个期望
```
但是如果是数组的引用，就必须长度也一样
```c++
void func(int (&arr)[10]);

int i[] = {1}, j[10], k = 0;
func(j) // 正确
func(i) // 错误
func(&k) // 错误
```
### 递归
不推荐对 `main` 函数递归

### 返回数组指针的函数
#### 普通方式
```cpp
数组内的元素类型 (*函数名(参数列表))[数组大小]
```
比如 `int (*func(double d, int a))[10]` ，`func(double d, int a)` 表示这个函数的函数名和参数列表，`(*func(double d, int a))` 表示这个函数的返回值可以解引用，`(*func(double d, int a))[10]` 表示解引用得到一个大小为10的数组，`int (*func(double d, int a))[10]` 表示数组里面装的是`int`

#### 尾置返回类型
```cpp
auto func(int i) -> int(*)[10]
```
#### decltype 方法
```cpp
int odd[] = {1, 2, 3};
decltype(odd)* func() {

}
```

## 默认参数
函数默认参数只在函数声明时检查类型，默认参数只在声明时使用就可以了，实现时不需要写默认参数

以下代码合法
```cpp
int sc(int, int char = ' ');
int sc(int = 1, int = 2, char);
// 下面这个和上两个同时出现时就是错误的
int sc(int, int, char = '*');
```
局部变量不能作为默认实参，全局变量作为默认实参时，其值可以修改。比如：
```cpp
int sc = 1;
void fun(int = sc);

int main() {
    sc = 2;
    // 此时调用的是fun(2)
    fun();
}
```
## constexpr
`constexpr` 指的是可以在编译时就算出值的式子，一个 `constexpr` 的函数体内有且只有一条 `return` 语句，该函数被隐式指定为内联函数

## 类型匹配
函数参数的类型匹配顺序如下；
1. 精确匹配
2. const转换
3. 类型提升（如int到char）
4. 算术类型转换（如double到long，short到char），所有算术类型转换的优先级都一样
5. 类类型转换

比如：
```cpp
void fun(long);
void fun(float);
// 错误，因为double到long和float都是算术类型转换，优先级一样，无法确认调用哪个函数
fun(3.14);
```

## 函数指针
```cpp
bool length(const string&, string&);
// 函数指针声明
bool (*fp)(const string&, string&);
// 加不加&都一样
fp = length;
fp = &length;
// 等价的调用
fp("111", "222");
(*fp)("111", "222");
length("111", "222");
```

在指向不同函数类型的指针间不存在转换规则，必须精确匹配

### 函数指针形参
```cpp
// 二者等价
void fun(int a, bool fp(const string&, const string&));
void fun(int a, bool (*fp)(const string&, const string&));
```

使用typedef
```cpp
// Fun 是类型名，是函数类型
typedef bool Func(const string&, const string&);
// PFun 是类型名，是函数指针
typedef bool (*PFunc)(const string&, const string&);
```

### 返回指向函数的指针
不能返回函数，但是可以返回指向函数的指针
```cpp
// 方法一：
using F = int(int*, int);
F* f1(); // 正确
F f2(); // 错误
using FP = int(*)(int*, int);
FP f3(); // 正确

// 方法二：
int (*f1(int))(int*, int); // f1是一个有int参数的函数，返回的是int(*)(int*, int)，返回值是一个函数指针

// 方法三：
auto f1(int) -> int (*)(int*, int);
```
# 类
## 类作用域和成员函数
成员体可以随意使用类中的其他成员而无需在意这些成员出现的次序，因为编译器分两步处理类，第一步先编译成员声明，第二步才轮到成员函数体

## 构造函数
构造函数不能被声明为const，但是构造函数在const对象的构造过程中可以向其写值，因为直到构造函数完成初始化，对象才能真正得到“常量”属性

## 声明和定义
```cpp
// 类的声明，但是在定义之前，它是不完整的类型
class Screen;

class Link_screen {
    Screen window; // 错误，不能声明一个不完整类型的对象
    Link_screen *next; // 正确，可以声明一个不完整类型的指针
};
```

不完整类型可以定义指向该类型的指针或引用，也可以作为函数声明中的参数或返回类型（也就是只要不是实际分配一个不完整类型的对象的行为，其余行为都可以出现）
```cpp
// 注意：这是一个函数声明而非创建了一个对象
Screen s();
```

## explicit
使用`static_cast`就可以使用`explicit`函数

```cpp
explicit Cls(int a);
...
// 下面代码正确
Cls c = static_cast<Cls>(1);
```

## 聚合类
- 所有成员都是public
- 没有定义任何构造函数
- 没有类内初始值
- 没有基类，没有虚函数

可以使用初始值列表，比如：
```cpp
struct Data {
    int ival;
    string s;
};
Data d = {0, "123"}; // 正确
Data d = {"123", 0}; // 错误
```
## 静态
`static` 关键字出现在类内部的声明语句中，不要在类外的定义中再写一遍 `static`。通常类的静态成员应该在类外部初始化

静态数据类型可以是不完全类型
```cpp
class Bar {
public:
    static Bar b; // 正确
    Bar c; // 错误
};
```

可以用静态成员变量作为默认实参
```cpp
class Screen {
public:
    // 使用静态成员变量
    void fun(char = bkground);
private:
    static char bkground;
};
```

# 标准库
## IO
IO对象不可拷贝或赋值，只能引用或返回
```cpp
ofstream out1, out2;
out1 = out2; // 错误
ofstream o(out1); // 错误
```
刷新缓冲区：
- endl 一个换行和刷新缓冲区
- flush 只刷新缓冲区
- ends 一个空字符和刷新缓冲区

如果程序崩溃，输出缓冲区不会被刷新

### fstream
每次fstream被析构时，都会自动调用close，以out模式打开文件会丢弃已有数据。保留`ofstream`打开文件中已有数据的唯一方法是显式指定`app`模式或`in`模式

```cpp
ofstream out;
// 模式自动设为输出和截断
out.open("xxx");
```

## 容器
容器的赋值相关操作会导致指向左边容器内部的迭代器、引用和指针失效。而 `swap` 操作将容器的内容交换，但是不会导致指向容器的迭代器、引用和指针失效（array和string除外）

## swap和assign
array 不支持 assign ，也不允许花括号赋值，assign 允许从一个不同但相容的类型赋值，或是从容器的一个子序列赋值
```cpp
list<string> names;
vector<const char*> oldstyle;
name = oldstyle; // 错误
name.assign(oldstyle.cbegin(), oldstyle.cend()); // 正确
```

swap操作交换两个相同类型容器的内容，除array外，swap不对任何元素进行拷贝、删除或插入操作。元素不会被移动

## emplace_back 和 push_back
emplace_back是构造元素，push_back是拷贝元素
```cpp
// c 是Sales_data对象的容器
c.emplace_back("100-10", 21, 25.99); // 调用了Sales_data的三参数构造函数
c.push_back("100-10", 21, 25.99); // 错误，没有这种参数的push_back
c.push_back(Sales_data("100-10", 21, 25.99)); // 和第一行的emplace_back一样
```
## front/back 和 begin/end
front/back返回的是元素，begin/end返回的是迭代器。对空容器调用front/back的行为是未定义

如果使用auto接收front/back的值，则要给auto加上&，否则auto只会被推断为是非引用

## ostream_iterator
`*it, ++it, it++` 对迭代器不作任何操作，只是返回迭代器本身，因此下面两种写法等价

```cpp
ostream_iterator<int> it(cout, ", ");
for (auto e : vec) 
    *it++ = e;
for (auto e : vec)
    it = e;
```

## 智能指针 
### shared_ptr
shared_ptr的构造函数是explicit，也就是不能隐式类型转换

```cpp
shared_ptr<int> sp1 = make_shared<int>(1); // 正确
shared_ptr<int> sp2(new int(1)); // 正确
shared_ptr<int> sp3 = new int(1); // 错误，不可隐式转换

shared_ptr<int> mkshare() {
    return new int(1); // 错误，不可隐式转换
}
```

可以用shared_ptr 实现自动断开连接之类的工作
```cpp
connection c = connect(&d);
// 即使是由于异常退出，连接也能正常关闭，但是这替换了原有的deleter，从而connection无法析构
shared_ptr<connection> p(&c, [](connection* c) {
    c->disconnect();
});
```

### unique_ptr
```cpp
// 将p2指向内存的所有权交给p3
p3.reset(p2.release());
```
u.release() 使u放弃所有权，返回指针，并将u置空

## 动态数组
unique_ptr可以托管动态数组，在release时会调用`delete[]`析构。shared_ptr托管动态数组时必须指定`deleter`

## allocator
分配的内存只有在真正需要时才执行对象创建操作，allocator分配的是原始的、未构造的空间

# 拷贝、赋值、销毁
直接初始化：
```cpp
string dots(10, '.');
string s(dots);
```
拷贝初始化：
```cpp
string s2 = dots;
string s4 = string(10, '.');
```
以下情况是拷贝初始化：
1. 使用赋值运算符定义变量
2. 将对象作为实参传递给一个非引用类型的形参
3. 将一个返回类型为非引用类型的函数返回一个对象
4. 用花括号列表初始化一个数组中的元素或一个聚合类中的成员

编译器可以跳过拷贝构造函数，从而`string s = "123`变成为`string s("123")` 

拷贝和赋值一定要同时指定

`nocopy(const nocopy&) = delete` 可以阻止拷贝构造函数

## 对象移动
左值：对象的身份，右值：对象的值。左值持久而右值短暂，使用`&&`表示右值引用

将左值转化为右值：`std::move`

```cpp
// 移动构造函数
cls::cls(cls &&s) {
}
```
不抛出异常的移动构造函数和移动赋值运算符必须标记为`noexcept`

移动构造函数和拷贝构造函数同时存在时，等号右边是左值时使用拷贝，是右值使用移动。如果没有移动构造函数，右值也被拷贝。赋值运算符即是移动赋值运算符，又是拷贝赋值运算符
```cpp
hp = hp2; // 使用拷贝方式
hp = std::move(hp2); // 使用移动方式
```

## 运算符重载
以下运算符要定义为成员函数：赋值、下标、解引用(*)、箭头、括号、类型转换

类型转换函数一般形式：`operator 类型() const;`，可以给这个函数加上`explicit`以防止隐式类型转换
```cpp
smallint si;
si + 3; // 将si隐式地转换为int
static_cast<int>(si) + 3; // 显示类型转换
```

后自增要加一个额外的int参数，前自增不需要

## 函数调用运算符
```cpp
// 只可装入函数指针，但是不可装入命令了的lambda
map<string, int(*)(int, int)> m;

// 可以装入函数指针、标准库函数对象、用户命名的函数对象、未命名/已命名的lambda
map<string, function<int(int, int)>> m;
```
# OOP
基类通常应该定义一个虚析构函数，即使该函数不执行任何实际的操作。只有这样才可以实现动态绑定，否则可能会只析构基类而不析构派生类

`class 类名 final`代表不允许继承

编译器在编译时只检验静态类型
```cpp
Base b;
Derive *d = &b; // 正确
Base *bp = d; // 错误，必须用dynamic_cast或static_cast
```

```cpp
void fn() final; // 可以防止后续覆盖
```

回避虚函数机制
```cpp
double d = baseP->Derived2::price(); // 强制使用Derived2的成员函数，此调用在编译时完成解析
```
可以为纯虚函数提供定义，但必须定义在类的外部。含有纯虚函数的类是抽象类，不可创建抽象类的对象

派生类对于一个基类的对象中受保护的成员没有任何访问特权，也就是友元只管自己的类，`clobber`是Der的友元，但是不是Base的友元。友元也不能继承
```cpp
class Base {
protected:
    int prot;
};

class Der : public Base {
    friend void clobber(Der&);
    friend void clobber(Base&);
    int j;
};

// 正确
void clobber(Der &d) {
    d.prot = 0;
    d.j = 0;
}

// 不可访问
void clobber(Base &b) {
    b.prot = 0;
}
```

派生类可以使用using来访问那些本就能访问的基类成员
```cpp
class Base {
public:
    int pub;
protected:
    int prot;
};

class Der : private Base {
public:
    // pub本就是public成员，但是进行private继承后就不可见了，这里重新using public后就又可见了
    using Base::pub;
protected:
    using Base::prot;
};
```

内层作用域（派生类）的名字会隐藏定义在外层作用域（基类）的名字。派生类不会重载，只会隐藏，但是可以显示指出基类的名字来调用。虚函数不会被隐藏

容器中存放继承体系的对象
```cpp
vector<基类> v;
v.push_back(派生类()); // 只能装入基类的部分，也只能访问到基类部分

vector<shared_ptr<基类>> v1;
v1.push_back(make_shared<派生类>()); // 可以访问到派生类部分
```

## 虚基类
承诺愿意共享基类：比如 istream 和 ostream 都继承自base_ios，而iostream是istream和ostream的子类，从而一个iostream里可能有istream的那份base_iso，也可能有ostream中的那份base_ios，虚基类允许istream和ostream共享一份base_ios
```cpp
class istream : public virtual base_ios;
class ostream : public virtual base_ios;
class iostream : public istream, public ostream;
```
虚基类总是先于非虚基类构造，与他们在继承中的位置无关


# 模板
尽量降低对模板参数的要求
```cpp
template <typename T>
int compare(const T &v1, const T &v2) {
    if (v1 < v2) return 1;
    // 只使用了小于，不强求T实现大于号
    if (v2 < v1) return -1;
    return 0;
}
```

在模板中可以定义非类型参数，表示一个值。常用来推断数组的长度
```cpp
template<unsinged N, unsigned M>
int comp(const char (&p1)[N], const char (&p2)[M]);
```

默认情况下，对于一个实例化了的类模板，其成员只有在使用时才被实例化

一个非模板类可以包含一个模板成员函数，但是这个模板成员函数不能是虚函数。编译器通常不对模板进行类型转换，而是生成一个新的模板实例，将实参传递给带模板类型的函数形参时，能自动应用类型转换的只有const转换或函数到指针的转换；如果函数参数不是模板类型，则会进行正常的类型转换

指定显式模板实参：
```cpp
template<typename T1, typename T2, typename T3>
T1 sum(T2, T3);

auto val = sum<double>(1.2, 3.4); // 返回值类型无法推断，只能显式给出

// 糟糕的设计，调用时必须把三个模板参数都指明。如果只指明第一个参数，T3还是不知道到底是什么类型
template<typename T1, typename T2, typename T3>
T3 sum(T1, T2);
```

使用std::forward可以保持类型信息

# 其他
int是内置类型，默认构造函数就是未初始化
```cpp
int *p = new int; // 未初始化
int *p1 = new int(); // 值初始化，新空间用0赋值

int *ap = new int[40]; // 未初始化
int *ap1 = new int[40](); // 值初始化

int arr[0]; // 错误
int *arr = new int[0]; // 正确，但是不能解引用

// 以下内容保证已初始化
int *p = new int{};
auto *p = ew complex<double>{};
```
## 异常
```cpp
void f() noexcept {
    throw exception(); // 能通过编译，但是会有警告
}

noexcept(f()); // noexcept 作为运算符使用，返回的是一个bool值，指示f()是否有noexcept标记

// h会不会被标记为noexcept取决于f有没有noexcept标记
void h() noexcept(noexcept(f())) {

}
```

## 命名空间
在namespace A第一次出现的地方前面加inline，后面的A也隐式地被指定为inline。内联命名空间中的名字可以直接被外层命名空间使用，即使inline命名空间已经在最外层了（即未命名空间）
```cpp
namespace outer {
    namespace inner1 {
        void f();
    }
    inline namespace inner2 {
        void f();
    }
    // 此时调用的是inner2的f 
    g() {
        f();
    }
}
```
例：外层已经没有命名空间了（未命名空间）
```cpp
namespace inner1 {
    void f() {
        cout << "inner1" << endl;
    }
}

inline namespace inner2 {
    void f() {
        cout << "inner2" << endl;
    }
}

void g() {
    f();
}


int main() {
    // 调用的是inner2的f
    g();

    return 0;
}
```
命名空间的别名：`namespace alias = xxx;`

## new
当只传入一个指针类型的实参时，new只会构造对象但是不会分配内存
```cpp
char *buf = new char[sizeof(int) * 3];
int *p1 = new (buf + sizeof(int) * 2)int(0); // 实例化一个int，并将其放置到buf的第三个位置
```

## 运行时类型识别（RTTI）
typeid作用于指针（而非对象）时返回的结果是该指针的静态编译时类型，如果要比较的话就把指针解引用

## 枚举
枚举将一组整数常量组织在一起，不既定作用域：`enum` ，既定作用域：`enum class`

```cpp
enum color {red, yellow, green};
enum class pepper {red, yellow, green};

color eyes = red; // 正确
peppers p = peppers::red; // 需要指定作用域
```
默认情况下，枚举值从0开始，依次加1，可以重复。限定作用域的枚举成员类型是int，不既定作用域的枚举成员没有默认类型

# 不可移植特性
## 位域
位域bit-filed在内存中的布局是机器相关的，且必须是整形或枚举类型，不能用取地址，不能用指针指向类的位域

## volatile
如果一个变量被volatile修饰，编译器将不会把它存放在寄存器中，而是每一次都去访问内存中该变量的位置

指针volatile的指针必须也是volatile的，合成的拷贝对volatile无效

## 链接指示
指示出任意非c++函数所用的语言。extern “XX”可以告诉编译器，该函数是用其它语言来编写的，且强制这些函数采用XX语言的方式进行编译。
```cpp
extern "C" {
    size_t strlen(const char*);

    // 可以直接引入头文件
    #include <stdio.h>
}
```
导出c++函数到其他语言：
```cpp
// 本函数可以被C语言调用
extern "C" double calc() {}
```
