---
layout: post
title: Local install of netCDF with parallel mode enabled
---
On our group server we have a global netCDF4 installation compiled without parallel I/O support. In the past we have found that attempting to reinstall netCDF in parallel has broken software which was compiled against the original library (e.g. GDAL). To avoid this issue, I made a local netCDF installation with parallel I/O support enabled. The specific application I had in mind was to build the land surface model JULES with parallel support, which requires a static build of the netCDF library. The basic guide I followed is the [Quick Instructions for Installing NetCDF on Unix](https://www.unidata.ucar.edu/software/netcdf/netcdf-4/newdocs/netcdf-install/Quick-Instructions-for-Installing-NetCDF-on-Unix.html).

# Build zlib
[zlib](https://en.wikipedia.org/wiki/Zlib) is a compression library used by netCDF. According to the guide above, netCDF-4 requires version 1.2.3 or better. I used version 1.2.11, which seemed to work fine. Download and unpack the source code:
```bash
wget https://www.zlib.net/zlib-1.2.11.tar.gz
tar -xvf zlib-1.2.11.tar.gz
```
Enter the source code directory and run the configure script, using the `--prefix` option to specify the installation directory. Then install the software in this location using `make`, as shown below:
```
cd zlib-1.2.11
./configure --prefix=$HOME/local/zlib
make
make install
```
Add the installation directory to your `PATH` variable by adding `export PATH=$HOME/local/zlib:$PATH` to your `~/.bashrc` file, then make this available in your current session by entering `source ~/.bashrc` at the command line.

# Install MPI
JULES uses MPI (Message Passing Interface) to facilitate parallel processing. In addition, JULES exploits the parallel I/O support in HDF5/netCDF4, which also use MPI. There are two common implementations of MPI: OpenMPI and MPICH. I've used OpenMPI here, but the steps in the rest of the post should be the same or similar if you prefer to use MPICH.

According to this (post)[https://forum.hdfgroup.org/t/hdf5-and-openmpi/5437], some failures have been observed when HDF5 is installed with certain versions of OpenMPI. So, if OpenMPI is already installed then you should check which version is available by entering `mpirun --version` at the command line. Using version 4.0.1 I found a particular test in `make check` (t_bigio) was hanging, so I made a local install of OpenMPI 4.0.0 and compiled HDF5 against this version instead, which seemed to solve the problem. Download and unpack the source code of this version: 
```bash
wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.0.tar.gz
tar -xvf openmpi-4.0.0.tar.gz
```
Installing OpenMPI is straightforward:
```bash
cd openmpi-4.0.0
./configure --prefix=$HOME/local/openmpi
make check
make install
```
Again, add the installation directory to your `PATH` using the procedure outlined previously.

# Build HDF5
[HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) is a file format designed for large amounts of data which is used as the data storage layer in netCDF4. To install `HDF5`, point to the C compiler you have just installed and enable parallel support with the `--enable-parallel` flag:
```bash
CC=$HOME/local/openmpi/bin/mpicc ./configure --with-zlib=$HOME/local/zlib --enable-parallel --prefix=$HOME/local/netcdf
make check
make install
```
The `release_docs/INSTALL_parallel` file contains two sample programs in C and Fortran 90, respectively, which should be used to check the installation. Simply copy the programs into their own files (`Sample_mpio.c` and `Sample_mpio.f90`) and compile using the C/Fortran compilers in your OpenMPI installation. We can run the compiled programs using `mpiexec`, using the `-np` option to specify the number of processes to use:
```bash
mpicc Sample_mpio.c -o c.out
mpiexec -np 4 c.out
```
and
```
mpif90 Sample_mpio.f90 -o f.out
mpiexec -np 4 c.out
```
These programs should print some output which verifies the tests have completed successfully.

# Build netCDF
Having successfully installed `HDF5` we can move on to install `netCDF`. We will do this in two parts: first we install the C netCDF library, then the Fortran library. Installing `netCDF` in parallel is not difficult so long as you know the various configuration options that are needed. The steps outlined below worked for me, but I cannot guarantee they will work for you. I found the Unidata helppages really helpful, especially [Getting and Building netCDF](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html#build_parallel) and [Building the NetCDF-4.2 and later Fortran libraries](https://www.unidata.ucar.edu/software/netcdf/docs/building_netcdf_fortran.html). These pages are a good place to start if you encounter any problems with the steps outlined below, or want to adapt them for your own purposes.

We first install the `netCDF` C library. Download and unpack the source code:
```bash
wget https://www.unidata.ucar.edu/downloads/netcdf/ftp/netcdf-c-4.6.3.tar.gz
tar -xvf netcdf-c-4.6.3.tar.gz
cd netcdf-c-4.6.3
```
To configure netCDF-C correctly we need to link to our `HDF5` installation. Assuming HDF5 has been correctly built with parallel I/O, netCDF will inherit support for parallel I/O, so no special options need to be passed to the configuration. That said, use the `--enable-parallel-tests` option to enable `make check` to test the parallel I/O support. To install netCDF-C, do the following:
```bash
H5DIR=$HOME/local/netcdf CC=$HOME/local/openmpi/bin/mpicc CPPFLAGS=-I${H5DIR}/include LDFLAGS=-L${H5DIR}/lib ./configure --disable-shared --enable-parallel-tests --prefix=$HOME/local/netcdf
make check
make install
```
Note that one of the parallel tests may fail if the number of threads available to you is less than 16. This is a [known issue](https://github.com/Unidata/netcdf-c/issues/932) which I don't think is that important, because in most circumstances you will specify the number of cores to use in a specific application. 

Installing the `netCDF` Fortran library follows a similar process. First download the source code:
```bash
wget https://www.unidata.ucar.edu/downloads/netcdf/ftp/netcdf-fortran-4.4.5.tar.gz
tar -xvf netcdf-fortran-4.4.5.tar.gz
cd netcdf-fortran-4.4.5
```
With the Fortran library we must link to the netCDF-C library we have just built, as well as specifying the location of the MPI Fortran 77/90 compilers:
```bash
NCDIR=$HOME/local/netcdf H5DIR=$HOME/local/netcdf CPPFLAGS="-I${NCDIR}/include" CC=$HOME/local/openmpi/bin/mpicc FC=$HOME/local/openmpi/bin/mpif90 F77=$HOME/local/openmpi/bin/mpif77 LDFLAGS="-L${NCDIR}/lib" LD_LIBRARY_PATH=${H5DIR}/lib LIBS="-lnetcdf -lhdf5_hl -lhdf5 -lm -ldl -lsz -lz -lcurl" ./configure --disable-shared --prefix=$HOME/local/netcdf
make check
make install
```
