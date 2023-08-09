+++
authors = ["Taichi Ichisawa"]
title = "Effective C++ chapter 5"
date = "2023-06-10"
description = "Effective C++ chapter 5"
tags = [
    "C++",
    "Effective C++",
]
categories = [
    "C++",
    "books",
]
+++


# Chapter 5

## Item 27 : Minimize casting.

Syntax of cast in C++.

```cpp

// Old-style casts
// (T)expression
int a;

(double)a; // C-Style
double(a); // Function-sty;e

//const_cast<T>(expression)
const int* p;
int* q = const_cast<int*>p; // cast away the constness.

// dynamic_cast<T>(expression)
class Base
{
public:
    virtual ~Base(){}
};
class Derived : public Base {}

Derived* derived = dynamic_cast<derived*>(new Base()); // Downcasting.

// reinterpret_cast<T>(expression)
chat cp[] = { 0x89, 0xAB, 0xCD, 0xEF};
int* a = reinterpret_cast<int*>(cp);
std::cout << std::hex << *a << std::endl;

// static_cast<T>(expression)
double a{10.4};
int x = static_cast<int>a; // 10;

```

In general, using dynamic_cast is not a good practice.
If you encounter the situation where you have to use dynamic_cast, you should reconsider the design.


***

## Item 28 : Avoid returning "handles" to object internals.


## Item 29 : Strive for exception-safe code.
Exception-safe... very important.

The author claims that there are two requirements that an exception-safe code has to meets.
1. Leak no resources.
2. Don't allow data structures to become corrupted.

1. Leak no resources.
=> Use RAII to avoid leak resouces.

```cpp
void something(cv::Mat& img)
{
    Lock ml(&mutex); // RAII
    // No matter what happens it automatically release the mutex.
}
```

2. Don't allow data structures to become corrupted.

There are three guarantees that excception-sage functions can offer.
1. **the baseic gurantee**
Even if an exception is thrown, everything in the data program remains in a valid state.

2. **the strong gurantee**
If an exception is thrown, the state of the progmram wouldn't be changed. In other words, there are only two state possible, before the call or after the call with suceess.

3. **the nothrow gurantee**
 A function which never throw exceptions.

 e.g.
 ```cpp
 #include <new>

int* p = new(std::nothrow) int[10];
if (p == nullptr)
{
    std::cout << "Allocation failed". << std::endl;
}

 ```

Again, the author introduces **copy and swap idiom** too.

Let me create a simple **copy and swap idiom**.
1. Acquire a new resource before release a present resource by using RAII.
2. If acquiring a new resource sucessfully, use copy nad swap(not throws exceptions)



***

## Item 30 : Understand the ins and outs of inlining.
Need to think about


***

## Item 31 : Minimize compilation dependencies between files.

