+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 3"
date = "2023-10-08"
description = "Effective Modern C++ chapter 3"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 3

**Moving to Modern C++**

This chapter explains notations like constexper, using, .... which are very important to understand when it comes to efficiency.

***

## Item 7 : Distinguish between () and {} when creating objects.
This chapter explains syntax choices for object initialization from C++11.
e.g.

```cpp

int  x(0); // initializer is in parentheses

int y = 0; // initializer follows "="

int z{0}; // initializer is in braces.

```

Actually, C++11 introduces a new initialization syntax to cover all scenarios(e.g. initialization of STL container). "Braced initialization" is introduced and is can be used as follows.

```cpp

#include <iostream>
#include <vector>

int main()
{
    // brace initialization
    std::vector<int> v{1, 2, 3}; // v's initialied with variables 1, 2, 3.

    // default initialization for non-static value members.
    class hoge
    {
    private:
        uint32_t x{0};
        uint32_t y = 0l;
        // uint32_t z(0) // error, cannnot be used.
    };

    return 0;
}

```

Also uncopyable object such as std::thread, std::atomic, ... can be initialized with parentheses.

```cpp
std::atomic<int> a{0};
std::atomic<int> b (0);
std::atomic<int> c = 0; // not ok
```

Now based on these examples, we can finally understand how efficient "Braced initialization" is because only it can be used everywhere.
In addition, this new syntax is useful to prevent C++ from implementing "implicit narrowing conversions" among built-in types. Let's take a look at an example here.


```cpp
double x, y, z;

int sum1{x + y + z}; // error

int sum2(x + y + z); // okay

int sum3 = x + y + z; // okay
```


Another situation where "Braced initialization" demonstrate its strength is when encountering C++'s "most vexing parse".

Most vexing parse refers to an situation where C++ grammer cannnot distinguish between the creation of an object parameter and specification of a funciont's type.

```cpp

Widget w1(10); // ctr with ().

Widget w2(); // can be interpreted as both function declaration and object creation.
            // C++ treats it as function declaration

```

e.g.
```cpp

class Widget
{
public:
    Widget(int num_ = 5) : num(num_)
    {
        std::cout << "num = " << this->num << std::endl;
    }

private:
    int num;

};

int main()
{
    Widget w1(10);

    //Widget w2(); // errpr

    Widget w3{};

    return 0;
}

// g++ 11.4.0
/*
item7.cpp:19:14: warning: empty parentheses were disambiguated as a function declaration [-Wvexing-parse]
   19 |     Widget w2();
      |
*/

```

## Item 8 : Prefer nullptr to 0 and NULL.

Use nullptr. NULL is not a pointer.

```cpp
void f(int);
void f(bool);
void f(void*);

f(0); // calls f(int)
f(NULL); // might not compile but typically f(int)!! not f(void*)


```

***

## Item 9 : Prefer alias declarations to typedefs.

Basically typedef is old, which is C++98. On the orher hand alias declaration is newer version which was introduced from C++11.

Alias declartion is newer, which means there was a reason that it was introduced.

```cpp

// these two do exactly the same thing

typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;

using UPtrMapSS = std::unique_ptr<std::unoredered_map<std::string, std::string>>;

```

What's good about alias declaration is that it can be templatized, while typedefs cannot.

```cpp
template <typename T>
using myAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;

// Then you can do
template <typename T>
class Widget
{
private:
    MyAllocList<T> list;
};

```

This is new, in effective C++ what we did was this.
Because MyAllocList<T>::type is a dependent type, we needed to add typename.

```cpp

template <typename T>
struct MyAllocList
{
    typedef std::list<T, MyAlloc<T>> type;
};

template <typename T>
class Widget
{
private:
    typename MyAllocList<T>::type list;
};

```

***


## Item 10 : Prefer scoped enums to unscoped enums.

Use scoped enum so that you don't pollute namespace.
C++98-style enums, which is unscoped enum, is not good in terms of namespace.

```cpp

enum Color // unscoped enum
{
    black,
    white,
    red
};

auto white = false; // ERROR, because you can refer to white.

```

Instead use C++11 snums, which is scoped enums.

```cpp

enum class Color
{
    black,
    white,
    red
};

auto white = false; // fine because enum's white is different one

Color c = Color::white; // fine
auto c = Color::white; // also fine

```

