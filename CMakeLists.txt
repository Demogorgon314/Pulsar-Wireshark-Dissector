cmake_minimum_required(VERSION 3.21)
project(pulsar_wireshark)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_FLAGS "-O3 -g ${CMAKE_CXX_FLAGS}")
set(CMAKE_BUILD_TYPE RELEASE)

# Wireshark dependency's
find_library(WIRESHARK_LIB wireshark)
find_library(WIRESHARK_UTIL_LIB wsutil)
find_path(WIRESHARK_INCLUDE_PATH wireshark/config.h)
find_library(GLIB_LIB glib)
include_directories(${GLIB_INCLUDE_DIRS})
include(FindPkgConfig)
pkg_check_modules(GLIB glib-2.0)
include_directories(${WIRESHARK_INCLUDE_PATH}/wireshark ${GLIB_INCLUDE_DIRS} ../lib )

MESSAGE(STATUS "Use WIRESHARK_LIB: ${WIRESHARK_LIB}")
MESSAGE(STATUS "Use WIRESHARK_UTIL_LIB: ${WIRESHARK_UTIL_LIB}")
MESSAGE(STATUS "Use WIRESHARK_INCLUDE_PATH: ${WIRESHARK_INCLUDE_PATH}")
MESSAGE(STATUS "Use GLIB_INCLUDE_DIRS: ${GLIB_INCLUDE_DIRS}")

# Protobuf libs
if (NOT PROTOC_PATH)
    set(PROTOC_PATH protoc)
endif()

include_directories(${Protobuf_INCLUDE_DIRS})
find_library(Protobuf_LIBRARIES protobuf libprotobuf)
find_path(Protobuf_INCLUDE_DIRS google/protobuf/stubs/common.h)

set(PROTO_SRC_PULSAR_API PulsarApi.pb.cc)
set(AUTOGEN_DIR ${CMAKE_BINARY_DIR}/generated)
file(MAKE_DIRECTORY ${AUTOGEN_DIR})
set(LIB_AUTOGEN_DIR ${AUTOGEN_DIR}/lib)
file(MAKE_DIRECTORY ${LIB_AUTOGEN_DIR})
include_directories(${LIB_AUTOGEN_DIR})

# Protobuf generation is only supported natively starting from CMake 3.8
# Using custom command for now
set(PROTO_SOURCES ${LIB_AUTOGEN_DIR}/PulsarApi.pb.cc ${LIB_AUTOGEN_DIR}/PulsarApi.pb.h)

ADD_CUSTOM_COMMAND(
        OUTPUT ${PROTO_SOURCES}
        COMMAND ${PROTOC_PATH} -Iproto proto/PulsarApi.proto --cpp_out=${LIB_AUTOGEN_DIR}
        DEPENDS
        proto/PulsarApi.proto
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

message(PROTO_SOURCES : ${PROTO_SOURCES})
link_libraries(${Protobuf_LIBRARIES})

# Build wireshark shared lib
add_library(pulsar-dissector SHARED src/pulsarDissector.cc ${PROTO_SOURCES})
SET(CMAKE_SHARED_LIBRARY_PREFIX )
SET(CMAKE_SHARED_LIBRARY_SUFFIX .so)
set_target_properties(pulsar-dissector PROPERTIES PREFIX "" DEFINE_SYMBOL "")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set(CMAKE_SHARED_LIBRARY_CREATE_CXX_FLAGS "${CMAKE_SHARED_LIBRARY_CREATE_CXX_FLAGS} -undefined dynamic_lookup")
endif()

if (APPLE)
    target_link_libraries(pulsar-dissector -Wl,-all_load ${PROTO_LIBRARIES})
else ()
    target_link_libraries(pulsar-dissector ${PROTOBUF_LIBRARIES})
endif ()
