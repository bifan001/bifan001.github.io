## 实验：研究C++的对象模型

### 题目

> 1、定义一个类，其中有静态数据成员、各种类型非静态数据成员（含字符指针），甚至包括引用（可选），静态和非静态成员函数（含分配空间的构造函数、析构函数）。
> 2、定义全局对象、`main`函数中局部对象、另一个被main调用的外部函数`func`中定义局部对象（可以是形参）、`main`函数中动态创建对象，每种对象至少2个。观察、分析各种对象地址。
> 3、输出对象中各个静态与非静态数据成员的值、地址、对象的存储空间大小等信息。由此理解对象的本质、静态数据成员是本类对象共享一份拷贝等问题。此外，应观察对齐现象。
> 4、（可选）输出对象的每个字节，以揭示引用的实现方法。
> 5、对于上述各种对象，输出静态、非静态成员函数地址，以及`main`、`func`等外部函数的地址，并分析。要求采用合理方法，避免编译器提出警告。

### 分析

#### 1、

- 定义类`MyClass`、静态数据成员`StaticInt`、非静态数据成员`nonStaticInt`、`charPtr`(字符指针)、`refToInt`(引用)
- 静态成员函数`Staticfunc`、非静态成员函数`nonStaticfunc`、构造函数`MyClass`、析构函数`~MyClass`

#### 2、

- 定义全局对象`globalobj1`和`globalobj2`
- 定义`main`函数中局部对象`localobj1`和`localobj2`
- 被`main`调用的外部函数`func`中定义局部对象`localfuncobj1`和`localfuncobj2`
- `main`函数中动态创建对象`dynamicobj1`和`dynamicobj2`

#### 3、

- 通过在类中定义对象成员输出函数输出对象中各个静态与非静态数据成员的值
- 输出对象地址及对象中每个成员的值理解对象的本质、静态数据成员是本类对象共享一份拷贝
- 在面向对象编程中，对象是类的实例。类定义了数据结构和可以对这些数据执行的操作（方法）。当创建一个类的实例时，就生成了一个对象，这个对象拥有该类定义的所有属性和行为。

#### 4、

- 一旦引用被初始化为指向某个变量，它就不能再指向其他变量。
- 引用总是有效的，并且始终引用最初绑定的那个变量。

#### 5、

- 静态成员函数的行为类似于普通函数，因此可以直接将其转换为普通函数指针并输出地址。
- 非静态成员函数需要通过成员函数指针来获取地址。成员函数指针的类型与普通函数指针不同，因此需要使用`reinterpret_cast`或`union`来避免编译器警告。



### 结果截图

