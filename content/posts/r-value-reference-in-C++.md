+++
authors = ["Taichi Ichisawa"]
title = "C++ value categories with std::move"
date = "2023-10-05"
description = "Sample"
tags = [
    "C++",
]
categories = [
    "C++",
]
+++


***

## Declaration of rvalue and lvalue reference

T type l-value reference would be

```cpp
T& lvalue_reference = ;
```

T type r-value reference would be
```cpp

T&& rvalue_reference = ;

```

When initializing lvalue reference, you need to use lvalue.
On the other hand, you need rvalue when initializing rvalue reference.
```cpp

int num = 0;
int& hoge() { return num }

int&& hoge2() { return 0; }
int hoge3() { return 0; }

int main()
{
    // lvalue reference
    int& a = num;
    int& b = hoge();


    // rvalue reference
    int&& a = 0;
    int&& b = hoge2();
    int&& c = hoge3();
}

```

You cannot initialize rvalue reference using lvalue, and lvalue reference using rvalue.


## The definition of rvalue, lvalue, xvalue, glvalue, and prvalue.

In C++ (from C++11), the expressions are categorized according to the following taxonomy.

            expression
          /             \
    glvalue            rvalue
    /       \        /     \
lvalue        xvalue     prvalue

I will go through each expression one by one.

### lvalue
A value (object) with a name.
```cpp
// lvalue
int value;
int& ref = value;
```

### prvalue
prvalue stands for pure rvalue which is an object that has no name.

```cpp
int f() { return 0; }

// pravalue
0;
1 + 1;
f();

```

### xvalue
xvalues refers to an object(eXpiring lvalue), usually near the end of its lifetime(so that its resources may be removed, for instance).

```cpp

int&& f() { return 0; }

// xvalue
// The result of return value of rvalu reference.
int&& r = f();

// Cast of rvalue reference to type T.
int object;
int&& r = static_cast<int&&>(object);

```

### rvalue
rvalue is an xvalue or prvalue.

```cpp

// lvalue object
int lvalue { };

// works
// l reference can be initialized with lvalue
int& l_reference = lvalue;

// works
// r value reference can be initialized with rvalue.
int&& r_reference = static_cast<int&&>(lvalue);

```

### glvalue
glvalue stands for generalized value. It is either lvalue or xvalue.

***

## std::move

std::move is a part of std library which is used to convert some values to xvalue.
For instance, if you have std::move(e), what it dose is to convert 'e' into xvalue.

```cpp

int main()
{
    int l_value;
    int&& r_reference = std::move(l_value);

    // same
    int&& r_reference2 = static_cast<int&&>(l_value);
}

```

### Implementation of std::move
It has to be able to cast both lvalue and rvalue into rvalue.
When template parameter is rvalue reference, C++ is capable of receiving lvalue as well, and it is called forward reference.

```cpp
// forward reference (T&&)
// can receive both lvalue and rvalue
// if it receives lvalue, template parameter T becomes T&.
template <typename T>
T&& move(T&& t) noexcept
{
    return static_cast<T&&>(t);
}

// e.g.

int lvalue;

// && is ignored and template parameter becomes T&.
move(lvalue)
```

So, forward reference in C++ makes move function looks like this when it receives lvalue.
This is not something desired.

```cpp
// if it receives int lvalue.

int& move(int& t) noexcept
{
    return static_cast<int&>(t);
}
```

Hence, we need to utilize another capability of C++, which is std::remove_reference<T>.
It just removes both rvalue and lvalue references from type T.

```cpp
template <typename T>
void func()
{
    // remove reference from type T
    using rf = std::remove_reference_t<T> &&;

    // e.g.
    // int
    using A = std::remove_reference_t<int>;

    // int
    using B = std::remove_reference_t<int&>;

    // int
    using C = std::remove_reference_t<int&&>;

    // int&
    using D = std::add_lvalue_reference_t<int>;

    // int &&
    using E = std::add_rvalue_reference_t<int>;
}
```
By combining forward reference and remove_reference, we can obtain the actual std::move.

```cpp
template <typename T>
std::remove_reference_t<T>&& move(T&& t) noexcept
{
    // always removes (&, &&) from template parameter T.
    // Then cast && to make it rvalue reference.
    return static_cast<std::remove_reference_t<T>&&>(t);
}

```

Ref : https://stackoverflow.com/questions/3601602/what-are-rvalues-lvalues-xvalues-glvalues-and-prvalues

## Usage of std::move with class

```cpp

template <typename T>
class dynamic_array
{
private:
    T* first;
    T* last;

public:
    dynamic_array(std::size_t size=0)
        :first(new T[size]), last(first + size)
        {}
    ~dynamic_array()
    {
        delete [] first;
    }

    // move constructor
    dynamic_array(dynamic_array&& r)
        : first(r.first), last(r.last)
    {
        r.first = nullptr;
        r.last = nullptr;
    }

    // move assignment operator
    dynamic_array& operator = (dynamic_array&& r)
    {
        delete first;

        first = r.first;
        last = r.last;

        r.first = nullptr;
        r.last = nullptr;

        return *this;
    }

};

```

