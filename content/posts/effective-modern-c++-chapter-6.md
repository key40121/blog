+++
authors = ["Taichi Ichisawa"]
title = "Effective Modern C++ chapter 6"
date = "2023-10-11"
description = "Effective Modern C++ chapter 6"
tags = [
    "C++",
    "Effective Modern C++",
]
categories = [
    "C++",
    "books",
]
+++

# chapter 6 Lambda Expressions

```cpp

std::function<int()> func()
{
    int x = 0;
    return [=](){ return ++x; };
}

int main()
{
    auto f = func();
    std::cout << f() << std::endl; // 0
    std::cout << f() << std::endl; // 1
    std::cout << f() << std::endl; // 2
}

```


***

## Item 31 : Avoid default capture modes.

Default capture modes.
- by-reference [&]
- by-value [=]

Basically we should avoid using both default capture modes.

For instance, this is a wrong code.
Since divisor is not a local-static variable, this cannot be captured.
Captures apply only to non-static local variables(including parameters.).

```cpp
using FilterContainer = std::vector<std::function<bool(int)>>;

FilterContainer filters;

class Widget
{
public:
    void addFilter() const;
private:
    int divisor;
};

void Widget::addFilter() const
{
    filters.emplace_back(
        [](int value){ return value % divisor == 0; }
    );
}
```

If use by-value capture, compilers will figure out that it is passed this pointer.

```cpp

void Widget::addFilter() const
{
    filters.emplace_back(
        [=](int value){ return value % divisor == 0; }
    );
}

// equivalent

void Widget::addFilter() const
{
    auto currentObjectPtr = this;
    filters.emplace_back(
        [currentObjectPtr](int value){ return value % currentObjectPtr->divisor == 0; }
    );
}


```

Stil this is a dangerous approach since it depends on the lifetime of widget.
It owns a copy of a pointer to widget which can be released at any time.
Thus, what we should do is to create a local copy of what we want to pass.

```cpp

void Widget::addFileter() const
{
    auto divisorCopy = divisor;

    filters.emplace_back(
        [divisorCopy](int value)
        {
            return value % divisorCopy == 0;
        }
    );
}

```

Consider using static variables inside lambda.
This lambda dosen't capture anything, it just simply referes to a static variable.

```cpp

void addDivisorFileter()
{
    static auto val1 = 100;
    static auto val12 = 100;
    static auto divisor = val2/val1;

    fileters.emplace_back(
        [=](int value)
        {
            return value % divisor == 0;
        }
    );

    ++divisor;
}

```

## Item 32 : Use init capture to move objects into closures.

Init capture makes it possible to move objects into closures.

e.g.

```cpp

// value capture
[ data = (expression) ];

// reference capture
[&data = (expression) ];

```

The scope on the left of the "=" is that of the closure class.
On the other hand, the scope on the right side is tha same as where the lambda is being defined.

```cpp
class Widget
{
public:
    bool isValidated() const;
    bool isProcessed() const;

};

bool Widget::isValidated() const
{
    return true;
}

bool Widget::isProcessed() const
{
    return false;
}

int main() // C++14
{
    auto pw = std::make_unique<Widget>();

    auto func = [pw = std::move(pw)]{ return pw->isValidated() && pw->isProcessed(); };
    if (!func())
    {
        std::cout << "FALSE" << std::endl;
    };

    return 0;
}
```

## Item 33 : Use decltype on auto&& parameters to std::forward them.

```cpp

auto f = [](auto x){ return func(normalie(x)); };

// equivalent

class SomeCompilerGeneratedClassName
{
public:
    template<typename T>
    auto operator()(T x)
    {
        return func(normalize(x));
    }

};

```

If you want to use std::forward inside a lambda, use decltype for template type T.

```cpp

auto f = [](auto&& param)
        {
            return func(normalize(std::forward<decltype(param)>(param)));
        };

```

## Item 34 : Prefer lambdas to std::bind

Since there are too many things that are only applied to C++11, I'll skip Item.
