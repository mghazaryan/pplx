# Extraction of Microsoft PPLX out of C++ REST SDK

## Table of Contents
**[Description](#description)**<br>
**[How](#how)**<br>
**[Build](#build)**<br>
**[Examples](#examples)**<br>
**[References](#references)**<br>

## Description

PPLX is a trimmed version of PPL ([Parallel Patterns Library](https://msdn.microsoft.com/en-us/library/dd492418.aspx), Microsoft's offering for task parallelism, parallel algorithms and containers for C++), featuring only tasks and task continuations, with plain vanilla thread pool implementation underneath. Officially PPLX comes bundled within [C++ REST SDK](https://github.com/Microsoft/cpprestsdk), a cross-platform library from Microsoft for creating REST services. If you use something else for REST, but still want to utilize PPLX for generic concurrency, you have to include C++ REST SDK in its entirety, which is weird.

This project is an extraction of PPLX out of C++ REST SDK, presenting it in a form of a shared or static library. It is of some value for *\*nix* systems, where you don't have the luxury of using the full-fledged version – PPL.

## How

What was done was basically extracting the minimum code subset representing PPLX out of a specific version of C++ REST SDK (refer to [``CMakeLists.txt``](./CMakeLists.txt) for the exact version number). While I tried to avoid touching the code base as much as possible, some adjustments still slipped in. Particularly, a few preprocessor include statements went through path and file name corrections due to header file relocations. For example:

```cpp
#include "cpprest/details/cpprest_compat.h"
```

has become:

```cpp
#include "compat.h"
```

## Build

### Requirements

* GCC >= 4.8, Visual Studio >= 2015
* CMake >= 3.2
* Boost Libraries >= 1.56

### Steps

Prepare CMake environment appropriately:

```commandline
$ mkdir build
$ cd build
$ cmake ..
```

Build via ``make`` on *\*nix* side:

```commandline
$ make
$ sudo make install
```

Build via VS on *Windows*:

* Open the corresponding solution in Visual Studio, run build appropriately;
* Copy the resultant DLL into a directory within your library search path.

## Examples

The concurrency model of PPL is quite similar to the one coming from the C++ Standard Library, with a slight skew towards the notion of task<sup>[[1]](#r1 "Microsoft Concurrency Runtime: Task Parallelism")</sup>. E.g. you launch an asynchronous operation by just creating a new task object (as opposed to an [`std::async<>`](https://en.cppreference.com/w/cpp/thread/async) call), and you basically deal with task objects rather than [*futures*](https://en.cppreference.com/w/cpp/thread/future). In the following example we'll obtain the angle between two Euclidean vectors from their dot product and magnitudes calculated concurrently.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <cmath>
#include <pplx/pplxtasks.h>

int calcDotProduct(const std::vector<int>& a, const std::vector<int>& b)
{
  return std::inner_product(a.begin(), a.end(), b.begin(), 0);
}

double calcMagnitude(const std::vector<int>& a)
{
  return std::sqrt(calcDotProduct(a, a));
}

int main()
{
  // To keep things simple let's assume we deal with non-zero vectors only.
  const std::vector<int> v1 { 2, -4, 7 };
  const std::vector<int> v2 { 5, 1, -3 };

  // Form separate tasks for calculating the dot product and magnitudes.
  pplx::task<int> v1v2Dot([&v1, &v2]() {
    return calcDotProduct(v1, v2);
  });
  pplx::task<double> v1Magnitude([&v1]() {
    return calcMagnitude(v1);
  });
  pplx::task<double> v2Magnitude([&v2]() {
    return calcMagnitude(v2);
  });

  std::cout << "Angle between the vectors: "
            << std::acos((double)v1v2Dot.get() / (v1Magnitude.get() * v2Magnitude.get()))
            << " radians."
            << std::endl;

  return 0;
}
```

## References

1. <a name="r1">[Microsoft Concurrency Runtime: Task Parallelism](https://msdn.microsoft.com/en-us/library/dd492427.aspx)</a>