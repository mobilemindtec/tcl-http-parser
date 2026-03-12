cmake_minimum_required(VERSION 3.16)

project(HttpParser
  VERSION 0.1
  DESCRIPTION "Tcl Pico Http Parser extension"
  LANGUAGES C)

include(CTest)

add_library(HttpParser SHARED
    src/picohttpparser.h
    src/picohttpparser.c
    src/lib.c
    src/pathparser.h
    src/pathparser.c
)

set_target_properties(HttpParser PROPERTIES
    OUTPUT_NAME "httpparser"
)

if (${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
    add_subdirectory(HttpParserTests)
endif ()

enable_testing()

target_compile_options(HttpParser PRIVATE
    -Wall
    -Wno-incompatible-pointer-types
)

#
# Find Tcl (works with Tcl 8.6, 8.7 and 9)
#
find_package(TCL REQUIRED COMPONENTS Tcl)

#
# Enable stubs (required for extensions)
#
target_compile_definitions(HttpParser PRIVATE USE_TCL_STUBS)

#
# Include directories
#
target_include_directories(HttpParser PRIVATE
    ${TCL_INCLUDE_PATH}
)

#
# Find stub library dynamically
#
find_library(TCL_STUB_LIBRARY
    NAMES
        tclstub
        tclstub9.0
        tclstub8.7
        tclstub8.6
    PATHS
        ${TCL_LIBRARY_PATH}
        /usr/lib
        /usr/lib64
        /usr/local/lib
)

if(NOT TCL_STUB_LIBRARY)
    message(FATAL_ERROR "Tcl stub library not found")
endif()

target_link_libraries(HttpParser PRIVATE ${TCL_STUB_LIBRARY})

#
# Copy TCL package index
#
add_custom_target(copy_pkg_index
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/httpparser
        ${CMAKE_CURRENT_BINARY_DIR}/httpparser
    SOURCES httpparser/pkgIndex.tcl
)

#
# Copy test folder
#
add_custom_target(copy_tcl_tests
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/tests
        ${CMAKE_CURRENT_BINARY_DIR}/tests
    SOURCES
        tests/all.tcl
        tests/parser.test
        tests/server.test
)

#
# Copy examples
#
add_custom_target(copy_examples
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/examples
        ${CMAKE_CURRENT_BINARY_DIR}/examples
    SOURCES
        examples/http_server.tcl
)

add_dependencies(HttpParser
    copy_pkg_index
    copy_tcl_tests
    copy_examples
)

#
# Copy built library into Tcl package directory
#
add_custom_command(TARGET HttpParser POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
        $<TARGET_FILE:HttpParser>
        ${CMAKE_CURRENT_BINARY_DIR}/httpparser
)


execute_process(
    COMMAND sh -c "echo 'puts \$tcl_library' | ${TCL_TCLSH}"
    OUTPUT_VARIABLE TCL_PACKAGE_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

install(TARGETS HttpParser
    LIBRARY DESTINATION ${TCL_PACKAGE_DIR}/httpparser
)

install(FILES
    httpparser/pkgIndex.tcl
    DESTINATION ${TCL_PACKAGE_DIR}/httpparser
)