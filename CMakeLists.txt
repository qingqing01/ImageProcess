# use opencv plugin
cmake_minimum_required(VERSION 3.2)

project(DeJpeg CXX C)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")
set(PROJ_ROOT ${CMAKE_SOURCE_DIR})
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules)
set(DEJPEG_LINKER_LIBS "")

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORT_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORT_CXX0X)

if(COMPILER_SUPPORT_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORT_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(FATAL_ERROR "Your compiler must support c++11")
endif()

# opencv
find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc)
include_directories(${OpenCV_INCLUDE_DIRS})
list(APPEND DEJPEG_LINKER_LIBS ${OpenCV_LIBS})
message(STATUS "OpenCV found (${OpenCV_CONFIG_PATH})")
add_definitions(-DUSE_OPENCV)

# Python and Numpy
find_package(PythonLibs 2.7 REQUIRED)
find_package(PythonInterp 2.7 REQUIRED)
find_package(Glog)
find_package(NumPy REQUIRED)
include_directories(${PYTHON_INCLUDE_DIR})
include_directories(${LIBGLOG_INCLUDE_DIR})
include_directories(${PYTHON_NUMPY_INCLUDE_DIR})

# boost-python
set(Boost_NO_SYSTEM_PATHS ON)
if (Boost_NO_SYSTEM_PATHS)
  set(BOOST_ROOT $ENV{BOOST_ROOT})
  set(Boost_DIR ${BOOST_ROOT})
  set(Boost_INCLUDE_DIR "${BOOST_ROOT}/include")
  set(Boost_LIBRARIES "${BOOST_ROOT}/lib/")
endif (Boost_NO_SYSTEM_PATHS)
find_package(Boost 1.46 COMPONENTS python)
include_directories(${Boost_INCLUDE_DIR})
message(STATUS "Boost found (${Boost_INCLUDE_DIR})")

list(APPEND DEJPEG_LINKER_LIBS ${Boost_LIBRARIES})

set(DEJPEG_SOURCES 
    DataTransformer.cpp
    PyDecodejpeg.cpp)

set(BUILD_FLAGS
    -fPIC
    -fno-omit-frame-pointer
    -Wextra
    -Wno-unused-parameter
    -Wno-unused-function
    -Wno-error=literal-suffix
    -Wno-error=unused-local-typedefs)

add_library(DeJpeg SHARED ${DEJPEG_SOURCES})
target_compile_options(DeJpeg BEFORE PRIVATE ${BUILD_FLAGS})
target_link_libraries(DeJpeg ${DEJPEG_LINKER_LIBS})
set_target_properties(DeJpeg PROPERTIES PREFIX "")
