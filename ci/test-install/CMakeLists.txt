# Copyright 2018 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# A minimal CMakeList.txt to *test* the install Cloud Bigtable C++ client.
#
# This is not intended to be a demonstration of how to write good CMakeList.txt
# files, nor is it a general solution to create CMake files for
# google-cloud-cpp. It is simply a minimal CMake file to verify that (a) the
# installed libraries contain all required headers, and (b) the installed CMake
# configuration files are usable.

cmake_minimum_required(VERSION 3.5)
project(google-cloud-cpp-install-integration-test CXX C)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Threads REQUIRED)

find_package(bigtable_client 0.1.0 REQUIRED)

# The dependencies are hard-coded because we just want to test the
# google-cloud-cpp installed files.  A real application would use find_package()
# for them too, but we do not install the CMake config files as part of the
# continous integration build.

# With cmake-3.9 and higher this can be simplified to:
#    find_package(Protobuf REQUIRED)
add_library(protobuf::libprotobuf INTERFACE IMPORTED)
set_target_properties(protobuf::libprotobuf PROPERTIES
        INTERFACE_LINK_LIBRARIES "/usr/local/lib/libprotobuf.a;Threads::Threads"
        INTERFACE_INCLUDE_DIRECTORIES "/usr/local/include")

# Ideally these are provided by grpc-config.cmake (or gRPCConfig.cmake), and
# that file is installed by the gRPC cmake, but the continuous integration build
# does not install (yet) those files.
add_library(gRPC::grpc INTERFACE IMPORTED)
set_target_properties(gRPC::grpc PROPERTIES
        INTERFACE_LINK_LIBRARIES "/usr/local/lib/libgrpc.so;protobuf::libprotobuf"
        INTERFACE_INCLUDE_DIRECTORIES "/usr/local/include")

add_library(gRPC::grpc++ INTERFACE IMPORTED)
set_target_properties(gRPC::grpc++ PROPERTIES
        INTERFACE_LINK_LIBRARIES "/usr/local/lib/libgrpc++.so;gRPC::grpc++"
        INTERFACE_INCLUDE_DIRECTORIES "/usr/local/include")

# Defining new targets should be easy.
add_executable(install_integration_test install_integration_test.cc)
target_link_libraries(install_integration_test bigtable::client)
