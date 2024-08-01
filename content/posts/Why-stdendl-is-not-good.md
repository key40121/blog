+++
authors = ["Taichi Ichisawa"]
title = "Why std::endl is not good in terms of performance"
date = "2024-08-02"
description = "Sample"
tags = [
    "C/C++",
]
categories = [
    "C/C++",
]
+++

***

## std::endl vs \n

I had never been cared about the performance of std::endl before, but I knew it was not preferable.
Here I explain why and did some tests to measure the actual difference.

### Difference

If we limit the discussion to linux, system call is very slow compared to calling ones from any library due to several reasons such as context switching from user space to kernel space.
Hence, we really have to minimize the number of system calls.

System calls has a function named write() which takes file descriptor ad buffer.
This function is simple but it is norm that we don't use it directly since again it is very slow especially when calling it with small chunk of data.
Therefore, it is encouraged to use wrppper funcitons provided by stdio, which has a buffer in it, and before stdio uses system call such as write(),
it actually saves data to the buffer, and then once buffer is full, it actually calls write().

The difference between \n and std::endl is when to flush the buffer in stdio. 
\n dosen't flush the buffer. It simply is treated as \n, and there is nothing more.
std::endl is different. It is not treated as LF(\n) but it explicitly flush the buffer and then calls write().
Therefore, it is safe to assume that the number of std::endl your code has is the number of calling write().
On the other hand, when using \n, write() should be called once the buffer is full, so we can get benefits from the implementation of stdio.

### test on stdout

I have prepared one simple program to measure a performance of outputing numbers one milion times to stdout using either std::endl or \n.

```cpp

#include <iostream>
#include <chrono>

int main() {

  auto start = std::chrono::high_resolution_clock::now();

  for (int i=0; i<1000000; i++) {
  std::cout << i << std::endl;
 }

  auto end = std::chrono::high_resolution_clock::now();

  double elapsed1 = std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count();
  std::cout << "endl took : " << elapsed << "[ms]\n";

  start = std::chrono::high_resolution_clock::now();

  for (int i=0; i<1000000; i++) {
  std::cout << i << "\n";
 }

  end = std::chrono::high_resolution_clock::now();

  double elapsed2 = std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count();
  std::cout << "endl took : " << elapsed1 << "[ms]\n";
  std::cout << "`\n` took : " << elapsed2 << "[ms]\n";

  return 0;
}

```

The result

```
endl took : 5520[ms]
\n took : 8012[ms]
```

This is because if a stream is standard output, it flushes a buffer when sees \n.
In this case, stream is standard output, and also c++ probably optimizes the performance for std::endl.

### test on a file

The test program is like this. Here we use a simple text file for output stream, which means there would be no flushes when \n is passed.

```cpp

#include <iostream>
#include <chrono>
#include <fstream>

int main() {
    std::ofstream file("output.txt");

    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < 1000000; ++i) {
        file << i << std::endl;
    }

    auto end = std::chrono::high_resolution_clock::now();

    double elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();
    std::cout << "Took with std::endl: " << elapsed << " ms\n";

    file.close();

    file.open("output.txt");

    start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < 1000000; ++i) {
        file << i << '\n';
    }

    end = std::chrono::high_resolution_clock::now();

    elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();
    std::cout << "Took with \\n: " << elapsed << " ms\n";

    file.close();

    return 0;
}


```

Now we cann see the huge difference between them.

```
Took with std::endl: 827 ms
Took with \n: 31 ms
```

