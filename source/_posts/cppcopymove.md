---
title: C++ 拷贝(深浅)构造函数和移动构造函数，还有完美转发（^_^）
date: 2024-07-07 22:35:16
tags: C++
categories: C++
---
**目录**
 - [代码示例](#代码示例)
 - [std::move](#std::move)
 - [完美转发](#完美转发)

# 代码示例
```C++
#pragma warning(disable:4996)

class MyString {
private:
    char* buffer; // 用于存储字符串的动态内存指针

public:
    // 默认构造函数
    MyString() : buffer(nullptr) {}

    //// 拷贝构造函数(浅)
    //MyString(const MyString& other) {
    //    if (other.buffer) {
    //        buffer = other.buffer;
    //    }
    //    else {
    //        buffer = nullptr;
    //    }
    //}

    // 拷贝构造函数(深)
    MyString(const MyString& other) {
        if (other.buffer) {
            //新创建指针
            buffer = new char[std::strlen(other.buffer) + 1];
            std::strcpy(buffer, other.buffer);
        }
        else {
            buffer = nullptr;
        }
    }

    // 移动构造函数
    MyString(MyString&& other) noexcept : buffer(nullptr) {
        if (other.buffer) {
            buffer = other.buffer; // 直接转移指针所有权
            other.buffer = nullptr; // 将原对象置为空指针，避免重复释放
        }
    }

    // 析构函数
    ~MyString() {
        if (buffer) {
            std::cout << "调用析构函数"<<std::endl;
            delete[] buffer;
        }
    }

    // 打印字符串内容
    void print() const {
        if (buffer) {
            std::cout << buffer;
        }
        else {
            std::cout << "(null)";
        }
        std::cout << std::endl;
    }

    // 设置字符串内容
    void set(const char* value) {
        if (buffer) {
            delete[] buffer;
        }
        buffer = new char[std::strlen(value) + 1];
        std::strcpy(buffer, value);
    }
};

int main(){
    MyString str1;
    str1.set("Hello");

    MyString str2 = str1; // 调用拷贝构造函数
    std::cout << "str1: ";
    str1.print(); // 输出 "Hello"
    std::cout << "str2: ";
    str2.print(); // 输出 "Hello"

    MyString str3 = std::move(str1); // 调用移动构造函数
    std::cout << "str1 移动后: ";
    str1.print(); // 输出 "(null)"，因为资源已经移动给了 str3
    std::cout << "str3: ";
    str3.print(); // 输出 "Hello"
}
    
```
# std::move

在 C++ 中，`std::move` 是一个函数模板，它用来显式地将其参数转换为右值引用。它的定义通常如下所示：

```cpp
template <typename T>
typename std::remove_reference<T>::type&& move(T&& arg) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(arg);
}
```

这意味着当你调用 `std::move(x)` 时，如果 `x` 是一个左值（如一个具名变量），它将被转换为一个右值引用。这个过程并不会实际搬移任何数据，而是通过编译器的类型转换机制告知编译器将 `x` 视为可以安全移动的对象。

### 内存的影响
当你使用 `std::move` 对左值进行操作时，内存本身并不会发生变化。`std::move` 只是改变了对象的类型标记，告诉编译器此对象可以被移动而非复制。具体来说：

1. **对象标记为可移动**：调用 `std::move(x)` 后，`x` 的类型被转换为一个右值引用，例如 `T&&`，其中 `T` 是 `x` 的原始类型。这意味着编译器可以将 `x` 的资源（如指针或资源所有权）移动到另一个对象，而不需要执行深拷贝操作。

2. **没有内存数据的搬移**：`std::move` 本身并不会导致任何内存的移动或拷贝。它仅仅是一个类型转换的语法工具，告知编译器如何处理这个对象。

3. **搬移语义的应用**：当你使用 `std::move` 返回值或传递给接受右值引用参数的函数时，这些函数可以利用搬移语义来实现高效的资源管理，比如移动构造函数和移动赋值运算符。

# 完美转发
```c++
#include <utility>
using namespace std;

// 另一个函数接受转发的参数
void some_function(int& arg) {
	cout << "左值引用" << endl;
}

void some_function(int&& arg) {
	cout << "右值引用" << endl;
}
// 模板函数实现完美转发
template <typename T, typename U>
void forwarder(U&& arg) {
	// 将参数 arg 完美转发给其它函数
	some_function(arg); // 因为函数形参就是左值，所以默认都是执行左值引用的函数
	some_function(forward<U>(arg)); // 进行完美转发后，就可以将arg继续按传进来的值类型继续传值
}

int main() {
	int x = 5;
	forwarder<int>(x);          // x 是左值，转发为 int&
	forwarder<int>(10);         // 10 是右值，转发为 int&&
	return 0;
}
```