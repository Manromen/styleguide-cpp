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
## [0](#dependencies) Dependencies

<a name="dependencies-library-version"></a>
### [0.1](#dependencies-library-version) Library version

Always use the latest stable version of a library if possible.
Make sure to migrate also major releases if possible.

<a name="dependencies-library-provision"></a>
### [0.2](#dependencies-library-provision) Library provision

Libraries must be included using git submodules if compiled within the project itself or using an artefact repository (like sonatype nexus) if precompiled. Provide a script for installing these if needed. The name of the script must be install-deps.sh and placed at the source root.

<a name="dependencies-folder"></a>
### [0.3](#dependencies-folder) Folder

Dependencies should be found and structured at a well known place. Therefore these must be placed under the `deps` folder at the souce root. Precompiled libraries must be placed under the subfolder `lib` and the corresponding header files must be under the subfolder `include`.

**[back to top](#table-of-contents)**

<a name="files"></a>
## [0](#files) Files

<a name="files-file-names"></a>
### [0.1](#files-file-names) File Names

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
### [0.2](#files-file-encoding) File encoding: UTF-8

Source files are encoded in UTF-8.

<a name="files-whitespace-characters"></a>
### [0.3](#files-whitespace-characters) Whitespace characters

Aside from the line terminator sequence, the ASCII horizontal space character (0x20) is the only whitespace character that appears anywhere in a source file.

This implies that:

* All other whitespace characters in string and character literals are escaped.
* Tab characters are not used for indentation.

<a name="files-header-file-structure"></a>
### [0.4](#files-header-file-structure) Header file structure

A header file consists of, *in order*:

* License or copyright information, if present
* An include guard (see [Header include guard](#files-header-include-guard))
* Include statements
* Forward declarations
* At most one top-level class

<a name="files-header-include-guard"></a>
### [0.5](#files-header-include-guard) Header include guard

All header files must use `#pragma once` to prevent multiple inclusion.

<a name="files-source-file-structure"></a>
### [0.6](#files-source-file-structure) Source file structure

A source file consists of, *in order*:

* License or copyright information, if present
* Include statements
* Exactly one top-level class

**[back to top](#table-of-contents)**

<a name="scoping"></a>
## [0](#scoping) Scoping

<a name="scoping-namespaces"></a>
### [0.1](#scoping-namespaces) Namespaces

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
### [0.2](#scoping-unnamed-namespaces) Unnamed Namespaces and Static Variables

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
### [0.3](#scoping-global-functions) Nonmember, Static Member, and Global Functions

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
### [0.4](#scoping-local-variables) Local Variables

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
### [0.5](#scoping-global-variables) Static and Global Variables

Variables of class type with static storage duration are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are constexpr: they have no dynamic initialization or destruction.

Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.

The order in which class constructors and initializers for static variables are called is only partially specified in C++ and can even change from build to build, which can cause bugs that are difficult to find. Therefore in addition to banning globals of class type, we do not allow non-local static variables to be initialized with the result of a function, unless that function (such as getenv(), or getpid()) does not itself depend on any other globals. However, a static POD variable within function scope may be initialized with the result of a function, since its initialization order is well-defined and does not occur until control passes through its declaration.

Likewise, global and static variables are destroyed when the program terminates, regardless of whether the termination is by returning from main() or by calling exit(). The order in which destructors are called is defined to be the reverse of the order in which the constructors were called. Since constructor order is indeterminate, so is destructor order. For example, at program-end time a static variable might have been destroyed, but code still running — perhaps in another thread — tries to access it and fails. Or the destructor for a static string variable might be run prior to the destructor for another variable that contains a reference to that string.

One way to alleviate the destructor problem is to terminate the program by calling quick_exit() instead of exit(). The difference is that quick_exit() does not invoke destructors and does not invoke any handlers that were registered by calling atexit(). If you have a handler that needs to run when a program terminates via quick_exit() (flushing logs, for example), you can register it using at_quick_exit(). (If you have a handler that needs to run at both exit() and quick_exit(), you need to register it in both places.)

As a result we only allow static variables to contain POD data. This rule completely disallows std::vector (use C arrays instead), or string (use const char []).

If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once(). Note that this must be a raw pointer, not a "smart" pointer, since the smart pointer's destructor will have the order-of-destructor issue that we are trying to avoid.

**[back to top](#table-of-contents)**

<a name="classes"></a>
## [0](#classes) Classes

**[back to top](#table-of-contents)**

<a name="functions"></a>
## [0](#functions) Functions

**[back to top](#table-of-contents)**

<a name="other-cpp-features"></a>
## [0](#other-cpp-features) Other C++ Features



**[back to top](#table-of-contents)**

<a name="naming"></a>
## [0](#naming) Naming

**[back to top](#table-of-contents)**

<a name="comments"></a>
## [0](#comments) Comments / Doxygen

<a name="comments-documentation"></a>
### [0.1](#comments-documentation) Documentation

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
### [0.2](#comments-multiline-comments) Multiline comments

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
### [0.3](#comments-singleline-comment) Singleline comment

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
### [0.4](#comments-spaces) Spaces

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
### [0.5](#comments-fixme) Fixme

Use `// FIXME:` to annotate problems.

```
Calculator::Calculator()
{
    // FIXME: shouldn't use a global here
    _total = 0;
}
```

<a name="comments-todo"></a>
### [0.6](#comments-todo) Todo

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
## [0](#rule-exceptions) Exceptions to the Rules

**[back to top](#table-of-contents)**

<a name="asset-naming-conventions"></a>
## [0](#asset-naming-conventions) Asset Naming Conventions

<a name="asset-naming-conventions-image-naming"></a>
### [0.1](#asset-naming-conventions-image-naming) Image Naming

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
## [0](#logging) Logging

<a name="logging-logger"></a>
### [0.1](#logging-logger) Logger

Today applications grow rapidly, becoming complicated and difficult to test and debug. This is where logging can help. For most applications we already use the [Boost libraries](http://www.boost.org). Boost provides also a library for logging. [Boost.Log v2](http://www.boost.org/doc/libs/1_66_0/libs/log/doc/html/index.html) and the appropiate log level must be used to help debug errors.

```
// example
#include <boost/log/trivial.hpp>

int main(int argc, char *argv[])
{
    BOOST_LOG_TRIVIAL(trace) << "A trace severity message";
    BOOST_LOG_TRIVIAL(debug) << "A debug severity message";
    BOOST_LOG_TRIVIAL(info) << "An informational severity message";
    BOOST_LOG_TRIVIAL(warning) << "A warning severity message";
    BOOST_LOG_TRIVIAL(error) << "An error severity message";
    BOOST_LOG_TRIVIAL(fatal) << "A fatal severity message";

    return 0;
}
```

<a name="logging-loglevels"></a>
### [0.2](#logging-loglevels) Log levels

Boost.Log provides a various amount of log levels each with its own purpose.
Here is a list of log levels and their meaning:

| Level | Meaning |
|---|---|
| trace | For developers only, can be used to follow the program execution. |
| debug | For developers only, for debugging purpose. |
| info | Production optionally, course grained (rarely written informations) that can help sysadmins etc. |
| warning | Production, simple application error or unexpected behaviour. Application can continue. Warn for example in case of bad login attempts, unexpected data during import jobs. |
| error | Production, application error/exception but application can continue. Part of the application is probably not working. |
| fatal | Production, fatal application error/exception, application cannot continue, for example database is down. |

<a name="logging-log-messages"></a>
### [0.3](#logging-log-messages) Logging Messages

Write meaningful log messages. This sounds easy but is in fact really hard. Keep in mind, that you sometimes log for the event of an error, which in reality occurs only rarely. 
But if it does, you depend on a clear log message along with an expressive payload. A good log message must consist of the following items:

* The filename and linenumber of the log statement (these are: `__FILE__` and `__LINE__` for C/C++)
* A meaningful log message
* An expressive payload, usually a JSON

<a name="logging-log-language"></a>
### [0.4](#logging-log-language) Logging Message Language

Write your log messages in english. English is a well known language both in terms of writing and reading. Furthermore does it not contain any special characters, which means that it can be logged with ASCII. This is especially important when performing log rotation, since you do not know where your logs are stored.

<a name="logging-payload"></a>
### [0.5](#logging-payload) Logging Payload

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
## [0](#roadmap) Roadmap

This section describes items, which will be added to this style guide in the future.
These items are categorized in two sections namely `next` for items added in the next release and `future` for items added in future releases without a fixed date.

<a name="roadmap-next"></a>
### [0.1](#roadmap-next) Next

<a name="roadmap-future"></a>
### [0.2](#roadmap-future) Future

* Linter for continuous integration based on this style guide

**[back to top](#table-of-contents)**

<a name="resources"></a>
## [0](#resources) Resources

<a name="resources-documentation"></a>
### [0.1](#resources-documentation) Documentation

* [http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html)

<a name="resources-logging"></a>
### [0.2](#resources-logging) Logging

* [Boost.Log v2](http://www.boost.org/doc/libs/1_66_0/libs/log/doc/html/index.html)

<a name="resources-related-work"></a>
### [0.2](#resources-related-work) Related Work

This Style Guide is based on the following Style Guides:

* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
* [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)

**[back to top](#table-of-contents)**

<a name="license"></a>
## [0](#license) License

(The MIT License)

Copyright (c) 2017 appcom interactive GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
