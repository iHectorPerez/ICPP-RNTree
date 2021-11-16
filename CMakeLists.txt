cmake_minimum_required(VERSION 2.4)

project("KVell_NVM_Indexes" DESCRIPTION "Some NVM aware indexes to be integrated into KVell...")

if (NOT CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
	message(STATUS "This project has a top-level one called [${CMAKE_PROJECT_NAME}]")
else()
	message(STATUS "This project is a top-level one")
endif()

#set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS} -DPERF_LATENCY -DUSE_NVM_MALLOC -DNO_CONCURRENT -DNDEBUG -g -O2 -mrtm")
#set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}  -DUSE_NVM_MALLOC -DCLEAR_NVM_POOL -g -O2 -mrtm")
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}  -DCLEAR_NVM_POOL -g -O2 -mrtm")

link_directories("${PROJECT_SOURCE_DIR}/third-party-lib")
link_libraries("${PROJECT_SOURCE_DIR}/third-party-lib/libtbb.so.2")
link_directories(/usr/lib/x86_64-linux-gnu/)
#link_libraries(boost_system boost_thread pthread gtest)
link_libraries(boost_system boost_thread pthread)

include_directories("third-party-lib")
include_directories("include")
#include_directories("test")

add_library(${PROJECT_NAME} STATIC)

target_sources(${PROJECT_NAME}
		PRIVATE
		include/fptree.h
		include/threadinfo.h
		include/index.h
		include/util.h
		include/timer.h
		include/pmalloc_wrap.h
		include/nvm_mgr.h
		src/threadinfo.cpp
		src/nvm_mgr.cpp
		src/util.cpp
		)

target_include_directories(${PROJECT_NAME}
		PRIVATE
		# where the library itself will look for its internal headers
		${CMAKE_CURRENT_SOURCE_DIR}/src
		PUBLIC
		# where top-level project will look for the library's public headers
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
		# where external projects will look for the library's public headers
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
		)

# without it public headers won't get installed
set(public_headers
		include/fptree.h
		include/fptree.h
		include/threadinfo.h
		include/index.h
		include/util.h
		include/timer.h
		include/pmalloc_wrap.h
		include/nvm_mgr.h
		)
# note that ${public_headers} has to be in quotes
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${public_headers}")

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "d")


# definitions of CMAKE_INSTALL_LIBDIR, CMAKE_INSTALL_INCLUDEDIR and others
include(GNUInstallDirs)

# install the target and create export-set
install(TARGETS ${PROJECT_NAME}
		EXPORT "${PROJECT_NAME}Targets"
		# these get default values from GNUInstallDirs, no need to set them
		#RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} # bin
		#LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # lib
		#ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # lib
		# except for public headers, as we want them to be inside a library folder
		PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME} # include/SomeLibrary
		INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} # include
		)

install(EXPORT "${PROJECT_NAME}Targets"
		FILE "${PROJECT_NAME}Targets.cmake"
		NAMESPACE ${namespace}::
		DESTINATION cmake
		)

set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)



add_executable(benchmark
		src/main.cpp
		src/util.cpp
		src/threadinfo.cpp
		src/nvm_mgr.cpp
		)

add_executable(recover_test
	src/recover_test.cpp
	src/util.cpp
	src/threadinfo.cpp
	src/nvm_mgr.cpp
	)

#add_executable(unittest
#    test/unittest.cpp
#	src/util.cpp
#	src/threadinfo.cpp
#	src/nvm_mgr.cpp
#)

#add_executable(datagen src/data_generator.cpp)
#add_executable(test_timer benchmarks/testtimer.cpp)
