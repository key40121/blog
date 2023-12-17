+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 4"
date = "2023-10-09"
description = "Effective Modern C++ chapter 4"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 4

***

## Item 18 : Use std::unique_ptr for exclusive-ownership resource management.

std::unique_ptr accepts a custome deleter.

This is an example of factory method using std::unique_ptr.

```cpp

#include <iostream>
#include <memory>

class Investment
{
public:
    Investment() { std::cout << "ctr : investment" << std::endl; }

    virtual std::unique_ptr<Investment> create()
    {
        return std::make_unique<Investment>();
    }
    virtual ~Investment() { std::cout << "dtr : investment" << std::endl; };
};

class Stock: public Investment
{
public:
    Stock() { std::cout << "ctr : Stock" << std::endl; }
    ~Stock() { std::cout << "dtr : Stock" << std::endl; }
    std::unique_ptr<Investment> create() override
    {
        return std::make_unique<Stock>();
    }
};

class Bond: public Investment
{
public:
    Bond() { std::cout << "ctr : Bond" << std::endl; }
    ~Bond() { std::cout << "dtr : Bond" << std::endl; }
    std::unique_ptr<Investment> create() override
    {
        return std::make_unique<Bond>();
    }
};

class RealEstate: public Investment
{
public:
    RealEstate() { std::cout << "ctr : RealEstate" << std::endl; }
    std::unique_ptr<Investment> create() override
    {
        return std::make_unique<RealEstate>();
    }
};

auto makeInvestment(const std::string& name)
{
    auto delInvmt = [](Investment* pInvestment)
                    {
                        std::cout << "log"<< std::endl;
                        delete pInvestment;
                    };
    std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);

    if (name == "Stock")
    {
        pInv.reset(new Stock());
    }
    else if (name == "Bond")
    {
        pInv.reset(new Bond());
    }
    else if (name == "RealEstate")
    {
        pInv.reset(new RealEstate());
    }
    else
    {
        pInv.reset(new Investment());
    }
    return pInv;
}

int main()
{
    auto ptr = makeInvestment("Bond");
    return 0;
}


```

***

## Item 19 : Use std::shared_ptr for shared-ownership resource management.

std::shared_ptr is a raw pointer with garbage collection(like you see in Java).
Unlike std::unique_ptr, it has an extra pointer that points to an extra information which is called control block.
Control block contains Reference Count, Weak Count, and other data (e.g. custom deleter, allocator, etc.)

An object's control block is set up by the function creating the first std::shared_ptr to the object.



```cpp

auto pw = new Widget;

std::shared_ptr<Widget> spw1(pw, loggingDel); // create control block for *pw
std::shared_ptr<Widget> spw2(pw, loggingDel); // create 2nd control block!!!

// Do this
std::shared_ptr<Widget> spw1(new Widget, loggingDel);
std::shared_ptr<Widget> spw2(spw1);



```

***

## Item 20 : Use std::weak_ptr for std::shared_ptr like pointers that can dangle.

std::weak_ptr is a kind of shared_ptr that dosen't affect an object's reference count.
std::weak_ptr is typically created from std::shared_ptr like this.

```cpp

auto spw = std::make_shared<Widget>();

std::weak_ptr<Widget> wpw(spw);

spw = nullptr;


```

The book introduces two use cases; caches and observers.
There was no code for observers so I created a simple observer pattern.

```cpp

#include <iostream>
#include <memory>
#include <forward_list>
#include <functional>

class Observer : public std::enable_shared_from_this<Observer>
{
public:
    virtual void update() = 0;
};

class ConcreteObserver : public Observer
{
public:
    void update() override
    {
        std::cout << "ConcreteObserver notified" << std::endl;
    }
};

class Subject
{
private:
    std::vector<std::weak_ptr<Observer>> m_observers;

public:
    void addObserver(const std::shared_ptr<Observer>& observer)
    {
        m_observers.push_back(observer);
    }

    void notifyObservers()
    {
        for (auto weakObserver : m_observers)
        {
            if (const auto& observer = weakObserver.lock())
            {
                observer->update();
            }
            else
            {
                std::cout << "observer no longer available" << std::endl;
            }
        }
    }

};

int main()
{
    Subject subject;
    std::shared_ptr<ConcreteObserver> observer1 = std::make_shared<ConcreteObserver>();
    std::shared_ptr<ConcreteObserver> observer2 = std::make_shared<ConcreteObserver>();

    // Add observers to the subject
    subject.addObserver(observer1);
    subject.addObserver(observer2);

    subject.notifyObservers();

    observer1.reset();
    observer2.reset();

    subject.notifyObservers();

}

```

