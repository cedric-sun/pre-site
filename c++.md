ODR
============
odr-used is a property of a (an)
- variable
- structured binding
- `*this`
- virtual member function
- function
- non-placement allocation or deallocation function for a class
- an assignment operator function in a class
- constructor for a class
- destructor for a class
- local entity (in a declarative region)

that is used in an expression in such a way that requires it to have exactly 1 definition across the whole program.

VALUE
====================
7.1.1   An expression is a sequence of operators and operands that specifies a computation. An expression can result in a value and can cause side effects.

Value is one of the 13 entities.

A value is the result of evaluating an expression.

It doesn't make sense to take the address of 3+5,

// and there is absolute no reason for compiler to arrange storage for such a result. At runtime the wildest thing compiler can write for you is just call the ADD instruction and put the result in the EAX register of x86_64.
```c++
int *p = &(3+5) // WTF
```

l/rvalue ref
======
non-const lvalue ref can only bind to lvalue expression
const lvalue ref can bind to prvalue ref
rvalue ref can only bind to rvalue expression

Interchangability of Reference and Object
=======================
7.2.2.1
If an expression initially has the type “reference to T” (9.3.3.2, 9.4.3), the type is adjusted to T prior to any further analysis.

That's why when you request an object but get a reference, everything works fine.
TODO:----
```
void foo(int c){}

int main() {
    int a = 42;
    int &b = a; // variable b has type "reference to int"
    foo(b); // expression `b` has type `int`
}
```

push_back and emplace_back
================
The signature of the only `push_back` in ancient c++98
```c++
void push_back (const value_type& val);
```

The parameter has to be a const reference, cuz that time only const reference can bind to a int literal (formally, the result of a rvalue expression)
```
vector<int> x;
int a = 0;
x.push_back(a); // #1
x.push_back(42); // #2 would be weird if we can't do this
```

This effectively create a 4 byte storage one the calling stack frame storing this 42, and bind a reference on `push_back`'s call stack to that storage.

Now, to officially put this 42 to the end of the storage of the vector, libstdc++ needs to copy that temporary 4 bytes again. libstdc++ developers cannot make assumption about how this reference is obtained, i.e. inside `push_back`, they can't tell #1 from #2.  Now replace int with any type this argument is still true.

With the introduction of rvalue reference in C++11, things changed. We now have a way in language level to tell such differences. Rvalue reference (even without const) can bind to literal (formally ... ).  It's an error to bind a rvalue reference to the result of a lvalue expression. The value category of the argument expression now join the overload resolution, and only the overload with a rvalue reference parameter will be matched and called. The libstdc++ developers now have a way to tell if the reference get passed in is binding to that temporary 4 byte storage on the calling stack, or is it a formal object living somewhere. This gives us the second signature added in C++11:
```
void push_back( T&& value );
```


The newly created temporary 42 is directly moved into the vector storage. No redundant copy.

value category of function call expression 
===========
Expression that call to a function that return a lvalue reference is lvalue expression:
```
int& foo() {
    int *c = new int{42};
    return *c;
}

int main() {
    foo() = 77;
}
```

Expression that call to a function that return a non-reference is prvalue expression:
```
struct X { int a; };

X foo() {
    return X{42};
}

int main() {
    X c = foo();
}
```

Expression that call to a function that return a rvalue reference is a xvalue expression:
```
TODO
```

What does it mean for a function to return a rvalue reference;


const member function
============
```
#include <cstdio>

struct X{
    void foo() const {
        puts("foo const");
    }

    void foo() {

        puts("foo non-const");
    }
};


int main() {
    X x;
    const X cx;
    x.foo();
    cx.foo();
}
```

TODO:: REALLY? A NON CONST object should be able to call a CONST
member function.

`void foo() const` can only be called on a const qualified `foo` instance. `const` effectively take part in the overload resolution.

Inside the `const` qualified member function, `*this` is const qualified, which means among other member functions, only `const` qualified member functions can be called;

Really, "const member function promised not to change its object", is a consequence, not its motivation.


https://en.cppreference.com/w/cpp/language/overload_resolution

> If any candidate function is a member function (static or non-static), but not a constructor, it is treated as if it has an extra parameter (implicit object parameter) which represents the object for which they are called and appears before the first of the actual parameters.

