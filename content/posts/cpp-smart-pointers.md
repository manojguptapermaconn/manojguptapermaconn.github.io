---
title: "Smart Pointers in Modern C++"
date: 2026-07-03
draft: false
tags:
  - cpp
  - memory
---

# Smart Pointers

Smart pointers provide automatic memory management in C++.

## unique_ptr

```cpp
auto ptr = std::make_unique<int>(42);