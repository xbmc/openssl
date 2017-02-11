cmake_minimum_required(VERSION 3.0)

project(openssl VERSION 1.0.2 LANGUAGES C)

add_library(openssl INTERFACE
)

if(CMAKE_SIZEOF_VOID_P EQUAL 4)
  message("32-bit")
  add_custom_target(openssl_build ALL
    COMMAND perl Configure VC-WIN32 enable-static-engine --prefix=${CMAKE_INSTALL_PREFIX}
    COMMAND cmd /c ${CMAKE_CURRENT_SOURCE_DIR}/ms/do_nasm.bat
    COMMAND nmake -f ${CMAKE_CURRENT_SOURCE_DIR}/ms/ntdll.mak
    WORKING_DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}
  )
else()
  add_custom_target(openssl_build ALL
    COMMAND perl Configure VC-WIN64A enable-static-engine --prefix=${CMAKE_INSTALL_PREFIX}
    COMMAND cmd /c ms/do_win64a.bat
    COMMAND nmake -f ms/ntdll.mak
    WORKING_DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}
  )
endif()

add_dependencies(openssl openssl_build)
# target_link_libraries(openssl INTERFACE openssl_build)

target_include_directories(openssl
  INTERFACE
  $<INSTALL_INTERFACE:include/openssl>
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS openssl EXPORT openssl
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

install(DIRECTORY
  ${CMAKE_CURRENT_SOURCE_DIR}/inc32/openssl
  DESTINATION include
)

install(EXPORT openssl
  FILE
  ${PROJECT_NAME}.cmake
  NAMESPACE
    ${PROJECT_NAME}::
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)

install(FILES
  cmake/${PROJECT_NAME}-config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)

install(FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/ssleay32.dll
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/libeay32.dll
  DESTINATION
  bin
)

install(FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/ssleay32.lib
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/ssleay32.pdb
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/libeay32.lib
  ${CMAKE_CURRENT_SOURCE_DIR}/out32dll/libeay32.pdb
  DESTINATION
  lib
)
export(TARGETS openssl
  FILE
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
  NAMESPACE ${PROJECT_NAME}::
)

if(MSVC AND BUILD_SHARED_LIBS)
  install(FILES
    $<TARGET_PDB_FILE:${PROJECT_NAME}>
    DESTINATION lib
  )
endif()