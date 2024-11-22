cmake_minimum_required(VERSION 3.10)
project(MediaProcessor VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 -DNDEBUG")

include(FetchContent)
set(FETCHCONTENT_BASE_DIR "${CMAKE_BINARY_DIR}/_deps")

option(BUILD_TESTING "Build Tests" OFF)
option(RELEASE_BUILD "Produce a packaged release build" OFF)

include(CTest)
include_directories(${CMAKE_SOURCE_DIR}/include)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

find_package(PkgConfig REQUIRED)
if(RELEASE_BUILD)
    set(PKG_CONFIG_USE_STATIC_LIBS TRUE)
    set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static")
else()
    # Default to dynamic linking
    set(PKG_CONFIG_USE_STATIC_LIBS FALSE)
endif()
pkg_check_modules(SNDFILE REQUIRED sndfile)
include_directories(${SNDFILE_INCLUDE_DIRS})

FetchContent_Declare(
  nlohmann_json
  GIT_REPOSITORY https://github.com/nlohmann/json.git
  GIT_TAG v3.11.3
)
FetchContent_MakeAvailable(nlohmann_json)

# Include the cmake files under cmake/
include(cmake/src.cmake)
if(BUILD_TESTING)
    include(cmake/test.cmake)
endif()

# Bundle with ffmpeg for release mode
if(NOT RELEASE_BUILD)
    install(FILES ${CMAKE_SOURCE_DIR}/lib/ffmpeg DESTINATION lib)
endif()

# Package with CPack for release mod
if(NOT RELEASE_BUILD)
    include(CPack)

    set(CPACK_GENERATOR "TGZ")
    set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
    set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
    set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Fully self-contained MediaProcessor alpha release for Linux")
    set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION}-Linux")
    set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/../LICENSE")
    set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/../README.md")

    # Bundle runtime components
    install(TARGETS MediaProcessor RUNTIME DESTINATION bin)
    install(DIRECTORY ${CMAKE_SOURCE_DIR}/lib DESTINATION lib)
    install(FILES ${CMAKE_SOURCE_DIR}/config.json DESTINATION .)

    include(CPack)
endif()
