auto_sources(IDL_SRCS "*.idl.json" "RECURSE" "${HPHP_HOME}/hphp/system/idl")
if (NOT LINUX)
	list(REMOVE_ITEM IDL_SRCS "${HPHP_HOME}/hphp/system/idl/calendar.idl.json")
endif()

add_custom_command(OUTPUT "class_map.cpp" "constants.h"
                   DEPENDS ${IDL_SRCS} gen-class-map
                   COMMAND "${HPHP_HOME}/hphp/tools/bootstrap/gen-class-map"
                   ARGS "--system"
                        "${CMAKE_CURRENT_SOURCE_DIR}/class_map.cpp"
                        "${CMAKE_CURRENT_SOURCE_DIR}/constants.h"
                        ${IDL_SRCS}
                   COMMENT "Generating class_map.cpp and constants.h")

set(CXX_SOURCES "class_map.cpp" "systemlib.cpp")

add_library(hphp_system STATIC ${CXX_SOURCES})

# Needed to force system/constants.h to generate prior to files which need it
add_dependencies(hphp_runtime_static hphp_system)
add_dependencies(hphp_analysis hphp_system)

FILE(STRINGS "php.txt" SYSTEMLIB_CLASSES)
set(SYSTEMLIB_SRCS)
set(SYSTEMLIB_SRCS_STR)
foreach(cls ${SYSTEMLIB_CLASSES})
	STRING(REGEX REPLACE "[ \t]*#.*" "" cls "${cls}")
	if (NOT "${cls}" STREQUAL "")
		list(APPEND SYSTEMLIB_SRCS "../../${cls}")
                set(SYSTEMLIB_SRCS_STR "${SYSTEMLIB_SRCS_STR} ${cls}")
	endif()
endforeach()

add_custom_target(systemlib
                  DEPENDS "php.txt" ${SYSTEMLIB_SRCS}
                  COMMAND "INSTALL_DIR=${HPHP_HOME}/bin"
                          "FBCODE_DIR=${HPHP_HOME}"
                          "${HPHP_HOME}/hphp/runtime/ext_hhvm/make_systemlib.sh"
                          "--install_dir=${HPHP_HOME}/bin" "--fbcode_dir=${HPHP_HOME}"
                          "${SYSTEMLIB_SRCS_STR}"
                  COMMENT "Generating systemlib.php")
