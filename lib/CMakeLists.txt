SET(FRIBIDI_SRC
	arabic-misc.tab.i
	fribidi.c
	fribidi-arabic.c
	fribidi-bidi.c
	fribidi-bidi-types.c
	fribidi-deprecated.c
	fribidi-joining.c
	fribidi-joining-types.c
	fribidi-mem.c
	fribidi-mirroring.c
	fribidi-run.c
	fribidi-shape.c
)

INCLUDE_DIRECTORIES(
  ${CMAKE_SOURCE_DIR}/SMP
  ${CMAKE_SOURCE_DIR}/SMP/lib
  ${CMAKE_SOURCE_DIR}/charset
  ${CMAKE_CURRENT_SOURCE_DIR}
)

ADD_LIBRARY(fribidi_shared SHARED ${FRIBIDI_SRC} $<TARGET_OBJECTS:fribidi-charset>)
ADD_LIBRARY(fribidi_static STATIC ${FRIBIDI_SRC} $<TARGET_OBJECTS:fribidi-charset>)
SET_TARGET_PROPERTIES(fribidi_shared
  PROPERTIES VERSION ${PROJECT_VERSION}
  SOVERSION ${SO_VERSION}
)
SET_TARGET_PROPERTIES(fribidi_shared PROPERTIES OUTPUT_NAME "fribidi")
SET_TARGET_PROPERTIES(fribidi_shared PROPERTIES CLEAN_DIRECT_OUTPUT 1)
message(STATUS "TARGET OS: ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_VERSION}")
#static lib use name libxxx.lib, shared lib use xxx.lib, like ucrt does
IF(MSVC)
  SET_TARGET_PROPERTIES(fribidi_static PROPERTIES OUTPUT_NAME "libfribidi")
ELSE()
  SET_TARGET_PROPERTIES(fribidi_static PROPERTIES OUTPUT_NAME "fribidi")
ENDIF()
SET_TARGET_PROPERTIES(fribidi_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

file(GLOB HEADERS "fribidi-*.h" "${CMAKE_SOURCE_DIR}/SMP/lib/fribidi-*.h")
# TODO: dll
INSTALL(FILES ${HEADERS} DESTINATION include)
INSTALL(TARGETS fribidi_shared fribidi_static EXPORT fribidi LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
INSTALL(EXPORT fribidi DESTINATION lib/cmake)