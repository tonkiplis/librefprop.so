#Welcome to librefprop.so!
These files allow you to compile the REFPROP fluid property database as a shared library for Linux and MacOS systems. This enables you to use the Fortran sources developed by NIST providing an alternative to the REFPROP.dll for Windows. 

## Installation Instructions
For installation on a Linux or OSX machine, please follow the steps described below. By default, the library and the header file are placed in system directories. Please change the paths if you do not have write access to this part of your file system. 

0.  Make sure that you have gcc and gfortran, for OSX use either [HPC](http://hpc.sourceforge.net/) **or** [Homebrew](http://brew.sh/) **and** install the [OSX command line tools](https://developer.apple.com/downloads). On a Linux machine, something like `apt-get install gcc` might do the job.
1.  Get a copy of this repository, either by downloading the latest [release](https://github.com/jowr/librefprop.so/releases/latest) or the current development version as [zip file](https://github.com/jowr/librefprop.so/archive/master.zip) or simply clone the [repository](https://github.com/jowr/librefprop.so.git) with git.
2.  Change the paths in the Makefile, if needed.
3.  Copy the REFPROP Fortran code to the *fortran* directory.
4.  Put the *fluids* and *mixtures* folders from REFPROP into the *files* folder.
5.  Call `make` to prepare the files. 
6.  Use `make install` (as root user) to copy the files to the destination directories.

You can remove the files again by calling `make uninstall` (as root user). 

## Testing the Installation
There is a simple Fortran file to test the library. You can call `make fortest` and run the executable with `./bin/ex_mix_for` to display some R410 two-phase properties:

| Temperature | Pressure  | Density, liquid | Density, vapour |
|-------------|-----------|-----------------|-----------------|
| 300.0000    | 1740.5894 |   14.4550       |   0.9628        |
| 300.0000    | 1735.1589 |   14.2345       |   0.9603        |


## Python Integration
There is a basic python package based on the examples from
[NIST](http://www.boulder.nist.gov/div838/theory/refprop/Frequently_asked_questions.htm#PythonApplications "NIST homepage")
in the `pyrp` folder. 

Please note that there is a much more mature Python interface available at https://github.com/BenThelen/python-refprop. Thank you Ben for sharing it!

## Matlab Integration
There is a Matlab prototype file available from
[NIST](http://www.boulder.nist.gov/div838/theory/refprop/Frequently_asked_questions.htm#MatLabApplications "NIST homepage"). Unfortunately, you have to change a few things in order to use the library on MacOS and GNU/Linux.

There is a makefile section and a shell script that help you with this. After installing the library as described above, you can run `make matlab` in order to use REFPROP with Matlab. Then run `make matlab-install’ as root user for a system-wide installation. 

The test.m is a simple code you can use to check if the intergration works.

Summary for the impatient:
  * Go to the directory with the downloaded files and open a command prompt.
  * Run `make` and then `sudo make install` to install the shared library.
  * Run `make matlab` to download files and edit them as written in the terminal.
  * Run `sudo make matlab-install` to copy the matlab file to `/opt/refprop`.

### MATLAB 64 bit Integration
This part was contributed partly by [nkampy](https://github.com/nkampy) and [speredenn](https://github.com/speredenn) and is still experimental. Please open new issues if you encounter any problems. Problems are likely to be encountered in setting up matlab with gcc, needed to use the builtin MEX functionality, which is required for the load library command in the thunk.m file. We hope that the user community and [nkampy's](https://github.com/nkampy) comments, left at the mathworks website ([here](http://www.mathworks.com/matlabcentral/answers/125301-maverick-r2014a-loadlibrary-error-loaddefinedlibrary) and [here](http://www.mathworks.com/matlabcentral/answers/124597-how-to-setup-gfortran-on-mac-osx-10-9-and-matlab-r2014a)), will help figuring out a good solution.

## Successful MATLAB 64 bit Integration (update from 20150424)

The important features of my system are:

* MAC OS X 10.8.5
* X-Code 5.1.1 with its command-line developer tool
* gcc: Apple LLVM version 5.1 (clang-503.0.40) (based on LLVM 3.4 svn)
* gfortran : 4.8.1
* MATLAB_R2011a 64 bit installation 

and I followed the installation instructions indicated on : https://github.com/jowr/librefprop.so
Everything worked fine until I called make fortest (in order to test the installation), the logs I got are : 

make fortest
%-----------------------------
`gfortran -O3 -ffast-math -fPIC  -g -o ./src/ex_mix.o -c ./src/ex_mix.for`
`gfortran  -g -o ./bin/ex_mix_for ./src/ex_mix.o -lrefprop -lgfortran`
*Undefined symbols for architecture x86_64:
  "start", referenced from:
     -u command line option
ld: symbol(s) not found for architecture x86_64
collect2: error: ld returned 1 exit status
make: [bin/ex_mix_for] Error 1*
%-----------------------------

To make it work, I had to change it to:

%-----------------------------
`gfortran -O3 -ffast-math -fPIC  -g -o ./src/ex_mix.o -c ./src/ex_mix.for`
`gfortran  -g -o ./bin/ex_mix_for ./src/ex_mix.o -lrefprop -lgfortran -lcrt1.o`
%-----------------------------
**Note: This can be done directly in the Makefile in the line 336.**

and then it compiled properly. I don't know exactly why, I found on the web it has to do with some static libraries which are not linked without this option, but I am not a specialist....
Afterward I could execute  `./bin/ex_mix_for` without any problem.

For the actual Matlab linkage (refpropm.m):
I performed the steps as indicated on: https://github.com/jowr/librefprop.so
1.  The first step of the manual work (as indicated during the make matlab) went w/o problem. 
2.  Then the second step:

 *Open Matlab, and run:*                                                                                                                                                           ` "cd /Users/thierrymeier/Documents/MATLAB/PhD/refProp9.1/librefprop.so-master/matlab;"`
    ` "run('thunk.m');" `
    *proceed by pressing ENTER.*

    *rm matlab/refpropm.m.org matlab/rp_proto64.m.org*

There I had a problem while running `thunk.m`, as I currently have two gcc compiler installed ( Apple LLVM version 5.1 and gcc4.8.1) but none of them correspond exactly to gcc-4.2. Thus I created a symlink:

`sudo ln -s /usr/bin/gcc /usr/bin/gcc4.2`

and it worked (I know its a little bit brute force but as a first shot...).

3. I ran the `sudo make matlab-install` and the matlab function and the libraries were successfully copied in */opt/refprop* as expected. 

4. running `path(path, '/opt/refprop')` permits to have a system wide use of the refprop functions.

5.  As I run `test.m` the refpropm function is called and Matlab crashes.... The log I get:

%-----------------------------
*Process:         MATLAB [2368]
Path:            /Applications/MATLAB_R2011a.app/Contents/MacOS/StartMATLAB
Identifier:      com.mathworks.matlab
Version:         MATLAB Release 2011a (2.1)
Code Type:       X86-64 (Native)
Parent Process:  launchd [127]
User ID:         501*

*Date/Time:       2015-04-22 12:05:40.073 +0200
OS Version:      Mac OS X 10.8.5 (12F2518)
Report Version:  10*

*Interval Since Last Report:          149802 sec
Crashes Since Last Report:           4
Per-App Interval Since Last Report:  14710 sec
Per-App Crashes Since Last Report:   3
Anonymous UUID:                      0FA54308-2534-36FE-FC59-4636403F2392*

Crashed Thread:  3

Exception Type:  EXC_BREAKPOINT (SIGTRAP)
Exception Codes: 0x0000000000000002, 0x0000000000000000

*Dyld Error Message:
  Symbol not found: __gfortran_transfer_character_write
  Referenced from: /opt/refprop/librefprop.dylib
  Expected in: /Applications/MATLAB_R2011a.app/sys/os/maci64/libgfortran.3.dylib*
%-----------------------------

suggests a problem with the dylib library. To overcome it, I read the thread on w3.mathworks.com [(here)](http://www.mathworks.com/matlabcentral/answers/125301-maverick-r2014a-loadlibrary-error-loaddefinedlibrary) where Nathan suggest to rename the DYLIB fil in the MATLAB application directory and create a symlink with the one located in `/usr/local/lib`:
`ln -s /usr/local/lib/libgfortran.3.dylib /Applications/MATLAB_R2011a.app/sys/os/maci64/libgfortran.3.dylib`

and then everything works. I hope this can help some users of Mac OS...
I am not sure whether it is the nicest solution to do it, but who cares?


## No root user access
It is possible to use the shared libraries without root access. However, you need to make sure that the libraries get found and it is recommended to add something like `export LD_LIBRARY_PATH=/home/USERNAME/lib:/home/USERNAME/refprop:$LD_LIBRARY_PATH` to the calls to executables that need REFPROP. The makefile will print more instructions when running `make install` as a non-root user.

## Known Problems
  * Older compilers might not work properly with the OpenMP directives used in the original Fortran code. If you experience any problems related to OpenMP, try removing OpenMP support by setting `USEOPENMP  :=FALSE` in line 51 of the Makefile.

## General Remarks
Please note that you need a working and licensed copy of REFPROP in order to use the software provided here. This is not a replacement for REFPROP. You can purchase REFPROP at http://www.nist.gov/srd/nist23.cfm

If you are interested in fluid property modelling, you might also be interested in [CoolProp](https://github.com/ibell/coolprop), an open-source thermodynamic fluid property package with over 100 compressible and over 50 incompressible fluids.
