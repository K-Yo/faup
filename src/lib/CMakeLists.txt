include_directories(AFTER "include/" ${FAUP_INCLUDE_DIRS})


if(WIN32)
	set(WIN32_PORTABLE
	compat/bsd-asprintf.c
	compat/bsd-snprintf.c
	compat/inet_pton.c
	compat/memrchr.c
	)
else()
	set(WIN32_PORTABLE)
endif(WIN32)

if(WEBSERVER)
	set(WEBSERVER_SRC
	civetweb/md5.inl
	civetweb/civetweb.c
	webserver.c
	)
else()
	set(WEBSERVER_SRC)
endif(WEBSERVER)


if(${LUA_FOUND})

set(LUA_MODULES_SRC_FILES
modules.c
)
set(LUA_MODULES_HDR_FILES
include/faup/modules.h
)

include_directories(AFTER ${LUA_INCLUDE_DIRS})

set(FAUP_LINK_LIBRARIES
${FAUP_LINK_LIBRARIES}
${LUA_LDFLAGS}
)

endif()

set(MINIZ_SOURCE miniz/miniz.c miniz/miniz_zip.c miniz/miniz_tinfl.c miniz/miniz_tdef.c)
include_directories(AFTER "miniz/" ${FAUP_INCLUDE_DIRS})

set(FAUP_LIB_SRC_FILES
${WIN32_PORTABLE}
${WEBSERVER_SRC}
${LUA_MODULES_SRC_FILES}
${MINIZ_SOURCE}
b64/cencode.c
b64/cdecode.c
hash/hash.c
hash/htable.c
datadir.c
decode.c
features.c
faup.c
options.c
output.c
parson.c
portable.c
snapshot.c
snapshot-file.c
tld.c
tld-tree.c
utils.c
)

set(FAUP_LIB_HDR_FILES
${LUA_MODULES_HDR_FILES}
include/faup/compat.h
include/faup/datadir.h
include/faup/decode.h
include/faup/errors.h
include/faup/faup.h
include/faup/features.h
include/faup/handler.h
include/faup/options.h
include/faup/output.h
include/faup/parson.h
include/faup/portable.h
include/faup/return-codes.h
include/faup/scheme-codes.h
include/faup/snapshot.h
include/faup/snapshot-file.h
include/faup/tld.h
include/faup/tld-tree.h
include/faup/urllengths.h
include/faup/utils.h
include/faup/webserver.h
)

# Our static library
add_library(faup_static ${FAUP_LIB_SRC_FILES})
set_target_properties(faup_static PROPERTIES SOVERSION 1)
target_link_libraries(faup_static ${FAUP_LINK_LIBRARIES})

# Our dynamic library
add_library(faupl SHARED ${FAUP_LIB_SRC_FILES})
set_target_properties(faupl PROPERTIES SOVERSION 1)
target_link_libraries(faupl ${FAUP_LINK_LIBRARIES})

if(NOT WIN32)
install(TARGETS faupl LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR} COMPONENT Libraries)
install(FILES ${FAUP_LIB_HDR_FILES} DESTINATION include/faup COMPONENT Headers)
endif()

add_subdirectory(modules)

