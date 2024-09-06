---
layout: post
title:  "Implement monadic and_then for std::variant"
date:   2023-03-19 08:00:00 +0000
categories: [C++]
---

In a previous [post](https://nicobombace.com/c++/monadic) we have explored the monadic operations `transform` and `and_then`, implemented for `std::optional` and `std::expected` in C++23. The main advantage of using these operations is that they make code easier to parse and reduce the use of intermidiate variables.

In this post we will focus on an implementation of `and_then` for `std::variant`. The first question to answer is what exactly should this operation do? We can adapt the definition used previously for this new type.

* `and_then` accepts an input  `std::variant<Q, R>` and a function for each variant type defined as:

```cpp
std::variant<S, T> f(Q q);
std::variant<S, T> f(R r);
```
and returns a `std::variant<S, T>` being the result of `f(q)` or `f(r)`. 

To better follow the implementation and the current limitations of `std::variant` API for this use case let's consider a toy example. Let's assume we have a `std::variant<T, std::complex<T>>` and we want to convert this to a `std::variant<T, std::tuple<T, T>>`, following this two-steps algorithm:

* If the starting variant holds a `T` we will first transform it to a `std::complex<T,T>` where the real and imaginary parts are equal to the value in the starting variant. Else if the variant holds a `std::complex<T,T>` we will first transform it to `T` returning the norm of the complex number. We will call this the `Cross` operation.

* In the second step, if the new variant holds a `T` we will just return it. Otherwise, if it holds a `std::complex<T,T>` we will return a `std::tuple<T,T>` containing the real and imaginary part of the starting complex number. We will call this the `MaybeConvert`operation.

We could implement this algorithm simply using `std::visit` as follows:

```cpp
#include <variant>
#include <complex>
#include <tuple>

template <typename T>
using variant_1 =  std::variant<T, std::complex<T>>;

template <typename T>
using variant_2 = std::variant<T, std::tuple<T, T>>;


struct CrossVisitor {
    
    template<typename T>
	constexpr variant_1<T> operator()(const T i) {
		return std::complex<T>{i, i};
	}

    template<typename T>
    constexpr variant_1<T> operator()(const std::complex<T> c) {
		return norm(c);
	}
};



struct MaybeConvertVisitor {

	template<typename T>
	constexpr variant_2<T> operator()(const T f) {
		return T{f};
	}

    template<typename T>
	constexpr variant_2<T> operator()(const std::complex<T> c) {
		return std::tuple<T, T>{std::real(c), std::imag(c)};
	}

};


int main() {

	constexpr variant_1<float> v = float{3};
    constexpr auto converted = std::visit(MaybeConvertVisitor{}, std::visit(CrossVisitor{}, v)); // (1)

    static_assert(std::get<0>(std::get<std::tuple<float, float>>(converted)) == float{3});
    static_assert(std::get<1>(std::get<std::tuple<float, float>>(converted)) == float{3});
}
```
In the snippet above we have leveraged the ability of `std::visit` to work with functors, and the result is correct however this solution is not optimal (if we had  `and_then` on `std::variant`) for at least two reasons:

* When reading (1) the order of operations is mixed, we first read `MaybeConvertedVisitor` and later `CrossVisitor`.
* Line (1) is cluttered by noise, namely `std::visit`.

If we had the possibility to add member functions to `std::variant` we could implement `and_then`, but we will choose a different and less intrusive option. We will implement this operation using the | operator. This feels like a natural choice since the pipe operator is already in use in `std::ranges` where it conveys a similar meaning, and also is the easiest option to modify this STL type. A simple implementation looks like this 
```cpp

namespace monadic_variant{
    template<typename... Ts, typename Visitor>
    constexpr auto operator| (const std::variant<Ts...>& v, Visitor&& callable) {
    	return std::visit(std::forward<Visitor>(callable), v);
    }

    template<typename... Ts, typename Visitor>
    constexpr auto operator| (std::variant<Ts...>&& v, Visitor&& callable) {
        return std::visit(std::forward<Visitor>(callable), v);
    }
}
```
The `monadic_variant` namespace implements `and_then` using the pipe operator. Its implementation is very simple and it consists of a template variadic | operator, so it can bind to any variant, and passes it to a `std::visit` together with a callable (functor, lambda etc...), passed in as a universal reference. We provide two overloads so that we can call an optimized version in the case we have a r-value or an l-value `std::variant`. 

Using this new operator we can change our main function to the following

```cpp
int main() {
	constexpr variant_1<float> v = float{3};
    {
        using namespace monadic_variant; // (1)   
        constexpr auto converted = v | CrossVisitor{} | MaybeConvertVisitor{}; // (2)
        static_assert(std::get<0>(std::get<std::tuple<float, float>>(converted)) == float{3});
        static_assert(std::get<1>(std::get<std::tuple<float, float>>(converted)) == float{3});
    }
}
```
The main differences with our previous `main` are in (1) and (2). In (1) we use the `monadic_namespace` namespace so that the compiler can resolve the | operator. Thank to this operator used in (2) we can now leverage a fluent and more natural API. This is closer to functional style programming for std::variant.

In conclusion we have implented one of the monadic operations, `and_then`,  defined in C++23 in terms of pipe operator for `std::variant` using `std::visit`, leading to more readable code.

