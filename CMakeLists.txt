cmake_minimum_required(VERSION 3.0)
project(uWebSockets CXX)

# Build shared libraries by default. This can be overridden at configuration
# time on the command line by adding the flag "-DBUILD_SHARED_LIBS=OFF".
if( NOT BUILD_SHARED_LIBS )
  set(
    BUILD_SHARED_LIBS ON
    CACHE BOOL "Determines whether or not libraries will be built shared or not (default: ON)"
    FORCE
  )
endif()

# Look for where the OpenSSL is installed. Custom locations can be specified at
# configuration time on the command line by adding the installation prefix to
# "-DCMAKE_PREFIX_PATH".
find_path(openssl_include_path "openssl/opensslv.h")
find_library(openssl_library ssl)

# Look for where the zlib is installed. Custom locations can be specified at
# configuration time on the command line by adding the installation prefix to
# "-DCMAKE_PREFIX_PATH".
find_path(zlib_include_path "zlib.h")
find_library(zlib_library z)

# Gather all dependencies.
list(APPEND include_dependencies ${openssl_include_path})
list(APPEND include_dependencies ${zlib_include_path})
list(APPEND library_dependencies ${openssl_library})
list(APPEND library_dependencies ${zlib_library})

# Identify all public headers and then copy them into the intended include
# namespace (i.e. include/uWS). Also add a header installation target. This
# also makes it easier to compile the examples.
file(GLOB headers "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")
file(COPY ${headers} DESTINATION "${CMAKE_BINARY_DIR}/include/uWS")
install(DIRECTORY "${CMAKE_BINARY_DIR}/include/uWS" DESTINATION include)

# Identify all of the library sources and add the uWS library target. Also add
# a library installation target.
file(GLOB libuWS_src "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(libuWS ${libuWS_src})
set_target_properties(
  libuWS
  PROPERTIES
    CXX_STANDARD 11 # cmake >= 3.1
    CXX_STANDARD_REQUIRED ON # cmake >= 3.1
    COMPILE_FLAGS "-std=c++11" # in case cmake < 3.1
    INCLUDE_DIRECTORIES "${include_dependencies}"
    LINK_LIBRARIES "${library_dependencies}"
    OUTPUT_NAME uWS
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    SOVERSION 0
    VERSION 0.13
)
install(TARGETS libuWS DESTINATION lib)

# Optionally build the example executables.
option(BUILD_EXAMPLES "build example executables" ON)
if(BUILD_EXAMPLES)

  add_custom_target(examples) # So you can run "make examples"
  macro(add_example EXAMPLE_NAME)
    add_executable(${EXAMPLE_NAME}_example ${ARGN})
    set_target_properties(
      ${EXAMPLE_NAME}_example
      PROPERTIES
        CXX_STANDARD 11 # cmake >= 3.1
        CXX_STANDARD_REQUIRED ON # cmake >= 3.1
        COMPILE_FLAGS "-std=c++11" # in case cmake < 3.1
        INCLUDE_DIRECTORIES "${CMAKE_BINARY_DIR}/include"
        LINK_LIBRARIES "libuWS;pthread;${library_dependencies}"
        OUTPUT_NAME "${EXAMPLE_NAME}"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/examples"
    )
    add_dependencies(examples "${EXAMPLE_NAME}_example")
  endmacro()

  add_example(
    echo
    "${CMAKE_CURRENT_SOURCE_DIR}/examples/echo.cpp"
  )
  add_example(
    multithreaded_echo
    "${CMAKE_CURRENT_SOURCE_DIR}/examples/multithreaded_echo.cpp"
  )

endif()

# Add test
enable_testing()
add_executable(main_test "${CMAKE_CURRENT_SOURCE_DIR}/tests/main.cpp")
set_target_properties(
  main_test
  PROPERTIES
    CXX_STANDARD 11 # cmake >= 3.1
    CXX_STANDARD_REQUIRED ON # cmake >= 3.1
    COMPILE_FLAGS "-std=c++11" # in case cmake < 3.1
    INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/src"
    LINK_LIBRARIES "libuWS;pthread;${library_dependencies}"
    OUTPUT_NAME main_test
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/tests"
)
add_test(NAME main COMMAND "${CMAKE_BINARY_DIR}/tests/main_test")
