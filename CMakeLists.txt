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

if(DEFINED OLD_FINDERS)
    if(NOT DEFINED OLD_FINDERS_WRONG_ORDER)
        find_package(PythonInterp REQUIRED)
    endif()
    find_package(PythonLibs REQUIRED)
    if(DEFINED OLD_FINDERS_WRONG_ORDER)
        find_package(PythonInterp REQUIRED)
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
endif()

#  ===========================================================================

if(DEFINED NEW_FINDERS)
    find_package(Python3 REQUIRED COMPONENTS Interpreter Development)

    message(STATUS "--- New finders---")
    message(STATUS "Python v${Python3_VERSION}")
    message(STATUS "EXE: ${Python3_EXECUTABLE}")
    message(STATUS "Includes: ${Python3_INCLUDE_DIRS}")
    message(STATUS "Libraries: ${Python3_LIBRARIES}")
    message(STATUS "Batteries: ${Python3_STDLIB}")

    file(TO_CMAKE_PATH ${Python3_STDLIB} DONT_CHOKE_ON_IT)

    add_executable(test_newfinders main.cpp)
    target_link_libraries(test_newfinders PRIVATE Python3::Python)
    target_compile_definitions(test_newfinders PRIVATE "BATTERIES=L\"${DONT_CHOKE_ON_IT}\"")
endif()
