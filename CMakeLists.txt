cmake_minimum_required(VERSION 4.0)

set(ORG coin-or)
set(NAME Clp)
set(VERSION "1.17.11")
set(EXT tar.gz)
set(SHA256 2c078e174dc1a7a308e091b6256fb34b4017897fc140ea707ba207b2913ea46d)
set(TAG_PREFIX releases/)

project("cmeel-${NAME}" VERSION "${VERSION}")

include(FindPkgConfig)
pkg_check_modules(CoinUtils REQUIRED coinutils)

if(DEFINED ENV{CMEEL_JOBS})
  set(CMEEL_JOBS
      $ENV{CMEEL_JOBS}
      CACHE STRING "number of build jobs" FORCE)
else()
  set(CMEEL_JOBS
      4
      CACHE STRING "number of build jobs")
endif()

string(REPLACE ";" " " CoinUtils_LDFLAGS_STR "${CoinUtils_LDFLAGS}")

if(APPLE)
  set(ORIGIN "@loader_path")
else()
  set(ORIGIN "\$\$ORIGIN")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://github.com/${ORG}/${NAME}/archive/refs/tags/${TAG_PREFIX}${VERSION}.${EXT}"
  URL_HASH "SHA256=${SHA256}"
  DOWNLOAD_EXTRACT_TIMESTAMP OFF
  BUILD_IN_SOURCE ON
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} "-E" "env" "LDFLAGS=-Wl,-rpath,'${ORIGIN}/../lib'" "--"
    "./configure" "--prefix=${CMAKE_INSTALL_PREFIX}"
    "--with-coinutils-lib=${CoinUtils_LDFLAGS_STR}"
    "--with-coinutils-incdir=${CoinUtils_INCLUDE_DIRS}"
  BUILD_COMMAND "make" "-j${CMEEL_JOBS}"
  TEST_COMMAND "make" "check"
  INSTALL_COMMAND "make" "install" "-j1"
  COMMAND
    "${CMAKE_COMMAND}" "-E" "rm" "${CMAKE_INSTALL_PREFIX}/lib/lib${NAME}.la"
    "${CMAKE_INSTALL_PREFIX}/lib/lib${NAME}Solver.la")

# dummy file for install target
install(FILES "README.md" DESTINATION "share/cmeel-${NAME}/")
