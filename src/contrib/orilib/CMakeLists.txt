# This file is part of the 'orilib' software.
# Copyright (C) 2007-2009, 2012 Romain Quey
# see the COPYING file in the top-level directory.

cmake_minimum_required(VERSION 2.6)
cmake_policy(SET CMP0012 NEW)
project(orilib C)

set (ORILIB_VERSION_MAJOR 2)
set (ORILIB_VERSION_MINOR 1)
set (ORILIB_VERSION_PATCH 1)
set (ORILIB_VERSION_DEVEL 2)

# should we use the optional libraries?
# option (HAVE_GSL      "Use the GNU Scientific Library (GSL)" ON)
option (HAVE_PNG      "Use the libpng library"               ON) # PNG
option (HAVE_GLPK     "Use glpk"                             OFF) # GLPK
option (HAVE_NLOPT    "Use nlopt"                            OFF)# NLOPT
option (HAVE_OPENMP   "Use OpenMP"                           ON)
option (DEVEL_OPTIMIZATION "Enable optimization (ON for production use)" ON)
option (DEVEL_DEBUGGING_FLAG "Enable debugging compilation flag (OFF for production use)" OFF)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/ol_config.h.in"
  "${PROJECT_BINARY_DIR}/ol_config.h"
)

include_directories("${PROJECT_BINARY_DIR}")

if (DEFINED UT_INCLUDE_DIR AND DEFINED UT_BINARY_DIR)
  include_directories(${UT_INCLUDE_DIR} ${UT_BINARY_DIR})
elseif (EXISTS ${PROJECT_SOURCE_DIR}/contrib/ut)
    include_directories(${PROJECT_SOURCE_DIR}/contrib/ut
                        ${PROJECT_BINARY_DIR}/contrib/ut)
    add_subdirectory(contrib/ut)
else ()
  message (FATAL_ERROR "Fail to find ut")
endif ()

add_definitions(-Wall -W)
if (DEVEL_OPTIMIZATION)
  add_definitions(-O3)
endif ()

if (DEVEL_DEBUGGING_FLAG)
  add_definitions(-g -Werror)
endif ()

set (CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS})

if (NOT DEFINED BUILD_SHARED_LIBS)
  set (BUILD_SHARED_LIBS ON)
endif ()

add_library (orilib
ol_cal/ol_com.c
ol_cal/ol_csys.c
ol_cal/ol_dis.c
ol_cal/ol_inv.c
ol_cal/ol_sym.c
ol_cal/ol_mis.c
ol_des/ol_des_gen.c
ol_des/ol_e.c
ol_des/ol_g.c
ol_des/ol_m.c
ol_des/ol_pole.c
ol_des/ol_q.c
ol_des/ol_R.c
ol_des/ol_homochoric.c
ol_des/ol_rtheta.c
ol_gen/ol_nb_e.c
ol_gen/ol_label_g.c
ol_gen/ol_nb_rtheta.c
ol_map/ol_map_alloc.c     # PNG
ol_map/ol_map_copy.c      # PNG
ol_map/ol_map_filter.c    # PNG
ol_map/ol_map_png.c       # PNG
ol_map/ol_map_rgb.c       # PNG
ol_map/ol_map_transform.c # PNG
ol_set/ol_mean.c
ol_set/ol_set_alloc.c
ol_set/ol_set_aniso.c
ol_set/ol_set_sample.c
ol_set/ol_spread.c
ol_xpl/ol_gnd_alloc.c     # NLOPT GLPK
ol_xpl/ol_gnd_nye.c       # NLOPT GLPK
ol_xpl/ol_gnd_rho.c       # NLOPT GLPK
ol_xpl/ol_gnd_rho2.c      # NLOPT GLPK
)

## Link libraries
# this is for the GSL
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

find_library (MATHLIB m)
if (MATHLIB)
  set (EXTRA_LIBS ${EXTRA_LIBS} ${MATHLIB})
endif ()

if (HAVE_GSL)
  find_package(GSL REQUIRED)
  set (EXTRA_LIBS ${EXTRA_LIBS} ${GSL_LIBRARIES})
  include_directories (${GSL_INCLUDE_DIRS})
endif ()

if (HAVE_GLPK)                                      # GLPK
  find_package(GLPK REQUIRED)                       # GLPK
  set (EXTRA_LIBS ${EXTRA_LIBS} ${GLPK_LIBRARIES})  # GLPK
  include_directories (${GLPK_INCLUDE_DIRS})        # GLPK
endif ()                                            # GLPK

if (HAVE_NLOPT)                                     # NLOPT
  find_package(NLOPT REQUIRED)                      # NLOPT
  set (EXTRA_LIBS ${EXTRA_LIBS} ${NLOPT_LIBRARIES}) # NLOPT
  include_directories (${NLOPT_INCLUDE_DIRS})       # NLOPT
endif ()                                            # NLOPT

if (HAVE_PNG)                                       # PNG
  find_package(PNG REQUIRED)                        # PNG
  set (EXTRA_LIBS ${EXTRA_LIBS} ${PNG_LIBRARIES})   # PNG
  include_directories (${PNG_INCLUDE_DIRS})         # PNG
endif ()                                            # PNG

if (HAVE_OPENMP)
  find_package (OpenMP REQUIRED)
  add_definitions(${OpenMP_C_FLAGS})
endif ()

target_link_libraries (orilib ${EXTRA_LIBS})

# INCLUDE_DIRECTORIES(${EXTRA_LIBS})

# Installation of the library
# SET (CMAKE_INSTALL_PREFIX /usr/local/lib)

set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_des.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_cal.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_gen.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_set.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_map.h) # PNG
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_glo.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/ol_xpl.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_BINARY_DIR}/ol_config.h)
set (INCLUDE_FILES ${INCLUDE_FILES} ${PROJECT_SOURCE_DIR}/orilib.h)

install (TARGETS orilib DESTINATION lib PERMISSIONS OWNER_READ GROUP_READ WORLD_READ)
install (FILES ${INCLUDE_FILES} DESTINATION include/orilib PERMISSIONS OWNER_READ GROUP_READ WORLD_READ)
