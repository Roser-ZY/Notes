[TOC]

<H3 style="">阅读建议</H3>

本文将介绍*Windows*平台下的动态库与静态库的生成与使用方法，并重点讲解动态库的符号导入与导出方法。

本文的代码示例及编译环境如下，本文代码编译运行过程采用命令行方式：

- *Windows*
- *VS 2022 (MSVC 14.3+)*

本文的示例内容如果影响阅读可自行跳过。

本文中提到的***模块***为不同链接库或可执行程序。

## 一、*Windows*静态库与动态库概述

### 1.1 静态库

生成静态库时会得到一个 `.lib` 文件，**该文件包含了静态库需要的所有信息**。

使用静态库时，**只需要将静态库 `.lib` 文件与目标文件链接即可**，链接器会将 `.lib` 文件的内容嵌入到可执行文件中。

运行可执行文件时，因为静态库的内容已经被嵌入到了可执行文件，所以运行时不再需要静态库文件（这也是称为静态库的原因）。

### 1.2 动态库

动态库生成的文件会因是否导出符号而不同，但不论是否导出符号，动态库都会生成 `.dll` 文件，该文件包含动态库**运行时**所需要的信息，但是**不包括符号链接信息**。

#### 1.2.1 导出符号

如果导出符号，则除了生成 `.dll` 文件外，还会生成 `.lib` 文件，`.lib` 记录了动态库的**符号链接信息**。

使用导出符号的动态库时，**需要在链接时将动态库的 `.lib` 文件与目标文件链接**，链接器同样会将 `.lib` 文件中的内容嵌入到目标文件中。

运行可执行文件并**访问动态库中的符号**时，会**检查链接时嵌入的 `.lib` 符号信息**，然后**根据这些信息在运行时加载的 `.dll` 文件中查找符号定义**（这也是被称为动态库的原因）。

> 如果使用了动态库没有导出的符号，则在链接时会报错：`undefined reference to XXX`，这是使用动态库时最常见的问题。
>
> 遇到上述问题时，**先检查头文件是不是使用了没有导出的符号，再检查动态库是否正确导出了需要的符号（如果是自己生成的动态库）**。
>
> *Windows*平台关于动态库符号找不到的问题，*90%*都可以通过上面的步骤检查出来。

#### 1.2.2 不导出符号

如果不导出符号，则动态库只生成一个 `.dll` 文件。

一般动态库都会导出符号，以便其他模块使用，否则其他模块无法直接**通过头文件声明**的方式直接访问动态库的函数或变量。

但是不导出符号并**不代表其他模块无法使用该动态库**，也有其他的方式（例如运行时*LoadLibrary*方法等）来加载动态库并间接访问其中的函数与变量。

无导出符号的动态库不是本文的重点，本文不讨论该内容。

## 二、符号导出与导入

符号导出与导入**只适用于动态库**，关键字分别为 `dllexport` 和 `dllimport`，用法如下：

```cpp
__declspec(dllexport) declarator;
__declspec(dllimport) declarator;
```

