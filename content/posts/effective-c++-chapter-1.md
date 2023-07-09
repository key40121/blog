---
title: "Effective C++ Chapter 1"
date: 2023-07-08T18:23:32Z
draft: true
---

# Chapter 1
It's been a year and a half since I have started using C++ at work, this would be a good timeing to take a look at some famous books in C++.

I am writing this articel in 2023, which means that we will be able to use C++23 soon; however, Effective C++ was written before C++11 published.

Thus, I will be trying to take a look at things from modern C++ point of view.


## Item 1 : View C++ as a federation of languages.
Nothing to mention.

## Item 2 : Prefer consts, enums, and inlines to #defines.
It starts from "Prefer the compier to the preprocessor".

```C++
#define ASPECT_RATIO 1.653
```

```
const double ASPECT_RATIO = 1.653;
```


## Item 4 : Make sure that objects are initialized before they’re used.
Uninitialized objects can exist in C++, and reading uninitialized values yields undefined behavior.

THere are complicated rules to define which object would be initialized