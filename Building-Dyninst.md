# Building Dyninst

### ---- ALERT ----
Dyninst-10.1 includes a new set of rules in the build system. One consequence of this is that the ```CMAKE_INSTALL_PREFIX/INSTALL_INCLUDE_DIR``` directory is now one of the places the compiler searches when looking for header files to build Dyninst. It is therefore necessary to delete your old installation directory before upgrading Dyninst from source. If you are contributing changes to Dyninst, please see the [Contributing to the project](https://github.com/dyninst/dyninst/wiki/Contributions) page for details.

***

##### Table of Contents
* [Spack](#spack)
* [From source (short)](#source_short)
* [From source (long)](#source_long)
    * [Configuration](#configuration)
    * [Building](#building)
* [FAQ](#faq)
***

<a name="spack"/>

## Build with Spack

```spack install dyninst```

<a name="source_short"/>

## Build from source (short)

1. Configure Dyninst with CMake

	```cmake /path/to/dyninst/source -DCMAKE_INSTALL_PREFIX=/path/to/installation```

2. Build and install Dyninst in parallel

	```make install -jN```

***

<a name="source_long"/>

## Build from source (long)

<a name="configuration"/>

### Configuration

Dyninst is built via CMake. We require CMake 3.4.0 as a minimum on all systems. CMake will automatically search for dependencies and download them when not found (to disable this, see [Sterile Builds](third-party-deps#sterile_build)).

#### Third-party Dependencies

The main dependencies are

1. [elfutils](https://sourceware.org/elfutils/) 0.173 or newer

2. [Boost](https://www.boost.org/) 1.61.0 or newer

3. [Intel Thread Building Blocks](https://www.threadingbuildingblocks.org/) (TBB) 2018 U6 or newer

4. OpenMP, optional for parallel code parsing. Note that Dyninst will not automatically download or install OpenMP

5. libiberty (part of [binutils](https://www.gnu.org/software/binutils/)). A C++ name demangler library. Only used if USE_GNU_DEMANGLER=NO.

***
Build options are passed to CMake with -DVAR=VALUE. Dyninst is highly configurable, but the most common options are

	Boost_ROOT_DIR: base directory of your boost installation
	
	LibElf_ROOT_DIR: base directory of your elfutils installation
	
	TBB_ROOT_DIR: base directory of your TBB installation
	
	LibIberty_ROOT_DIR: base directory of your iberty installation (only used if USE_GNU_DEMANGLER=NO).

	CMAKE_BUILD_TYPE: may be set to Debug, Release, or RelWithDebInfo for unoptimized, optimized, and optimized with debug information builds respectively. Note that RelWithDebInfo is the default.
	
	CMAKE_INSTALL_PREFIX: like PREFIX for autotools-based systems. Where to install things.
	
	USE_OpenMP: whether or not use OpenMP for parallel parsing. Default to be ON
	
	ENABLE_STATIC_LIBS: also build dyninst in static libraries. Default to be NO. Set to YES to build static libraries.

	STERILE_BUILD: Do not download/build third-party dependencies from source. (default: OFF)

>See the [Third-party Dependencies](https://github.com/dyninst/dyninst/wiki/third-party-deps) page for a more thorough discussion of options available.

CMake's default generator on Linux is normally "Unix Makefiles", and on Windows, it will normally produce project files for the most recent version of Visual Studio on your system. Other generators should work but are not tested. After the CMake step concludes, you will have appropriate Makefiles or equivalent and can build Dyninst.

If you are cross-compiling Dyninst, including builds for various Cray and Intel MIC systems, you will either need a toolchain file that specifies how to properly cross-compile, or you will need to manually define the appropriate compiler, library locations, include locations, and the CROSS_COMPILING flag so that the build system will properly evaluate what can be built and linked in your environment.


<a name="building"/>

### Building and installing

To build Dyninst and all its components, 

    make && make install

from the top-level directory of the source tree. To build and install a single component and its dependencies, do the same thing from that component's subdirectory. Libraries will be installed into

    CMAKE_INSTALL_PREFIX/INSTALL_LIB_DIR

and headers will be installed into 

    CMAKE_INSTALL_PREFIX/INSTALL_INCLUDE_DIR

If you wish to import Dyninst into your own CMake projects, the export information is in

    CMAKE_INSTALL_PREFIX/INSTALL_CMAKE_DIR

PDF documentation is included and installed to 

    CMAKE_INSTALL_PREFIX/INSTALL_DOC_DIR

If you update the LaTeX source documents for any manuals, 

    make doc

will rebuild them. Components may be built and installed individually: 

    make $COMPONENT

and

    make $COMPONENT-install

respectively; this will appropriately respect inter-component dependencies.


<a name="faq"/>

## FAQ


1. **Q**: What should I do if the build fails with an error message like

	```"/lib64/libdw.so.1: version `ELFUTILS_0.173' not found```
	
	or
	
	```error: 'dwarf_next_lines' was not declared in this scope```

	**A**: Dyninst now depends on elfutils-0.173 or later. If you are seeing the above errors, it means the elfutils installed on your system is older than 0.173. We recommend that you set ```ElfUtils_ROOT_DIR``` to empty, which will trigger the build system to automatically download the correct version of elfutils. Or you can upgrade your elfutils with your system package manager. As of Dyninst-10.1, you should never see this error. If you are, please submit an [issue](https://github.com/dyninst/dyninst/issues).

2. **Q**: Where are the dependency libraries downloaded by Dyninst?

	**A**: Dyninst copies all library and header files for each third-party dependency into the install locations ```CMAKE_INSTALL_PREFIX/lib``` and ```CMAKE_INSTALL_PREFIX/include```, respectively.

3. **Q**: My system has pre-installed Boost, but the build failed due to that cannot find boost libraries.

	**A**: Dyninst-10.1 and newer requires at least Boost-1.61. If your system version of Boost is newer than this, but still not found, then point ```Boost_ROOT_DIR``` to the base of your system's Boost install.
