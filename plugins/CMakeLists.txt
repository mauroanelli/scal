cmake_minimum_required(VERSION 2.8)
PROJECT(scald_plugins C)

ADD_DEFINITIONS(-fvisibility=hidden)
IF(APPLE)
	SET(CMAKE_SHARED_MODULE_CREATE_C_FLAGS "${CMAKE_SHARED_MODULE_CREATE_C_FLAGS} -undefined dynamic_lookup")
ENDIF(APPLE)

ADD_LIBRARY(example MODULE example.c)
SET_TARGET_PROPERTIES(example PROPERTIES
	OUTPUT_NAME scapi_example
	PREFIX ""
)

ADD_LIBRARY(json MODULE
	json/plugin.c json/parser.c json/util.c json/object.c json/param.c json/script.c
	json/backend-file.c json/backend-static.c json/backend-shell.c
	json/backend-uci.c json/backend-ubus.c json/backend-system.c
	json/filter-map.c
)
SET_TARGET_PROPERTIES(json PROPERTIES
	OUTPUT_NAME scapi_json
	PREFIX ""
)

FIND_PATH(json_include_dir .h)

INCLUDE(FindPkgConfig)
PKG_SEARCH_MODULE(JSONC json-c)
IF(JSONC_FOUND)
  INCLUDE_DIRECTORIES(${JSONC_INCLUDE_DIRS})
ENDIF()

INCLUDE_DIRECTORIES(${ubus_include_dir})

FIND_LIBRARY(json_library NAMES json-c)
FIND_LIBRARY(blobmsg_json_library NAMES blobmsg_json)
FIND_LIBRARY(ubus_library NAMES ubus)
FIND_LIBRARY(uci_library NAMES uci)
FIND_LIBRARY(json_script_library NAMES json_script)

TARGET_LINK_LIBRARIES(json ${json_library} ${blobmsg_json_library} ${ubus_library} ${json_script_library} ${uci_library})

INSTALL(TARGETS json
	LIBRARY DESTINATION lib/scald
)
