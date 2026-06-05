# Copyright (C) 2008 by BogDan Vatra < bogdan@licentia.eu >
# Copyright (C) 2009-2022 Robin Stuart <rstuart114@gmail.com>
# vim: set ts=4 sw=4 et :

project(zint)

if(ZINT_USE_PNG)
    find_package(PNG)
endif()

set(zint_COMMON_SRCS common.c library.c large.c reedsol.c gs1.c eci.c general_field.c)
set(zint_ONEDIM_SRCS bc412.c code.c code128.c 2of5.c upcean.c telepen.c medical.c plessey.c rss.c)
set(zint_POSTAL_SRCS postal.c auspost.c imail.c mailmark.c)
set(zint_TWODIM_SRCS code16k.c codablock.c dmatrix.c pdf417.c qr.c maxicode.c composite.c aztec.c code49.c code1.c gridmtx.c hanxin.c dotcode.c ultra.c)
if(ZINT_USE_PNG AND PNG_FOUND)
    set(zint_OUTPUT_SRCS vector.c ps.c svg.c emf.c bmp.c pcx.c gif.c png.c tif.c raster.c output.c)
else()
    set(zint_OUTPUT_SRCS vector.c ps.c svg.c emf.c bmp.c pcx.c gif.c tif.c raster.c output.c)
endif()
set(zint_SRCS ${zint_OUTPUT_SRCS} ${zint_COMMON_SRCS} ${zint_ONEDIM_SRCS} ${zint_POSTAL_SRCS} ${zint_TWODIM_SRCS})

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

# Custom library name
set(library_name "zint")
if(ZINT_SHARED)
    add_library(${library_name} SHARED ${zint_SRCS})

    if(WIN32)
        target_sources(${PROJECT_NAME} PRIVATE libzint.rc)
    endif()
endif()

# Alias name
add_library(ZINT::ZINT ALIAS ${library_name})

if(ZINT_STATIC)
    add_library(${library_name}-static STATIC ${zint_SRCS})
    if(WIN32)
        set_target_properties(${library_name}-static PROPERTIES OUTPUT_NAME zint-static)
    else()
        set_target_properties(${library_name}-static PROPERTIES OUTPUT_NAME zint)
    endif()
endif()

function(zint_target_link_libraries library)
    if(ZINT_SHARED)
        target_link_libraries(${library_name} ${library})
    endif()
    if(ZINT_STATIC)
        target_link_libraries(${library_name}-static ${library})
    endif()
endfunction()

function(zint_target_compile_definitions scope definition)
    if(ZINT_SHARED)
        target_compile_definitions(${library_name} ${scope} ${definition})
    endif()
    if(ZINT_STATIC)
        target_compile_definitions(${library_name}-static ${scope} ${definition})
    endif()
endfunction()

function(zint_target_include_directories)
    if(ZINT_SHARED)
        target_include_directories(${library_name} ${ARGN})
    endif()
    if(ZINT_STATIC)
        target_include_directories(${library_name}-static ${ARGN})
    endif()
endfunction()

if(ZINT_SHARED)
    set_target_properties(${library_name} PROPERTIES   SOVERSION "${ZINT_VERSION_MAJOR}.${ZINT_VERSION_MINOR}"
                                            VERSION ${ZINT_VERSION})
    if(APPLE)
      set_target_properties(${library_name} PROPERTIES INSTALL_NAME_DIR "@rpath")
    endif()
endif()

if(ZINT_USE_PNG AND PNG_FOUND)
    zint_target_link_libraries(PNG::PNG)
    message(STATUS "Using PNG")
else()
    zint_target_compile_definitions(PRIVATE ZINT_NO_PNG)
    message(STATUS "Not using PNG")
endif()

if(ZINT_TEST)
    zint_target_compile_definitions(PUBLIC ZINT_TEST)
endif()

if(NOT MSVC)
    # Link with standard C math library.
    zint_target_link_libraries(m)
endif()

if(MSVC)
    if(ZINT_SHARED)
        target_compile_definitions(${library_name} PRIVATE DLL_EXPORT)
    endif()
endif()

zint_target_include_directories(PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${INCLUDE_INSTALL_DIR}>
)

if(ZINT_SHARED)
    install(TARGETS ${library_name} EXPORT zint-targets ${INSTALL_TARGETS_DEFAULT_ARGS})
endif()
if(ZINT_STATIC)
    install(TARGETS ${library_name}-static EXPORT zint-targets ${INSTALL_TARGETS_DEFAULT_ARGS})
endif()
install(EXPORT zint-targets NAMESPACE zint:: DESTINATION "${SHARE_INSTALL_PREFIX}/zint")
install(FILES zint.h DESTINATION ${INCLUDE_INSTALL_DIR} COMPONENT Devel)

if(ZINT_TEST)
    add_subdirectory(tests)
endif()
