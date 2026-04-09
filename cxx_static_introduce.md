* ©️ Author: [starxsky](https://starxsky.github.io)
* Email: starxsky@outlook.com
* Date : Fri Apr 10 12:00 AM
* Editor: Visual Studio Code 1.64.3
* CMake Version : 3.29.3
* CXX Complier : clang++ (arm64-apple-darwin25.0.0 clang-1600.0.26.6)
# 何意为 static of C++ ?
static 是C++中的一个修饰符，用来控制变量/函数的存储方式和可见性


## 为何引入static
在函数内部定义的变量，程序执行到它的定义处时，会为它在栈上分配空间，函数在栈上分配的资源空间会在函数结束时被编译器释放资源。如果想要实现该变量的持久化寿命周期（也就是保持整个程序在执行过程中该变量所存储的资源在内存中一直保存），首先想到的是将该变量定义为全局变量（Global）但问题是，这样的行为导致破坏变量的访问范围（使得在函数中定义的变量可以被其他地方的函数/行为所操控）。那么这时候就通过引入static修饰符解决此问题。

另外，在 C++ 中，需要一个数据对象为整个类而非某个具体的对象服务,同时又力求不破坏类的封装性,即要求此成员隐藏在类的内部，对外不可见时，也可将其定义为静态数据 。
## 静态数据的存储
首先介**全局静态存储区** ：
全局静态存储区分为 .data段和.bss段。其中：
* .data 段表示的是全局初始化区段：用来存放**初始化**的全局变量和静态变量
* .bss 段表示的是全局为初始化区段： 用来存放**未初始化**的全局变量和静态变量在.bass段的变量，程序执行前会被系统自动清0，所以，**未初始化**的全局变量和静态变量在程序执行前已经为0.

存储在静态数据区的变量会在程序执行之初完成初始化，唯一初始化。

## 在 C++ 中 static 的内部实现机制：
**静态数据成员要在程序一开始运行时就必须存在** 因为函数在程序运行中被调用，所以静态数据成员不能在任何函数内分配空间和初始化。


静态数据成员要实际地分配空间，故**不能在类的声明中定义**（只能声明数据成员）。类声明只声明一个类属性和方法，并不进行实际的内存分配，所以在类声明中写成定义是错误的。它也不能在头文件中类声明的外部定义，因为那会造成在多个使用该类的源文件中，对其重复定义。（例如：当存在A.cpp 和B.cpp两个文件同时包含同一个G.h头文件时将会出现重复）

## 使用 static的优势：
* 节省内存开销：由于变量只需要初始化一次，且静态数据成员只存储一处，供所有对象共用。
* 提高了时间利用效率： 静态数据成员的值对每个对象都是一样，但它的值是可以更新的。只要对静态数据成员的值更新一次，保证所有对象存取更新后的相同的值，这样可以提高时间效率。

## C++ 中static的作用总结：
1. 在修饰**变量** 的时候，static 修饰的静态局部变量只执行初始化一次，而且延长了局部变量的生命周期，**直到程序运行结束**以后才释放。
2. static 修饰全局变量的时候，这个全局变量**只能在本文件**中访问，不能在其它文件中访问，**即便是 extern 外部声明也不可以**。
3. static 修饰一个**函数**，则这个函数的只能在本文件中调用，不能被其他文件调用。
4. static 修饰的**变量**存放在**全局数据区的静态变量区**，包括全局静态变量和局部静态变量，都在全局数据区分配内存。初始化的时候自动初始化为 0。
5. 不想被释放的时候，可以使用static修饰。比如修饰函数中存放在栈空间的数组。如果不想让这个数组在函数调用结束释放可以使用 static 修饰。
6. 考虑到数据安全性（当程序想要使用全局变量的时候应该先考虑使用 static）。
7. 在c++的class中声明的static变量需要在类外部进行定义（初始化）。
8. 类中的static函数不需要实例化即可通过类名使用::调用

## 静态变量和普通变量的区别：
* 全局变量： 全局变量是不显式用 static 修饰的全局变量，全局变量默认是有外部链接性的，作用域是整个工程，在一个文件内定义的全局变量，在另一个文件中，通过 extern 全局变量名的声明，就可以使用全局变量。
* 全局静态变量： 全局静态变量是显式用 static 修饰的全局变量，作用域是声明**此变量所在的文件**，其他的文件即使用 extern 声明也不能使用。

静态全局变量不能被其它文件所用；其它文件中可以定义相同名字的变量，不会发生冲突。

## 静态局部变量的优势
静态局部变量有以下特点：
1. 该变量在全局数据区分配内存；
2. 静态局部变量在程序执行到该对象的声明处时被首次初始化，即以后的函数调用不再进行初始化；
3. 静态局部变量一般在声明处初始化，如果没有显式初始化，会被程序自动初始化为 0；
4. 它始终驻留在全局数据区，直到程序运行结束。但其作用域为局部作用域，当定义它的函数或语句块结束时，其作用域随之结束。

程序把新产生的动态数据存放在堆区，函数内部的自动变量存放在栈区。自动变量一般会随着函数的退出而释放空间，静态数据（即使是函数内部的静态局部变量）也存放在全局数据区。全局数据区的数据并不会因为函数的退出而释放空间。

## 实操Example
### static修饰的变量的声明周期贯穿程序始终：
```cpp
#include <iostream>

int number() {
    static int counter = 0;
    return ++counter;
}; 

int main() {
    for (int i = 0; i <= 10; ++i) {
        std::cout << number() << std::endl;
    }
    return 0;
}

/* Output :
1
2
3
4
5
6
7
8
9
10
11
*/
```
如果不使用static：
```cpp
#include <iostream>

int number() {
    int counter = 0;
    return ++counter;
}; 

int main() {
    for (int i = 0; i <= 10; ++i) {
        std::cout << number() << std::endl;
    }
    return 0;
}

/* Output :
1
1
1
1
1
1
1
1
1
1
1
*/
```

### 类class 中使用static修饰变量和函数
```cpp
#include <iostream>

class YU {
    public:
    // 构造函数
    YU() {
        std::cout << "YU constructor" << std::endl;
        std::cout << b << "\n";
    }
    // 静态函数 ++ b
    static int func() {
        std::cout << "call static function" << std::endl;
        // std::cout << a << std::endl; // static类型的函数无法访问non-static类型的变量
        return ++b;
    }

    // 非静态函数 ++ a
    int func1() {
        a = 0; // 初始化，否则会产生为定义行为
        std::cout << "call non-static function1" << std::endl;
        return ++a;
    }

    void func2() {
        std::cout << "call non-static function2" << std::endl;
        std::cout <<"b:"  << b << "\n";
        std::cout << "a:" << a << "\n"; // non-static类型的函数可访问 static和non-static类型的变量
    }

private:

    int a;
    static int b;
};


int YU::b = 0; // static变量需要在类外部进行定义（初始化）


int main() {
    YU yu; // create 'yu' instance
    std::cout << "===========" << std::endl;
    for (int i = 0; i < 8; i++) {YU::func(); yu.func1(); }
    std::cout << "===========" << std::endl;
    YU();
    std::cout << "===========" << std::endl;
    yu.func2();
    return 0;
}
/* Output :
/Users/...../Downloads/C/build/C
YU constructor
0
===========
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
call static function
call non-static function1
===========
YU constructor
8
===========
call non-static function2
b:8
a:1

Process finished with exit code 0
*/
```