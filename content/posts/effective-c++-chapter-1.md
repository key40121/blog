---
title: "Effective C++ Chapter 1"
date: 2023-06-11T18:23:32Z
draft: true
---

# Chapter 1
It's been a year since I have started using C++ at work, so this would be good timing to read one of the most famous books in C++ "Effective C++".
Even though there is a book called "Modern Effective C++", it is not a new version of this old "Effective C++"; thus, I am going to understand what the author wants to express and interpret it in a modern way of C++.

I am writing this articel in 2023, which means that we will be able to use C++23 soon; however, Effective C++ was written before C++11 published so there are many things that I have to translate from the classsic C++ to the modern C++.

Also, I will totally abandon old C++(before C++11). Instead, I am trying to compensate those classic parts with the modern way of C++.

## Item 1 : View C++ as a federation of languages.
Nothing to mention....well what you have to know is that there are ways which are C, OOP C++, Template C++, and STL.

## Item 2 : Prefer consts, enums, and inlines to #defines.
It starts from "Prefer the compier to the preprocessor".

```C++
#define ASPECT_RATIO 1.653 // NO WAY lol
```

```C++
const double ASPECT_RATIO = 1.653; // OK what about name convention?
```

The book recommend that we should use "inline". Actually this might be wrong now, because compiler is way smarter than us nowadays, so let compiler decide when to use inline by chooseing optimization option right?

## Item 4 : Make sure that objects are initialized before they’re used.
Uninitialized objects can exist in C++, and reading uninitialized values yields undefined behavior.
!!! Naturally, global static object has an issue of initialization in general in C++, so let's not use it.
Instead, let's use local static object like we do when we use singleton design pattern ;).

```C++
Singleton& getInstance()
{
    static Singleton instance;
    return instance;
}

```