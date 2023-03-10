cmake_minimum_required(VERSION 3.26)

project(directx-shader-compiler)

include(FetchContent)

FetchContent_Declare(
  DIRECTX_SHADER_COMPILER
  URL      https://github.com/microsoft/DirectXShaderCompiler/releases/download/v1.7.2212/dxc_2022_12_16.zip
  URL_HASH SHA256=ed77c7775fcf1e117bec8b5bb4de6735af101b733d3920dda083496dceef130f
)

FetchContent_MakeAvailable(DIRECTX_SHADER_COMPILER)

FetchContent_GetProperties(
    DIRECTX_SHADER_COMPILER
    SOURCE_DIR DIRECTX_SHADER_COMPILER_SOURCE_DIR
    BINARY_DIR DIRECTX_SHADER_COMPILER_BINARY_DIR
    POPULATED  DIRECTX_SHADER_COMPILER_POPULATED
)

if (DIRECTX_SHADER_COMPILER_POPULATED)
    message(STATUS "Successfully downloaded DirectX Shader Compiler (abbrev. as DXC later on)!")        

    set(DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR ${DIRECTX_SHADER_COMPILER_SOURCE_DIR}/bin/x64                CACHE PATH "The directory that contains DXC (DLLs).")
    set(DIRECTX_SHADER_COMPILER_SHARED_LIBRARY     ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR}/dxcompiler.dll CACHE PATH "The compiler DLL provided by the DXC (should be in DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR).")
    set(DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY  ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR}/dxil.dll       CACHE PATH "The IL DLL provided by the DXC (should be in DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR).")
    set(DIRECTX_SHADER_COMPILER_EXECUTABLE         ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR}/dxc.exe        CACHE PATH "The compiler executable provided by the DXC (should be in DIRECTX_SHADER_COMPILER_SHARED_LIBRARY_DIR).")

    set(DIRECTX_SHADER_COMPILER_LIBRARY_DIR        ${DIRECTX_SHADER_COMPILER_SOURCE_DIR}/lib/x64                CACHE PATH "The directory that contains DXC (LIBs).")
    set(DIRECTX_SHADER_COMPILER_LIBRARY            ${DIRECTX_SHADER_COMPILER_LIBRARY_DIR}/dxcompiler.lib        CACHE PATH "The compiler LIB provided by the DXC (should be in DIRECTX_SHADER_COMPILER_LIBRARY_DIR).")

    set(DIRECTX_SHADER_COMPILER_INCLUDE_DIR        ${DIRECTX_SHADER_COMPILER_SOURCE_DIR}/inc                    CACHE PATH "The directory that contains the include files for DXC.")
endif()

message(STATUS "-- DIRECTX_SHADER_COMPILER_SHARED_LIBRARY      : ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY}")
message(STATUS "-- DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY   : ${DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY}")
message(STATUS "-- DIRECTX_SHADER_COMPILER_EXECUTABLE          : ${DIRECTX_SHADER_COMPILER_EXECUTABLE}")
message(STATUS "-- DIRECTX_SHADER_COMPILER_LIBRARY             : ${DIRECTX_SHADER_COMPILER_LIBRARY}")
message(STATUS "-- DIRECTX_SHADER_COMPILER_INCLUDE_DIR         : ${DIRECTX_SHADER_COMPILER_INCLUDE_DIR}")

list(APPEND SHARED_LIBRARIES ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY})
list(APPEND SHARED_LIBRARIES ${DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY})

list(APPEND LIBRARIES ${DIRECTX_SHADER_COMPILER_LIBRARY})

list(APPEND INCLUDES ${DIRECTX_SHADER_COMPILER_INCLUDE_DIR})

add_library(dxcompiler SHARED IMPORTED)
set_target_properties(dxcompiler PROPERTIES
    IMPORTED_LOCATION ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY}
    IMPORTED_IMPLIB   ${DIRECTX_SHADER_COMPILER_LIBRARY}
)
target_include_directories(dxcompiler INTERFACE ${INCLUDES})

add_library(dxil SHARED IMPORTED)
set_target_properties(dxil PROPERTIES
    IMPORTED_LOCATION ${DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY}
    IMPORTED_IMPLIB   ${DIRECTX_SHADER_COMPILER_LIBRARY}
)
target_include_directories(dxil INTERFACE ${INCLUDES})

function(copy_directx_shader_compiler_shared_libraries)
    set(options)
    set(oneValueArgs TARGET)
    set(multiValueArgs)
    cmake_parse_arguments(PARAM "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})

    if(NOT PARAM_TARGET)
        message(FATAL_ERROR "Required parameter `TARGET` not defined")
    endif()

    if(NOT TARGET ${PARAM_TARGET})
        message(FATAL_ERROR "Not a target")
    endif()

    add_custom_command(TARGET ${PARAM_TARGET} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${DIRECTX_SHADER_COMPILER_SHARED_LIBRARY} ${DIRECTX_SHADER_COMPILER_IL_SHARED_LIBRARY} $<TARGET_FILE_DIR:${PARAM_TARGET}>
	)

endfunction()
