cmake_minimum_required(VERSION 3.10)

project(simplewebm)

option(SIMPLEWEBM_LIBWEBM "Search for libwebm instead of using its vendored subset" OFF)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(SIMPLEWEBM_SRC
    "${CMAKE_SOURCE_DIR}/src/OpusVorbisDecoder.cpp"
    "${CMAKE_SOURCE_DIR}/src/VPXDecoder.cpp"
    "${CMAKE_SOURCE_DIR}/src/WebMDemuxer.cpp"
)

set(SIMPLEWEBM_INCL
    "${CMAKE_SOURCE_DIR}/src"
)

set(SIMPLEWEBM_DEPS)

find_package(PkgConfig OPTIONAL)

find_package(Opus OPTIONAL)
find_package(Vorbis OPTIONAL)
find_package(libvpx OPTIONAL)

if(NOT Vorbis_FOUND)
    if(PkgConfig_FOUND)
        pkg_check_modules(vorbis REQUIRED IMPORTED_TARGET vorbis)
        add_library(Vorbis::vorbis ALIAS PkgConfig::vorbis)
    else()
        message(FATAL_ERROR "Couldn't find Vorbis")
    endif()
endif()

list(APPEND SIMPLEWEBM_DEPS Vorbis::vorbis)

if(NOT Opus_FOUND)
    if(PkgConfig_FOUND)
        pkg_check_modules(opus REQUIRED IMPORTED_TARGET opus)
        add_library(Opus::opus ALIAS PkgConfig::opus)
    else()
        message(FATAL_ERROR "Couldn't find Opus")
    endif()
endif()

list(APPEND SIMPLEWEBM_DEPS Opus::opus)

if(NOT libvpx_FOUND)
    if(PkgConfig_FOUND)
        pkg_check_modules(vpx REQUIRED IMPORTED_TARGET vpx)
        add_library(libvpx::libvpx ALIAS PkgConfig::vpx)
    else()
        message(FATAL_ERROR "Couldn't find libvpx")
    endif()
endif()

list(APPEND SIMPLEWEBM_DEPS libvpx::libvpx)

if(SIMPLEWEBM_LIBWEBM)
    find_package(webm)

    if(NOT webm_FOUND)
        if(PkgConfig_FOUND)
            pkg_check_modules(webm REQUIRED IMPORTED_TARGET webm)
            add_library(webm::webm ALIAS PkgConfig::webm)
        else()
            message(FATAL_ERROR "Couldn't find libwebm")
        endif()
    endif()

    list(APPEND SIMPLEWEBM_DEPS webm::webm)
else()
    list(APPEND SIMPLEWEBM_INCL "${CMAKE_SOURCE_DIR}/libwebm")
    list(APPEND SIMPLEWEBM_SRC "${CMAKE_SOURCE_DIR}/libwebm/mkvparser/mkvparser.cc")
endif()

add_library(${CMAKE_PROJECT_NAME} ${SIMPLEWEBM_SRC})
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${SIMPLEWEBM_INCL})
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE ${SIMPLEWEBM_DEPS})
