# CMakeLists.txt
#
# This file is part of NEST.
#
# Copyright (C) 2004 The NEST Initiative
#
# NEST is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 2 of the License, or
# (at your option) any later version.
#
# NEST is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with NEST.  If not, see <http://www.gnu.org/licenses/>

cmake_minimum_required( VERSION 2.8.12 )
# juqueen 2.8.12.2

# add cmake modules: for all `include(...)` first look here
list( APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake )

project( nest CXX C )
set( NEST_USER_EMAIL "nest_user@nest-initiative.org" )

# VERSION exported to libnestutil/config.h, extras/create_release.sh
set( NEST_VERSION_MAJOR 2 )
set( NEST_VERSION_MINOR 12 )
set( NEST_VERSION_PATCHLEVEL 0 )

set( NEST_VERSION_VERSION "${NEST_VERSION_MAJOR}.${NEST_VERSION_MINOR}.${NEST_VERSION_PATCHLEVEL}" )
set( NEST_VERSION_PRGNAME "${PROJECT_NAME}-${NEST_VERSION_VERSION}" )

################################################################################
##################         All User Defined options           ##################
################################################################################

# set NEST defaults
set( tics_per_ms "1000.0" CACHE STRING "Specify elementary unit of time. [default 1000.0]" )
set( tics_per_step "100" CACHE STRING "Specify resolution. [default 100]" )
option( with-ps-arrays "Use PS array construction semantics. [default=ON]" ON )

# add user modules
set( external-modules OFF CACHE STRING "External NEST modules to be linked in, separated by ';'. [default=OFF]" )

# connect NEST with external projects
set( with-libneurosim OFF CACHE STRING "Request the use of libneurosim. Optionally give the directory, where libneurosim is installed. [default=OFF]" )
set( with-music OFF CACHE STRING "Request the use of MUSIC. Optionally give the directory, where MUSIC is installed. [default=OFF]" )

# set parallelization scheme
set( with-mpi OFF CACHE STRING "Request compilation with MPI. Optionally give directory with MPI installation. [default=OFF]" )
set( with-openmp ON CACHE STRING "Enable OpenMP multithreading. Optional: set OMP flag. [default=ON]" )

# define default libraries
set( with-gsl ON CACHE STRING "Find a gsl library. To set a specific gsl installation, set install path. [default=ON]" )
set( with-readline ON CACHE STRING "Find a readline library. To set a specific readline, set install path. [default=ON]" )
set( with-ltdl ON CACHE STRING "Find a ltdl library. To set a specific ltdl, set install path. [default=ON]" )
set( with-python ON CACHE STRING "Build PyNEST. To set a specific Python, set install path. [default=ON]" )
option( cythonize-pynest "Use Cython to cythonize pynestkernel.pyx. If OFF, PyNEST has to be build from a pre-cythonized pynestkernel.pyx. [default=ON]" ON )

# Whether to build a 'mostly' static executable and static libraries.
option( static-libraries "Build static executable and libraries. [default=OFF]" OFF )
# additional compile flags
set( with-optimize ON CACHE STRING "Enable user defined optimizations. [default ON, when ON, defaults to '-O2']" )
set( with-warning ON CACHE STRING "Enable user defined warnings. [default ON, when ON, defaults to '-Wall']" )
set( with-debug OFF CACHE STRING "Enable user defined debug flags. [default OFF, when ON, defaults to '-g']" )
set( with-libraries OFF CACHE STRING "Link additional libraries. Give full path. Separate multiple libraries by ';'. [default OFF]" )
set( with-includes OFF CACHE STRING "Add additional include paths. Give full path without '-I'. Separate multiple include paths by ';'. [default OFF]" )
set( with-defines OFF CACHE STRING "Additional defines, e.g. '-DXYZ=1'. Separate multiple defines by ';'. [default OFF]" )

# cross-compiling
# should be set via toolchain files
set( enable-bluegene OFF CACHE STRING "Configure for BlueGene." )
option( k-computer "Enable K computer." OFF )

################################################################################
##################      Project Directory variables           ##################
################################################################################

# In general use the CMAKE_INSTALL_<dir> and CMAKE_INSTALL_FULL_<dir> vars from
# GNUInstallDirs, but the CMAKE_INSTALL_DATADIR is usually just CMAKE_INSTALL_DATAROOTDIR
# and we want it to be CMAKE_INSTALL_DATAROOTDIR/PROJECT_NAME
set( CMAKE_INSTALL_DATADIR "share/${PROJECT_NAME}" CACHE STRING "Relative directory, where NEST installs its data (share/nest)" )

include( GNUInstallDirs )

################################################################################
#################        Do not install into /usr/local        #################
################################################################################

if ( "${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local" )
  # no prefix given?
  message( FATAL_ERROR "Do not install into '${CMAKE_INSTALL_PREFIX}'. "
                    "Please set -DCMAKE_INSTALL_PREFIX:PATH=<install prefix>." )
endif ()

################################################################################
##################           Find utility programs            ##################
################################################################################

# needed for target doc and fulldoc
find_package( Doxygen )
find_program( SED NAMES sed gsed )

