+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 5"
date = "2023-10-10"
description = "Effective Modern C++ chapter 5"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 5

***

## Item 23 : Understand std::move and std::forward.

std::move and std::forward do the same thing and they are just cast.
std::move always casts its argument to an rvalue, while std::forward performs cast only if a particular condition is met.


The implementation of std::move would be...
```cpp

// C++14
template<typename T>
std::remove_reference_t<T>&& move(T&& param) noexcept
{
    return static_cast<std::remove_reference_t<T>&&>(param);
}

```

std::forward is typically used when template takes a universal reference parameter and pass it to another function.
Universal referenve can mean both Rvalue reference and Lvalue reference, so...

```cpp

void func(const Widget& lvalArg);
void func(Widget&& rvalArg);

template<typename T>
void forward(T&& param)
{
    func(std::forward<T>(param));
}d

Widget w;

forward(w); // call with lvalue/
forward(std::move(w)) // call with rvalue

```



***

## Item 24 : Distinguish universal references from rvalue references.


This chapter is devoted to universal reference and the behavior of type deduction with T&&.

```cpp

template<typename T>
void f(T&& param); // param is universal reference

Widget&& var1  = Widget(); // rvalue referenve
auto&& var2 = var1; // universal reference

```


***

## Item 25 : Use std::move on rvalue references, std::forward on universal references.

When working on universal reference in your class, use std::forward.

```cpp

class Widget
{
public:
    template<typename T>
    void setName(T&& newName)
    {
        name = std::forward<T>(newName);
    }
};

```

If this is rvalue reference, we can use std::move

```cpp

class Widget
{
public:
    Widget(Widget&& rhs)
    : name(std::move(rhs.name)), p(std::move(rhs.p)) {}
private:
    std::string name;
    std::shared_ptr<DataStructure> p;
};

```

RVO : return value optimization.


***

## Item 26 : Avoid overloading on universal references.

Think about a class which has cope/move assignments, copy/move constructors, and std::string as a private member variable and pass it to a function with template universal reference.

```cpp

#include <iostream>
#include <string>
#include <list>
#include <chrono>
#include <set>

class MyClass
{
public:
    MyClass(std::string name) : name(name) { std::cout << "Ctr" << std::endl; }
    ~MyClass() { std::cout << "Dtr" << std::endl; }

    MyClass(const MyClass& other) : name(other.name)
    {
        std::cout << "Copy ctr" << std::endl;
    }

    MyClass& operator=(const MyClass& rhs)
    {
        if (this != &rhs)
        {
            name = rhs.name;
        }
        std::cout << "Copy assignment" << std::endl;
        return *this;
    }

    MyClass(MyClass&& other) noexcept
    {
        name = std::move(other.name);
        std::cout << "Move ctr" << std::endl;
    }
    MyClass& operator=(MyClass&& rhs)
    {
        if (this != &rhs)
        {
            name = std::move(rhs.name);
        }
        std::cout << "Move assignment" << std::endl;
        return *this;
    };

    // Less-than operator for ordering in std::multiset
    bool operator<(const MyClass& rhs) const
    {
        return name < rhs.name;
    }

private:
    std::string name;
};

std::multiset<MyClass> names;

template<typename T>
void logAndAdd(T&& name)
{
    auto now = std::chrono::system_clock::now();
    names.emplace(std::forward<T>(name));
}

```
What if we declare logAndAdd like this?

```cpp

void logAndAdd(const MyClass& name)
{
    auto now = std::chrono::system_clock::now();
    names.emplace(name);
}

int main()
{
    MyClass petName("Darla");

    logAndAdd(petName); // passing lvalue

    logAndAdd(MyClass("Patty Dog")); // rvalue

    return 0;
}

```

Obviously, we would end up calling copy constructor in the first call because name is an lvalue and it is copied into names.


In the second call, passing value is an rvalue but inside function name is an lvalue, so copy constructor will be called.

Both results shoud be

Ctr
Copy ctr
Dtr
Dtr


```cpp

template<typename T>
void logAndAdd(T&& name)
{
    auto now = std::chrono::system_clock::now();
    names.emplace(std::forward<T>(name));
}

```
This way, if we pass rvalue it will call move constructor; thus more efficiency.

