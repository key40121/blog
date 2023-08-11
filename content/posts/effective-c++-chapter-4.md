+++
authors = ["Taichi Ichisawa"]
title = "Effective C++ chapter 4"
date = "2023-05-20"
description = "Effective C++ chapter 4"
tags = [
    "C++",
    "Effective C++",
]
categories = [
    "C++",
    "books",
]
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

class Month
{
public:
    static Month Jan(){ return Month(1); }

private:
    explicit Month(int m);

};

Data(Month::Mar(), 30, 1995);

```

## Item 19 : Treat class design as type design.

When you create a class, you really have to be careful with implementations.

***

## Item 20 : Prefer pass-by-reference-to-const to pass-by-value.

I think this item is now too popular enough that everyone uses.

```cpp

class Test() {};

bool something(Test test); // expensive operation

// Instead
bool something(const Test& test);

```

There is an explanation about **Slicing problem** as well.
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

***
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

***

## Item 22 : Declare data members private.
Just don't use data members as an interface.

Advantages
1. Affords fine-grained access control.
2. Offers class authors implementation flexibility.
3. Limit the access point to data members.

```cpp
template <typename T, typename U>
T& get(std::pair<T, U>& p)
{
    return p.first;
}

```

***

## Item 23 : Prefer non-member functions when type conversions should apply to all parameters.

```cpp

template <typename T>
class C
{
    T x;
    void A();
    void B();
    void C();
    void D()
    {
        A(); B(); C();
    }
}

D(T& classC)
{
    classC.A();
    classC.B();
    classC.C();
}

```
The question is which is better code.
There is a misunderstanding that the functions should be bundled together according to object-oriented principle.
It is no coreccct and object-oriented principles dictate that data should be as **encapsulated** as possible.

Get back to the question which is better, actually non-member function is better in terms of encapsulation because it won't increase the number of functions that can access the private parts of the class.


***When your function has multiple public member functions, you can have non-frinend non-member function to implement them***

***

## Item 24 : Declare non-member functions when type conversions should apply to all parameters.
...
***

## Item 25 : Consider support for a non-throwing swap.

The simplest swap implementation.
```cpp

template <typename T>
void swap(T& a, T& b)
{
    T temp(a);
    a = b;
    b = temp;
}

namespace swapswap
{

template <typename T>
class Widget
{
    T* x;
    void swap(Widget& rhs)
    {
        using std::swap;
        swap(this->x, rhs.x);
    }

}
}


```

***