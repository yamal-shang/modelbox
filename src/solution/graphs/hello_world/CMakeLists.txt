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

subdirlist(SUBDIRS ${CMAKE_CURRENT_SOURCE_DIR} "CMakeLists.txt")

set(GRAPH_NAME "hello_world.toml")
set(TEST_HELLO_WORLD_NAME "test_hello_world.py")
set(SOLUTION_GRAPH ${CMAKE_CURRENT_BINARY_DIR}/${GRAPH_NAME})
set(SOLUTION_TEST ${CMAKE_CURRENT_BINARY_DIR}/${TEST_HELLO_WORLD_NAME})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${GRAPH_NAME}.in ${SOLUTION_GRAPH} @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${TEST_HELLO_WORLD_NAME}.in ${SOLUTION_TEST} @ONLY)

install(FILES 
    ${SOLUTION_GRAPH}  ${SOLUTION_TEST} DESTINATION ${SOLUTION_GRAPHS_SUB}
    COMPONENT solution
)
