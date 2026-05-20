# MQRemote -- remote-control RPC plugin (peonMQ origin, protobuf-based).
#
# Hand-authored CMakeLists to bypass tools/conversions/Convert-VcxprojToCMake.ps1,
# which omits the necessary `find_package(protobuf)` call and leaves PROTOC_OUT_DIR
# empty.

include("../../src/Plugin.cmake")

# ---------------------------------------------------------------------
# vcpkg dependencies
# ---------------------------------------------------------------------
find_package(protobuf CONFIG REQUIRED)

# ---------------------------------------------------------------------
# Sources / headers / other files
# ---------------------------------------------------------------------
set(MQRemote_HEADERS
    "Channel.h"
    "ChannelManager.h"
    "Logger.h"
    "resource.h"
)

set(MQRemote_OTHERFILES
    "MQRemote.rc"
    "README.md"
)

set(MQRemote_SOURCES
    "Channel.cpp"
    "ChannelManager.cpp"
    "MQRemote.cpp"
)

source_groups("Header Files" ${MQRemote_HEADERS})
source_group("Resource Files" FILES ${MQRemote_OTHERFILES})
source_groups("Source Files" ${MQRemote_SOURCES})

# ---------------------------------------------------------------------
# Protobuf generation
# ---------------------------------------------------------------------
add_library(MQRemote-proto OBJECT
    "Remote.proto"
)

set_target_properties(MQRemote-proto PROPERTIES FOLDER "external/proto")

target_Plugin_props(MQRemote-proto)

target_link_libraries(MQRemote-proto PUBLIC
    protobuf::libprotobuf
)

target_include_directories(MQRemote-proto PUBLIC
    "$<TARGET_PROPERTY:MQRemote-proto,BINARY_DIR>"
)

# Generate .pb.cc / .pb.h next to the object library's binary dir so the
# include path above resolves them. protobuf_generate auto-adds the
# generated sources to the target.
protobuf_generate(
    TARGET MQRemote-proto
    LANGUAGE cpp
    PROTOC_OUT_DIR "$<TARGET_PROPERTY:MQRemote-proto,BINARY_DIR>"
)

# ---------------------------------------------------------------------
# Target definition
# ---------------------------------------------------------------------
add_library(MQRemote SHARED
    ${MQRemote_HEADERS}
    ${MQRemote_OTHERFILES}
    ${MQRemote_SOURCES}
)

set_target_properties(MQRemote PROPERTIES FOLDER "external")

target_Plugin_props(MQRemote)

target_link_libraries(MQRemote PRIVATE
    pluginapi
    MQ2Main
    MQRemote-proto
)

# ---------------------------------------------------------------------
# PRE_BUILD events
# ---------------------------------------------------------------------
add_custom_command(TARGET MQRemote PRE_BUILD
    COMMAND powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File "${CMAKE_SOURCE_DIR}/tools/build_scripts/Plugin_Versioning.ps1" -ProjectName "${PROJECT_NAME}" -ProjectDirectory "${CMAKE_CURRENT_SOURCE_DIR}/"
    COMMENT "Updating version resource for ${PROJECT_NAME}..."
    VERBATIM
)
