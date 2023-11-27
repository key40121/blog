+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 1"
date = "2023-10-06"
description = "Effective Modern C++ chapter 1"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 1

***

## Item 1 : Prefer **auto** to explicit type declarations.

T : type of template.
ParamType : T with other
e.g. ParamType = T, const T&, T*,...

```cpp
template<typename T>
void f(ParamType param);

f(expr);

```
Basically, C++ deduces T based on ParamType.

e.g. ParamType = T&
```cpp

template<typename T>
void f(T& param);

int x {10};
const int cx = x;
const int& cref = x;

f(x); // T = int, param = int&
f(cx); // T = const int, param = const int&
f(cref); // T = const int, param = const int

```

e.g. ParamType = const T&

```cpp

template<typename T>
void f(const T& param);

int x {10};
const int cx = x;
const int& cref = x;

f(x); // T = int, param = const int&
f(cx); // T = int, param = const int&
f(cref) // T = int, param = const int&

```

e.g. ParamType = T*

```cpp

template<typename T>
void f(T* param);

int x {10};
const int* ptr = &x;

f(&x); // T = int, param = int*
f(ptr); // T = const int, param = const int*

```

e.g. ParamType = T&&
If expr is rvalue, same as before.
If expr is lvalue, T and ParamType are treated as lvalue reference, so only this situation makes T reference(T&).
```cpp

template<typename T>
void f(T&& param);

int x {10};
const int cx = x;
const int& cref = x;

f(27); //prvalue : T = int, param = int&&
f(x); // lvalue : T = int&, param = int&
f(cx); // lvalue : T = const int&, param = const int&
f(cref); // lvalue : T = const int&, param = const int&

```

e.g. ParamType = T
Remove const and reference.
```cpp
template <typename T>
void f(T param);

int x {10};
const int cx = x;
const int& cref = x;

f(x); // T = param = int.
f(cx); // T = param = int
f(cref) // T = param = int

```


***

## Item 2 : Understand auto type deduction.

auto bahaves almost same as templete type deduction that I have introfuced.

e.g.

```cpp
auto x = 10; // int
const auto constx = x; // const int
const auto& refx = x; // const int&

auto&& rx1 = x; // x is lvalue int, so int&
auto&& rx2 = constx; // constx is lvalue const int, so const int&
auto&& rx3 = 10; // 10 is pure rvalue(prvalue), so int&&

```

auto always becomes initializer list with {} initialization.

```cpp

int val1(3); // val1 = 3;
int val2 {5}; // val2 = 5;

auto a1 (3); // int a1 = 3;
auto a2{5}; // std::initializer_list<int> a2 = {5};

```

The situation where you have to take care of how to declare auto.

```cpp

// Not efficient
std::vector<yourDefinedType> vec;
for (auto vec : yourDefinedType)
{

}

// Good
std::vector<yourDefinedType> vec;
for (const auto& vec : yourDefinedType)
{

}

// or
for (auto& vec : yourDefinedType)
{

}

```
***

## Item 3 : Understand decltype.

Start off with typical cases which are very intuitive.

```cpp

const int i = 0; // decltype(i) is const int.
Widget w; // decltype(w) is widget
bool f(const Widget& w); // decltype(w) is const Widget&

struct Point { int x, int y}; // decltype(Point::x) is int.
f(w); // decltype(f(w)) is bool.

```

If you want to return a variable that has a type declared in an argument list of the function, you can simply use auto as a return type of the function from C++14.

```cpp

// c and i is not defined when declareing type of the funciton.
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index i)
{
    return c[i];
}
```

This returnrs rvalue. If you want it to return lvalue reference, you can use decltype(auto)

```cpp
// C++14
template <typename Container, typename Index>
decltype<auto> authAndAccess(Container& c, Index i)
{
    return c[i];
}

// This is valid now
std::deque<uint8_t> d;
authAndAccess(d, 5) = 10; // decltype(authAndAccess(d, 5)) is int&.

```

Simply put, decltype(auto) dosen't ignore const/volatile which are ignored when using auto.
```cpp

Widget w;
const Widget& cw = w;
auto widget1 = cw; // Widget.
decltype(auto) widget2 = cw; // const Widget&

```

In the above example, if you want authAndAccess to return rvalue, you need to use std::forward

```cpp

template <typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
{
    return std::forward<Container>(c)[i];
}

// valid
std::deque<std::string> makeStringDeque();
auto s = authAndAccess(makeStringDeque(), 5);

```


