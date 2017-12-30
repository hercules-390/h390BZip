## BZlib2 for use with Hercules

This repository contains an unmodified copy of BZip2 1.0.6, retrieved
October 2017 from http://www.bzip.org/ and stored in the `pkg_src`
subdirectory.  This should allow a complete replacement of the `pkg_src`
directory contents when an uplevel upstream BZip2 distribution becomes
available.

A CMake build script has been added.  The CMake script, while based on
makefile and and makefile.msc in the upstream BZip2 distribution, is
specific to Hercules' requirements and only builds a bzlib shared
library. The build script exports targets for the build directory tree
and, if the library is installed, for the installation directory tree.  
The CMake build for Hercules can import either target.  If BZip2 is 
built by the CMake build for Hercules, the build tree target is used.

The CMake build for Hercules will clone and build this repository if
BZip2 is not installed on the target system or if the version installed
on the target system is older than the version in this repository.

Build scripts and other programming in the top level is copyright by
Stephen Orso and is licensed under the Boost license.   Documentation
and other writing is licensed using the Creative Commons By-SA 4.0
license.

Each file contains specific copyright and license information.

The BZip2 package retains its original copyright and license, 
which can be found at `pkg_src/LICENSE'

---
&nbsp;
### CMake -D Configuration Option Variables


Except for `BUILD_TESTING`, the configuration options used by the CMake
build for Hercules ensure that the BZip2 library is built in a manner
consistent with the options and target for which Hercules is being
built.

- `BUILD_TESTING  ON | OFF`, default is `OFF`
    When `ON`, include the `bzip2` executable in the build tree and
    generate test cases to verify the operation of BZip2
- `DEBUG ON | OFF`, default is `OFF`
    Applicable to single-configuration generators like Makefiles and
    Ninja.  When ON, generate a Debug configuration build script.
    When OFF, generate a Release script.  (The configuration used
    for multi-configuration generators like Microsoft Visual Studio
    or Apple Xcode is determined at build time.)
- `WINTARGET`  blank | `HOST` | `DIST` | windows-version
    Applicable when building on Windows.  Option windows-version may be
    any of `WinXP364`, `WinVista`, `Win7`, `Win8`, `Win10` and is case
    insensitive.  When blank or `HOST`, build for the version of Windows
    on the host.  When `DIST`, build for the earliest version of Windows
    supported by Hercules, Windows XP SP3 64-bit.  Otherwise build for
    the Windows version specified.
    
---
&nbsp;
### Differences from the Makefile build

- Normally, only the bzlib library is built, and it is built as a shared
  library (libhercbz2.so or libhercbz2.dll).  On Windows, the DLL import
  library libhercbz2.lib is also built.

  [The file names are really libhercbz2.so and libhercbz2.dll.
  Normally, the library would be referred to as hercbz2; the Windows file
  names would be hercbz2.dll, hercbz2.lib, and hercbz2.pdb, and on UNIX-
  like systems the name would be libhercbz2.so.   The Windows and UNIX-like
  makefiles did not implement this convention for the original file names,
  and for the sake of consistency with the this package's original 
  makefiles, the convention is ignored here too.]

- If the option `BUILD_TESTING` is `ON` (`-DBUILD_TESTING=ON`), then the
  hercbzip2 executable is built and test cases are added.  The hercbzip2
  executable is not included in the installed components.
  
---
&nbsp;
### CMake-specific characteristics of this CMakeLists.txt

- The library installation target is exported to the bzip2-targets
  sub-directory for inclusion by Hercules.

- The library target is also exported from the build tree to allow the
  BZip2 library to be included in a build that references the build
  directory.  If this package is built within the Hercules build, there 
  is no need to install it; the build tree target can be imported.

- The single public header is copied to the build tree so that the
  exported build tree target does not make any references to the source
  tree.

- For targets that support build-time configuration selection (Visual 
  Studio and Xcode), only the Release and Debug configurations are 
  included.  The MinSizeRel and RelWithDebInfo configurations are 
  removed if they are present in CMAKE\_CONFIGURATION\_TYPES.

- On Windows, for both Release and Debug configurations, linker .pdb
  files are created and included in the install of the library.

CMake 3.4 is the minimum version required to build BZip2 because it is
the first version to include the boolean WINDOWS\_EXPORT\_ALL\_SYMBOLS,
which creates the exports file (.def) needed to build DLLs from source
that lacks specific \_\_declspec( dllexport ) declarations.

&nbsp;
&nbsp;

-----------

This README.md file Copyright © 2017 by Stephen R. Orso.

This work is licensed under the Creative Commons Attribution-ShareAlike
4.0 International License.

To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/
or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