Similarly, the object on which a member function is being called is prepended to the argument list as the implied object argument.

For member functions of class X, the type of the implicit object parameter is affected by cv-qualifications and ref-qualifications of the member function as described in member functions.


TODO: see ref-qualified member funciton, and relate to the implied object parameter/argument as well.


FUCKING COPY-ELISION AND RVO
===============



=========================================================


rvalue (prvalue and xvalue) expression as argument of a function call will match function overloads that takes a rvalue reference.


`decltype<t> t2 = std::move(t)` return (cast t to?) a rvalue reference so the expression itself is a rvalue expression, so the fucking move assignment operator get fucking fired instead of the plain old copy assignment opeartor.

done.


```c++
#include <cstdio>
void foo(int &x) {
    printf("%s\n", "lv ref");
}

void foo(int &&x) {
    printf("%s\n", "RVALUE ref");
}

int &lv() {
    int *c = new int{42};
    return *c;
}

int &&rv() {
    return 3+2;
}

int main() {
    foo(lv()); // lv ref
    foo(rv()); // RVALUE ref
    int &&gg = rv();
    foo(gg); // lv ref
}
```

Rule of 3
=========================================================

Rule of 5 ? 4.5?
=========================================================
https://stackoverflow.com/questions/45754226/what-is-the-rule-of-four-and-a-half
https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom

copy and swap ? pre c++11 ?
=========================================================
copy constructor / copy assignment operator pass by value?


if xvalue expression fire rv-ref overload, and prvalue expression fire rv-ref overload too, why differentiating these two?
==============


use `make_(unique|shared)` helpers instead of smart pointer constructor
============
https://stackoverflow.com/a/22571331/8311608


Overload a operator as a non-member function and declare it as friend to grant access to internal data members
https://stackoverflow.com/questions/2828280/friend-functions-and-operator-overloading-what-is-the-proper-way-to-overlo

The expression after the return keyword is implicit treated as a rvalue
=====================
https://stackoverflow.com/questions/4986673/c11-rvalues-and-move-semantics-confusion-return-statement
"Please do not return local variables by reference, ever. An rvalue reference is still a reference."


Rare case when return a rvalue ref is useful
========
https://stackoverflow.com/questions/5770253/is-there-any-case-where-a-return-of-a-rvalue-reference-is-useful


Member initialization list - initialization order
==============
The order of member initializers in the list is irrelevant.

non-static data member are initialized in order of declaration in the class definition.
```
struct A {
    int a;
    int b;

    A(int x): b(x), a(b){} // warning: a(b) but b is not initializerd
}

struct B {
    int b;
    int a;

    B(int x): a(b), b(x){} // Okay
}
```

Avoid instantiating class in loop
============
```
#include <utility>
#include <vector>

using std::vector;

#include <cstdio>
#include <stdio_ext.h>

struct GG {
    GG() { puts("constructed!"); }
    ~GG() { puts("DE-structed!"); }
};

int main() {
    __fsetlocking(stdout,FSETLOCKING_BYCALLER);
    printf("sizeof(vector<int>)=%lu\n",sizeof(vector<int>));
    vector<vector<int>> x;
    for (int i=0;i<10;i++) {
        vector<int> y;
        GG _gg;
        for (int j=0;j<10;j++) {
            y.push_back((i<<8) + j);
        }
        printf("i=%d:",i);
        for (int z:y) {
            printf("%d\t",z);
        }
        putchar('\n');
        x.push_back(y);
    }
}
```

It run `GG`'s ctor and dtor, and `vector<int>`'s ctor and dtor every single loop.

reuse a moved container & the moved-from vector
============
https://stackoverflow.com/questions/9168823/reusing-a-moved-container

Technically when the moving `x.push_back()` is called (thus work on y destructively), the lifetime of `y` is far from ending; but's it's still okay to do the following. It avoid creating a vector<int> inside the loop every round. Also mind the moved-from vector problem.

