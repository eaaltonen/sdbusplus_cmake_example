# Example CMake build with sdbusplus code generator

Requires [sdbusplus](https://github.com/openbmc/sdbusplus) library,
code generator and it's dependencies to run. Please see
https://github.com/openbmc/sdbusplus/#installing-sdbusplus-on-custom-distributions

for the projects dependencies.

Build this example with the commands

    mkdir build; cd build
    cmake .. -G "Ninja" [-DCMAKE_PREFIX_PATH=<custom/sdbusplus/install/path>]
    cmake --build .
