---
title: "Smart Pointers in Modern C++"
date: 2026-07-03
draft: false
tags:
  - cpp
  - memory
---

# Smart Pointers

Smart pointers in C++ are special objects that manage the lifetime of dynamically allocated memory automatically, preventing memory leaks and dangling pointers. They were introduced in C++11 and are defined in the `<memory>` header.

## Why Smart Pointers Matter

They make code safer, exception‑proof, and easier to maintain.

Traditional raw pointers require manual `new` and `delete`. Forgetting `delete` leads to memory leaks; deleting twice causes undefined behavior.

Smart pointers wrap raw pointers and use RAII (Resource Acquisition Is Initialization) to ensure memory is released when the smart pointer goes out of scope.

## Types of Smart Pointers

### a) `std::unique_ptr`

- Can transfer ownership using `std::move`
- Owns the object exclusively (no copies allowed)
- Object is deleted when the `unique_ptr` goes out of scope

```cpp
#include <iostream>
#include <memory>

struct Foo { 
    Foo() { std::cout << "Foo created\n"; } 
    ~Foo() { std::cout << "Foo destroyed\n"; } 
};

int main() {
    std::unique_ptr<Foo> ptr1 = std::make_unique<Foo>(); // create
    // std::unique_ptr<Foo> ptr2 = ptr1; // ERROR: cannot copy
    std::unique_ptr<Foo> ptr2 = std::move(ptr1); // OK: transfer ownership

    if(!ptr1) std::cout << "ptr1 is null\n";
}
```

✅ **Key points:**

- Single owner
- Lightweight
- Best for exclusive ownership

### b) `std::shared_ptr`

- Multiple smart pointers can **share ownership**
- Uses **reference counting**. When count reaches 0, object is deleted.

```cpp
#include <iostream>
#include <memory>

struct Foo {
    Foo() { std::cout << "Foo created\n"; }
    ~Foo() { std::cout << "Foo destroyed\n"; }
};

int main() {
    std::shared_ptr<Foo> sp1 = std::make_shared<Foo>();
    std::shared_ptr<Foo> sp2 = sp1; // shared ownership
    std::cout << "Use count: " << sp1.use_count() << "\n"; // 2
}
```

✅ **Key points:**

- Shared ownership
- Keeps track of references
- Slightly more overhead than `unique_ptr`

### c) `std::weak_ptr`

- Works with `shared_ptr` to **break circular references**
- Does **not increase reference count**
- Used to observe or access shared objects **without owning them**

```cpp
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> sp = std::make_shared<int>(42);
    std::weak_ptr<int> wp = sp; // observe sp

    std::cout << "Use count: " << sp.use_count() << "\n"; // 1

    if(auto temp = wp.lock()) { // converts weak_ptr to shared_ptr temporarily
        std::cout << "Value: " << *temp << "\n";
    }
}
```

✅ **Key points:**

- Prevents **memory leaks** due to cyclic references
- Observes objects without ownership

## Smart Pointers Ownership Diagram

![Smart Pointers Ownership Diagram](/smartptr1.webp)

## Best Practices

1. Prefer `std::unique_ptr` by default — simplest and safest.
2. Use `std::shared_ptr` only if ownership must be shared.
3. Use `std::weak_ptr` to break cycles and for observer patterns.
4. Never mix raw pointers and smart pointers for ownership.
5. Always use `std::make_unique` / `std::make_shared` instead of `new`.
