+++
draft = true
date = 2023-07-16T13:25:47Z
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++


# Chapter 6

This chapter is all about OOP, and some techniques which are unique to C++.

There are several keywords that are associated with OOP in C++, so I'd like to define those keywords so that I won't be confused with terms when I use them.


***
## Item 32 : Make sure public inheritance models "is-a".

C++ dose not make sure that the hierarchy of inheritance is correctly implemented. Thus, you really have to be careful that the public inheritance ensures "is-a" relationship between base class and derived class.

***

## Item 33 : Avoid hiding inherited names.

***

## Item 34 : Differentiate between inheritance of interface and inheritance of implementation.

Inheritance of functioin interfaces and inheritance of function implementations are different notion. Of courece, that's why C++ has the mechanism to force us to differentiate them.

```cpp

class Shape
{
public:
    virtual void draw() const = 0; // pure virtual function
    virtual void error(const std::string& msg); // simple virtual function
    int objectID() const; // non-virtual function
};

```

Shape is an abstarct class; its pure virtual function drae marks it as such. So, clients cannnot create instances of the Shape class.

- Pure virtual function
Pure virtual function is to have derived classes inherit a function interface only.
"Only interface"

- Simple virtual function
Simple virtual function is to have derived classes inherit a function interface as well as a default implementation.
"Interface and a defaulut implementation"

- Non virtual function
Non virtual function is to have derived classes inherit a function interface as well as mandartory implementation. ("Invariant over specialization")
"Interface and a mandatory implementation"


***

## Item 35 : Consider alternatives to virtual functions.

Non-virtual interface (NVM) idiom.

The strategy pattern via function pointers.

I rewrite what the auther wants to achieve here by using modern C++ not tr1.

```cpp

#include <iostream>
#include <functional>
#include <memory>

class HealthCalcFunc
{
public:
    virtual void execute() = 0;
    virtual ~HealthCalcFunc() {}
};

class GameCharacter
{
public:
    explicit GameCharacter(std::unique_ptr<HealthCalcFunc>&& hcf)
        : healthCalucFunc(std::move(hcf)) {}

    void healthValue()
    {
        if (healthCalucFunc)
        {
            healthCalucFunc->execute();
        }
    }

    void setHandler(std::unique_ptr<HealthCalcFunc> hcf)
    {
        healthCalucFunc = std::move(hcf);
    }

private:
    std::unique_ptr<HealthCalcFunc> healthCalucFunc;
};

int main()
{
    return 0;
}

```

As for std::function it would be...

```cpp

using HealthCalcFunc = std::function<int(const GameCharacter&)>;

```
Then, this is used in GameCharacter class. This is more like a callback design actually.



***

## Item 36 : Never redefine an inherited non-virtual function.

Title explains pretty much everything.
Use virtual function if you want your derived class to override the function of the base class.

***

## Item 37 : Never redefine a function's inherited default parameter value.

Virtually functions are dynamically bound, but default parameter values are statically bound.


***


## Item 38 : Model "has-a" or "is-implemented-in-terms-of" through composition.

layering, containment, aggregation, and composition..They have the same meaning in the context of software.


***

## Item 39 : Use private inheritance judiciously.

***

## Item 40 : Use multiple inheritance judiciously.