+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 2"
date = "2023-10-07"
description = "Effective Modern C++ chapter 2"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 2

***

## Item 5 : Prefer **auto** to explicit type declarations.

This item explains the advantage of using auto.

auto can avoid creating uninitialized variables.

```cpp

int x1; // potantially uninitialized

auto x2; // invalid

auto x3 = 0; // valid

```
auto makes codes more readable (e.g. when using iterator)

```cpp

template <typename It>
void dwim(It b, It e)
{
    while (b != e)
    {
        typename std::iterator_traits<It>::value_type currValue = *b;
    }
}

// This can be
template <typename It>
void dwim(It b, It e)
{
    while (b != e)
    {
        auto currValue = *b;
    }
}

```

Also, you can avoid platform dependent built-in type such as unsigned.

```cpp

std::vector<int> vec;
unsigned sz = vec.size(); // unsigned can be different on different platform

auto sz = vec.size(); // always std::vector<int>::size_type

```



***

## Item 6 : Use the explicitly typed initializer idiom when auto deduces undesired types.

This item explains disadvantage of using auto idiom.
For instance, std::vector<bool> has a tricky behavior.

```cpp
std::vector<bool> features;
bool bit = features[1];

// if bool is auto
auto bit = features[1]; // This invokes undefined behavior.

```

std::vector<bool>'s operator [] returns an object of type std::vector<bool>::reference, not a reference to an element of the container.
C++ forbids references to bits; thus, std::vector<bool> cannot returns its reference of elements.

```cpp

bool bit = features[1];
auto bool = static_cast<bool>(features[1]);

```
