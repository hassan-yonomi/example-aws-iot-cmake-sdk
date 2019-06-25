cmake_minimum_required(VERSION 2.6)
project(v2_sdk C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_VERBOSE_MAKEFILE ON)

set(AWS_SRC_DIR ${PROJECT_SOURCE_DIR}/aws-iot-device-sdk-embedded-C)
set(PLATFORM_DIR ${AWS_SRC_DIR}/platform/linux/mbedtls)
set(PLATFORM_COMMON_DIR ${AWS_SRC_DIR}/platform/linux/common)
set(MBEDTLS_DIR ${PROJECT_SOURCE_DIR}/mbedtls)
set(TLS_LIB_DIR ${MBEDTLS_DIR}/library)
set(CRYPTO_LIB_DIR ${MBEDTLS_DIR}/crypto/library)

add_custom_target(mbedtls
        COMMAND make
        WORKING_DIRECTORY ${MBEDTLS_DIR}
        )

file(GLOB IOT_SRC_FILES ${AWS_SRC_DIR}/src/*.c
        ${AWS_SRC_DIR}/external_libs/jsmn/*.c
        ${PLATFORM_COMMON_DIR}/*.c
        ${PLATFORM_DIR}/*.c
        )

add_library(aws-sdk STATIC ${IOT_SRC_FILES})
add_dependencies(aws-sdk mbedtls)
target_include_directories(
        aws-sdk
        PRIVATE
        ${PLATFORM_COMMON_DIR}
        ${PLATFORM_DIR}
        ${MBEDTLS_DIR}/include
        ${AWS_SRC_DIR}/include
        ${AWS_SRC_DIR}/external_libs/jsmn
        .
)
target_link_libraries(aws-sdk
        dl
        pthread
        ${TLS_LIB_DIR}/libmbedtls.a
        ${CRYPTO_LIB_DIR}/libmbedcrypto.a
        ${TLS_LIB_DIR}/libmbedx509.a
        )

add_executable(v2_sdk library.c test.c)
target_link_libraries(v2_sdk
        aws-sdk
        )
target_include_directories(
        v2_sdk
        PRIVATE
        ${PLATFORM_COMMON_DIR}
        ${PLATFORM_DIR}
        ${MBEDTLS_DIR}/include
        ${AWS_SRC_DIR}/include
        ${AWS_SRC_DIR}/external_libs/jsmn
        .
)
