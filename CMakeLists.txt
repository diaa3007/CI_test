# Change money test project build

cmake_minimum_required(VERSION 3.1)

project(change)

enable_testing()


#------------------------------ google test setup -------------------------------------

# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)

# Should add c++11 flag but it gives a wierd error in my machine

# Flag for gtest to compile without errors
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")

execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" . 
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
  
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
  
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines the gtest and gtest_main targets.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
                 ${CMAKE_CURRENT_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

# The gtest/gtest_main targets carry header search path dependencies automatically
# when using CMake 2.8.11 or later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()

# -------------------------------------------------------------------------------------

# ------------------------ Building ---------------------------------------

# Soruce variables
set(TEST_SOURCES 
	src/change.cpp 
	test/changeUT.cpp
)

set(EXE_SOURCES 
	src/change.cpp 
	src/main.cpp
)

#Executables and links
add_executable(change "${EXE_SOURCES}")

target_include_directories(change PUBLIC
			"${PROJECT_SOURCE_DIR}/include" )


add_executable(testChange "${TEST_SOURCES}")

target_include_directories(testChange PUBLIC
			"${PROJECT_SOURCE_DIR}/include"		
			"${CMAKE_CURRENT_BINARY_DIR}/googletest-src/googletest/include/gtest")	

target_link_libraries(testChange gtest gtest_main gcov)

# -------------------------------------------------------------------------------------

# --------------------------- Code coverage -------------------------------------------

# You should add_test to be able to use it for code coverage
add_test(NAME ctestChange COMMAND testChange)

if(CMAKE_BUILD_TYPE STREQUAL "Coverage")
	list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/CMakeModules")
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage")	# c++
	SET(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage")	# c

	include(CodeCoverage)
	setup_target_for_coverage(ct_coverage testChange coverage)
endif()

