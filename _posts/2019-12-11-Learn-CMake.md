---
layout: post
title: Learn CMake
---

## [CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

### A Basic Starting Point (Step 1)

#### create `CMakeLists.txt` file

    cmake_minimum_required(VERSION 3.10)

    # add the executable
    add_executable(Tutorial tutorial.cxx)

    # set the project name and version
    project(Tutorial VERSION 1.0)

    # configure a header file to pass the version number to the source code
    configure_file(TutorialConfig.h.in TutorialConfig.h)

    # add build directory to the list of paths to search for include files
    target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")

    # specify the C++ standard
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED True)

##### illustrate

[`add_executable()`](https://cmake.org/cmake/help/v3.0/command/add_executable.html?highlight=add_executable)

> Add an executable to the project using the specified source files.
>
>     add_executable(<name> [WIN32] [MACOSX_BUNDLE]
>                    [EXCLUDE_FROM_ALL]
>                    source1 [source2 ...])
> Adds an executable target called `<name>` to be built from the source files listed in the command invocation. The `<name>` corresponds to the logical target name and must be globally unique within a project. The actual file name of the executable built is constructed based on conventions of the native platform (such as `<name>.exe` or just `<name>`.
>  
> By default the executable file will be created in the **build tree directory** corresponding to the source tree directory in which the command was invoked. See documentation of the [`RUNTIME_OUTPUT_DIRECTORY`](https://cmake.org/cmake/help/v3.0/prop_tgt/RUNTIME_OUTPUT_DIRECTORY.html#prop_tgt:RUNTIME_OUTPUT_DIRECTORY) target property to change this location. See documentation of the [`OUTPUT_NAME`](https://cmake.org/cmake/help/v3.0/prop_tgt/OUTPUT_NAME.html#prop_tgt:OUTPUT_NAME) target property to change the `<name>` part of the final file name.

[`project()`](https://cmake.org/cmake/help/v3.0/command/project.html?highlight=project)

> Set a name, version, and enable languages for the entire project.  
>
>     project(<PROJECT-NAME> [LANGUAGES] [<language-name>...])
>     project(<PROJECT-NAME>
>             [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
>             [LANGUAGES <language-name>...])
> Sets the name of the project and stores the name in the PROJECT_NAME variable. Additionally this sets variables
>
> * `PROJECT_SOURCE_DIR`, `<PROJECT-NAME>_SOURCE_DIR`
> * `PROJECT_BINARY_DIR`, `<PROJECT-NAME>_BINARY_DIR`
>
> The `project()` command stores the version number and its components in variables
>
> * `PROJECT_VERSION`, `<PROJECT-NAME>_VERSION`
> * `PROJECT_VERSION_MAJOR`, `<PROJECT-NAME>_VERSION_MAJOR`
> * `PROJECT_VERSION_MINOR`, `<PROJECT-NAME>_VERSION_MINOR`
> * `PROJECT_VERSION_PATCH`, `<PROJECT-NAME>_VERSION_PATCH`
> * `PROJECT_VERSION_TWEAK`, `<PROJECT-NAME>_VERSION_TWEAK`
>
> The top-level `CMakeLists.txt` file for a project must contain a literal, direct call to the `project()` command; loading one through the `include()` command is not sufficient. If no such call exists CMake will implicitly add one to the top that enables the default languages (`C` and `CXX`).

[`configure_file()`](https://cmake.org/cmake/help/latest/command/configure_file.html?highlight=configure_file)
> Copy a file to another location and modify its contents.
>
>     configure_file(<input> <output>
>                    [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
>                    [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
> Copies an `<input>` file to an `<output>` file and substitutes variable values referenced as `@VAR@` or `${VAR}` in the input file content. Each variable reference will be replaced with the current value of the variable, or the empty string if the variable is not defined. Furthermore, input lines of the form
>
>     #cmakedefine VAR ...
> will be replaced with either
>
>     #define VAR ...
> or
>
>     /* #undef VAR */
> depending on whether `VAR` is set in CMake to any value not considered a false constant by the `if()` command. The `“…”` content on the line after the variable name, if any, is processed as above. Input file lines of the form `#cmakedefine01 VAR` will be replaced with either `#define VAR 1` or `#define VAR 0` similarly.
> If the input file is modified the build system will re-run CMake to re-configure the file and generate the build system again. The generated file is modified and its timestamp updated on subsequent cmake runs only if its content is changed.

[`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html?highlight=target_include_directories)
> Add include directories to a target.
>
>     target_include_directories(<target> [SYSTEM] [BEFORE]
>       <INTERFACE|PUBLIC|PRIVATE> [items1...]
>       [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
> Specifies include directories to use when compiling a given target. The named `<target>` must have been created by a command such as `add_executable()` or `add_library()` and must not be an `ALIAS target`.

#### create `TutorialConfig.h.in` file in the source directory

    // the configured options and settings for Tutorial
    #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
    #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@

#### create `tutorial.cxx` file includes the configured header file, `TutorialConfig.h`

    #include <iostream>
    #include "TutorialConfig.h"

    int main(int argc, char* argv[])
    {
        std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;
        return 0;
    }

#### Build and Test

    # mkdir build
    # cd build
    # cmake ../
    # cmake --build .
    # ./Tutorial

### Adding a Library (Step 2)

`CMakeLists.txt`

    cmake_minimum_required(VERSION 3.10)

    # add the executable
    add_executable(Tutorial tutorial.cxx)

    # set the project name and version
    project(Tutorial VERSION 1.0)

    # put this before configure_file()
    option(USE_FOO "Use foo" ON)

    # configure a header file to pass the version number to the source code
    configure_file(TutorialConfig.h.in TutorialConfig.h)

    # add build directory to the list of paths to search for include files
    target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")

    # specify the C++ standard
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED True)

    if(USE_FOO)
      add_subdirectory(foo)
      list(APPEND EXTRA_LIBS foo)
      list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/foo")
    endif()

    target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

    # add the binary tree to the search path for include files so that we will find TutorialConfig.h
    target_include_directories(Tutorial PUBLIC
                               "${PROJECT_BINARY_DIR}"
                               ${EXTRA_INCLUDES}
                               )

`foo/CMakeLists.txt`

    add_library(foo foo.cxx)

`foo/foo.h`

    #include <string>
    void FooFunc(std::string&& s);

`foo/foo.cxx`

    #include <iostream>
    #include "foo.h"

    void FooFunc(std::string&& s)
    {
        std::cout << "call FooFunc(" << s << ")" << std::endl;
    }

`TutorialConfig.h.in`

    // the configured options and settings for Tutorial
    #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
    #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@

    #cmakedefine USE_FOO

`tutorial.cxx`

    #include <iostream>
    #include "TutorialConfig.h"

    #ifdef USE_FOO
    #  include "foo.h"
    #endif

    int main(int argc, char* argv[])
    {
        std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;

        #ifdef USE_FOO
        FooFunc("s");
        #endif

        return 0;
    }

### Adding Usage Requirements for Library (Step 3)

**Usage requirements** allow for far better control over a library or executable’s **link** and **include** line while also giving more control over the **transitive property** of targets inside CMake. The primary commands that leverage usage requirements are:

* `target_compile_definitions`
* `target_compile_options`
* `target_include_directories`
* `target_link_libraries`

refactor the code to use the modern CMake approach of **usage requirements**. First state that anybody linking to `foo` needs to include the current source directory, while `foo` itself doesn’t. So this can become an `INTERFACE` usage requirement.

Remember `INTERFACE` means things that consumers require but the producer doesn’t. Add the following lines to the end of `foo/CMakeLists.txt`:

    target_include_directories(foo
              INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
              )

remove the uses of the `EXTRA_INCLUDES` variable from the top-level `CMakeLists.txt`, here:

    if(USE_MYMATH)
      add_subdirectory(foo)
      list(APPEND EXTRA_LIBS foo)
    endif()

And here:

    target_include_directories(Tutorial PUBLIC
                               "${PROJECT_BINARY_DIR}"
                               )

`CMakeLists.txt`

    cmake_minimum_required(VERSION 3.10)

    # add the executable
    add_executable(Tutorial tutorial.cxx)

    # set the project name and version
    project(Tutorial VERSION 1.0)

    # put this before configure_file()
    option(USE_FOO "Use foo" ON)

    # configure a header file to pass the version number to the source code
    configure_file(TutorialConfig.h.in TutorialConfig.h)

    # add build directory to the list of paths to search for include files
    target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")

    # specify the C++ standard
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED True)

    if(USE_FOO)
      add_subdirectory(foo)
      list(APPEND EXTRA_LIBS foo)
    endif()

    target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

    # add the binary tree to the search path for include files so that we will find TutorialConfig.h
    target_include_directories(Tutorial PUBLIC
                               "${PROJECT_BINARY_DIR}"
                               )

`foo/CMakeLists.txt`

    add_library(foo foo.cxx)
    target_include_directories(foo
        INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
        )

### Installing and Testing (Step 4)

#### Install Rules

The install rules for `foo`: install the **library** and **header file**.  
The install rules for `Tutorial`: install the **executable** and **configured header**.

`foo/CMakeLists.txt`

    add_library(foo foo.cxx)
    target_include_directories(foo
        INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
        )
    
    install(TARGETS foo DESTINATION lib)
    install(FILES foo.h DESTINATION include)

`CMakeLists.txt`

    cmake_minimum_required(VERSION 3.10)

    # add the executable
    add_executable(Tutorial tutorial.cxx)

    # set the project name and version
    project(Tutorial VERSION 1.0)

    # put this before configure_file()
    option(USE_FOO "Use foo" ON)

    # configure a header file to pass the version number to the source code
    configure_file(TutorialConfig.h.in TutorialConfig.h)

    # add build directory to the list of paths to search for include files
    target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")

    # specify the C++ standard
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED True)

    if(USE_FOO)
      add_subdirectory(foo)
      list(APPEND EXTRA_LIBS foo)
    endif()

    target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

    # add the binary tree to the search path for include files so that we will find TutorialConfig.h
    target_include_directories(Tutorial PUBLIC
                               "${PROJECT_BINARY_DIR}"
                               )

    install(TARGETS Tutorial DESTINATION bin)
    install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
            DESTINATION include
           )

Run cmake or cmake-gui to configure the project and then build it with your chosen build tool. Run the install step by typing cmake --install . (introduced in 3.15, older versions of CMake must use make install) from the command line, or build the INSTALL target from an IDE. This will install the appropriate header files, libraries, and executables.

The CMake variable CMAKE_INSTALL_PREFIX is used to determine the root of where the files will be installed. If using cmake --install a custom installation directory can be given via --prefix argument. For multi-configuration tools, use the --config argument to specify the configuration.

##### illustrate

`CMAKE_INSTALL_PREFIX`

> Install directory used by `install()`.
>
> If `make install` is invoked or `INSTALL` is built, this directory is prepended onto all install directories. This variable defaults to `/usr/local` on UNIX and `c:/Program Files/${PROJECT_NAME}` on Windows. See `CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT` for how a project might choose its own default.
>
> On UNIX one can use the `DESTDIR` mechanism in order to relocate the whole installation. See `DESTDIR` for more information.
>
> The installation prefix is also added to `CMAKE_SYSTEM_PREFIX_PATH` so that `find_package()`, `find_program()`, `find_library()`, `find_path()`, and `find_file()` will search the prefix for other software.

see: [How to use CMAKE_INSTALL_PREFIX](https://stackoverflow.com/questions/6241922/how-to-use-cmake-install-prefix)

##### build

    # mkdir build
    # cd build
    # mkdir install
    # cmake -DCMAKE_INSTALL_PREFIX=./install ..
    # cmake --build .
    # make install

#### Testing Support

At the end of the top-level CMakeLists.txt file we can enable testing and then add a number of basic tests to verify that the application is working correctly.

### Adding System Introspection (Step 5)

## [An Introduction to Modern CMake](https://cliutils.gitlab.io/modern-cmake/)  

#### Why do I need a good build system?

Do any of the following apply to you?

* You want to avoid **hard-coding paths**
* You need to build a package on more than one computer
* You want to use CI (continuous integration)
* You need to support different OSs (maybe even just flavors of Unix)
* You want to support multiple compilers
* You want to use an IDE, but maybe not all of the time
* You want to describe how your program is structured logically, not flags and commands
* You want to use a library
* You want to use tools, like Clang-Tidy, to help you code
* You want to use a debugger

If so, you'll benefit from a CMake-like build system.

##### Other sources

[It's time to do CMake Right](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/): A nice set of best practices for Modern CMake projects.

[The Ultimate Guide to Modern CMake](https://rix0r.nl/blog/2015/08/13/cmake-guide/): A slightly dated post with similar intent.

[toeb/moderncmake](https://github.com/toeb/moderncmake): A nice presentation and examples about CMake 3.5+, with intro to syntax through project organization

#### Running CMake

##### Building a project

Unless otherwise noted, you should always make a **build directory** and build from there. You can technically do an in-source build, but you'll have to be careful not to overwrite files or add them to git, so just don't.

Here's the Classic CMake Build Procedure (TM):

    ~/package $ mkdir build
    ~/package $ cd build
    ~/package/build $ cmake ..
    ~/package/build $ make

You can replace the make line with `cmake --build .` if you'd like, and it will call `make` or whatever build tool you are using. If you are using a newer version of CMake (which you usually should be, except for checking compatibility with older CMake), you can instead do this:

    ~/package $ cmake -S . -B build
    ~/package $ cmake --build build

Any one of these commands will install:

    # From the build directory (pick one)
    ~/package/build $ make install
    ~/package/build $ cmake --build . --target install
    ~/package/build $ cmake --install . # CMake 3.15+ only

    # From the source directory (pick one)
    ~/package $ cmake --build build --target install
    ~/package $ cmake --install build # CMake 3.15+ only

##### Picking a compiler

Selecting a compiler must be done on the **first run** in an empty directory. It's not CMake syntax per se, but you might not be familiar with it. To pick Clang:

    ~/package/build $ CC=clang CXX=clang++ cmake ..

That sets the environment variables in bash for `CC` and `CXX`, and CMake will respect those variables. This sets it just for that one line, but that's the only time you'll need those; afterwards CMake continues to use the paths it deduces from those values.

##### Picking a generator

You can build with a variety of tools; `make` is usually the default. To see all the tools CMake knows about on your system, run

    ~/package/build $ cmake --help

And you can pick a tool with `-G"My Tool"` (quotes only needed if spaces are in the tool name). You should pick a tool on your **first CMake call** in a directory, just like the compiler. **Feel free to have several build directories**, like `build/` and `buildXcode`. You can set the environment variable `CMAKE_GENERATOR` to control the default generator (CMake 3.15+). Note that makefiles will only run in **parallel** if you explicilty pass a number of threads, such as `make -j2`, while Ninja will automatically run in parallel. You can directly pass a parallelization option such as `-j2` to the `cmake --build .` command in recent versions of CMake.

##### Setting options

You set options in CMake with `-D`. You can see a list of options with `-L`, or a list with human-readable help with `-LH`. If you don't list the `source/build` directory, the listing will not rerun CMake (`cmake -L` instead of `cmake -L .)`.

##### Verbose and partial builds

Again, not really CMake, but if you are using a command line build tool like `make`, you can get verbose builds:

    ~/package/build $ VERBOSE=1 make

You can actually write `make VERBOSE=1`, and `make` will also do the right thing, though that's a feature of `make` and not the command line in general.

You can also build just a part of a build by specifying a target, such as the name of a library or executable you've defined in CMake, and make will just build that target.

##### Options

CMake has support for cached options. A Variable in CMake can be marked as "cached", which means it will be written to the cache (a file called `CMakeCache.txt` in the build directory) when it is encountered. You can preset (or change) the value of a cached option on the command line with `-D`. When CMake looks for a cached variable, it will use the existing value and will not overwrite it.

##### Standard options

These are common CMake options to most packages:

`-DCMAKE_BUILD_TYPE=` Pick from `Release`, `RelWithDebInfo`, `Debug`, or sometimes more.
`-DCMAKE_INSTALL_PREFIX=` The location to install to. System install on `UNIX` would often be `/usr/local` (the default), user directories are often `~/.local`, or you can pick a folder.

`-DBUILD_SHARED_LIBS=` You can set this `ON` or `OFF` to control the default for shared libraries (the author can pick one vs. the other explicitly instead of using the default, though)

`-DBUILD_TESTING=` This is a common name for enabling tests, not all packages use it, though, sometimes with good reason.

##### Debugging your CMake files

We've already mentioned verbose output for the build, but you can also see verbose CMake configure output too. The `--trace` option will print every line of CMake that is run. Since this is very verbose, CMake 3.7 added `--trace-source="filename"`, which will print out every executed line of just the file you are interested in when it runs. If you select the name of the file you are interested in debugging (usually by selecting the parent directory when debugging a `CMakeLists.txt`, since all of those have the same name), you can just see the lines that run in that file. Very useful!

#### Do's and Don'ts

##### CMake Antipatterns

The next two lists are heavily based on the excellent gist [Effective Modern CMake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1). That list is much longer and more detailed, feel free to read it as well.

* **Do not use global functions**: This includes `link_directories`, `include_libraries`, and similar.
* **Don't add unneeded PUBLIC requirements**: You should avoid forcing something on users that is not required (`-Wall`). Make these `PRIVATE` instead.
* **Don't GLOB files**: Make or another tool will not know if you add files without rerunning CMake. Note that CMake 3.12 adds a `CONFIGURE_DEPENDS` flag that makes this far better if you need to use it.
* **Link to built files directly**: Always link to targets if available.
* **Never skip `PUBLIC/PRIVATE` when linking**: This causes all future linking to be keyword-less.

##### CMake Patterns

* **Treat CMake as code**: It is code. It should be as clean and readable as all other code.
* **Think in targets**: Your targets should represent concepts. Make an (`IMPORTED`) `INTERFACE` target for anything that should stay together and link to that.
* **Export your interface**: You should be able to run from `build` or `install`.
* **Write a `Config.cmake` file**: This is what a library author should do to support clients.
* **Make `ALIAS` targets to keep usage consistent**: Using `add_subdirectory` and `find_package` should provide the same targets and namespaces.
* **Combine common functionality into clearly documented functions or macros**: Functions are better usually.
* **Use lowercase function names**: CMake functions and macros can be called lower or upper case. Always user lower case. Upper case is for variables.
* **Use `cmake_policy` and/or range of versions**: Policies change for a reason. Only piecemeal set OLD policies if you have to.

### Introduction to the basics

##### Minimum Version

Here's the first line of every `CMakeLists.txt`, which is the required name of the file CMake looks for:

    cmake_minimum_required(VERSION 3.1)

Let's mention a bit of CMake syntax. The command name `cmake_minimum_required` is case insensitive, so the common practice is to use lower case. The `VERSION` is a special keyword for this function. And the value of the version follows the keyword.

This line is special! The version of CMake will also dictate the policies, which define behavior changes. So, if you set `minimum_required` to VERSION 2.8, you'll get the wrong linking behavior on macOS, for example, even in the newest CMake versions. If you set it to 3.3 or less, you'll get the wrong hidden symbols behaviour, etc. A list of policies and versions is available at [policies](https://cliutils.gitlab.io/modern-cmake/chapters/basics.html).

In CMake 3.12, this will support a range, such as `VERSION 3.1...3.12`; this means you support as low as 3.1 but have also tested it with the new policy settings up to 3.12. This is much nicer on users that need the better settings, and due to a trick in the syntax, it's backward compatible with older versions of CMake (though actually running CMake 3.2-3.11 will only set the 3.1 version of the policies in this example). New versions of policies tend to be most important for macOS and Windows users, who also usually have a very recent version of CMake.

This is what new projects should do:

    cmake_minimum_required(VERSION 3.1...3.15)

    if(${CMAKE_VERSION} VERSION_LESS 3.12)
        cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
    endif()

If CMake version is less than `3.12`, the if block will be true, and the policy will be set to the current CMake version. If CMake is `3.12` or higher, the if block will be false, but the new syntax in `cmake_minimum_required` will be respected and this will continue to work properly!

##### Setting a project

Now, every top-level CMake file will have the next line:

    project(MyProject VERSION 1.0
                      DESCRIPTION "Very nice project"
                      LANGUAGES CXX)

Now we see even more syntax. Strings are quoted, whitespace doesn't matter, and the name of the project is the first argument (positional). All the keyword arguments here are optional. The version sets a bunch of variables, like `MyProject_VERSION` and `PROJECT_VERSION`. The languages are `C`, `CXX`, `Fortran`, `ASM`, `CUDA` (CMake 3.8+), `CSharp` (3.8+), and `SWIFT` (CMake 3.15+ experimental). `C` `CXX` is the default. In CMake 3.9, `DESCRIPTION` was added to set a project description, as well.

##### Making an executable

Although libraries are much more interesting, and we'll spend most of our time with them, let's start with a simple executable.

    add_executable(one two.cpp three.h)

There are several things to unpack here. one is both the name of the executable file generated, and the name of the CMake target created (you'll hear a lot more about targets soon, I promise). The source file list comes next, and you can list **as many as you'd like**. CMake is smart, and will only compile source file extensions. The headers will be, for most intents and purposes, **ignored**; the only reason to list them is to get them to show up in IDEs. Targets show up as folders in many IDEs. More about the general build system and targets is available at [buildsystem](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html).

##### Making a library

Making a library is done with add_library, and is just about as simple:

    add_library(one STATIC two.cpp three.h)

You get to pick a type of library, `STATIC`, `SHARED`, or `MODULE`. If you leave this choice off, the value of `BUILD_SHARED_LIBS` will be used to pick between `STATIC` and `SHARED`.

As you'll see in the following sections, often you'll need to make a **fictional target**, that is, one where nothing needs to be compiled, for example, for a header-only library. That is called an `INTERFACE` library, and is another choice; the only difference is it cannot be followed by filenames.

You can also make an `ALIAS` library with an existing library, which simply gives you a new name for a target. The one benefit to this is that you can make libraries with `::` in the name

###### Targets are your friend

Now we've specified a target, how do we **add information** about it? For example, maybe it needs an include directory:

    target_include_directories(one PUBLIC include)

`target_include_directories` adds an include directory to a target. `PUBLIC` doesn't mean much for an executable; for a library it lets CMake know that any targets that **link to** this target must also need that include directory. Other options are `PRIVATE` (only affect the current target, not **dependencies**), and `INTERFACE` (only needed for **dependencies**).

We can then chain targets:

    add_library(another STATIC another.cpp another.h)
    target_link_libraries(another PUBLIC one)

`target_link_libraries` is probably the most useful and confusing command in CMake. It takes a target (another) and adds a **dependency** if a target is given. If no target of that name (one) exists, then it adds a **link to** a library called one on your path (hence the name of the command). Or you can give it a full path to a library. Or a **linker flag**. Just to add a final bit of confusion, classic CMake allowed you to **skip** the keyword selection of `PUBLIC`, etc. If this was done on a target, you'll get an error if you try to mix styles further down the chain.

Focus on using targets everywhere, and keywords everywhere, and you'll be fine.

Targets can have **include directories**, **linked libraries** (or **linked targets**), **compile options**, **compile definitions**, **compile features** (see the C++11 chapter), and more. As you'll see in the two including projects chapters, you can often get targets (and always make targets) to represent all the libraries you use. Even things that are not true libraries, like OpenMP, can be **represented** with targets. This is why Modern CMake is great!

##### Dive in

See if you can follow the following file. It makes a simple C++11 library and a program using it. No dependencies. I'll discuss more C++ standard options later, using the CMake 3.8 system for now.

    cmake_minimum_required(VERSION 3.8)

    project(Calculator LANGUAGES CXX)

    add_library(calclib STATIC src/calclib.cpp include/calc/lib.hpp)
    target_include_directories(calclib PUBLIC include)
    target_compile_features(calclib PUBLIC cxx_std_11)

    add_executable(calc apps/calc.cpp)
    target_link_libraries(calc PUBLIC calclib)

#### Variables and the Cache

##### Local Variables

We will cover variables first. A local variable is set like this:

    set(MY_VARIABLE "value")

The names of variables are usually all caps, and the value follows. You access a variable by using `${}`, such as `${MY_VARIABLE}`. CMake has the concept of scope; you can access the value of the variable after you set it as long as you are in the same scope. If you leave a function or a file in a sub directory, the variable will no longer be defined. You can set a variable in the scope immediately above your current one with `PARENT_SCOPE` at the end.

Lists are simply a series of values when you set them:

    set(MY_LIST "one" "two")

which internally become `;` separated values. So this is an identical statement:

    set(MY_LIST "one;two")

The `list(` command has utilities for working with lists, and `separate_arguments` will turn a space separated string into a list (inplace). Note that an unquoted value in CMake is the same as a quoted one if there are **no spaces** in it; this allows you to skip the quotes most of the time when working with value that you know could not contain spaces.

When a variable is expanded using `${}` syntax, all the same rules about spaces apply. Be especially careful with paths; paths may contain a space at any time and should always be quoted when they are a variable (never write `${MY_PATH}`, always should be `"${MY_PATH}"`).

##### Cache Variables

If you want to set a variable from the command line, CMake offers a variable cache. Some variables are already here, like `CMAKE_BUILD_TYPE`. The syntax for declaring a variable and setting it if it is not already set is:

    set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")

This will **not** replace an existing value. This is so that you can set these on the command line and not have them overridden when the CMake file executes. If you want to use these variables as a **make-shift global variable**, then you can do:

    set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "" FORCE)
    mark_as_advanced(MY_CACHE_VARIABLE)

The first line will cause the value to be set no matter what, and the second line will keep the variable from showing up in the list of variables if you run `cmake -L ..` or use a GUI. This is so common, you can also use the `INTERNAL` type to do the same thing (though technically it forces the `STRING` type, this won't affect any CMake code that depends on the variable):

    set(MY_CACHE_VARIABLE "VALUE" CACHE INTERNAL "")

Since `BOOL` is such a common variable type, you can set it more succinctly with the shortcut:

    option(MY_OPTION "This is settable from the command line" OFF)

For the `BOOL` datatype, there are several different wordings for `ON` and `OFF`.

##### Environment variables

You can also `set(ENV{variable_name} value)` and get `$ENV{variable_name}` environment variables, though it is generally a very good idea to **avoid them**.

##### The Cache

The cache is actually just a text file, `CMakeCache.txt`, that gets created in the build directory when you run CMake. This is how CMake remembers anything you set, so you don't have to re-list your options every time you rerun CMake.

##### Properties

The other way CMake stores information is in properties. This is like a variable, but it is attached to some other item, like a **directory** or a **target**. A global property can be a useful uncached global variable. Many target properties are initialized from a matching variable with `CMAKE_` at the front. So setting `CMAKE_CXX_STANDARD`, for example, will mean that all new targets created will have `CXX_STANDARD` set to that when they are created. There are two ways to set properties:

    set_property(TARGET TargetName
                 PROPERTY CXX_STANDARD 11)

    set_target_properties(TargetName PROPERTIES
                          CXX_STANDARD 11)

The first form is more general, and can set **multiple** targets/files/tests at once, and has useful options. The second is a shortcut for setting several properties on one target. And you can get properties similarly:

    get_property(ResultVariable TARGET TargetName PROPERTY CXX_STANDARD)

See [cmake-properties](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html) for a listing of all known properties.

#### Programming in CMake

##### Control flow

CMake has an `if` statement, though over the years it has become rather complex. There are a series of all caps keywords you can use inside an `if` statement, and you can often refer to variables by either directly by name or using the `${}` syntax (the if statement historically predates variable expansion). An example if statement:

    if(variable)
        # If variable is `ON`, `YES`, `TRUE`, `Y`, or non zero number
    else()
        # If variable is `0`, `OFF`, `NO`, `FALSE`, `N`, `IGNORE`, `NOTFOUND`, `""`, or ends in `-NOTFOUND`
    endif()
    # If variable does not expand to one of the above, CMake will expand it then try again

Since this can be a little confusing if you explicitly put a variable expansion, like `${variable}`, due to the potential expansion of an expansion, a policy (CMP0054) was added in CMake 3.1+ that keeps a quoted expansion from being expanded yet again. So, as long as the minimum version of CMake is 3.1+, you can do:

    if("${variable}")
        # True if variable is not false-like
    else()
        # Note that undefined variables would be `""` thus false
    endif()

There are a variety of keywords as well, such as:

* Unary: `NOT`, `TARGET`, `EXISTS (file)`, `DEFINED`, etc.
* Binary: `STREQUAL`, `AND`, `OR`, `MATCHES` (regular expression), `VERSION_LESS`, `VERSION_LESS_EQUAL` (CMake 3.7+), etc.
* Parentheses can be used to group

##### Generator-expressions

Generator-expressions are really powerful, but a bit odd and specialized. Most CMake commands happen at configure time, include the if statements seen above. But what if you need logic to occur at build time or even install time? Generator expressions were added for this purpose. They are evaluated in target properties.

The simplest generator expressions are **informational expressions**, and are of the form `$<KEYWORD>`; they evaluate to a piece of **information relevant** for the current configuration. The other form is `$<KEYWORD:value>`, where `KEYWORD` is a keyword that controls the evaluation, and value is the item to evaluate (an informational expression keyword is allowed here, too). If `KEYWORD` is a generator expression or variable that evaluates to `0` or `1`, value is substituted if `1` and not if `0`. You can nest generator expressions, and you can use variables to make reading nested variables bearable. Some expressions allow multiple values, separated by commas.

If you want to put a compile flag only for the `DEBUG` configuration, for example, you could do:

    target_compile_options(MyTarget PRIVATE "$<$<CONFIG:Debug>:--my-flag>")

This is a newer, better way to add things than using specialized `*_DEBUG` variables, and generalized to all the things generator expressions support. Note that you should never, never use the configure time value for the current configuration, because multi-configuration generators like IDEs do not have a "current" configuration at configure time, only at build time through generator expressions and custom `*_<CONFIG>` variables.

Other common uses for generator expressions:

* Limiting an item to a certain language only, such as `CXX`, to avoid it mixing with something like `CUDA`, or wrapping it so that it is different depending on target language.
* Accessing configuration dependent properties, like target file location.
* Giving a different location for build and install directories.
That last one is very common. You'll see something like this in almost every package that supports installing:

    target_include_directories(
        MyTarget
     PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    )

##### Macros and Functions

You can define your own CMake `function` or `macro` easily. The only difference between a `function` and a `macro` is scope; `macros` don't have one. So, if you set a variable in a `function` and want it to be visible outside, you'll need `PARENT_SCOPE`. Nesting functions therefore is a bit tricky, since you'll have to explicitly set the variables you want visible to the outside world to `PARENT_SCOPE` in each `function`. But, functions don't "leak" all their variables like `macros` do. For the following examples, I'll use functions.

An example of a simple `function` is as follows:

    function(SIMPLE REQUIRED_ARG)
        message(STATUS "Simple arguments: ${REQUIRED_ARG}, followed by ${ARGV}")
        set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
    endfunction()

    simple(This)
    message("Output: ${This}")

If you want positional arguments, they are listed explicitly, and all other arguments are collected in `ARGN` (`ARGV` holds all arguments, even the ones you list). You have to work around the fact that CMake does not have return values by setting variables. In the example above, you can explicitly give a variable name to set.

##### Arguments

CMake has a named variable system that you've already seen in most of the build in CMake functions. You can use it with the `cmake_parse_arguments` function. If you want to support a version of CMake less than 3.5, you'll want to also include the `CMakeParseArguments` module, which is where it used to live before becoming a built in command. Here is an example of how to use it:

    function(COMPLEX)
        cmake_parse_arguments(
            COMPLEX_PREFIX
            "SINGLE;ANOTHER"
            "ONE_VALUE;ALSO_ONE_VALUE"
            "MULTI_VALUES"
            ${ARGN}
        )
    endfunction()

    complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)

Inside the function after this call, you'll find:

    COMPLEX_PREFIX_SINGLE = TRUE
    COMPLEX_PREFIX_ANOTHER = FALSE
    COMPLEX_PREFIX_ONE_VALUE = "value"
    COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
    COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"

If you look at the official page, you'll see a slightly different method using set to avoid explicitly writing the semicolons in the list; feel free to use the structure you like best. You can mix it with the positional arguments listed above; any remaining arguments (therefore optional positional arguments) are in `COMPLEX_PREFIX_UNPARSED_ARGUMENTS`.

#### Communication with your code

##### Configure File

CMake allows you to access CMake variables from your code using `configure_file`. This command copies a file (traditionally ending in `.in` from one place to another, substituting all CMake variables it finds. If you want to avoid replacing existing `${}` syntax in your input file, use the `@ONLY` keyword. There's also a `COPY_ONLY` keyword if you are just using this as a replacement for `file(COPY`.

This functionality is used quite frequently; for example, on `Version.h.in`:

Version.h.in

    #pragma once

    #define MY_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
    #define MY_VERSION_MINOR @PROJECT_VERSION_MINOR@
    #define MY_VERSION_PATCH @PROJECT_VERSION_PATCH@
    #define MY_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
    #define MY_VERSION "@PROJECT_VERSION@"

CMake lines:

    configure_file (
        "${PROJECT_SOURCE_DIR}/include/My/Version.h.in"
        "${PROJECT_BINARY_DIR}/include/My/Version.h"
    )

You should include the binary include directory as well when building your project. If you want to put any true/false variables in a header, CMake has C specific `#cmakedefine` and `#cmakedefine01` replacements to make appropriate define lines.

You can also (and often do) use this to produce `.cmake` files, such as the configure files (see the section on configuring).

##### Reading files

The other direction can be done too; you can read in something (like a version) from your source files. If you have a header only library that you'd like to make available with or without CMake, for example, then this would be the best way to handle a version. This would look something like this:

    # Assuming the canonical version is listed in a single line
    # This would be in several parts if picking up from MAJOR, MINOR, etc.
    set(VERSION_REGEX "#define MY_VERSION[ \t]+\"(.+)\"")

    # Read in the line containing the version
    file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/include/My/Version.hpp"
    VERSION_STRING REGEX ${VERSION_REGEX})

    # Pick out just the version
    string(REGEX REPLACE ${VERSION_REGEX} "\\1" VERSION_STRING "${VERSION_STRING}")

    # Automatically getting PROJECT_VERSION_MAJOR, My_VERSION_MAJOR, etc.
    project(My LANGUAGES CXX VERSION ${VERSION_STRING})

Above, file(STRINGS file_name variable_name REGEX regex) picks lines that match a regex; and the same regex is used to then pick out the parentheses capture group with the version part. Replace is used with back substitution to output only that one group.

#### How to structure your project

The following information is biased. But in a good way, I think. I'm going to tell you how to structure the directories in your project. This is based on convention, but will help you:

* Easily read other projects following the same patters,
* Avoid a pattern that causes conflicts,
* Keep from muddling and complicating your build.

First, this is what your files should look like when you start if your project is creatively called `project`, with a library called `lib`, and a executable called `app`:

    - project
      - .gitignore
      - README.md
      - LICENCE.md
      - CMakeLists.txt
      - cmake
        - FindSomeLib.cmake
        - something_else.cmake
      - include
        - project
          - lib.hpp
      - src
        - CMakeLists.txt
        - lib.cpp
      - apps
        - CMakeLists.txt
        - app.cpp
      - tests
        - CMakeLists.txt
        - testlib.cpp
      - docs
        - CMakeLists.txt
      - extern
        - googletest
      - scripts
        - helper.py

The names are not absolute; you'll see contention about `test/` vs. `tests/`, and the application folder may be called something else (or not exist for a library-only project). You'll also sometime see a `python` folder for python bindings, or a `cmake` folder for helper CMake files, like `Find<library>.cmake` files. But the basics are there.

Notice a few things already apparent; the `CMakeLists.txt` files are split up over all source directories, and are **not in the include directories**. This is because you should be able to copy the contents of the include directory to /usr/include or similar directly (except for configuration headers, which I go over in another chapter), and not have any extra files or cause any conflicts. That's also why there is a directory for your project inside the include directory. Use `add_subdirectory` to add a subdirectory containing a `CMakeLists.txt`.

You often want a `cmake` folder, with all of your **helper modules**. This is where your `Find*.cmake` files go. An set of some common helpers is at [github.com/CLIUtils/cmake](https://github.com/CLIUtils/cmake). To add this folder to your CMake path:

Your `extern` folder should contain **git submodules** almost exclusively. That way, you can control the version of the dependencies explicitly, but still upgrade easily. See the Testing chapter for an example of adding a submodule.

You should have something like `/build*` in your `.gitignore`, so that users can make build directories in the source directory and use those to build. A few packages prohibit this, but it's much better than doing a true out-of-source build and having to type something different for each package you build.

If you want to avoid the build directory being in a valid source directory, add this near the top of your CMakeLists:

    ### Require out-of-source builds
    file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
    if(EXISTS "${LOC_PATH}")
        message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
    endif()

See the [extended code example here](https://gitlab.com/CLIUtils/modern-cmake/tree/master/examples/extended-project).

#### Running other programs

##### Running a command at configure time

Running a command at configure time is relatively easy. Use `execute_process` to run a process and access the results. It is generally a good idea to avoid hard coding a program path into your CMake; you can use `${CMAKE_COMMAND}`, `find_package(Git)`, or `find_program` to get access to a command to run. Use `RESULT_VARIABLE` to check the return code and `OUTPUT_VARIABLE` to get the output.

Here is an example that updates all git submodules:

    find_package(Git QUIET)

    if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()

##### Running a command at build time

Build time commands are a bit tricker. The main complication comes from the target system; when do you want your command to run? Does it produce an output that another target needs? With this in mind, here is an example that calls a Python script to generate a header file:

    find_package(PythonInterp REQUIRED)
    add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp"
        COMMAND "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/scripts/GenerateHeader.py" --argument
        DEPENDS some_target)

    add_custom_target(generate_header ALL
        DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp")

    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp DESTINATION include)

Here, the generation happens after `some_target` is complete, and happens when you run make without a target (`ALL`). If you make this a dependency of another target with `add_dependencies`, you could avoid the `ALL` keyword. Or, you could require that a user explicitly builds the `generate_header` target when making.

##### Included common utilities

A useful tool in writing CMake builds that work cross-platform is `cmake -E <mode>` (seen in CMake files as `${CMAKE_COMMAND} -E`). This mode allows CMake to do a variety of things without calling system tools explicitly, like copy, `make_directory`, and `remove`. It is mostly used for the build time commands.

#### A simple example

This is a simple yet complete example of a proper CMakeLists. For this program, we have one library (MyLibExample) with a header file and a source file, and one application, MyExample, with one source file.

    # Almost all CMake files should start with this
    # You should always specify a range with the newest
    # and oldest tested versions of CMake. This will ensure
    # you pick up the best policies.
    cmake_minimum_required(VERSION 3.1...3.16)

    # This is your project statement. You should always list languages;
    # Listing the version is nice here since it sets lots of useful variables
    project(ModernCMakeExample VERSION 1.0 LANGUAGES CXX)

    # If you set any CMAKE_ variables, that can go here.
    # (But usually don't do this, except maybe for C++ standard)

    # Find packages go here.

    # You should usually split this into folders, but this is a simple example

    # This is a "default" library, and will match the *** variable setting.
    # Other common choices are STATIC, SHARED, and MODULE
    # Including header files here helps IDEs but is not required.
    # Output libname matches target name, with the usual extensions on your system
    add_library(MyLibExample simple_lib.cpp simple_lib.hpp)

    # Link each target with other targets or add options, etc.

    # Adding something we can run - Output name matches target name
    add_executable(MyExample simple_example.cpp)

    # Make sure you link your targets with this command. It can also link libraries and
    # even flags, so linking a target that does not exist will not give a configure-time error.
    target_link_libraries(MyExample PRIVATE MyLibExample)

The complete example is available in [examples folder](https://gitlab.com/CLIUtils/modern-cmake/tree/master/examples/simple-project).

A larger, multi-file example is [also available](https://gitlab.com/CLIUtils/modern-cmake/tree/master/examples/extended-project).

### Adding features

##### Default build type

CMake normally does a "non-release, non debug" empty build type; if you prefer to set the default build type yourself, you can follow this recipe for the default build type modified from the [Kitware blog](https://blog.kitware.com/cmake-and-the-default-build-type/):

    set(default_build_type "Release")
    if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
      message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
      set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
          STRING "Choose the type of build." FORCE)
      # Set the possible values of build type for cmake-gui
      set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
        "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
    endif()

#### C++11 and beyond

##### CMake 3.8+: Meta compiler features

As long as you can require that a user install CMake, you'll have access to the newest way to enable C++ standards. This is the most powerful way, with the nicest syntax and the best support for new standards, and the best target behavior for mixing standards and options. Assuming you have a target named myTarget, it looks like this:

    target_compile_features(myTarget PUBLIC cxx_std_11)
    set_target_properties(myTarget PROPERTIES CXX_EXTENSIONS OFF)

For the first line, we get to pick between `cxx_std_11`, `cxx_std_14`, and `cxx_std_17`. The second line is optional, but will avoid extensions being added; without it you'd get things like `-std=g++11` replacing `-std=c++11`. The first line even works on `INTERFACE` targets; only actual compiled targets can use the second line.

If a target further down the dependency chain specifies a higher C++ level, this interacts nicely. It's really just a more advanced version of the following method, so it interacts nicely with that, too.

##### CMake 3.1+: Compiler features

##### CMake 3.1+: Global and property settings

There is another way that C++ standards are supported; a specific set of three properties (both global and target level). The global properties are:

    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)

The first line sets a C++ standard level, and the second tells CMake to use it, and the final line is optional and ensures `-std=c++11` vs. something like `-std=g++11`. This method isn't bad for a final package, but shouldn't be used by a library. You can also set these values on a target:

    set_target_properties(myTarget PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES
        CXX_EXTENSIONS NO
    )

Which is better, but still doesn't have the sort of explicit control that compiler features have for populating `PRIVATE` and `INTERFACE` properties.

You can find more information about the final two methods on [Craig Scott's useful blog post](https://crascit.com/2015/03/28/enabling-cxx11-in-cmake/).

#### Small but common needs

##### Position independent code

This is best known as the `-fPIC` flag. Much of the time, you don't need to do anything. CMake will include the flag for `SHARED` or `MODULE` libraries. If you do explicitly need it:

    set(CMAKE_POSITION_INDEPENDENT_CODE ON)

will do it **globally**, or:

    set_target_properties(lib1 PROPERTIES POSITION_INDEPENDENT_CODE ON)

to explicitly turn it ON (or OFF) for a target.

##### Little libraries

If you need to link to the `dl` library, with `-ldl` on Linux, just use the built-in CMake variable `${CMAKE_DL_LIBS}` in a `target_link_libraries` command. No module or `find_package` needed. (This adds whatever is needed to get `dlopen` and `dlclose`)

Unfortunately, the **math library** is not so lucky. If you need to **explicitly link** to it, you can always do `target_link_libraries(MyTarget PUBLIC m)`, but it might be better to use CMake's generic `find_library`:

    find_library(MATH_LIBRARY m)
    if(MATH_LIBRARY)
        target_link_libraries(MyTarget PUBLIC ${MATH_LIBRARY})
    endif()

You can pretty easily find `Find*.cmake`'s for this and other libraries that you need with a quick search; most major packages have a helper library of CMake modules. See the chapter on existing package inclusion for more.

##### Interprocedural optimization

`INTERPROCEDURAL_OPTIMIZATION`, best known as link time optimization and the `-flto` flag, is available on very recent versions of CMake. You can turn this on with `CMAKE_INTERPROCEDURAL_OPTIMIZATION` (CMake 3.9+ only) or the `INTERPROCEDURAL_OPTIMIZATION` property on targets. Support for GCC and Clang was added in CMake 3.8. If you set `cmake_minimum_required(VERSION 3.9)` or better (see CMP0069), setting this to `ON` on a target is an error if the compiler doesn't support it. You can use `check_ipo_supported()`, from the built-in `CheckIPOSupported` module, to see if support is available before hand. An example of 3.9 style usage:

    include(CheckIPOSupported)
    check_ipo_supported(RESULT result)
    if(result)
      set_target_properties(foo PROPERTIES INTERPROCEDURAL_OPTIMIZATION TRUE)
    endif()

#### CCache and Utilities

##### CCache

##### Utilities

##### Clang tidy

##### Include what you use

##### Link what you use

##### Clang-format

#### Useful Modules

There are a ton of useful modules in CMake's module collection; but some of them are more useful than others. Here are a few highlights.

##### CMakeDependentOption

##### CMakePrintHelpers

##### CheckCXXCompilerFlag

##### WriteCompilerDetectionHeader

##### try_compile/try_run

##### FeatureSummary

#### Supporting IDEs

In general, IDEs are already supported by a standard CMake project. There are just a few extra things that can help IDEs perform even better.

##### Folders for targets

##### Folders for files

##### Running with an IDE

#### Debugging code

You might need to debug your CMake build, or debug your C++ code. Both are covered here.

##### Printing variables

The time honored method of print statements looks like this in CMake:

    message(STATUS "MY_VARIABLE=${MY_VARIABLE}")

However, a built in module makes this even easier:

    include(CMakePrintHelpers)
    cmake_print_variables(MY_VARIABLE)

If you want to print out a property, this is much, much nicer! Instead of getting the properties one by one of of each target (or other item with properties, such as `SOURCES`, `DIRECTORIES`, `TESTS`, or `CACHE_ENTRIES` - global properties seem to be missing for some reason), you can simply list them and get them printed directly:

    cmake_print_properties(
        TARGETS my_target
        PROPERTIES POSITION_INDEPENDENT_CODE
    )

##### Tracing a run

Have you wanted to watch exactly what happens in your CMake file, and when? The `--trace-source="filename"` feature is fantastic. Every line run in the file that you give will be echoed to the screen when it is run, letting you follow exactly what is happening. There are related options as well, but they tend to bury you in output.

For example:

    cmake -S . -B build --trace-source=CMakeLists.txt

If you add `--trace-expand`, the variables will be expanded into their values.

##### Building in debug mode

For single-configuration generators, you can build your code with `-DCMAKE_BUILD_TYPE=Debug` to get debugging flags. In multi-configuration generators, like many IDEs, you can pick the configuration in the IDE. There are distinct flags for this mode (variables ending in `_DEBUG` as opposed to `_RELEASE`), as well as a generator expression value `CONFIG:Debug` or `CONFIG:Release`.

Once you make a debug build, you can run a debugger, such as `gdb` or `lldb` on it.

### Including Small Projects

This is where a good Git system plus CMake shines. You might not be able to solve all the world's problems, but this is pretty close for C++!

There are several methods listed in the chapters in this section.

#### Git Submodule Method

#### GoogleTest: Download method

##### Downloading Method: build time

Until CMake 3.11, the primary download method for packages was done at build time. This causes several issues; most important of which is that add_subdirectory doesn't work on a file that doesn't exist yet! The tool for this, ExternalProject, has to work around this by doing the build itself. (It can, however, build non-CMake packages as well).

##### Downloading Method: configure time

If you prefer configure time, see the [Crascit/DownloadProject](https://github.com/Crascit/DownloadProject) repository for a drop-in solution. Submodules work so well, though, that I've discontinued most of the downloads for things like GoogleTest and moved them to submodules. Auto downloads are harder to mimic if you don't have internet access, and they are often implemented in the build directory, wasting time and space if you have multiple build directories.

#### FetchContent (CMake 3.11+)

Often, you would like to do your download of data or packages as part of the configure instead of the build. This was invented several times in third party modules, but was finally added to CMake itself as part of CMake 3.11 as the FetchContent module.

### Testing

##### General Testing Information

In your main `CMakeLists.txt` you need to add the following function call (not in a subfolder):

    if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
        include(CTest)
    endif()

Which will enable testing and set a `BUILD_TESTING` option so users can turn testing on and off (Along with [a few other things](https://gitlab.kitware.com/cmake/cmake/blob/master/Modules/CTest.cmake)). Or you can do this yourself by directly calling `enable_testing()`.

When you add your test folder, you should do something like this:

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    add_subdirectory(tests)
endif()

The reason for this is that if someone else includes your package, and they use `BUILD_TESTING`, they probably do not want your tests to build. In the rare case that you really do want to enable testing on both packages, you can provide an override:

    if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR MYPROJECT_BUILD_TESTING) AND BUILD_TESTING)
        add_subdirectory(tests)
    endif()

The main use case for the override above is actually in this book's own examples, as the master CMake project really does want to run all the subproject tests.

You can register targets with:

    add_test(NAME TestName COMMAND TargetName)

If you put something else besides a target name after `COMMAND`, it will register as a command line to run. It would also be valid to put the generator expression:

    add_test(NAME TestName COMMAND $<TARGET_FILE:${TESTNAME}>)

which would use the output location (thus, the executable) of the produced target.

##### Building as part of a test

If you want to run CMake to build a project as part of a test, you can do that too (in fact, this is how CMake tests itself). For example, if your master project was called `MyProject` and you had an examples/simple project that could build by itself, this would look like:

    add_test(
      NAME
        ExampleCMakeBuild
      COMMAND
        "${CMAKE_CTEST_COMMAND}"
                 --build-and-test "${My_SOURCE_DIR}/examples/simple"
                                  "${CMAKE_CURRENT_BINARY_DIR}/simple"
                 --build-generator "${CMAKE_GENERATOR}"
                 --test-command "${CMAKE_CTEST_COMMAND}"
    )

##### Testing Frameworks

Look at the subchapters for recipes for popular frameworks.

* [GoogleTest](https://cliutils.gitlab.io/modern-cmake/chapters/testing/googletest.html): A popular option from Google. Development can be a bit slow.
* [Catch2](https://cliutils.gitlab.io/modern-cmake/chapters/testing/catch.html): A modern, PyTest-like framework with clever macros.
* [DocTest](https://github.com/onqtam/doctest): A replacement for Catch2 that is supposed to compile much faster and be cleaner. See Catch2 chapter and replace with DocTest.

#### GoogleTest

##### Submodule method (preferred)

To use this method, just checkout GoogleTest as a submodule:

    git submodule add --branch=release-1.8.0 ../../google/googletest.git extern/googletest

Then, in your main `CMakeLists.txt`:

    option(PACKAGE_TESTS "Build the tests" ON)
    if(PACKAGE_TESTS)
        enable_testing()
        include(GoogleTest)
        add_subdirectory(tests)
    endif()

I would recommend using something like `PROJECT_NAME` `STREQUAL` `CMAKE_PROJECT_NAME` to set the default for the `PACKAGE_TESTS` option, since this should only build by default if this is the current project. As mentioned before, you have to do the `enable_testing` in your main CMakeLists.

Now, in your tests directory:

    add_subdirectory("${PROJECT_SOURCE_DIR}/extern/googletest" "extern/googletest")

If you did this in your main CMakeLists, you could use a normal `add_subdirectory`; the extra path here is needed to correct the build path because we are calling it from a subdirectory.

The next line is optional, but keeps your `CACHE` cleaner:

    mark_as_advanced(
        BUILD_GMOCK BUILD_GTEST BUILD_SHARED_LIBS
        gmock_build_tests gtest_build_samples gtest_build_tests
        gtest_disable_pthreads gtest_force_shared_crt gtest_hide_internal_symbols
    )

If you are interested in keeping IDEs that support folders clean, I would also add these lines:

    set_target_properties(gtest PROPERTIES FOLDER extern)
    set_target_properties(gtest_main PROPERTIES FOLDER extern)
    set_target_properties(gmock PROPERTIES FOLDER extern)
    set_target_properties(gmock_main PROPERTIES FOLDER extern)

Then, to add a test, I'd recommend the following macro:

    macro(package_add_test TESTNAME)
        # create an exectuable in which the tests will be stored
        add_executable(${TESTNAME} ${ARGN})
        # link the Google test infrastructure, mocking library, and a default main fuction to
        # the test executable.  Remove g_test_main if writing your own main function.
        target_link_libraries(${TESTNAME} gtest gmock gtest_main)
        # gtest_discover_tests replaces gtest_add_tests,
        # see https://cmake.org/cmake/help/v3.10/module/GoogleTest.html for more options to pass to it
        gtest_discover_tests(${TESTNAME}
            # set a working directory so your project root so that you can find test data via paths relative to the project root
            WORKING_DIRECTORY ${PROJECT_DIR}
            PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${PROJECT_DIR}"
        )
        set_target_properties(${TESTNAME} PROPERTIES FOLDER tests)
    endmacro()

    package_add_test(test1 test1.cpp)

This will allow you to quickly and simply add tests. Feel free to adjust to suit your needs. If you haven't seen it before, `ARGN` is "every argument after the listed ones". Modify the macro to meet your needs. For example, if you're testing libraries and need to link in different libraries for different tests, you might use this:

    macro(package_add_test_with_libraries TESTNAME FILES LIBRARIES TEST_WORKING_DIRECTORY)
        add_executable(${TESTNAME} ${FILES})
        target_link_libraries(${TESTNAME} gtest gmock gtest_main ${LIBRARIES})
        gtest_discover_tests(${TESTNAME}
            WORKING_DIRECTORY ${TEST_WORKING_DIRECTORY}
            PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${TEST_WORKING_DIRECTORY}"
        )
        set_target_properties(${TESTNAME} PROPERTIES FOLDER tests)
    endmacro()

    package_add_test_with_libraries(test1 test1.cpp lib_to_test "${PROJECT_DIR}/european-test-data/")

##### Download method

##### FetchContent: CMake 3.11

#### Catch

### Exporting and Installing

### Looking for libraries

## Useful materials

[CMake Useful Variables](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/Useful-Variables)  

[How to copy DLL files into the same folder as the executable using CMake?](https://stackoverflow.com/questions/10671916/how-to-copy-dll-files-into-the-same-folder-as-the-executable-using-cmake)  

[CMake RPATH handling](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling)  

[https://cmake.org/examples/](https://cmake.org/examples/)
