# cblas (virtual package)

This is a virtual package to a concrete CBLAS implementation.

## Installation

Normally, `./configure && make install` would work just fine.  However, it may be necessary to pass in additional arguments to `./configure` if the CBLAS implementation cannot be detected automatically:

    ./configure CBLAS_CFLAGS=-I/usr/local/include CBLAS_LIBS=-lcblas

## FAQ

### What is a “virtual” package?

As a virtual package, it does not implement anything concrete.  Rather, the purpose of this package is to locate an existing CBLAS implementation and then figure out what compilation flags are necessary to utilize it.  The compilation flags are then stored into a [pkg-config file](https://www.freedesktop.org/wiki/Software/pkg-config/), which can be consumed by other programs.

### What do I need to use this package?

You need a concrete CBLAS implementation.  The “C” in CBLAS is to highlight the fact that a proper C interface is required.  Most BLAS implementations provide both a Fortran and C interface.  For the few that have provide a Fortran interface, it is possible to build a C interface via the [Netlib CBLAS wrapper](http://www.netlib.org/blas/#_cblas).

There are several ways to get a BLAS implementation:

  - On personal Linux systems, you will probably find it easiest to install [OpenBLAS](http://www.openblas.net/).  For most distributions, this is as easy as running your system's package manager (such as `apt-get`).
  - On Mac OS X, the [Accelerate framework](https://developer.apple.com/reference/accelerate) provides a CBLAS interface.
  - On Windows, install OpenBLAS (see above).
  - On a computing cluster, a BLAS implementation is usually provided by the vendors.  Netlib contains a [listing of common vendor implementations](http://www.netlib.org/blas/faq.html#_5_a_id_are_optimized_blas_libraries_available_where_can_i_find_vendor_supplied_blas_a_are_optimized_blas_libraries_available_where_can_i_find_optimized_blas_libraries).
  - Worst case, if none of the above solutions work, you can use the [original reference implementation on Netlib](http://www.netlib.org/blas/#_reference_blas_version_3_6_0) too.  But keep in mind it is very slow.
