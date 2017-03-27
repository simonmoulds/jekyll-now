---
layout: post
title: Maintain several versions of R on Linux
---

A common experience for R developers is the need to test packages on the latest R development version as well as the latest stable release. In this blog post I show how to maintain multiple versions of R on the same system. By way of illustration I will install the latest stable release (3.3.3, Another Canoe) and the development version of R on my Ubuntu 16.04 LTS machine. A similar procedure should work on most Linux distributions.

First, download the source code for the two versions of R from [CRAN](https://cran.r-project.org/) and unpack the tarballs in your working directory:

```bash
cd ~/Downloads
tar -xvzf R-3.3.3.tar.gz
tar -xvzf R-devel_2017-3-26.tar.gz
```

Each version of R should be installed in a unique location by setting the `--prefix` flag, as follows:

```bash
cd R-3.3.3
./configure --prefix=/usr/local/lib/R-3.3.3
make
make install

cd ../R-devel
./configure --prefix=/usr/local/lib/R-devel
make
make install
```

Next, define symbolic links to the R script of the respective installations somewhere on your PATH, like so:

```bash
cd /usr/local/bin
ln -sf /usr/local/lib/R-3.3.3/bin/R R-3.3.3
ln -sf /usr/local/lib/R-devel/bin/R R-devel
```

After creating these files it should be possible to start the different versions of R from the command line by typing either `R-3.3.3` or `R-devel` and pressing ENTER. Accessing the different installations in [ESS](https://ess.r-project.org/) should also be straightforward. According to the [manual](http://ess.r-project.org/Manual/ess.html):

> R on Unix systems: If you have "R-1.8.1" on your exec-path, it can be started using M-x R-1.8.1. By default, ESS will find versions of R beginning "R-1" or "R-2". If your versions of R are called other names, consider renaming them with a symbolic link or change the variable ess-r-versions. To see which functions have been created for starting different versions of R, type M-x R- and then hit [Tab]. These other versions of R can also be started from the "ESS->Start Process->Other" menu. 

Nowadays, in fact, ESS will find versions of R beginning "R-1", "R-2", "R-3", "R-devel" and "R-patched". To start R from Emacs, therefore, type `M-x R-3.3.3` or `M-x R-devel` as required. 
