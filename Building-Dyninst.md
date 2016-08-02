To build Dyninst and all its components, 

    make && make install

from
the top-level directory of the source tree. To build and install a
single component and its dependencies, do the same thing from that
component's subdirectory. Libraries will be installed into

    CMAKE_INSTALL_PREFIX/INSTALL_LIB_DIR

and headers will be installed into 

    CMAKE_INSTALL_PREFIX/INSTALL_INCLUDE_DIR

If you wish to import
Dyninst into your own CMake projects, the export information is in

    CMAKE_INSTALL_PREFIX/INSTALL_CMAKE_DIR

PDF documentation is included
and installed to 

    CMAKE_INSTALL_PREFIX/INSTALL_DOC_DIR

If you update
the LaTeX source documents for any manuals, 

    make doc

will rebuild
them. Components may be built and installed individually: 

    make $COMPONENT

and

    make $COMPONENT-install

respectively; this will
appropriately respect inter-component dependencies.