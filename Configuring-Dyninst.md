Dyninst is built via CMake. We recommend performing an interactive
configuration with "ccmake ." first, in order to see which options are
relevant for your system. You may also perform a batch configuration
with "cmake .".  Options are passed to CMake with -DVAR=VALUE. Common
options include:

Boost_INCLUDE_DIR 
CMAKE_BUILD_TYPE 
CMAKE_INSTALL_PREFIX
LIBDWARF_INCLUDE_DIR 
LIBDWARF_LIBRARIES 
LIBELF_INCLUDE_DIR
LIBELF_LIBRARIES 
IBERTY_LIBRARIES

CMake's default generator on Linux is normally "Unix Makefiles", and
on Windows, it will normally produce project files for the most recent
version of Visual Studio on your system. Other generators should work
but are not tested. After the CMake step concludes, you will have
appropriate Makefiles or equivalent and can build Dyninst.

We require CMake 2.6 as a minimum on all systems, and CMake 2.8.11
allows us to automatically download and build libelf/libdwarf/binutils
on ELF systems if they are needed. If you do not have a sufficiently
recent CMake, you may need to manually specify the location of these
dependencies. If you are cross-compiling Dyninst, including builds for
various Cray and Intel MIC systems, you will either need a toolchain
file that specifies how to properly cross-compile, or you will need to
manually define the appropriate compiler, library locations, include
locations, and the CROSS_COMPILING flag so that the build system will
properly evaluate what can be built and linked in your environment.