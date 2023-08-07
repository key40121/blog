+++
draft = true
date = 2023-07-16T13:25:37Z
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Chapter 4

## Item 18 : Make interfaces easy to use correctly and hard to use incorecctly.

The author says that an interface should be simple but reasonable.

```cpp
class Date
{
public:
    Date(int mounth, int day, int year);
};

// This is not wrong but easy to pass a wrong argument.
// So

class Month
{
public:
    static Month Jan(){ return Month(1); }

private:
    explicit Month(int m);

};

Data(Month::Mar(), 30, 1995);

```

## ITem 19 : Treat class design as type design.

When you create a class, you really have to be careful with implementations.

***

## Item 20 : Prefer pass-by-reference-to-const to pass-by-value.

I think pass-by-reference-to-const is something almost anyone knows.

```cpp

class Test() {};

bool something(Test test); // expensive operation

// Instead
bool something(const Test& test);

```

I didn't know what **slicing problem** is though.
I implemented a simple example.
Basically, if you pass a derived class object as a base class object, the base class copy constructor is called and derived class part will be sliced off.

```cpp

#include <iostream>

class Base
{
public:
    virtual void print() const
    {
        std::cout << "Base" << std::endl;
    }
};

class Derived : public Base
{
public:
    void print() const override
    {
        std::cout << "Derived" << std::endl;
    }
};

void print_val(Base v)
{
    v.print();
}

void print_ref(const Base& r)
{
    r.print();
}

int main()
{
    Base base;
    Derived derived;

    print_val(base); // => Base : OK
    print_val(derived); // => Base : NOT OKAY , Sliced!!

    print_ref(base); // => Base : OK as expected.
    print_ref(derived); // => derived : OK

    return 0;
}

```

## Item 21 : Don't try to return a reference when you must return an object.

```cpp
class Rational
{

friend
    const Rationsl operator*(const Rational& lhs, const Rational& rhs);
};

// Don't do
const Rational& operator*(const Rational& lhs, const RAtional& rhs)
{
    Rational result(lhs.n*rhs.n, lhs.d*rhs.d);
    return result;
}
// Where is your referrence to result go after function exits...?


// Just return object instead.
inline const Rational operator*(const Rational& lhs, const RAtional& rhs)
{
    return Rational(lhs.n*rhs.n, lhs.d*rhs.d);
}

```

It seems like an expensive operation, but in most cases compilers will apply RVO(Return Value Optimization).

An exmple to show RVO.
```cpp
#include <iostream>

class Student
{
public:
    Student() {std::cout << "ctr" << std::endl;}
    ~Student() {std::cout << "dtr" <<std::endl;}

    Student(const Student& rhs) { std::cout << "Copy ctd" << std::endl;}
    Student(Student&& rhs){ std::cout << "Move ctr" << std::endl;}

    Student& operator=(const Student&)
    {
        std::cout << "copy assignment" << std::endl;
        return *this;
    }

    Student& operator=(Student&& rhs)
    {
        std::cout << "Move assignment" << std::endl;
        return *this;
    }
};

Student example_rvo()
{
    return Student();
}

int main()
{
    Student A = example_rvo();
}

```

When you run this, you'd see only "ctr" and "dtr" thanks to RVO.

Refs : https://shaharmike.com/cpp/rvo/

