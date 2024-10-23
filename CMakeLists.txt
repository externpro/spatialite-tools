cmake_minimum_required(VERSION 3.28)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/.devcontainer/cmake ${CMAKE_CURRENT_LIST_DIR}/cmake)
include(preproject)
project(spatialite-tools VERSION 5.1.0)
set(xVer a) # letter after CMAKE_PROJECT_VERSION, for example 5.1.0a
include(flags)
include(GNUInstallDirs)
if(TRUE)
  set(targetsFile ${CMAKE_PROJECT_NAME}-targets)
  if(DEFINED XP_NAMESPACE)
    set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
  endif()
  if(NOT DEFINED XP_INSTALL_CMAKEDIR)
    set(XP_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake)
  endif()
  xpPackageDevel(TARGETS_FILE ${targetsFile})
  xpFindPkg(PKGS libiconv libspatialite)
  include(configure)
  set(spatialite_srcs shell.c)
  set(spatialite_deps xpro::libiconv)
  set(exes
    spatialite # a complete CLI front-end, more or less equivalent to the well known sqlite3 front-end, but fully supporting SpatiaLite
    spatialite_tool # a tool supporting import/export of Shapefiles and DBF-files
    shp_doctor # diagnostic tool to check anomalous shapefiles
    exif_loader # an import tool for EXIF or EXIF-GPS (JPEG) pictures
    spatialite_network # a tool creating a VirtualNetwork (supporting shortest path / routing SQL queries)
#   spatialite_gml # a tool importing GML files
    spatialite_convert # a tool converting DB-files from different versions of SpatiaLite
#   spatialite_dxf # an tool importing DXF files
    )
  set(osm-tools # tools supporting OpenStreetMap datasets
    spatialite_osm_raw
    spatialite_osm_filter
    spatialite_osm_map
    spatialite_osm_net
    spatialite_osm_overpass
    )
  set(xml-tools # tools supporting XML files processing
    spatialite_xml_load
    spatialite_xml_collapse
    spatialite_xml_print
    spatialite_xml_validator
    spatialite_xml2utf8
    )
  set(misc-tools
    shp_sanitize
    spatialite_dem
    )
  ########################################
  foreach(exe ${exes})
    if(DEFINED ${exe}_srcs)
      add_executable(${exe} ${${exe}_srcs})
    else()
      add_executable(${exe} ${exe}.c)
    endif()
    target_link_libraries(${exe} PRIVATE xpro::libspatialite ${${exe}_deps})
    target_include_directories(${exe} PRIVATE ${CMAKE_BINARY_DIR}/cmake)
    target_compile_definitions(${exe} PRIVATE SQLITE_OMIT_TRACE) # TODO
    install(TARGETS ${exe} EXPORT ${targetsFile} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Release)
  endforeach()
  install(EXPORT ${targetsFile} DESTINATION ${XP_INSTALL_CMAKEDIR} ${nameSpace})
else()
  include(ExternalProject)
  set_property(DIRECTORY PROPERTY "EP_BASE" ${CMAKE_BINARY_DIR}/epbase)
  ExternalProject_Add(${PROJECT_NAME}
    DOWNLOAD_COMMAND sudo dnf -y install --setopt=tsflags=nodocs epel-release && sudo dnf -y install sqlite-devel libxml2-devel
    SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR} BINARY_DIR ${CMAKE_CURRENT_LIST_DIR}
    CONFIGURE_COMMAND export LIBXML2_CFLAGS=-I/usr/include/libxml2 && export LIBXML2_LIBS=-lxml2 &&
      PKG_CONFIG_PATH=/bpvol/workspace/libspatialite/_bld/epbase/Install/libspatialite/lib/pkgconfig/ &&
      <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-readosm
    )
endif()
