# Appcom C++ Style Guide

This document describes the style guide applied to C++ projects for appcom interactive GmbH. It describes rules how to organize your project, dependencies and files, so that some best practises are held.

So use it in your favor if you want to and/or override the style guide in any way you want.

<a name="table-of-contents"></a>
## Table of Contents

1. [Dependencies](#dependencies)
1. [Roadmap](#roadmap)
1. [Resources](#resources)
1. [License](#license)


### Documentation
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