Also, unscoped enums inplicitly convert its element to integral types.
```cpp

enum Color
{
    black,
    white,
    red
};

// you can do this
Color c = red;
if (c < 2) // This implicit conversion is valid.
{
    std::cout << "old";
}

```

If you use scoped enum, implicit conversion never occurs.

```cpp

enum class Color
{
    black,
    white,
    red
};

Color c = Color::red;

if (c < 2) .... // not valid anymore


```

### Forward declaration.
C++98 supports only enum (unscoped enum) definitions, enum declarations are not allowed because compiler can choose the undelying type of enum.
In other words, the size is not known when declaring enum; thus, it was not allowed to only declare enum.

But appearently, the inability to forward-declare enums has drawbacks. It increases compilation dependencies.

The reason that C++11's enums get away with forward declarations is that the underlying type is known.

```cpp

enum class Status; // underlying type is int.

enum class Status: std::uint32_t; // can be overriden.


```

Also when you work with std::tuple, you'd definitely use std::get.

```cpp

using UserInfo = std::tuple<std::string, std::string, std::size_t>;

UserInfo uInfo;
...

auto val = std::get<1>(uInfo); // get value of field 1.
```

The verbosity can be reduced by writing a function that takes an enumerator and returns its corresponding underlying type value.

```cpp

template <typename E>
constexpr std::underlying_type_t<E> toUType(E enumerator) noexcept
{
    return static_cast<std::underlying_type_t<E>>(enumerator);
}

auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);

```


***

## Item 11 : Prefer deleted functions to private undefined ones.

Use "delete" when you don't need some of special member functions.

```cpp

// e.g. singleton
Singleton(const Singleton&) = delete;
Singleton& operator=(const Singleton&) = delete;
Singleton(Singleton&&) = delete;
Singleotn& operator=(Singleton&&) = delete;

```

By convention, deleted functions are declared public, not private because when they are declared private and client code tried to use a deleteed pricate dunction,
some compilers might complain only about the function being private.


***

## Item 12 : Declare overriding functions override.

This item is explained in effecitve C++ too.

Use override so that you can see errors that compilers elicit.

```cpp

class Base
{
public:
    virtual void mf1() const;
    virtual void mf2(int x);
    virtual void mf3()&;
    void mf4() const;
};

class Derived: public Base
{
public:
    virtual void mf1(); // not override
    virtual void mf2(unsigned int x); // not override
    virtual void mf3() &&; // not override
    void mf4() const; // not override
};

```

Derived member functions are intended to override member functions in Base class, but they are not because each of them are slightly different from Base class's.
The important thing is that compiler won't say anything about this.
So, if you intend to override member funcitons in your derived class, use override.

```cpp


class Derived: public Base
{
public:
    virtual void mf1() const override;
    virtual void mf2(int x) override;
    virtual void mf3() & override;
    // non virtual mf4 cannnot be overriden.
};

```

***

## Item 13 : Prefer const_iterators to iterators.

Basically if you use C++14/17/20, use const_iterators whenever possible.

```cpp

template <typename C, typename V>
void findAndInsert(C& container, const V& targetVal, const V& insertVal)
{
    using std::cbeing;
    using std::cend;

    auto it = std::find(cbegin(contaienr), cend(container), targetVal);

    container.insert(it, insertVal);
}

```

***

## Item 14 : Declare functions noexcept if they won't emit exceptions.

noexcept functions are more optimizable than non-except functions.

If you don't declare noexcept on your own move constructor, std container will copy each elements even if you are using std::move.

```cpp

Widget w;
std::vector<Widget> vec;

vec.push_back(std::move(w)); // if move construcotr is noexcept, it is actually moved.
// std::move_if_noexcept()

```
***

## Item 15 : Use constexpr whenever possible.

For object,

constexpr => compile time constness(known at compile time)
const => runtime constness.

An object with constexpr may be placed in read-only memory; thus, constexpr is very important especially for develpers of embedded systems.

```cpp

int sz; // non-constexpr

constexpr auto arraySize = 10; // fine
std::array<int, arraySize> data; // fine

```


If constexpr is applied to functions, things are get complicated.
If constexpr functions are called with compile-time constants, they produce compile-time constants. If they're called with values not known until runtime, they produce runtime values.

