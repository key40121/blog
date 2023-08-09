+++
authors = ["Taichi Ichisawa"]
title = "Effective C++ chapter 3"
date = "2023-05-15"
description = "Effective C++ chapter 3"
tags = [
    "C++",
    "Effective C++",
]
categories = [
    "C++",
    "books",
]
+++

# Chapter 3

This chapter explains what RAII is in general even though the autoher dosen't bring up the word **RAII**.

RAII : Resource Acquisition is Initialization.

Smart pointers do a resource management for you. In the example, ptr has been created by using std::unique_ptr and it relased a resouce when it went out of the scope.

Refer to **Effective Modern C++**.

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::cout << "No ptr" << std::endl;
    {
        auto ptr = std::make_unique<int>(10); // dynamically acquired a memory.
        std::cout << *ptr << std::endl; // only exists in this scope.
    }
    std::cout << "No ptr anymore" << std::endl; // ptr has been released automatically.
    return 0;
}

```cpp
#include <iostream>
#include <memory>
#include <string>

int main()
{
    std::unique_ptr<int> ptr;
    ptr.reset(new int(10)); // reset and old one will be released

    // access
    auto uptr = std::make_unique<std::string>("unique");
    std::cout << *uptr << std::endl;

    // Move
    auto uptr2 = std::make_unique<int>(10);
    auto uptr3 = std::make_unique<int>(15);

    uptr2 = std::move(uptr3); // uptr3 will be released.

    // array
    auto ptrArrayU = std::make_unique<int[]>(10);

    return 0;
}

```

***

## Item 13 : Use objects to manage resources.

It explains what RAII is, but uses std::auto_ptr which is obsoleted or not recommended to use anymore.

Use unique_ptr, shared_ptr, and weak_ptr instead if you use modern C++.

***

## Item 14 : Think carefully about copying behavior in resource-managing class.
So this item explains what you have to care about when you create a resource-managing class by your own, but is it needed?
I am not sure.

In my perspective, a custome deleter is enough.

***


## Item 16 : Use the same form in corresponding uses of new and delete.

No need as long as you use smart pointers from C++11.

***

## Item 17 : Store newed objects in smart pointers in standalone statements.
Need to remember that C++ compilers dosen't detemine the order in which argument is performed first.

So let's say you declare a function like this

```cpp
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());

f(std::unique_ptr<T>(new T), g());

```

This is really dangerous because if new Widget occurs at first and then calling priority occurs and throws exception, the pointer will be lost.

So, always do like this.

```cpp

std::tr1::shared_ptr<Widget>pw(mew Widget);

processWidget(pw, priority());

std::unique_ptr<T> p(new T);
f(std::move(p), g());

```

***
