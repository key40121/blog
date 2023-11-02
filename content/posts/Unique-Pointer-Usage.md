+++
authors = ["Taichi Ichisawa"]
title = "Smart Pointer"
date = "2023-10-4"
description = "Sample"
tags = [
    "C++",
]
categories = [
    "C++",
]
+++

# Basics

Smart pointers are wrappers around raw classic pointers with RAII mechanism, so smart pointers automatically avoid many of raw pointer's pitfull.

There are four smart pointers in C+11: std::auto_ptr, std::unique_ptr, std::shared_ptr, and std::weak_ptr.
But std::auto_ptr is a legacy API that was invented when there was no move semantics, so I will not use it here.

In this article, I will not examine the detailed mechanism of unique_ptr.
Instead, I will list up actual implementation patterns that you would see out there.

***

## Basics
```cpp

#include <iostream>
#include <memory>
#include <vector>

int main()
{
    // use make_unique (from C++14)
    // always should use make_unique helper function.
    // auto can be used because it dosen't call a constructor of int but of unique_ptr<int>.
    auto ptr = std::make_unique<int>(1);

    std::cout << "ptr : " << *ptr << std::endl;

    // e.g.
    auto upv = std::make_unique<std::vector<int>>(10, 20);

    // should be avoided because even though ptr2 would automatically be deleted but it uses new, and it is confusing.
    std::unique_ptr<int> ptr2 {new int{1}};

    std::cout << "ptr2 : " << *ptr2 << std::endl;

    // move
    auto ptr3 = std::make_unique<int>(100);
    auto ptr4 = std::move(ptr3);

    std::cout << "ptr4 : " << *ptr4 << std::endl;

    return 0;
}

```

***

## Factory method
Use std::make_unique to return a unique pointer instead of a raw pointer.

```cpp
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

***

## Class

If you want your class to have a smart pointer as a member variable, this implementation can be one of examples.

```cpp
class Widget
{
private:
    std::unique_ptr<int> num;
public:
    Widget(std::unique_ptr<int> num)
        : num(std::move(num)) {}

    void setNum(std::unique_ptr<int> num)
    {
        this->num = std::move(num);
    }

    void printNum()
    {
        std::cout << *num << std::endl;
    }
};

int main()
{
    auto ptr = std::make_unique<int>(100);

    Widget widget(std::move(ptr));
    widget.printNum();

    return 0;
}
```

The question is how to pass unique_ptr to constructr.
There are four ways.

(1) By value.
```cpp
Widget(std::unique_ptr<int> num)
    :num(std::move(num)){}

// initialization
auto ptr = std::make_unique<int>(100);
Widget widget(std::move(ptr));
```

To take a unique pointer by value means that you are transferring ownership of the pointer to your object.
After widget is constructed, num is guranteed to be empty.

When we pass std::move(ptr) which is int&& r-value reference, C++ will automatically construct a temporary for us.
It creates std::unique_ptr<int> from int&& and it goes to function argument num.

(2) By r-value reference.
```cpp
Widget(std::unique_ptr<int>&& num)
    :num(std::move(num)) {}

// initialization
auto ptr = std::make_unique<int>(100);
Widget widget(std::move(ptr));

```

This one dosen't ensure that num is actually moved by looking at declaration.
It totally depends on the implementation(Here it is removed because we pass std::move(num) to num).

Reference : https://stackoverflow.com/questions/8114276/how-do-i-pass-a-unique-ptr-argument-to-a-constructor-or-a-function

