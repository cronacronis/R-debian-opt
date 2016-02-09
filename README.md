Multiple versions of R on Ubuntu/Debian
==================

Very often we face a case in our laboratory to ensure stable execution of old legacy code. Of course we can always rewrite and test the code with new versions of R, but it implies significant amount of work. I believe, that support of multiple versions of R is cooler.

**The document provides easy way to setup multiple R environments on Ubuntu/Debian.**

# CRAN Repository

## Add repository

You can find the list of all cran repos [here](https://cran.r-project.org/mirmon_report.html). I have Ubuntu Precise installed.

Line below should be included in your `sources.list` in apt. 
```
deb http://stat.ethz.ch/CRAN/bin/linux/ubuntu precise/
```

*Please refer to [MANUAL UBUNTU](https://cran.r-project.org/bin/linux/ubuntu/README) and [MANUAL DEBIAN](https://cran.r-project.org/bin/linux/debian/) how to add CRAN repository.*


My status: I have version `2.14.1-1` installed in the system.
`$ sudo aptitude versions ^r-base-core$` shows following versions available 

```
i   2.14.1-1                                                      precise
...
p   3.1.3-1precise2                                               precise
...
p   3.2.3-4precise0                                               precise
```

## R version folder

We will proceed with version `3.1.3-1precise2`, thus, first there is a need to download it.
All further files for particular version will end up in the folder `/opt/R/3.1.3/`. Let's create it `sudo mkdir -p /opt/R/3.1.3/ && cd /opt/R/3.1.3/`.


# Unpack `deb` file

First download deb file from a CRAN repository.
```
$ sudo aptitude download r-base-core=3.1.3-1precise2
```
You should see file `r-base-core_3.1.3-1precise2_amd64.deb` in the `/opt/R/3.1.3/`
```
$ ls /opt/R/3.1.3/r-base-core_3.1.3-1precise2_amd64.deb 
/opt/R/3.1.3/r-base-core_3.1.3-1precise2_amd64.deb
```

Next step is to unpack deb file with R binaries and configuration files to custom folder.

```
$ sudo dpkg -x r-base-core_3.1.3-1precise2_amd64.deb /opt/R/3.1.3/
```
After that you should see unpacked folders in our newly created R folder.

```
vuser@ubuntu:/opt/R/3.1.3$ ls
etc  r-base-core_3.1.3-1precise2_amd64.deb  usr
```

# Change environment

This step is very important and and carries configuration changes needed for unpacked R to understand  whereto it is installed. We will look carefully now on file which executed R. When you launch R from terminal `$ R` it launches `/usr/bin/R`. It is easy to check with
```
vuser@ubuntu:/opt/R/3.1.3$ which R
/usr/bin/R
```
It is just a script, which prepares all needed variables and executes real R binary.
Our execution script file is in `/opt/R/3.1.3/usr/bin` and it is needed to be adjusted. In fact you can try to launch the `/opt/R/3.1.3/usr/bin/R` , but it will still load default R environment. In my particular setup it still loads default version `2.14.1-1`. It can be changed with adjustments of `/opt/R/3.1.3/usr/bin/R`

There are following variables needed to be adjusted:

```
R_HOME_DIR
R_HOME
R_SHARE_DIR
R_INCLUDE_DIR
R_DOC_DIR
```

This is example portion file with adjustments done:
```
#!/bin/bash
# Shell wrapper for R executable.

R_HOME_DIR=/opt/R/3.1.3/usr/lib/R
if test "${R_HOME_DIR}" = "/opt/R/3.1.3/usr/lib/R"; then
   case "linux-gnu" in
   linux*)
     run_arch=`uname -m`
     case "$run_arch" in.
        x86_64|mips64|ppc64|powerpc64|sparc64|s390x)
          libnn=lib64
          libnn_fallback=lib
        ;;
        *)
          libnn=lib
          libnn_fallback=lib64
        ;;
     esac
     if [ -x "/opt/R/3.1.3/usr/${libnn}/R/bin/exec/R" ]; then
        R_HOME_DIR="/opt/R/3.1.3/usr/${libnn}/R"
     elif [ -x "/opt/R/3.1.3/usr/${libnn_fallback}/R/bin/exec/R" ]; then
        R_HOME_DIR="/opt/R/3.1.3/usr/${libnn_fallback}/R"
     ## else -- leave alone (might be a sub-arch)
     fi
     ;;
  esac
fi
.
if test -n "${R_HOME}" && \
   test "${R_HOME}" != "${R_HOME_DIR}"; then
  echo "WARNING: ignoring environment value of R_HOME"
fi
R_HOME="${R_HOME_DIR}"
export R_HOME
R_SHARE_DIR=/opt/R/3.1.3/usr/share/R/share
export R_SHARE_DIR
R_INCLUDE_DIR=/opt/R/3.1.3/usr/share/R/include
export R_INCLUDE_DIR
R_DOC_DIR=/opt/R/3.1.3/usr/share/R/doc
export R_DOC_DIR
```

After this changes you can try to load it (**mind `./` before R**):
```
vuser@ubuntu:/opt/R/3.1.3/usr/bin$ ./R

R version 3.1.3 (2015-03-09) -- "Smooth Sidewalk"
Copyright (C) 2015 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)
```

# Installation of packages
By default our custom version of R will install packages in `/usr/local/lib/R/site-library`

The paths can be checked with command **`.libPaths()`**
```
> .libPaths();
[1] "/opt/R/3.1.3/usr/lib/R/site-library" "/usr/local/lib/R/site-library"      
[3] "/usr/lib/R/site-library"             "/usr/lib/R/library"                 
[5] "/opt/R/3.1.3/usr/lib/R/library"     
```

There are environment variables affecting these locations `R_LIBS_USER` and `R_LIBS_SITE`(as a non-empty colon-separated list of library trees).

```
export R_LIBS_USER=/opt/R/3.1.3/usr/lib/R/site-library
export R_LIBS_SITE=/opt/R/3.1.3/usr/lib/R/library
```

Let's get rid of `/usr/local/lib/R/site-library` `/usr/lib/R/site-library` `/usr/lib/R/library`

```
vuser@ubuntu:/opt/R/3.1.3/usr/bin$ export R_LIBS_USER=/opt/R/3.1.3/usr/lib/R/site-library
vuser@ubuntu:/opt/R/3.1.3/usr/bin$ export R_LIBS_SITE=/opt/R/3.1.3/usr/lib/R/library
```
After we set parameters we can execute R and check `.libPaths()`:
```
vuser@ubuntu:/opt/R/3.1.3/usr/bin$ ./R

R version 3.1.3 (2015-03-09) -- "Smooth Sidewalk"
Copyright (C) 2015 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)
...

> .libPaths();
[1] "/opt/R/3.1.3/usr/lib/R/site-library" "/opt/R/3.1.3/usr/lib/R/library"     
```

# Symbolic link for launch

`sudo ln -s /opt/R/3.1.3/usr/bin/R /usr/bin/R_3.1.3`
Then you can just launch R by `$ R-3.1.3`
