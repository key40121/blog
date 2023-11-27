<!-- +++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 3"
date = "2023-10-08"
description = "Effective Modern C++ chapter 3"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 3

**Moving to Modern C++**

This chapter explains notations like constexper, using, .... which are very important to understand when it comes to efficiency.

***

## Item 7 : Distinguish between () and {} when creating objects.
This chapter explains syntax choices for object initialization from C++11.
e.g.

```cpp

int  x(0); // initializer is in parentheses

int y = 0; // initializer follows "="

int z{0}; // initializer is in braces.

```

Actually, C++11 introduces a new initialization syntax to cover all scenarios(e.g. initialization of STL container). "Braced initialization" is introduced and is can be used as follows.

```cpp

#include <iostream>
#include <vector>

int main()
{
    // brace initialization
    std::vector<int> v{1, 2, 3}; // v's initialied with variables 1, 2, 3.

    // default initialization for non-static value members.
    class hoge
    {
    private:
        uint32_t x{0};
        uint32_t y = 0l;
        // uint32_t z(0) // error, cannnot be used.
    };

    return 0;
}

```

Also uncopyable object such as std::thread, std::atomic, ... can be initialized with parentheses.

```cpp
std::atomic<int> a{0};
std::atomic<int> b (0);
std::atomic<int> c = 0; // not ok
```

Now based on these examples, we can finally understand how efficient "Braced initialization" is because only it can be used everywhere.
In addition, this new syntax is useful to prevent C++ from implementing "implicit narrowing conversions" among built-in types. Let's take a look at an example here.


```cpp
double x, y, z;

int sum1{x + y + z}; // error

int sum2(x + y + z); // okay

int sum3 = x + y + z; // okay
```


Another situation where "Braced initialization" demonstrate its strength is when encountering C++'s "most vexing parse".

Most vexing parse refers to an situation where C++ grammer cannnot distinguish between the creation of an object parameter and specification of a funciont's type.

```cpp

Widget w1(10); // ctr with ().

Widget w2(); // can be interpreted as both function declaration and object creation.
            // C++ treats it as function declaration

```

e.g.
```cpp

class Widget
{
public:
    Widget(int num_ = 5) : num(num_)
    {
        std::cout << "num = " << this->num << std::endl;
    }

private:
    int num;

};

int main()
{
    Widget w1(10);

    //Widget w2(); // errpr

    Widget w3{};

    return 0;
}

// g++ 11.4.0
/*
item7.cpp:19:14: warning: empty parentheses were disambiguated as a function declaration [-Wvexing-parse]
   19 |     Widget w2();
      |
*/

```

### std::initializer_list with braced initialization


## Item 8 : Prefer nullptr to 0 and NULL.

## Item 9 : Prefer alias declarations to typedefs.

## Item 10 : Prefer scoped enums to unscoped enums.

## Item 11 : Prefer deleted functions to private undefined ones.

## Item 12 : Declare overriding functions override.

## Item 13 : Prefer const_iterators to iterators.

## Item 14 : Declare functions noexcept if they won't emit exceptions.

## Item 15 : Use constexpr whenever possible.

## Item 16 : Make const member functions thread safe.

## Item 17 : Understand special member function generation. -->