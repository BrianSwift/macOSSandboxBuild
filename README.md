# MacOS Sandbox Build 

## What is this?

MacOS sandbox profiles and instructions for command line (Terminal) building of software in a sandboxed environment.

## Audience

People who build software from the command line on macOS and have security related concerns about what could be occurring in a complex build process.

## How do I use it?

After some setup, software build commands are executed in a restricted environment defined by the `confined.sb` profile using macOS command `sandbox-exec`. Three parameters passed to `confined.sb` specify directories accessible to the build command that are additions to a limited set of standard system directories.
* `_RX1` : contents are readable and executable, and metadata can be read from its path-ancestors
* `_RW1`: contents are readable and writable
* `_TMPDIR`: contents are readable and writable

The following example steps through building `cmake`.

### Download Profiles

```
mkdir -p "$HOME/Development/github"
cd "$HOME/Development/github"
git clone https://github.com/BrianSwift/macOSSandboxBuild.git
```

### Setup `TMPDIR`

```
export TMPDIR="$HOME/Dev Space/sandtmp"
mkdir -p "$TMPDIR"
```

### Load `xcrun_db` cache
Identify tool names in `/usr/bin` that might bounce to an XCode tool (this is intersection of tools in Xcode.app and tools in `/usr/bin`.)
This annoying setup process only needs to be done once.
```
(ls /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin ; ls /Applications/Xcode.app/Contents/Developer/usr/bin ) | sort -u >/tmp/tool_list_xcode.txt
ls /usr/bin | sort >/tmp/tool_list_usr_bin.txt
comm -12 /tmp/tool_list_usr_bin.txt /tmp/tool_list_xcode.txt >/tmp/tool_list_to_cache.txt
```
Execute `xcrun -find` for each tool name to load `xcrun_db`.  Execution in sandbox using  `nomach.sb`  profile causes `xcrun` to fallback to using  `$TMPDIR` for location of  `xcrun_db`, rather than `_CS_DARWIN_USER_TEMP_DIR` returned by `confstr(3)`. `nomach.sb` only denies `mach-lookup` operations. It is not more restrictive because only Apple provided code is executed.

