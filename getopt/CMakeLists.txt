project(zint_bundled_getopt)

# Set MAC SDK Path
if(APPLE)
    execute_process(
        COMMAND xcrun --sdk macosx --show-sdk-path
        OUTPUT_VARIABLE OSX_SDK_PATH
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    set(CMAKE_OSX_SYSROOT ${OSX_SDK_PATH})
    message(STATUS "Using macOS SDK: ${OSX_SDK_PATH}")
endif()

add_library(${PROJECT_NAME} STATIC getopt.c getopt1.c)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
