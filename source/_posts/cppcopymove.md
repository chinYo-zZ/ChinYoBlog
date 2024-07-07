---
title: C++ 拷贝(深浅)构造函数和移动构造函数
date: 2024-07-07 22:35:16
tags: C++
categories: C++
---

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