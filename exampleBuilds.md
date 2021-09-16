#  Example Builds

## Building More Stuff

Now that `cmake` is bulit, let's build some more stuff

### Add cmake to PATH, override 'ps'
```
export PATH="$HOME/Dev Space/Net/cmake/Inst/bin:$PATH"
cp /usr/bin/false "$HOME/Dev Space/Net/cmake/Inst/bin/ps"
```

### gflags
```
mkdir -p "$HOME/Dev Space/Net/gflags/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/gflags/Src"
git clone https://github.com/gflags/gflags.git
cd "$HOME/Dev Space/Net/gflags/Build"
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/Dev Space/Net/gflags/Src/gflags" -B . --install-prefix "$HOME/Dev Space/Net/gflags/Inst"
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
...
-- Installing: /Users/sand/Dev Space/Net/gflags/Inst/lib/pkgconfig/gflags.pc
CMake Error at cmake_install.cmake:128 (file):
  file cannot create directory: /Users/sand/.cmake/packages/gflags.  Maybe
  need administrative privileges.

make: *** [install] Error 1
```
Tallied sandbox log messages for `gflags`.
```
tr '0123456789' '##########' </tmp/sblog-gflags-01.txt | sed 's/[^ ]*  *[^ ]*  *[^ ]* *[^ ]*  *[^ ]*//'  | grep -v duplicate | sort | uniq -c | sort -rn | grep -e allow -e deny | less
14  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
14  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
 2  Sandbox: clang(#####) deny(#) file-read-metadata /Library/Frameworks
 1  Sandbox: ld(#####) deny(#) file-read-metadata /Library/Frameworks
 1  Sandbox: cmake(#####) deny(#) file-write-create /Users/sand/.cmake/packages/gflags
b
```
We see that the install process is denied creating a `.cmake` directory in the home directory  `/Users/sand/.cmake/packages/gflags`.  This error hasn't (yet) prevented me from building dependent software components, so I'm ignoring it, content in the knowledge that the build isn't writing to unexpected places.

### glog
Note setting `CMAKE_PREFIX_PATH`
```
mkdir -p "$HOME/Dev Space/Net/glog/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/glog/Src"
git clone https://github.com/google/glog.git
cd "$HOME/Dev Space/Net/glog/Build"
export CMAKE_PREFIX_PATH="$HOME/Dev Space/Net/gflags/Inst"
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/Dev Space/Net/glog/Src/glog" -B . --install-prefix "$HOME/Dev Space/Net/glog/Inst"
```
Output from above includes this error message. But it doesn't terminate the command. 
```
...
-- Performing Test COMPILER_HAS_DEPRECATED_ATTR - Success
CMake Warning at CMakeLists.txt:903 (export):
  Cannot create package registry file:
    /Users/sand/.cmake/packages/glog/1875892d32396b70f8f1afed2967474b
  Operation not permitted
-- Configuring done
...
```
Continuing...
```
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
...
-- Installing: /Users/sand/Dev Space/Net/glog/Inst/lib/cmake/glog/glog-targets-noconfig.cmake
```
Tallied log messages
```
68  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
 68  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
 16  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##R#
 11  Sandbox: cmake(#####) deny(#) file-read-metadata /opt
 11  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Frameworks
  9  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin
  6  Sandbox: cmake(#####) deny(#) file-read-metadata /sbin
  6  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr/bin
  5  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr
  5  Sandbox: cmake(#####) deny(#) file-read-data /
  3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/standalone
  3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sendmail
  3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/libexec
  3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##
  3  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/System/Library/CoreServices/SafariSupport.bundle/Contents/MacOS/safaridriver
  3  Sandbox: cmake(#####) deny(#) file-read-data /usr/sbin
  3  Sandbox: cmake(#####) deny(#) file-read-data /sbin
  3  Sandbox: cmake(#####) deny(#) file-read-data /opt
  3  Sandbox: cmake(#####) deny(#) file-read-data /Library/Frameworks
  3  Sandbox: cmake(#####) deny(#) file-read-data /Library/Apple/usr/bin
  2  Sandbox: cmake(#####) deny(#) sysctl-read vm.swapusage
  2  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.family
  2  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.cores_per_package
  2  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.brand_string
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.physicalcpu
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.pagesize
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.memsize
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.logicalcpu
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#icachesize
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#cachesize
  2  Sandbox: cmake(#####) deny(#) sysctl-read hw.cpufrequency
  2  Sandbox: clang(#####) deny(#) file-read-metadata /Library/Frameworks
  1  Sandbox: ld(#####) deny(#) file-read-metadata /Library/Frameworks
  1  Sandbox: cmake(#####) deny(#) file-write-create /Users/sand/.cmake/packages/glog/#######d#####b##f#f#afed#######b.tmp##f##
  1  Sandbox: cmake(#####) deny(#) file-read-metadata /Users/sand/.cmake/packages/glog
```