################################################################################
##################   CPack, checks, ... in external modules   ##################
################################################################################

# This include checks the symbols, etc.
include( CheckIncludesSymbols )

# These includes publish function names.
include( ProcessOptions )
include( WriteStaticModules_h )
include( CPackVariables )
include( CheckExtraCompilerFeatures )
include( ConfigureSummary )
include( GetTriple )

# get triples arch-vendor-os
get_host_triple( NEST_HOST_TRIPLE NEST_HOST_ARCH NEST_HOST_VENDOR NEST_HOST_OS )
get_target_triple( NEST_TARGET_TRIPLE NEST_TARGET_ARCH NEST_TARGET_VENDOR NEST_TARGET_OS )

# set cpack variables and
# generate CPackConfig.cmake CPackSourceConfig.cmake files
nest_set_cpack_variables()

# Process the command line arguments
nest_process_with_optimize()
nest_process_with_debug()
nest_process_with_warning()
nest_process_with_libraries()
nest_process_with_includes()
nest_process_with_defines()
nest_process_k_computer()
nest_process_enable_bluegene()
nest_process_static_libraries()
nest_process_external_modules()
nest_process_tics_per_ms()
nest_process_tics_per_step()
nest_process_with_ps_array()
nest_process_with_libltdl()
nest_process_with_readline()
nest_process_with_gsl()
nest_process_with_python()
nest_process_with_openmp()
nest_process_with_mpi()
nest_process_with_libneurosim()
nest_process_with_music()

nest_get_color_flags()
set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${NEST_C_COLOR_FLAGS}" )
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${NEST_CXX_COLOR_FLAGS}" )

# requires HAVE_LIBNEUROSIM
nest_default_modules()

nest_write_static_module_header( "${PROJECT_BINARY_DIR}/nest/static_modules.h" )

# check additionals
nest_check_exitcode_abort()
nest_check_exitcode_segfault()
nest_check_have_cmath_makros_ignored()
nest_check_have_alpha_cxx_std_bug()
nest_check_have_sigusr_ignored()
nest_check_have_static_template_declaration_fail()
nest_check_have_stl_vector_capacity_base_unity()
nest_check_have_stl_vector_capacity_doubling()
nest_check_have_xlc_ice_on_using()
nest_check_have_std_nan()
nest_check_have_std_isnan()

################################################################################
##################         Enable Testing Targets             ##################
################################################################################
enable_testing()
set( TEST_LOGFILE ${PROJECT_BINARY_DIR}/reports/installcheck.log )
file( MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/reports/ )

if ( HAVE_PYTHON )
  add_custom_target( installcheck
      #COMMAND ${CMAKE_MAKE_PROGRAM} install
      COMMAND export PATH="${CMAKE_INSTALL_FULL_BINDIR}:$ENV{PATH}" &&
      export PYTHON="${PYTHON}" &&
      export PYTHONPATH="${CMAKE_INSTALL_PREFIX}/${PYEXECDIR}:$ENV{PYTHONPATH}" &&
      ${CMAKE_INSTALL_FULL_DATADIR}/extras/do_tests.sh --test-pynest --source-dir="${PROJECT_SOURCE_DIR}"
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}"
      COMMENT "Execute NEST's testsuite...."
      )
else ()
  add_custom_target( installcheck
      #COMMAND ${CMAKE_MAKE_PROGRAM} install
      COMMAND export PATH="${CMAKE_INSTALL_FULL_BINDIR}:$ENV{PATH}" &&
      ${CMAKE_INSTALL_FULL_DATADIR}/extras/do_tests.sh --source-dir="${PROJECT_SOURCE_DIR}"
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}"
      COMMENT "Execute NEST's testsuite...."
      )
endif ()

add_custom_target( check
    COMMAND export CTEST_OUTPUT_ON_FAILURE=1 &&
    ${CMAKE_CTEST_COMMAND} -C ${CMAKE_BUILD_TYPE}
    WORKING_DIRECTORY "${PROJECT_BINARY_DIR}"
    )

################################################################################
##################        Define Subdirectories here          ##################
################################################################################

if ( HAVE_LIBNEUROSIM )
  add_subdirectory( conngen )
endif ()
add_subdirectory( doc )
add_subdirectory( examples )
add_subdirectory( extras )
add_subdirectory( lib )
add_subdirectory( libnestutil )
add_subdirectory( librandom )
add_subdirectory( models )
add_subdirectory( sli )
add_subdirectory( nest )
add_subdirectory( nestkernel )
add_subdirectory( precise )
add_subdirectory( testsuite )
add_subdirectory( topology )
if ( HAVE_PYTHON )
  add_subdirectory( pynest )
endif ()

################################################################################
##################           Summary of flags                 ##################
################################################################################

# used in nest-config

# all compiler flags
if ( NOT CMAKE_BUILD_TYPE OR "${CMAKE_BUILD_TYPE}" STREQUAL "None" )
  set( ALL_CFLAGS "${CMAKE_C_FLAGS}" )
  set( ALL_CXXFLAGS "${CMAKE_CXX_FLAGS}" )
