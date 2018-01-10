# Appcom C++ Style Guide

This document describes the style guide applied to C++ projects for appcom interactive GmbH. It describes rules how to organize your project, dependencies and files, so that some best practises are held.

So use it in your favor if you want to and/or override the style guide in any way you want.

<a name="table-of-contents"></a>
## Table of Contents

1. [Dependencies](#dependencies)
1. [Comments / Doxygen](#comments)
1. [Files](#files)
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
* Include statements
* Forward declarations
* At most one top-level class

<a name="files-source-file-structure"></a>
### [0.5](#files-source-file-structure) Source file structure

A source file consists of, *in order*:

* License or copyright information, if present
* Include statements
* Exactly one top-level class

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

This Style Guide is inspired by the following Style Guides:

* [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)

**[back to top](#table-of-contents)**

<a name="license"></a>
## [0](#license) License

(The MIT License)

Copyright (c) 2017 appcom interactive GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
