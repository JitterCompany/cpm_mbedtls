cmake_minimum_required(VERSION 3.5.0 FATAL_ERROR)


set(CPM_MODULE_NAME cpm_mbedtls)

#-----------------------------------------------------------------------
# Required CPM Setup - no need to modify - See: https://github.com/iauns/cpm
#-----------------------------------------------------------------------
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
    set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
    include(CPM)
else()
    set (CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
    find_package(Git)
    if(NOT GIT_FOUND)
        message(FATAL_ERROR "CPM requires Git.")
    endif()
    if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
        message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
        execute_process(
            COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
            RESULT_VARIABLE error_code
            OUTPUT_QUIET ERROR_QUIET)
        if(error_code)
            message(FATAL_ERROR "CPM failed to get the hash for HEAD")
        endif()
    endif()
    include(${CPM_DIR}/CPM.cmake)
endif()

#-----------------------------------------------------------------------
# Project sources / settings
#-----------------------------------------------------------------------

# this is required because this is an 'external' module
CPM_ForceOnlyOneModuleVersion()

set(INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/external")
set(MBEDTLS_SRC_DIR "${INC_DIR}/mbedtls/")
set(MBEDTLS_LIB_DIR "${MBEDTLS_SRC_DIR}library/")
set(MBEDTLS_INC_DIR "${MBEDTLS_SRC_DIR}include/")

message(STATUS "target dir: '${MBEDTLS_SRC_DIR}'")
CPM_EnsureRepoIsCurrent(
    TARGET_DIR ${MBEDTLS_SRC_DIR}
    GIT_REPOSITORY "https://github.com/ARMmbed/mbedtls.git"
    GIT_TAG "mbedtls-2.4.0"
    USE_CACHING TRUE
    )

CPM_ExportAdditionalIncludeDir("${MBEDTLS_INC_DIR}")

set(LIB_INSTALL_DIR lib)
set(libs ${CPM_LIBRARIES})
include_directories("${MBEDTLS_INC_DIR}")
add_subdirectory("${MBEDTLS_LIB_DIR}")
                                           
add_library(${CPM_LIB_TARGET_NAME} ALIAS mbedtls)

CPM_InitModule(${CPM_MODULE_NAME})

