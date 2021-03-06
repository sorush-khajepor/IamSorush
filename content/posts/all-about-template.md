---
title: "How and where to use C++ templates"
date: 2020-10-30T19:34:31+01:00
image: /images/hive.jpg
tags: ["C++", "Template"]
categories: "C++"
summary: "Using templates in C++, you can create functions or classes having similar behaviours for different types. Here, all the useful features of templates are explained with examples."
---

## Template

Using templates in C++, we can create a function or class template having a generic behaviour for different types. Therefore, we write and maintain less code. 

At compile time, the compiler assesses the code and can turn a class (or function) template to multiple classes (or functions). Afterwards, at run-time, the classes create objects.

## Class Template

Instead of hard-coding two similar classes, Point2dInt,

```cpp
class Point2dInt{
    
    int data[2];
    
    public:
    int& operator[] (size_t i){
        return data[i];
    }
};
```

and Point2dString,

```cpp
class Point2dString{
    
    string data[2];
    
    public:
    string& operator[] (size_t i){
        return data[i];
    }
};
```

we can have one class template with a generic type which can be `int`, `string`, and so on.

```cpp
#include <iostream>
template<class T>
class Point2D{
    
    T data[2];
    
    public:
    T& operator[] (size_t i){
        return data[i];
    }
};
```
If the compiler only sees the above code, it doesn't instantiate any Point2D class. 

```cpp
using namespace std;
int main()
{
  Point2D<int> a; // T = int
  a[0] = 1; a[1]=2;
  cout<< a[0] << a[1] << endl; //12

  Point2D<string> b; // T = string
  b[0] = "X"; b[1] = "Y";
  cout<< b[0] << b[1] << endl; //XY
}
```

However, when the compiler sees the above code, it instantiates two classes `Point2D<int>` and `Point2D<string>` which are similar to `point2dInt` and `point2dString`. 


Values, which are needed at compile time, can be injected into a template. If we want a C-style array as the storage for the data, its size is needed at compile time

```cpp
#include <iostream>

template<class T, size_t size>
class FixedArray{
  T data[size];
public:
  T& operator[] (size_t i){
    return data[i];
  }
};

int main()
{
  FixedArray<int,5> a;
  a[0] = 8;
  std::cout<< a[0] << std::endl; // 8
}
```


