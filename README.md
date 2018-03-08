# Appcom C++ Style Guide

This document describes the style guide applied to C++ projects for appcom interactive GmbH. It describes rules how to organize your project, dependencies and files, so that some best practises are held.

So use it in your favor if you want to and/or override the style guide in any way you want.

<a name="table-of-contents"></a>
## Table of Contents

1. [Dependencies](#dependencies)
1. [Files](#files)
1. [Scoping](#scoping)
1. [Classes](#classes)
1. [Functions](#functions)
1. [Other C++ Features](#other-cpp-features)
1. [Naming](#naming)
1. [Comments / Doxygen](#comments)
1. [Formatting](#formatting)
1. [Exceptions to the Rules](#rule-exceptions)
1. [Asset Naming Conventions](#asset-naming-conventions)
1. [Logging](#logging)
1. [Roadmap](#roadmap)
1. [Resources](#resources)
1. [License](#license)

<a name="dependencies"></a>
## [1](#dependencies) Dependencies

<a name="dependencies-library-version"></a>
### [1.1](#dependencies-library-version) Library version

Always use the latest stable version of a library if possible.
Make sure to migrate also major releases if possible.

<a name="dependencies-library-provision"></a>
### [1.2](#dependencies-library-provision) Library provision

Libraries must be included using git submodules if compiled within the project itself or using an artefact repository (like sonatype nexus) if precompiled. Provide a script for installing these if needed. The name of the script must be install-deps.sh and placed at the source root.

<a name="dependencies-folder"></a>
### [1.3](#dependencies-folder) Folder

Dependencies should be found and structured at a well known place. Therefore these must be placed under the `deps` folder at the souce root. Precompiled libraries must be placed under the subfolder `lib` and the corresponding header files must be under the subfolder `include`.

**[back to top](#table-of-contents)**

<a name="files"></a>
## [2](#files) Files

<a name="files-file-names"></a>
### [2.1](#files-file-names) File Names

File names must reflect the name of the class implementation that they contain, including case.
Follow the convention that your project uses. File extensions must be as follows:

| Extension | Type |
|---|---|
| .h | C header file |
| .hpp | C++ header file |
| .c | C implementation file |
| .cpp | C++ implementation file |

Files containing code that may be shared across projects or used in a large project must have a clearly unique name, typically including the project.

<a name="files-file-encoding"></a>
### [2.2](#files-file-encoding) File encoding: UTF-8

Source files are encoded in UTF-8.

<a name="files-whitespace-characters"></a>
### [2.3](#files-whitespace-characters) Whitespace characters

Aside from the line terminator sequence, the ASCII horizontal space character (0x20) is the only whitespace character that appears anywhere in a source file.

This implies that:

* All other whitespace characters in string and character literals are escaped.
* Tab characters are not used for indentation.

<a name="files-header-file-structure"></a>
### [2.4](#files-header-file-structure) Header file structure

A header file consists of, *in order*:

* License or copyright information, if present
* An include guard (see [Header include guard](#files-header-include-guard))
* Include statements
* Forward declarations
* At most one top-level class

<a name="files-header-include-guard"></a>
### [2.5](#files-header-include-guard) Header include guard

All header files must use `#pragma once` to prevent multiple inclusion.

<a name="files-forward-declarations"></a>
### [2.6](#files-forward-declarations) Forward Declarations

Avoid using forward declarations where possible. Just `#include` the headers you need.

**Definition:**

A "forward declaration" is a declaration of a class, function, or template without an associated definition.

**Pros:**

* Forward declarations can save compile time, as #includes force the compiler to open more files and process more input.
* Forward declarations can save on unnecessary recompilation. #includes can force your code to be recompiled more often, due to unrelated changes in the header.

**Cons:**

* Forward declarations can hide a dependency, allowing user code to skip necessary recompilation when headers change.
* A forward declaration may be broken by subsequent changes to the library. Forward declarations of functions and templates can prevent the header owners from making otherwise-compatible changes to their APIs, such as widening a parameter type, adding a template parameter with a default value, or migrating to a new namespace.
* Forward declaring symbols from namespace `std::` yields undefined behavior.
* It can be difficult to determine whether a forward declaration or a full `#include` is needed. Replacing an `#include` with a forward declaration can silently change the meaning of code:

```
// B.hpp:
struct B {};
struct D : B {};

// Good 
// User.cpp:
#include "B.hpp"
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // calls f(B*)
```

If the `#include` was replaced with forward decls for `B` and `D`, `test()` would call `f(void*)`.
* Forward declaring multiple symbols from a header can be more verbose than simply `#include`ing the header.
* Structuring code to enable forward declarations (e.g. using pointer members instead of object members) can make the code slower and more complex.

**Decision:**

Try to avoid forward declarations of entities defined in another project.
When using a function declared in a header file, always `#include` that header.
When using a class template, prefer to `#include` its header file.

<a name="files-inline-functions"></a>
### [2.7](#files-inline-functions) Inline Functions

Define functions inline only when they are small, say, 10 lines or fewer.

**Definition:**

You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism.

**Pros:**

Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.

**Cons:**

Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease. Inlining a very small accessor function will usually decrease code size while inlining a very large function can dramatically increase code size. On modern processors smaller code usually runs faster due to better use of the instruction cache.

**Decision:**

A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.

<a name="files-source-file-structure"></a>
### [2.8](#files-source-file-structure) Source file structure

A source file consists of, *in order*:

* License or copyright information, if present
* Include statements
* Exactly one top-level class

**[back to top](#table-of-contents)**

<a name="scoping"></a>
## [3](#scoping) Scoping

<a name="scoping-namespaces"></a>
### [3.1](#scoping-namespaces) Namespaces

With few exceptions, place code in a namespace. Namespaces should have unique names based on the project name, and possibly its path. Do not use using-directives (e.g. using namespace foo). Do not use inline namespaces. For unnamed namespaces, see Unnamed Namespaces and Static Variables.

**Definition:**

Namespaces subdivide the global scope into distinct, named scopes, and so are useful for preventing name collisions in the global scope.

**Pros:**

Namespaces provide a method for preventing name conflicts in large programs while allowing most code to use reasonably short names.

For example, if two different projects have a class Foo in the global scope, these symbols may collide at compile time or at runtime. If each project places their code in a namespace, project1::Foo and project2::Foo are now distinct symbols that do not collide, and code within each project's namespace can continue to refer to Foo without the prefix.

Inline namespaces automatically place their names in the enclosing scope. Consider the following snippet, for example:

```
namespace X 
{
inline namespace Y 
{
  void foo();
}  // namespace Y
}  // namespace X
```

The expressions X::Y::foo() and X::foo() are interchangeable. Inline namespaces are primarily intended for ABI compatibility across versions.

**Cons:**

Namespaces can be confusing, because they complicate the mechanics of figuring out what definition a name refers to.

Inline namespaces, in particular, can be confusing because names aren't actually restricted to the namespace where they are declared. They are only useful as part of some larger versioning policy.

In some contexts, it's necessary to repeatedly refer to symbols by their fully-qualified names. For deeply-nested namespaces, this can add a lot of clutter.

**Decision:**

Namespaces should be used as follows:

* Follow the rules on Namespace Names.
* Terminate namespaces with comments as shown in the given examples.
* Namespaces wrap the entire source file after includes, [gflags](https://gflags.github.io/gflags/) definitions/declarations and forward declarations of classes from other namespaces.

```
// In the .hpp file
namespace mynamespace 
{
// All declarations are within the namespace scope.
// Notice the lack of indentation.
class MyClass 
{
 public:
  ...
  void Foo();
};

}  // namespace mynamespace
```

```
// In the .cpp file
namespace mynamespace {

// Definition of functions is within scope of the namespace.
void MyClass::Foo() {
  ...
}

}  // namespace mynamespace
```

More complex .cpp files might have additional details, like flags or using-declarations.

```
#include "a.hpp"

DEFINE_FLAG(bool, someflag, false, "dummy flag");

namespace a 
{
using ::foo::bar;

...code for a...         // Code goes against the left margin.

}  // namespace a
```

* Do not declare anything in namespace std, including forward declarations of standard library classes. Declaring entities in namespace std is undefined behavior, i.e., not portable. To declare entities from the standard library, include the appropriate header file.
* You may not use a using-directive to make all names from a namespace available.

```
// Forbidden -- This pollutes the namespace.
using namespace foo;
```

* Do not use Namespace aliases at namespace scope in header files except in explicitly marked internal-only namespaces, because anything imported into a namespace in a header file becomes part of the public API exported by that file.

```
// Shorten access to some commonly used names in .cpp files.
namespace baz = ::foo::bar::baz;
```

```
// Shorten access to some commonly used names (in a .hpp file).
namespace librarian 
{
// Internal, not part of the API.
namespace impl
{
namespace sidetable = ::pipeline_diagnostics::sidetable;
}  // namespace impl

inline void my_inline_function() 
{
  // namespace alias local to a function (or method).
  namespace baz = ::foo::bar::baz;
  ...
}
}  // namespace librarian
```

* Do not use inline namespaces.

<a name="scoping-unnamed-namespaces"></a>
### [3.2](#scoping-unnamed-namespaces) Unnamed Namespaces and Static Variables

When definitions in a .cpp file do not need to be referenced outside that file, place them in an unnamed namespace or declare them static. Do not use either of these constructs in .hpp files.

**Definition:**

All declarations can be given internal linkage by placing them in unnamed namespaces, and functions and variables can be given internal linkage by declaring them static. This means that anything you're declaring can't be accessed from another file. If a different file declares something with the same name, then the two entities are completely independent.

**Decision:**

Use of internal linkage in .cpp files is encouraged for all code that does not need to be referenced elsewhere. Do not use internal linkage in .hpp files.

Format unnamed namespaces like named namespaces. In the terminating comment, leave the namespace name empty:

```
namespace 
{
...
}  // namespace
```

<a name="scoping-global-functions"></a>
### [3.3](#scoping-global-functions) Nonmember, Static Member, and Global Functions

Prefer placing nonmember functions in a namespace; use completely global functions rarely. Prefer grouping functions with a namespace instead of using a class as if it were a namespace. Static methods of a class should generally be closely related to instances of the class or the class's static data.

**Pros:**

Nonmember and static member functions can be useful in some situations. Putting nonmember functions in a namespace avoids polluting the global namespace.

**Cons:**

Nonmember and static member functions may make more sense as members of a new class, especially if they access external resources or have significant dependencies.

**Decision:**

Sometimes it is useful to define a function not bound to a class instance. Such a function can be either a static member or a nonmember function. Nonmember functions should not depend on external variables, and should nearly always exist in a namespace. Rather than creating classes only to group static member functions which do not share static data, use namespaces instead. For a header myproject/foo_bar.hpp, for example, write

```
namespace myproject
{
namespace foo_bar
{
void Function1();
void Function2();
}  // namespace foo_bar
}  // namespace myproject
```

instead of

```
namespace myproject
{
class FooBar
{
  public:
    static void Function1();
    static void Function2();
};
} // namespace myproject
```

If you define a nonmember function and it is only needed in its .cpp file, use internal linkage to limit its scope.

<a name="scoping-local-variables"></a>
### [3.4](#scoping-local-variables) Local Variables

Place a function's variables in the narrowest scope possible, and initialize variables in the declaration.

C++ allows you to declare variables anywhere in a function. We encourage you to declare them in as local a scope as possible, and as close to the first use as possible. This makes it easier for the reader to find the declaration and see what type the variable is and what it was initialized to. In particular, initialization should be used instead of declaration and assignment, e.g.:

```
// bad - initialization separate from declaration.
int i;
i = f();
```

```
// good - declaration has initialization.
int j = g();
```

```
std::vector<int> v;
v.push_back(1);  // Prefer initializing using brace initialization.
v.push_back(2);
```

```
// good - v starts initialized.
std::vector<int> v = {1, 2};
```

Variables needed for if, while and for statements should normally be declared within those statements, so that such variables are confined to those scopes. E.g.:

```
while (const char* p = strchr(str, '/')) str = p + 1;
```

There is one caveat: if the variable is an object, its constructor is invoked every time it enters scope and is created, and its destructor is invoked every time it goes out of scope.

```
// Inefficient implementation:
for (int i = 0; i < 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}
```

It may be more efficient to declare such a variable used in a loop outside that loop:

```
Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i < 1000000; ++i) {
  f.DoSomething(i);
}
```

<a name="scoping-global-variables"></a>
### [3.5](#scoping-global-variables) Static and Global Variables

Variables of class type with static storage duration are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are constexpr: they have no dynamic initialization or destruction.

Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.

The order in which class constructors and initializers for static variables are called is only partially specified in C++ and can even change from build to build, which can cause bugs that are difficult to find. Therefore in addition to banning globals of class type, we do not allow non-local static variables to be initialized with the result of a function, unless that function (such as getenv(), or getpid()) does not itself depend on any other globals. However, a static POD variable within function scope may be initialized with the result of a function, since its initialization order is well-defined and does not occur until control passes through its declaration.

Likewise, global and static variables are destroyed when the program terminates, regardless of whether the termination is by returning from main() or by calling exit(). The order in which destructors are called is defined to be the reverse of the order in which the constructors were called. Since constructor order is indeterminate, so is destructor order. For example, at program-end time a static variable might have been destroyed, but code still running — perhaps in another thread — tries to access it and fails. Or the destructor for a static string variable might be run prior to the destructor for another variable that contains a reference to that string.

One way to alleviate the destructor problem is to terminate the program by calling quick_exit() instead of exit(). The difference is that quick_exit() does not invoke destructors and does not invoke any handlers that were registered by calling atexit(). If you have a handler that needs to run when a program terminates via quick_exit() (flushing logs, for example), you can register it using at_quick_exit(). (If you have a handler that needs to run at both exit() and quick_exit(), you need to register it in both places.)

As a result we only allow static variables to contain POD data. This rule completely disallows std::vector (use C arrays instead), or string (use const char []).

If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once(). Note that this must be a raw pointer, not a "smart" pointer, since the smart pointer's destructor will have the order-of-destructor issue that we are trying to avoid.

**[back to top](#table-of-contents)**

<a name="classes"></a>
## [4](#classes) Classes

Classes are the fundamental unit of code in C++. Naturally, we use them extensively. This section lists the main dos and don'ts you should follow when writing a class.

<a name="classes-constructors"></a>
### [4.1](#classes-constructors) Doing Work in Constructors

Avoid virtual method calls in constructors, and avoid initialization that can fail if you can't signal an error.

**Definition:**

It is possible to perform arbitrary initialization in the body of the constructor.

**Pros:**

* No need to worry about whether the class has been initialized or not.
* Objects that are fully initialized by constructor call can be const and may also be easier to use with standard containers or algorithms.

**Cons:**

* If the work calls virtual functions, these calls will not get dispatched to the subclass implementations. Future modification to your class can quietly introduce this problem even if your class is not currently subclassed, causing much confusion.
* There is no easy way for constructors to signal errors, short of crashing the program (not always appropriate) or using exceptions (which are forbidden).
* If the work fails, we now have an object whose initialization code failed, so it may be an unusual state requiring a bool IsValid() state checking mechanism (or similar) which is easy to forget to call.
* You cannot take the address of a constructor, so whatever work is done in the constructor cannot easily be handed off to, for example, another thread.

**Decision:**

Constructors should never call virtual functions. If appropriate for your code, terminating the program may be an appropriate error handling response. Otherwise, consider a factory function or Init() method. Avoid Init() methods on objects with no other states that affect which public methods may be called (semi-constructed objects of this form are particularly hard to work with correctly).

<a name="classes-implicit-conversion"></a>
### [4.2](#classes-implicit-conversion) Implicit Conversions

Do not define implicit conversions. Use the `explicit` keyword for conversion operators and single-argument constructors.

**Definition:**

Implicit conversions allow an object of one type (called the source type) to be used where a different type (called the destination type) is expected, such as when passing an int argument to a function that takes a double parameter.

In addition to the implicit conversions defined by the language, users can define their own, by adding appropriate members to the class definition of the source or destination type. An implicit conversion in the source type is defined by a type conversion operator named after the destination type (e.g. `operator bool()`). An implicit conversion in the destination type is defined by a constructor that can take the source type as its only argument (or only argument with no default value).

The `explicit` keyword can be applied to a constructor or (since C++11) a conversion operator, to ensure that it can only be used when the destination type is explicit at the point of use, e.g. with a cast. This applies not only to implicit conversions, but to C++11's list initialization syntax:

```
class Foo 
{
  explicit Foo(int x, double y);
  ...
};

void Func(Foo f);
```

```
Func({42, 3.14});  // Error
```

This kind of code isn't technically an implicit conversion, but the language treats it as one as far as `explicit` is concerned.

**Pros:**

* Implicit conversions can make a type more usable and expressive by eliminating the need to explicitly name a type when it's obvious.
* Implicit conversions can be a simpler alternative to overloading.
* List initialization syntax is a concise and expressive way of initializing objects.

**Cons:**

* Implicit conversions can hide type-mismatch bugs, where the destination type does not match the user's expectation, or the user is unaware that any conversion will take place.
* Implicit conversions can make code harder to read, particularly in the presence of overloading, by making it less obvious what code is actually getting called.
* Constructors that take a single argument may accidentally be usable as implicit type conversions, even if they are not intended to do so.
* When a single-argument constructor is not marked explicit, there's no reliable way to tell whether it's intended to define an implicit conversion, or the author simply forgot to mark it.
* It's not always clear which type should provide the conversion, and if they both do, the code becomes ambiguous.
* List initialization can suffer from the same problems if the destination type is implicit, particularly if the list has only a single element.

**Decision:**

Type conversion operators, and constructors that are callable with a single argument, must be marked explicit in the class definition. As an exception, copy and move constructors should not be explicit, since they do not perform type conversion. Implicit conversions can sometimes be necessary and appropriate for types that are designed to transparently wrap other types. In that case, contact your project leads to request a waiver of this rule.

Constructors that cannot be called with a single argument should usually omit explicit. Constructors that take a single std::initializer_list parameter should also omit explicit, in order to support copy-initialization (e.g. `MyType m = {1, 2};`).

<a name="classes-copy-movable-types"></a>
### [4.3](#classes-copy-movable-types) Copyable and Movable Types

Support copying and/or moving if these operations are clear and meaningful for your type. Otherwise, disable the implicitly generated special functions that perform copies and moves.

**Definition:**

A copyable type allows its objects to be initialized or assigned from any other object of the same type, without changing the value of the source. For user-defined types, the copy behavior is defined by the copy constructor and the copy-assignment operator. string is an example of a copyable type.

A movable type is one that can be initialized and assigned from temporaries (all copyable types are therefore movable). std::unique_ptr<int> is an example of a movable but not copyable type. For user-defined types, the move behavior is defined by the move constructor and the move-assignment operator.

The copy/move constructors can be implicitly invoked by the compiler in some situations, e.g. when passing objects by value.

**Pros:**

Objects of copyable and movable types can be passed and returned by value, which makes APIs simpler, safer, and more general. Unlike when passing objects by pointer or reference, there's no risk of confusion over ownership, lifetime, mutability, and similar issues, and no need to specify them in the contract. It also prevents non-local interactions between the client and the implementation, which makes them easier to understand, maintain, and optimize by the compiler. Further, such objects can be used with generic APIs that require pass-by-value, such as most containers, and they allow for additional flexibility in e.g., type composition.

Copy/move constructors and assignment operators are usually easier to define correctly than alternatives like Clone(), CopyFrom() or Swap(), because they can be generated by the compiler, either implicitly or with = default. They are concise, and ensure that all data members are copied. Copy and move constructors are also generally more efficient, because they don't require heap allocation or separate initialization and assignment steps, and they're eligible for optimizations such as copy elision.

Move operations allow the implicit and efficient transfer of resources out of rvalue objects. This allows a plainer coding style in some cases.

**Cons:**

Some types do not need to be copyable, and providing copy operations for such types can be confusing, nonsensical, or outright incorrect. Types representing singleton objects (Registerer), objects tied to a specific scope (Cleanup), or closely coupled to object identity (Mutex) cannot be copied meaningfully. Copy operations for base class types that are to be used polymorphically are hazardous, because use of them can lead to object slicing. Defaulted or carelessly-implemented copy operations can be incorrect, and the resulting bugs can be confusing and difficult to diagnose.

Copy constructors are invoked implicitly, which makes the invocation easy to miss. This may cause confusion for programmers used to languages where pass-by-reference is conventional or mandatory. It may also encourage excessive copying, which can cause performance problems.

**Decision:**

Provide the copy and move operations if their meaning is clear to a casual user and the copying/moving does not incur unexpected costs. If you define a copy or move constructor, define the corresponding assignment operator, and vice-versa. If your type is copyable, do not define move operations unless they are significantly more efficient than the corresponding copy operations. If your type is not copyable, but the correctness of a move is obvious to users of the type, you may make the type move-only by defining both of the move operations.

If your type provides copy operations, it is recommended that you design your class so that the default implementation of those operations is correct. Remember to review the correctness of any defaulted operations as you would any other code, and to document that your class is copyable and/or cheaply movable if that's an API guarantee.

```
class Foo 
{
 public:
  Foo(Foo&& other) : field_(other.field) {}
  // Bad, defines only move constructor, but not operator=.

 private:
  Field field_;
};
```

Due to the risk of slicing, avoid providing an assignment operator or public copy/move constructor for a class that's intended to be derived from (and avoid deriving from a class with such members). If your base class needs to be copyable, provide a public virtual Clone() method, and a protected copy constructor that derived classes can use to implement it.

If you do not want to support copy/move operations on your type, explicitly disable them using `= delete` in the `public:` section:

```
// MyClass is neither copyable nor movable.
MyClass(const MyClass&) = delete;
MyClass& operator=(const MyClass&) = delete;
```

<a name="classes-structs-vs-classes"></a>
### [4.4](#classes-structs-vs-classes) Structs vs. Classes

Use a `struct` only for passive objects that carry data; everything else is a `class`.

The `struct` and `class` keywords behave almost identically in C++. We add our own semantic meanings to each keyword, so you should use the appropriate keyword for the data-type you're defining.

`structs` should be used for passive objects that carry data, and may have associated constants, but lack any functionality other than access/setting the data members. The accessing/setting of fields is done by directly accessing the fields rather than through method invocations. Methods should not provide behavior but should only be used to set up the data members, e.g., constructor, destructor, `Initialize()`, `Reset()`, `Validate()`.

If more functionality is required, a class is more appropriate. If in doubt, make it a class.

For consistency with STL, you can use `struct` instead of `class` for functors and traits.

Note that member variables in structs and classes have different naming rules.

<a name="classes-inheritance"></a>
### [4.5](#classes-inheritance) Inheritance

Composition is often more appropriate than inheritance. When using inheritance, make it `public`.

**Definition:**

When a sub-class inherits from a base class, it includes the definitions of all the data and operations that the parent base class defines. In practice, inheritance is used in two major ways in C++: implementation inheritance, in which actual code is inherited by the child, and interface inheritance, in which only method names are inherited.

**Pros:**

Implementation inheritance reduces code size by re-using the base class code as it specializes an existing type. Because inheritance is a compile-time declaration, you and the compiler can understand the operation and detect errors. Interface inheritance can be used to programmatically enforce that a class expose a particular API. Again, the compiler can detect errors, in this case, when a class does not define a necessary method of the API.

**Cons:**

For implementation inheritance, because the code implementing a sub-class is spread between the base and the sub-class, it can be more difficult to understand an implementation. The sub-class cannot override functions that are not virtual, so the sub-class cannot change implementation. The base class may also define some data members, so that specifies physical layout of the base class.

**Decision:**

All inheritance should be `public`. If you want to do private inheritance, you should be including an instance of the base class as a member instead.

Do not overuse implementation inheritance. Composition is often more appropriate. Try to restrict use of inheritance to the "is-a" case: Bar subclasses Foo if it can reasonably be said that Bar "is a kind of" Foo.

Make your destructor `virtual` if necessary. If your class has virtual methods, its destructor should be virtual.

Limit the use of protected to those member functions that might need to be accessed from subclasses. Note that data members should be private.

Explicitly annotate overrides of virtual functions or virtual destructors with an `override` or (less frequently) `final` specifier. Older (pre-C++11) code will use the `virtual` keyword as an inferior alternative annotation. For clarity, use exactly one of `override`, `final`, or `virtual` when declaring an override. Rationale: A function or destructor marked `override` or `final` that is not an override of a base class virtual function will not compile, and this helps catch common errors. The specifiers serve as documentation; if no specifier is present, the reader has to check all ancestors of the class in question to determine if the function or destructor is virtual or not.

<a name="classes-multiple-inheritance"></a>
### [4.5](#classes-multiple-inheritance) Multiple Inheritance

Only very rarely is multiple implementation inheritance actually useful. We allow multiple inheritance only when at most one of the base classes has an implementation; all other base classes must be pure interface classes tagged with the `Interface` suffix.

**Definition:**

Multiple inheritance allows a sub-class to have more than one base class. We distinguish between base classes that are `pure interfaces` and those that have an `implementation`.

**Pros:**

Multiple implementation inheritance may let you re-use even more code than single inheritance (see [Inheritance](#classes-inheritance)).

**Cons:**

Only very rarely is multiple `implementation` inheritance actually useful. When multiple implementation inheritance seems like the solution, you can usually find a different, more explicit, and cleaner solution.

**Decision:**

Multiple inheritance is allowed only when all superclasses, with the possible exception of the first one, are pure interfaces. In order to ensure that they remain pure interfaces, they must end with the `Interface` suffix.

**Note:**

There is an [exception](#rule-exceptions-windows) to this rule on Windows.

<a name="classes-interfaces"></a>
### [4.6](#classes-interfaces) Interfaces

Classes that satisfy certain conditions are allowed, but not required, to end with an `Interface` suffix.

**Definition:**

A class is a pure interface if it meets the following requirements:

* It has only public pure virtual ("= 0") methods and static methods (but see below for destructor).
* It may not have non-static data members.
* It need not have any constructors defined. If a constructor is provided, it must take no arguments and it must be protected.
* If it is a subclass, it may only be derived from classes that satisfy these conditions and are tagged with the `Interface` suffix.

An interface class can never be directly instantiated because of the pure virtual method(s) it declares. To make sure all implementations of the interface can be destroyed correctly, the interface must also declare a virtual destructor (in an exception to the first rule, this should not be pure). See Stroustrup, The C++ Programming Language, 3rd edition, section 12.4 for details.

**Pros:**

Tagging a class with the Interface suffix lets others know that they must not add implemented methods or non static data members. This is particularly important in the case of multiple inheritance. Additionally, the interface concept is already well-understood by Java programmers.

**Cons:**

The Interface suffix lengthens the class name, which can make it harder to read and understand. Also, the interface property may be considered an implementation detail that shouldn't be exposed to clients.

**Decision:**

A class may end with Interface only if it meets the above requirements. We do not require the converse, however: classes that meet the above requirements are not required to end with Interface.

<a name="classes-operator-overloading"></a>
### [4.7](#classes-operator-overloading) Operator Overloading

Overload operators judiciously. Do not create user-defined literals.

**Definition:**

C++ permits user code to declare overloaded versions of the built-in operators using the operator keyword, so long as one of the parameters is a user-defined type. The operator keyword also permits user code to define new kinds of literals using `operator""`, and to define type-conversion functions such as `operator bool()`.

**Pros:**

Operator overloading can make code more concise and intuitive by enabling user-defined types to behave the same as built-in types. Overloaded operators are the idiomatic names for certain operations (e.g. ==, <, =, and <<), and adhering to those conventions can make user-defined types more readable and enable them to interoperate with libraries that expect those names.

User-defined literals are a very concise notation for creating objects of user-defined types.

**Cons:**

* Providing a correct, consistent, and unsurprising set of operator overloads requires some care, and failure to do so can lead to confusion and bugs.
* Overuse of operators can lead to obfuscated code, particularly if the overloaded operator's semantics don't follow convention.
* The hazards of function overloading apply just as much to operator overloading, if not more so.
* Operator overloads can fool our intuition into thinking that expensive operations are cheap, built-in operations.
* Finding the call sites for overloaded operators may require a search tool that's aware of C++ syntax, rather than e.g. grep.
* If you get the argument type of an overloaded operator wrong, you may get a different overload rather than a compiler error. For example, `foo < bar` may do one thing, while `&foo < &bar` does something totally different.
* Certain operator overloads are inherently hazardous. Overloading unary & can cause the same code to have different meanings depending on whether the overload declaration is visible. Overloads of `&&`, `||`, and `,` (comma) cannot match the evaluation-order semantics of the built-in operators.
* Operators are often defined outside the class, so there's a risk of different files introducing different definitions of the same operator. If both definitions are linked into the same binary, this results in undefined behavior, which can manifest as subtle run-time bugs.
* User-defined literals allow the creation of new syntactic forms that are unfamiliar even to experienced C++ programmers.

**Decision:**

Define overloaded operators only if their meaning is obvious, unsurprising, and consistent with the corresponding built-in operators. For example, use | as a bitwise- or logical-or, not as a shell-style pipe.

Define operators only on your own types. More precisely, define them in the same headers, .cpp files, and namespaces as the types they operate on. That way, the operators are available wherever the type is, minimizing the risk of multiple definitions. If possible, avoid defining operators as templates, because they must satisfy this rule for any possible template arguments. If you define an operator, also define any related operators that make sense, and make sure they are defined consistently. For example, if you overload `<`, overload all the comparison operators, and make sure `<` and `>` never return true for the same arguments.

Prefer to define non-modifying binary operators as non-member functions. If a binary operator is defined as a class member, implicit conversions will apply to the right-hand argument, but not the left-hand one. It will confuse your users if `a < b` compiles but `b < a` doesn't.

Don't go out of your way to avoid defining operator overloads. For example, prefer to define `==`, `=`, and `<<`, rather than `Equals()`, `CopyFrom()`, and `PrintTo()`. Conversely, don't define operator overloads just because other libraries expect them. For example, if your type doesn't have a natural ordering, but you want to store it in a `std::set`, use a custom comparator rather than overloading `<`.

Do not overload `&&`, `||`, `,` (comma), or `unary &`. Do not overload `operator""`, i.e. do not introduce user-defined literals.

Type conversion operators are covered in the section on implicit conversions. The `=` operator is covered in the section on copy constructors. Overloading `<<` for use with streams is covered in the section on streams. See also the rules on function overloading, which apply to operator overloading as well.

<a name="classes-access-control"></a>
### [4.8](#classes-access-control) Access Control

Make data members `private`, unless they are `static const` (and follow the naming convention for constants). For technical reasons, we allow data members of a test fixture class to be `protected` when using [Google Test](https://github.com/google/googletest)).

<a name="classes-declaration-order"></a>
### [4.9](#classes-declaration-order) Declaration Order

Group similar declarations together, placing `public` parts earlier.

A class definition should usually start with a `public:` section, followed by `protected:`, then `private:`. Omit sections that would be empty.

Within each section, generally prefer grouping similar kinds of declarations together, and generally prefer the following order: types (including `typedef`, `using`, and nested structs and classes), constants, factory functions, constructors, assignment operators, destructor, all other methods, data members.

Do not put large method definitions inline in the class definition. Usually, only trivial or performance-critical, and very short, methods may be defined inline. See Inline Functions for more details.

**[back to top](#table-of-contents)**

<a name="functions"></a>
## [5](#functions) Functions

**[back to top](#table-of-contents)**

<a name="other-cpp-features"></a>
## [6](#other-cpp-features) Other C++ Features

<a name="other-cpp-features-boost"></a>
### [6.1](#other-cpp-features-boost) Boost

Use only allowed libraries from the Boost library collection.

**Definition:**

The [Boost library collection](http://www.boost.org) is a popular collection of peer-reviewed, free, open-source C++ libraries.

**Pros:**

Boost code is generally very high-quality, is widely portable, and fills many important gaps in the C++ standard library, such as type traits and better binders.

**Cons:**

Some Boost libraries encourage coding practices which can hamper readability, such as metaprogramming and other advanced template techniques, and an excessively "functional" style of programming.

**Decision:**

Currently we allow all libraries except the ones that have been superseded by standard libraries in C++.
This might change if we gained some more experience in the readability. 

The following libraries are forbidden, because they've been superseded by standard libraries in C++11:

* [Array](https://www.boost.org/libs/array/) from boost/array.hpp: use [std::array](http://en.cppreference.com/w/cpp/container/array) instead.
* [Pointer Container](https://www.boost.org/libs/ptr_container/) from boost/ptr_container: use containers of [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr) instead.


**[back to top](#table-of-contents)**

<a name="naming"></a>
## [7](#naming) Naming

The most important consistency rules are those that govern naming. The style of a name immediately informs us what sort of thing the named entity is: a type, a variable, a function, a constant, a macro, etc., without requiring us to search for the declaration of that entity. The pattern-matching engine in our brains relies a great deal on these naming rules.

Naming rules are pretty arbitrary, but we feel that consistency is more important than individual preferences in this area, so regardless of whether you find them sensible or not, the rules are the rules.

<a name="naming-general"></a>
### [7.1](#naming-general) General Naming Rules

<a name="naming-file-names"></a>
### [7.2](#naming-file-names) File Names

<a name="naming-type-names"></a>
### [7.3](#naming-type-names) Type Names

<a name="naming-variable-names"></a>
### [7.4](#naming-variable-names) Variable Names

<a name="naming-constant-names"></a>
### [7.5](#naming-constant-names) Constant Names

<a name="naming-function-names"></a>
### [7.6](#naming-function-names) Function Names

<a name="naming-namespace-names"></a>
### [7.6](#naming-namespace-names) Namespace Names

<a name="naming-enumerator-names"></a>
### [7.7](#naming-enumerator-names) Enumerator Names

<a name="naming-macro-names"></a>
### [7.8](#naming-macro-names) Macro Names

<a name="naming-exceptions-to-rules"></a>
### [7.9](#naming-exceptions-to-rules) Exceptions to Naming Rules

**[back to top](#table-of-contents)**

<a name="comments"></a>
## [8](#comments) Comments / Doxygen

<a name="comments-documentation"></a>
### [8.1](#comments-documentation) Documentation

Every class and it's methods must contain a description of what it is for and how it does it's task. To utilize IDEs with doxygen support and to be able to generate a documentation, these descriptions must use the doxygen syntax. To be uniformly the `!` is used to mark a doxygen comment and `@` to mark commands like `@brief` and `@param`.

```
// good

/*!
* @brief This does something.
* @details A more detailed description ...
* @param tag The tag that is used to do something.
* @return true if done successfully, false on failure.
* @sa doSomethingRelated()
*/
bool doSomething(string* tag);

// bad

/**
* \brief This does something.
* \param tag The tag that is used to do something.
*/
void doSomething(string* tag);
```

<a name="comments-multiline-comments"></a>
### [8.2](#comments-multiline-comments) Multiline comments

Use `/*! ... */` for block comments.

```
// good

/*!
* @brief This does something.
* @param tag The tag.
*/
bool doSomething(string* tag);

// bad

// @brief This does something.
// @param tag The tag.
bool doSomething(string* tag);
```

<a name="comments-singleline-comment"></a>
### [8.3](#comments-singleline-comment) Singleline comment

Use `//` for single line comments.
Place single line comments on a newline above the subject of the comment. Put an empty line before the comment unless it's on the first line of a block.

```
// good
// is current tab
int active = true;

// bad
int active = true; // is current tab

// good
int getType()
{
    BOOST_LOG_TRIVIAL(debug) << __FILE__ << "(" << __LINE__ << "): " << "Fetching type...";

    // set the default type to 'no type'
    int typeOut = _type ? _type : -1;

    return typeOut;
}

// bad
int getType()
{
    BOOST_LOG_TRIVIAL(debug) << __FILE__ << "(" << __LINE__ << "): " << "Fetching type...";
    // set the default type to 'no type'
    int typeOut = _type ? _type : -1;
    
    return typeOut;
}

// also good
int getType()
{
    // set the default type to 'no type'
    int typeOut = _type ? _type : -1;
    
    return typeOut;
}
```

<a name="comments-spaces"></a>
### [8.4](#comments-spaces) Spaces

Start all comments with a space to make it easier to read.

```
// good
// is current tab
int active = true;

// bad
//is current tab
int active = true;
```

<a name="comments-fixme"></a>
### [8.5](#comments-fixme) Fixme

Use `// FIXME:` to annotate problems.

```
Calculator::Calculator()
{
    // FIXME: shouldn't use a global here
    _total = 0;
}
```

<a name="comments-todo"></a>
### [8.6](#comments-todo) Todo

Use `// TODO:` to annotate solutions to problems.

```
Calculator::Calculator()
{
    // TODO: total should be configurable by an options param
    _total = 0;
}
```

**[back to top](#table-of-contents)**

<a name="formatting"></a>
## [9](#formatting) Formatting

Coding style and formatting are pretty arbitrary, but a project is much easier to follow if everyone uses the same style. Individuals may not agree with every aspect of the formatting rules, and some of the rules may take some getting used to, but it is important that all project contributors follow the style rules so that they can all read and understand everyone's code easily.

To help you format code correctly, we've created a `.clang-format` file.

<a name="formatting-line-length"></a>
### [9.1](#formatting-line-length) Line Length

Each line of text in your code should be at most 120 characters long.

We recognize that this rule is controversial, but we feel that 120 characters are a reasonable compromise.

**Pros:**

Those who favor a rule of 80 characters argue that it is rude to force them to resize their windows and there is no need for anything longer. Some folks are used to having several code windows side-by-side, and thus don't have room to widen their windows in any case. People set up their work environment assuming a particular maximum window width, and 80 columns has been the traditional standard. Why change it?

**Cons:**

Proponents of change argue that a wider line can make code more readable. The 80-column limit is an hidebound throwback to 1960s mainframes; modern equipment has wide screens that can easily show longer lines.

**Decision:**

120 characters is the maximum.

**Exception:**

Comment lines can be longer than 120 characters if it is not feasible to split them without harming readability, ease of cut and paste or auto-linking -- e.g. if a line contains an example command or a literal URL longer than 120 characters.

**Exception:**

A raw-string literal may have content that exceeds 120 characters. Except for test code, such literals should appear near the top of a file.

**Exception:**

An #include statement with a long path may exceed 120 columns.

<a name="formatting-non-ascii"></a>
### [9.2](#formatting-non-ascii) Non-ASCII Characters

Non-ASCII characters should be rare, and must use UTF-8 formatting.

You shouldn't hard-code user-facing text in source, even English, so use of non-ASCII characters should be rare. However, in certain cases it is appropriate to include such words in your code. For example, if your code parses data files from foreign sources, it may be appropriate to hard-code the non-ASCII string(s) used in those data files as delimiters. More commonly, unittest code (which does not need to be localized) might contain non-ASCII strings. In such cases, you should use UTF-8, since that is an encoding understood by most tools able to handle more than just ASCII.

Hex encoding is also OK, and encouraged where it enhances readability — for example, "\xEF\xBB\xBF", or, even more simply, u8"\uFEFF", is the Unicode zero-width no-break space character, which would be invisible if included in the source as straight UTF-8.

Use the u8 prefix to guarantee that a string literal containing \uXXXX escape sequences is encoded as UTF-8. Do not use it for strings containing non-ASCII characters encoded as UTF-8, because that will produce incorrect output if the compiler does not interpret the source file as UTF-8.

You shouldn't use the C++11 char16_t and char32_t character types, since they're for non-UTF-8 text. For similar reasons you also shouldn't use wchar_t (unless you're writing code that interacts with the Windows API, which uses wchar_t extensively).

<a name="formatting-spaces-vs-tabs"></a>
### [9.3](#formatting-spaces-vs-tabs) Spaces vs. Tabs

Use only spaces, and indent 4 spaces at a time.

We use spaces for indentation. Do not use tabs in your code. You should set your editor to emit spaces when you hit the tab key.

<a name="formatting-function-declaration"></a>
### [9.4](#formatting-function-declaration) Function Declarations and Definitions

Return type on the same line as function name, parameters on the same line if they fit. Wrap parameter lists which do not fit on a single line as you would wrap arguments in a [function call](#formatting-function-calls).

Functions look like this:

```
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) 
{
    DoSomething();
    ...
}
```

If you have too much text to fit on one line:

```
ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) 
{
    DoSomething();
    ...
}
```

or if you cannot fit even the first parameter:

```
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) 
{
    DoSomething();  // 4 space indent
    ...
}
```

Some points to note:

* Choose good parameter names.
* A parameter name may be omitted only if the parameter is not used in the function's definition.
* If you cannot fit the return type and the function name on a single line, break between them.
* If you break after the return type of a function declaration or definition, do not indent.
* The open parenthesis is always on the same line as the function name.
* There is never a space between the function name and the open parenthesis.
* There is never a space between the parentheses and the parameters.
* The open curly brace is on the start of the next line if the function declaration and the body doesn't fit in one line.
* The close curly brace is either on the last line by itself or on the same line as the open curly brace.
* There should be a space between the close parenthesis and the open curly brace.
* All parameters should be aligned if possible.
* Default indentation is 4 spaces.
* Wrapped parameters have a 4 space indent.

Unused parameters that are obvious from context may be omitted:

```
class Foo
{
  public:
    Foo(Foo&&);
    Foo(const Foo&);
    Foo& operator=(Foo&&);
    Foo& operator=(const Foo&);
};
```

Unused parameters that might not be obvious should comment out the variable name in the function definition:

```
class Shape 
{
  public:
    virtual void Rotate(double radians) = 0;
};

class Circle : public Shape 
{
  public:
    void Rotate(double radians) override;
};

void Circle::Rotate(double /*radians*/) {}
```

```
// Bad - if someone wants to implement later, it's not clear what the
// variable means.
void Circle::Rotate(double) {}
```

Attributes, and macros that expand to attributes, appear at the very beginning of the function declaration or definition, before the return type:

```
MUST_USE_RESULT bool IsOK();
```

<a name="formatting-lambda-expressions"></a>
### [9.5](#formatting-lambda-expressions) Lambda Expressions

<a name="formatting-function-calls"></a>
### [9.6](#formatting-function-calls) Function Calls

<a name="formatting-braced-initializer-list"></a>
### [9.7](#formatting-braced-initializer-list) Braced Initializer List Format

<a name="formatting-conditionals"></a>
### [9.8](#formatting-conditionals) Conditionals

<a name="formatting-loops-and-switch-statements"></a>
### [9.9](#formatting-loops-and-switch-statements) Loops and Switch Statements

<a name="formatting-pointer-and-reference-expressions"></a>
### [9.10](#formatting-pointer-and-reference-expressions) Pointer and Reference Expressions

<a name="formatting-boolean-expressions"></a>
### [9.11](#formatting-boolean-expressions) Boolean Expressions

<a name="formatting-return-values"></a>
### [9.12](#formatting-return-values) Return Values

<a name="formatting-variable-and-array-initialization"></a>
### [9.13](#formatting-variable-and-array-initialization) Variable and Array Initialization

<a name="formatting-preprocessor-directives"></a>
### [9.14](#formatting-preprocessor-directives) Preprocessor Directives

<a name="formatting-class-format"></a>
### [9.15](#formatting-class-format) Class Format

<a name="formatting-constructor-initializer-lists"></a>
### [9.16](#formatting-constructor-initializer-lists) Constructor Initializer Lists

<a name="formatting-namespace-formatting"></a>
### [9.17](#formatting-namespace-formatting) Namespace Formatting

<a name="formatting-horizontal-whitespace"></a>
### [9.18](#formatting-horizontal-whitespace) Horizontal Whitespace

<a name="formatting-horizontal-whitespace-general"></a>
#### [9.19.1](#formatting-horizontal-whitespace-general) General

<a name="formatting-horizontal-whitespace-loops-and-conditionals"></a>
#### [9.19.2](#formatting-horizontal-whitespace-loops-and-conditionals) Loops and Conditionals

<a name="formatting-horizontal-whitespace-operators"></a>
#### [9.19.3](#formatting-horizontal-whitespace-operators) Operators

<a name="formatting-horizontal-whitespace-templates-and-casts"></a>
#### [9.19.4](#formatting-horizontal-whitespace-templates-and-casts) Templates and Casts

<a name="formatting-vertical-whitespace"></a>
### [9.20](#formatting-vertical-whitespace) Vertical Whitespace


**[back to top](#table-of-contents)**

<a name="rule-exceptions"></a>
## [10](#rule-exceptions) Exceptions to the Rules

The coding conventions described above are mandatory. However, like all good rules, these sometimes have exceptions, which we discuss here.

<a name="rule-exceptions-old-code"></a>
### [10.1](#rule-exceptions-old-code) Existing Non-conformant Code

You may diverge from the rules when dealing with code that does not conform to this style guide.

If you find yourself modifying code that was written to specifications other than those presented by this guide, you may have to diverge from these rules in order to stay consistent with the local conventions in that code. If you are in doubt about how to do this, ask the original author or the person currently responsible for the code. Remember that consistency includes local consistency, too.

<a name="rule-exceptions-windows"></a>
### [10.2](#rule-exceptions-windows) Windows Code

Windows programmers have developed their own set of coding conventions, mainly derived from the conventions in Windows headers and other Microsoft code. We want to make it easy for anyone to understand your code, so we have a single set of guidelines for everyone writing C++ on any platform.

It is worth reiterating a few of the guidelines that you might forget if you are used to the prevalent Windows style:

* Do not use Hungarian notation (for example, naming an integer iNum). Use the appcom naming conventions, including the .cpp extension for source files.
* Windows defines many of its own synonyms for primitive types, such as DWORD, HANDLE, etc. It is perfectly acceptable, and encouraged, that you use these types when calling Windows API functions. Even so, keep as close as you can to the underlying C++ types. For example, use `const TCHAR *` instead of `LPCTSTR`.
* When compiling with Microsoft Visual C++, set the compiler to warning level 3 or higher, and treat all warnings as errors.
* In fact, do not use any nonstandard extensions, like `#pragma` and `__declspec`, unless you absolutely must. Using `__declspec(dllimport)` and `__declspec(dllexport)` is allowed; however, you must use them through macros such as `DLLIMPORT` and `DLLEXPORT`, so that someone can easily disable the extensions if they share the code.

However, there are just a few rules that we occasionally need to break on Windows:

* Normally we forbid the use of multiple implementation inheritance; however, it is required when using `COM` and some `ATL/WTL` classes. You may use multiple implementation inheritance to implement `COM` or `ATL/WTL` classes and interfaces.
* Although you should not use exceptions in your own code, they are used extensively in the `ATL` and some `STLs`, including the one that comes with Visual C++. When using the `ATL`, you should define `_ATL_NO_EXCEPTIONS` to disable exceptions. You should investigate whether you can also disable exceptions in your `STL`, but if not, it is OK to turn on exceptions in the compiler. (Note that this is only to get the `STL` to compile. You should still not write exception handling code yourself.)
* The usual way of working with precompiled headers is to include a header file at the top of each source file, typically with a name like `StdAfx.h` or `precompile.h`. To make your code easier to share with other projects, avoid including this file explicitly (except in `precompile.cpp`), and use the `/FI` compiler option to include the file automatically.
* Resource headers, which are usually named `resource.h` and contain only macros, do not need to conform to these style guidelines.

**[back to top](#table-of-contents)**

<a name="asset-naming-conventions"></a>
## [11](#asset-naming-conventions) Asset Naming Conventions

<a name="asset-naming-conventions-image-naming"></a>
### [11.1](#asset-naming-conventions-image-naming) Image Naming

Images must be named after the following pattern:

```
<WHAT>_<WHERE>_<DESCRIPTION>[_<SIZE>][_STATE]
```

```
// example
bg_login_background
ic_login_button_small_pressed
```

The `WHAT` denodes the type of the drawable. It can be one of the following:

* ic (Icons)
* bg (Backgrounds)
* shape (Shapes)

The `WHERE` describes in which part of the app the drawable is used. If the drawable is used in more than one screen use `all`.
The `DESCRIPTION` must tell the reader what the actual purpose of the drawable is.
The `SIZE` part is optional and describes the size of the drawable. This can either be a measurable (e.g. 24dp) or a generic term (e.g. small).
`STATE` indicates optionally the state of the drawable. It can be one of the following:

* normal (can be omitted)
* disabled
* selected
* pressed

**[back to top](#table-of-contents)**

<a name="logging"></a>
## [12](#logging) Logging

<a name="logging-logger"></a>
### [12.1](#logging-logger) Logger

Today applications grow rapidly, becoming complicated and difficult to test and debug. This is where logging can help.  

The [Boost.Log v2](http://boost.org/libs/log) and the appropiate log level must be used to help debug errors.

Use the debug logging version for message that are only intended to be seen by developers.

```
// example
#include <cstdlib>

#include <boost/log/trivial.hpp>

int main(int argc, char *argv[])
{
#if defined(NDEBUG)
    boost::log::core::get()->set_filter(boost::log::trivial::severity >= boost::log::trivial::info);
#else
    boost::log::core::get()->set_filter(boost::log::trivial::severity >= boost::log::trivial::debug);
#endif

    BOOST_LOG_TRIVIAL(trace) << "A trace severity message";
    BOOST_LOG_TRIVIAL(debug) << "A debug severity message";
    BOOST_LOG_TRIVIAL(info) << "An informational severity message";
    BOOST_LOG_TRIVIAL(warning) << "A warning severity message";
    BOOST_LOG_TRIVIAL(error) << "An error severity message";
    BOOST_LOG_TRIVIAL(fatal) << "A fatal severity message";

    return EXIT_SUCCESS;
}
```

<a name="logging-loglevels"></a>
### [12.2](#logging-loglevels) Log levels

[Boost.Log v2](http://boost.org/libs/log) provides a various amount of log levels each with its own purpose.
Here is a list of log levels and their meaning:

| Level | Meaning |
|---|---|
| trace | More detailed debugging informations, mostly used to hunt down bugs. Only to be read by developers. |
| debug | Informations written for debugging purposes (only on debug builds), meant only to be read by developers. |
| info | Course grained (rarely written informations) that can help sysadmins etc. |
| warning | Simple application error or unexpected behaviour. Application can continue. Warn for example in case of bad login attempts, unexpected data during import jobs. |
| error | Application error/exception but application can continue. Part of the application is probably not working. |
| fatal | Fatal application error/exception, application cannot continue, for example database is down. |

By default, the logging level of an application should be `debug` on debug builds and `info` on release builds.

<a name="logging-log-messages"></a>
### [12.3](#logging-log-messages) Logging Messages

Write meaningful log messages. This sounds easy but is in fact really hard. Keep in mind, that you sometimes log for the event of an error, which in reality occurs only rarely. 
But if it does, you depend on a clear log message along with an expressive payload. A good log message must consist of the following items:

* The filename and linenumber of the log statement (these are: `__FILE__` and `__LINE__` for C/C++)
* A meaningful log message
* An expressive payload, usually a JSON

<a name="logging-log-language"></a>
### [12.4](#logging-log-language) Logging Message Language

Write your log messages in english. English is a well known language both in terms of writing and reading. Furthermore does it not contain any special characters, which means that it can be logged with ASCII. This is especially important when performing log rotation, since you do not know where your logs are stored.

<a name="logging-payload"></a>
### [12.5](#logging-payload) Logging Payload

Always log with payload (i.e. context).
The log message often is not sufficient when tracing bugs. You almost always need additional information. So just log them along with the message and you keep yourself from deploying a new version of the app just to improve log messages.

Make sure that the payload is well formatted and complete. The format helps you to filter and/or search for certain events.

```
// good
Transaction failed: { id: 63287, checksum: null }

// bad
Transaction failed!

// bad. Payload included but hard to read/parse
Transaction '63287' failed: Checksum 'null' is invalid!
```

**[back to top](#table-of-contents)**

<a name="roadmap"></a>
## [13](#roadmap) Roadmap

This section describes items, which will be added to this style guide in the future.
These items are categorized in two sections namely `next` for items added in the next release and `future` for items added in future releases without a fixed date.

<a name="roadmap-next"></a>
### [13.1](#roadmap-next) Next

<a name="roadmap-future"></a>
### [13.2](#roadmap-future) Future

* Linter for continuous integration based on this style guide

**[back to top](#table-of-contents)**

<a name="resources"></a>
## [14](#resources) Resources

<a name="resources-documentation"></a>
### [14.1](#resources-documentation) Documentation

* [http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html)

<a name="resources-logging"></a>
### [14.2](#resources-logging) Logging

* [Boost.Log v2](http://boost.org/libs/log)

<a name="resources-related-work"></a>
### [14.2](#resources-related-work) Related Work

This Style Guide is based on the following Style Guides:

* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
* [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)

**[back to top](#table-of-contents)**

<a name="license"></a>
## [15](#license) License

(The MIT License)

Copyright (c) 2018 appcom interactive GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