> `__declspec()`关键字为*Microsoft*用于指定存储类信息的扩展属性语法，用法详见[官方文档](https://learn.microsoft.com/en-us/cpp/cpp/declspec?view=msvc-170)。

`declarator` 为声明符，例如函数声明、变量声明、类声明、结构体声明等。

### 2.1 符号导出

符号需要 `__declspec(dllexport)` 修饰后，才会从当前动态库导出符号。

- 对于**全局函数和变量**而言，将 `__declspec(dllexport)` 写在声明前即可导出符号；

- 对于**类和结构体**而言，如果希望将**整个类或结构体导出**（所有成员函数和静态变量），则将 `__declspec(dllexport)` 写在类或结构体声明前；

  如果希望将**部分成员导出**，则将 `__declspec(dllexport)` 分别写在这些成员的声明前；

导出符号时，要求**被导出的符号必须在当前模块的某个编译单元内有定义**（*Definition*），否则会导致编译错误。

> 编译单元通常是指一个源文件及其所包含的头文件。
>
> 纯虚函数是一个例外，纯虚函数可以没有定义。

以下代码简单列举了 `dllexport` 的使用方法。

#### 2.1.1 导出函数和全局变量

```cpp
extern __declspec(dllexport) int externNum;
int __declspec(dllexport) display(int num);
```

<p style="font-size:14px;text-align:center"><i>Code-1.1 导出函数和全局变量</i></p>

`__declspec(dllexport)` 的位置可以在完整声明前，也可以紧跟被导出的符号。

上述例子中，`__declspec(dllexport) void display(int);` 和 `void __declspec(dllexport) display(int);` 效果是一样的。

- 对于变量和函数，建议将`__declspec(dllexport)` 放在声明前；

- 对于类和结构体，建议将 `__declspec(dllexport)` 紧跟被导出的类名和结构体名；

`__declspec(dllimport)` 同理。

> 这里没有介绍文件作用域下 `static` 变量的导出，实际上 `dllexport` 不能用于文件作用域的 `static` 变量，试图导出文件作用域的 `static` 变量会导致编译错误。

#### 2.1.2 导出结构体和类

因为结构体和类在*C++*中十分相似，因此放在一起进行讲解。

##### 导出结构体

```cpp
// 导出完整结构体
struct __declspec(dllexport) Add {
    int lh;
    static int rh;

    Add(int l, int r) {
        lh = l;
        rh = r;
    }
    
    int operator()(){
        return lh + rh;
    }
};

int Add::rh = 0;

// 导出部分结构体
struct Add {
    int lh;
    __declspec(dllexport) static int rh;

    __declspec(dllexport) Add(int l, int r) {
        lh = l;
        rh = r;
    }
    __declspec(dllexport) int operator()(){
        return lh + rh;
    }
};

int Add::rh = 0;
```

<p style="font-size:14px;text-align:center"><i>Code-1.2 导出结构体</i></p>

##### 导出类

```cpp
// 导出完整类
class __declspec(dllexport) UtilClass {
private:
    int result;

public:
    static int utilType;
    int add(int , int);
};

int UtilClass::utilType = 0;

// 导出部分类
class UtilClass {
private:
    int result;

public:
    __declspec(dllexport) static int utilType;
    __declspec(dllexport) int add(int , int);
};

int UtilClass::utilType = 0;
```

<p style="font-size:14px;text-align:center"><i>Code-1.3 导出类</i></p>

**只有 `public` 类成员函数和静态成员能够被导出，普通成员变量和 `private/protected` 成员无法被导出。**

> 结构体可以看做成员可见性均为 `public` 的类，因此其成员函数和静态成员均可被导出。

导出整个类或结构体时，编译器会对类或结构体的成员进行检查，不会导出普通成员变量。

导出部分类或结构体时，需要由开发者注意被导出的成员类型：

- 试图导出普通成员变量会导致编译错误；
- 导出 `private/protected` 成员函数和静态成员不会导致编译错误，但最终得到的 `.lib` 中不会包含这些非公有符号；

<span id="Tag-错误导出部分类成员">此外</span>，在导出部分类或结构体成员时，要求**类或结构体的声明前**不使用 `__declspec(dllexport)`（或 `__declspec(dllimport)`）修饰，即以下用法是错误的：

```cpp
// 导出部分类成员
class __declspec(dllexport) UtilClass {
private:
    int result;

public:
    __declspec(dllexport) int add(int , int);		// 错误，类声明中已经导出了整个类，此处会导致编译错误
};
```

<p style="font-size:14px;text-align:center"><i>Code-1.4 错误导出部分类成员</i></p>

### 2.2 符号导入

导入符号与导出符号的使用方法是一样的，通过 `__declspec(dllimport)` 修饰，表示当前符号为外部模块中的符号，要将其链接到外部模块。

*dllimport*对**函数**而言不是必须的，函数声明前可以不使用 `__declspec(dllimport)` 修饰。

> 建议不要省略，因为准确声明可以让编译器更好地进行代码优化，并且提高代码可读性。
>

但**静态成员变量必须要通过 `__declspec(dllimport)` 修饰**。因此如果访问了其他模块中的类的静态成员变量，则**必须导出整个类或单独导出被使用的静态成员变量**。

> 外部模块的静态成员变量之所以必须通过 `__declspec(dllimport)` 修饰，大概是编译器会更倾向于将静态成员变量限制在模块内访问而非跨模块访问，因此在没有进行导入符号修饰时，编译器不会主动查找符号链接信息，而是会在当前模块查找其定义。
>
> 如果静态成员变量没有表示为导入符号，且当前模块内没有其定义，则会导致链接错误。

如果已经使用 `__declspec(dllimport)` 修饰了符号，则当前模块中的任何编译单元不得再对该符号进行**定义**，否则会导致编译错误。

### 2.3 应用

通常使用宏来控制符号导出与导入，其他模块使用该动态库时直接引入头文件即可将符号导出切换为符号导入，示例代码如下：

```cpp
#ifdef DEVELOP_MODULE						// 所有能够标记当前为开发代码的宏都可以作为判断条件，该宏需要在编译时手动配置（也可利用 CMAKE 等）
#define DLL_EXPORT __declspec(dllexport)	// 定义为导出宏
#else
#define DLL_EXPORT __declspec(dllimport)	// 定义为导入宏
#endif
```

以上代码中的 `DEVELOP_MODULE` 宏可自定义，但必须要确保在编译时有定义，在使用时未定义，否则会导致错误使用*dllexport*和*dllimport*。

`DEVELOP_MODULE` 的命名应尽量特化，以免发布后其他开发者使用时错误定义了宏导致符号链接错误。

> 想要杜绝宏误定义的情况，最好单独维护发布的头文件。