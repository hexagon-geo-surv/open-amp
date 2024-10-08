
set_property (GLOBAL PROPERTY "PROJECT_LIB_EXTRA_CFLAGS")

collector_create (PROJECT_LIB_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}")
collect (PROJECT_LIB_DIRS "${CMAKE_CURRENT_BINARY_DIR}")
collect (PROJECT_INC_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/include")
collect (PROJECT_LIB_SOURCES version.c)


add_subdirectory (virtio)
add_subdirectory (rpmsg)
add_subdirectory (remoteproc)
add_subdirectory (utils)
if (WITH_VIRTIO_MMIO_DRV)
add_subdirectory (virtio_mmio)
endif (WITH_VIRTIO_MMIO_DRV)

if (WITH_PROXY)
  add_subdirectory (proxy)
  add_subdirectory (service/rpmsg/rpc)
endif (WITH_PROXY)

set (OPENAMP_LIB open_amp)

configure_file(version.h.in ${PROJECT_BINARY_DIR}/include/generated/openamp/version_def.h)
collect (PROJECT_INC_DIRS " ${PROJECT_BINARY_DIR}/include/generated/openamp")
collect (PROJECT_INC_DIRS " ${PROJECT_BINARY_DIR}/include/internal")

if (NOT CMAKE_INSTALL_LIBDIR)
	set (CMAKE_INSTALL_LIBDIR "lib")
endif (NOT CMAKE_INSTALL_LIBDIR)

collector_list (_include PROJECT_INC_DIRS)
include_directories (${_include})

collector_list (_deps PROJECT_LIB_DEPS)

get_property (_ecflags GLOBAL PROPERTY "PROJECT_LIB_EXTRA_CFLAGS")

collector_list (_sources PROJECT_LIB_SOURCES)
set_property (SOURCE ${_sources}
  APPEND_STRING PROPERTY COMPILE_FLAGS " ${_ecflags}")

# Build a shared library if so configured.
if (WITH_ZEPHYR)
  zephyr_library_named(${OPENAMP_LIB})
  add_dependencies(${ZEPHYR_CURRENT_LIBRARY} ${OFFSETS_H_TARGET})
  zephyr_library_sources(${_sources})
  zephyr_include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
else (WITH_ZEPHYR)
  if (WITH_SHARED_LIB)
    set (_lib ${OPENAMP_LIB}-shared)
    add_library (${_lib} SHARED ${_sources})
    target_link_libraries (${_lib} ${_deps})
    install (TARGETS ${_lib} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
    set_target_properties (${_lib} PROPERTIES
      OUTPUT_NAME       "${OPENAMP_LIB}"
      VERSION           "${PROJECT_VERSION}"
      SOVERSION         "${PROJECT_VERSION_MAJOR}"
    )
  endif (WITH_SHARED_LIB)

  if (WITH_STATIC_LIB)
    set (_lib ${OPENAMP_LIB}-static)
    add_library (${_lib} STATIC ${_sources})
    target_include_directories(${_lib} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
    install (TARGETS ${_lib} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
    set_target_properties (${_lib} PROPERTIES
      OUTPUT_NAME       "${OPENAMP_LIB}"
    )
  endif (WITH_STATIC_LIB)
endif (WITH_ZEPHYR)

install (DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/openamp" DESTINATION include)
install (DIRECTORY "${PROJECT_BINARY_DIR}/include/generated/openamp" DESTINATION include)
