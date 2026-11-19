cmake_minimum_required(VERSION 3.10)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    project(ars_common LANGUAGES C)
endif()

# Parent projects normally add shm-ipc first. An explicit source path also
# supports standalone generation/checks without embedding another repository.
set(ARS_COMMON_SHM_IPC_SOURCE_DIR "" CACHE PATH "Path to the separate shm-ipc library checkout")
if(NOT COMMAND shm_ipc_generate_interface)
    if(NOT ARS_COMMON_SHM_IPC_SOURCE_DIR OR
       NOT EXISTS "${ARS_COMMON_SHM_IPC_SOURCE_DIR}/CMakeLists.txt")
        message(FATAL_ERROR "Add shm-ipc before ars-common, or set ARS_COMMON_SHM_IPC_SOURCE_DIR to its checkout")
    endif()
    add_subdirectory("${ARS_COMMON_SHM_IPC_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}/shm-ipc")
endif()

# Keep both directions in one device-owned IDL. Generated C/headers live only
# in the consuming build directory, with automatic function IDs from the IDL.
shm_ipc_generate_interface(ars_ipc IDL "${CMAKE_CURRENT_SOURCE_DIR}/interfaces/device_ipc.shm")
add_library(ars_common::ipc ALIAS ars_ipc)
