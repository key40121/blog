---
title: "Effective C++ Chapter 2"
date: 2023-07-09T04:55:58Z
draft: true
---

# chapter 2
## Constructors, Destructors, and Assignment Operators.

## Item 5 : Know what functions C++ silently writes and calls.
This item is all about default constructos and destructors.


### Default costructos and siignment operators.
If you don't declare them, compilers will declare their own versions.

```C++
class Emptry{};

// equals to

class Empty
{
public:
    Empty(){}
    Empty(const Empty& rhs){}
    ~Empty(){}
    Empty & operator=(const Empty&rhs){}
};

```

Ok. The problem arises when your object has a reference or pointer.(Don't know why it is not mentioned in the book.)

For example,
```C++
class Test
{
public:
    Test(int val)
    int* val;
    { this->val = new int(val); }
};

void bug_func(Test t) // copy constructor
{
    // nothing
}

```

When you call declare this class and call bug_func the program will crash because of  **shallow copy or member-wise ccopy**.

Basically, both default copy constructor and copy assignment do shallow copy not deep copy.
So, when you have a raw pointer, you need to implement a copy constructo with deep copy by yourself.

```C++
// copy constructor with shallo copy
Test(const Test &source)
    :val(source.val)
{
    std::cout << "shallow copy";
}

// copy constructor with deep copy
Test(const Test & source)
{
    val = new int;
    *val = *source.val;
    std::cout << "deep copy";
}
```

From c++14, we can use std::unique_ptr instead of raw pointer, so here I am showing an examp;e by using a class with unique_ptr.

```C++
#include <iostream>
#include <memory>

class A
{
private:
    std::unique_ptr<int[]> data;
    std::size_t size;
public:
    A(std::size_t size)
        :size(size), data(std::make_unique<int[]>(size))
    {}

    A& operator=(const A& other) //  user defined copy assignment
    {
        if (this != &other) // not a self-assignment
        {
            if (size != other.size) // resource cannot be reused
            {
                data.reset(new int[other.size]);
                size = other.size;
            }
            std::copy(&other.data[0], &other.data[0] + size, &data[0]);
        }
        return *this;
    }

    A(const A& other) // user defined copy constructor
    {
        if (this != &other)
        {
            if (size != other.size)
            {
                data.reset(new int[other.size]);
                size = other.size;
            }
            std::copy(&other.data[0], &other.data[0] + size, &data[0]);
        }
    }

    std::size_t get() const
    {
        return size;
    }
};

int main()
{
    A a1(1);
    A a2(a1);
    A a3 = a1;

    std::cout << "a1 : " << a1.get() << std::endl;
    std::cout << "a2 : " << a2.get() << std::endl;
    std::cout << "a3 : " << a3.get() << std::endl;

    return 0;
}

```


Refs :
https://medium.com/@seanoughton/c-copy-constructor-45df7ab5b047


## Item 6 : Explicitly disallow the use of compiler generated functions you do not want.

In this chapter, the book explains

From C++11, we can use a new form of function declaration which allows us to append the "=default" or "=delete" specifier to the end of a function declaration to declare that function as an explicitly defaulted function.

The book tries to declare constructors that should not be calles as private so that anyone who calls it would get an error at link-time.
However, since we can use "=delete" specifier from c++11, we don't need to follow the way the book does.
The most obvious example would be singleton where you have to delete all copy and move construtors and copy and move assignments.

```C++
class Singleton
{
private:
    Singleton() = default;
public:
    Singleton(Singleton & other) = delete; // copy constructor
    Singleton& operator=(const Singleton&) = delete; // copy assignment
    Singleton(Singleton&&) = delete; // move construcotr
    Singleton& operator=(Singleton && ) = delete; // move assignment

    static Singleton& getInstance()
    {
        static Singleton singleton;
        return singleton;
    }
    int data;
};

int main()
{
    Singleton::getInstance().data = 10;
    std::cout << Singleton::getInstance().data << std::endl;
    return 0;
}
```

## Item 7 : Declare destructors virtual in polymorphic base classes.
This item is all about just giving a polymorphic base class a virtual destructor, and classes not designed to be base classes or not designe to be used polymorphically should not declare virtual destructors.
This book dosen't deal with the problem that can be arise if we use raw pointer in factory method.
So I implemented a factory method with std::unique_ptr.

```C++
#include <iostream>
#include <memory>

class Product
{
public:
    virtual void display() const = 0;
    virtual ~Product() {}
};

class ConcreteProduct1 : public Product
{
public:
    void display() const override
    {
        std::cout << "Concrete product 1" << std::endl;
    }
};

class ConcreteProduct2 : public Product
{
public:
    void display() const override
    {
        std::cout << "Concrete product 2" << std::endl;
    }
};

std::unique_ptr<Product> createProduct(int item_num)
{
    if (item_num == 1)
    {
        return std::make_unique<ConcreteProduct1>();
    }
    else if (item_num == 2)
    {
        return std::make_unique<ConcreteProduct2>();
    }
}

int main()
{
    auto product1 = createProduct(1);
    auto product2 = createProduct(2);

    product1->display();
    product2->display();

    return 0;
}

```

## Item 8 : Prevent exceptions from leaving destructors.

Nothing to mention...
Just not make your destructor throw exception.

## Item 9 : Never call virtual functions during construction or destruction.

