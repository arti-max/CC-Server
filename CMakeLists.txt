cmake_minimum_required(VERSION 3.15)
project(CrossCraftServer C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)

set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(BUILD_CURL_EXE OFF CACHE BOOL "" FORCE)
set(CURL_ENABLE_SSL ON CACHE BOOL "" FORCE)

if(WIN32)
    set(CURL_USE_SCHANNEL ON CACHE BOOL "" FORCE)
else()
    set(CURL_USE_OPENSSL ON CACHE BOOL "" FORCE)
endif()

FetchContent_Declare(
    curl
    URL https://github.com/curl/curl/releases/download/curl-8_7_1/curl-8.7.1.tar.gz
    OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(curl)

find_package(CURL REQUIRED)

include_directories(include)

file(GLOB_RECURSE SOURCES 
    "src/cpp/*.cpp"
    "src/cpp/*.c"
)

add_executable(CrossCraftServer ${SOURCES})

target_link_libraries(CrossCraftServer PRIVATE CURL::libcurl)

if(WIN32)
    target_link_libraries(CrossCraftServer PRIVATE ws2_32 crypt32)
else()
    target_link_libraries(CrossCraftServer PRIVATE pthread)
endif()

set(BIN_OUTPUT_DIR "${CMAKE_BINARY_DIR}/bin")
set_target_properties(CrossCraftServer PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${BIN_OUTPUT_DIR}"
)

add_custom_command(TARGET CrossCraftServer POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${CMAKE_SOURCE_DIR}/README.TXT"
        "${BIN_OUTPUT_DIR}/README.TXT"
    COMMENT "Copying README.TXT to output directory"
)

if(WIN32 AND MINGW)
    target_link_options(CrossCraftServer PRIVATE -static-libgcc -static-libstdc++ -static)
endif()

if(WIN32)
    file(WRITE "${BIN_OUTPUT_DIR}/start_server.bat" "@echo off\r\nstart CrossCraftServer.exe\r\n")
else()
    file(WRITE "${BIN_OUTPUT_DIR}/start_server.sh" "#!/bin/sh\n./CrossCraftServer\n")
    execute_process(COMMAND chmod +x "${BIN_OUTPUT_DIR}/start_server.sh")
endif()