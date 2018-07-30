#

Below is a step-by-step tutorial covering common build system use cases that CMake helps to address. Many of these topics have been introduced in Mastering CMake as separate issues but seeing how they all work together in an example project can be very helpful. This tutorial can be found in the [Tests/Tutorial](https://gitlab.kitware.com/cmake/cmake/tree/master/Tests/Tutorial) directory of the CMake source code tree. Each step has its own subdirectory containing a complete copy of the tutorial for that step.

See also the introductory sections of the cmake-buildsystem(7) and cmake-language(7) manual pages for an overview of CMake concepts and source tree organization.

## A Basic Starting Point (Step1)

The most basic project is an executable built from source code files. For simple projects a two line `CMakeLists.txt` file is all that is required. This will be the starting point for our tutorial. The `CMakeLists.txt` file looks like:

    ```bash
    cmake_minimum_required (VERSION 2.6)
    project (Tutorial)
    add_executable(Tutorial tutorial.cxx)
    ```

Note that this example uses lower case commands in the `CMakeLists.txt` file. Upper, lower, and mixed case commands are supported by CMake. The source code for `tutorial.cxx` will compute the square root of a number and the first version of it is very simple, as follows:

```c
// A simple program that computes the square root of a number
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
int main (int argc, char *argv[])
{
  if (argc < 2)
    {
    fprintf(stdout,"Usage: %s number\n",argv[0]);
    return 1;
    }
  double inputValue = atof(argv[1]);
  double outputValue = sqrt(inputValue);
  fprintf(stdout,"The square root of %g is %g\n",
          inputValue, outputValue);
  return 0;
}
```

## Adding a Version Number and Configured Header File

The first feature we will add is to provide our executable and project with a __version number__.

While you can do this exclusively in the source code, doing it in the CMakeLists.txt file provides more _flexibility_.

To add a version number we modify the CMakeLists.txt file as follows:

    ```bash
    cmake_minimum_required (VERSION 2.6)
    project (Tutorial)
    # The version number.
    set (Tutorial_VERSION_MAJOR 1)
    set (Tutorial_VERSION_MINOR 0)

    # configure a header file to pass some of the CMake settings
    # to the source code
    configure_file (
      "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
      "${PROJECT_BINARY_DIR}/TutorialConfig.h"
      )

    # add the binary tree to the search path for include files
    # so that we will find TutorialConfig.h
    include_directories("${PROJECT_BINARY_DIR}")

    # add the executable
    add_executable(Tutorial tutorial.cxx)
    ```

Since the configured file will be written into the binary tree we must add that directory to the list of paths to search for include files. We then create a `TutorialConfig.h.in` file in the source tree with the following contents:

    ```bash
    // the configured options and settings for Tutorial
    #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
    #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
    ```

When CMake configures this header file the values for @Tutorial_VERSION_MAJOR@ and @Tutorial_VERSION_MINOR@ will be replaced by the values from the CMakeLists.txt file. Next we modify tutorial.cxx to include the configured header file and to make use of the version numbers. The resulting source code is listed below.

    ```c
    // A simple program that computes the square root of a  number
    #include <stdio.h>
    #include <stdlib.h>
    #include <math.h>
    #include "TutorialConfig.h"

    int main (int argc, char *argv[])
    {
        if (argc < 2) {
            fprintf(stdout,"%s Version %d.%d\n",
            argv[0],
            Tutorial_VERSION_MAJOR,
            Tutorial_VERSION_MINOR);
            fprintf(stdout,"Usage: %s number\n",argv[0]);

            return 1;
        }
        
        double inputValue = atof(argv[1]);
        double outputValue = sqrt(inputValue);
        fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);

        return 0;
    }
    ```

The main changes are the inclusion of the TutorialConfig.h header file and printing out a version number as part of the usage message.

