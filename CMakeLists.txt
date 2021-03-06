cmake_minimum_required(VERSION 3.9)
project(coolq-cpp-sdk)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS "/utf-8 ${CMAKE_CXX_FLAGS}")  # use UTF-8 source files
set(CMAKE_CXX_FLAGS "/MP ${CMAKE_CXX_FLAGS}")  # build with object level parallelism

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

include(cotire)
include(FindVcpkgIncludeDir)
include(FixDebugLibraryLookup)

include_directories(${VCPKG_INCLUDE_DIR})
include_directories(src)

add_compile_definitions(BOOST_CONFIG_SUPPRESS_OUTDATED_MESSAGE
                        _WIN32_WINNT=0x0501
                        WIN32_LEAN_AND_MEAN
                        NOMINMAX)

# read app id from app_id.txt
file(READ "app_id.txt" APP_ID)
string(STRIP "${APP_ID}" APP_ID)
set(APP_ID "\"${APP_ID}\"")
add_compile_definitions(APP_ID=${APP_ID})

find_package(unofficial-iconv CONFIG REQUIRED)

include(FixLinkConflict)

file(GLOB_RECURSE SOURCE_FILES src/*.cpp)
set(LIB_NAME "app")
add_library(${LIB_NAME} SHARED ${SOURCE_FILES})

target_link_libraries(${LIB_NAME} PRIVATE unofficial::iconv::libiconv unofficial::iconv::libcharset)

cotire(${LIB_NAME})

add_custom_command(TARGET ${LIB_NAME}
                   POST_BUILD
                   COMMAND
                   powershell -ExecutionPolicy Bypass -NoProfile -File "${PROJECT_SOURCE_DIR}/scripts/post_build.ps1" ${APP_ID} ${LIB_NAME} "$<TARGET_FILE_DIR:${LIB_NAME}>")
