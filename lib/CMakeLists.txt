
set(EmbeddedProto_DIR "${CMAKE_CURRENT_LIST_DIR}/EmbeddedProto" CACHE INTERNAL "")

if (CMAKE_HOST_SYSTEM_NAME STREQUAL "Darwin")
    # Need to specify these manually since we're cross-compiling and otherwise FindProtobuf looks for the target
    # library types rather than the host, even if you tell it not to.
    if (NOT Protobuf_LIBRARIES)
        if (NOT Protobuf_LIBRARIES AND EXISTS "/opt/homebrew/lib/libprotobuf.dylib")
            set(Protobuf_LIBRARIES "/opt/homebrew/lib/libprotobuf.dylib" CACHE INTERNAL "")
        elseif (NOT Protobuf_LIBRARIES AND EXISTS "/usr/local/lib/libprotobuf.dylib")
            set(Protobuf_LIBRARIES "/usr/local/lib/libprotobuf.dylib" CACHE INTERNAL "")
        endif ()
    endif ()

    if (NOT Protobuf_LITE_LIBRARIES)
        if (NOT Protobuf_LITE_LIBRARIES AND EXISTS "/opt/homebrew/lib/libprotobuf-lite.dylib")
            set(Protobuf_LITE_LIBRARIES "/opt/homebrew/lib/libprotobuf-lite.dylib" CACHE INTERNAL "")
        elseif (NOT Protobuf_LITE_LIBRARIES AND EXISTS "/usr/local/lib/libprotobuf-lite.dylib")
            set(Protobuf_LITE_LIBRARIES "/usr/local/lib/libprotobuf-lite.dylib" CACHE INTERNAL "")
        endif ()
    endif ()

    if (NOT Protobuf_PROTOC_LIBRARIES)
        if (EXISTS "/opt/homebrew/lib/libprotoc.dylib")
            set(Protobuf_PROTOC_LIBRARIES "/opt/homebrew/lib/libprotoc.dylib" CACHE INTERNAL "")
        elseif (EXISTS "/usr/local/lib/libprotoc.dylib")
            set(Protobuf_PROTOC_LIBRARIES "/usr/local/lib/libprotoc.dylib" CACHE INTERNAL "")
        endif ()
    endif ()
endif ()

find_package(Protobuf REQUIRED)

find_package(Python3 COMPONENTS Interpreter REQUIRED)

if (CMAKE_HOST_SYSTEM_NAME STREQUAL "Windows")
    set(EmbeddedProto_PLUGIN_PATH "${EmbeddedProto_DIR}/venv/Scripts/protoc-gen-eams" CACHE INTERNAL "")
else ()
    set(EmbeddedProto_PLUGIN_PATH "${EmbeddedProto_DIR}/venv/bin/protoc-gen-eams" CACHE INTERNAL "")
endif ()

if (NOT EXISTS "${EmbeddedProto_PLUGIN_PATH}")
    message(STATUS "Setting up EmbeddedProto...")
    execute_process(COMMAND "${Python3_EXECUTABLE}" setup.py --include "${Protobuf_INCLUDE_DIR}" --ignore_version_diff
            WORKING_DIRECTORY "${EmbeddedProto_DIR}"
            COMMAND_ERROR_IS_FATAL ANY)
else ()
    message(STATUS "EmbeddedProto already set up, skipping.")
endif ()

file(GLOB_RECURSE EmbeddedProto_SOURCES
        CONFIGURE_DEPENDS
        "${EmbeddedProto_DIR}/src/*.h"
        "${EmbeddedProto_DIR}/src/*.cpp"
)

add_library(EmbeddedProto STATIC
        ${EmbeddedProto_SOURCES}
)

target_include_directories(EmbeddedProto PUBLIC
    "${EmbeddedProto_DIR}/src"
)