### eigen
```
mkdir -p "$HOME/Dev Space/Net/eigen/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/eigen/Dist"
curl -OL https://gitlab.com/libeigen/eigen/-/archive/3.3.9/eigen-3.3.9.tar.gz
cd "$HOME/Dev Space/Net/eigen/Src"
;: untar with sandbox protection
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" tar xf "$HOME/Dev Space/Net/eigen/Dist/eigen-3.3.9.tar.gz"
cd "$HOME/Dev Space/Net/eigen/Build"
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/Dev Space/Net/eigen/Src/eigen-3.3.9" -B . --install-prefix "$HOME/Dev Space/Net/eigen/Inst"
```
Output from above includes several error message like this

```
...
-- Could NOT find PkgConfig (missing: PKG_CONFIG_EXECUTABLE) 
CMake Warning (dev) at /Users/sand/Dev Space/Net/cmake/Inst/share/cmake-3.21/Modules/FindPackageHandleStandardArgs.cmake:438 (message):
  The package name passed to `find_package_handle_standard_args` (BLAS) does
  not match the name of the calling package (BLASEXT).  This can lead to
  problems in calling code that expects `find_package` result variables
  (e.g., `_FOUND`) to follow a certain pattern.
Call Stack (most recent call first):
  cmake/FindBLASEXT.cmake:377 (find_package_handle_standard_args)
  cmake/FindPASTIX.cmake:214 (find_package)
  bench/spbench/CMakeLists.txt:41 (find_package)
This warning is for project developers.  Use -Wno-dev to suppress it.
...
-- 
CMake Warning at CMakeLists.txt:578 (export):
  Cannot create package registry file:

    /Users/sand/.cmake/packages/Eigen3/aeefd2fb089633968c17f476bf3a866b

  No such file or directory



-- Configuring done
CMake Warning (dev):
  Policy CMP0042 is not set: MACOSX_RPATH is enabled by default.  Run "cmake
  --help-policy CMP0042" for policy details.  Use the cmake_policy command to
  set the policy and suppress this warning.

  MACOSX_RPATH is not specified for the following targets:

   eigen_blas
   eigen_lapack

This warning is for project developers.  Use -Wno-dev to suppress it.

-- Generating done
-- Build files have been written to: /Users/sand/Dev Space/Net/eigen/Build
```
Continuing...
```
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
...
-- Installing: /Users/sand/Dev Space/Net/eigen/Inst/include/eigen3/unsupported/Eigen/CXX11/src/Tensor/TensorAssign.h
```
Tallied log messages
```
156  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##R#
 92  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin
 69  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.distributed_notifications@Uv#
 69  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.FSEvents
 69  Sandbox: xcodebuild(#####) deny(#) file-read-data /
 60  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
 60  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
 57  Sandbox: cmake(#####) deny(#) file-read-metadata /opt
 57  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Frameworks
 51  Sandbox: cmake(#####) deny(#) file-read-metadata /sbin
 51  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr/bin
 41  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/standalone
 41  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sendmail
 41  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/libexec
 41  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##
 41  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/System/Library/CoreServices/SafariSupport.bundle/Contents/MacOS/safaridriver
 41  Sandbox: cmake(#####) deny(#) file-read-data /usr/sbin
 41  Sandbox: cmake(#####) deny(#) file-read-data /sbin
 41  Sandbox: cmake(#####) deny(#) file-read-data /opt
 41  Sandbox: cmake(#####) deny(#) file-read-data /Library/Apple/usr/bin
 39  Sandbox: cmake(#####) deny(#) file-read-data /Library/Frameworks
 10  Sandbox: cmake(#####) deny(#) sysctl-read vm.swapusage
 10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.family
 10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.cores_per_package
 10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.brand_string
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.physicalcpu
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.pagesize
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.memsize
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.logicalcpu
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#icachesize
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#cachesize
 10  Sandbox: cmake(#####) deny(#) sysctl-read hw.cpufrequency
  6  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr
  6  Sandbox: cmake(#####) deny(#) file-read-data /
  3  Sandbox: clang(#####) deny(#) file-read-metadata /Library/Frameworks
  2  Sandbox: ld(#####) deny(#) file-read-metadata /Library/Frameworks
  1  Sandbox: routined(#####) deny(#) mach-lookup com.apple.Maps.MapsSync.store
  1  Sandbox: cmake(#####) deny(#) file-write-create /Users/sand/.cmake/packages/Eigen#
```
  
