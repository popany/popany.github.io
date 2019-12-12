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

## Useful materials

[CMake Useful Variables](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/Useful-Variables)  

[How to copy DLL files into the same folder as the executable using CMake?](https://stackoverflow.com/questions/10671916/how-to-copy-dll-files-into-the-same-folder-as-the-executable-using-cmake)  

[CMake RPATH handling](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling)  
