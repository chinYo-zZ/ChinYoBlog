---
title: C++ 反射
date: 2024-06-24 22:50:34
tags:
---
c++构建反射系统的学习记录
## 一、类对象反射
当前框架

![alt text](image01.png)

首先进行工厂的搭建：ClassFactory

```C++
#pragma once
#include <string>
#include <map>
using namespace std;
#include "utility/Singleton.h"

using namespace yazi::utility;
namespace yazi {
namespace reflect {
    //注册反射类需继承的基类
	class Object {
	public:
		Object() {};
		virtual ~Object() {};
	};
    //为函数指针取别名
	typedef Object* (*create_object)(void);
    //工厂
	class ClassFactory
	{
        //友元类，让单例类工厂可以访问私有变量
		friend class Singleton<ClassFactory>;
	public:
		void register_class(const string& className, create_object method);
		Object* create_class(const string& className);

	private:
        //储存注册反射的类
		map<string, create_object> m_classMap;
	};
}}

// ClassFactory.cpp ------------------------------------------------------------

#include "ClassFactory.h"
using namespace yazi::reflect;
void ClassFactory::register_class(const string& className, create_object method)
{
	m_classMap[className] = method;
}

Object* ClassFactory::create_class(const string& className)
{
	// tip : C ++ map find()函数用于查找具有给定键值k 的元素。
	// 如果找到该元素，则返回指向该元素的迭代器。否则，它返回一个指向map末尾的迭代器，即map :: end()。
	auto it = m_classMap.find(className);
	if (it == m_classMap.end()) {
		return nullptr;
	}
	return it->second();
}
```
将注册操作封装进类和宏中
```C++
//ClassRegister.h
#pragma once
#include "reflect/ClassFactory.h"

namespace yazi {
namespace reflect {

	class ClassRegister {
	public:
		ClassRegister(const string& className, create_object method) {
			Singleton<ClassFactory>::instance()->register_class(className, method);
		}
	};

#define REGISTER_CLASS(className)      \
Object * createObject##className(){   \
	Object* obj = new className();     \
	return obj;                        \
	}                                  \
ClassRegister classRegister##className(#className,createObject##className)

}}
```
单例模式类模版
```C++
//Singleton.h
#pragma once
namespace yazi {
namespace utility {

	template<typename T>
	class Singleton
	{
	public:
		static T* instance() {
			if (m_instance == NULL)
				m_instance = new T();
			return m_instance;
		}
	private:
		Singleton() {}
		Singleton(const Singleton<T>&);
		Singleton<T>& operator = (const Singleton<T>&);
		~Singleton() {};
	private:
		static T* m_instance;
	};
	template<typename T>
	T* Singleton<T>::m_instance = NULL;
}}
```
测试类
```C++
#pragma once
#include <iostream>
#include "reflect/ClassRegister.h"
using namespace std;
using namespace yazi::reflect;

class A : public Object
{
public:
	A(){}
	~A(){}

	void show() {
		cout << "A：C++！启动！" << endl;
	}

public:
	string m_name;
};
// tip：注意宏的命名空间
REGISTER_CLASS(A);
```
测试
```C++
//main函数
#include <iostream>
using namespace std;
#include "reflect/ClassFactory.h"
using namespace yazi::reflect;
#include "test/A.h"

int main()
{
    ClassFactory* factory = Singleton<ClassFactory>::instance();
    A* a = (A*)factory->create_class("A");
    a->show();

}
```
输出测试
![test](image02.png)
## 二、类成员数据反射