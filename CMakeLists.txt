cmake_minimum_required(VERSION 3.9)

project(fribidi)
message(STATUS "TARGET OS: ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_VERSION} ${CMAKE_SYSTEM_PROCESSOR}")

set(VERSION_MAJOR 1)
set(VERSION_MINOR 0)
set(VERSION_MICRO 12)
set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_MICRO})
set(SO_VERSION ${VERSION_MAJOR})

set(PACKAGE "${PROJECT_NAME}")
set(PACKAGE_NAME "GNU Fribidi")
set(PACKAGE_BUGREPORT "https://github.com/fribidi/fribidi/issues/new")
set(FRIBIDI_VERSION "${PROJECT_VERSION}")
set(FRIBIDI_MAJOR_VERSION "${VERSION_MAJOR}")
set(FRIBIDI_MINOR_VERSION "${VERSION_MINOR}")
set(FRIBIDI_MICRO_VERSION "${VERSION_MICRO}")
set(FRIBIDI_INTERFACE_VERSION 4)

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/tools.cmake)
  set(TOOLS_CMAKE ON)
  include(tools.cmake)
endif()
include(CheckIncludeFile)
include(CheckTypeSize)

if(POLICY CMP0069) # lto
  cmake_policy(SET CMP0069 NEW)
endif()

CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/lib/fribidi-config.h.in"
    "${PROJECT_BINARY_DIR}/fribidi-config.h")

include_directories(
  ${PROJECT_SOURCE_DIR}/lib
  ${PROJECT_SOURCE_DIR}/gen.tab
  ${PROJECT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
)

add_definitions(-DHAVE_CONFIG_H)

add_subdirectory(lib)
if(CMAKE_CROSSCOMPILING)
  if(NOT EXISTS "${PROJECT_SOURCE_DIR}/gen.tab/fribidi-unicode-version.h")
    MESSAGE(WARNING "Run host build to generate intermedia files first. 'make gen_tab' or 'ninja gen_tab'")
  endif()
else()
  add_subdirectory(gen.tab)
  add_dependencies(fribidi_shared gen_tab)
  add_dependencies(fribidi_static gen_tab)
endif()

# Needed for fribidi.pc.in
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix "\${prefix}")
set(libdir "\${exec_prefix}/lib${LIB_SUFFIX}")
set(bindir "\${exec_prefix}/bin")
set(includedir "\${prefix}/include")
set(PACKAGE_VERSION "${PROJECT_VERSION}")
set(VERSION "${PROJECT_VERSION}")

CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/${PROJECT_NAME}.pc.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}.pc"
    @ONLY)


CHECK_INCLUDE_FILE(memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILE(stdlib.h HAVE_STDLIB_H)
set(HAVE_STRINGIZE 1)
CHECK_INCLUDE_FILE(string.h HAVE_STRING_H)
CHECK_TYPE_SIZE(wchar_t SIZEOF_WCHAR_T)
CHECK_INCLUDE_FILE(stddef.h STDC_HEADERS)
CHECK_INCLUDE_FILE(sys/times.h HAVE_SYS_TIMES_H)

FOREACH(VAR HAVE_MEMORY_H HAVE_STDINT_H HAVE_STDLIB_H HAVE_STRINGIZE HAVE_STRING_H STDC_HEADERS)
  if(${VAR})
    set(CONF "${CONF}\n#define ${VAR} ${${VAR}}")
  endif()
ENDFOREACH()
# why no #ifndef CONFIG_H?
file(WRITE "${PROJECT_BINARY_DIR}/config.h" "${CONF}")

install(EXPORT ${PROJECT_NAME} DESTINATION lib/cmake)