elseif ( ${CMAKE_BUILD_TYPE} STREQUAL "Debug" )
  set( ALL_CFLAGS "${CMAKE_C_FLAGS}   ${CMAKE_C_FLAGS_DEBUG}" )
  set( ALL_CXXFLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_DEBUG}" )
elseif ( ${CMAKE_BUILD_TYPE} STREQUAL "Release" )
  set( ALL_CFLAGS "${CMAKE_C_FLAGS}   ${CMAKE_C_FLAGS_RELEASE}" )
  set( ALL_CXXFLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE}" )
elseif ( ${CMAKE_BUILD_TYPE} STREQUAL "RelWithDebInfo" )
  set( ALL_CFLAGS "${CMAKE_C_FLAGS}   ${CMAKE_C_FLAGS_RELWITHDEBINFO}" )
  set( ALL_CXXFLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELWITHDEBINFO}" )
elseif ( ${CMAKE_BUILD_TYPE} STREQUAL "MinSizeRel" )
  set( ALL_CFLAGS "${CMAKE_C_FLAGS}   ${CMAKE_C_FLAGS_MINSIZEREL}" )
  set( ALL_CXXFLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_MINSIZEREL}" )
else ()
  message( FATAL_ERROR "Unknown build type: '${CMAKE_BUILD_TYPE}'" )
endif ()
if ( with-defines )
  foreach ( def ${with-defines} )
    set( ALL_CFLAGS "${def} ${ALL_CFLAGS}" )
    set( ALL_CXXFLAGS "${def} ${ALL_CXXFLAGS}" )
  endforeach ()
endif ()

# all libraries
set( ALL_LIBS
  "-lnestutil"
  "-lnest"
  "-lrandom"
  "-lsli"
  "-lnestkernel"
  "${OpenMP_CXX_FLAGS}"
  "${LTDL_LIBRARIES}"
  "${READLINE_LIBRARIES}"
  "${GSL_LIBRARIES}"
  "${LIBNEUROSIM_LIBRARIES}"
  "${MUSIC_LIBRARIES}"
  "${MPI_CXX_LIBRARIES}" )
if ( with-libraries )
  set( ALL_LIBS "${ALL_LIBS};${with-libraries}" )
endif ()
string( REPLACE ";" " " ALL_LIBS "${ALL_LIBS}" )

# all includes
set( ALL_INCLUDES_tmp
  "${PROJECT_SOURCE_DIR}/libnestutil"
  "${PROJECT_BINARY_DIR}/libnestutil"
  "${PROJECT_SOURCE_DIR}/librandom"
  "${PROJECT_SOURCE_DIR}/sli"
  "${PROJECT_SOURCE_DIR}/topology"
  "${PROJECT_SOURCE_DIR}/nestkernel"
  "${PROJECT_SOURCE_DIR}/nest"
  "${LTDL_INCLUDE_DIRS}"
  "${READLINE_INCLUDE_DIRS}"
  "${GSL_INCLUDE_DIRS}"
  "${LIBNEUROSIM_INCLUDE_DIRS}"
  "${MUSIC_INCLUDE_DIRS}"
  "${MPI_CXX_INCLUDE_PATH}" )
set( ALL_INCLUDES "" )
foreach ( INC ${ALL_INCLUDES_tmp} ${with-includes} )
  if ( INC AND NOT INC STREQUAL "" )
    set( ALL_INCLUDES "${ALL_INCLUDES} -I${INC}" )
  endif ()
endforeach ()

if ( HAVE_LIBNEUROSIM )
  set( ALL_LIBS "-lconngen ${ALL_LIBS}" )
  set( ALL_INCLUDES "-I${PROJECT_SOURCE_DIR}/conngen ${ALL_INCLUDES}" )
endif ()

################################################################################
##################           File generation here             ##################
################################################################################

configure_file(
    "${PROJECT_SOURCE_DIR}/libnestutil/config.h.in"
    "${PROJECT_BINARY_DIR}/libnestutil/config.h" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/pynest/setup.py.in"
    "${PROJECT_BINARY_DIR}/pynest/setup.py" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/pynest/do_tests.py.in"
    "${PROJECT_BINARY_DIR}/pynest/do_tests.py" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/topology/setup.py.in"
    "${PROJECT_BINARY_DIR}/topology/setup.py" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/testsuite/do_tests.sh.in"
    "${PROJECT_BINARY_DIR}/testsuite/do_tests.sh" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/extras/EditorSupport/emacs/sli.el.in"
    "${PROJECT_BINARY_DIR}/extras/EditorSupport/emacs/sli.el" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/extras/nest-config.in"
    "${PROJECT_BINARY_DIR}/extras/nest-config" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/extras/create_release.sh.in"
    "${PROJECT_BINARY_DIR}/extras/create_release.sh" @ONLY
)

configure_file(
    "${PROJECT_SOURCE_DIR}/extras/nest_vars.sh.in"
    "${PROJECT_BINARY_DIR}/extras/nest_vars.sh" @ONLY
)

################################################################################
##################            Install Extra Files             ##################
################################################################################

install( FILES LICENSE README.md NEWS
    DESTINATION ${CMAKE_INSTALL_DOCDIR}
    )

nest_print_config_summary()
