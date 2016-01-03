

AUX_SOURCE_DIRECTORY(. SRC_LIST)
ADD_LIBRARY(fribidi-charset OBJECT ${SRC_LIST})

INSTALL(FILES fribidi-char-sets.h fribidi-char-sets-list.h DESTINATION include)