### ceres
Note setting `CMAKE_PREFIX_PATH` and `touch`

```
mkdir -p "$HOME/Dev Space/Net/ceres/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/ceres/Src"
git clone https://ceres-solver.googlesource.com/ceres-solver
cd "$HOME/Dev Space/Net/ceres/Build"
export CMAKE_PREFIX_PATH="$HOME/Dev Space/Net/eigen/Inst:$HOME/Dev Space/Net/glog/Inst:$HOME/Dev Space/Net/gflags/Inst"
;: touch - because if the .git/hooks/commit-msg doesn't exist in the source, it will try to download and copy it there, which fails because *sandbox*
touch  "$HOME/Dev Space/Net/ceres/Src/ceres-solver/.git/hooks/commit-msg"
/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/DevSpace/Net/ceres/Src/ceres-solver" -B . --install-prefix "$HOME/Dev Space/Net/ceres/Inst"
...
-- Build files have been written to: /Users/sand/Dev Space/Net/ceres/Build
       16.87 real         6.54 user         5.48 sys

/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
...
-- Installing: /Users/sand/Dev Space/Net/ceres/Inst/lib/libceres.a
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: /Users/sand/Dev Space/Net/ceres/Inst/lib/libceres.a(cxsparse.cc.o) has no symbols
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: /Users/sand/Dev Space/Net/ceres/Inst/lib/libceres.a(float_suitesparse.cc.o) has no symbols
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: /Users/sand/Dev Space/Net/ceres/Inst/lib/libceres.a(float_cxsparse.cc.o) has no symbols
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: /Users/sand/Dev Space/Net/ceres/Inst/lib/libceres.a(suitesparse.cc.o) has no symbols
      506.95 real      1911.76 user        81.74 sys
```
Tallied log messages
```
122  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##R#
101  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin
100  Sandbox: cmake(#####) deny(#) file-read-metadata /opt
100  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Frameworks
 91  Sandbox: cmake(#####) deny(#) file-read-metadata /sbin
 91  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr/bin
 18  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
 18  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/standalone
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sendmail
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/libexec
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/System/Library/CoreServices/SafariSupport.bundle/Contents/MacOS/safaridriver
 10  Sandbox: cmake(#####) deny(#) file-read-data /usr/sbin
 10  Sandbox: cmake(#####) deny(#) file-read-data /sbin
 10  Sandbox: cmake(#####) deny(#) file-read-data /opt
 10  Sandbox: cmake(#####) deny(#) file-read-data /Library/Apple/usr/bin
  9  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr
  9  Sandbox: cmake(#####) deny(#) file-read-data /Library/Frameworks
  9  Sandbox: cmake(#####) deny(#) file-read-data /
  3  Sandbox: clang(#####) deny(#) file-read-metadata /Library/Frameworks
  2  Sandbox: ld(#####) deny(#) file-read-metadata /Library/Frameworks
  1  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.lsd.modifydb
  1  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.coreservices.quarantine-resolver
  1  Sandbox: xcodebuild(#####) deny(#) file-read-metadata /Users/sand/Library/Preferences/com.apple.LaunchServices/com.apple.launchservices.secure.plist
  1  Sandbox: xcodebuild(#####) deny(#) file-read-data /Users/sand/Library/Preferences/com.apple.LaunchServices/com.apple.launchservices.secure.plist
```
Try running the tests
```
/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make test
...
165/165 Test #165: ba_sparseschur_acceleratesparse_user_threads_test .............   Passed    2.47 sec

95% tests passed, 8 tests failed out of 165

Total Test time (real) = 151.28 sec

The following tests FAILED:
     19 - corrector_test (SEGFAULT)
     21 - covariance_test (SEGFAULT)
     39 - graph_test (SEGFAULT)
     53 - local_parameterization_test (SEGFAULT)
     58 - ordered_groups_test (SEGFAULT)
     61 - parameter_block_test (SEGFAULT)
     66 - problem_test (SEGFAULT)
     85 - triplet_sparse_matrix_test (SEGFAULT)
Errors while running CTest
Output from these tests are in: /Users/sand/Dev Space/Net/ceres/Build/Testing/Temporary/LastTest.log
Use "--rerun-failed --output-on-failure" to re-run the failed cases verbosely.
make: *** [test] Error 8
      151.33 real       160.65 user        15.60 sys
```
Most tests pass. Nothing in sandbox log indicating sandbox was cause of failure.

