# C++ Notes
## from A Tour of C++

`double sum(const vector<double>&)`
Using a reference as an argument allows access without copying. `const` ensures that the argument cannot be mutated. Function declaration does not need names in argument. 

Can have an initializer in an `if` statement:

```
if (auto n = v.size(); n == 1) {
...
}
```

#### Derefernces are implicit

```
int x = 2;
int y = 3;
int &r = x;
int &r2 = y;
r = r2; // implicity deref r2, sets x == 3
```

#### Initialization 

```
int v[] = {0, 1, 2, 3} // Type is int[4]
```

`constexpr` - must be constant at compile time

Use `{}` when initalizing to avoid errors due to conversion
```
int a {2} // ok
int b {2.2} // error: floating point to integer
```

`auto` is used for type deduction, use `=` when using auto instead of `{}`

Iterate in `for` loops like so:

```
int v[] = {0, 1, 2, 3}

for (auto& v : v) {
    ++x; // x will reference an element in v
}

```

C++ `variant` is similar to algebraic data types

#### Modules / includes
Separate function declarations and implementations in .h file
#include - Old and error-prone. Slow.
modules - new, experimental

#### Namespaces

Namespaces used separate declarations
``` 
namespace my_code {
    void fun(...)
}
```
later, call: `my_code:: fun(...)`
or import `using mycode::fun`
or import entire namespace: `using namespace my_code`

#### Error handling
A function that should never throw an exception can be declares `noexept`
`new` can throw a std::bad_alloc error

Q: Should we put all `new`s in a try block?
"Well desgiend code rarely has `try` blocks"

Throw an exception only on failure that cannot be handled by the immediate caller

Q: What happens to unhandled exceptions?

Use error codes when failure is normal and expected

`static_assert` for printing compiler errors with constnat expressions

#### Function arguments and return values
Can have default arguments `void print(int value, int base = 10)` this avoids operator overloading

Return types can be deduced w/ auto:
```
auto fun(void) {
    return 5;
}
```

#### Structured binding
Allows us to unpack values from a struct
```
struct Entry {
    string name;
    int value;
}; 

auto [n,v] = read_entry(is); // read_entry returns an Entry
```
n contains a string
v contains a value

### Classes

#### Concrete types
Behave just like built-in types

Example:
```
class complex {
    double re, im; // class members, private by default
public: 
    complex(double r, double i) :re{r}, im{i} {} // Consturctor
    complex() :re{0}, im{0} {} // Defualt constructor

    double real() const { return re; } // getter
    void real(double d) const { re=d; } // setter

    complex& operator+=(complex z) // operator overloading
    {
        re += z.re; // add to re and im
        im+=z.im;
        return *this;
    }
}
```
Q: In the + operator accesses private member z.im and z.re. Why is this allowed?

Functions defined within `class` are inlined by default. Explicit inlining by preceeding function declaration with `inline`

`const` in the function declaration tells that it does not modify the object the method is called on 

#### Container
```
class Vector {
public: 
    Vector (int s): elem{new double[s]} sz{s} // constructor
    {
        for (int i = 0; i != s; ++i)
            elem[i] = 0;
    }

    ~Vector() { delete[] elem; }            // destructor
    double& operator[](int i);          // access
private:
    double* elem;
    int sz;
};
```

Because we have declared a constructor and destructor, users do not need to explicitly `delete` a `Vector`. It wll be deleted when it leaves scope, much like other built-in types. This is RAII. Allows avoiding of naked `new` operations, which should be avoided.

We can use an initializer-list constructor: `Vector(std::initializer_list<double>);` to allow us to create a vector from a list: `Vector v = {1,2,3,4,5};`

#### Abstract Types
No implementation details. Must be manipulated through pointers or references.

Used for polymorphism

Example:
```
class Container {
public: 
    virtual double& operator[](int) = 0;
    virtual int size() const = 0;
    virtual ~Container() {}
};
```

`virtual` means it may be redefined later in a class derived from this one
`= 0` means it's purely virtual, a class derived from it must implement this function
No constructor needed because it does not have data. 

Q: Why is ~Container() not a purely virtual function?

An implementing class:
```
class Vector_container : public Container {
public:
    Vector_container(int s) : v(s) {}
    ~Vector_container() {}      // Implicitly destructs its member v

    double& operator[](int i) override {return v[i];}
    int size() const override {return v.size();}
private:
    Vector v;
};
```
`:public` means its derived from, or is a subtype of `Container`

`override` makes it clear that we want to override functions from the base class. It is optional but recommended to catch misspellings.

### Virtual Functions

Convert name of virtual function into an index into a vable of pointers to functions -- called the virtual function table or `vtbl`. Each class has its own vtbl identifying virtual functions. This allows the object to be used correctly even when the size of the object is not known. Almost as efficient as a "normal function call" 

Q: From book: "Classes in class hierarchies are different: we tend to allocate them on the free store using `new`" does this mean RAII cannot be used for classes in class hierarchies? 

A function may return an abstract class, if we need to know its particular derviced class, we must be `dynamic_cast`

Example: for Tree like Shape -> Circle, we can test `if (Circle *p = dynamic_cast<Circle*>(ps))` where `ps` is a `Shape` type. `dynamic_cast` will return `nullptr` if it's not a `Circle`. If we try casting ot a reference type: `Circle& r {dynamic_cast<Circle&>(*ps)}` this will throw a `bad_cast` exception. Use this with restraint, reply more on typing infromation given. 

#### Avoiding resource leaks

We can use `unique_ptr` rather than a naked pointer to keep RAII and track ownership.
