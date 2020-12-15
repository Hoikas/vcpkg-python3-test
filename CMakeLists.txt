cmake_minimum_required(VERSION 3.12)

if(DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
endif()
if(DEFINED ENV{VCPKG_DEFAULT_TRIPLET} AND NOT DEFINED VCPKG_TARGET_TRIPLET)
    set(VCPKG_TARGET_TRIPLET "$ENV{VCPKG_DEFAULT_TRIPLET}" CACHE STRING "")
endif()

project(vcpkg_python3_test)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#  ===========================================================================

enable_testing()

option(OLD_FINDERS "Build the FindPythonLibs and FindPythonInterp Test" ON)
option(NEW_FINDERS "Build the FindPython3 Test" ON)

include(CMakeDependentOption)
cmake_dependent_option(OLD_FINDERS_WRONG_ORDER "Execute the find_package() calls in the wrong order" OFF OLD_FINDERS OFF)

#  ===========================================================================

if(OLD_FINDERS)
    if(OLD_FINDERS_WRONG_ORDER)
        find_package(PythonLibs REQUIRED)
        find_package(PythonInterp REQUIRED)
    else()
        find_package(PythonInterp REQUIRED)
        find_package(PythonLibs REQUIRED)
    endif()

    message(STATUS "--- Old finders---")
    message(STATUS "Python v${PYTHON_VERSION_STRING}")
    message(STATUS "EXE: ${PYTHON_EXECUTABLE}")
    message(STATUS "Includes: ${PYTHON_INCLUDE_DIRS}")
    message(STATUS "Libraries: ${PYTHON_LIBRARIES}")

    execute_process(COMMAND ${PYTHON_EXECUTABLE} -c "import distutils.sysconfig; import sys; sys.stdout.write(distutils.sysconfig.get_python_lib(plat_specific=False,standard_lib=True))"
        OUTPUT_VARIABLE PYTHON_STDLIB
    )
    message(STATUS "Batteries: ${PYTHON_STDLIB}")
    file(TO_CMAKE_PATH ${PYTHON_STDLIB} DONT_CHOKE_ON_IT)

    add_executable(test_oldfinders main.cpp)
    target_link_libraries(test_oldfinders PRIVATE ${PYTHON_LIBRARIES})
    target_include_directories(test_oldfinders PRIVATE ${PYTHON_INCLUDE_DIRS})
    target_compile_definitions(test_oldfinders PRIVATE "BATTERIES=L\"${DONT_CHOKE_ON_IT}\"")

    add_test(NAME oldfinders COMMAND test_oldfinders)
endif()

#  ===========================================================================

if(NEW_FINDERS)
    find_package(Python3 REQUIRED COMPONENTS Interpreter Development)

    message(STATUS "--- New finders---")
    message(STATUS "Python v${Python3_VERSION}")
    message(STATUS "EXE: ${Python3_EXECUTABLE}")
    message(STATUS "Includes: ${Python3_INCLUDE_DIRS}")
    message(STATUS "Libraries: ${Python3_LIBRARIES}")
    message(STATUS "Batteries: ${Python3_STDLIB}")

    # Debug link libs
    get_target_property(DEBUG_IFACE Python3::Python INTERFACE_LINK_LIBRARIES)
    get_target_property(DEBUG_LIBS Python3::Python LINK_LIBRARIES)
    message(STATUS "Interface Link: ${DEBUG_IFACE}")
    message(STATUS "Link Libs: ${DEBUG_LIBS}")

    file(TO_CMAKE_PATH ${Python3_STDLIB} DONT_CHOKE_ON_IT)

    add_executable(test_newfinders main.cpp)
    target_link_libraries(test_newfinders PRIVATE Python3::Python)
    target_compile_definitions(test_newfinders PRIVATE "BATTERIES=L\"${DONT_CHOKE_ON_IT}\"")

    add_test(NAME newfinders COMMAND test_newfinders)
endif()
