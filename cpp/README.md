# BlueBrain HPC Team C++ Development Guidelines

This document describes both C++ Best Practices adopted by
HPC team, and the tools and processes required to
ensure they are properly followed over time.

## Documentation

Development Guidelines are split in the following sections:
* [Tooling](./Tooling.md)
* [Process](./Process.md)
* [Code Formatting](./formatting/README.md)
* Best Practices
* Python bindings

## Status

This project in currently under development, and shall not provide the features
its documentation pretends. Here is a raw summary of the status:

| Feature               | Definition         | Documentation      | Integration        | Adoption |
| --------------------- | ------------------ | ------------------ | ------------------ | -------- |
| ClangFormat           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |          |
| ClangTidy             | :heavy_check_mark: |                    |                    |          |
| Naming Conventions    |                    |                    |                    |          |
| Writing Documentation |                    |                    |                    |          |
| Good Practices        |                    |                    |                    |          |
| Memory Check          |                    |                    |                    |          |
| UT Code Coverage      |                    |                    |                    |          |

## CMake Project

This repository provides a CMake project that allows you to use the tools and the processes described in this document.

### Requirements

This CMake project expects for the following utilities to be available:
* [ClangFormat 7](https://releases.llvm.org/7.0.0/tools/clang/docs/ClangFormat.html)
* [ClangTidy 7](https://releases.llvm.org/7.0.0/tools/clang/tools/extra/docs/clang-tidy/index.html)
* [cmake-format](https://github.com/cheshirekow/cmake_format) Python package
* [pre-commit](https://pre-commit.com/) Python package
Optionally, it will also looks for:
* [valgrind](http://valgrind.org/) memory checker
* code coverage utilities like gcov, lcov, or gcov

### Installation

You can import this CMake project into your Git repository using a git submodule:
```
git submodule add https://github.com/BlueBrain/hpc-coding-conventions.git
git submodule update --init --recursive
```

Then simply add the following line in the top `CMakeLists.txt`, after your project
declaration:
```
project(mylib CXX)
# [...]
add_subdirectory(hpc-coding-conventions/cpp)
```

After cloning of updating this git submodule, run CMake to take benefits of the latest changes.
This will setup or update git [pre-commit](https://pre-commit.com) hooks of this repository.

### Usage

#### Code Formatting

To activate code formatting of both C++ and CMake files,
enable CMake variable `${PROJECT}_FORMATTING` where `${PROJECT}` is the name given
to the CMake `project` function.

For instance, given a project `foo`:

`cmake -Dfoo_FORMATTING:BOOL=ON <path>`

##### Usage

This will add the following *make* targets:

* `clang-format`: to format C/C++ code
* `check-clang-format`: task fails it at least one C/C++ file has improper format.

##### Advanced configuration

A list of CMake cache variables can be used to customize code formatting:

* `${PROJECT}_ClangFormat_OPTIONS`: additional options given to `clang-format` command.
  Default value is `""`.
* `${PROJECT}_ClangFormat_FILES_RE`: list of regular expressions matching C/C++ filenames
  to format. Default is:<br/>
  `"^.*\\\\.c$$" "^.*\\\\.h$$" "^.*\\\\.cpp$$" "^.*\\\\.hpp$$"`
* `${PROJECT}_ClangFormat_EXCLUDES_RE`: list of regular expressions to exclude C/C++ files
  from formatting. Default value is:<br/>
  `".*/third[-_]parties/.*$$" ".*/third[-_]party/.*$$"`
* `${PROJECT}_ClangFormat_DEPENDENCIES`: list of CMake targets to build before
  formatting C/C++ code. Default value is `""`

Where `${PROJECT}_FORMATTING` CMake variable is supposed to be defined by the user,
the variables above are meant to be overridden inside CMake project directly.
They are CMake CACHE variables whose value must be forced.
For instance, to ignore code of third-parties located in `ext/` subdirectory,
add this to your CMake project:

```cmake
set(
  foo_ClangFormat_EXCLUDES_RE "${PROJECT_SOURCE_DIR}/ext/.*$$"
  CACHE STRING "list of regular expressions to exclude C/C++ files from formatting"
  FORCE)
```

#### Pre-Commit

Enable CMake variable `${PROJECT}_PRECOMMIT` to enable automatic checks before git commits.

For instance, given a project `foo`:

`cmake -Dfoo_PRECOMMIT:BOOL=ON <path>`

if `${PROJECT}_FORMATTING` CMake variable is enabled, when performing a git commit,
a serie of checks will be executed to ensure that your change
complies with the coding conventions. It will be discarded otherwise.