cmake_minimum_required(VERSION 3.10)
project(arrow_protobuf_reproduction VERSION 1.0.0 LANGUAGES CXX)

find_package(Arrow REQUIRED)
# ARROW-12175: Arrow libraries are not CMake components
get_filename_component(ARROW_CONFIG_PATH ${Arrow_CONFIG} DIRECTORY)
find_package(ArrowFlight REQUIRED HINTS ${ARROW_CONFIG_PATH})
find_package(Protobuf REQUIRED)

include_directories(${Protobuf_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(GENERATED_PROTO_FILES
  "${CMAKE_CURRENT_BINARY_DIR}/Flight.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/test.pb.cc")
set_source_files_properties(${GENERATED_PROTO_FILES} PROPERTIES GENERATED TRUE)
add_custom_command(OUTPUT ${GENERATED_PROTO_FILES}
                   COMMAND protobuf::protoc "-I${CMAKE_CURRENT_SOURCE_DIR}/src"
                   "--cpp_out=${CMAKE_CURRENT_BINARY_DIR}"
                   "Flight.proto" "test.proto")

if (WITH_GENERATED_FLIGHT)
  add_executable(demo src/main.cc "${GENERATED_PROTO_FILES}")
else()
  add_executable(demo src/main.cc "${CMAKE_CURRENT_BINARY_DIR}/test.pb.cc")
endif()
target_link_libraries(demo ${Protobuf_LIBRARIES} arrow_shared arrow_flight_shared)
