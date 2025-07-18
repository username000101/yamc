cmake_minimum_required(VERSION 3.30)

# Requires C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Pthread
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads)

# Google Test
if(MSVC)
  set(GTEST_DEBUG_POSTFIX d)
  set(GTEST_FORCE_SHARED_CRT ON)
endif()
include(ExternalProject)
set(EXTERNAL_INSTALL_LOCATION ${CMAKE_BINARY_DIR}/external)
ExternalProject_Add(googletest
  GIT_REPOSITORY https://github.com/google/googletest
  GIT_TAG release-1.11.0
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${EXTERNAL_INSTALL_LOCATION}
             -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
             -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
             -Dgtest_force_shared_crt=${GTEST_FORCE_SHARED_CRT}
)

include_directories(${EXTERNAL_INSTALL_LOCATION}/include)
link_directories(${EXTERNAL_INSTALL_LOCATION}/lib)

# dump type information
add_executable(dump_sizeof dump_typeinfo.cpp)
add_executable(dump_layout dump_typeinfo.cpp)
set_target_properties(dump_sizeof PROPERTIES COMPILE_DEFINITIONS "DUMP_SIZEOF")
set_target_properties(dump_layout PROPERTIES COMPILE_DEFINITIONS "DUMP_STANDARD_LAYOUT")

# performance test
add_executable(perf_rwlock perf_rwlock.cpp)
target_link_libraries(perf_rwlock Threads::Threads)

# Unit tests
add_executable(compile_test compile_test.cpp)
target_link_libraries(compile_test Threads::Threads)
add_test(compile compile_test)

macro (do_test name srcfile)
  set(testname ${name}_test)
  add_executable(${testname} ${srcfile}.cpp)
  add_dependencies(${testname} googletest)
  target_link_libraries(${testname}
    Threads::Threads
    debug gtest${GTEST_DEBUG_POSTFIX} optimized gtest
    debug gtest_main${GTEST_DEBUG_POSTFIX} optimized gtest_main)
  add_test(${name} ${testname})
endmacro (do_test)

do_test(basic basic_test)
do_test(spinlock spinlock_test)
do_test(checked0 checked_test)
do_test(checked1 checked_test)
set_target_properties(checked0_test PROPERTIES COMPILE_DEFINITIONS "YAMC_CHECKED_CALL_ABORT=0")
set_target_properties(checked1_test PROPERTIES COMPILE_DEFINITIONS "YAMC_CHECKED_CALL_ABORT=1")
do_test(deadlock0 deadlock_test)
do_test(deadlock1 deadlock_test)
set_target_properties(deadlock0_test PROPERTIES COMPILE_DEFINITIONS "YAMC_CHECKED_CALL_ABORT=0")
set_target_properties(deadlock1_test PROPERTIES COMPILE_DEFINITIONS "YAMC_CHECKED_CALL_ABORT=1")
do_test(fairness fairness_test)
do_test(rwlock rwlock_test)
do_test(lock lock_test)
do_test(semaphore semaphore_test)
do_test(latch latch_test)
do_test(barrier barrier_test)
