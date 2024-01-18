cmake_minimum_required(VERSION 3.22)
if(APPLE) # MoltenVK
    include_directories(/usr/local/include)
endif()
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake")
project(ProjectCxxPy)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(SWIG REQUIRED)
include(${SWIG_USE_FILE})
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_PATH})

find_package(Qt6 CONFIG REQUIRED)
find_package(Qt6Gui CONFIG REQUIRED)
find_package(Qt6Widgets CONFIG REQUIRED)
find_package(unofficial-shaderc CONFIG REQUIRED)
find_package(Vulkan REQUIRED)

qt_standard_project_setup()
set(CMAKE_AUTOMOC ON)

set(SOURCE_FILES lib.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_SWIG_FLAGS)
set_property(SOURCE lib.i PROPERTY CPLUSPLUS ON)
swig_add_library(LibPy TYPE SHARED LANGUAGE python SOURCES lib.i ${SOURCE_FILES})
target_include_directories(LibPy PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
swig_link_libraries(LibPy ${PYTHON_LIBRARIES})

# set(CMAKE_CXX_CLANG_TIDY clang-tidy)

add_library(LibCxx ${SOURCE_FILES})

qt_add_executable(MainCxx main.cpp)
# set_target_properties(MainCxx PROPERTIES
#     WIN32_EXECUTABLE TRUE
#     MACOSX_BUNDLE TRUE)
target_link_libraries(MainCxx PRIVATE
    LibCxx
    Qt::Core Qt::Gui Qt::Widgets
    unofficial::shaderc::shaderc
    Vulkan::Vulkan)