### pbrt-v4
```
mkdir -p "$HOME/Dev Space/Net/pbrt-v4/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/pbrt-v4/Src"
git clone --recursive https://github.com/mmp/pbrt-v4.git
cd "$HOME/Dev Space/Net/pbrt-v4/Build"
unset CMAKE_PREFIX_PATH
/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/Dev Space/Net/pbrt-v4/Src/pbrt-v4" -B . --install-prefix "$HOME/Dev Space/Net/pbrt-v4/Inst"
/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
...
-- Installing: /Users/sand/Dev Space/Net/pbrt-v4/Inst/lib/cmake/double-conversion/double-conversionTargets-release.cmake
      438.93 real      2226.95 user        59.35 sys
```
Tallied log messages
```
30  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##R#
27  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
27  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
17  Sandbox: cmake(#####) deny(#) file-read-metadata /opt
17  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Frameworks
11  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin
10  Sandbox: cmake(#####) deny(#) sysctl-read vm.swapusage
10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.family
10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.cores_per_package
10  Sandbox: cmake(#####) deny(#) sysctl-read machdep.cpu.brand_string
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.physicalcpu
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.pagesize
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.memsize
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.logicalcpu
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#icachesize
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.l#cachesize
10  Sandbox: cmake(#####) deny(#) sysctl-read hw.cpufrequency
 9  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr
 9  Sandbox: cmake(#####) deny(#) file-read-data /
 8  Sandbox: cmake(#####) deny(#) file-read-metadata /sbin
 8  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr/bin
 3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/standalone
 3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sendmail
 3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/libexec
 3  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##
 3  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/System/Library/CoreServices/SafariSupport.bundle/Contents/MacOS/safaridriver
 3  Sandbox: cmake(#####) deny(#) file-read-data /usr/sbin
 3  Sandbox: cmake(#####) deny(#) file-read-data /sbin
 3  Sandbox: cmake(#####) deny(#) file-read-data /opt
 3  Sandbox: cmake(#####) deny(#) file-read-data /Library/Frameworks
 3  Sandbox: cmake(#####) deny(#) file-read-data /Library/Apple/usr/bin
 3  Sandbox: clang(#####) deny(#) file-read-metadata /Library/Frameworks
 2  Sandbox: ld(#####) deny(#) file-read-metadata /Library/Frameworks
 1  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.distributed_notifications@Uv#
 1  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.FSEvents
 1  Sandbox: xcodebuild(#####) deny(#) file-read-data /
```
Test render simple scene with pbrt
```
mkdir -p "$HOME/Dev Space/Net/tests/pbrt"
cd "$HOME/Dev Space/Net/tests/pbrt"
cat >s8v4.pbrt <<'EOFF'
Film "rgb"
    "integer yresolution" [ 300 ]
    "integer xresolution" [ 300 ]
    "string filename" [ "s8.png" ]

Sampler "halton"
    "integer pixelsamples" [ 128 ]
Integrator "bdpt"
LookAt 796800000 0 0
    800000000 0 0
    0 0 1
Camera "perspective"
    "float fov" [ 4 ]
WorldBegin
LightSource "distant"
    "float scale" [1.5]
    "blackbody L" [ 6500 ]
    "point3 to" [ 1 0 0 ]
AttributeBegin
    Material "diffuse"
        "rgb reflectance" [ 0.9 0.9 0.9 ]
    Translate 800000000 0 0
    Shape "sphere"
        "float radius" [ 80000 ]
AttributeEnd
EOFF

/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" "$HOME/Dev Space/Net/pbrt-v4/Inst/bin/pbrt" s8v4.pbrt
pbrt version 4 (built Sep 13 2021 at 20:35:33)
Copyright (c)1998-2021 Matt Pharr, Wenzel Jakob, and Greg Humphreys.
The source code to pbrt (but *not* the book contents) is covered by the Apache 2.0 License.
See the file LICENSE.txt for the conditions of the license.
Rendering: [+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]  (10.6s)       
       10.82 real        78.99 user         0.17 sys
```
### llvm clang
```
mkdir -p "$HOME/Dev Space/Net/llvm/"{Src,Build,Dist,Inst}
cd "$HOME/Dev Space/Net/llvm/Src"
git clone --depth=1 https://github.com/llvm/llvm-project.git

;: patch llvm bug that causes abort durring clang link of build directory path contains space character
cd "$HOME/Dev Space/Net/llvm/Src/llvm-project/clang/tools/driver"
patch <<'EOFF'
--- CMakeLists.txt    2021-09-14 13:10:43.000000000 -0700
+++ CMakeLists-fixed.txt    2021-09-14 13:13:16.000000000 -0700
@@ -82,7 +82,7 @@
   set(TOOL_INFO_PLIST_OUT "${CMAKE_CURRENT_BINARY_DIR}/${TOOL_INFO_PLIST}")
   target_link_libraries(clang
     PRIVATE
-    "-Wl,-sectcreate,__TEXT,__info_plist,${TOOL_INFO_PLIST_OUT}")
+    "-Wl,-sectcreate,__TEXT,__info_plist,\"${TOOL_INFO_PLIST_OUT}\"")
   configure_file("${TOOL_INFO_PLIST}.in" "${TOOL_INFO_PLIST_OUT}" @ONLY)
 
   set(TOOL_INFO_UTI)
EOFF


cd "$HOME/Dev Space/Net/llvm/Build"
unset CMAKE_PREFIX_PATH
/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" cmake -S "$HOME/Dev Space/Net/llvm/Src/llvm-project/llvm" -B . --install-prefix "$HOME/Dev Space/Net/llvm/Inst" -DLLVM_ENABLE_PROJECTS=clang -G "Unix Makefiles"
...
-- Performing Test HAVE_STEADY_CLOCK -- success
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/sand/Dev Space/Net/llvm/Build
       65.92 real        27.86 user        34.20 sys

/usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install ; date
...
-- Installing: /Users/sand/Dev Space/Net/llvm/Inst/lib/cmake/llvm/./TableGen.cmake
     5701.77 real     20778.79 user      1164.39 sys
Tue Sep 14 15:07:26 PDT 2021
```
Test trivial compile with built clang
```
mkdir -p "$HOME/Dev Space/Net/tests"
cd "$HOME/Dev Space/Net/tests"
cat >main.c <<'EOFF'
#include <stdio.h>
int main(){
printf("hello you\n");
return 0;
}
EOFF

;: Note: needed -isysroot to find stdio.h
;: Maybe there is better way to build clang to specify default system search
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" "$HOME/Dev Space/Net/llvm/Inst"/bin/clang  -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk main.c
sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" "`pwd`/a.out"
```
