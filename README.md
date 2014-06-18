
# Overview


# Building


# Log Notes

- Found out a [ncdf4 repo on GitHub](https://github.com/cran/ncdf4)
- Forked [ncdf4, jmp75](https://github.com/jmp75/ncdf4)
- Added a .gitignore and .Rbuildignore


```cmd
%R% CMD BUILD ncdf4
%R% CMD check ncdf4_1.10.tar.gz
```

```
ncdf.c:3:20: fatal error: netcdf.h: No such file or directory
```

- Modify the Makefile.win to use the netcdf headers in a clone of the new netcdf-c now on github
- Entry point for ncdf4 documentation (CRAN link of the ncdf4 package brings a site under maintenance page) [here](http://cirrus.ucsd.edu/~pierce/ncdf/)
- It is mentioned that one cannot use Visual Studio netcdf libraries because R is build by mingw. Scratching my head here for a while; this is exactly what I am doing with rClr. However, I am forcing the compilation with Visual Studio instead of Mingw GCC, so the context is different. Be this as it may, I think this is possible to get something going by capitalizing on rClr.

# Planning next steps

- Build process for netcdf4-c from source. I did that on Windows and Linux 9 months ago: get back to it if the readme is not enough to get going. 
- modify the Makefile.win of ncdf4 to use MSbuild, but as an optional thing if MSVC is found.
- [D. Piercs's notes on ncdf4 for Win64](http://cirrus.ucsd.edu/~pierce/ncdf/how_to_build_on_windows.html)
- [netCDF CMake on Windows](http://www.unidata.ucar.edu/software/netcdf/docs/netCDF-CMake.html)
- [Getting CMake version 3 at time of writing](http://www.cmake.org/cmake/resources/software.html)
- [Need HDF5 to compile against. Binaries?](http://www.hdfgroup.org/HDF5/release/obtain5.html)

# Log of steps

```
F:\src\github\netcdf-c>cmake . -L
-- Building for: Visual Studio 12 2013
-- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
CMake Error at CMakeLists.txt:640 (FIND_PACKAGE):
  Could not find a package configuration file provided by "HDF5" with any of
  the following names:

    HDF5Config.cmake
    hdf5-config.cmake

  Add the installation prefix of "HDF5" to CMAKE_PREFIX_PATH or set
  "HDF5_DIR" to a directory containing one of the above files.  If "HDF5"
  provides a separate development package or SDK, be sure it has been
  installed.


-- Configuring incomplete, errors occurred!
See also "F:/src/github/netcdf-c/CMakeFiles/CMakeOutput.log".
```

Read through F:\src\github\netcdf-c\README.md
downloading F:\downloads\hdf5-1.8.13-win64-VS2012-shared.zip and F:\downloads\hdf5-1.8.13-win32-VS2012-shared.zip. Install.

hdf5-config.cmake is under:
F:\bin\HDF5\x64\cmake\hdf5\hdf5-config.cmake

We go a bit further with:

F:\src\github\netcdf-c>cmake . -L -DHDF5_DIR=f:/bin/HDF5/x64/cmake/hdf5
-- Building for: Visual Studio 12 2013
-- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
-- Unable to determine hdf5 version.  NetCDF requires at least version 1.8.6
-- Could NOT find ZLIB (missing:  ZLIB_LIBRARY ZLIB_INCLUDE_DIR)
CMake Error at CMakeLists.txt:732 (MESSAGE):
  HDF5 Support specified, cannot find ZLib.


-- Configuring incomplete, errors occurred!
See also "F:/src/github/netcdf-c/CMakeFiles/CMakeOutput.log".
-- Cache values
snip...
USE_HDF5:BOOL=ON
USE_NETCDF4:BOOL=ON
USE_SZIP:BOOL=OFF


Using http://www.unidata.ucar.edu/software/netcdf/docs/netCDF-CMake.html
``` bat
:: F:\src\github\netcdf-c
mkdir build
cd build
mkdir x64
cd x64
cmake ..\.. -L -DHDF5_DIR=f:/bin/HDF5/x64/cmake/hdf5 -DZLIB_LIBRARY=F:/bin/HDF5/x64/lib/zlib.lib  -DENABLE_DAP=OFF -DHDF5_HL_LIB=f:/bin/HDF5/x64/lib/hdf5_hl.lib -DHDF5_LIB=f:/bin/HDF5/x64/lib/hdf5.lib -DHDF5_INCLUDE_DIR=f:/bin/HDF5/x64/include/ -DZLIB_INCLUDE_DIR=f:/bin/HDF5/x64/include/ -G"Visual Studio 12 Win64" >  cmakeoutlog.txt 2>&1

:: Do NOT use the following msbuild, otherwise it will try the v11 compiler on v12 (VS2013) projects.
:: set MSB=C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe
:: Same msbuild seems possible to use irrespective of 32/64 bits.
set MSB="C:\Program Files (x86)\MSBuild\12.0\Bin\amd64\MSBuild.exe"
%MSB% netCDF.sln > buildoutlog.txt 2>&1

cd ..
mkdir x86
cd x86
cmake ..\.. -L -DHDF5_DIR=f:/bin/HDF5/x86/cmake/hdf5 -DZLIB_LIBRARY=F:/bin/HDF5/x86/lib/zlib.lib  -DENABLE_DAP=OFF -DHDF5_HL_LIB=f:/bin/HDF5/x86/lib/hdf5_hl.lib -DHDF5_LIB=f:/bin/HDF5/x86/lib/hdf5.lib -DHDF5_INCLUDE_DIR=f:/bin/HDF5/x86/include/ -DZLIB_INCLUDE_DIR=f:/bin/HDF5/x86/include/ -G"Visual Studio 12" >  cmakeoutlog.txt 2>&1

%MSB% netCDF.sln > buildoutlog.txt 2>&1
```

This compiles without error the solution, albeit with 700+ warnings however.

# Building ncdf4 C/C++ code with visual studio

```
cd F:\src\github_jm
rm -rf ncdf4.Rcheck
rm ncdf4_1.10.tar.gz
%R% CMD build ncdf4
%R% CMD check ncdf4_1.10.tar.gz
%R% CMD INSTALL --no-test-load ncdf4_1.10.tar.gz
```

```
xcopy F:\src\github\netcdf-c\build\x86\liblib\Debug\netcdf.* F:\Rlib\ncdf4\libs\i386\
xcopy F:\src\github\netcdf-c\build\x64\liblib\Debug\netcdf.* F:\Rlib\ncdf4\libs\x64\
xcopy F:\bin\HDF5\x86\bin\*.dll F:\Rlib\ncdf4\libs\i386\
xcopy F:\bin\HDF5\x64\bin\*.dll F:\Rlib\ncdf4\libs\x64\

```

