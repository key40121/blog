+++
draft = true
date = 2023-07-25T13:40:38Z
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

C++ supports a concurrency from C++11.

I will take a look at the basis of concurrency in C++(11/14/17).

Let's output "Hello World" with concurrency program.

## Basic usages - thread, join, detatch, and joinable.

```cpp
#include <iostream>
#include <thread>

void hello()
{
    std::cout << "Hello Concurrent World" << std::endl;
}

int main()
{
    std::thread t(hello);
    t.join();
}

```

The functions and classes for managing threads are declared in <thread>.
Every thread has a have an initial function, which can be any callble, so you can pass, for instance, class object and lambda as well.

```cpp
class Task
{
public:
    void do_task() const
    {
        std::cout << "Hello Concurrent World from object functor" << std::endl;
    }

    void operator() () const
    {
        do_task();
    }
};

void do_func()
{
    std::cout << "Hello Concurrent World from lambda" << std::endl;
}

int main()
{
    Task f;
    std::thread t(f);
    t.join();

    std::thread thread([]{
        do_func();
    });
    thread.join();
}

```

The supplied function object is copied into the storage belonging to the newly made thread, so you might need to deal with copy behavior.

Once starting a thread, we need to explicitly decide whether to wait for it to finish or leave it to run on its own.
Basically, there are two options. You can use ***.join()*** to wait for a thread to complete, or ***.detatch()*** to make your thread run on its own.


.detatch() can cause an undefined behavir by accessing an object which has been destroyed.

```cpp
#include <iostream>
#include <thread>

struct Functor
{
    int& i;
    Functor(int& i) : i(i) {}
    void operator () ()
    {
        for (int j=0; i<100; ++j)
        {
            i++; // undefined behavior.
        }
    }
};

void not_valid()
{
    int local_ver = 0;
    Functor f(local_ver);
    std::thread my_thread(f);
    my_thread.detach(); // ERROR
}

int main()
{
    not_valid();
}
```

You have to deal with exception as well when it is involved. It dosen't make sense to add ***.join()*** in try/catch block or wherever it is needed.
Simply you can use RAII like this.

```cpp
#include <iostream>
#include <thread>

class thread_RAII
{
    std::thread& t;
public:
    explicit thread_RAII(std::thread& t) : t(t) {}
    ~thread_RAII()
    {
        if(t.joinable())
        {
            t.join();
        }
    }

    thread_RAII(thread_RAII const&)=delete;
    thread_RAII& operator=(thread_RAII const&)=delete;
};

int main()
{
    std::thread my_thread;
    thread_RAII thread(my_thread);
}
```

***

## Passing arguments to a thread function

std::thread constructor dosen't know what arguments are passed in.
However, internal code passes copied arguments as rvalues in order to work with move-only types.
So, if you pass reference you need to wrap arguments with ***std::ref***.

```cpp

#include <iostream>
#include <thread>

void f(int& i, const std::string& s)
{
    std::cout << "Do nothing" << std::endl;
}

void error()
{
    int i = 100;
    std::string str = "test";
    std::thread t(f, std::ref(i), std::ref(str));
    t.join();
}

int main()
{
    error();
}

```
If you want to invoke a member function on the new thread, you can pass arguments the way you do when using std::bind.

```cpp
class Test
{
public:
    void do_something();
};

int main()
{
    Test test;
    std::thread t(&Test::do_something, &test); // like std::bind
}

```

***

## Transfer the ownership of a thread.

std::therad is not copyable but moveable like std::unique_ptr.

```cpp
#include <iostream>
#include <thread>

int main()
{
    std::thread t1(do_something());
    std::thread t2 = std::move(t1); // explicitly call move constructor;
}
```

You can guard a thread by creating a gurad thread class, which is similar with RAII thread class.

Destructor always make sure to call .join(), and constructor covers an exception.

```cpp
#include <iostream>
#include <thread>

class GuardThread
{
public:
    explicit GuardThread(std::thread&& t) : t(std::move(t))
    {
        if (t.joinable() == false) throw std::logic_error("No thread");
    }
    ~GuardThread()
    {
        t.join();
    }

private:
    std::thread t;
};

int main()
{
    std::thread t1;
    GuardThread guardThread{std::move(t1)};
}

```