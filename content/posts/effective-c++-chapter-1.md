+++
authors = ["Taichi Ichisawa"]
title = "Effective C++ chapter 1"
date = "2023-05-10"
description = "Sample"
tags = [
    "C++",
    "Effective C++",
]
categories = [
    "C++",
    "books",
]
+++

# Chapter 1
It's been a year since I have started using C++ at work, so I believe this would be good timing to read one of the most famous books in C++ **Effective C++**.

There is a simillar book called **Effective Modern C++** which is written by the same author, but it is not a precise new version of this old **Effective C++** for some reasons; thus, I am trying to interpret what the author wanted to express in **Effective C++** by using examples that are based on C++14/17/20.

This is just a personal blog, so don't take it seriously.

***

## Item 1 : View C++ as a federation of languages.
Nothing to mention....well what you have to know is that there are several aspects in C++, which are C, OOP C++, Template C++, and STL.

***

## Item 2 : Prefer consts, enums, and inlines to #defines.
It starts from "Prefer the compier to the preprocessor".

```cpp
#define ASPECT_RATIO 1.653 // NO WAY
```

```cpp
// These two are different. It is explained in effective modern C++.
const double ASPECT_RATIO = 1.653; // => RAM
constexpr double ASPCET_RATIO = 1.653; // => ROM
```

The book recommend that we should use "inline". Actually this might be wrong now, because compiler is way smarter than us nowadays, so let compiler decide when to use inline by chooseing optimization option :)

***

## Item 4 : Make sure that objects are initialized before they’re used.
Uninitialized objects can exist in C++, and reading uninitialized values yields undefined behavior.

Naturally, global static object has an issue of initialization in general in C++, so let's not use it.

Instead, let's use local static object like we do when we use singleton design pattern ;).


```cpp
Singleton& getInstance()
{
    static Singleton instance;
    return instance;
}

```