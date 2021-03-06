cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

project(libnova)

# define the name of the library
set(LIBRARY_NAME libnova)

option(BUILD_SHARED_LIBRARY "Set to ON to build libnova as a shared library." ON)
if(BUILD_SHARED_LIBRARY)
    set(BUILD_SHARED_LIBS TRUE)
    add_definitions(-DLIBNOVA_SHARED)
else(BUILD_SHARED_LIBRARY)
    set(BUILD_SHARED_LIBS FALSE)
    add_definitions(-DLIBNOVA_STATIC)
endif(BUILD_SHARED_LIBRARY)

# add some definitions
if(WIN32)
    add_definitions(-D_USE_MATH_DEFINES)
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D__WIN32__)
endif(WIN32)

# add more optimization flags to the visual studio release compilation
if(MSVC)
    set(CMAKE_CXX_FLAGS_RELEASE "/MD /Ox /Ob2 /D NDEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2 /GS-")
endif(MSVC)


# set the debug postfix
set(CMAKE_DEBUG_POSTFIX "d" CACHE STRING "add a postfix, usually d on windows")

# set LIB_POSTFIX
if(UNIX AND NOT WIN32 AND NOT APPLE)
  if(CMAKE_SIZEOF_VOID_P MATCHES "8")
      set(LIB_POSTFIX "64" CACHE STRING "suffix for 32/64 dir placement")
      mark_as_advanced(LIB_POSTFIX)
  endif(CMAKE_SIZEOF_VOID_P MATCHES "8")
endif(UNIX AND NOT WIN32 AND NOT APPLE)
if(NOT DEFINED LIB_POSTFIX)
    set(LIB_POSTFIX "")
endif(NOT DEFINED LIB_POSTFIX)


# set and create the EXECUTABLE_OUTPUT_PATH
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
file(MAKE_DIRECTORY ${EXECUTABLE_OUTPUT_PATH})

# set and create the LIBRARY_OUTPUT_PATH
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
file(MAKE_DIRECTORY ${LIBRARY_OUTPUT_PATH})


# define a macro to setup executable program which are linked with libnova
macro(SETUP_EXECUTABLE EXECUTABLE_NAME)
    add_executable(${EXECUTABLE_NAME} ${EXECUTABLE_NAME}.c)
    target_link_libraries(${EXECUTABLE_NAME} libnova)
    set_target_properties(${EXECUTABLE_NAME} PROPERTIES DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}")
    if(MSVC)
        set_target_properties(${EXECUTABLE_NAME} PROPERTIES PREFIX "../")
    endif(MSVC)
    INSTALL(TARGETS ${EXECUTABLE_NAME} RUNTIME DESTINATION bin)
endmacro(SETUP_EXECUTABLE EXECUTABLE_NAME)

# add include directory to all projects
include_directories(${libnova_SOURCE_DIR}/src)

# traverse the sub directories
add_subdirectory(src)
add_subdirectory(lntest)
add_subdirectory(examples)
