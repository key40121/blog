+++
authors = ["Taichi Ichisawa"]
title = "C++ Template Basics"
date = "2023-05-05"
description = "C++ Template Basics"
tags = [
    "C++",
    "Basic",
]
categories = [
    "C++",
]
+++

## Compile-time polymorphism

C++ achieves compile-time polymorphism through ***template***.

A template is a class or function with template parameters.
***Template instantiation*** is the process of creating a class or a function from a template.

## Template Class Definitions

You can declare template parameters using either template or class keyword followed by an identifier like this example.
```cpp
template <typename X, typename Y, typename Z>
class MyTemplate
{
public:
    X foo(Y&);
private:
    Z* member;
};

template <typename T>
T my_template_func(T& a, T& b)
{
    return a > b;
}

```

## Instantiating Templates

To instantiate a template class and a template function

```cpp
// temmplate class
// class_name<t_param,...> concrete_class{};
//e.g.
std::vector<int> my_vec{0, 1};

// template function
// auto result = func_name<t_param,...>(f_param, ...);
// e.g. using my_template_func
auto a = 10; auto b = 14;
auto result = my_template_func<int>(a, b);

```

## What is the benefit of generic programming in C++.

Consider this class

```cpp
long mean(const long* values, size_t length)
{
    long result{};
    for(size_t i{}; i<length; i++) {
        result += values[i];
    }
    return result / length;
}
```

What if you want to use other numeric types like int and short?
You can generalize this function using template.

```cpp

template <typename T>
T mean(const T* values, size_t length)
{
    T result{};

    for (size_t i{0}; i < length; i++)
    {
        result += values[i];
    }

    return result / length;
}

int main()
{

    const double num_d[] = {1.1, 2.2, 3.4, 4.5, 5.0};
    const auto res = mean<double>(num_d, 5);

    return 0;
}

```

## Non-Type Template Parameters

```cpp
template <size_t Index, typename T, size_t Length>
T& get(T (&arr)[Length])
{
    static_assert(Index < Length, "Out of bounds");
    return arr[Index];
}

int main()
{

    int num[] = {0, 1, 2, 3};
    std::cout << get<0>(num) << std::endl;

    return 0;
}

```

