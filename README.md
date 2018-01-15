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

**Defenition:**

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

// GoodUser.cpp:
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

**[back to top](#table-of-contents)**

<a name="functions"></a>
## [5](#functions) Functions

**[back to top](#table-of-contents)**

<a name="other-cpp-features"></a>
## [6](#other-cpp-features) Other C++ Features

<a name="other-cpp-features-boost"></a>
### [6.1](#other-cpp-features-boost) Boost

Use only approved libraries from the Boost library collection.

**Definition:**

The [Boost library collection](http://www.boost.org) is a popular collection of peer-reviewed, free, open-source C++ libraries.

**Pros:**

Boost code is generally very high-quality, is widely portable, and fills many important gaps in the C++ standard library, such as type traits and better binders.

**Cons:**

Some Boost libraries encourage coding practices which can hamper readability, such as metaprogramming and other advanced template techniques, and an excessively "functional" style of programming.

**Decision:**

In order to maintain a high level of readability for all contributors who might read and maintain code, we only allow an approved subset of Boost features. Currently, the following libraries are permitted:

* [Call Traits](https://www.boost.org/libs/utility/call_traits.htm) from boost/call_traits.hpp
* [Compressed Pair](https://www.boost.org/libs/utility/compressed_pair.htm) from boost/compressed_pair.hpp
* The [Boost Graph Library (BGL)](https://www.boost.org/libs/graph/) from boost/graph, except serialization (adj_list_serialize.hpp) and parallel/distributed algorithms and data structures (boost/graph/parallel/* and boost/graph/distributed/*).
* [Property Map](https://www.boost.org/libs/property_map/) from boost/property_map, except parallel/distributed property maps (boost/property_map/parallel/*).
* [Iterator](https://www.boost.org/libs/iterator/) from boost/iterator
* The part of [Polygon](https://www.boost.org/libs/polygon/) that deals with Voronoi diagram construction and doesn't depend on the rest of Polygon: boost/polygon/voronoi_builder.hpp, boost/polygon/voronoi_diagram.hpp, and boost/polygon/voronoi_geometry_type.hpp
* [Bimap](https://www.boost.org/libs/bimap/) from boost/bimap
* [Statistical Distributions and Functions](https://www.boost.org/libs/math/doc/html/dist.html) from boost/math/distributions
* [Special Functions](https://www.boost.org/libs/math/doc/html/special.html) from boost/math/special_functions
* [Multi-index](https://www.boost.org/libs/multi_index/) from boost/multi_index
* [Heap](https://www.boost.org/libs/heap/) from boost/heap
* The flat containers from [Container](https://www.boost.org/libs/container/): boost/container/flat_map, and boost/container/flat_set
* [Intrusive](https://www.boost.org/libs/intrusive/) from boost/intrusive.
* The [boost/sort](https://www.boost.org/libs/sort/) library.
* [Preprocessor](https://www.boost.org/libs/preprocessor/) from boost/preprocessor.

We are actively considering adding other Boost features to the list, so this list may be expanded in the future.

The following libraries are forbidden, because they've been superseded by standard libraries in C++11:

* [Array](https://www.boost.org/libs/array/) from boost/array.hpp: use [std::array](http://en.cppreference.com/w/cpp/container/array) instead.
* [Pointer Container](https://www.boost.org/libs/ptr_container/) from boost/ptr_container: use containers of [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr) instead.


**[back to top](#table-of-contents)**

<a name="naming"></a>
## [7](#naming) Naming

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

<a name="rule-exceptions"></a>
## [9](#rule-exceptions) Exceptions to the Rules

**[back to top](#table-of-contents)**

<a name="asset-naming-conventions"></a>
## [10](#asset-naming-conventions) Asset Naming Conventions

<a name="asset-naming-conventions-image-naming"></a>
### [10.1](#asset-naming-conventions-image-naming) Image Naming

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
## [11](#logging) Logging

<a name="logging-logger"></a>
### [11.1](#logging-logger) Logger

Today applications grow rapidly, becoming complicated and difficult to test and debug. This is where logging can help.  

The [Google logging module](https://github.com/google/glog) and the appropiate log level must be used to help debug errors. If the Google logging module is not available for the target platform, use [miniglog](https://github.com/tzutalin/miniglog) instead. Miniglog has the same interface as glog and can be interchanged with glog for other platforms.

Use the debug logging version for message that are only intended to be seen by developers.

```
// example
#include <glog/logging.h>

int main(int argc, char *argv[])
{
    LOG(INFO) << "msg";
    LOG(ERROR) << "error msg";
    LOG(FATAL) << "fatal msg";

    // Debug only version
    DLOG(INFO) << "msg";
    DLOG(ERROR) << "error msg";
    DLOG(FATAL) << "fatal msg";

    return 0;
}
```

<a name="logging-loglevels"></a>
### [11.2](#logging-loglevels) Log levels

Google provides a various amount of log levels each with its own purpose.
Here is a list of log levels and their meaning:

| Level | Meaning |
|---|---|
| INFO | Course grained (rarely written informations) that can help sysadmins etc. |
| WARNING | Simple application error or unexpected behaviour. Application can continue. Warn for example in case of bad login attempts, unexpected data during import jobs. |
| ERROR | Application error/exception but application can continue. Part of the application is probably not working. |
| FATAL | Fatal application error/exception, application cannot continue, for example database is down. |

<a name="logging-log-messages"></a>
### [11.3](#logging-log-messages) Logging Messages

Write meaningful log messages. This sounds easy but is in fact really hard. Keep in mind, that you sometimes log for the event of an error, which in reality occurs only rarely. 
But if it does, you depend on a clear log message along with an expressive payload. A good log message must consist of the following items:

* The filename and linenumber of the log statement (these are: `__FILE__` and `__LINE__` for C/C++)
* A meaningful log message
* An expressive payload, usually a JSON

<a name="logging-log-language"></a>
### [11.4](#logging-log-language) Logging Message Language

Write your log messages in english. English is a well known language both in terms of writing and reading. Furthermore does it not contain any special characters, which means that it can be logged with ASCII. This is especially important when performing log rotation, since you do not know where your logs are stored.

<a name="logging-payload"></a>
### [11.5](#logging-payload) Logging Payload

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
## [12](#roadmap) Roadmap

This section describes items, which will be added to this style guide in the future.
These items are categorized in two sections namely `next` for items added in the next release and `future` for items added in future releases without a fixed date.

<a name="roadmap-next"></a>
### [12.1](#roadmap-next) Next

<a name="roadmap-future"></a>
### [12.2](#roadmap-future) Future

* Linter for continuous integration based on this style guide

**[back to top](#table-of-contents)**

<a name="resources"></a>
## [13](#resources) Resources

<a name="resources-documentation"></a>
### [13.1](#resources-documentation) Documentation

* [http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html)

<a name="resources-logging"></a>
### [13.2](#resources-logging) Logging

* [glog - Google logging module](https://github.com/google/glog)
* [miniglog - Portable glog for cross-platforms](https://github.com/tzutalin/miniglog)

<a name="resources-related-work"></a>
### [13.2](#resources-related-work) Related Work

This Style Guide is based on the following Style Guides:

* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
* [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)

**[back to top](#table-of-contents)**

<a name="license"></a>
## [14](#license) License

(The MIT License)

Copyright (c) 2017 appcom interactive GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
