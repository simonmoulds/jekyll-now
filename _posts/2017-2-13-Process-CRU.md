---
layout: post
title: Process CRU TS 2.0 on Ubuntu 16.04 LTS
---

The CRU TS 2.0 dataset is widely used in regional and global hydrological modelling. Download the data [here](https://crudata.uea.ac.uk/cru/data/hrg/) or use the following wget command:

```bash
wget -r -np -nH --reject index.html* --cut-dirs=3 'https://crudata.uea.ac.uk/cru/data/hrg/cru_ts_2.02/data_dec/'

```

The raw data files need to be processed before they can be used in applications. A software tool called CRU2nc can be used for this purpose:

```bash
wget 'https://crudata.uea.ac.uk/cru/data/hrg/timm/grid/CRU2nc.tgz'
```

Compilation of the software is straightforward if the proprietary PGI Fortran compiler is installed on your system. Otherwise you will need to make a couple of changes to `Makefile.Linux` so that `gfortran` is used instead. So, change `pgf90 -o cru2nc cru2nc.o -L/distrib/local/netcdf/pgf/lib -lnetcdf` to `gfortran -o cru2nc cru2nc.o -L/usr/lib64 -lnetcdf -lnetcdff`. Next, change `pgf90 -c -I/distrib/local/netcdf/include cru2nc.f90` to `gfortran -c -I /usr/include cru2nc.f90`. Note that the paths following `-L` and `-I` should be the locations of the NetCDF library and include files, respectively.

Next, replace line 22 of the executable file `do_onlinux` with the following:

```bash
gzip -d -c ${dir}/obs.globe.lan.${YY}.${var}.Z > obs.globe.lan.${YY}.${var}.dat || exit 1
```

In the same file it will probably be necessary to change `dir` to the location of the decadal ascii files (e.g. `cru_ts_2.02/data_dec`). Once the necessary changes have been made, run the executable. This will generate a series of NetCDF files in the `data_dec` directory.




