---
title: "C++ auto keyword makes your life easier"
date: 2020-12-03T18:10:20+01:00
image: /images/gears.jpg
imageQuality: "q65"
imageAnchor: "Center" # Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
tags: ["C++"]
categories: "C++" 
summary: "In C++, *auto* keyword can speed up coding and improve the maintainability of code. Here, I show cases that *auto* can make a difference."
---

## Introduction

Since *C++11*, a compiler can deduce the type of a variable being declared from its initializer using `auto` keyword. It is also called placeholder type specifier.


## General Behavior

`auto` infers the type of a variable from its initializer. The variable gets a copy of the initializer

```cpp
int i;
auto j = i; // j is int contains a copy of i
```

`auto` deduces reference type as a value type. To enforce being a reference, use `auto&`

```cpp
int m = 0;
int& i = m; // i is an alias to m
auto j = i; // j is int contains a copy of i
auto& k = m; // k is int& and alias to m
```
`auto` can Not infer `const` qualifier. To enforce that, use `const auto`

```cpp
int i =0;
const int m = 0;
auto j = m; // j is int (non-const)
const auto k = m; // k is const int
const auto l = i; // l is const int, holds a copy of i
```

`auto&` infers `const` qualifier

```cpp
const int i =0;
auto& j = i; // const int&, alias for i
```



## Literals

`auto` figures out the below types: 

```cpp
auto i = 1; // int
auto j= 7ul; // unsigned long
auto x = 2.0; // double
auto y = 2.0f; // float
auto c = 'A'; // char
auto s = "hi"; // char const*
auto b = true; // bool
```

From C++14, we can have std::string literals:

```cpp
using namespace std; // This is necessary
auto s = "hellow"s // std::string, note the s operator. 
```

## Classes

Custom classes are also inferred:

```cpp
class LongNameClass {};
auto a = new LongNameClass(); // a is LongNameClass*
```

List initializer:

```cpp
auto l = {1,2,3}; // l is std::initializer_list<int>
```

Long name types can be easily replaced by `auto`:

```cpp
std::map<std::string, std::string> m; 
auto n = m;

auto p = std::make_unique<int>(); //unique pointer

std::vector<double> vec(100);
auto& r = vec;
```

A vector iterator type can be hidden with auto

```cpp
// type of i is std::vector<double>::iterator
for (auto i = vec.begin(); i != vec.end(); i++){
    cout<< *i;
}
```

The elements of a vector can be easily read:

```cpp
std::vector<unsigned long long int> vec = {1, 2, 3};

for (auto& x : vec)
    cout << x ;
```


## Pointers

A pointer type is also deduced:

```cpp
int i;
auto p = &i; // int*
auto q = p;  //int*
auto r = new int[5]; // int*
```

We can emphasize pointer type with `*` for readability. The outcome of the example below is the same as the previous one:

```cpp
int i;
auto* p = &i; // int*
auto* q = p; // int*
```

Note both `const auto` and `auto const` are turned to a constant pointer with a mutable target. We still can customise the outcome pointer with `*`:

```cpp
const auto r = &i; // int* const: constant pointer, mutable target
auto const s = &i; // int* const
auto* const t = &i;// int* const
    
const auto* u = &i;// const int*: mutable pointer, constant target
auto const* v = &i;// const int* or int const* 

const auto* const w = &i;// const int* const: pointer & target constant
```


## References

By default, `auto` deduces reference as the value type 

```cpp
int y;
int& p = y;
auto x = p; // x is int and contains a copy of p
```

This value copy behavior can dramatically create a problem if you just want to create an alias to a big size vector. To overcome it, we have two options: `auto&` or `decltype(auto)` since *C++14*

```cpp
int y;
auto& x = y; // x is an alias to y
decltype(auto) z = (y); // z is an alias to y
```

Note, we should do the same when initializing with a function

```cpp
int& f(int& i){ return ++i;};
int m=0;
auto  j = f(m);// j is int but a copy return of f
auto& k = f(m);// k is int& and an alias to return of f
decltype(auto) l = f(m); // l is int& and an alias to return of f
```

## Const

Using bare `auto`, `const` is not inferred. We have to use `auto&`, `const auto`, `const auto&` or `decltype(auto)` 

```cpp
const int m = 0;
auto j = m; // j is int
auto& k = m; // k is const int&
const auto l = m // l is const int
const auto& n = m // n is const int& 
decltype(auto) o = m // o is const int
```

## auto vs decltype(auto)

In the examples shown in this post, we understand that `auto` alone cannot convert to a constant or reference type. To do so, we should use `auto&` or `const auto`. They give the programmer more control over the declared type.

```cpp
int& f(int& i){return ++i;}
auto i = f(2);// i gets a copy of f(2)
auto& j = f(2);// j is an alias of f(2)
```

But sometimes, like a wrapper, we want the compiler to deduce the type exactly as it is:

```cpp
decltype(auto) FindTaxReturn(double& data){
    return Compute(data);
}

```

## auto vs decltype

`auto` deduces the type of a variable when declared with the help of  its initializer

```cpp
auto i = 1; // int
```