You can find more info about creating arrays in [this post](https://iamsorush.com/posts/create-arrays/).

## Function template

A function template can also be created which is dependent on a generic type

```cpp
#include<iostream>

template<class T>
void Swap(T& a, T& b){
    auto tmp = a;
    a = b;
    b = tmp;  
}

int main()
{
    int a =0;
    int b =1;
    Swap(a,b);
    std::cout<< a << b << std::endl; // 10
    
    std::string x ="Hi ";
    std::string y ="Jack ";
    Swap(x,y);
    std::cout<< x << y << std::endl; // Jack Hi
}
```
For more info about `auto` keyword in this example, see [this post](https://iamsorush.com/posts/auto-cpp/).

A function template can also be dependent on a compile-time value

```cpp
#include<iostream>
#include <array>
using namespace std;

template<class T, size_t n> // n is a compile-time value
T dot(array<T, n> a, array<T,n> b){
  
  T sum = a[0]*b[0];
  for (size_t i=1;i<n;i++){
    sum += a[i]*b[i];
  }
  return sum;
}

int main()
{
  array<int,2> v1 = {3,2};
  array<int,2> v2 = {2,3};
  cout<< dot<int,2>(v1,v2) <<endl; // 12
}

```


## Call function template explicitly

In the previous examples, compiler figures type `T` out. But it gets confused for this case:

```cpp
using namespace std;
int main()
{
  int a=2;
  double b=4;
  cout<< Min(a,b) << endl; // Error: T is int or double?
}
```
To solve this, template type needs to be mentioned explicitly:

```cpp
cout<< Min<int>(a,b) << endl;
```

Another example that the compiler cannot figure out the type is:

```cpp
template<class T>
void MyCast(double d){
    T a = (T) d;
    std::cout<< a << std::endl;
}

int main()
{
  MyCast(2.3); // Error: So what is T?
  MyCast<int>(2.3); // 2  OK, T is int.
}
```

## Template specialization

Sometimes the details of a template class/function need to be special for a type. For example, we define a special calculation for type `string` of `Min` function: 

```cpp
template<class T>
T Min(T a, T b){
  return a>=b?b:a;
}

template<>
string Min<string>(string a, string b){
    auto m = a.length();
    auto n = b.length();
    return    m>=n?b:a;
}

int main()
{
  int a=2;
  int b=4;
  string x = "Hello";
  string y = "Hi";
  std::cout<< Min(a,b) << std::endl;// 2
  std::cout<< Min(x,y) << std::endl; // Hi
}
```

## Default types

A template parameter can have a default type

```cpp
#include<iostream>
using namespace std;

template <typename T = int>
struct A{
    T x=10.5;
};
int main(){
    A<> a; // T = int
    cout<< a.x; // 10
    
    return 0;
}
```
Since *C++17*, it is not necessary to mention empty brackets:
```cpp
A a; // equals to A<int> a;
```


## Definition & declaration files

It's a good practice to separate a class definition (implementation) file from its declaration file. However, a problem is that the compiler creates a class out of a template only when it sees a template specialization or a class instantiation. 

```cpp

//file sample.h
template<class T>
struct Sample
{
	T Compute();
};

//file sample.cpp
#include "sample.h"

template<class T>
T Sample<T>::Compute(){

	return T();
}

//file main.cpp
#include "sample.h"
using namespace std;

int main()
{
	Sample<int> s;
	
	std::cout << s.Compute();
	return 0;
}

```

*sample.cpp* is successfully compiled to sample.o, but it doesn't have a clue that, in *main.cpp*, `Sample<int>` is used, so the compiler does not create that special class. Therefore, when *main.cpp* is compiled with *sample.o* we get a linking error. 


The easiest solution is to put both declaration and definition in the same file, *sample.h*. But it increases the size of the executable file and surpasses capabilities that separation of declaration and definition brings like solving circular dependencies. 

The second solution is to inform the compiler that we need that special class

```cpp
// file sample.cpp

#include "sample.h"

template<class T>
T Sample<T>::Compute(){

	return T();
}

// This line is the key!
template class Sample<int>;

```

Now compiling *sample.cpp*, we get a *sample.o* containing `Sample<int>` class.

To instantiate a function template, see example below:

```cpp
// abs.cpp
#include<vector>
using namespace std;

template<class T>
auto Abs (const T& a)
{
    size_t n =  a.size();
    vector<decltype(a[0]+a[1])> result(n); // decltype of prvalue to remove const &
	for (decltype(n) i = 0; i < n; i++)
		result[i] = a[i]>0? a[i]: -a[i];
	return result;
}

// This is the key for function instantiation 
// in the object file
template auto Abs(const vector<int>& a) ;

```
## Store types & parameters

The types and parameters used in a template can be stored in the class by `using` and `static` variables:

```cpp
template<size_t dim, class T>
struct Specs{
    static const size_t Dim = dim; // store dim
    using Type = T; // store T
};
```

In another scope, we have access to this info

```cpp

template<class S>
struct Particle{
    typename S::Type Position[S::Dim];// access S::Type & S::Dim
};

template<class S>
struct Box{
    typename S::Type Extent[S::Dim]; // access S::Type & S::Dim
};
```

Note that `typename` is necessary to let the compiler know `S::Type` is a type. In `Particle` and `Box`, We can also create an alias for `Specs` type:

```cpp
using Type = typename S::Type;
Type Position[S::Dim];
```

And the implementation is:

```cpp
int main(){
    using specs = Specs<3,double>;
    Particle<specs> p;
    Box<specs> b;
    cout<< sizeof(p)<<" "<<sizeof(b)<<endl; //24 24
}
```



Note that storing template types in a class is easier to read than nested templates:

```cpp
template <typename U, template <typename> class T>
struct domain{...}
```

## Type Constraints

We can limit the types that a template can take using `static_assert`, `std::is_same`, and `std::is_base_of` :

```cpp
#include <type_traits>

template<class T>
class Foo
{
	Foo() {
		static_assert(std::is_same<T, int>::value, "T must be int");
	}
};

int main()
{
  Foo<int> a; // works fine
  Foo<bool> b; // Error: T must be int

	return 0;
}
```

From *C++20*, we have `concept` which systematically constrains a template

```cpp
#include <iostream>
#include <vector>
#include <array>
using namespace std;

template<class T>
concept IsVector = requires (T x) {
    // below lines must be valid expressions
    x.size(); // size() is defined.
    x[0]+x[0]; // addable elements
    x.push_back(0); // push_back is defined
}; 
 
template<class T> 
requires IsVector<T> // imposing IsVector concept 
T add(T a, T b) {
    T c(a.size());
    for (size_t i=0; i<a.size(); i++)
        c[i]=a[i]+b[i];    
    return c; 
}

int main(){
    vector<int> x={1,2};
    vector<int> y={2,1};
    auto z = add(x,y);
    cout<<z[0]<<" "<<z[1]; // 3, 3
    
    array<int,2> m={3,4};
    array<int,2> n={4,3};
    // auto k = add(m,n); // Error:: 'x.push_back(0)' is invalid
    
    // auto w = add(1,2); // Error 'x.size()' is invalid
    
return 0;
}
```


## Enforcing inheritance

When inheriting from a template class, the compiler overlooks what is inherited.

```cpp
template<class T>
struct Animal{
  void Move(){}
};

template<class T>
struct Dog: Animal<T>{
  void Run(){
    Move(); // Error: Move() no defined!
  }
};
```

In the example above, when `Dog` is compiled, the compiler overlooks the content of `Animal` because it is 
dependent on `T`. Therefore, it has no idea of `Move()` coming from `Animal`. To fix that, call the method using `this` to defer the check until template instantiation:

```cpp
this->Move();
```

Another fix is to declare the method:

```cpp
template<T>
struct Dog: Animal{
  using Animal<T>::Move;
// rest...
}
```
## Type Alias

Since *C++11*, we can define a type alias for hard to read names

```cpp
template<T>
using VecPoint = vector<Point<T>>
```
See [here](https://iamsorush.com/posts/how-use-cpp-raw-pointer/#owner-convention), I created an alias for pointers to have an ownership convention.

## Metaprogramming

Metaprogramming is to write a program that writes another program. Here in C++, we use metaprogramming to create desired classes and functions at compile time. Moreover, we can target compile-time values and types. For concluding values, 
I recommend before resorting to templates, have a look at `constexpr` which is clean and easy to read. 

We can assess and conclude types using templates, for example, `const` qualifier can be dropped by

```cpp
#include<iostream>
#include <type_traits>

// generic type
template<typename T> 
struct RemoveConst{ 
    typedef T type;             
};

// const specialization
template<typename T> 
struct RemoveConst<const T> { 
    typedef T type;               
};

int main(){    
    std::cout<<std::is_same<int, RemoveConst<int>::type>::value; // true
    std::cout<<std::is_same<int, RemoveConst<const int>::type>::value;  // true
    return 0;
}
```
The metafunction above is defined in the standard library as `std::remove_const`. In most cases, metafunctions  defined in  `<type_traits>` header such as `true_type`, `false_type`, `conditional`, `is_same` along with `decltype`, `if constexpr` and `static_assert`  meet our metaprogramming needs. 

## Practical Cases

### Customize STL Containers

Templates are mostly used for creating custom containers. In the below example,  a vector is created that can print its items on the screen. The type of items can be any that supports `cout<<` command.

```cpp
#include <iostream>
#include <vector>
using namespace std;

template<class T>
class PrintableVector{
    vector<T> data;
    public:
    PrintableVector(std::initializer_list<T> list){
        data.assign(list);
    }
    void Print(){
      for (auto& item: data)
        cout<< item<<" ";
    }
};
int main() {
   PrintableVector<int> a({1,3,5,7});
   a.Print(); // 1 3 5 7
   return 0;
}
```

A more intricate example can be found [on GitHub](https://github.com/sorush-khajepor/Sarray), where I created a generic array that supports: +, -, *, /, dot product and so on.

### Multidimensional space

Templates can helps us design a program with different dimensions with the same code. The below code, defines `Point` in 1D, 2D, ..., and nD dimensions:

```cpp
#include <iostream>
#include <vector>
using namespace std;

template<class T, size_t size>
class Point{
    T data[size];
    public:
    Point(){
        for (auto& item: data)
            item = T();
    }
    Point(std::initializer_list<T> list){
        size_t i=0;
        for (auto& item: list){
            data[i] = item;
            i++;
            if (i>=size) break;
        }
    }
    Point GetDistanceTo(Point& point){
        Point distance;
        for (size_t i=0;i<size;i++){
            distance[i] = data[i] - point[i];
        }
        return distance;
    }
    T& operator[] (size_t i){
        return data[i];
    }
    void Print(){
      for (auto& item: data)
        cout<< item<<" ";
      cout<<endl;
    }
};
int main() {
  
   Point<int,3> a({2,2,2}); // 3D point
   Point<int,3> origin({0,0,0}); // 3D point
   
   Point<int,2> m({5,5}); // 2D point
   Point<int,2> n({4,4}); // 2D point
   
   a.GetDistanceTo(origin).Print(); // 2 2 2
   m.GetDistanceTo(n).Print();// 1 1
return 0;
}
```
### Interface

When we know the general behaviour of a class and we want that behaviour to be implemented, an interface can be useful. The interface becomes more flexible if it uses a template. 

A rough example is shown below. We have a set of solver classes that inherit from `ISolver` interface. They are always run with that interface. The outcome of these solvers needs to be visualized with `Visualizer` class. However, their outcomes are different from each other: string, double, vector<int>, and so on. 

To connect visualizers and solvers, we define `Message` template class that represents the outcome of solvers. `Visualizer` class template is also created which is specialized concerning message types. Finally, the execution and visualization of different types are handled within a simple and generic function, `RunAndDisplay()`.

```cpp
#include <iostream>
using namespace std;

template<class T>
struct Message{
    Message(T _content):content(_content){}
    virtual T GetContent() {return content;}
    private:
    T content;
};

template<class T>
struct ISolver{
    virtual Message<T> Run()=0;
};

template<class T>
struct Visualizer{
    void Show(Message<T> message){};
};
template<>
struct Visualizer<string>{
    void Show(Message<string>& message){
        cout<<"String Message: "<<message.GetContent()<<endl;
    };
};
template<>
struct Visualizer<double>{
    void Show(Message<double> message){
        cout<<"Number Message: "<<message.GetContent()<<endl;
    };
};

struct SolverA: ISolver<string>{
    Message<string> Run() override{
        Message<string> m("Solver A is Run.");
        return m;
    }
};
struct SolverB: ISolver<double>{
    Message<double> Run() override{
        Message<double> m(3.14);
        return m;
    }
};
template<class T>
void RunAndDisplay(ISolver<T>* solver){
    auto m = solver->Run();
    Visualizer<T> v;
    v.Show(m);
}
int main() {
   
   ISolver<string>* solverA = new SolverA();
   ISolver<double>* solverB = new SolverB();
   
   RunAndDisplay(solverA);
   RunAndDisplay(solverB);
return 0;
}

```

## References

[isocpp.org](https://isocpp.org/wiki/faq/templates#fn-templates)
[cppreference.com](https://en.cppreference.com/w/cpp/language/templates)

