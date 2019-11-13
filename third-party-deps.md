# Third-party Dependencies in Dyninst

##### Table of Contents
* [Sterile Build](#sterile_build)
* [Boost](#boost)
    * [tagged layout](#boost_tagged_layout)
* [ElfUtils](#elfutils)
* [TBB](#tbb)
    * [correct linking](#tbb_correct_linking)
* [iberty](#iberty)
* [Capstone](#capstone)
	* [Building from source](#capstone_build_from_source)
***

Starting with Dyninst-10.1, the build system has been changed to be more coherent and consistent with the handling of user input. This both makes it easier to automate Dyninst builds for continuous integration testing and allows for exporting the Dyninst CMake cache configuration for use in the Testsuite to create a unified build. See [Building from source](Building-Dyninst#source_long) for details on required library versions.

<a name="sterile_build"/>

## Sterile Build

A sterile build requires that all dependencies are already installed on the system and thus will not be downloaded and built from source by the Dyninst build system. This is accomplished by using ```-DSTERILE_BUILD=ON``` when [building from source](Building-Dyninst#source_long). This is the default when [building with Spack](Building-Dyninst#spack).

<a name="boost"/>

## Boost

Basic options:

	Boost_ROOT_DIR            - Hint directory that contains the Boost installation
	PATH_BOOST                - Alias for Boost_ROOT_DIR
	Boost_MIN_VERSION         - Minimum acceptable version of Boost
	Boost_USE_MULTITHREADED   - Use the multithreaded version of Boost
	Boost_USE_STATIC_RUNTIME  - Use libraries linked statically to the C++ runtime.
	BOOST_INCLUDEDIR          - Hint directory that contains the Boost headers files
	BOOST_LIBRARYDIR          - Hint directory that contains the Boost library files

Advanced options:

	Boost_DEBUG               - Enable debug output from FindBoost
	Boost_NO_SYSTEM_PATHS     - Disable searching in locations not specified by hint variables 

Exports the following CMake cache variables

	Boost_ROOT_DIR            - Computed base directory the of Boost installation
	Boost_INCLUDE_DIRS        - Boost include directories
	Boost_INCLUDE_DIR         - Alias for Boost_INCLUDE_DIRS
	Boost_LIBRARY_DIRS        - Link directories for Boost libraries
	Boost_DEFINES             - Boost compiler definitions
	Boost_LIBRARIES           - Boost library files
	Boost_<C>_LIBRARY_RELEASE - Release libraries to link for component <C> (<C> is upper-case)
	Boost_<C>_LIBRARY_DEBUG   - Debug libraries to link for component <C>
	Boost_THREAD_LIBRARY      - The filename of the Boost thread library
	Boost_USE_MULTITHREADED   - Use the multithreaded version of Boost
	Boost_USE_STATIC_RUNTIME  - Use libraries linked statically to the C++ runtime.

**NOTE:**
>The exported ```Boost_ROOT_DIR``` can be different from the value provided by the user in the case that
it is determined to build Boost from source. In such a case, ```Boost_ROOT_DIR``` will contain the
directory of the from-source installation.

See the CMake file ```cmake/Modules/FindBoost.cmake``` in the Dyninst source directory for additional input and exported variables.

<a name="boost_tagged_layout"/>

### Tagged Layout Builds

Some systems build Boost using the tagged layout system where compile-time options used to build the libraries are encoded in the final library names. For example, ```libboost_system-mt.so``` is the ```boost::system``` library compiled in multithreaded mode. This is not turned on by default, and so should generally not be an issue. However, if your local Boost installation uses tagged layout, it is now fully supported as of Dyninst-10.1. See the CMake module ```cmake/Modules/FindBoost.cmake``` in the Dyninst source directory for a complete list of available options.

***

<a name="elfutils"/>

## ElfUtils

Basic options:

	ElfUtils_ROOT_DIR       - Base directory the of elfutils installation
	ElfUtils_INCLUDEDIR     - Hint directory that contains the elfutils headers files
	ElfUtils_LIBRARYDIR     - Hint directory that contains the elfutils library files
	ElfUtils_MIN_VERSION    - Minimum acceptable version of elfutils

Exports the following CMake variables

	ElfUtils_ROOT_DIR       - Computed base directory the of elfutils installation
	ElfUtils_INCLUDE_DIRS   - elfutils include directories
	ElfUtils_LIBRARY_DIRS   - Link directories for elfutils libraries
	ElfUtils_LIBRARIES      - elfutils library files

**NOTE:**
>The exported ```ElfUtils_ROOT_DIR``` can be different from the value provided by the user
in the case that it is determined to build elfutils from source. In such a case,
```ElfUtils_ROOT_DIR``` will contain the directory of the from-source installation.

See the CMake files ```cmake/Modules/FindLibElf.cmake``` and ```cmake/Modules/FindLibDwarf.cmake``` in the Dyninst source directory for additional input and exported variables.

***

<a name="tbb"/>

## Intel Threading Building Blocks (TBB)

Basic options:

	TBB_ROOT_DIR        - Hint directory that contains the TBB installation
	TBB_INCLUDEDIR      - Hint directory that contains the TBB headers files
	TBB_LIBRARYDIR      - Hint directory that contains the TBB library files
	TBB_LIBRARY         - Alias for TBB_LIBRARYDIR
	TBB_USE_DEBUG_BUILD - Use debug version of tbb libraries, if present
	TBB_MIN_VERSION     - Minimum acceptable version of TBB

Exports the following CMake cache variables

	TBB_ROOT_DIR        - Computed base directory of TBB installation
	TBB_INCLUDE_DIRS    - TBB include directory
	TBB_INCLUDE_DIR     - Alias for TBB_INCLUDE_DIRS
	TBB_LIBRARY_DIRS    - TBB library directory
	TBB_LIBRARY_DIR     - Alias for TBB_LIBRARY_DIRS
	TBB_DEFINITIONS     - TBB compiler definitions
	TBB_LIBRARIES       - TBB library files
	
	TBB_<c>_LIBRARY_RELEASE - Path to the release version of component <c>
	TBB_<c>_LIBRARY_DEBUG   - Path to the debug version of component <c>

**NOTE:**
>The exported ```TBB_ROOT_DIR``` can be different from the value provided by the user
in the case that it is determined to build TBB from source. In such a case,
```TBB_ROOT_DIR``` will contain the directory of the from-source installation.

See the CMake file ```cmake/Modules/FindTBB.cmake``` in the Dyninst source directory for additional input and exported variables.

<a name="tbb_correct_linking"/>

### Correctly linking TBB when built from source

Due to interoperability issues with the TBB build system when TBB is built from source, then only the location of the main TBB library (`libtbb.so` on linux) is entered into the rpath of the requesting Dyninst library. The `tbbmalloc` and `tbbmalloc_proxy` libraries are not entered. This can lead to incorrect versions of the `tbbmalloc` and `tbbmalloc_proxy` libraries being transitively loaded, if other versions are present on the system. The workaround for this is to add the `CMAKE_INSTALL_PREFIX/INSTALL_LIB_DIR` (see [Building and Installing](https://github.com/dyninst/dyninst/wiki/Building-Dyninst#building-and-installing) for details) directory to your `LD_LIBRARY_PATH` when linking to Dyninst or executing your program linked against Dyninst.

***

<a name="iberty"/>

## libiberty (from binutils)

Basic options:

	LibIberty_ROOT_DIR     - Base directory the of LibIberty installation
	LibIberty_LIBRARYDIR   - Hint directory that contains the LibIberty library files
	IBERTY_LIBRARIES       - Alias for LibIberty_LIBRARIES (backwards compatibility only)
	USE_GNU_DEMANGLER      - Use the GNU C++ name demanger (if yes, this disables using LibIberty)

Exports the following CMake cache variables

	LibIberty_FOUND        - True if headers and requested libraries were found
	IBERTY_FOUND           - Alias for LibIberty_FOUND (backwards compatibility only)
	LibIberty_LIBRARY_DIRS - Link directories for LibIberty libraries
	LibIberty_LIBRARIES    - LibIberty library files

**NOTE:**
>The exported ```LibIberty_ROOT_DIR``` can be different from the value provided by the user
in the case that it is determined to build LibIberty from source. In such a case,
```LibIberty_ROOT_DIR``` will contain the directory of the from-source installation.

***

<a name="capstone"/>

## Capstone

Basic options:

	Capstone_ROOT_DIR       - Base directory of the Capstone installation
	Capstone_INCLUDEDIR     - Hint directory that contains the Capstone headers files
	Capstone_LIBRARYDIR     - Hint directory that contains the Capstone library files
	Capstone_MIN_VERSION    - Minimum acceptable version of Capstone

Exports the following CMake cache variables

	Capstone_ROOT_DIR       - Base directory of the Capstone installation
	Capstone_INCLUDE_DIRS   - Capstone include directories
	Capstone_LIBRARY_DIRS   - Link directories for Capstone libraries
	Capstone_LIBRARIES      - Capstone library files

<a name="capstone_build_from_source"/>

### Building Capstone from source

```
$ git clone https://github.com/aquynh/capstone.git
$ cd capstone
$ git checkout next
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=/your/install/path -DCAPSTONE_ARCHITECTURE_DEFAULT=OFF -DCAPSTONE_X86_SUPPORT=ON -DCAPSTONE_BUILD_TESTS=OFF
$ make
$ [sudo] make install
```
