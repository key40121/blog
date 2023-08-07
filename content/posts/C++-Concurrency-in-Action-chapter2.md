+++
draft = true
date = 2023-07-25T13:40:44Z
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++


## Launching a thread
By using std::thread, you can start running another thread.

```cpp
void do_some_work();
std::therad my_thread(do_some_work); // functor

// also you can pass a class with function call oparator or lambda
class background_task
{
public:
    void operator() () const
    {
        do_something();
        do_something_else();
    }
};

background_task f;
std::thread my_thread(f); // callable

std::thread my_thread([]{ // lambda
    do_something();
    do_something_else();
});

```

Needs to be careful when your thread has an access to data bevause it is easy to create a situation where your thread access to an object that has been destroyed.

```cpp

struct func
{
    int& i;
    func(int& i)
        :i(i_)
    {}
    void operator() ()
    {
        for (int j=0; j<1000000; ++j)
        {
            do_something(i);
        }
    }
}

void oops()
{
    int some_local_state = 0;
    func my_func(some_local_state); // Pass a local reference.
    std::thread my_thread(my_func);
    my_thread.detach(); // Dangerous!! undefined behavior;
}

```

When you deal with exceptions, RAII is best (almost always RAII is the answer for any problems)
```cpp
class thread_guard
{
    std::thread& t;
public:
    explicit thread_guard(std::thread &t)
        :t(t) {}
    ~thread_guard()
    {
        if(t.joinable())
            t.join();
    }
    thread_guard(thread_guard const&)=delete; // copy constructor
    thread_guard& operator=(thread_guard const&)=delete; // assignment operator
};

struct func;
void f()
{
    int local(0);
    func my_func(local);
    std::thread t(my_func);
    thread_guard g(t); // RAII!!
}

```

Passing arguments to std::thread is kind of simillar with std::bind.
For example, if you want to invoke a class's public member function, you can do this.

```cpp



```