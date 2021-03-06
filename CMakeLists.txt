cmake_minimum_required(VERSION 2.6)
project(avhttp)
#SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

OPTION(ENABLE_OPENSSL "Enable use of OpenSSL" ON)

find_package(Boost 1.49  REQUIRED COMPONENTS locale date_time thread filesystem system program_options regex)
find_package(Threads)

find_package(ZLIB REQUIRED)

add_definitions(-DAVHTTP_ENABLE_ZLIB)

if (ENABLE_OPENSSL)
	find_package(OpenSSL)
	add_definitions(-DAVHTTP_ENABLE_OPENSSL)
endif()

if (UNIX AND NOT APPLE AND DEBUG)
	add_definitions(-DDEBUG)
endif()

include_directories(${Boost_INCLUDE_DIRS})
include_directories(include)

add_library(libavhttp test/avhttp_include.cpp)

set_target_properties(libavhttp
		PROPERTIES
		OUTPUT_NAME avhttp
		PREFIX "lib"
		CLEAN_DIRECT_OUTPUT 1)

add_executable(avhttp example/multi_download.cpp)
add_executable(form example/form.cpp)

target_link_libraries(libavhttp ${ZLIB_LIBRARIES})

target_link_libraries(libavhttp ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES} ${CMAKE_DL_LIBS})

if (WIN32)
	add_definitions(-D_WIN32_WINNT=0x0501 -DWIN32_LEAN_AND_MEAN -DBOOST_THREAD_USE_LIB)
	target_link_libraries(libavhttp ws2_32)
endif()

if (UNIX AND NOT APPLE)
	target_link_libraries(libavhttp rt)
endif()

target_link_libraries(avhttp libavhttp)
target_link_libraries(form libavhttp)