![Image](https://github.com/user-attachments/assets/45586c95-198d-4ea0-8a30-b91dca703ce3)

![Image](https://github.com/user-attachments/assets/74b4e5dc-cdac-44bb-8fd7-3c9d76d707b9)

### 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;

class MyClass {
public:
    static int StaticInt; //静态数据成员
    int nonStaticInt; //非静态数据成员
    char* charPtr; //字符指针非静态数据成员
    int& refToInt; //引用

    //构造函数
    MyClass(int val, const char* ptr) : nonStaticInt(val), refToInt(nonStaticInt) {
        charPtr = new char[strlen(ptr)+1];
        strcpy(charPtr,ptr);
    }

    //析构函数
    ~MyClass() {
        delete[] charPtr;
    }

    //静态成员函数
    static void Staticfunc() {
        
    }

    //非静态成员函数
    void nonStaticfunc()  {
        
    }

    //输出对象中每个成员的函数
    void printMembers() const {
        cout << "StaticInt" << StaticInt << "\n"
             << "nonStaticInt: " << nonStaticInt << "\n"
             << "charPtr: " << charPtr << "\n"
             << "refToInt: " << refToInt << std::endl;
    }


    //输出对象的每个字节的成员函数
    void printObjectBytes() const {
        const unsigned char* p = reinterpret_cast<const unsigned char*>(this);
        for (size_t i = 0; i < sizeof(*this); ++i) {
            printf("%02x ", p[i]);
        }
        cout << endl;
    }
};

//初始化静态成员变量
int MyClass::StaticInt = 0;

//定义全局对象
MyClass globalobj1(10, "gloabalone");
MyClass globalobj2(20, "gloabaltwo");

//外部函数func中定义局部对象，被main调用
void func1(MyClass paramobj1) {
    MyClass localfuncobj1(30, "Local in Func one");
    //在外部函数中输出对象地址
    cout << "Address of localfuncobj1: " << &localfuncobj1 <<endl;
}
void func2(MyClass paramobj2) {
    MyClass localfuncobj2(40, "Local in Func two");
    //在外部函数中输出对象地址
    cout << "Address of localfuncobj2: " << &localfuncobj2 <<endl;
}

//main
int main() {
    //定义main函数中局部对象
    MyClass localobj1(50, "localone");
    MyClass localobj2(60, "localtwo");

    //定义main函数中动态创建对象
    MyClass* dynamicobj1 = new MyClass(70, "dynamicone");
    MyClass* dynamicobj2 = new MyClass(80, "dynamictwo");

    // 输出对象的每个字节
    cout << "输出对象的每个字节" <<endl;
    cout << "globalobj1:" << endl;
    globalobj1.printObjectBytes();
    cout << "globalobj2:" << endl;
    globalobj2.printObjectBytes();
    cout << "localobj1:" << endl;
    localobj1.printObjectBytes();
    cout << "localobj2:" << endl;
    localobj2.printObjectBytes();
    cout << "dynamicobj1: " <<endl;
    dynamicobj1->printObjectBytes();
    cout << "dynamicobj2: " <<endl;
    dynamicobj2->printObjectBytes();

    // 输出对象每个成员的值
    cout << "输出对象中每个成员的值" <<endl;
    cout << "globalobj1:" << endl;
    globalobj1.printMembers();
    cout << "globalobj2:" << endl;
    globalobj2.printMembers();
    cout << "localobj1:" << endl;
    localobj1.printMembers();
    cout << "localobj2:" << endl;
    localobj2.printMembers();
    cout << "dynamicobj1: " <<endl;
    dynamicobj1->printMembers();
    cout << "dynamicobj2: " <<endl;
    dynamicobj2->printMembers();

    //输出各种对象地址
    cout << "输出各种对象地址及对象中成员的值" << endl;
    cout << "Address of globalobj1: " << &globalobj1 <<endl; 
    cout << "Address of globalobj2: " << &globalobj2 <<endl;
    cout << "Address of localobj1: " << &localobj1 <<endl;
    cout << "Address of localobj2: " << &localobj2 <<endl;
    cout << "Address of dynamicobj1: " << &dynamicobj1 <<endl;
    cout << "Address of dynamicobj2: " << &dynamicobj2 <<endl;

    //调用外部函数
    func1(localobj1);
    func2(localobj2);

    //输出函数地址
    cout << "输出函数地址" << endl;

    //输出静态函数地址
    cout << "Address of Staticfunc: " << reinterpret_cast<void*>(&MyClass::Staticfunc) << endl;

    //定义非静态函数的成员函数指针
    void (MyClass::*nonStaticfuncptr)() = &MyClass::nonStaticfunc;
    cout << "Address of nonStaticfunc: " << reinterpret_cast<void*&>(nonStaticfuncptr) << endl;
    
    //输出main函数地址
    cout << "Address of main: " << reinterpret_cast<void*>(&main) << endl;

    //输出外部函数地址
    cout << "Address of func1: " << reinterpret_cast<void*>(&func1) << endl;
    cout << "Address of func2: " << reinterpret_cast<void*>(&func2) << endl;

    delete dynamicobj1;
    delete dynamicobj2;



    return 0;
}

```

