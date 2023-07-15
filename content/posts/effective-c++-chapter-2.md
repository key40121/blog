---
title: "Effective C++ Chapter 2"
date: 2023-06-20T04:55:58Z
draft: true
---

# chapter 2
## Constructors, Destructors, and Assignment Operators.
**Keep in mind**
Move semantics were introduced from C++11; therefore, there is no statements of move semantics at all in this book.

## Item 5 : Know what functions C++ silently writes and calls.
This item is all about default constructos and destructors.
so...

Let me explain the important rules around ctr and dtr.
1. Rule of three
If you need to define any of dtr, copy ctr or copy assignment, then you have to declare all of them.

2. Rule of five
If you need to define any of the five, then you probably need to define or delete all five.

3. Rule of zero
You should prefer the case where no special member functions need to be defined.

Refs : https://rules.sonarsource.com/cpp/RSPEC-3624/

### Default costructos and assignment operators.
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

Let's say...
```C++
class Base
{
public:
    ~Base() = default;
    virtual void f() = 0;
};

class VBase
{
public:
    virtual ~VBase() = default;
    virtual void f() = 0;
};

class Derived : public Base
{
public:
    void f() override {}
};

class VDerived : public VBase
{
public:
    void f() override {}
};

int main()
{
    std::unique_ptr<Base> b = std::make_unique<Derived>(); // Dangerous!! Derived::~Derived() won't be called
    std::unique_ptr<VBase> vb = std::make_unique<VDerived>(); // Ok
}

```

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

It says that you have to distinguish whether derived class or base class you are dealing with.
Thus, don't call virtual functions during construction or destruction because you cannot get the result you expected because of base constructor and destructor.

## Item 10 : Have assignment operators return a reference to *this.

Assigment has to return a reference to its left-hand argument as a convention.
```C++

class Widget
{
public:
    Widget& operator=(const Widget & rhs)
    {
        ...

        return *this;
    }
};

```
Well, this is just a convetion so we actually can violate this convention from compiler's point of view.

## Item 11 : Handle assignment to self in operator=
When you use copy assignment, you have to be careful with self assignment in addition to shallow copy. Let's say we have class with raw pointer, and if it is assigned with the same instance, it should not delete the resource it has.

```C++
class Widget
{
private:
    Bitmap *pb;

};

// This is not okay
Widget& Widget::operator=(const Widget & rhs)
{
    if (this == &rhs)
        return *this;
    delete pb;
    pb = new Bitmap(*rhs.pb); // might throw an exception
    return *this;
}

// This should be exception safe.
Widget &  Widget::operator=(const Widget& rhs)
{
    Bitmap *pOrig = pb; // rememver original pb
    pb = new Bitmap(*rhs.pb); // point pb to a copy of rhs's bitmap
    delete pOrig;

    return *this;
}

```

## Item 12 : Copy all parts of an object.
Only copy constructr and copy assignment operator can copy objects; hence, they should be called copying functions...Okay.

When you declare copying functions, you have to be very careful because compiler

I feel like I've encounterd the same theme already, which is shallow copy or deep copy.  Basically, as for this item, what the book tried to say is that you have to copy all objects.

If I sumarilze all things you have to take care when you deal with copy constuctor and copy assignment are
1. Copy all of an object's data members.
2. Be careful to implement copy functionality when there is a raw pointer.

So if there is no raw pointers, this test code I created below is fine at all.



```C++
class Customer
{
public:
    Customer(std::string name)
        :name(name) {}

    Customer(const Customer& rhs)
        :name(rhs.name)
    {
        std::cout << "copy constructor : name " << name << std::endl;
    }

    Customer& operator=(const Customer & rhs)
    {
        std::cout << "copy assignment operaotr : name " << name << std::endl;
        this->name = rhs.name;
        return *this;
    }
private:
    std::string name;
};

int main()
{
    Customer c1{"aaaa"}, c2{"bbbb"};

    Customer c3(c1);
    c3 = c2;
    return 0;
}

```

When you deal with inheritance.
```C++
class PriorityCustomer : public Customer
{
public:
    PriorityCustomer(int priority, std::string name)
        :priority(priority), Customer(name){}
    PriorityCustomer(const PriorityCustomer & rhs)
        :Customer(rhs), priority(rhs.priority)
    {}
    PriorityCustomer& operator=(const PriorityCustomer & rhs)
    {
        Customer::operator=(rhs);
        priority = rhs.priority;
        return *this;
    }
private:
    int priority;
};
```

## Memo : If you want to implement move constructor and move assignment operator in general
Too long lol
Well if I just made it have std::vector not a pointer of std::vector, it wouldn't be troublesome.
```C++
#include <iostream>
#include <vector>
#include <memory>

struct Person
{
    std::string name;
    int num;
};

class PersonList
{
private:
    std::unique_ptr<std::vector<Person>> personList;
public:
    PersonList() : personList(std::make_unique<std::vector<Person>>()){}
    ~PersonList() = default;

    PersonList(const PersonList& rhs) // copy constructor
        : personList(std::make_unique<std::vector<Person>>())
        {
            personList->insert(personList->end(), rhs.personList->begin(), rhs.personList->end());
        }
    PersonList& operator=(const PersonList & rhs) // copy assignment operator
    {
        if (&rhs != this)
        {
            personList->clear();
            personList->insert(personList->end(), rhs.personList->begin(), rhs.personList->end());
        }
        return *this;
    }
    PersonList(PersonList &&rhs) noexcept // Move constructor
        :personList(nullptr)
    {
        personList = std::move(rhs.personList);
        // rhs.personList = nullptr; No need if you use unique_ptr
    }
    PersonList & operator=(PersonList && rhs) noexcept // Move assignment operator
    {
        if (&rhs != this)
        {
            personList = std::move(rhs.personList);
            // rhs.personList = nullptr; No need if you use unique_ptr
        }
        return *this;
    }
};

int main()
{
    PersonList p1, p3;

    p1 = std::move(p3);
    PersonList p2(std::move(p3));
    return 0;
}


```
About move semantics when you use.
Refs : https://en.cppreference.com/w/cpp/language/noexcept_spec