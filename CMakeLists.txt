cmake_minimum_required(VERSION 3.28)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/.devcontainer/cmake)
include(preproject)
project(spatialite-tools VERSION 5.0.0)
include(flags)
include(GNUInstallDirs)
include(ExternalProject)
set_property(DIRECTORY PROPERTY "EP_BASE" ${CMAKE_BINARY_DIR}/epbase)
ExternalProject_Add(${PROJECT_NAME}
  DOWNLOAD_COMMAND sudo dnf -y install --setopt=tsflags=nodocs epel-release && sudo dnf -y install sqlite-devel libspatialite-devel proj-devel geos-devel freexl-devel readosm-devel libxml2-devel minizip-devel
  SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR} BINARY_DIR ${CMAKE_CURRENT_LIST_DIR}
  CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR>
  #    --disable-proj --disable-freexl --disable-geos --disable-libxml2 --disable-minizip --disable-rttopo --disable-gcp
  )
