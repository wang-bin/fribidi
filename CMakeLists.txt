cmake_minimum_required(VERSION 3.1)

PROJECT(fribidi)
message(STATUS "TARGET OS: ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_VERSION} ${CMAKE_SYSTEM_PROCESSOR}")
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/tools.cmake)
  set(TOOLS_CMAKE ON)
  include(tools.cmake)
endif()
INCLUDE(CheckIncludeFile)
INCLUDE(CheckTypeSize)

SET(VERSION_MAJOR 0)
SET(VERSION_MINOR 9)
SET(VERSION_MICRO 7)
set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_MICRO})
set(SO_VERSION ${VERSION_MAJOR})

OPTION(USE_FRIBIDI_CHARSETS "charset conversion routines" ON)

SET(PACKAGE "${PROJECT_NAME}")
SET(PACKAGE_NAME "GNU Fribidi")
SET(PACKAGE_BUGREPORT "http://bugs.freedesktop.org/enter_bug.cgi?product=fribidi")
SET(FRIBIDI_VERSION "${PROJECT_VERSION}")
SET(FRIBIDI_MAJOR_VERSION "${VERSION_MAJOR}")
SET(FRIBIDI_MINOR_VERSION "${VERSION_MINOR}")
SET(FRIBIDI_MICRO_VERSION "${VERSION_MICRO}")
SET(FRIBIDI_INTERFACE_VERSION 3)
CHECK_TYPE_SIZE(int SIZEOF_INT) #TODO: 0 for apple multi arch

IF(USE_FRIBIDI_CHARSETS)
  SET(FRIBIDI_CHARSETS 1)
ELSE()
  SET(FRIBIDI_CHARSETS 0)
ENDIF()
SET(FRIBIDI_USE_GLIB 0)

CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/lib/fribidi-config.h.in"
    "${PROJECT_BINARY_DIR}/fribidi-config.h")

INCLUDE_DIRECTORIES(
  ${PROJECT_SOURCE_DIR}/charset
  ${PROJECT_SOURCE_DIR}/lib
  ${PROJECT_SOURCE_DIR}/gen.tab
  ${PROJECT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
)

ADD_DEFINITIONS(-DHAVE_CONFIG_H)

ADD_SUBDIRECTORY(lib)
IF(USE_FRIBIDI_CHARSETS)
  ADD_SUBDIRECTORY(charset)
  ADD_DEPENDENCIES(fribidi_shared fribidi-charset)
  ADD_DEPENDENCIES(fribidi_static fribidi-charset)
ENDIF()
IF(CMAKE_CROSSCOMPILING)
  IF(NOT EXISTS "${PROJECT_SOURCE_DIR}/gen.tab/fribidi-unicode-version.h")
    MESSAGE(WARNING "Run host build to generate intermedia files first. 'make gen_tab' or 'ninja gen_tab'")
  ENDIF()
ELSE()
  ADD_SUBDIRECTORY(gen.tab)
  IF(USE_FRIBIDI_CHARSETS)
    ADD_DEPENDENCIES(fribidi-charset gen_tab)
  ENDIF()
  ADD_DEPENDENCIES(fribidi_shared gen_tab)
  ADD_DEPENDENCIES(fribidi_static gen_tab)
ENDIF()

# Needed for fribidi.pc.in
SET(prefix ${CMAKE_INSTALL_PREFIX})
SET(exec_prefix "\${prefix}")
SET(libdir "\${exec_prefix}/lib${LIB_SUFFIX}")
SET(bindir "\${exec_prefix}/bin")
SET(includedir "\${prefix}/include")
SET(PACKAGE_VERSION "${PROJECT_VERSION}")
SET(VERSION "${PROJECT_VERSION}")

CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/${PROJECT_NAME}.pc.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}.pc"
    @ONLY)


CHECK_INCLUDE_FILE(asm/page.h HAVE_ASM_PAGE_H) #var is not set if not found
CHECK_INCLUDE_FILE(dlfcn.h HAVE_DLFCN_H)
CHECK_INCLUDE_FILE(inttypes.h HAVE_INTTYPES_H)
#HAVE_SYS_TYPES_H HAVE_UNISTD_H HAVE_SYS_STAT_H HAVE_WCHAR_H RETSIGTYPE SIZEOF_SHORT memmove memset strdup #not used
CHECK_INCLUDE_FILE(memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)
CHECK_INCLUDE_FILE(stdlib.h HAVE_STDLIB_H)
SET(HAVE_STRINGIZE 1)
CHECK_INCLUDE_FILE(string.h HAVE_STRING_H)
SET(SIZEOF_VOID_P "sizeof(void*)") #not used as a '#if' macro, so sizeof() is fine
CHECK_TYPE_SIZE(wchar_t SIZEOF_WCHAR_T)
SET(STDC_HEADERS 1) #TODO

FOREACH(VAR HAVE_ASM_PAGE_H HAVE_DLFCN_H HAVE_INTTYPES_H HAVE_MEMORY_H HAVE_STDINT_H HAVE_STDLIB_H HAVE_STRINGIZE SIZEOF_VOID_P HAVE_STRING_H SIZEOF_WCHAR_T STDC_HEADERS)
  IF(${VAR})
    SET(CONF "${CONF}\n#define ${VAR} ${${VAR}}")
  ENDIF()
ENDFOREACH()
# why no #ifndef CONFIG_H?
file(WRITE "${PROJECT_BINARY_DIR}/config.h" "${CONF}")

INSTALL(EXPORT ${PROJECT_NAME} DESTINATION lib/cmake)