First pass takes a few seconds per tool, and may get error messages when looking up `DeRez` and `swift`. Second command produces lots of errors, but that is expected because some apps don't recognize `--version`. Final command should only take a fraction of second per tool, and produce no errors.
```
</tmp/tool_list_to_cache.txt xargs -n 1 -I % /usr/bin/time sandbox-exec -f $HOME/Development/github/macOSSandboxBuild/nomach.sb xcrun -find %
: Some tools like "make" are not cached correctly with "xcrun -find" so also make a pass attempting to run each with "--version". For some tools this is produces an error, but that is OK since we are only trying to get the cache initialized. "true" at end keeps xargs from terminating early when tools exit with error.
</tmp/tool_list_to_cache.txt xargs -n 1 -I % /usr/bin/time sandbox-exec -f $HOME/Development/github/macOSSandboxBuild/nomach.sb sh -c "% --version ; true"
: This should not produce errors and each command should only take fraction of a second
</tmp/tool_list_to_cache.txt xargs -n 1 -I % /usr/bin/time sandbox-exec -f $HOME/Development/github/macOSSandboxBuild/nomach.sb xcrun -find %

```
Verify `xcrun_db` has been created in `$TMPDIR`.
```
ls -l "$TMPDIR"
total 64
-rw-------  1 sand  notstaff  31950 Sep  9 21:46 xcrun_db
```
`xcrun_db` is pre-loaded because `xcodebuild` (which is invoked by shim apps in `/usr/bin` does not like trying to determine the developer tools configuration while running in a confined sandbox. After producing `xcrun_db`, `xcodebuild` will be able to lookup paths to real tools whenever a shim is invoked rather than performing a search.

This setup probably needs to be repeated anytime the default developer tools are changed with `xcode-select`, but that hasn't been tested.

This may be avoidable if environment variables are set  to specify the SDK and path to XCode tools, but this has not been tested.

### Build Something (`cmake`) in Confined Sandbox (with Activity Logging)

Setup Build and Install tree
```
mkdir -p "$HOME/Dev Space/Net/cmake/"{Src,Build,Dist,Inst}
```
Download `cmake` source distribution (using "download.sb" sandbox)
```
cd "$HOME/Dev Space/Net/cmake/Dist"
sandbox-exec -D_RW1="$PWD" -f /Users/bswift/Development/github/macOSSandboxBuild/download.sb curl -OL https://github.com/Kitware/CMake/releases/download/v3.21.2/cmake-3.21.2.tar.gz
```
Unpack source within confined sandbox
```
cd "$HOME/Dev Space/Net/cmake/Src"
sandbox-exec -D_RX1="$HOME/Dev Space/Net/cmake/Dist" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" tar xf "$HOME/Dev Space/Net/cmake/Dist/cmake-3.21.2.tar.gz"
```
Prepare build directory by copying `/usr/bin/false` to `ps` to keep `cmake` build from spewing `"Failure calling sysctl: Operation not permitted"` messages. `ps` is a suid program, and Apple's sandbox environment (understandably) denies execution of suid programs.
```
cd "$HOME/Dev Space/Net/cmake/Build"
cp /usr/bin/false ps
```
Start logging (optional)

As a user with administrator privileges, in a separate window start recording sandbox log messages for review after build ends.
```
log stream --style compact --info --debug  --predicate '(((processID == 0) AND (senderImagePath CONTAINS "/Sandbox")) OR (subsystem == "com.apple.sandbox.reporting"))' >/tmp/sblog-cmake-01.txt
```
bootstrap `cmake` within confined sandbox
```
cd "$HOME/Dev Space/Net/cmake/Build"
PATH="`pwd`:$PATH" /usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="$PWD" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" "$HOME/Dev Space/Net/cmake/Src/cmake-3.21.2/bootstrap"  --prefix="$HOME/Dev Space/Net/cmake/Inst"
: Expected final output
: CMake has bootstrapped.  Now run make.
:      463.29 real       376.10 user        79.49 sys
```
build and install `cmake` within confined sandbox
```
PATH="`pwd`:$PATH" /usr/bin/time sandbox-exec -D_RX1="$HOME/Dev Space/Net" -D_RW1="`dirname $PWD`" -D_TMPDIR="$TMPDIR" -f "$HOME/Development/github/macOSSandboxBuild/confined.sb" make -j 4 install
: Expected final output
: -- Installing: /Users/sand/Dev Space/Net/cmake/Inst/share/bash-completion/completions/ctest
:      501.18 real      1820.55 user       124.39 sys
```
Stop logging

Stop (^C) the `log stream...` running in another window.

A tally of sandbox allow/deny log messages can be produced with this command. 
```
tr '0123456789' '##########' </tmp/sblog-cmake-01.txt | sed 's/[^ ]*  *[^ ]*  *[^ ]* *[^ ]*  *[^ ]*//'  | grep -v duplicate | sort | uniq -c | sort -rn | grep -e allow -e deny | less 
443  Sandbox: cmake(#####) deny(#) process-exec* /usr/sbin/sysctl
443  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sysctl
 43  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##R#
 25  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.distributed_notifications@Uv#
 25  Sandbox: xcodebuild(#####) deny(#) mach-lookup com.apple.FSEvents
 25  Sandbox: xcodebuild(#####) deny(#) file-read-data /
 20  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin
 19  Sandbox: cmake(#####) deny(#) file-read-metadata /opt
 19  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Frameworks
 13  Sandbox: routined(###) deny(#) mach-lookup com.apple.Maps.MapsSync.store
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/standalone
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/sbin/sendmail
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/libexec
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /usr/X##
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /sbin
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/usr/bin
 10  Sandbox: cmake(#####) deny(#) file-read-metadata /Library/Apple/System/Library/CoreServices/SafariSupport.bundle/Contents/MacOS/safaridriver
 10  Sandbox: cmake(#####) deny(#) file-read-data /usr/sbin
 10  Sandbox: cmake(#####) deny(#) file-read-data /sbin
 10  Sandbox: cmake(#####) deny(#) file-read-data /opt
...
```
Note: the log messages come from all sandbox activity on the system, not just the build process, so you may see messages unrelated to the build.

Note: not all denied operations are being logged. Some denied operations that occur frequently (with every process launch) are configured to not log messages to keep down pollution/noise.

Note: Raw (un-tallied) log also contains many messages related to `Symbolicator for cmake[90573] is NULL` , and stack traces. I haven't figured out how to control these yet.

### Build More Stuff
See `exampleBuilds.md` for walkthroughs of building `gflags`, `glog`,`eigen`, `ceres`, `llvm clang`

## Tell me more

### What is the macOS Sandbox?
"*App Sandbox is an access control technology provided in macOS, enforced at the kernel level.*" (From https://developer.apple.com/library/archive/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html)

The macOS sandbox enables creation of an environment that restricts access to a configurable subset of system resources and capabilities. These include what files can be read/write/stat/exec, network access, mach messaging and other interprocess communication, and system information via sysctl.

The specification of what is and is not accessible from the sandboxed environment is provide by a sandbox profile.

For our purposes, a sandboxed environment is created by running `sandbox-exec` with the `-f` parameters supplying the path to the profile configuring the restricted environment and also the command to be executed within the restricted environment.

Sandbox (also referred to as Seatbelt) was introduced in 2007 in OS X 10.5 Leopard. However, the profile definition language remains undocumented by Apple and is considered an "Apple System Private Interface". Also, while the  `sandbox-exec` man page has indicated it is DEPRECATED since at least 2017, it remains on the system. Also Apple continues to use the technology given that a profile was created for the Apple's relatively new *blastdoor* feature.

Even though Apple hasn't publicly documented the profile definition language, numerous examples can be found in `/System/Library/Sandbox/Profiles` and `/usr/share/sandbox`.

Additional information can be found in the References.

### Describe `confined.sb` profile
The primary goals of  `confined.sb` are to be *understandable* and thus *trustworthy*. The profile uses fairly simple `allow` and `deny` operations that are intended to be understandable on inspection. The idea is that if someone can understand the simple rules, they can have some amount of trust and confidence in the profile's effectiveness.

The profile starts with `(deny default)` which configures the environment to deny all operations with all denied operations being logged.

`(allow process-fork)` enables execution of multiple commands as part of build process.

Six different `(allow file...)` operations enable various types of file access to different parts of the system. Of note, `file-write*` is only enabled for directory hierarchies passed in as parameters `_RW1` and `_TMPDIR`, and for `/dev/null` and `/dev/zero`. `file-read*` and ` process-exec` capabilities are given to the directory hierarchy passed in as parameter `_RX1`,  `/bin`, `/usr/bin`, `/Applications/Xcode.app`.

`file-read*` and `file-read-metadata` (stat) are granted to various system files/directories. Also, `file-read-metadata` is given to `(path-ancestors (param "_RX1"))` which is needed for `cd`-ing.

Subsequent, `(deny (with no-report) ...)` operations suppress logging of operations that are frequently denied to reduce log pollution.
`(allow sysctl-read ...)` grants access to a few system information values.

Finally, `(deny syscall-unix (with no-report) (syscall-number SYS_getfsstat64)` prevents `xcodebuild` from enumerating mounts on every invocation.

Throughout `confined.sb` comments with tool names indicated the named tool was observed using or needing the listed feature.

The singularly focused simplicity of  `confined.sb`  stands in contrast to the complexity of Apple profile `/System/Library/Sandbox/Profiles/application.sb` which must support a wide range of use cases.

### Why aren't those build commands simplified in a script?
Providing a script with sufficient flexibility to accommodate various build styles/layouts would add a layer of complexity, requiring more analysis by the user to develop understanding and trust.

### Why `Dev Space`?
I wanted to verify quoting is sufficient to work with pathnames containing spaces.

### What's this ```-D_RW1="`dirname $PWD``` business
Read/Write access is granted to parent of build directory to permit writes to both the build directory and the install directory. Perhaps there should be a `_RW2` parameter.

### What is include in `macOSSandboxBuild`
* `confined.sb` : Profile used for building software (described above). Network access is denied.
* `download.sb` : Allows network access, and writing to directory specified by `_RW1`, but has little additional access to system
* `nomach.sb` : Just does `(deny mach-lookup)`. Only used durring xcrun_db setup
* `ping.sb` : minimal profile allowing execution of `ping`/`ping6`
* `exampleBuilds.md` : Examples of building additional software within sandbox

### Have you investigated the origins of all the denied operations in builds
Nope.

However, `cmake` frequently accessing `/private` and `/private/tmp` is possibly due to  `kwsys/SystemTools.cxx:  SystemTools::AddKeepPath("/tmp/");)`
Also, frequent exec `/usr/sbin/sysctl` possibly from `Modules/ProcessorCount.cmake:    find_program(ProcessorCount_cmd_sysctl sysctl )`
Don't know where attempts to refernece `/usr/sbin/sendmail` come from. I didn't find a reference to it `cmake` source.

## References

* A long evening with iOS and macOS Sandbox - https://geosn0w.github.io/A-Long-Evening-With-macOS's-Sandbox/
* About App Sandbox - https://developer.apple.com/library/archive/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html#//apple_ref/doc/uid/TP40011183-CH1-SW1
* App Sandbox in Depth - https://developer.apple.com/library/archive/documentation/Security/Conceptual/AppSandboxDesignGuide/AppSandboxInDepth/AppSandboxInDepth.html
* Run IBTool from within Sandboxed App - https://developer.apple.com/forums/thread/26389?answerId=89866022#89866022
* How to build a replacement for sandbox-exec? - https://developer.apple.com/forums/thread/661939
* Why Xcode tools are slow after reboot - https://lapcatsoftware.com/articles/xcrun.html
* mktemp on macOS not honouring $TMPDIR - https://unix.stackexchange.com/questions/555058/mktemp-on-macos-not-honouring-tmpdir
* confstr source - https://opensource.apple.com/source/Libc/Libc-1439.40.11/gen/confstr.c.auto.html


## Future Work
~~Profile with sufficient networking support to allow downloading source via git and curl~~

???
