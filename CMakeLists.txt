cmake_minimum_required(VERSION 3.16.3)

project(sdbusplus-cmake-example VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 17)

include(FindPkgConfig)
if(NOT PKG_CONFIG_FOUND)
    message( FATAL_ERROR "pkg-config not found! The build needs pkg-config to locate libraries." )
endif()
pkg_check_modules(sdbusplus REQUIRED IMPORTED_TARGET sdbusplus)

set(DBUS_INTERFACE_NAME "net.poettering.Calculator")
set(DBUS_ROOT_OBJECT_PATH "/net/poettering/Calculator")

# A directory for generated C++ sources
# I would also like to have this show up source code directory in CLion
# but _NOT_ for version control
set(GENERATED_SOURCES_DIR "${CMAKE_BINARY_DIR}/generated-src")
file(MAKE_DIRECTORY "${GENERATED_SOURCES_DIR}")

set(SDBUSPLUS_GENERATED_SOURCES_DIR "${GENERATED_SOURCES_DIR}/net/poettering/Calculator")
file(MAKE_DIRECTORY "${SDBUSPLUS_GENERATED_SOURCES_DIR}")
set(DBUS_DECLARATIONS
    "${CMAKE_CURRENT_SOURCE_DIR}/net/poettering//Calculator.errors.yaml"
    "${CMAKE_CURRENT_SOURCE_DIR}/net/poettering//Calculator.interface.yaml"
)
list(APPEND GENERATED_SOURCE_FILES
    ${SDBUSPLUS_GENERATED_SOURCES_DIR}/error.cpp
    ${SDBUSPLUS_GENERATED_SOURCES_DIR}/error.hpp
    ${SDBUSPLUS_GENERATED_SOURCES_DIR}/server.cpp
    ${SDBUSPLUS_GENERATED_SOURCES_DIR}/server.hpp
)

add_custom_command(OUTPUT ${GENERATED_SOURCE_FILES}
    COMMAND sdbus++ interface server-header ${DBUS_INTERFACE_NAME} > ${SDBUSPLUS_GENERATED_SOURCES_DIR}/server.hpp
    COMMAND sdbus++ interface server-cpp ${DBUS_INTERFACE_NAME} > ${SDBUSPLUS_GENERATED_SOURCES_DIR}/server.cpp
    COMMAND sdbus++ error exception-header ${DBUS_INTERFACE_NAME} > ${SDBUSPLUS_GENERATED_SOURCES_DIR}/error.hpp
    COMMAND sdbus++ error exception-cpp ${DBUS_INTERFACE_NAME} > ${SDBUSPLUS_GENERATED_SOURCES_DIR}/error.cpp
    DEPENDS ${DBUS_DECLARATIONS}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

add_library(CalculatorServer STATIC
    "${SDBUSPLUS_GENERATED_SOURCES_DIR}/server.cpp"
    "${SDBUSPLUS_GENERATED_SOURCES_DIR}/error.cpp"
)
target_include_directories(CalculatorServer PUBLIC "${GENERATED_SOURCES_DIR}")
target_link_libraries(CalculatorServer PUBLIC PkgConfig::sdbusplus)

add_executable(calculator-server calculator-server.cpp)
target_link_libraries(calculator-server CalculatorServer)

