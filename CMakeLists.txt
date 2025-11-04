cmake_minimum_required(VERSION 3.14)
project(process)

message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  (This can be changed with `cmake -DCMAKE_INSTALL_PREFIX=/some/place`")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(CheckIncludeFileCXX) # CHECK_INCLUDE_FILE_CXX will be used in later scripts

# If I *require* std::format, I can start using it across this library. That
# limits us to gcc-13 on Ubuntu (though it's the libc++ that contains
# <format>). For now, test and use it only in examples.
check_include_file_cxx(format HAVE_STD_FORMAT)
if(NOT HAVE_STD_FORMAT)
  message(INFO " Your compiler does not appear to provide std::format, some examples won't build.")
  message(INFO " In future, <format> may be required.")
endif()

# Add the host definition to CXXFLAGS along with other switches
if (APPLE)
  set(CMAKE_CXX_FLAGS "-Wall -g -O3")
else()
  if(CMAKE_CXX_COMPILER_ID MATCHES MSVC OR CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES MSVC)
    # Set flags for Windows.
    # -DNOMINMAX to prefer std::min and std::max
    # /EHsc required for use of exceptions
    # /Zc:__cplusplus ensures __cplusplus define does not lie
    # /constexpr:steps is equivalent to -fconstexpr-steps or -fconstexpr-ops-limit
    if(CMAKE_CXX_COMPILER_ID MATCHES MSVC)
      set(CMAKE_CXX_FLAGS "-DNOMINMAX /EHsc /Zc:__cplusplus /constexpr:steps500000000")
    else()
      set(CMAKE_CXX_FLAGS "-DNOMINMAX /EHsc /Zc:__cplusplus")
    endif()
  elseif(CMAKE_CXX_COMPILER_ID MATCHES Intel)
    # To use Intel compiler, you can call cmake as: `cmake -DCMAKE_CXX_COMPILER=icpc ..` or `CXX=icpc cmake ..`
    message(WARNING "Intel compiler has not been tested for some time")
    set(CMAKE_CXX_FLAGS "-Wall -g -std=c++20 -xHOST -O3")
  else() # GCC or Clang
    if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
      # Add compiler version check, to ensure gcc is version 11 or later.
      if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 11)
        message(FATAL_ERROR "GCC version must be at least 11")
      else()
        message(STATUS "GCC version ${CMAKE_CXX_COMPILER_VERSION} OK!")
      endif()
    endif()
    set(COMPREHENSIVE_WARNING_FLAGS "-Wall -Wextra -Wpedantic -pedantic-errors -Werror -Wfatal-errors -Wno-psabi")
    set(CMAKE_CXX_FLAGS "-g ${COMPREHENSIVE_WARNING_FLAGS} -O3")
    if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
      # Make it possible to compile complex constexpr functions (gcc only)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fconstexpr-ops-limit=5000000000")
    else()
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fconstexpr-steps=500000000")
    endif()
  endif()
endif()

# If you tell the code you have std::format, then it may use it
if(HAVE_STD_FORMAT)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DMATHS_HAVE_STD_FORMAT")
endif()

# In Visual Studio, if CMAKE_BUILD_TYPE is "Debug", some programs will run very slowly
if(WIN32)
  set(CMAKE_BUILD_TYPE "Release")
endif(WIN32)

# Some programs may require the ability to define literal template args of
# type std::array (a C++-20 feature). Clang versions less than 18 can't.
set(ARRAY_LITERAL_TEMPL_ARGS_ARE_OK 0)
if(CMAKE_CXX_COMPILER_ID MATCHES Clang)
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 18)
    set(ARRAY_LITERAL_TEMPL_ARGS_ARE_OK 1)
  endif()
else()
  set(ARRAY_LITERAL_TEMPL_ARGS_ARE_OK 1)
endif()

# All the headers are here
add_subdirectory(sm)

# Unit testing using the ctest framework
option(BUILD_TESTS "Build tests" ON)
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(tests)
endif(BUILD_TESTS)

# For debugging of variables:
option(DEBUG_VARIABLES OFF)
if(DEBUG_VARIABLES)
  get_cmake_property(_variableNames VARIABLES)
  foreach(_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
  endforeach()
endif(DEBUG_VARIABLES)
