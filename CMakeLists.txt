cmake_minimum_required(VERSION 3.22)

project(synth C)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type." FORCE)
endif()

option(BUILD_STANDALONE "Build the standalone application" ON)
option(BUILD_CLAP_PLUGIN "Build the CLAP plugin" OFF)

include(FetchContent)

# raylib (fetched, built from source)
set(RAYLIB_VERSION fafa2cea97d0cb379d832086ba45a989fa6f9c8e)
FetchContent_Declare(
    raylib
    URL https://github.com/raysan5/raylib/archive/${RAYLIB_VERSION}.tar.gz
    URL_HASH SHA256=596658422add8aeec5d3047ff562eda5dd635885162e29be61013473107e9a22
)

# raygui (fetched, header only)
set(RAYGUI_VERSION 580961b63be9afa76cf78b7d1ad090768108b5ca)
FetchContent_Declare(
    raygui
    URL https://github.com/raysan5/raygui/archive/${RAYGUI_VERSION}.tar.gz
    URL_HASH SHA256=08192f6bec80408310709356f66f1cfd40979bf84ead96dfc9245f338860e839
)

# CLAP (fetched, header only)
set(CLAP_VERSION a47f6badb49d948fd009998f28309cdab78979c9)
FetchContent_Declare(
    clap
    URL https://github.com/free-audio/clap/archive/${CLAP_VERSION}.tar.gz
    URL_HASH SHA256=515a92c8f0f82ed5078721a878b209eddfbba862ad4dbc2991af22e2963d4786
)

# libxml2 (fetched, built from source)
set(LIBXML2_VERSION 2.15.2)
FetchContent_Declare(
    libxml2
    URL https://gitlab.gnome.org/GNOME/libxml2/-/archive/v${LIBXML2_VERSION}/libxml2-v${LIBXML2_VERSION}.tar.gz
    URL_HASH SHA256=0380419e8edcafe8425cb39a84cda3a477672ca66793b15545394eb0e4f6f47f
)

# stb_truetype is vendored in third_party/ (single header, implementation
# compiled into exactly one translation unit, no build system needed)

if(BUILD_STANDALONE)
    # raygui is header only and ships no CMake project, so provide the
    # INTERFACE target that FetchContent_MakeAvailable would not create
    add_library(raygui INTERFACE)
endif()

set(CORE_SRCS
    src/core/synth.c
    src/core/effects.c
)

if(BUILD_STANDALONE)
    add_executable(synth ${CORE_SRCS}
        src/standalone/main.c
        src/standalone/audio_thread.c
        src/standalone/interface.c
        src/standalone/keyboard.c
        src/standalone/midi.c
        src/standalone/record.c
        src/standalone/xml.c
    )
    set_target_properties(synth PROPERTIES C_STANDARD 17)
    target_include_directories(synth PRIVATE include assets)

    if(WIN32)
        target_compile_definitions(synth PRIVATE __WINDOWS__ _WIN32_WINNT=0x0601)
    elseif(APPLE)
        target_compile_definitions(synth PRIVATE __MACOS__)
    else()
        target_compile_definitions(synth PRIVATE __LINUX__)
    endif()
endif()

if(BUILD_CLAP_PLUGIN)
    add_library(synth_clap SHARED ${CORE_SRCS}
        src/clap/clap_entry.c
        src/clap/clap_factory.c
        src/clap/clap_plugin.c
        src/clap/clap_params.c
        src/clap/clap_audio_ports.c
        src/clap/clap_note_ports.c
        src/clap/gui/clap_gui.c
        src/clap/gui/clap_gui_linux.c
        src/clap/gui/clap_gui_windows.c
    )
    set_target_properties(synth_clap PROPERTIES
        OUTPUT_NAME synth
        PREFIX ""
        SUFFIX ".clap"
        C_STANDARD 17
    )
    target_include_directories(synth_clap PRIVATE include assets third_party)
    target_compile_definitions(synth_clap PRIVATE __CLAP__)
endif()

# Populate and build the dependencies that the selected targets need.
# raylib and libxml2 are compiled here; raygui and clap are header only
# (clap ships its own INTERFACE target).
set(fetch_deps)
if(BUILD_STANDALONE)
    list(APPEND fetch_deps raylib raygui libxml2)
endif()
if(BUILD_CLAP_PLUGIN)
    list(APPEND fetch_deps clap)
endif()

if(BUILD_STANDALONE)
    set(LIBXML2_WITH_PROGRAMS OFF CACHE BOOL "" FORCE)
    set(LIBXML2_WITH_TESTS OFF CACHE BOOL "" FORCE)
    # iconv is not always available (e.g. minimal mingw-w64 toolchains),
    # libxml2 falls back to its internal ISO8859 converter when it is missing
    find_package(Iconv QUIET)
    set(LIBXML2_WITH_ICONV ${Iconv_FOUND} CACHE BOOL "" FORCE)
endif()

FetchContent_MakeAvailable(${fetch_deps})

if(BUILD_STANDALONE)
    target_include_directories(raygui INTERFACE ${raygui_SOURCE_DIR}/src)
endif()

if(BUILD_STANDALONE)
    target_link_libraries(synth PRIVATE raylib raygui LibXml2)
    if(WIN32)
        target_link_libraries(synth PRIVATE m ksuser winmm gdi32 opengl32 ole32 uuid shell32 ws2_32)
    elseif(APPLE)
        target_link_libraries(synth PRIVATE m)
    else()
        find_package(Threads REQUIRED)
        target_link_libraries(synth PRIVATE asound X11 m Threads::Threads)
    endif()
endif()

if(BUILD_CLAP_PLUGIN)
    target_link_libraries(synth_clap PRIVATE clap)
    if(WIN32)
        target_link_libraries(synth_clap PRIVATE gdi32 user32 m)
    elseif(APPLE)
        target_link_libraries(synth_clap PRIVATE "-framework Cocoa" m)
    else()
        target_link_libraries(synth_clap PRIVATE X11 m)
    endif()
endif()
