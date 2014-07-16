# Overview

This README explains how to build the ncdf4 R package and the netcdf-c library it depends on, 
for both 32/64 bits architectures with Windows operating systems, using the Visual C++ toolchain. 

The CRAN page for the ncdf4 package directs at the time of writing to [a site under maintenance page](http://dwpierce.com/software). 
The entry point for ncdf4 documentation appears to be [here](http://cirrus.ucsd.edu/~pierce/ncdf/). 
It is mentioned in [D. Pierce's notes on ncdf4 for Win64](http://cirrus.ucsd.edu/~pierce/ncdf/how_to_build_on_windows.html) that 
one cannot use netcdf libraries built with Visual Studio because R is build using mingw. I think this means it is at least difficult 
to compile using the mingw gcc compiler. 

This fork/branch [jmp75/ncdf4 at devel](https://github.com/jmp75/ncdf4/tree/devel) takes a different approach and uses the Visual C++ compiler.

# Building

The source code of the C API of netCDF can be found [in this github repo](https://github.com/Unidata/netcdf-c). The README has a link to compile
[netCDF with CMake on Windows](http://www.unidata.ucar.edu/software/netcdf/docs/netCDF-CMake.html). One optional requirement, 
which is used in this documentation, is the HDF5 Libraries for netCDF4/HDF5 support. 

You should also read through [D. Pierce's notes on ncdf4 for Win64](http://cirrus.ucsd.edu/~pierce/ncdf/how_to_build_on_windows.html)

You need RTools, as appropriate for your version of R, to build the ncdf4 package. The ncdf4 Makefile.win also uses the command `cp` that comes 
with RTools, so you need its directory in your PATH environment variable.

## Installing CMake

CMake can be downloaded from [here](http://www.cmake.org/cmake/resources/software.html). As of July 2014, this is CMake version 3. 
You may want to add cmake to your PATH environment variable. 

## Installing HDF5

### Installing HDF5 binaries

There is a page on [obtaining HDF5](http://www.hdfgroup.org/HDF5/release/obtain5.html) as source code or precompiled binaries. 
Downloading the binary packages hdf5-1.8.13-win64-VS2012-shared.zip hdf5-1.8.13-win32-VS2012-shared.zip. 

Install the binaries in your choice of location, F:\bin\HDF5\x64 and F:\bin\HDF5\x86. hdf5-config.cmake for 64 bits is under 
F:\bin\HDF5\x64\cmake\hdf5\hdf5-config.cmake. The location of this config file must be passed to cmake when building netcdf-c

### Building HDF5 binaries

Placeholder; not documented yet.

## Building netCDF-c

Once you have installed or compiled HDF5, you can compile netcdf-c with HDF5 support enabled.

Clone the git [netcdf-c repo](https://github.com/Unidata/netcdf-c)

Other options than HDF5 and ZLIB mentioned in [netCDF with CMake on Windows](http://www.unidata.ucar.edu/software/netcdf/docs/netCDF-CMake.html) are set to OFF.

Note that the options arguments to the `-G` option of cmake are not obvious, and its use not explicitely specified in the "netCDF with CMake" document. 
I think I figured out by looking through the netCDF mailing lists.

```cmd
:: F:\src\github\netcdf-c
:: set BuildConfiguration=Debug
set BuildConfiguration=Release
mkdir build
cd build
mkdir x64
cd x64
set HDF5_BIN_DIR=f:/bin/HDF5
cmake ..\.. -L -DHDF5_DIR=%HDF5_BIN_DIR%/x64/cmake/hdf5 -DZLIB_LIBRARY=F:/bin/HDF5/x64/lib/zlib.lib  -DENABLE_DAP=OFF -DHDF5_HL_LIB=%HDF5_BIN_DIR%/x64/lib/hdf5_hl.lib -DHDF5_LIB=%HDF5_BIN_DIR%/x64/lib/hdf5.lib -DHDF5_INCLUDE_DIR=%HDF5_BIN_DIR%/x64/include/ -DZLIB_INCLUDE_DIR=%HDF5_BIN_DIR%/x64/include/ -G"Visual Studio 12 Win64" >  cmakeoutlog.txt 2>&1
:: Do NOT use the following msbuild, otherwise it will try the v11 compiler on v12 (VS2013) projects.
:: set MSB=C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe
:: Same msbuild seems possible to use irrespective of 32/64 bits.
set MSB="C:\Program Files (x86)\MSBuild\12.0\Bin\amd64\MSBuild.exe"
%MSB% netCDF.sln /p:Configuration=%BuildConfiguration% /p:Platform=x64 > buildoutlog.txt 2>&1
cd ..
mkdir x86
cd x86
cmake ..\.. -L -DHDF5_DIR=%HDF5_BIN_DIR%/x86/cmake/hdf5 -DZLIB_LIBRARY=F:/bin/HDF5/x86/lib/zlib.lib  -DENABLE_DAP=OFF -DHDF5_HL_LIB=%HDF5_BIN_DIR%/x86/lib/hdf5_hl.lib -DHDF5_LIB=%HDF5_BIN_DIR%/x86/lib/hdf5.lib -DHDF5_INCLUDE_DIR=%HDF5_BIN_DIR%/x86/include/ -DZLIB_INCLUDE_DIR=%HDF5_BIN_DIR%/x86/include/ -G"Visual Studio 12" >  cmakeoutlog.txt 2>&1
%MSB% netCDF.sln /p:Configuration=%BuildConfiguration% /p:Platform=Win32 > buildoutlog.txt 2>&1
```

This compiles without error the solution, albeit with 700+ warnings however.


# Building ncdf4 C/C++ code with visual studio

To set up the build such that it uses VC++:

```
cd F:\src\github_jm\ncdf4\src
create_lib.cmd
cp Makefile.win.in Makefile.win
```

`create_lib.cmd` creates the file Rdll.lib necessary for VC++ to know what functions are exported by R.dll. Makefile.win overrides the 
default behavior of R i.e. to look for source files and compile each.

You need to specify two locations in the file src\ncdf4.props, so that the compilation finds the right header and libraries of dependencies. 
You should use the "short path" format, and must do so for paths with blanks in the directory names.

```xml
<NetcdfInstallPath>F:/src/github/netcdf-c</NetcdfInstallPath>
<RInstallPath>C:/PROGRA~1/R/R-31~1.0</RInstallPath>
```

You also need to set again NetcdfInstallPath, which is also used outside of the compilation VC++, by Makefile.win, to copy binaries.

```
cd F:\src\github_jm
rm -rf ncdf4.Rcheck
```

As of July 2014, you have to install from the source package, not a tarball. 
This is "just" a question of not having the time to polish the logistical aspects, not a limitation.

```
set NetcdfInstallPath=F:/src/github/netcdf-c
set Hdf5InstallPath=F:/bin/HDF5
set R="c:\Program Files\R\R-3.1.0\bin\x64\R.exe"
%R% CMD check ncdf4
:: And/or
%R% CMD INSTALL ncdf4
```

Noticed that running one of the examples worked on 32 bits but same thing failed on 64 bits. Compiled with Visual studio then, to have a debuggable ncdf4.dll; surprise it then all works.

If you are using the `devtools` package in R:

```R
library(devtools)
Sys.setenv(NetcdfInstallPath='F:/src/github/netcdf-c', Hdf5InstallPath='F:/bin/HDF5')
document('f:/src/github_jm/ncdf4', roclets = c("collate", "rd")) # leave "namespace" and "rd" out of the roclets options.
build('f:/src/github_jm/ncdf4')
```

```
cd F:\src\github_jm
set NetcdfInstallPath=F:/src/github/netcdf-c
set Hdf5InstallPath=F:/bin/HDF5
set R="c:\Program Files\R\R-3.1.0\bin\x64\R.exe"
%R% CMD INSTALL ncdf4_1.12-1.tar.gz
```



# Log Notes

For the record, where I started from using David Pierce's work

- Found out a [ncdf4 repo on GitHub](https://github.com/cran/ncdf4)
- Forked [ncdf4, jmp75](https://github.com/jmp75/ncdf4)
- Added a .gitignore and .Rbuildignore

# TODO

The ncdf4 package can be compiled from a tarball with VC++ as the compiler, i.e. something like the following.

```
%R% CMD build ncdf4
set NetcdfInstallPath=F:/src/github/netcdf-c
set Hdf5InstallPath=F:/bin/HDF5
%R% CMD check ncdf4_1.10.tar.gz
%R% CMD INSTALL --no-test-load ncdf4_1.10.tar.gz
```

This means fully duplicating (refactoring...) the "configure" mechanisms found in the [rClr package](https://rclr.codeplex.com).
