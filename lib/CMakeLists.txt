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

FILE(READ fribidi.def DEF_CONTENT)
IF(USE_FRIBIDI_CHARSETS)
  SET(DEF_CONTENT "fribidi_parse_charset\nfribidi_unicode_to_charset\nfribidi_charset_to_unicode\n${DEF_CONTENT}")
  FILE(WRITE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def" "EXPORTS\n${DEF_CONTENT}")
# $<TARGET_OBJECTS:xxx> can only be used in add_library & add_executable, not in set_target_properties(...link_libraries...)
  ADD_LIBRARY(fribidi_shared SHARED ${FRIBIDI_SRC} $<TARGET_OBJECTS:fribidi-charset> ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def)
  ADD_LIBRARY(fribidi_static STATIC ${FRIBIDI_SRC} $<TARGET_OBJECTS:fribidi-charset>)
ELSE()
  FILE(WRITE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def" "EXPORTS\n${DEF_CONTENT}")
  ADD_LIBRARY(fribidi_shared SHARED ${FRIBIDI_SRC} ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def)
  ADD_LIBRARY(fribidi_static STATIC ${FRIBIDI_SRC})
ENDIF()
SET_TARGET_PROPERTIES(fribidi_shared PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${SO_VERSION}
  OUTPUT_NAME "fribidi"
  CLEAN_DIRECT_OUTPUT 1
)
#static lib use name libxxx.lib, shared lib use xxx.lib, like ucrt does
IF(MSVC)
  SET_TARGET_PROPERTIES(fribidi_static PROPERTIES OUTPUT_NAME "lib${PROJECT_NAME}")
ELSE()
  SET_TARGET_PROPERTIES(fribidi_static PROPERTIES OUTPUT_NAME "${PROJECT_NAME}")
ENDIF()
SET_TARGET_PROPERTIES(fribidi_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

file(GLOB HEADERS "fribidi*.h" "${PROJECT_SOURCE_DIR}/gen.tab/fribidi-*.h" "${PROJECT_BINARY_DIR}/fribidi-*.h")
FILE(GLOB H "${PROJECT_SOURCE_DIR}/gen.tab/fribidi-*.h")
INSTALL(FILES ${HEADERS} DESTINATION include/${PROJECT_NAME})
INSTALL(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def DESTINATION lib)
INSTALL(TARGETS fribidi_shared fribidi_static EXPORT ${PROJECT_NAME}
  RUNTIME DESTINATION bin # *.dll
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)
INSTALL(EXPORT ${PROJECT_NAME} DESTINATION lib/cmake)