cmake_minimum_required(VERSION 3.5)
PROJECT(ZED_GeoTacking)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_BUILD_TYPE "Release")

find_package(ZED 4 REQUIRED)
find_package(CUDA ${ZED_CUDA_VERSION} REQUIRED)
find_package(GLUT REQUIRED)
find_package(GLEW REQUIRED)
SET(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

# GPSD - If found
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)
find_package(GPS)

if(GPS_FOUND)
    add_definitions(-DGPSD_FOUND)
    message("Found GPSD library")
else()
    message("GPSD was not FOUND. the sample will not be able to read live GNSS data")
endif()

# ZEDHub - if found
find_package(SL_IOT)

if(SL_IOT_FOUND)
    message("Found ZEDHub library")
    add_definitions(-DCOMPILE_WITH_ZEDHUB=TRUE)
    link_directories(${SL_IOT_LIB_DIR})
    include_directories(${SL_IOT_INCLUDE_DIR})
endif()

IF(NOT MSVC)
    SET(SPECIAL_OS_LIBS "pthread")
ENDIF()

# Include external lib header:
include_directories(${ZED_INCLUDE_DIRS})
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${GLEW_INCLUDE_DIRS})
include_directories(${GLUT_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

link_directories(${ZED_LIBRARY_DIR})
link_directories(${GLEW_LIBRARY_DIRS})
link_directories(${GLUT_LIBRARY_DIRS})
link_directories(${OpenGL_LIBRARY_DIRS})
link_directories(${CUDA_LIBRARY_DIRS})


FILE(GLOB_RECURSE SRC_FILES src/*.c*)
FILE(GLOB_RECURSE HDR_FILES include/*.h*)

add_executable(${PROJECT_NAME} ${HDR_FILES} ${SRC_FILES})

if(SL_IOT_FOUND)
    set(LIBS sl_iot ${ZED_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENGL_LIBRARIES} ${GLUT_LIBRARY} ${GLEW_LIBRARIES})
else()
    set(LIBS ${ZED_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENGL_LIBRARIES} ${GLUT_LIBRARY} ${GLEW_LIBRARIES})
endif()

if(GPS_FOUND)
    list(APPEND LIBS, ${GPS_LIBRARIES})
endif()

target_link_libraries(${PROJECT_NAME} ${LIBS} ${SPECIAL_OS_LIBS})