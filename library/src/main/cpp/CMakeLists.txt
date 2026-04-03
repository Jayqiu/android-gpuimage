# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

project(yuv-decoder)

add_library(
        yuv-decoder
        SHARED
        yuv-decoder.c)

find_library(android-lib android)
find_library(log-lib log)
find_library(opengles-lib GLESv2)
find_library(jnigraphics-lib jnigraphics)

target_link_libraries(yuv-decoder
        ${android-lib}
        ${log-lib}
        ${opengles-lib}
        ${jnigraphics-lib})

set_target_properties(yuv-decoder PROPERTIES
    LINK_FLAGS "-Wl,-z,max-page-size=16384")