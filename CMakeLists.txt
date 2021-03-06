# This file is part of adms - http://sourceforge.net/projects/mot-adms.
#
# adms is a code generator for the Verilog-AMS language.
#
# Copyright (C) 2014-2015 Guilherme Brondani Torri <guitorri@gmail.com>
#               2014 Luther T. Cat <lutherthecat@github>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

# skip CMake 3.0 project version management for the moment
if(POLICY CMP0048)
  cmake_policy(SET CMP0048 OLD)
endif()

#==================================
PROJECT(ADMS C)
cmake_minimum_required(VERSION 2.8)

# TODO: store version in a sensible place
# e.g. qucs: file(STRINGS "${qucs-project_SOURCE_DIR}/VERSION" QUCS_VERSION)

SET(PACKAGE_NAME "adms")
SET(PACKAGE_VERSION "2.3.7")
SET(PACKAGE_BUGREPORT "qucs-bugs@lists.sourceforge.net")

# Default to static libs ( shared needs work, BUG with MinGW). 
OPTION ( BUILD_STATIC_LIBS "Build static libraries." ON )
IF ( MINGW )
  IF ( BUILD_STATIC_LIBS )
    ADD_DEFINITIONS(-Dstaticlink)
  ENDIF()
ENDIF()

INCLUDE( CheckIncludeFile )
check_include_file( sys/stat.h HAVE_SYS_STAT_H )
check_include_file( locale.h HAVE_LOCALE_H)
check_include_file( float.h HAVE_FLOAT_H)
check_include_file( unistd.h HAVE_UNISTD_H)
check_include_file( stdlib.h HAVE_STDLIB_H)
check_include_file( string.h HAVE_STRING_H)

INCLUDE( CheckFunctionExists )
CHECK_FUNCTION_EXISTS (putenv HAVE_PUTENV)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

# Perl is used to recreate sources
# * mkelements.pl
# * mktext.pl
# * mkgrammar.pl
# Checking for perl modules requires FindPerlModules.cmake
# Check all required modules at once
SET(CMAKE_MODULE_PATH "${ADMS_SOURCE_DIR}/cmake/modules")
include(FindPerlModules)
find_package(PerlModules COMPONENTS XML::LibXML)
if(NOT PERLMODULES_FOUND)
  message(FATAL_ERROR "Not all required perl modules were found on your system")
endif(NOT PERLMODULES_FOUND)


find_package(BISON 2.5)

# handle api.prefix, unused for the moment
IF (${BISON_VERSION} VERSION_LESS "2.6")
  SET (BISON_APIPREFIX "-p")
ELSE (${BISON_VERSION} VERSION_LESS "2.6")
  SET (BISON_APIPREFIX "-Dapi.prefix=")
ENDIF (${BISON_VERSION} VERSION_LESS "2.6")

find_package(FLEX)

ADD_SUBDIRECTORY(admsXml)

