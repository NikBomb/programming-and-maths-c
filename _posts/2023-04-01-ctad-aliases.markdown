---
layout: post
title:  "Effective C++ Programming Techniques: Mastering CTAD for Alias Templates in C++20"
date:   2023-04-01 08:00:00 +0000
categories: [C++]
---


CTAD (Class Template Argument Deduction) is an important feature introduced in C++17 that simplifies the process of initializing objects of template classes. Before CTAD, when initializing an object of a template class, developers had to explicitly provide the template arguments, which could result in verbose and difficult-to-read code.

With CTAD, C++17 introduced the ability for the compiler to deduce the template arguments from the constructor arguments, making it possible to omit the explicit template arguments. This not only simplifies the initialization process, but also makes code more concise and easier to read.

As always a code example is worth more than a thousand words:

```cpp
// pre C++17 with explicit template arguments
std::pair<int, const char*> my_pair_explicit{42, "hello"};

// Using CTAD to deduce the template arguments of std::pair
auto my_pair = std::pair{42, "hello"};
```

In the above snippet, we use CTAD to create an instance of `std::pair` with two values: an `int` and a `const char*`. Instead of explicitly specifying the template arguments (`<int, const char*>`), we simply use the brace initialization syntax with std::pair, and let the compiler deduce the types of the two values.

CTAD can be particularly useful when working with template classes that have long and complex type signatures. By allowing the compiler to deduce the template arguments, developers can write more concise and readable code, without sacrificing type safety.

However, there was a gap in the CTAD feature introduced in C++17: it did not work with alias templates. While CTAD allowed for the automatic deduction of template arguments when initializing objects of template classes, this only applied to primary templates, not to alias templates. This meant that developers using alias templates were still required to manually specify the template arguments when initializing objects, leading to verbose and error-prone code.

```cpp
// Custom allocator for std::vector
template<typename T>
struct MyAllocator {
    // Implementation details omitted for brevity
};

// Alias template for std::vector with custom allocator
template<typename T>
using MyVector = std::vector<T, MyAllocator<T>>;

// Create an instance of MyVector with CTAD - does not work in C++17!
auto my_vector = MyVector{1, 2, 3};

```

In the above code, we define a custom allocator `MyAllocator` and use it to create an alias template `MyVector` for `std::vector`. We then try to create an instance of `MyVector` using CTAD, but this will not work in C++17, as CTAD is not supported for template aliases.

To create an instance of `MyVector` in C++17, we would need to explicitly specify the template arguments:

```cpp
MyVector<int> my_vector_explicit{1, 2, 3};
```
As we can see, this leads to more verbose code, which can be more difficult to read and maintain, particularly for complex types.

In C++20 we can now use CTAD with template aliases, including our `MyVector` alias template. This means that we can create instances of `MyVector` using brace initialization without needing to explicitly specify the template arguments, making our code more concise and easier to read:

```cpp
auto my_vector_cxx20 = MyVector{1, 2, 3};
```

This code automatically deduces the template arguments for `MyVector` based on the initializer values {1, 2, 3}, and creates an instance of `MyVector<int>` using the custom allocator `MyAllocator`.

By using CTAD with alias templates, we can make our code even more concise and easier to read, without sacrificing type safety or clarity.

In conclusion, C++20 CTAD for alias templates allows for more concise and readable code, especially when working with complex or nested types. With this feature, you can create objects of an alias template without having to specify the template arguments explicitly, just like with class templates.