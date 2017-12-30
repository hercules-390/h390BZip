File README.BuildBzip2.txt - BZip2 build instructions for use with Hercules

This document Copyright © 2017 by Stephen R. Orso.  License at the bottom.

BUILDING the BZip2 library for use by Hercules

    This repository builds a BZip2 library for use by Hercules in a
    UNIX-like or Windows system (mac OS requires testing).  Hercules does
    not require anything but the library, and a CMake build script is
    included to generate the scripts needed to compile the library.
    A shared library (.so or .dll) is created; for Windows environments,
    a DLL import library is created as well.

    The original Bzip2 distribution, available from http://bzip.org, has
    an nmake-based build process that works with VS2015CE to build both
    32-bit and 64-bit versions of the libraries.   Two builds are needed
    to build the required libraries.


BUILDING the BZip2 library using CMake

    If the Hercules build determines that the BZip2 library included on
    target system is missing, does not have the development headers, or is
    at a lower version than the BZip2 sources available in the
    Hercules-390 repository, Hercules will automatically build the BZip2
    library in the Hercules build directory; no additional steps are
    needed.

    If the target system has a BZip2 library version the same as or higher
    than this version and the development headers and library are
    installed, the Hercules build will use that library, and again, no
    additional steps are required.

    If you wish to test an uplevel version of BZip2 with Hercules, then
    use the following steps to build the BZip2 library.  There is no
    need to "install" the library; Hercules can be pointed to the
    uplevel BZip2 build directory.

    1.  If you are building on Windows, open a Visual Studio command
        prompt for the bit-ness of your system.  For example, on
        a 64-bit Windows 10 system with Visual Studio 2017 Community
        Edition installed, this would be "x64 Native Tools Command
        Prompt for VS 2017"

    2.  Clone this repository: git clone https://hercules-390/BZip2

    3.  Create the directory that will be the build directory you wish
        to populate with BZip2 if needed.

    4.  Change to that build directory

    5.  Create the build scripts for the BZip2 library by using the
        following CMake command:

            cmake <source-dir>

    6.  Build the BZip2 library using the following CMake command:

            cmake --build .

        Include the option "--config Release" if building on Windows.

    7.  When building Hercules using CMake, use the command line option

            -DBZIP2_DIR=<build-dir>

        to point the Hercules build at your BZip2 build directory.


BUILDING the BZip2 library using makefile.msc (NOT RECOMMENDED)

    The following procedure builds a BZip2 library and header tree that
    is compatible only with the nmake/makefile.bat build included in
    earlier versions of Hercules.  That build procedure is deprecated.
    You will have a better experience with the CMake build procedure.

    In the examples and instructions below, Visual Studio 2015 Community
    Edition is presumed.  This process also works with Visual Studio
    2015 Community Edition, and might well work with future versions of
    Visual Studio.

    Create the dll target in makefile.msc

        The provided bzip2 build nmake does not create a dll.  Do the
        following to add the target for the dll and corresponding import
        library:

        1. Edit makefile.msc and locate the following target on or about
         line 22:

            all: lib bzip2 test

         Change the line to:

            all: dll lib bzip2 test

        2. Locate the bzip2: target on or about line 22 and change it to:

            bzip2: dll lib
                $(CC) $(CFLAGS) /Febzip2 bzip2.c libbz2.lib setargv.obj
                $(CC) $(CFLAGS) /Febzip2recover bzip2recover.c

         These changes link the executable with the dll and remove
         obsolete c compiler flags.

        3. Add the dll: target between the current lib: and test: targets.
         Be sure to leave a blank line between the dll: target and each
         of the exsting targets.

            dll: $(OBJS)
                $(LINK) $(LFLAGS) /dll /implib:libbz2.lib /out:libbz2.dll /def:libbz2.def $(OBJS)

        4. Change the lib: target to change the static library created by
         the target so that it does not conflict with the import library
         created by the dll: target.  When you are done, the lib: target
         should look like this:

            lib: $(OBJS)
                lib -nologo /out:libbz2s.lib $(OBJS)

        5. Add the following new files to the clean: target:

            del libbz2s.lib
            del libbz2.dll
            del libbz2.pdb

        Now you are ready to build.

    Build the binaries:

        1.  Open a Visual Studio 2015 32-bit tools command prompt.  The
            start menu item is titled:

                VS2015 x86 Native Tools Command Prompt

        2.  Issue the following command to build the 32-bit dll and
            corresponding import library:

                nmake -f makefile.msc all

        3.  Copy the 32-bit files to the winbuild directory

                <source_dir>\bzip2.dll  --> winbuild\bzip2
                <source_dir>\bzip2.lib  --> winbuild\bzip2
                <source_dir>\bzip2.h            --> winbuild\bzip2

        4.  Remove the 32-bit binaries from the source directory

                nmake -f makefile.msc clean

        5.  Open a Visual Studio 2015 64-bit tools command prompt using the
            Windows Start menu.  On a 64-bit Windows machine, the start
            menu item is titled:

                VS2015 x64 Native Tools Command Prompt

            On a 32-bit machine, look for:

                VS2015 x86 x64 Cross Tools Command Prompt

        6.  Issue the following command to build the 64-bit dll and
            corresponding import library:

                nmake -f makefile.msc all

        7.  Copy the 64-bit files to the winbuild directory

                <source_dir>\bzip2.dll  --> winbuild\bzip2\x64
                <source_dir>\bzip2.lib  --> winbuild\bzip2\x64



This work is licensed under the Creative Commons Attribution- 
ShareAlike 4.0 International License. 

To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ 
or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.

