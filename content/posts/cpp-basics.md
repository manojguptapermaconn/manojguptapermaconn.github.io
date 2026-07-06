---
title: "C++ Basics"
date: 2026-07-06
draft: false
tags:
  - cpp
---

A focused reference for the C++ concepts that come up most in technical interviews. Each section covers the key ideas, common pitfalls, and what interviewers are really testing.

## 1. Stack vs Heap Memory

**Stack** memory is automatically managed — variables are allocated when a function is called and freed when it returns. It is fast but limited in size. **Heap** memory is manually managed (via `new`/`delete` or smart pointers) and far larger, but allocation is slower and you are responsible for cleanup.

Interview angle: interviewers expect you to know when to prefer stack allocation (small, short-lived objects) versus heap (large objects, unknown lifetime, polymorphism). Stack overflow is a common trap question.

## 2. RAII — Resource Acquisition Is Initialisation

RAII is the most important C++ idiom. It ties the lifetime of a resource (memory, file handle, mutex, socket) to the lifetime of an object. The resource is acquired in the constructor and released in the destructor — so it is automatically cleaned up when the object goes out of scope, even if an exception is thrown.

```cpp
class FileHandle {
    FILE* f;
public:
    FileHandle(const char* name) { f = fopen(name, "r"); }
    ~FileHandle()               { if (f) fclose(f); }  // automatic cleanup
};

void read() {
    FileHandle fh("data.txt");  // opens file
    // ... use fh ...
}   // destructor called here — file closed automatically
```

Smart pointers (`unique_ptr`, `shared_ptr`) are RAII wrappers for heap memory. See the [C++ Smart Pointers](/posts/cpp-smart-pointers/) page for a full breakdown.

## 3. Value vs Reference Semantics

Passing by **value** copies the object. Passing by **reference** (`&`) avoids the copy and lets the callee modify the caller's object. Passing by **const reference** (`const &`) avoids the copy without allowing modification — the preferred way to pass large objects you only need to read.

```cpp
void byValue(std::string s);         // copies s — expensive for large strings
void byRef(std::string& s);          // no copy, can modify
void byConstRef(const std::string& s); // no copy, read-only — prefer this
```

## 4. Move Semantics (C++11)

Move semantics allow resources to be *transferred* from a temporary (rvalue) rather than copied. This is critical for performance with large objects like vectors or strings. `std::move` casts an lvalue to an rvalue reference, enabling the move constructor or move assignment operator.

```cpp
std::vector<int> a = {1, 2, 3, 4, 5};
std::vector<int> b = std::move(a);  // transfers ownership — no copy
// a is now in a valid but unspecified state
```

Interview angle: understand the rule of five — if you define a destructor, copy constructor, copy assignment, move constructor, or move assignment, you probably need all five.

## 5. Pointers vs References

References must be initialised at declaration and cannot be reseated (made to refer to a different object). Pointers can be null, reseated, and used with pointer arithmetic. Prefer references when you always have a valid object; use pointers when nullability or reseating is needed.

## 6. Virtual Functions & Polymorphism

A `virtual` function enables runtime polymorphism — the correct overridden version is called based on the actual (dynamic) type of the object, not the declared (static) type. This is implemented via a vtable. Always declare destructors `virtual` in base classes to ensure proper cleanup.

```cpp
struct Animal {
    virtual void speak() const { std::cout << "..."; }
    virtual ~Animal() = default;  // virtual destructor — essential
};
struct Dog : Animal {
    void speak() const override { std::cout << "Woof"; }
};

Animal* a = new Dog();
a->speak();  // prints "Woof" — correct dynamic dispatch
delete a;    // ~Dog() called correctly because destructor is virtual
```

## 7. const Correctness

`const` tells the compiler (and the reader) that something should not be modified. Apply it to member functions that do not modify state, to parameters you only read, and to variables that should not change. Interviewers use `const` correctness as a proxy for code quality awareness.

## Quick Reference: Common Interview Topics

- **Rule of Zero / Three / Five** — when to let the compiler generate special member functions vs defining your own
- **nullptr vs NULL** — always prefer `nullptr` in modern C++
- **static keyword** — different meanings: static local variables (persist across calls), static class members (shared across instances), static free functions (internal linkage)
- **inline** — suggests to the compiler to expand the function at the call site; also controls ODR (One Definition Rule) for headers
- **Templates** — compile-time polymorphism; know the basics of function and class templates, and template specialisation
