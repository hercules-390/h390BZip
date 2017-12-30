# CMakeLists.txt - Build BZip2 as needed for Hercules

#[[   Copyright 2017 by Stephen Orso.

      Distributed under the Boost Software License, Version 1.0.
      See accompanying file BOOST_LICENSE_1_0.txt or copy at
      http://www.boost.org/LICENSE_1_0.txt)
]]

#[[
This CMakeLists.txt for BZip2 is based on the files Makefile and
and Makefile.msc that are provided with the upstream BZip2 distribution
and are included in the pkg_src directory for reference.

The contents of pkg_src are BZip2 1.0.6, retrieved October 2017 from
http://www.bzip.org/.

Take note: to prevent conflicts between this library and any library
provided by a UNIX-like system distribution or any library built from
sources, "bz2" is prefixed with "herc" to create a unique name.

There are NO modifications to BZip2.  The name change is just to prevent
a target system from contracting "versionitis" because Hercules built a
and installed a newer version than the one provided with the target
distribution.  This could otherwise happen if Hercules is installed in
a systems's standard install directories.

In the notes below, the "herc" prefixed names are used because that is
what is created by this configure script.

Differences from the makefile and makefile.msc included in BZip2:

- Normally, only the library is built, and it is built as a shared
  library (libhercbz2.so or libhercbz2.dll).  On Windows, a DLL import
  library is also built.  The upstream source builds only a static
  library, and it builds two executables, bzip2 and bzip2recover.

- The file names are really libhercbz2.so and libhercbz2.dll.  Normally,
  the library would be referred to as hercbz2; the Windows file names
  would be hercbz2.dll, hercbz2.lib, and hercbz2.pdb, and the UNIX-like
  system name would be libhercbz2.so.   Not so much for the build
  scripts included with the upstream source.

- If the option BUILD_TESTING is ON (-DBUILD_TESTING=ON), then the
  hercbzip2 executable is built and test cases are added.  The
  executable is not included in the items installed by the generated
  build scripts.

- The entire upstream BZip2 distribution is stored in the pkg_src
  directory.  This should allow a for a complete replacement of the
  pkg_src directory contents when a new uplevel upstream BZip2
  distribution becomes available.

CMake -D variables specific to this CMakeLists.txt

- BUILID_TESTING  ON | OFF, default is OFF
    When ON, include the hercbzip2 executable in the build tree and
    generate test cases to verify the operation of the hercbz2
    library.
- DEBUG ON | OFF, default is OFF
    Applies only to single-configuration generators like Makefiles and
    Ninja; ignored for multi-configuration generators such as Xcode and
    Visual Studio.  When ON, generate a Debug configuration build script.
    When OFF, generate a Release script.  Multi-configuration generators
    generate build scripts that require naming of the configuration
    when the package is built.  When CMake is used to run the build
    step, the --config option names the configuration.
- WINTARGET  <blank> | HOST | DIST | <windows-version>
    Applicable when building on Windows.  <windows-version may be
    any of WinXP364, WinVista, Win7, Win8, Win10.  Case insensitive.
    When blank or HOST, build for the version of Windows on the host.
    When DIST, build for the earliest version of Windows supported by
    Hercules, Windows XP SP3 64-bit.  Otherwise build for the Windows
    version specified.

CMake-specific characteristics of this CMakeLists.txt

- The library import target is exported to the bzip2-targets
  subdirectory of the install  directory for inclusion by Hercules.

- The library import target is also exported from the build tree to
  allow the BZip2 library to be included in a build that references the
  build directory.  If a Hercules build elects to build this BZip2
  package, there is no need to install it; the build tree target can be
  imported.

- The single public header is copied to the build tree so that the
  exported build tree target does not make any references to the source
  tree.

- For targets that support configurations (Visual Studio and Xcode),
  only the Release and Debug configurations are included.  The
  MinSizeRel and RelWithDebInfo configurations are removed if they are
  present in CMAKE_CONFIGURATION_TYPES.

- On Windows, for both Release and Debug configurations, linker .pdb
  files are created and included in the install of the library.

CMake 3.4 is the minimum version required to build BZip2 because it is
the first version to include the boolean WINDOWS_EXPORT_ALL_SYMBOLS,
which creates the exports file (.def) needed to build DLLs from source
that lacks specific __declspec( dllexport ) declarations.

]]


# ----------------------------------------------------------------------
# Preamble: basic CMake setup, set configuration type(s), minimum CMake
# version required, and whether tests should be enabled.
# ----------------------------------------------------------------------

cmake_minimum_required( VERSION 3.4.0 )

set( CMAKE_REQUIRED_QUIET TRUE )

project( BZip2 VERSION 1.0.6 LANGUAGES C )

