**Note.** When you configure a project using CMake, it does not compile the code itself, instead, it generates files for other build system (Makefiles, Ninja files, or VS project files).
# Main Structure
- A variety of concepts such as `targets`, `generators`, `commands` are implemented as C++ classes and are referenced in CMake commands.
- lowest level of CMake classes are source files.
    - typical C / C++ source code
    - **Source files** are combined into **target**
    - **Target** is typically an executable  / library.
    - **Directory** typically has a CMakeLists.txt file and with one or more targets associated with it.
    - Every **Directories** has a **local generator** responsible for generating Makefiles for that directory.
    - **Local Generators** share a common **global generator** that controls the entire build process.

- details:
    - execution begins by creating an instance of `cmake` class and pass the cmd line args to it. The class holds the information that is global to the build process (eg. cache values).
    - Create correct global generator.
    - Pass control to the global generator.

## Generators
- THe global generator is responsible for managing the configuration and generation of all of the Makefiles for a project.
- Generally, one local generator is created for each directory of the project that the global generator processed.
- Each local generator has an instance of a class `cmMakefile`. This is where the results of parsing the CMakeLists files stored in the directories are recorded.
- For each directory in a project, there will be a `single cmMakefile` instance. This instance will hold all of the information local generator parsed from the directory's CMakeLists.txt.
- > cmMakefile reading in a CMakeLists.txt is like CMake executing commands it finds in the order it encounteres them.
- Each command in CMake is implemented as a C++ class. They have two main parts
    1. InitialPass
        - The initialPass method receives args and `cmMakefile` instance and performs the command operations.
        ```CMake
        set(TARGET Hello.c World.c)
        ```
        - if all arguments ar correct (Hello.c, World.c), cmake calls a method in the `cmMakefile` instance.
    2. FinalPass
        - The finalPass of a command is executed after all comands for the entire CMake project have had their initialPass invoked.
- Once all the CMakeLists.txt have been processed, the generators use the information collected by `cmMakefile` instances to produce the appropriate files for the target build systems.

## Targets
- A key item stored in the `cmMakefile` instance.
- Targets represent **executables**, **libraries**, **utilities**, built by CMake.
- Every `add_library`, `add_executable`, `add_custom_target` command creates a target.
- for instance
```CMake
add_library(foo STATIC foo1.c foo2.c)
```
- The command create a target named `**foo**`, which is a static library with foo1.c, foo2.c as source files.
- The name foo after this command can be used as a library name elsewhere in the project.
- Libraries can be declared to be **STATIC, SHARED, MODULE**.
- defualt CMake library declaration is STATIC.
- In addition to storing target types, cmake also keep track of target properties.
- Properties can be set and get using commands like `s(g)et_target_propert(y)ies, s(g)et_property`.
- Targets store a list of libraries they link against. This is set using command: `target_link_libraries`.
- For each library cmake creates, it keeps trac of all the libraries where this library depends.
- for instance
```CMake
add_library(foo foo.cxx)
target_link_libraries(foo bar)

add_executable(go go.cxx)
target_link_libraries(go foo)
```
- This links the libraries `foo` and `bar` to executable `go`. (In spite of the fact that only foo was explicitly linked into go)
- For static builds, this extra linkage is needed. Since the foo library uses symbols in bar library, go will most likely also beed bar since it uses foo.

## Source files
- cmake stores the filename, extensions, and a number of general properties.
- **COMMON PROPERTIES**
    1. COMPILE_FLAGS
        - Compile flags specific to the source file. (-D, -I flags)
    2. GERNERATED
        - This property indicates that the source file is generated as part of the build process.
        - > the fource file may not exist when cmake runs.
    3. OBJECT_DEPENDS
        - cmake automatically performs dependecy analysis to determine the usual C, CXX and Fortan dependencies.
        - This parameter is used in rare case.
    4. WRAP_EXCLUDE (ABSTRACT)

## Directories, Tests and Properties
- Some useful properties for a directory
    1. LISTFILE_STACK
        - THis property is useful when trying to debug errors in CMakeLists.txt
        - It retuens a list of what list files are currently being processed.
    2. Pending for descoveries

## Variables
- When you set a variable that is visiable to the current CMakeListst.txt, any subdirectories' CMakeLists.txt and functions or macros that are invoked can access the variable.
- Any new variables created in the child scope, or changes made to exisiting variables will not impact the parent scope.\
- for instance
```CMake
function (foo)
    message(${TEST})
    set(test 2 PARENT_SCOPE)
    message(${TEST}) # test will be **1**
endfunction()

# start here
set(TEST 1)
foo()
message(${TEST}) # test will be 2
```
> In some cases, I might want to set a variable from the CMake UI. -> variables must be a cache entry.
> WHenever CMaje is run, it produces a cache file in the binary directory (build/). The purpose of this cache is to store the user's selection and choices, so that if they run CMake again, they won't need to reenter the selection.
- for instance
```CMake
option(USE_JEPG "Do you want to you the jpeg library? ")
```
- The above command would create a variable called `USE_JPEG` and put it into the cache.
- There are ore commands you can use to create variable in CMake cache.
    - option
    - find_file
    - set
    - ```set(USE_JPEG ON CACHE BOOL "include jpeg support?")```
    - Variable types include BOOL, PATH, FILEPATH and STRING
