---
title: "C++11 / C++14 / C++17 / C++20 / C++23"
date: 2026-07-06
draft: false
tags:
  - cpp
---

A tour of the major language and library features introduced in each C++ standard since C++11, with an emphasis on what actually comes up in technical interviews.

## C++11

C++11 was the biggest overhaul of the language since the original standard — it's the baseline every "modern C++" claim is measured against.

### 1. `auto` — type inference

The compiler deduces the variable's type from its initializer. Reduces noise, especially with long iterator/template types.

```cpp
auto x = 42;                 // int
auto it = myMap.begin();     // whatever that iterator type actually is
```

### 2. Range-based for loops

Iterate over a container without manually managing iterators.

```cpp
for (const auto& item : container) {
    // use item
}
```

### 3. `nullptr`

A type-safe null pointer constant, replacing `0`/`NULL`. Fixes overload resolution ambiguity between pointer and integer overloads.

```cpp
void f(int);
void f(char*);
f(NULL);     // ambiguous/surprising on some compilers — NULL is often just 0
f(nullptr);  // unambiguously calls f(char*)
```

### 4. Move semantics & rvalue references

`&&` denotes an rvalue reference, enabling **move** instead of **copy** for temporaries — the single biggest performance-related addition in C++11. See the [C++ Basics](/posts/cpp-basics/) page for the full breakdown, and [Smart Pointers](/posts/cpp-smart-pointers/) for how `unique_ptr` relies on it.

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = std::move(a);  // transfers internal buffer, no copy
```

### 5. Lambda expressions

Anonymous inline functions, with a capture list controlling what surrounding state they can see.

```cpp
auto add = [](int a, int b) { return a + b; };
int threshold = 10;
auto isAboveThreshold = [threshold](int x) { return x > threshold; };
```

Interview angle: know the difference between capture by value `[x]`, by reference `[&x]`, and capture-all `[=]` / `[&]`.

### 6. Scoped enumerations (`enum class`)

Unlike plain `enum`, values don't leak into the surrounding scope and don't implicitly convert to `int`.

```cpp
enum class Color { Red, Green, Blue };
Color c = Color::Red;
// int x = c; // error — no implicit conversion, unlike a plain enum
```

### 7. `override` and `final`

`override` tells the compiler a function is meant to override a virtual base method — catches typos/signature mismatches at compile time instead of silently creating a new function. `final` prevents further overriding or inheritance.

```cpp
struct Base {
    virtual void speak() const;
};
struct Derived : Base {
    void speak() const override;  // compiler-checked
};
```

### 8. Uniform initialization

Curly-brace initialization works consistently across built-in types, aggregates, containers, and classes with a constructor taking `std::initializer_list`.

```cpp
int arr[] = {1, 2, 3};
std::vector<int> v = {1, 2, 3};
struct Point { int x, y; };
Point p{1, 2};
```

### 9. `constexpr`

Marks a function or variable as computable at compile time (given constant inputs), moving work from runtime to compile time.

```cpp
constexpr int square(int x) { return x * x; }
constexpr int result = square(5);  // computed at compile time
```

### 10. Standard threading support

`<thread>`, `<mutex>`, `<atomic>`, and `<condition_variable>` brought portable, standard-library concurrency primitives — no more relying on platform-specific APIs or third-party libraries for basic multithreading.

```cpp
#include <thread>
#include <mutex>

std::mutex m;
void worker() {
    std::lock_guard<std::mutex> lock(m);
    // critical section
}
std::thread t(worker);
t.join();
```

### Quick Reference: Other notable C++11 additions

- `static_assert` — compile-time assertions
- `=default` / `=delete` — explicitly request or suppress compiler-generated special member functions
- `std::tuple`, `std::array` — fixed-size, lighter-weight alternatives to existing containers
- Unordered containers — `std::unordered_map`, `std::unordered_set` (hash-table backed)
- Variadic templates — templates taking an arbitrary number of type parameters
- `decltype` — deduces the type of an expression without evaluating it

## C++14

Coming soon.

## C++17

Coming soon.

## C++20

Coming soon.

## C++23

Coming soon.