***

## Item 21 : Prefer std::make_unique and std::make_shared to direct use of new.

The implementation of make_unique is quite simple.
```cpp

template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}

```

There are some reasons to prefer make_unique and make_shared.

Firstly, the versions using new repeat the type being created, but the make functions don't.
Repeating types is not clear, and you don't want to write the same type again and again.


```cpp

auto upw1(std::make_unique<Widget>());
std::unique_ptr<Widget> upw2(new Widget);

auto spw1(std::make_shared<Widget>());
std::shared_ptr<Widget> spw2(new Widget);

```

The second reason to prefer make functions has to do with exception safety.

```cpp
void processWidget(std::shared_ptr<Widget>(new Widget), computePriority()); // potential resource leak

```
This potential leak issue is because C++ dosen't define the order of functions to be executed in arguments. (explained in effective C++).

In a worst-case scenario,
1. Pefrom "new Widget"
2. Execute computePrioriry()
3. Run std::shared_ptr<> constructor

If such code is generated and, at runtime, computePriority() produces an exception, the dynamically allocated "new Widget" will be definitely leaked.

So let's use std::make_shared instead.
```cpp

void processWidget(std::make_shared<Widget>(), computePriority());

```
This way, the order of execution dosen't matter.

1. std::make_shared<Widget>
2. computePriority()

or

1. computePriority()
2. std::make_shared<Widget>

Both are exception safe in terms of a resource leak.

Regarding memory allocation, std::make_shared is better than std::shared_ptr with new.
Since std::make_shared will allocate memory for both the object itself and the control block(extra information such as count reference). However, if you use std::shared_ptr with new, it requires one memory allocation for the object and the a second allocation for the control block.

```cpp

std::shared_ptr<Widget> spw(new Widget); // allocation for widget and allocation for control block.

// This is better.
auto spw = std::make_shared<Widget>(); // allocation for both widget and control block.

```

Having said advantages of using make function over direct use of new, there are circumstances where we have to use new.
For example, none of the make funtions permit the specification of custom deleters.

```cpp

auto widgetDeleter = [](Widget* pw)
                    {
                        // e.g. log
                        std::cout << "log" << std::endl;
                        delete pw;
                    }

std::unique_ptr<Widget, decltype(widgetDeleter)> upw(new Widget, widgetDeleter);

std::shared_ptr<Widget> spw(new Widget, widgetDeleter);

```

One more situation where you have to use new is that when the performance and resource management is critical for your software.

If you create an object through std::make_shared, the object and the control block will be allocated together as I mentioned.
If this object is refered by std::weak_ptr and reference count becomes zero, this object will not be destroyed. It has to wait until std::weak_ptr is destroyed, because this information is stored in the control block and this control block is allcoated with the object.

```cpp

class ReallyBigType{};

auto pBigObj = std::make_shared<ReallyBigType>();

// if this ptr is refered by std::weak_ptr
// memory release is dictated by std::weak_ptr not std::shared_ptr.
```

So if you prioritize memory resource management, use shared_ptr with new.
```cpp

std::shared_ptr<ReallyBigType> pBigObj(new ReallyBigType);

```

The exception-unsafe call to a function with new which we discussed earlier,
we can avoid such a pitfall with a small change.

```cpp
void processWidget(std::shared_ptr<Widget>(new Widget), computePriority()); // potential resource leak

// Do this

std::shared_ptr<Widget> spw(new Widget, customDeleter);
processWidget(std::move(spw), computePriority());

```


***

## Item 22 : When using the Pimpl Idiom, define special member functions in the implementation file.

Nothing.


