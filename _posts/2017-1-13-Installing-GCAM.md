---
layout: post
title: Installing GCAM v4.3 on Ubuntu 16.04 LTS
---

The Global Change Assessment Model is a dynamic-recursive global integrated assessment model which links submodules representing the economy, energy system, land use, agriculture and climate. It is available under the terms of the [Educational Community License, Version 2.0](https://opensource.org/licenses/ECL-2.0). The official GCAM [documentation](http://jgcri.github.io/gcam-doc/toc.html) is an excellent resource which should provide a sufficiently detailed guide to install the software on Windows, Mac and Posix systems. The aim of this blog post is to provide a slightly more detailed guide for Linux users.

# Installation

## Get the code

The GCAM source code resides on Github. Here, we clone the GCAM project source code to a new directory which we'll call `GCAM`:

```bash
mkdir GCAM
cd GCAM
git clone https://github.com/JGCRI/gcam-core.git
```

Henceforth, the path to the `GCAM` directory will be referred to as `<GCAM Workspace>`. 

## Third party libraries

GCAM requires the following third-party libraries:

+ Boost
+ Xerces
+ Java

Make a directory `libs` in the GCAM workspace:

```bash
cd <GCAM Workspace>
mkdir gcam-core/libs
```

### Boost

Boost provides various general purpose utlities for the C++ language. I downloaded version 1.63.0 from [Boost](http://www.boost.org/users/download) and unpacked the compressed tarball to `<GCAM Workspace>/gcam-core/libs`. Installation was then as follows:

```bash
cd <GCAM Workspace>/gcam-core/libs/boost_1_63_0
./bootstrap.sh --with-libraries=system,filesystem --prefix=<GCAM Workspace>/gcam-core/libs/boost_1_63_0/stage/lib
./b2 stage
```

### Xerces

The Xerces XML parser is used by GCAM to read model inputs and configurations. I downloaded the latest C++ version of Xerces from [Apache](http://xerces.apache.org/#xerces2-j) and, again, unpacked the compressed tarball to `<GCAM Workspace>/libs`. Following the official GCAM guide, I set two environment variables before installation:

```bash
export XERCES_SRC=<GCAM Workspace>/gcam-core/libs/xerces-c-3.1.4
export XERCES_INSTALL=<GCAM Workspace>/gcam-core/libs/xercesc
cd $XERCES_SRC
./configure CFLAGS="-m64" CXXFLAGS="-m64" --prefix=$XERCES_INSTALL --disable-netaccessor-curl
make install
make clean
```

### Java

GCAM requires Java version 1.6 or newer in order to save model results in a BaseX XML database, which is written in Java. The Java Development Kit, rather than simply the Java Runtime Environment, should be installed. According to the official GCAM documentation both the Oracle version and OpenJDK should work. I used OpenJDK, which can be installed from the Ubuntu repositories:

```bash
sudo apt-get install openjdk
java -version
```

After installing Java it may be necessary to set the JAVA_HOME environment variable (such that JAVA_HOME/bin contains all executables). In my case this was as follows:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

## Java components

JAR files, which are inherently cross-platform, should be obtained from either the Windows or Mac [binary distributions](https://github.com/JGCRI/gcam-core/releases) and placed in `<GCAM Workspace>/libs/jars`:

```bash
cd `<GCAM Workspace>`
mkdir mac_binaries
cd mac_binaries
wget https://github.com/JGCRI/gcam-core/releases/download/gcam-v4.3/mac_binaries.tar.gz
tar -xvzf mac_binaries.tar.gz
cp -r libs/jars ../libs
```

## Hector

Hector (Hartin *et al.*, 2015) is a reduced-form climate carbon-cycle model which represents the most important earth system processes while running effectively instantaneously. Install the model as follows:

```bash
cd <GCAM Workspace>/gcam-core
make install_hector
```

## Building with Makefile

Set the following environment variables:

```bash
export GCAM_WORKSPACE=<GCAM Workspace>
export BOOST_INCLUDE=$GCAM_WORKSPACE/v4.3/gcam-core/libs/boost_1_63_0
export BOOST_LIB=$GCAM_WORKSPACE/v4.3/gcam-core/libs/boost_1_63_0/stage/lib
export XERCES_INCLUDE=$GCAM_WORKSPACE/v4.3/gcam-core/libs/xercesc/include
export XERCES_LIB=$GCAM_WORKSPACE/v4.3/gcam-core/libs/xercesc/lib
export JARS_LIB=$GCAM_WORKSPACE/v4.3/gcam-core/libs/jars/*
export JAVA_INCLUDE=$JAVA_HOME/include
export JAVA_LIB=$JAVA_HOME/jre/lib/amd64/server
```

Execute the following command to build GCAM:

```bash
cd <GCAM Workspace>/gcam-core/cvs/objects/build/linux
make gcam -j 8
```

## Running GCAM with simplified data system

To test the installation we run the model using the simplified GCAM data system, which is available from [Github](https://github.com/JGCRI/gcam-core/releases). Unpack the compressed tarball to `<GCAM Workspace>`, then replace `<GCAM Workspace>/gcam-core/input/gcam-data-system` with `<GCAM Workspace>/input/gcam-data-system`. If you want to use the full data system at a later date, save this to a new directory:

```bash
cd <GCAM Workspace>
mv gcam-core/input/gcam-data-system gcam-data-system_full
wget https://github.com/JGCRI/gcam-core/releases/download/gcam-v4.3/data-system.tar.gz
tar -xvzf data-system.tar.gz
cp -r input/gcam-data-system gcam-core/input
```

Finally, run the model:

```bash
cd <GCAM Workspace>/gcam-core/exe
cp configuration_ref.xml configuration.xml
./gcam.exe
```

## Running GCAM with full data system

The full data system relies on the 2012 edition of IEA energy balances for OECD and non-OECD countries, which is a proprietary dataset. Researchers based at UK higher education institutions can freely access the dataset through [the UK Data Service](https://www.ukdataservice.ac.uk/). The only way of obtaining the full dataset is by [contacting](https://www.ukdataservice.ac.uk/help/get-in-touch/accessing-data) UKDS directly. You should request the 2012 edition of the IEA World Energy Balances with separate files for OECD and non-OECD countries. After obtaining these files you will need to use the [Beyond 20/20 browser](http://www.statcan.gc.ca/eng/public/beyond20-20) to change the format of the dataset. Data should be exported in units of kilotonne of oil equivalent (ktoe). The official GCAM data system [documentation](http://jgcri.github.io/gcam-doc/data-system.html) provides further information about the required format of the IEA data files. 

## References

1. Hartin, C. A., Patel, P., Schwarber, A., Link, R. P., and Bond-Lamberty, B. P.: A simple object-oriented and open-source model for scientific and policy analyses of the global climate system – Hector v1.0, Geosci. Model Dev., 8, 939-955, [doi:10.5194/gmd-8-939-2015](http://www.geosci-model-dev.net/8/939/2015/), 2015.
