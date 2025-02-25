
set(PROTO_OUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")

add_library(kilight-protocol INTERFACE
        kilight/protocol/Color.proto
        kilight/protocol/FanState.proto
        kilight/protocol/OutputIdentifier.proto
        kilight/protocol/OutputState.proto
        kilight/protocol/OutputWriteRequest.proto
        kilight/protocol/OutputWriteResponse.proto
        kilight/protocol/Request.proto
        kilight/protocol/Response.proto
        kilight/protocol/SystemInfoRequest.proto
        kilight/protocol/SystemInfoResponse.proto
        kilight/protocol/SystemStateRequest.proto
        kilight/protocol/SystemStateResponse.proto
        kilight/protocol/SystemTemperatures.proto
        kilight/protocol/VersionInfo.proto
)

target_include_directories(kilight-protocol INTERFACE
    "${PROTO_OUT_DIR}"
)

target_link_libraries(kilight-protocol INTERFACE
        EmbeddedProto
)

set_target_properties(kilight-protocol PROPERTIES
        C_STANDARD 11
        C_STANDARD_REQUIRED ON
        CXX_STANDARD 20
        CXX_STANDARD_REQUIRED ON
        CXX_EXTENSIONS ON
)

get_target_property(PROTO_FILES kilight-protocol SOURCES)

list(FILTER PROTO_FILES INCLUDE REGEX ".+\\.proto$")

foreach(currentProtoFile IN LISTS PROTO_FILES)
    cmake_path(GET currentProtoFile STEM LAST_ONLY currentProtoFileStem)
    # protobuf_generate caused weird build errors, probably due to using the EmbeddedProto plugin,
    # so just run it as a custom command for now...
    add_custom_command(
            COMMENT "Running protoc on proto file \"${currentProtoFile}\"..."
            MAIN_DEPENDENCY "${currentProtoFile}"
            OUTPUT "${PROTO_OUT_DIR}/${currentProtoFileStem}.h"
            VERBATIM
            WORKING_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}"
            COMMAND "${Protobuf_PROTOC_EXECUTABLE}"
                "--plugin=${EmbeddedProto_PLUGIN_PATH}"
                "-I${EmbeddedProto_DIR}/generator"
                "-I${CMAKE_CURRENT_LIST_DIR}"
                "--eams_out=${PROTO_OUT_DIR}"
                "${currentProtoFile}"
    )
    target_sources(kilight-protocol INTERFACE
            "${PROTO_OUT_DIR}/${currentProtoFileStem}.h"
    )
endforeach ()
