
# Copyright 2024 Proyectos y Sistemas de Mantenimiento SL (eProsima).
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

cmake_minimum_required(VERSION 3.20)

project("generated_code")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)

# Find requirements
find_package(fastcdr REQUIRED)
find_package(fastdds 3 REQUIRED)

# Set CMAKE_BUILD_TYPE to Release by default.
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()


message(STATUS "Configuring HelloWorld...")
add_library(HelloWorld_lib HelloWorldTypeObjectSupport.cxx HelloWorldPubSubTypes.cxx)
target_compile_definitions(HelloWorld_lib PRIVATE $<$<BOOL:${BUILD_SHARED_LIBS}>:EPROSIMA_USER_DLL_EXPORT>)
target_link_libraries(HelloWorld_lib fastcdr fastdds)

add_executable(HelloWorld HelloWorldApplication.cxx HelloWorldPublisherApp.cxx HelloWorldSubscriberApp.cxx HelloWorldmain.cxx)
target_link_libraries(HelloWorld fastcdr fastdds
        HelloWorld_lib 
        )



