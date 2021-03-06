cmake_minimum_required (VERSION 3.1)
project (bond)

cmake_policy (SET CMP0022 NEW)

set (CMAKE_MODULE_PATH
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/cmake-modules)

enable_testing()

set (BOND_IDL ${CMAKE_CURRENT_SOURCE_DIR}/idl)
set (BOND_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/cpp/inc)
set (BOND_PYTHON_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/python/inc)
set (BOND_GENERATED ${CMAKE_CURRENT_SOURCE_DIR}/cpp/generated)
set (BOND_COMPAT_TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/test/compat)

include (Config)
include (Bond)
include (NoDebug)
include (Compiler)
include (PythonTest)

add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND} -C ${CMAKE_CFG_INTDIR} --output-on-failure)

if (NOT BOND_GBC_PATH)
    add_subfolder (compiler "compiler")
    if (NOT BOND_SKIP_GBC_TESTS)
        add_dependencies (check gbc-tests)
    endif()
endif()

add_subdirectory (cpp)
add_subfolder (doc "doc")
add_python_subdirectory (python)
add_subdirectory (examples)

install (DIRECTORY
    cpp/inc/bond
    cpp/generated/bond
    python/inc/bond
    thirdparty/rapidjson/include/rapidjson
    DESTINATION include
    PATTERN *.cpp EXCLUDE)

install (EXPORT bond
    DESTINATION lib/bond
    EXPORT_LINK_INTERFACE_LIBRARIES)

# if BOND_GBC_PATH is set we must copy over that gbc to the install location
if (BOND_GBC_PATH)
    install (FILES
        ${BOND_GBC_PATH}
        DESTINATION bin)
endif()
