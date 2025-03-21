#
# Copyright 2021 The Modelbox Project Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


cmake_minimum_required(VERSION 3.10)

set(UNIT_DEVICE "cpu")
set(UNIT_NAME "java")

project(modelbox-flowunit-${UNIT_DEVICE}-${UNIT_NAME})

if (NOT WITH_JAVA) 
    message(STATUS "java support is disabled")
    return()
endif()

if(NOT ${JNI_FOUND})
    message(STATUS "Not found java, disable java flowunit")
    return()
endif()

file(GLOB_RECURSE UNIT_SOURCE *.cpp *.cc *.c)
group_source_test_files(MODELBOX_UNIT_SOURCE MODELBOX_UNIT_TEST_SOURCE "_test.c*" ${UNIT_SOURCE})

include_directories(${CMAKE_CURRENT_LIST_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${LIBMODELBOX_INCLUDE})
include_directories(${LIBMODELBOX_BASE_INCLUDE})
include_directories(${LIBMODELBOX_DEVICE_CPU_INCLUDE})
include_directories(${JNI_INCLUDE_DIRS})
include_directories(${MODELBOX_COMMON_MODELBOX_API_INCLUDE})
include_directories(${LIBMODELBOX_VIRTUALDRIVER_JAVA_INCLUDE})

set(EMPTY_SOURCE_FILE ${CMAKE_BINARY_DIR}/empty.cc)
file(WRITE ${EMPTY_SOURCE_FILE})

set(MODELBOX_UNIT_SHARED modelbox-unit-${UNIT_DEVICE}-${UNIT_NAME}-shared)
set(MODELBOX_UNIT_STATIC modelbox-unit-${UNIT_DEVICE}-${UNIT_NAME}-static)
set(MODELBOX_UNIT_SOURCE_INCLUDE ${CMAKE_CURRENT_LIST_DIR})

add_library(${MODELBOX_UNIT_STATIC} STATIC ${MODELBOX_UNIT_SOURCE})
target_link_libraries(${MODELBOX_UNIT_STATIC} ${JNI_LIBRARIES})
set_property(TARGET ${MODELBOX_UNIT_STATIC} PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(${MODELBOX_UNIT_SHARED} SHARED ${EMPTY_SOURCE_FILE})
target_link_libraries(${MODELBOX_UNIT_SHARED} ${MODELBOX_UNIT_STATIC} )
target_link_libraries(${MODELBOX_UNIT_SHARED} -Wl,--whole-archive ${MODELBOX_UNIT_STATIC} -Wl,--no-whole-archive)
set(LIBMODELBOX_FLOWUNIT_JAVA_SHARED ${MODELBOX_UNIT_SHARED})
target_link_libraries(${MODELBOX_UNIT_SHARED} ${MODELBOX_COMMON_MODELBOX_API_LIBRARY})
target_link_libraries(${MODELBOX_UNIT_SHARED} ${LIBMODELBOX_SHARED})
add_dependencies(${MODELBOX_UNIT_SHARED} ${MODELBOX_COMMON_MODELBOX_API_LIBRARY})

set_target_properties(${MODELBOX_UNIT_SHARED} PROPERTIES 
    SOVERSION ${MODELBOX_VERSION_MAJOR}
    VERSION ${MODELBOX_VERSION_MAJOR}.${MODELBOX_VERSION_MINOR}.${MODELBOX_VERSION_PATCH}
)

target_link_libraries(${MODELBOX_UNIT_SHARED} ${LIBMODELBOX_SHARED})
target_link_libraries(${MODELBOX_UNIT_SHARED} pthread)
target_link_libraries(${MODELBOX_UNIT_SHARED} rt)
target_link_libraries(${MODELBOX_UNIT_SHARED} dl)

set_target_properties(${MODELBOX_UNIT_SHARED} PROPERTIES OUTPUT_NAME "modelbox-unit-${UNIT_DEVICE}-${UNIT_NAME}")

install(TARGETS ${MODELBOX_UNIT_SHARED}
    COMPONENT cpu-device-flowunit
    RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
    OPTIONAL
    )

install(DIRECTORY 
    ${HEADER} DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}
    COMPONENT cpu-device-flowunit
    )

set(LIBMODELBOX_FLOWUNIT_JAVA_SHARED ${MODELBOX_UNIT_SHARED} CACHE INTERNAL "")
set(LIBMODELBOX_FLOWUNIT_JAVA_INCLUDE ${MODELBOX_UNIT_SOURCE_INCLUDE} CACHE INTERNAL "")
set(LIBMODELBOX_FLOWUNIT_JAVA_SOURCES ${MODELBOX_UNIT_SOURCE} CACHE INTERNAL "")
set(LIBMODELBOX_FLOWUNIT_JAVA_SO_PATH ${CMAKE_CURRENT_BINARY_DIR}/libmodelbox-unit-${UNIT_DEVICE}-${UNIT_NAME}.so CACHE INTERNAL "")

# driver test
list(APPEND DRIVER_UNIT_TEST_SOURCE ${MODELBOX_UNIT_TEST_SOURCE})
list(APPEND DRIVER_UNIT_TEST_TARGET ${MODELBOX_UNIT_SHARED})
list(APPEND DRIVER_UNIT_TEST_INCLUDE ${JAVA_INCLUDE_DIRS})
list(APPEND DRIVER_UNIT_TEST_LINK_LIBRARIES pybind11::module pybind11::embed)
set(DRIVER_UNIT_TEST_SOURCE ${DRIVER_UNIT_TEST_SOURCE} CACHE INTERNAL "")
set(DRIVER_UNIT_TEST_TARGET ${DRIVER_UNIT_TEST_TARGET} CACHE INTERNAL "")
set(DRIVER_UNIT_TEST_INCLUDE ${DRIVER_UNIT_TEST_INCLUDE} CACHE INTERNAL "")
set(DRIVER_UNIT_TEST_LINK_LIBRARIES ${DRIVER_UNIT_TEST_LINK_LIBRARIES} CACHE INTERNAL "")
set(LIBMODELBOX_FLOWUNIT_JAVA_SO_PATH ${LIBMODELBOX_FLOWUNIT_JAVA_SO_PATH} CACHE INTERNAL "")

