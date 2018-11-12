cmake_minimum_required(VERSION 3.2)
include(GNUInstallDirs)

project(pplx VERSION 2.9.1 DESCRIPTION "Extraction of Microsoft PPLX out of C++ REST SDK")

find_package(Boost COMPONENTS thread date_time)

# Provide an option for choosing between shared or static library builds. Build as shared by default.
option(${PROJECT_NAME}_as_shared "build as a shared library" ON)
set(BUILD_SHARED_LIBS ${${PROJECT_NAME}_as_shared})
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -std=c++11)

add_library(${PROJECT_NAME} "")
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION} POSITION_INDEPENDENT_CODE ON)
target_include_directories(${PROJECT_NAME} PRIVATE ${Boost_INCLUDE_DIRS})
target_include_directories(${PROJECT_NAME} PRIVATE include)
target_sources(${PROJECT_NAME}
    PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/compat.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxcancellation_token.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplx.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxinterface.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxtasks.h"
    PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/src/pplx.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/stdafx.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/stdafx.h"
)

# There's a dedicated thread pool implementation for the UNIX side, while Windows provides its own.
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxlinux.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxlinux.cpp"
            "${CMAKE_CURRENT_SOURCE_DIR}/src/threadpool.cpp"
            "${CMAKE_CURRENT_SOURCE_DIR}/src/threadpool.h"
    )
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxlinux.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxapple.cpp"
    )
elseif(WIN32)
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxwin.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxwin.cpp"
    )
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

set(include_install_dir "include")
set(lib_install_dir "lib")
set(bin_install_dir "bin")
install(
    DIRECTORY "${PROJECT_SOURCE_DIR}/include/pplx"
    DESTINATION ${include_install_dir}
)
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION ${bin_install_dir}
    LIBRARY DESTINATION ${lib_install_dir}
    ARCHIVE DESTINATION ${lib_install_dir}
)
