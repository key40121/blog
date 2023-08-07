---
title: "Effective C++ Chapter 3"
date: 2023-07-01T04:55:58Z
draft: true
---

# Chapter 3

This chapter explains what RAII is in general even though the autoher dosen't bring up the word **RAII**.

RAII : Resource Acquisition is Initialization.
Smart pointers do a resource management for you. In the example, ptr has been created by using std::unique_ptr and it relased a resouce when it went out of the scope.

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

```
***

## Item 13 : Use objects to manage resources.

It explains what RAII is, but uses std::auto_ptr which is obsoleted or not recommended to use anymore.
Use unique_ptr, shared_ptr, and weak_ptr.

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

```

This is really dangerous because if new Widget occurs at first and then calling priority occurs and throws exception, the pointer will be lost.

So, always do like this.

```cpp

std::tr1::shared_ptr<Widget>pw(mew Widget);

processWidget(pw, priority());

```

***