`decltype` infers the type of expression which can be used for variable declaration or injection into
a template

```cpp
int f(){return 0;}
decltype(f()) i; // i is integer
vector<decltype(f())> v; // vector<int>, cannot be done with auto
```

Note that `f()` within `decltype(f())` is not called. In fact during compilation and before the program is run, the declarations are concluded to be `int i`, `vector<int> v`. 

## Test 

A good IDE like VS Code is your best friend to assess the outcome of `auto`. However, you can check the types using metafunctions in `<type_traits>` header:

```cpp
#include <type_traits>
using namespace std;

int main(){
  auto x = true;
  cout<<is_same<decltype(x), bool>::value;
  return 0;
}
```
To ensure the type is correctly inferred, we can implement `static_assert`. It throws a compile-time
error if the type is not correct:

```cpp
static_assert(std::is_same<decltype(x), bool>::value, "x must be bool");
```

Another solution to check types is to use `typeid` as below. 

```cpp
#include <iostream>
#include <typeinfo>

int main()
{
    auto x=1.0u;
    std::cout<<typeid(x).name()<<std::endl;

    return 0;
}
```
The printed output is compiler dependent. *GCC* on my machine produces strings like *i*, *m*, or very long texts. To interpret them, use command `c++filt -t` in the terminal. So for *j* output, we run

```bash
c++filt -t j
```
It produced `unsigned int` for me. `typeid` is not as reliable as `typetraits` functions, read [here](https://en.cppreference.com/w/cpp/types/type_info/name) to for more info.


## Function

At function definition, the type can be inferred with the aid of 
its trailing return type (C++14)

```cpp
auto add(int a, int b){
    return a + b; // return type is int
}
```

Therefore, we have to guide the compiler if a function is only declared but not defined

```cpp
auto subtract(int a, int b); // Error: cannot figure out return type
auto multiply(int a, int b) -> int; // works fine 
```

The compiler must be notified when the return type is ambiguous. In the example below, the compiler cannot deduce the type of `f`, that's why we mention it as `-> double`. 

```cpp
auto f(bool cond) -> double
{
    if (cond)
        return 1;  // returns int
    else
        return 2.5; // returns double
}

```

*C++20* lets us have `auto` function arguments. The function below adds containers that have `size()` method, `[]` operator, and `+` operator for their elements. The outcome type is automatically promoted, for example, `vector<int>` plus `vector<double>` is `vector<double>`. 

```cpp
#include<iostream>
#include<vector>
#include<array>
using namespace std;

auto add (const auto& a1, const auto& a2)
{
    auto n =  a1.size();
    vector<decltype(a1[0] + a2[0])> result(n);
	for (decltype(n) i = 0; i < n; i++)
		result[i] = a1[i] + a2[i];
	return result;
}

int main(){
    
    array<int,3> a = {1,2,3};
    vector<double> b = {3.1,2.1,1.1};
    auto r = add(a,b);
    for (auto& item:r)
        cout<<item<<endl; // 4.1 4.1 4.1
    
    return 0;
}
```

## Auto in header file

A function with `auto` argument is acceptable in a header file (*C++20*). A function, which returns `auto`, must declare return type with `->` operator. The example below is on [GitHub](https://github.com/sorush-khajepor/CppExamples/tree/main/basic/example002).

```cpp
// a.h
template<class T>
struct A
{
    T x;
    auto Add(auto a)->decltype(a+x);
};
```

Note that the type of the function argument is relevant just to the function scope, but the type it returns affects the scope the function is called in. That's why the header must hint at the return type. 

In the  *.cpp* file, all the necessary classes must be instantiated:

```cpp
// compiler GCC 10.3 flag -std=c++20
// a.cpp
#include "a.h"

template<class T>
auto A<T>::Add(auto a)->decltype(a+x){
    return a+x;
}

// intantiate classes
template
auto A<double>::Add(double a)->decltype(a+x);

template
auto A<int>::Add(int a)->decltype(a+x);

// main.cpp
#include <iostream>
#include "a.h"
int main(){
  A<double> a1{.x=10};
  A<int> a2{.x=10};
  A<int> a3{.x=10};
  
  auto b = a1.Add(1.0); // OK. From header, compiler knows b is double

  std::cout<<a1.Add(20.0)<<std::endl; // OK
  std::cout<<a2.Add(20)<<std::endl; //OK
  std::cout<<a3.Add(20.0)<<std::endl; // Error: double A<int>::Add<double>(double) not found!

  return 0;
}


```

## Lambda

`auto` is perfect for specifying the type of lambda functions.

```cpp
auto l = [](int i) { return i + 1; };
```

## Templates

`auto` can infer the return type of a templated function (C++14).

```cpp
template<class T, class U>
auto add(T t, U u) 
{ return t + u; } // during compilation, type of t+u is deduced.  
```

If the return type is not clear for the compiler, we can help it with `decltype`.

```cpp
template<class T, class U>
auto f(T t, U u, bool cond) -> decltype(t+1) // tell compiler, return type is of t+1
{
    if (cond)
        return t+1;
 
    return u+1;
};
```

