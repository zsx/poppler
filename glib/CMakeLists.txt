add_definitions(-DG_LOG_DOMAIN=\"Poppler\" ${GTK2_CFLAGS})

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${GTK2_INCLUDE_DIRS}
)
if (GTK2_FOUND)
  include_directories(${GTK2_CAIRO_INCLUDE_DIR})
  set (CAIRO_FEATURE "#define POPPLER_HAS_CAIRO 1")
  add_definitions(${CAIRO_CFLAGS})
else (GTK2_FOUND)
  set (CAIRO_FEATURE "#undef POPPLER_HAS_CAIRO")
endif (GTK2_FOUND)

if (GTK2_GDK_FOUND)
  add_definitions(${GDK2_CFLAGS})
endif (GTK2_GDK_FOUND)

configure_file(poppler-features.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/poppler-features.h @ONLY)

if (GTK2_FOUND)
  add_subdirectory(demo)
endif (GTK2_FOUND)

set(poppler_glib_public_headers
  poppler-action.h
  poppler-date.h
  poppler-document.h
  poppler-page.h
  poppler-attachment.h
  poppler-form-field.h
  poppler-annot.h
  poppler.h
)

find_program(GLIB2_MKENUMS glib-mkenums)

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.h
  COMMAND ${GLIB2_MKENUMS}
  ARGS
    --fhead \"\#ifndef __POPPLER_TYPE_ENUMS_H__\\n\"
    --fhead \"\#define __POPPLER_TYPE_ENUMS_H__ 1\\n\\n\"
    --fhead \"\#include <glib-object.h>\\n\\n\"
    --fhead \"G_BEGIN_DECLS\\n\\n\"
    --ftail \"G_END_DECLS\\n\\n\"
    --ftail \"\#endif /* __POPPLER_TYPE_ENUMS_H__ */\\n\"
    --fprod \"\\n/* --- @filename@ --- */\"
    --eprod \"\#define POPPLER_TYPE_@ENUMSHORT@ \(@enum_name@_get_type\(\)\)\\n\"
    --eprod \"GType @enum_name@_get_type \(void\) G_GNUC_CONST\;\\n\"
    ${poppler_glib_public_headers} > ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.h
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  DEPENDS ${poppler_glib_public_headers}
)

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.c
  COMMAND ${GLIB2_MKENUMS}
  ARGS
    --fhead \"\#include \\"poppler-enums.h\\"\\n\\n\"
    --fprod \"\\n/* enumerations from \\"@filename@\\" */\"
    --fprod \"\\n\#include \\"@filename@\\"\"
    --vhead \"static const G@Type@Value _@enum_name@_values[] = {\"
    --vprod \"  { @VALUENAME@, \\"@VALUENAME@\\", \\"@valuenick@\\" },\"
    --vtail \"  { 0, NULL, NULL }\\n}\;\\n\\n\"
    --vtail \"GType\\n@enum_name@_get_type \(void\)\\n{\\n\"
    --vtail \"  static GType type = 0\;\\n\\n\"
    --vtail \"  if \(G_UNLIKELY \(type == 0\)\)\\n\"
    --vtail \"    type = g_\@type\@_register_static \(\\"@EnumName@\\", _@enum_name@_values\)\;\\n\\n\"
    --vtail \"  return type\;\\n}\\n\\n\"
    ${poppler_glib_public_headers} > ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.c
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  DEPENDS ${poppler_glib_public_headers} ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.h
)


set(poppler_glib_SRCS
  poppler-action.cc
  poppler-date.cc
  poppler-document.cc
  poppler-page.cc
  poppler-attachment.cc
  poppler-form-field.cc
  poppler-annot.cc
  poppler-layer.cc
  poppler.cc
  ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.c
)
if (GTK2_FOUND)
  set(poppler_glib_SRCS ${poppler_glib_SRCS}
    ${CMAKE_SOURCE_DIR}/poppler/CairoFontEngine.cc
    ${CMAKE_SOURCE_DIR}/poppler/CairoOutputDev.cc
  )
endif (GTK2_FOUND)
if(MSVC) 
	add_library(poppler-glib STATIC ${poppler_glib_SRCS})
else(MSVC)
	add_library(poppler-glib SHARED ${poppler_glib_SRCS})
endif(MSVC)
set_target_properties(poppler-glib PROPERTIES VERSION 4.0.0 SOVERSION 4)
target_link_libraries(poppler-glib poppler ${GTK2_LIBRARIES})
if (GTK2_FOUND)
  target_link_libraries(poppler-glib ${CAIRO_LIBRARIES})
endif (GTK2_FOUND)
install(TARGETS poppler-glib RUNTIME DESTINATION bin LIBRARY DESTINATION lib${LIB_SUFFIX} ARCHIVE DESTINATION lib${LIB_SUFFIX})

install(FILES
  ${poppler_glib_public_headers}
  ${CMAKE_CURRENT_BINARY_DIR}/poppler-enums.h
  ${CMAKE_CURRENT_BINARY_DIR}/poppler-features.h
  DESTINATION include/poppler/glib)


if (GTK2_GDK_FOUND)
  set(test_poppler_glib_SRCS
    test-poppler-glib.cc
  )
  poppler_add_test(test-poppler-glib BUILD_GTK_TESTS ${test_poppler_glib_SRCS})
  target_link_libraries(test-poppler-glib poppler-glib ${GTK2_LIBRARIES})
endif (GTK2_GDK_FOUND)

