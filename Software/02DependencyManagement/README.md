# C/C++ & CMake Project Dependency Management

## Motivation

[CMake](https://cmake.org/) and build environments in general
used to be much simpler and required you to do much more manually
when I started programming with C++.
For example,
you would

* set up a folder with all your repositories manually,
* download each required dependency to that one folder,
* build each dependency separately
* and potentially install the headers and libraries for your system/account.

If everything went well,
you would then go back to the project you actually wanted to work on
and make sure your [CMake](https://cmake.org/) project setup
points your build chain to the right include directories
and lists all the libraries you needed from the dependencies.
Every time you started working on a new machine,
you would go through the same steps despite that
it required specifying the same things again and again.

This can already be cumbersome if you work on a project on your own,
but will be disastrous if you work on a large team project.
I assume that is why some big tech companies came up with mono repos
that entomb all dependencies in a single giant repository.
That approach has its own pros and cons.
One disadvantage is that all projects depend on this one giant repository by design
which can put an unnecessary burden on you
if you just want to use a small set of the dependencies.
It can also be disadvantageous
if you want to use a different version of what is currently the recommended one
in the mono repo.

There is a different approach
which I personally prefer and that I fortunately stumbled upon by chance.
Instead of a mono repo or manually specifying your dependencies and getting them by calling `git clone` yourself,
you can make use of [CMake](https://cmake.org/)'s `FetchContent` module
which I found quite handy.

## The Modern Way Using CMake and FetchContent

Note that this approach requires that
[CMake](https://cmake.org/) is able to call a
[git client](https://git-scm.com/tools).

Using [CMake](https://cmake.org/)'s module
for "fetching content" is quite straightforward:

* First, you tell [CMake](https://cmake.org/) that
  you want to make use of its dependency fetching feature.
  That is why you want to include the `FetchContent` module itself:
    ```cmake
    include(FetchContent)
    ```

* For each dependency,
  you specify where [CMake](https://cmake.org/) should get the dependency from
  using `FetchContentDeclare` and which version you prefer.

    * To for example fetch [fastgltf](https://github.com/spnda/fastgltf)
    from [github](http://github.com/)
    and request the version tagged "v0.9.0",\
    you would add something like this to your [CMake](https://cmake.org/) code:

      ```cmake
      # get the version tagged "v0.9.0"
      FetchContent_Declare(
          fastgltf
          GIT_REPOSITORY https://github.com/spnda/fastgltf.git
          GIT_TAG        v0.9.0
      )
      FetchContent_MakeAvailable(fastgltf)
      ```

    *  Instead of a version tag you can also specify a commit hash:

       ```cmake
       # get a specific commit of fast gltf from April 2026
       FetchContent_Declare(
          fastgltf
          GIT_REPOSITORY https://github.com/spnda/fastgltf.git
          GIT_TAG        a31be255ff041fa1d6697e5040fef3c9b6405795
       )
       FetchContent_MakeAvailable(fastgltf)
       ```

    * What I find practical is that you can directly specify
      what version of the code you want to use.
      The downloaded dependency copy is then part
      of your current [CMake](https://cmake.org/) project
      and your project only.
      It will not be part of a giant mono repo or
      a manually assembled and shared repositories folder.
      Besides working automatically,
      the advantage is that your code directly defines the required version
      (and potentially also [CMake](https://cmake.org/) build variables)
      with respect to your current project.
      This means that there will not be any dependency clash
      due to multiple projects using the same dependency,
      but different versions of it.
      However, the drawback is that
      you might have the same or similar versions of a dependency
      multiple times on your local machine
      (which I find okay as a tradeoff).

* The above code downloads the dependencies automatically
  if you run [CMake](https://cmake.org/)'s configure step.
  All that is left to do is telling [CMake](https://cmake.org/)
  what target you want to use the dependency for.
  * You specify the target and
    how the target makes use of the dependency using a simple line:

    ```cmake
    target_link_libraries(${aceLibName} PRIVATE fastgltf::fastgltf)
    ```

    The above line tells [CMake](https://cmake.org/)
    that [fastgltf](https://github.com/spnda/fastgltf) is internally used
    to build the [Ace](https://AceNerds.com) library.

  * You can replace `PRIVATE` with `PUBLIC` if you want the dependency
    to be visible externally and beyond the specified target if
    for example users of [Ace](https://AceNerds.com) should also be able
    to make use of [fastgltf](https://github.com/spnda/fastgltf) directly.

  * One aspect I find quite neat is that
    you do not need to directly specify any include directory
    for [fastgltf](https://github.com/spnda/fastgltf) in your project.
    The [CMake](https://cmake.org/) project files of
    [fastgltf](https://github.com/spnda/fastgltf) itself already define
    what include files are necessary for targets
    that make use of [fastgltf](https://github.com/spnda/fastgltf).
    [CMake](https://cmake.org/) then simply makes the headers available according to
    your target and potentially its ancestors targets
    using the `PRIVATE`/`PUBLIC`/`INTERFACE` scheme.

  * To make this header forwarding scheme work for your own project,
    you can define the location of your public project header files
    using the [CMake](https://cmake.org/) command `target_include_directories`
    for your own project similarly to this:
    ```cmake
    target_include_directories(${aceLibName} PUBLIC ${PROJECT_SOURCE_DIR})
    ```


* [CMake](https://cmake.org/) downloads the dependencies
  to your build folder by default.
  That means they get deleted if you clear your build files
  by deleting the whole build folder (which I like to do sometimes).
  If you want to avoid that,
  you can also specify a different download directory like so:

  ```cmake
  set(FETCHCONTENT_BASE_DIR "${CMAKE_SOURCE_DIR}/Deps" CACHE PATH
      "Folder into which 3rd-party code is fetched.")
  ```

  This code for example says that I want [CMake](https://cmake.org/)
  to download the source code of the dependencies
  into a `deps` folder inside my project's root directory.
  It might be a good idea
  to add this folder with the dependencies to your `.gitignore` file.
  Besides, the user can also change the target directory if that is desired
  (thanks to `CACHE`).

That is all you need.
These few lines of code will tell [CMake](https://cmake.org/)
to get the dependency automatically during its configure step
(if the code is not already on your local machine in the target directory).
There is no need for manual downloading.
There is also no need for further manual setting of
include directories or required libraries.
Every time you clone your repository into a new environment,
[CMake](https://cmake.org/) will take care of this!

## Full fastgltf Example

Here is one full example that shows how I integrate `fastgltf` into my project:

In my main `CMakeLists.txt`:
```cmake
# make fastgltf optional and integrate it by default
option(ACE_PORTER_FAST_GLTF "Enables GLTF file loading using fastGLTF." on)

# fetch dependencies into this folder
set(FETCHCONTENT_BASE_DIR "${CMAKE_SOURCE_DIR}/Dependencies" CACHE PATH
    "Directory into which 3rd party dependencies are fetched and built.")
include(FetchContent)

# declare the main Ace library
# this must happen before any module links dependencies against it
# ace_add_library is a wrapper of CMake's add_library()
ace_add_library(${aceLibName})

include(${PROJECT_SOURCE_DIR}/CMake/FastGLTF.cmake)
```

The content of my `fastgltf` [CMake](https://cmake.org/) module is this:
```cmake
# cmake functionality for the dependency fastgltf

if (ACE_PORTER_FAST_GLTF)

    # use fast gltf from April 2026
    FetchContent_Declare(
        fastgltf
        GIT_REPOSITORY https://github.com/spnda/fastgltf.git
        GIT_TAG        a31be255ff041fa1d6697e5040fef3c9b6405795
    )
    FetchContent_MakeAvailable(fastgltf)

    # make fastgltf part of Ace
    target_link_libraries(${aceLibName} PRIVATE fastgltf::fastgltf)
    target_compile_definitions(${aceLibName} PUBLIC ACE_PORTER_FAST_GLTF)

endif (ACE_PORTER_FAST_GLTF)

```

Hopefully,
this approach will also simplify the management of the dependencies\
you need for your own projects!