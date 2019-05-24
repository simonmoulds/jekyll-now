---
layout: post
title: Get started with JULES
---

The Joint UK Land and Environment Simulator (JULES) is a community land surface model (See [Wikipedia](https://en.wikipedia.org/wiki/Land_surface_models_(climate)) for a general definition). In this post I will go through the process of installing JULES in serial mode on your computer. The contents are adapted from Toby Marthew's excellent 'JULES From Scratch' [tutorial](https://jules.jchmr.org/content/scratch). 

# Prerequisites
1. A MOSRS login
2. An internet connection
3. Access to a UNIX/Linux system which has the following installed:
   * a Fortran compiler (e.g. gfortran)
   * Python (>=2.0)
   * netCDF libraries
   * Git version control system
   * Subversion (>=1.9)
   * Your favourite text editor (e.g. emacs, vi)
    
# Install software requirements
There are three programs which you must have installed in order to run JULES. These are Cylc, Rose, and FCM. Before starting to install these dependencies, you will need to look up the Rose [change log](https://github.com/metomi/rose/blob/master/CHANGES.md) and select the versions of Cylc and FCM which are compatible with the most recent version of Rose (e.g. Rose 2019.01.0 is compatible with cylc-7.8.1 and fcm-2017.10.0). 

We will be installing the three software packages locally (as opposed to globally, which would require root priveleges), so our first task is to create a local installation directory. This can have any name, but the convention is either $HOME/local or $HOME/.local (prefixing with a dot means the directory is hidden - in other words, it will only display if you use `ls -a`). Let's do the following:

    mkdir $HOME/local
    cd $HOME/local
    
It is sensible to add this directory to your `PATH` environment variable, which specifies a set of directories where executable programs reside. This can be set from the command line, but it is more convenient to add it to the `~/.bashrc` settings file so that it is updated each time you start a new bash shell. Open this file with your favourite text editor and add the following line `export PATH=$HOME/local/cylc/bin:$PATH`, before saving and exiting.

## cylc
[Cylc](https://cylc.github.io/) is an open-source [workflow engine](https://en.wikipedia.org/wiki/Workflow_engine) specifically designed for cycling systems. We first obtain a copy of the Cylc source code from GitHub, as follows:

    git clone https://github.com/cylc/cylc.git
    cd cylc
    git tag -l

With the final command we list the available Cylc versions (in Git terminology, tags are references to specific points in a project's Git history which are typically used to mark version releases). The following command checks out (updates the files in the repository) to match the specified version:

    git checkout tags/7.6.0

That's it for Cylc. All that remains is to add the cylc directory to your `PATH`, using a text editor to add the line `export PATH=$HOME/local/cylc/bin:$PATH` to your `~/.bashrc` file. After sourcing this file Cylc should be available everywhere on your system. Let's verify that everything is working:

    source ~/.bashrc
    cd
    cylc --version

If the last command prints the correct version then cylc has been successfully installed.

## Rose
According to its documentation, "Rose is a toolkit for writing, editing and running application configurations." In essence, Rose is a wrapper to Cylc which is designed to make that software more user friendly.

The installation procedure is similar to that of Cylc. So, we download the source code to our local installation directory, then checkout the specific version we need:

    cd ~/local
    git clone https://github.com/metomi/rose.git
    cd rose
    git tag -l
    git checkout tags/2019.01.0

As before, we need to add `export PATH=$HOME/local/rose/bin:$PATH` to our `$HOME/.bashrc` file so that the Rose executable files are on our `PATH`. Once you source this file Rose should be available anywhere on your system:

    source ~/.bashrc
    cd
    rose --version

Assuming this works as expected we can move on to configure Rose for our system, which is done using two text files: `~/.metomi/rose.conf` and `~/local/rose/etc/rose.conf`. 

The first configuration file contains your MOSRS username. Create it as follows:

    mkdir ~/.metomi
    cd ~/.metomi
    touch rose.conf

Then use a text editor to add the following lines at the top of the file:

    [rosie-id]
    prefix-username.u = simonmoulds

Note that if you are doing [rose stem testing](https://code.metoffice.gov.uk/trac/jules/wiki/TestingJULESChanges) (i.e. you are doing code development) then you need an additional entry to this file. See the 'Get Rose' [tutorial](https://jules.jchmr.org/content/get-rose) on the JULES website for more information about this.

The second file sets the remote location of rose suites. Create the file with the command `touch ~/local/rose/etc/rose.conf`, then add the following text:

    [rosie-id]
    prefix-default=u
    prefixes-ws-default=u
    prefix-location.u=https://code.metoffice.gov.uk/svn/roses-u
    prefix-web.u=https://code.metoffice.gov.uk/trac/roses-u/intertrac/source:
    prefix-ws.u=https://code.metoffice.gov.uk/rosie/u

Once these two files have been created we're ready to check the Rose installation. Entering the command `rosie hello` at the command line should prompt you for your MOSRS login. Entering this should then return `hello <username>` from the Met Office server.

## FCM
FCM, which is an acronym for Flexible Code Management, performs two tasks:
1. Handles the JULES version control system
2. Compiles the JULES model code
FCM requires that Subversion (>=1.9.0) is also installed on your machine. While it's likely that Subversion will be available, it is by no means guaranteed that you will have the correct version. You can check by entering `svn --version` at the command line. If the version is older than 1.9 then you will need to install a more recent one. Unless you have root access you will need to install the software from source in your local directory:

    wget https://www-us.apache.org/dist/subversion/subversion-1.9.10.tar.gz
    tar -xvf subversion-1.9.10.tar.gz
    cd subversion-1.9.10
    ./configure --prefix=~/local/svn
    make
    make install

This should work if `~/local/svn` is on your `PATH` (follow the procedure outlined for Cylc and Rose, above, to add it).

The first step to install FCM is the same as Cylc and Rose:

    cd ~/local
    git clone https://github.com/metomi/fcm.git
    cd fcm
    git tag -l
    git checkout tags/2017.10.0

Add `export PATH=$HOME/local/fcm/bin:$PATH` to your `~/.bashrc` file, and check the installation with `fcm --version`.

The next step is to enable Subversion access to MOSRS, which you can do by modifying the `~/.subversion/servers` file. You may not have this file on your machine if you haven't used Subversion before. Check by doing `ls ~/.subversion/servers`; If this returns `No such file or directory` then you will need to create the file:

    mkdir ~/.subversion
    touch ~/.subversion/servers

Open the file and add the following text, substituting `simonmoulds` for your MOSRS username:

    [groups]
    metofficesharedrepos = code*.metoffice.gov.uk
 
    [metofficesharedrepos]
    username = simonmoulds
    store-plaintext-passwords = no

If you already have the `~/.subversion/servers` file then you may well already have a `[groups]` section. If this is the case, add `metofficesharedrepos = code*.metoffice.gov.uk` to this section, adding the `[metofficesharedrepos]` section beneath. Leave the rest of the file unmodified.

Lastly, we need to set up keywords for the JULES repositories. Create the file `~/.metomi/fcm/keywords.cfg` and add the following text:

    location{primary, type:svn}[jules.x] = https://code.metoffice.gov.uk/svn/jules/main
    browser.loc-tmpl[jules.x] = https://code.metoffice.gov.uk/trac/{1}/intertrac/source:/{2}{3}
    browser.comp-pat[jules.x] = (?msx-i:\A // [^/]+ /svn/ ([^/]+) /*(.*) \z)

    location{primary, type:svn}[jules_doc.x] = https://code.metoffice.gov.uk/svn/jules/doc
    browser.loc-tmpl[jules_doc.x] = https://code.metoffice.gov.uk/trac/{1}/intertrac/source:/{2}{3}
    browser.comp-pat[jules_doc.x] = (?msx-i:\A // [^/]+ /svn/ ([^/]+) /*(.*) \z)

## Cache password
Before starting this process, consider the warning on Toby Marthews' [guide](https://jules.jchmr.org/content/get-fcm) for caching your MOSRS password:

> CACHING YOUR MOSRS PASSWORD may be problematic on Fedora 23 systems and/or if your MOSRS password has special characters in it: please email JULES support if you encounter problems

There are two methods for caching your password: 
1. GNOME Keyring (a software application for storing security credentials), 
2. gpg-agent

This guide focuses on using gpg-agent and is based on the official Met Office [guide](https://code.metoffice.gov.uk/trac/home/wiki/AuthenticationCaching/GpgAgent#no1). First, download the two scripts (`mosrs-setup-gpg-agent` and `mosrs-cache-password`) from the previous link. Unfortunately `wget` doesn't work for these files (or at least I don't know the combination of options necessary to get it to work), so you may need to download them using your browser. 

Copy the two files to your local installation directory (which should be on your `PATH`), change the file permissions to make them executable, and run `mosrs-setup-gpg-agent`:

    cd ~/local
    chmod 777 ~/local/mosrs-setup-gpg-agent
    chmod 777 ~/local/mosrs-cache-password
    . mosrs-setup-gpg-agent

This should prompt you to enter your MOSRS password more than once (three, in my case). If the password has been cached properly the following commands should work without prompting you to enter your password:

    rosie hello
    svn info --username=simonmoulds https://code.metoffice.gov.uk/svn/test

The password will need to be cached every time you log in to your machine. To do this automatically, add the following lines to your `~/.bashrc` file:

    [[ "$-" != *i* ]] && return # Stop here if not running interactively
    . mosrs-setup-gpg-agent

## Download JULES
Choose a sensible location for the JULES source code. I've chosen `$HOME/models`:

    cd
    mkdir models
    cd models

Now we use FCM to download a version of JULES. This can either be a trunk (released) or branch (under development). 

To download a trunk use the following command, replacing `X.X` with a real version (e.g. `5.4`) and `<dir>` with an instructive location for the downloaded source code (e.g. `jules-vn5.4`):

    fcm co fcm:jules.x_tr@vnX.X <dir>

If the `@vnX.X` is omitted then fcm will automatically download the latest version.

To download a branch:

    fcm co fcm:x_br/dev/<user>/<branch-name> <dir>

(I've never done this).

Another warning from Toby Marthews:

> If you see a message about the password for the 'authentication realm' then DO NOT agree to store the password unencrypted: please check that your MOSRS password has been cached correctly

In the next section we refer to `<dir>` as `$JULES_ROOT`.

## Compile JULES
If you intend to run JULES through Rose there is no need to compile it manually, because it will be done automatically when you run a Rose suite. However, it may be still be a useful exercise in order to check that JULES correctly links to the `netCDF` libraries. If you want to do this then you will need to modify the FCM configuration file (`$JULES_ROOT/etc/fcm-make/make.cfg`) so that the following variables are correctly assigned according to your system:

    $JULES_COMPILER{?}        = gfortran
    $JULES_BUILD{?}           = normal
    $JULES_OMP{?}             = noomp
    $JULES_MPI{?}             = nompi
    $JULES_NETCDF{?}          = netcdf
    $JULES_NETCDF_PATH{?}     =
    $JULES_NETCDF_INC_PATH{?} = /usr/lib64/gfortran/modules
    $JULES_NETCDF_LIB_PATH{?} = /usr/lib64

The `JULES_NETCDF_INC_PATH` is the location of `netcdf.mod` and `JULES_NETCDF_LIB_PATH` is the location of `libnetcdf.a` and/or `libnetcdff.a`. If you're not sure of the location of these files then you can use a tool such as `locate`, for example:

    locate netcdf.mod

Once these variables are correctly set we can compile the model using `fcm make`, as follows:

    cd $JULES_ROOT
    fcm make -j 2 -f etc/fcm-make/make.cfg --new

After a few minutes you will hopefully see `[done] make`, which indicates that compilation was successful. Verify the executable has been created:

    ls $JULES_ROOT/build/bin

Having successfully compiled the model we know that JULES correctly links to the `netCDF` libraries. Make a note of the values of the variables in the FCM configuration file, because you will need to set these when you use Rose (albeit in a slightly different way). Equally, if you ever change your `netCDF` installation (such as installing a version with parallel I/O support) I suggest attempting a manual compilation of JULES to ensure it correctly links to the new libraries.