Assuming we are trying to write original power function, and use the return value for std::array.

```cpp

constexpr int pow(int base, int exp) noexcept
{

}

constexpr auto numConds = 5;
std::array<int, pow(3, numConds)> ret; // This is okay since pow(3, numConds) is compile-time constants.

```

If base and/or exp are not compile-time constants, pow's resuklt will be computed at runtime.

```cpp

auto base = readFromDB("base"); // runtime
auto exp = readFromDB("exponent"); // runtime

auto baseToExp = pow(base, exp); // call pow function at runtime.

```

You can also use constexpr in your class and make a lot of things migrate to compile time.

```cpp

class Point
{
public:
    constexpr Point(double xVal = 0, double yVal = 0) noexcept
        : x(xVal), y(yVal) {}

    constexpr double xValue() const noexcept { return x; }
    constexpr double yValue() const noexcept { return y; }

    constexpr setX(double newX) noexcept { x = newX; }
    constexpr setY(double newY) noexcept { y = newY; }

private:
    double x, y;

};


```

***

## Item 16 : Make const member functions thread safe.

Seemingly this chapter is related to constness, but actually this chapters is about the distinction between std::mutex and std::atomic.

Imagine, you have a function which returns a cached value and only when the value is not cached it calculates the value.

It would look like this.

```cpp

class Polynomial
{
public:
    using RootsType = std::vector<double>;

    RootsType roots() const
    {
        if(!rootsAreValid)
        {
            /*code*/
            rootsAreValid = true;
        }
        return rootsVals;
    }

private:
    mutable bool rootsAreValid{false}; // cach
    mutable RootsType rootVals{};
};

```

Seems fine.
Now imagine if this roots() is called from more than two threads.
We will have a race condition.

Simple solution would be using std::mutex.

```cpp

class Polynomial
{
public:
    using RootsType = std::vector<double>;

    RootsType roots() const
    {
        std::lock_guard<std::mutex> g(m); // RAII lock and unlock

        if(!rootsAreValid)
        {
            /*code*/
            rootsAreValid = true;
        }
        return rootsVals;
    }

private:
    mutable std::mutex m;
    mutable bool rootsAreValid{false}; // cach
    mutable RootsType rootVals{};
};

```
Disadvantage of this implementation is that it cannot be copyable anymore because of the existance of std::mutex.

Like std::mutex, you can use std::atomic. In general, std::atomic variables are less expensive than mutex acquisition and release.

```cpp

class Polynomial
{
public:
    using RootsType = std::vector<double>;

    RootsType roots() const
    {

        if(cacheValid)
            return cachedValue;
        else
        {
            /* code */

            cachedValue = val1 + val2; // NOT OKAY
            cacheValid = true; // NOT OKAY
            return cachedValue;
        }
    }

private:
    mutable std::atomic<bool> cacheValid{false};
    mutable std::atomic<RootsType> cachedValue;
};

```

This looks fine, but not good. Two threads can see cacheValid as false, thus both will carry out the same expensive computation simlutaneously.

Even if we reverse the order of the assignments ot cachedValue and CacheValid, it won't eliminate the race condition.

For a single variable or memory location requiring synchronization, use of std::atomic is adequate, but once you get to two or more variables, you need std::mutex.

The lesson is

single variable => std::atomic
scope(multiple variables) => std::mutex

***
## Item 17 : Understand special member function generation.

Special member functions are the default constructor, the destructor, the copy constructor, the copy assignment operator,
the move construcotr, and the move assignment operator.

They are generated only if they're needed.
Also they are implicitly public and inline.


**Rule of Three**
The rule of three states that if you declare any of a copy constructor, copy assigment, or destructor, you should declare all three, which was highligtened in effective C++.
Because a copy operation almost always performs memory(resource) management, delaring all three is very important.

e.g. shallow copy and deep copy are different notion.
```cpp

// copy constructor with shallow copy
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

Regarding move operations, they are generated only if these three things are true:
- No copy operations are declared in the class. (If there is user-defined copy operations, C++ compilers assume that there is resoure management involved.)
- No move operations are declared in the class. (C++ cannot generate only one of them)
- No destructor is declared in the class. (Existance of destructor means it involves resource management.)

