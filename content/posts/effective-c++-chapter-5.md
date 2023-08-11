+++
authors = ["Taichi Ichisawa"]
title = "Effective C++ chapter 5"
date = "2023-06-10"
description = "Effective C++ chapter 5"
tags = [
    "C++",
    "Effective C++",
]
categories = [
    "C++",
    "books",
]
+++


# Chapter 5

***

## Item 26 : Postpone variable definitions as long as possible.

Initialization is always immportant to avoid undefined behavior.
However, sometimes it is important to consider when to declare and initialize variables.

Conside for loop

```cpp

int some = 0;
for (size_t i = 0; i < n; i++)
{
    some = /*some value dependent on i*/
}

// OR

for (size_t i = 0; i < n; i++)
{
    int some = 0;
    some = /*some value dependend on i*/
}
```

The first one incurs 1 constructr, 1 destructor, and n assignements.

The other one incurs n constructors and n destructors.

Efficiency depends on what class you deal with  but basically for classes where an assignment costs less than a constructor and destructor pair, the first one is more efficient.

***

## Item 27 : Minimize casting.

Use C++ style casting whenever possible. These are four types of casting in C++.

```cpp

// Old-style casts
// (T)expression
int a;

(double)a; // C-Style
double(a); // Function-sty;e

//const_cast<T>(expression)
const int* p;
int* q = const_cast<int*>p; // cast away the constness.

// dynamic_cast<T>(expression)
class Base
{
public:
    virtual ~Base(){}
};
class Derived : public Base {}

Derived* derived = dynamic_cast<derived*>(new Base()); // Downcasting.

// reinterpret_cast<T>(expression)
chat cp[] = { 0x89, 0xAB, 0xCD, 0xEF};
int* a = reinterpret_cast<int*>(cp);
std::cout << std::hex << *a << std::endl;

// static_cast<T>(expression)
double a{10.4};
int x = static_cast<int>a; // 10;

```

In general, using dynamic_cast is not a good practice.

If you encounter the situation where you have to use dynamic_cast, you should reconsider the design.


***

## Item 28 : Avoid returning "handles" to object internals.

Having a member function returining handles(references, pointers, or iterators) to object internals is not something you want to do in C++.

If you define a member function like this, it allows clients to modify object internals.

```cpp

struct RectData
{
    Point ulhc;
    Point lrhc;
}

int main()
{

class Rec
{
public:
    Point& upperLeft() const {return pData->ulhc;}
};

}

```

Even if you make it return const, it still has a problem that can cause **dangling pointer**.

```cpp
const int& f()
{
    std::vector<int> v = {1, 2, 3};
    return v.front();
}

```

***

## Item 29 : Strive for exception-safe code.
Exception-safe... very important.

The author claims that there are two requirements that an exception-safe code has to meets.
1. Leak no resources.
2. Don't allow data structures to become corrupted.



About the rule no.1 : Leak no resources.

=> Use RAII to avoid leak resouces.

```cpp
void something(cv::Mat& img)
{
    Lock ml(&mutex); // RAII
    // No matter what happens it automatically release the mutex.
}
```

Regarding rule no.2 : Don't allow data structures to become corrupted.

There are three guarantees that excception-sage functions can offer.
1. **the baseic gurantee**
Even if an exception is thrown, everything in the data program remains in a valid state.

2. **the strong gurantee**
If an exception is thrown, the state of the progmram wouldn't be changed. In other words, there are only two state possible, before the call or after the call with suceess.

3. **the nothrow gurantee**
 A function which never throw exceptions.

 e.g.
 ```cpp
 #include <new>

int* p = new(std::nothrow) int[10];
if (p == nullptr)
{
    std::cout << "Allocation failed". << std::endl;
}

 ```

Using **noexcept** specifiesr dosen't make sure the nothrow guarante.
If a function with noexecpt throws an exception, std::terminate will be called and it'd call std::abort.



***

## Item 30 : Understand the ins and outs of inlining.
The idea behind an inline function is to replace each call of that funciton with its code body.

There are cons and pros.

Pros :
1. If an inline function body is very short, the code generated for the function body may be smaller than the code generated for a function call.

Cons :
1. Overzeauous inlining can make programs too big.


```cpp

// Implicit way : inside header
template <typename T>
class Widget
{
private:
    T a;
public:
    T value() const
    {
        return a;
    }
}


// Explicit way
template <typename T>
inline const T& std::max(const T& a, const T& b)
{
    return a < b ? b :a ;
}

```

***

## Item 31 : Minimize compilation dependencies between files.

To define a class, information like data members, and base class are typically needed, because the size of the class in the stack has to be known by a compiler.
In other words, the size of pointer or reference are known by a compiler so you can play the "hide the object implementation behind a pointer" like Java.

pimpl idiom is the one that separate implementation and declaration of a class.

```cpp

class ExampleImpl;

class Example
{
public:
    Example();
    void func1();
    void func2();
private:
    std::unique_ptr<ExampleImpl> pimpl;
};

class Example::ExampleImpl
{
public:
    void func3();
    void func4();
};

void Example::func1()
{
    pimpl->func1(); // This way, we don't have to modify interfaces as well.
}

```