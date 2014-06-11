
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