include( herc_setWindowsTarget.cmake )

# In the Hercules repo for this package, source is one level down.
set( src_dir "pkg_src/"  )

option( BUILD_TESTING "Build testing targets in addition to the library" OFF )
option( DEBUG         "Build for debugging (no optimization etc)"        OFF )

if( "${CMAKE_CONFIGURATION_TYPES}" STREQUAL "" )
    # Generator in use creates build scripts for exactly one configuration.
    # Determine whether this is a release or debug build.
    if( DEBUG )
        set( CMAKE_BUILD_TYPE "Debug" )
    else( )
        set( CMAKE_BUILD_TYPE "Release" )
    endif( )
    message( STATUS "Single configuration generator; configuring for ${CMAKE_BUILD_TYPE}" )

else( )
    # For generators that support multiple configurations with build-time
    # configuration selection (Visual Studio or mac OS Xcode), CMake
    # defaults to four configurations: Release, Debug, MinSizeRel, and
    # RelWithDebInfo.  For now, only the first two are needed.  Remove the
    # other two.
    list( REMOVE_ITEM CMAKE_CONFIGURATION_TYPES
                MinSizeRel
                RelWithDebInfo
                )
    message ( STATUS "Available configuration Types: ${CMAKE_CONFIGURATION_TYPES}"  )
endif( )


# ----------------------------------------------------------------------
# Source code definitions.
# ----------------------------------------------------------------------

set( BZLIB_PUBLIC_HEADERS  ${PROJECT_SOURCE_DIR}/${src_dir}bzlib.h )

set( BZLIB_HEADERS  ${BZLIB_PUBLIC_HEADERS}
                    ${PROJECT_SOURCE_DIR}/${src_dir}bzlib_private.h )

# Copy the public headers to the build tree so that an export of build
# tree targets does not make any references to the source tree.

file( COPY ${BZLIB_PUBLIC_HEADERS} DESTINATION ${PROJECT_BINARY_DIR}/include )

set( BZLIB_SOURCES
            ${src_dir}blocksort.c
            ${src_dir}huffman.c
            ${src_dir}crctable.c
            ${src_dir}randtable.c
            ${src_dir}compress.c
            ${src_dir}decompress.c
            ${src_dir}bzlib.c
            )

# Configure the resource compiler script and include it in sources if
# compiling on Windows

if( MSVC )
    configure_file( hercbz2.rc.in hercbz2.rc @ONLY )
    set( BZLIB_SOURCES
            ${BZLIB_SOURCES} ${PROJECT_BINARY_DIR}/hercbz2.rc )
endif( )

# If building for all targets, define the sources for those targets.

if( BUILD_TESTING )
    set( BZIP2_SOURCES
            ${src_dir}bzip2.c
            )
    if( MSVC )
        configure_file( hercbzip2.rc.in hercbzip2.rc @ONLY )
        set( BZIP2_SOURCES
                ${BZIP2_SOURCES} ${PROJECT_BINARY_DIR}/hercbzip2.rc )
    endif( )
# bzip2recover is not needed for running current test scripts.
#   set( BZIP2RECOVER_SOURCES
#           ${src_dir}bzip2recover.c
#           )
endif( )


# ----------------------------------------------------------------------
# Compiler and linker flags.  Because BZip2 is a relatively small
# package and does not use a config.h header, macro definitions are set
# on the command line.  Optimization flags are set in the configuration-
# specific CMake variables.
# ----------------------------------------------------------------------

# When building for Windows, ensure that the correct Windows API version
# is used.  The default is the API version of the host system.  While we
# are here, ensure the debug library gets a different name and a exports
# file is created for later creation of the import library.

if( WIN32 )
    add_definitions( -DWIN32 )
    herc_setWindowsTarget( "${WINTARGET}" Windows_version )
    if( WINVER )
        add_definitions( -DWINVER=${WINVER}
                         -D_WIN32_WINNT=${_WIN32_WINNT}
                         -DNTDDI_VERSION=${NTDDI_VERSION}  )
    endif( )
    set( CMAKE_DEBUG_POSTFIX "d" )      # provide unique names for debug libs
    # While MS is quite clear that the ANSI predefined macros are _not_
    # defined for the resource compiler, apparently at least one important
    # MS-specific one is not either:  _WIN64.  So we shall use SIZEOF_SIZE_P
    # to set it.  See: https://msdn.microsoft.com/en-us/library/windows/desktop/aa381032(v=vs.85).aspx
    # for Microsoft's limited discussion of this.

    if( CMAKE_SIZEOF_VOID_P GREATER 4 )
        set( CMAKE_RC_FLAGS "${CMAKE_RC_FLAGS} -D_WIN64" )
    else( )
        message( "CMAKE_SIZEOF_VOID_P: ${CMAKE_SIZEOF_VOID_P}" )
    endif( )

