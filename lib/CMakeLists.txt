aux_source_directory(. FRIBIDI_SRC)
list(APPEND FRIBIDI_SRC
    arabic-misc.tab.i
)

file(READ fribidi.def DEF_CONTENT)
file(WRITE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def" "EXPORTS\n${DEF_CONTENT}")
add_library(fribidi_shared SHARED ${FRIBIDI_SRC} ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def)
add_library(fribidi_static STATIC ${FRIBIDI_SRC})

# TODO:
file(GLOB HEADERS "fribidi*.h" "${PROJECT_SOURCE_DIR}/gen.tab/fribidi-*.h" "${PROJECT_BINARY_DIR}/fribidi-*.h")

set_target_properties(fribidi_shared PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${SO_VERSION}
  OUTPUT_NAME "fribidi"
  CLEAN_DIRECT_OUTPUT 1
  PUBLIC_HEADER "${HEADERS}"
  INTERPROCEDURAL_OPTIMIZATION 1
)
#static lib use name libxxx.lib, shared lib use xxx.lib, like ucrt does
if(MSVC)
  set_target_properties(fribidi_static PROPERTIES OUTPUT_NAME "lib${PROJECT_NAME}")
else()
  set_target_properties(fribidi_static PROPERTIES OUTPUT_NAME "${PROJECT_NAME}")
endif()
set_target_properties(fribidi_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

if(WIN32)
  install(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.def DESTINATION lib)
endif()
install(TARGETS fribidi_shared fribidi_static
  EXPORT ${PROJECT_NAME}
  RUNTIME DESTINATION bin # *.dll COMPONENT bin
  LIBRARY DESTINATION lib COMPONENT shlib
  ARCHIVE DESTINATION lib COMPONENT dev
  PUBLIC_HEADER DESTINATION include/${PROJECT_NAME} COMPONENT dev
)