```cpp

logAndAdd(MyClass("Patty Dog"));

```
This would be
Ctr
Move Ctr
Dtr
Dtr


If we want to have overloading functions which accept int as an argument, we can declare it.
But what if a client call this function with "short".

```cpp

void logAndAdd(int idx) // new overload
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(nameFromIdx(idx));
}

// wrong calling
short var = 10;
void logAndAdd(var); // this will instantiate template version func. thus causing an error.

```

In general, combining overloading and universal references is almost always a bad idea.

If universal references get into a constructor, the situation will get worse.

```cpp

class Person
{
public:
    template<typename T> // perfect forwardinf ctr
    explicit Person(T&& n)
    : name(std::forward<T>(n)) {}

    explicit Person(int idx); // int ctr

    Person(const Person& rhs); // copy constructor (compiler-generated)

    Person(Person&& rhs) // move constructor (compiler-generated)

private:
    std::string name;
};

```

and if we call

```
Person p("Nancy);

auto cloneOfP(p); // hoping to call copy construcotr but this won't compile.
```

This one will call the perfect-forwarding consturoctr. Then it will try to initialize Person's std::string private data member with a Person(p).
So it will fail.

***

## Item 27 : Familiiarize yourself with alternatives to overloading on universal references.

This chapter introduces alternative approaches of item 26.

### Tag dispatch

We can route client's calls by using std::true_type and std::false_type.



std::weak_ptr is a kind of shared_ptr that dosen't affect an object's reference count.
std::weak_ptr is typically created from std::shared_ptr like this.

```cpp

std::multiset<std::string> names;

template<typename T>
void logAndAddImpl(T&& name, std::false_type);
void logAndAddImpl(int idx, std::true_type);

std::string nameFromIdx(int idx)
{
    std::string temp = "Temp";
    return temp;
}

template<typename T>
void logAndAdd(T&& name)
{
    logAndAddImpl(std::forward<T>(name), std::is_integral<std::remove_reference_t<T>>());
}

template<typename T>
void logAndAddImpl(T&& name, std::false_type)
{
    std::cout << "emplace : std::false_type" << std::endl;
    names.emplace(std::forward<T>(name));
}

void logAndAddImpl(int idx, std::true_type)
{
    std::cout << "emplace : std::true_type" << std::endl;
    logAndAdd(nameFromIdx(idx));
}


int main()
{
    logAndAdd("aaa");
    logAndAdd(10);
}


```

emplace : std::false_type
emplace : std::true_type
emplace : std::false_type

You can tell that when passing int(10) it will call void logAndAddImpl(int idx, std::true_type) properly.
Even when you pass other integral types like size_t, short, .... you will get the same result.

### Constraining templates that take universal references

In terms of a perfect-forwarding constructor, tag dispatch design can not be a solution.

Need to use std::enable_if_t (from C++14) to make a constructor to choose which one to call.

```cpp

class Person
{
public:
    template<typename T, typename = std::enable_if_t<!std::is_base_of<Person, std::decay_t<T>>::value
                                                    &&
                                                    !std::is_integral<std::remove_reference_t<T>>::value>>
    explicit Person(T&& n) : name(std::forward<T>(n))
    {}
    // only if T is not person (derived person class) nor integral type, this templatized constructor will be called.

    explicit Person(int idx)
        : name(nameFromIdx(idx)) {}

private:
    std::string name;

};

```

The design is called SFINAE, and I will explain it in another article.

### Trade-offs

The drawback of using template is always difficulty of error messages. Even if you use g++, clang, ... always very long and hard to read.
But by using static_assert, you can make your error messages easier to comprehend.

```cpp

class Person
{
public:
    template<typename T, typename = std::enable_if_t<!std::is_base_of<Person, std::decay_t<T>>::value
                                                    &&
                                                    !std::is_integral<std::remove_reference_t<T>>::value>>
    explicit Person(T&& n) : name(std::forward<T>(n))
    {
        static_assert( // newly added line
            std::is_constructible<std::string, T>::value,
            "Parameter n can't be used to construct a std::string"
        );
    }

private:
    std::string name;
};


int main()
{
    Person p(u"Konrad Zuse");
}

```