```
#include <utility>
#include <vector>

using std::vector;

#include <cstdio>
#include <stdio_ext.h>


void print_vec(const vector<int> &vec) {
    for (const int x: vec) {
        printf("%d\t",x);
    }
    putchar('\n');
}

int main() {
    __fsetlocking(stdout,FSETLOCKING_BYCALLER);
    vector<vector<int>> x;
    vector<int> y;
    for (int i=0;i<10;i++) {
        y.clear(); // so call clear manually
        for (int j=0;j<10;j++) {
            y.push_back((i<<8) + j);
        }
        x.push_back(std::move(y));
        // the moved-from vector is not always empty:
        // https://stackoverflow.com/questions/17730689/is-a-moved-from-vector-always-empty
    }
    for (const auto &vec:x) {
        print_vec(vec);
    }
}
```

TODO: is reference in a "for-each for" always faster than (even) int?




c++ shift
==========
https://en.cppreference.com/w/cpp/language/operator_arithmetic#Bitwise_shift_operators

unlike java which has the unsigned right shift `>>>` operator...

TODO: show mapping between shift operators and x86_64 instructions

vector over list
==========
https://stackoverflow.com/questions/2209224/vector-vs-list-in-stl#comment20988862_2209233
https://www.stroustrup.com/bs_faq.html#list
Bjarne Strostrup actually made a test where he generated random numbers and then added them to a list and a vector respectively. The insertions were made so that the list/vector was ordered at all times. Even though this is typically "list domain" the vector outperformed the list by a LARGE margin. Reason being that memory acces is slow and caching works better for sequential data. It's all available in his keynote from "GoingNative 2012"

Same class template specialization shares the same static member
=================
And different specializations don't share.

```c++
#include <random>
#include <algorithm>
#include <benchmark/benchmark.h>

// Singleton random int array across the whole benchmark program.
// Each benchmark function will see the same random array `data[]`
template<int MAXN>
struct SingletonRandomIntArrayFixture : benchmark::Fixture {
    static int data[MAXN];
    static bool isInited;

    void SetUp(const benchmark::State& state) {
        if (!isInited) {
            for(int i=0;i<MAXN;++i) {
                data[i]=i;
            }
            auto rng= std::default_random_engine{};
            std::shuffle(data,data+MAXN,rng);
            isInited=true;
        }
    }

    void TearDown(const benchmark::State& state) {
    }
};

template<int MAXN> int SingletonRandomIntArrayFixture<MAXN>::data[MAXN];
template<int MAXN> bool SingletonRandomIntArrayFixture<MAXN>::isInited = false;

#include <cstdio>
int main() {
    printf("%lu\n",sizeof(SingletonRandomIntArrayFixture<120>::data));
    printf("%lu\n",sizeof(SingletonRandomIntArrayFixture<240>::data));
    printf("%p\n",SingletonRandomIntArrayFixture<120>::data); //           0x55df3d3db060
    printf("%p\n",SingletonRandomIntArrayFixture<120>::data); // the same: 0x55df3d3db060
    printf("%p\n",SingletonRandomIntArrayFixture<240>::data); //           0x55df3d3db240
    printf("%p\n",SingletonRandomIntArrayFixture<240>::data); // the same: 0x55df3d3db240
}
```


Execution Order Control
=======================
It's common in practise that you may want one global (namespace-level) variable get set before 


Non-usefulness of volatile in C++
==============
https://stackoverflow.com/questions/2484980/why-is-volatile-not-considered-useful-in-multithreaded-c-or-c-programming

TLDR: `volatile` variable only guarantees
1. read/write of that variable is actually from/to main memory (i.e. they penetrate the cache)
2. read/write of that variable happens in the source text order

ABSOLUTELY NO GUARANTEE ABOUT:
1. atomicity of read/write
2. forbidding instruction reordering. i.e. compiler still reorders instructions.
    "Assume that we use a volatile variable as a flag to indicate whether or not some data is ready to be read. In our code, we simply set the flag after preparing the data, so all looks fine. But what if the instructions are reordered so the flag is set first?"
3. cache flush or jvm piggyback whatever

Uninherited static data member in base class in the deadly diamond
```c++
#include <iostream>

struct A {
  static int c;
};

int A::c=42;

struct B : A {};
struct C : A{};
struct D : B,C {
  bool foo() {
                                     // nothing to do with inheritance here
    return (&B::A::c) == &(C::A::c); // this is just global namespace resolution...
  }

  static bool bar() {
    return (&B::A::c) == &(C::A::c);
  }
};


int main() {
  D d;
  std::cout<<std::boolalpha << D::bar() << d.foo();
}
```
