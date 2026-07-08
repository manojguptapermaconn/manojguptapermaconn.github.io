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

C++14 was a refinement release — it built on C++11 and knocked off rough edges rather than introducing another major overhaul. Features below are roughly ordered by how often they show up in day-to-day code.

### 1. Generic lambdas

A lambda parameter can be declared `auto`, making the lambda's call operator effectively a template — the same lambda body works across argument types.

```cpp
auto add = [](auto a, auto b) {
    return a + b;
};
add(1, 2);       // int
add(1.5, 2.5);   // double
```

Frequently used with STL algorithms where the same lambda gets reused across containers of different element types.

### 2. `std::make_unique`

```cpp
auto emp = std::make_unique<Employee>("John");
```

`std::make_shared` already existed in C++11 — `make_unique` was simply left out, an oversight fixed in C++14. Beyond the cleaner syntax, it closes a real exception-safety hole: in something like `foo(std::unique_ptr<T>(new T), mightThrow())`, the order in which arguments are evaluated isn't guaranteed, so if `mightThrow()` runs and throws *after* `new T` but *before* the raw pointer is wrapped, that memory leaks — it was never given to anything that would clean it up. `make_unique` keeps allocation and ownership as a single, non-interruptible expression, so that gap can't happen.

### 3. Return type deduction

```cpp
auto multiply(int a, int b) {
    return a * b;
}
```

Reduces boilerplate and eases refactoring, but with real restrictions worth knowing for interviews: every `return` statement in the function must deduce to the *same* type (or it's a compile error), and it can't be used on `virtual` functions — the compiler needs the full function body visible at the call site to deduce anything, which a virtual dispatch can't guarantee.

### 4. `decltype(auto)`

Deduces a type using `decltype`'s rules instead of `auto`'s — the difference matters because `auto` strips references and top-level `const`, while `decltype(auto)` preserves them exactly.

```cpp
int x = 10;
int& getRef() { return x; }

auto a = getRef();           // int   — auto strips the reference, copies x
decltype(auto) b = getRef(); // int&  — reference preserved
```

Interview angle: this is the tool for writing generic forwarding wrappers that need to return *exactly* what the wrapped call returns — value, reference, or const reference — without accidentally collapsing it to a value.

### 5. Generalized lambda capture (init capture)

A capture can now declare and initialize a new variable with an arbitrary expression, not just reference an existing one — most notably, this makes move-only types capturable.

```cpp
auto ptr = std::make_unique<int>(42);

auto func = [p = std::move(ptr)]() {
    std::cout << *p;
};
```

Common in asynchronous/callback code where a `unique_ptr` needs to transfer ownership into a closure.

### 6. Relaxed `constexpr`

C++11's `constexpr` functions were limited to a single `return` statement — no loops, no local variables, no mutation. C++14 relaxed that considerably.

```cpp
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i)
        result *= i;
    return result;
}
```

This made `constexpr` practical for real algorithms instead of just simple expressions, and laid the groundwork for the much heavier compile-time programming added in C++17/20.

### 7. `[[deprecated]]` attribute

A standard, portable way to mark a function, class, or variable as deprecated — any call site gets a compiler warning, without removing the symbol outright.

```cpp
[[deprecated("use newFunction() instead")]]
void oldFunction();
```

### 8. `std::exchange`

Sets a variable to a new value and returns its *old* value, in one call. Small utility, but it shows up constantly in move constructors and move-assignment operators to null out the moved-from object cleanly.

```cpp
MyClass(MyClass&& other) noexcept
    : ptr(std::exchange(other.ptr, nullptr)) {}
```

### 9. Heterogeneous lookup in associative containers

Passing the transparent comparator `std::less<>` lets you look up a `map`/`set` using a *different but comparable* type, without constructing a temporary of the container's actual key type first.

```cpp
std::map<std::string, int, std::less<>> m;
m["hello"] = 1;

auto it = m.find("hello");  // const char* — no temporary std::string built just to search
```

Interview angle: a genuine, measurable win on hot lookup paths, and a good signal of performance awareness in a review.

### 10. Reader-writer locks

`<shared_mutex>` brought reader-writer locking to the standard library: any number of threads can hold a shared (read) lock at once, while an exclusive (write) lock still locks out everyone else.

```cpp
#include <shared_mutex>

std::shared_timed_mutex rw;

void reader() {
    std::shared_lock<std::shared_timed_mutex> lock(rw);  // multiple readers OK
}
void writer() {
    std::unique_lock<std::shared_timed_mutex> lock(rw);  // exclusive
}
```

Careful with the exact type here: C++14 only added `std::shared_timed_mutex` and `std::shared_lock`. The simpler, non-timed `std::shared_mutex` wasn't added until **C++17** — an easy detail to get backwards.

### 11. Digit separators

```cpp
long population = 1'000'000;
```

Purely a readability aid — the single quote is ignored by the compiler, making large numeric literals easier to read at a glance and harder to mistype.

### 12. Binary literals

```cpp
int mask = 0b10101010;
```

Combines naturally with digit separators for longer bit patterns: `0b1010'0110'0001'1101`.

### 13. Variable templates

A template that parameterizes a *variable* rather than a function or class — most commonly seen for compile-time constants that need to work across multiple types.

```cpp
template<typename T>
constexpr T pi = T(3.1415926535897932385);

float f = pi<float>;
double d = pi<double>;
```

### Quick Reference: Most impactful day-to-day

If you only remember five: **generic lambdas**, **`std::make_unique`**, **return type deduction**, **`decltype(auto)`**, and **generalized lambda capture** account for the bulk of C++14 usage in real codebases — the rest are smaller, situational conveniences.

## C++17

Coming soon.

## C++20

Coming soon.

## C++23

Coming soon.