endif( )   # if( WIN32 )

# When building with MSVC, set /W1 to suppress some nuisance warnings.
# /W1 turns off the CMake-provided default of /W3.

if( MSVC )
    add_definitions( -D_CRT_NONSTDC_NO_DEPRECATE )
    set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /MP" )
    set( CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG" )
    if( CMAKE_C_FLAGS MATCHES "/W[1234]")
        string(REGEX REPLACE "/W[1234]" "/W1" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}" )
    else( )
        set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W1" )
    endif( )
endif(  )

# CMake pre-defined MSVC Release and Debug compiler flags are fine.
# CMake gcc flags need work; as provided, gcc provides "-O3 -DNDEBUG"
# for Release and "-g" for Debug.  We will avoid for the moment target
# architecture-specific optimizations.
# CMake clang flags are the same as those for gcc.  This script treats
# clang as identical to gcc.  For the moment we assume that this works
# for AppleClang as well.

if( ( ${CMAKE_C_COMPILER_ID} STREQUAL "GNU" )
        OR ( ${CMAKE_C_COMPILER_ID} STREQUAL "Clang" )
        OR ( ${CMAKE_C_COMPILER_ID} STREQUAL "AppleClang" ) )
    set( CMAKE_C_FLAGS_RELEASE "-Wall -Winline -O2 -g -DNDEBUG" )
    set( CMAKE_C_FLAGS_DEBUG   "-g3 -gdb3 -O0 -DDEBUG" )
endif( )

## Large file support is not required for Herecules' use of the library,
## nor is it required for the files in the test cases.  This saves us the
## trouble of figuring out how each target enables large files.
#if( MSVC AND BUILD_TESTING )
#    add_definitions( -D_FILE_OFFSET_BITS=64 )
#endif( )

message( STATUS "C flags (Release) are: \"${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_RELEASE}\"" )
message( STATUS "C flags (Debug) are:   \"${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_DEBUG}\"" )



# ----------------------------------------------------------------------
# Create target(s).  Only one is always created: BZlib2.  A target
#                    for bzip2 is created only if BUILD_TESTING is ON.
# ----------------------------------------------------------------------

include_directories( ${PROJECT_SOURCE_DIR}/${src_dir}  )

# Add a target for the shared library libbz2.so or libbz2.dll.  If
# building on Windows, CMake will create the import library libbz2.lib.
# The CMake import target scripts will include the import library
# if building on a Windows host.  Note: the default dll name for
# a target "bz2" is bz2.dll.  Hercules expects libbz2.dll because that
# is what the original BZip2 NMake build created.  So we must change the
# base output name for Windows.

add_library( bz2 SHARED ${BZLIB_HEADERS} ${BZLIB_SOURCES} )
if( WIN32 )
    set_target_properties( bz2 PROPERTIES
            WINDOWS_EXPORT_ALL_SYMBOLS TRUE
            OUTPUT_NAME libhercbz2 )
else( )
    set_target_properties( bz2 PROPERTIES
            OUTPUT_NAME hercbz2 )
endif( )

# Define the location of the public headers that need to be part of the
# exported targets (build tree and install) for this target.  The
# generator expression $<BUILD_INTERFACE:...> has the expected value at
# build time and is the null string at install time, and vice versa for
# $<INSTALL_INTERFACE:..>.  Without target_include_directories(), the
# public header directory will never  appear in the exported targets.

target_include_directories( bz2 PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
            $<INSTALL_INTERFACE:include> )

# If building testing targets, add the bzip2 executable and create
# the three tests.  The bzip2 target is never installed, but is instead
# used to run tests on the libbz2 library.

# If building on Windows, include setargv.obj from the Windows runtime
# for file name wildcarding.  While not really a link flag, including
# "setargv.obj" as a flag achieves the desired result.

if( BUILD_TESTING )
    add_executable( bzip2 ${BZIP2_SOURCES} )
    target_link_libraries( bzip2 bz2 )
    SET_TARGET_PROPERTIES(bzip2 PROPERTIES
            OUTPUT_NAME hercbzip )
    if( WIN32 )
        SET_TARGET_PROPERTIES(bzip2 PROPERTIES
                LINK_FLAGS "setargv.obj" )
    endif( )
##  bzip2recover is not required for testing.  But one never knows what
##  the future holds.
#   add_executable( bzip2recover ${BZIP2RECOVER_SOURCES} )
#   target_link_libraries( bzip2recover BZlib2 )

    enable_testing( )

    add_test( NAME sample1.ref
          COMMAND ${CMAKE_COMMAND}
            -DBZIP2_PATH=$<TARGET_FILE:bzip2>
            -DBZIP2_OPTION=-1
            -DSRC_DIR=${CMAKE_CURRENT_SOURCE_DIR}/${src_dir}
            -DTEST_CASE=sample1
          -P ${CMAKE_CURRENT_SOURCE_DIR}/test_bzip2.cmake
    )

    add_test( NAME sample2.ref
          COMMAND ${CMAKE_COMMAND}
            -DBZIP2_PATH=$<TARGET_FILE:bzip2>
            -DBZIP2_OPTION=-2
            -DSRC_DIR=${CMAKE_CURRENT_SOURCE_DIR}/${src_dir}
            -DTEST_CASE=sample2
          -P ${CMAKE_CURRENT_SOURCE_DIR}/test_bzip2.cmake
    )

    add_test( NAME sample3.ref
          COMMAND ${CMAKE_COMMAND}
            -DBZIP2_PATH=$<TARGET_FILE:bzip2>
            -DBZIP2_OPTION=-3
            -DSRC_DIR=${CMAKE_CURRENT_SOURCE_DIR}/${src_dir}
            -DTEST_CASE=sample3
          -P ${CMAKE_CURRENT_SOURCE_DIR}/test_bzip2.cmake
    )

endif( )

# While we are creating targets, we shall copy the package author's
# copyright, the copyright for the scripts, and the README.md created
# for the Hercules-390 build for this package.

file( COPY ${PROJECT_SOURCE_DIR}/${src_dir}/LICENSE
            DESTINATION ${PROJECT_BINARY_DIR} )
file( RENAME ${PROJECT_BINARY_DIR}/LICENSE ${PROJECT_BINARY_DIR}/LICENSE-BZip2 )

file( COPY ${PROJECT_SOURCE_DIR}/BOOST_LICENSE_1_0.txt
            DESTINATION ${PROJECT_BINARY_DIR} )

file( COPY ${PROJECT_SOURCE_DIR}/README.md
            DESTINATION ${PROJECT_BINARY_DIR} )


# ----------------------------------------------------------------------
# Install the target, public headers, and if on Windows, the .pdb files
# ----------------------------------------------------------------------

# Install the target.  Include the target in the export of the install
# target.  For Windows, the shared DLL will be installed to the bin
# directory and the DLL import library will end up in lib.

install( TARGETS bz2
            EXPORT  bzip2              # Include target in export of
                                       # ..install targets
            RUNTIME DESTINATION bin    # Used for Windows DLLs
            LIBRARY DESTINATION lib    # Used for static libraries or
                                       # ..UNIX-like shared libraries
            ARCHIVE DESTINATION lib )  # ARCHIVE used for the DLL import
                                       # ..library.
install( FILES ${BZLIB_PUBLIC_HEADERS} DESTINATION include )
install( FILES ${src_dir}LICENSE DESTINATION . )

# Any .pdb debugging file generated by the MSVC linker is installed.
# The MSVC linker creates .pdb files when /DEBUG is specified as a
# linker option.

if( MSVC )
    install( FILES $<TARGET_PDB_FILE:bz2>
            DESTINATION bin
            CONFIGURATIONS Release
            OPTIONAL )
    install( FILES $<TARGET_PDB_FILE:bz2>
            DESTINATION bin
            CONFIGURATIONS Debug
            OPTIONAL )
endif( )


# ----------------------------------------------------------------------
# Export the targets for later import by a downstream consumer of the
# BZip2 library.  Use of imported targets means the consumer of the
# BZip2 does not need to understand the directory structure of the
# BZip2 build tree or install tree.
# ----------------------------------------------------------------------

# Create the export of the installation target.  This export can be
# imported by the CMake build for Hercules.

install( EXPORT bzip2 DESTINATION bzip2-targets  )

# Create an export target of the build tree.  This allows the build tree
# to be used as an import target without requiring an install.  The build
# tree target import script is stored in the same relative location in
# the build tree as the installation target import script would be in
# install tree.  This lets Hercules include the import script without
# needing to know if it is from a build tree or an install tree.

export( EXPORT bzip2 FILE bzip2-targets/bzip2.cmake   )


# ----------------------------------------------------------------------
# Set up CPack to create installation packages
# ----------------------------------------------------------------------

# Generator-independent CPack settings.  Get more artful in setting the
# version number by extracting it from bzip2.h.

set( CPACK_PACKAGE_VERSION_MAJOR 1 )
set( CPACK_PACKAGE_VERSION_MINOR 0 )
set( CPACK_PACKAGE_VERSION_PATCH 6 )

include( CPack )


# end CMakeLists.txt