It will create an error message like this (this is g++ 11.4.0)
```bash

error: static assertion failed: Parameter n can't be used to construct a std::string
   14 |             std::is_constructible<std::string, T>::value,

```


***


## Item 28 : Understand reference collapsing.

```cpp

template<typename T>
void func(T&& param);

```

The reason why universal references can have an ability of encoding lvalue/rvalue references is because there is one rule called "reference collapsing" which a compiler must follow.

***
If either reference is an lvalue reference, the result is an lvalue reference.
Otherwise (i.e., if both are rvalue references) the result is an rvalue reference.
***

So let's say we pass both lvalue and rvalue to the func, what would happen to template instantiation?

```cpp
class Widget{};
Widget widgetFactory();

Widget w; // lvalue

func(w); // func with lvalue; T deduces to be Widget&

func(widgetFactory()); // func with rvalue; T deduced to be Widget

```

It should look like this

```cpp
void func(Widget& && param); // lvalue

void func(Widget&& && param); // rvalue

```
Thus, following the rule, they would look like this

```cpp

void func(Widget& param);

void func(Widget&& param);

```

What about the implementation of std::forward?
It can be implemented like this

```cpp

template<typename T>
void f(T&& fParam)
{
    someFunc(std::forward<T>(fParam));
}

template<typename T>
T&& forward(std::remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}

```

If the argument passed to f is an lvalue of type Widget, T will be deduced as Widget&.

```cpp
Widget& && forward(std::remove_reference_t<Widget&>& param)
{
    return static_cast<Widget& &&>(param);
}

// reference collapsing
Widget& forward(Widget& param)
{
    return static_cast<Widget&>(param);
}
```

If the argument passed to f is an rvalue of type Widget,

```cpp

Widget&& forward(std::remove_reference_t<Widget>& param)
{
    return static_cast<Widget&&>(param);
}

```

***

## Item 29 : Assume that move operations are not present, not cheap, and not used.

Move semantic is not efficient and useful when

- There is no move operator.
- Using a continer like std::array which dosen't store an element in a heap memory.
(std::string has an optimization called small string optimiazation whicch enables code to store "small" strings in a stack, not heap)
- move operators cannot be used. Especially when using a method that gurantees strong exception safe.
e.g. std::vector::push_back



***

## Item 30 : Familiarize yourself with perfect forwarding failure cases.

Perfect forwarding can fail in some circumstances.

Given this function
```cpp

template<typename T>
void fwd(T&& param)
{
    f(std::forward<T>(param));
}

```

### Braced initializers

```cpp

void f(const std::vector<int>& v);
f({1, 2, 3}); // OK implicit conversion will occur.

fwd({1, 2, 3}); // Error

```

Because std::inirialier_list is non-deduced context, T will mpt ne deduced as std::initialize_list; thus, it will fail.

Declaring it with auto, and passing it to fwd is fine.

```cpp

auto il = {1, 2, 3}; // std::initializer_list
fwd(il);
```

### 0 or NULL
0 and NULL will be deduced as an integral type instead of a poiter type.

### Declaration-only integral static const data members.

static const data members cannot be passed as universal reference.

```cpp
// no definition
struct Widget
{
    static const std::size_t MinVals = 28;
}

std::vector<int> widgetData;
widgetData.reserve(Widget::MinVals); // compilers make it work even though there is no definition.
```

This would fail at link-time.
```cpp
fwd(Widget::MinVals); // error! cannot be linked.
```

### Overloaded function names and template names

```cpp
int processVal(int val);
int processVal(int val, int priority);

void f(int (*fp)(int));

fwd(processVal); // Error, it cannot know which one to invoke.

```

### Bitfields

You cannot pass arbitrary bits to fwd because there's no way to bind a reference to arbitrary bits.
The smallest thing C++ can point to is a char(uint8_t).

```cpp

struct IPv4Header
{
    std::uint32_t version:4,
                IHL:4,
                DSCP:6,
                ECN:2,
                totalLength:16;
}

void f(std::size_t sz);

IPv4Header h;
f(h.totalLength); // fine

fwd(h.totalLength); // error
```

***
