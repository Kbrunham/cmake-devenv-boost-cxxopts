cmake_minimum_required(VERSION 3.8)

project(cmake_example
        VERSION 1.0
        DESCRIPTION "cmake example"
        LANGUAGES CXX
        )

# Set the minimum required C++ standard.
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


# Only process these if this is the toplevel
if (CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    # Environment Setup
    if (NOT NO_ENVIRONMENT_SETUP)
        # Boost
        if (WIN32)
            set(BOOST_INCLUDEDIR ${CMAKE_CURRENT_SOURCE_DIR}/../extlibs/boost/include/boost-1_75)
        else ()
            set(BOOST_INCLUDEDIR ${CMAKE_CURRENT_SOURCE_DIR}/../extlibs/boost/include)
        endif ()
        set(Boost_NO_BOOST_CMAKE ON)
        set(Boost_NO_SYSTEM_PATHS ON)
        set(BOOST_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../extlibs/boost)
        # OpenSSL
        set(OPENSSL_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../extlibs/openssl)
    endif ()


    # Include build common macros
    message("INFO: Including CMake bcommon")
    include("${CMAKE_CURRENT_SOURCE_DIR}/extern/cmake-helpers/bcommon.cmake")

    # Set folder support for use with IDEs
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)

    # Enable testing
    include(CTest)

    # Explicitly add option for building shared libs
    option(BUILD_SHARED_LIBS "Builds the project's libraries as shared libs. This does not affect imported libraries." OFF)

    # Include 3rd party libraries from environment

    # Add openssl
    find_package(OpenSSL REQUIRED)
    message(STATUS "OpenSSL version: ${OPENSSL_VERSION}")
    message(STATUS "OpenSSL include dir: ${OPENSSL_INCLUDE_DIR}")
    message(STATUS "OpenSSL ssl library: ${OPENSSL_SSL_LIBRARY}")
    message(STATUS "OpenSSL crypto library: ${OPENSSL_CRYPTO_LIBRARY}")

    # Add boost
    find_package(Boost 1.50 REQUIRED)
    message(STATUS "Boost version: ${Boost_VERSION}")
    message(STATUS "Boost include dir: ${Boost_INCLUDE_DIRS}")
    message(STATUS "Boost library dir: ${Boost_LIBRARY_DIRS}")
    message(STATUS "Boost libraries: ${Boost_LIBRARIES}")

    # Include 3rd party libraries from source tree

    # Add Google Test
    include(GoogleTest)
    # Disable installing googletest
    set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
    # Prevent Google Test from modifying compiler/linker options (https://crascit.com/2015/07/25/cmake-gtest/)
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    # Force building GoogleTest as static, but then leave BUILD_SHARED_LIBS to control
    # defaults for the rest of the project`
    set(__TEMP_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    add_subdirectory("${PROJECT_SOURCE_DIR}/extern/googletest" "extern/googletest")
    set(BUILD_SHARED_LIBS ${__TEMP_BUILD_SHARED_LIBS} CACHE BOOL "" FORCE)


    # Add cxxopts
    add_subdirectory("${PROJECT_SOURCE_DIR}/extern/cxxopts" "extern/cxxopts" EXCLUDE_FROM_ALL)

    # Add source to this project's executable.
    add_executable(main "main.cpp" "main.h")
    target_link_libraries(main Boost::headers)
    target_link_libraries(main OpenSSL::SSL)
    target_link_libraries(main cxxopts)

    project_add_test(
            # Exe name
            main_test1

            # Source files. Semicolon separated
            "test_main.cpp"

            # Libs. Space separated
            Boost::headers
    )


endif ()

message("INFO: CMake Variable for project ${PROJECT_NAME}")
include("${CMAKE_CURRENT_SOURCE_DIR}/extern/cmake-modules/PrintVariables.cmake")

message("Properties for main")
include("${CMAKE_CURRENT_SOURCE_DIR}/extern/cmake-helpers/print_properties.cmake")
print_target_properties(main)
message("End properties")


# TODO: Add tests and install targets if needed.
