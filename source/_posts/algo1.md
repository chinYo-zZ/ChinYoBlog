---
title: C++ 标准库提供了可以用来填充或生成容器的内容
date: 2024-08-23 00:41:12
tags: C++
---

当然可以！下面是对 `<algorithm>` 头文件中填充相关函数的详细解释，包括它们的功能和用法：

### 1. `std::fill`

**功能**：用指定的值填充范围中的所有元素。

**原型**：
```cpp
template< class ForwardIt, class T >
void fill( ForwardIt first, ForwardIt last, const T& value );
```

**示例**：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v(10);
    std::fill(v.begin(), v.end(), 42);  // 填充所有元素为 42

    for (int num : v) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 2. `std::generate`

**功能**：用指定的生成器函数填充范围中的元素。生成器函数需要返回一个值，该值会被填充到容器中。

**原型**：
```cpp
template< class ForwardIt, class Generator >
void generate( ForwardIt first, ForwardIt last, Generator gen );
```

**示例**：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v(10);
    int start = 1;
    std::generate(v.begin(), v.end(), [start]() mutable { return start++; });

    for (int num : v) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 3. `std::iota`

**功能**：用递增的值填充范围中的元素。起始值是指定的，从这个值开始递增。

**原型**：
```cpp
template< class ForwardIt, class T >
void iota( ForwardIt first, ForwardIt last, T value );
```

**示例**：
```cpp
#include <iostream>
#include <vector>
#include <numeric>  // For std::iota

int main() {
    std::vector<int> v(10);
    std::iota(v.begin(), v.end(), 1);  // 从 1 开始递增填充

    for (int num : v) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 4. `std::fill_n`

**功能**：用指定的值填充指定数量的元素。这是 `std::fill` 的一种变体，适用于填充固定数量的元素。

**原型**：
```cpp
template< class OutputIt, class Size, class T >
OutputIt fill_n( OutputIt first, Size count, const T& value );
```

**示例**：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v(10);
    std::fill_n(v.begin(), 5, 99);  // 填充前 5 个元素为 99

    for (int num : v) {
        std::cout << num << " ";
    }

    return 0;
}
```

这些函数提供了多种方式来填充容器中的元素，可以根据你的需求选择合适的函数。


### 5. `自定义函数`

如果需要一个更灵活的生成序列的方法，可以编写自定义函数来填充容器。以下是一个示例，展示了如何使用自定义函数实现类似于 `std::iota` 的行为。

```cpp
#include <iostream>
#include <vector>

template<typename ForwardIterator, typename T>
void custom_iota(ForwardIterator first, ForwardIterator last, T value) {
    while (first != last) {
        *first++ = value++;
    }
}

int main() {
    std::vector<int> v(10);
    custom_iota(v.begin(), v.end(), 1);  // 从 1 开始填充

    for (int num : v) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 6. `std::partial_sum`

`std::partial_sum` 计算范围内元素的部分和。如果使用递增的生成器填充源范围，可以得到类似 `std::iota` 的结果。

```cpp
#include <iostream>
#include <vector>
#include <numeric>  // For std::partial_sum

int main() {
    std::vector<int> v(10);
    std::iota(v.begin(), v.end(), 1);  // 用 1, 2, 3, ..., 10 填充 v
    std::vector<int> result(v.size());

    std::partial_sum(v.begin(), v.end(), result.begin());

    for (int num : result) {
        std::cout << num << " ";
    }

    return 0;
}
```

`std::partial_sum` 的结果是部分和，所以这不完全等同于 `std::iota`，但展示了如何通过标准库的算法来生成类似的结果。

这些方法提供了不同的方式来填充或生成序列，能够在需要类似 `std::iota` 功能时作为替代方案。