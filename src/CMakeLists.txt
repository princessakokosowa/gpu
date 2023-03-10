cmake_minimum_required(VERSION 3.26)

# Standard bullshit that a programmer has to deal with every time they set up a build
# system, why do I even have to do this?
set(TARGET gpu)
add_executable(${TARGET})

# Specify the working directory for the MSVC debugger (and probably any other debugger
# that runs on Windows, e.g., RemedyBG, which I use on a daily basis).
set_property(TARGET ${TARGET} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# Specify the required C standard.
set_target_properties(${TARGET} PROPERTIES C_STANDARD                17)
set_target_properties(${TARGET} PROPERTIES CMAKE_C_STANDARD_REQUIRED True)

# Specify libraries for the target (and since we are not linking ANYTHING yet, we cannot
# do anything like what is shown below, because there is absolutely not symmetry between
# similarly looking commands in CMake). I did this - `list(APPEND LIBRARIES .)`.

# This is possible due to the fact that Windows SDK is in the standard search paths
# e.g., of the linker for libraries.
list(APPEND LIBRARIES d3d12.lib)
list(APPEND LIBRARIES dxgi.lib)
list(APPEND LIBRARIES dxguid.lib)

# The rest, though, has to be searched for and properly handled.
list(APPEND LIBRARIES d3d12-headers)
list(APPEND LIBRARIES dxcompiler)
# list(APPEND LIBRARIES dxil)

# Specify sources and include directories for the target.
list(APPEND SOURCES  main.c)
# list(APPEND INCLUDES os.h)
# list(APPEND INCLUDES foundation.h)

# Link libraries against the target.
target_link_libraries(${TARGET} ${LIBRARIES})

# Bind the sources and the include directories to the target.
target_sources(${TARGET} PRIVATE ${SOURCES})
# target_include_directories(${TARGET} PRIVATE ${INCLUDES})

copy_directx_12_agility_sdk_shared_libraries(TARGET ${TARGET})
copy_directx_shader_compiler_shared_libraries(TARGET ${TARGET})
