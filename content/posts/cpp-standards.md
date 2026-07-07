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

Beyond capturing individual variables by value (`[x]`) or reference (`[&x]`), a lambda can capture everything it uses with a default:

```cpp
int threshold = 10;
int count = 0;
auto f = [=, &count](int x) {
    if (x > threshold) count++;   // threshold: copied in, count: by reference
};
```

- `[=]` — capture everything used in the body **by value** (a copy each).
- `[&]` — capture everything used in the body **by reference**.
- Defaults can be mixed with explicit exceptions, e.g. `[=, &count]` or `[&, threshold]`.

Interview angle: `[&]` capturing locals in a lambda that outlives its enclosing scope (returned from a function, or stored and invoked later) leaves dangling references — a common source of bugs.

### 6. Scoped enumerations (`enum class`)

Unlike plain `enum`, values don't leak into the surrounding scope and don't implicitly convert to `int`.

```cpp
enum class Color { Red, Green, Blue };
Color c = Color::Red;
// int x = c; // error — no implicit conversion, unlike a plain enum
```

### 7. `override` and `final`

`override` tells the compiler a function is meant to override a virtual base method — catches typos/signature mismatches at compile time instead of silently creating a new function. `final` prevents further overriding or inheritance.

The base class function **must already be `virtual`** for `override` to compile. `override` doesn't make a function overridable by itself — it's a compiler-checked assertion that a matching virtual function (same name, parameters, const/ref-qualifiers, and a covariant return type) already exists in a base class. If it doesn't — because the base function isn't `virtual`, or the signature is slightly off — the compiler rejects the code instead of silently creating an unrelated new function that just hides the base one.

```cpp
struct Base {
    virtual void speak() const;   // must be virtual
};
struct Derived : Base {
    void speak() const override;  // OK — matches Base::speak exactly
    // void speak() override;     // ERROR: missing const, doesn't match — caught at compile time
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

### 9. `std::initializer_list`

A lightweight, non-owning view over a braced list of values (`{1, 2, 3}`), letting constructors and functions accept that syntax directly.

```cpp
class MyContainer {
public:
    MyContainer(std::initializer_list<int> list) {
        for (int v : list) { /* ... */ }
    }
};
MyContainer c = {1, 2, 3, 4};
```

Interview angle: if a class has both a `(size_t, T)` constructor and an `initializer_list` constructor, brace syntax always prefers the `initializer_list` one — a classic gotcha:

```cpp
std::vector<int> a(10, 5); // 10 elements, each = 5
std::vector<int> b{10, 5}; // 2 elements: {10, 5}  — initializer_list wins
```

### 10. Delegating constructors

One constructor can call another constructor of the *same* class in its member-init list, instead of duplicating initialization logic.

```cpp
class Rectangle {
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    Rectangle() : Rectangle(1, 1) {}   // delegates to the two-arg constructor
};
```

Rule: if a constructor delegates, that delegation must be the *only* entry in its member-init list — you can't delegate and also initialize other members in the same list. The delegated-to constructor runs to completion (including its body) before control returns to the delegating constructor's own body.

### 11. Variadic templates

Templates that accept an arbitrary number of type parameters via a **parameter pack** (`typename... Args` / `Args... args`). Pre-C++17, processing the arguments typically means recursing, peeling off one at a time:

```cpp
void print() {}  // base case: no more args

template<typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << " ";
    print(rest...);  // recurse with one fewer argument
}
```

C++17's fold expressions do the same thing without recursion:

```cpp
template<typename... Args>
void print(Args... args) {
    ((std::cout << args << " "), ...);  // unary right fold over the comma operator
}
```

`sizeof...(Args)` gives the pack size at compile time. This machinery underlies `std::make_unique`, `vector::emplace_back`, and `std::tuple`.

### 12. `constexpr`

Marks a function or variable as computable at compile time (given constant inputs), moving work from runtime to compile time.

```cpp
constexpr int square(int x) { return x * x; }
constexpr int result = square(5);  // computed at compile time
```

### 13. Standard threading support

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
- `decltype` — deduces the type of an expression without evaluating it

## C++14

Coming soon.

## C++17

Coming soon.

## C++20

Coming soon.

## C++23

Coming soon.
