
cmake_minimum_required(VERSION 3.7 FATAL_ERROR)
project (HPTT C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
option(ENABLE_IBM "ENABLE IBM" OFF)
option(ENABLE_AVX "ENABLE AVX" OFF)
option(ENABLE_ARM "ENABLE ARM" OFF)
option(FINE_TUNE "FINE TUNE for native hardware" OFF)

#if(CMAKE_SYSTEM_PROCESSOR STREQUAL "ppc64le")
#  set(ENABLE_IBM ON)
#endif()

#set(FINE_TUNE OFF)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -qopenmp -xhost)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  if(ENABLE_IBM)
    set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fopenmp)
  else()
    set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fopenmp)
    if(FINE_TUNE)
        set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -march=native -mtune=native)
    else()
        set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -march=x86-64 -mtune=generic)
    endif()
  endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fopenmp -march=native)
    if(FINE_TUNE)
        set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fopenmp -march=native)
    else()
        set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fopenmp)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "PGI")
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -silent -w -Mnovect)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "XL")
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -qsmp=omp)
endif()

if(ENABLE_AVX)
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -mavx -DHPTT_ARCH_AVX)
elseif(ENABLE_ARM)
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -mfpu=neon -DHPTT_ARCH_ARM)
elseif(ENABLE_IBM)
  set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -mtune=native -DHPTT_ARCH_IBM -maltivec -mabi=altivec)
endif()

set(HPTT_CXX_FLAGS ${HPTT_CXX_FLAGS} -fPIC)

set(HPTT_SRCS src/hptt.cpp src/plan.cpp src/transpose.cpp src/utils.cpp)

add_library(hptt STATIC ${HPTT_SRCS})
target_compile_features(hptt PUBLIC cxx_std_11)
target_include_directories(hptt PUBLIC ${PROJECT_SOURCE_DIR}/include)
#target_compile_definitions(hptt PRIVATE ${HPTT_CXX_COMPILE_DEFS})
target_compile_options(hptt PUBLIC ${HPTT_CXX_FLAGS})

message(STATUS, "ENABLE IBM: ${ENABLE_IBM}")
message(STATUS, "ENABLE AVX: ${ENABLE_AVX}")
message(STATUS, "ENABLE ARM: ${ENABLE_ARM}")
message(STATUS, "FINE_TUNE: ${FINE_TUNE}")

install(TARGETS hptt
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

set(HPTT_INCLUDES 
    include/compute_node.h 
    include/hptt_types.h 
    include/hptt.h 
    include/macros.h 
    include/plan.h 
    include/utils.h 
    include/transpose.h)

install(FILES ${HPTT_INCLUDES}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
