
cmake_minimum_required(VERSION 3.24)

cmake_policy(PUSH)
# Enable setting VERSION in project() call
cmake_policy(SET CMP0048 NEW)

project(kilight-protocol
        VERSION 0.1.0
        DESCRIPTION "Library containing the Protobuf files that define the network protocol of the KiLight project"
        LANGUAGES C CXX)

add_subdirectory(lib)

add_subdirectory(src)

cmake_policy(POP)
