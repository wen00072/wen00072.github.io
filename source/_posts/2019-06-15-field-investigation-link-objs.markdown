---
layout: post
title: "田野調查：Ubuntu 18.04.2 產生執行檔細節"
date: 2019-06-15 12:09:34 +0800
comments: true
categories: [C, Linux, linker, glibc]
---

本篇旨在從command line角度拆解 `gcc hello.c -o hello` 中間`gcc` 幫你執行了哪些指令，以及這些執行指令大概在做什麼。每個指令光要分析都要花不少時間，當作以後的作業吧。

## 目錄

* [測試環境](#fl0707-env)
* [（不）完整的gcc 編譯步驟](#fl0707-step)
    * [除了hello.o以外偷塞的object files以及liked libraries](#fl0707-step-list)
* [附錄](#fl0707-app)
    * [`cc` 指令拆解](#fl0707-app-cc)
    * [`as` 指令拆解](#fl0707-app-as)
    * [`collect2` 指令拆解](#fl0707-app-cc2)
    * [用 `strace` 觀察 `gcc hello.c -o hello` 會呼叫哪些執行檔](#fl0707-app-strace)
    * [完整的 `gcc -v -o hello hello.c` 輸出](#fl0707-app-gcc)
    * [collect2 說明](#fl0707-app-cc2-help)


<a name="fl0707-env"></a>
## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.2 LTS
Release:	18.04
Codename:	bionic
```

```
$ gcc --version
gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

<a name="fl0707-step"></a>
## （不）完整的gcc 編譯步驟

* [將hello.c 轉譯成組合語言](#fl0707-app-cc)
* [將產生的組合語言檔案編譯成ELF object file](#fl0707-app-as)
* [連結object 檔案和系統runtime object files以及函式庫連結後產生執行檔](#fl0707-app-cc2)

<a name="fl0707-step-list"></a>
### 除了hello.o以外偷塞的object files以及liked libraries
這部份是在[連結](#fl0707-app-cc2)的時候 `gcc` 做的事。記得 `gcc` 全名是 `GNU Compiler Collection`，也就是說他不是編譯器，是一個通包的程式。直接列出多連結的檔案，要注意的是在連結時似乎有個順序，所以會多次出現`--push-state` -> `--as-needed` -> 指定連結函式庫 -> `--pop-state` 的參數，以後有空再來看這部份。


* /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o
* /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o
* /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o
* /lib64/ld-linux-x86-64.so.2
* /usr/lib/gcc/x86_64-linux-gnu/7/32/libgcc.a  
* /lib/x86_64-linux-gnu/libc.so.6
* /usr/lib/x86_64-linux-gnu/libc.a
* /usr/lib/gcc/x86_64-linux-gnu/7/libgcc_s.so.1
* /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o
* /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o
* /lib/x86_64-linux-gnu/libc.so.6

<a name="fl0707-app"></a>
## 附錄

<a name="fl0707-app-cc"></a>
### `cc` 指令拆解

```
/usr/lib/gcc/x86_64-linux-gnu/7/cc1
-quiet
-v
-imultiarch
x86_64-linux-gnu
hello.c
-quiet
-dumpbase
hello.c
-mtune=generic
-march=x86-64
-auxbase
hello
-version
-fstack-protector-strong
-Wformat
-Wformat-security
-o
/tmp/ccUCGQCr.s
```

<a name="fl0707-app-as"></a>
### `as` 指令拆解

```
as -v --64 -o /tmp/cct36Qgi.o /tmp/ccUCGQCr.s
```


<a name="fl0707-app-cc2"></a>
### `collect2` 指令拆解

```
/usr/lib/gcc/x86_64-linux-gnu/7/collect2
-plugin
/usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so
-plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper
-plugin-opt=-fresolution=/tmp/ccizEA38.res
-plugin-opt=-pass-through=-lgcc
-plugin-opt=-pass-through=-lgcc_s
-plugin-opt=-pass-through=-lc
-plugin-opt=-pass-through=-lgcc
-plugin-opt=-pass-through=-lgcc_s
--sysroot=/
--build-id
--eh-frame-hdr
-m
elf_x86_64
--hash-style=gnu
--as-needed
-dynamic-linker
/lib64/ld-linux-x86-64.so.2
-pie
-z
now
-z
relro
-o
hello
/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o
/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o
/usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o
-L/usr/lib/gcc/x86_64-linux-gnu/7
-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu
-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib
-L/lib/x86_64-linux-gnu
-L/lib/../lib
-L/usr/lib/x86_64-linux-gnu
-L/usr/lib/../lib
-L/usr/lib/gcc/x86_64-linux-gnu/7/../../..
/tmp/cct36Qgi.o
-lgcc
--push-state
--as-needed
-lgcc_s
--pop-state
-lc
-lgcc
--push-state
--as-needed
-lgcc_s
--pop-state
/usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o
/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o
```


<a name="fl0707-app-strace"></a>
### 用 `strace` 觀察 `gcc hello.c -o hello` 會呼叫哪些執行檔

```
$ strace -s 8192 -f gcc  -o hello hello.c 2>&1 | grep execve
execve("/usr/bin/gcc", ["gcc", "-o", "hello", "hello.c"], 0x7fff89e3d720 /* 71 vars */) = 0
[pid 23886] execve("/usr/lib/gcc/x86_64-linux-gnu/7/cc1", ["/usr/lib/gcc/x86_64-linux-gnu/7/cc1", "-quiet", "-imultiarch", "x86_64-linux-gnu", "hello.c", "-quiet", "-dumpbase", "hello.c", "-mtune=generic", "-march=x86-64", "-auxbase", "hello", "-fstack-protector-strong", "-Wformat", "-Wformat-security", "-o", "/tmp/ccLpZPzY.s"], 0x211e8c0 /* 76 vars */ <unfinished ...>
[pid 23886] <... execve resumed> )      = 0
[pid 23887] execve("/usr/bin/as", ["as", "--64", "-o", "/tmp/cceDpnfo.o", "/tmp/ccLpZPzY.s"], 0x211e8c0 /* 76 vars */ <unfinished ...>
[pid 23887] <... execve resumed> )      = 0
[pid 23888] execve("/usr/lib/gcc/x86_64-linux-gnu/7/collect2", ["/usr/lib/gcc/x86_64-linux-gnu/7/collect2", "-plugin", "/usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so", "-plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper", "-plugin-opt=-fresolution=/tmp/ccN3BxWN.res", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "-plugin-opt=-pass-through=-lc", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "--sysroot=/", "--build-id", "--eh-frame-hdr", "-m", "elf_x86_64", "--hash-style=gnu", "--as-needed", "-dynamic-linker", "/lib64/ld-linux-x86-64.so.2", "-pie", "-z", "now", "-z", "relro", "-o", "hello", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o", "/usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o", "-L/usr/lib/gcc/x86_64-linux-gnu/7", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib", "-L/lib/x86_64-linux-gnu", "-L/lib/../lib", "-L/usr/lib/x86_64-linux-gnu", "-L/usr/lib/../lib", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../..", "/tmp/cceDpnfo.o", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "-lc", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "/usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o"], 0x211f750 /* 78 vars */ <unfinished ...>
[pid 23888] <... execve resumed> )      = 0
[pid 23889] execve("/usr/bin/ld", ["/usr/bin/ld", "-plugin", "/usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so", "-plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper", "-plugin-opt=-fresolution=/tmp/ccN3BxWN.res", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "-plugin-opt=-pass-through=-lc", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "--sysroot=/", "--build-id", "--eh-frame-hdr", "-m", "elf_x86_64", "--hash-style=gnu", "--as-needed", "-dynamic-linker", "/lib64/ld-linux-x86-64.so.2", "-pie", "-z", "now", "-z", "relro", "-o", "hello", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o", "/usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o", "-L/usr/lib/gcc/x86_64-linux-gnu/7", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib", "-L/lib/x86_64-linux-gnu", "-L/lib/../lib", "-L/usr/lib/x86_64-linux-gnu", "-L/usr/lib/../lib", "-L/usr/lib/gcc/x86_64-linux-gnu/7/../../..", "/tmp/cceDpnfo.o", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "-lc", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "/usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o", "/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o"], 0x7ffc38ceec88 /* 78 vars */ <unfinished ...>
[pid 23889] <... execve resumed> )      = 0
```

<a name="fl0707-app-gcc"></a>
### 完整的 `gcc -v -o hello hello.c` 輸出

```
$ gcc -v -o hello hello.c
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 7.4.0-1ubuntu1~18.04.1' --with-bugurl=file:///usr/share/doc/gcc-7/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-7 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-libmpx --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1) 
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/7/cc1 -quiet -v -imultiarch x86_64-linux-gnu hello.c -quiet -dumpbase hello.c -mtune=generic -march=x86-64 -auxbase hello -version -fstack-protector-strong -Wformat -Wformat-security -o /tmp/ccUCGQCr.s
GNU C11 (Ubuntu 7.4.0-1ubuntu1~18.04.1) version 7.4.0 (x86_64-linux-gnu)
	compiled by GNU C version 7.4.0, GMP version 6.1.2, MPFR version 4.0.1, MPC version 1.1.0, isl version isl-0.19-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/7/../../../../x86_64-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-linux-gnu/7/include
 /usr/local/include
 /usr/lib/gcc/x86_64-linux-gnu/7/include-fixed
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
GNU C11 (Ubuntu 7.4.0-1ubuntu1~18.04.1) version 7.4.0 (x86_64-linux-gnu)
	compiled by GNU C version 7.4.0, GMP version 6.1.2, MPFR version 4.0.1, MPC version 1.1.0, isl version isl-0.19-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
Compiler executable checksum: fa57db1fe2d756b22d454aa8428fd3bd
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-mtune=generic' '-march=x86-64'
 as -v --64 -o /tmp/cct36Qgi.o /tmp/ccUCGQCr.s
GNU assembler version 2.30 (x86_64-linux-gnu) using BFD version (GNU Binutils for Ubuntu) 2.30
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/7/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper -plugin-opt=-fresolution=/tmp/ccizEA38.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie -z now -z relro -o hello /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o -L/usr/lib/gcc/x86_64-linux-gnu/7 -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/7/../../.. /tmp/cct36Qgi.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-mtune=generic' '-march=x86-64'
```


<a name="fl0707-app-cc2-help"></a>
### collect2 說明

```
$ /usr/lib/gcc/x86_64-linux-gnu/7/collect2 --help
Usage: collect2 [options]
 Wrap linker and generate constructor code if needed.
 Options:
  -debug          Enable debug output
  --help          Display this information
  -v, --version   Display this program's version number

Overview: http://gcc.gnu.org/onlinedocs/gccint/Collect2.html
Report bugs: <file:///usr/share/doc/gcc-7/README.Bugs>

Usage: /usr/bin/ld [options] file...
Options:
  -a KEYWORD                  Shared library control for HP/UX compatibility
  -A ARCH, --architecture ARCH
                              Set architecture
  -b TARGET, --format TARGET  Specify target for following input files
  -c FILE, --mri-script FILE  Read MRI format linker script
  -d, -dc, -dp                Force common symbols to be defined
  --force-group-allocation    Force group members out of groups
  -e ADDRESS, --entry ADDRESS Set start address
  -E, --export-dynamic        Export all dynamic symbols
  --no-export-dynamic         Undo the effect of --export-dynamic
  -EB                         Link big-endian objects
  -EL                         Link little-endian objects
  -f SHLIB, --auxiliary SHLIB Auxiliary filter for shared object symbol table
  -F SHLIB, --filter SHLIB    Filter for shared object symbol table
  -g                          Ignored
  -G SIZE, --gpsize SIZE      Small data size (if no size, same as --shared)
  -h FILENAME, -soname FILENAME
                              Set internal name of shared library
  -I PROGRAM, --dynamic-linker PROGRAM
                              Set PROGRAM as the dynamic linker to use
  --no-dynamic-linker         Produce an executable with no program interpreter header
  -l LIBNAME, --library LIBNAME
                              Search for library LIBNAME
  -L DIRECTORY, --library-path DIRECTORY
                              Add DIRECTORY to library search path
  --sysroot=<DIRECTORY>       Override the default sysroot location
  -m EMULATION                Set emulation
  -M, --print-map             Print map file on standard output
  -n, --nmagic                Do not page align data
  -N, --omagic                Do not page align data, do not make text readonly
  --no-omagic                 Page align data, make text readonly
  -o FILE, --output FILE      Set output file name
  -O                          Optimize output file
  --out-implib FILE           Generate import library
  -plugin PLUGIN              Load named plugin
  -plugin-opt ARG             Send arg to last-loaded plugin
  -flto                       Ignored for GCC LTO option compatibility
  -flto-partition=            Ignored for GCC LTO option compatibility
  -fuse-ld=                   Ignored for GCC linker option compatibility
  --map-whole-files           Ignored for gold option compatibility
  --no-map-whole-files        Ignored for gold option compatibility
  -Qy                         Ignored for SVR4 compatibility
  -q, --emit-relocs           Generate relocations in final output
  -r, -i, --relocatable       Generate relocatable output
  -R FILE, --just-symbols FILE
                              Just link symbols (if directory, same as --rpath)
  -s, --strip-all             Strip all symbols
  -S, --strip-debug           Strip debugging symbols
  --strip-discarded           Strip symbols in discarded sections
  --no-strip-discarded        Do not strip symbols in discarded sections
  -t, --trace                 Trace file opens
  -T FILE, --script FILE      Read linker script
  --default-script FILE, -dT  Read default linker script
  -u SYMBOL, --undefined SYMBOL
                              Start with undefined reference to SYMBOL
  --require-defined SYMBOL    Require SYMBOL be defined in the final output
  --unique [=SECTION]         Don't merge input [SECTION | orphan] sections
  -Ur                         Build global constructor/destructor tables
  -v, --version               Print version information
  -V                          Print version and emulation information
  -x, --discard-all           Discard all local symbols
  -X, --discard-locals        Discard temporary local symbols (default)
  --discard-none              Don't discard any local symbols
  -y SYMBOL, --trace-symbol SYMBOL
                              Trace mentions of SYMBOL
  -Y PATH                     Default search path for Solaris compatibility
  -(, --start-group           Start a group
  -), --end-group             End a group
  --accept-unknown-input-arch Accept input files whose architecture cannot be determined
  --no-accept-unknown-input-arch
                              Reject input files whose architecture is unknown
  --as-needed                 Only set DT_NEEDED for following dynamic libs if used
  --no-as-needed              Always set DT_NEEDED for dynamic libraries mentioned on
                                the command line
  -assert KEYWORD             Ignored for SunOS compatibility
  -Bdynamic, -dy, -call_shared
                              Link against shared libraries
  -Bstatic, -dn, -non_shared, -static
                              Do not link against shared libraries
  -Bsymbolic                  Bind global references locally
  -Bsymbolic-functions        Bind global function references locally
  --check-sections            Check section addresses for overlaps (default)
  --no-check-sections         Do not check section addresses for overlaps
  --copy-dt-needed-entries    Copy DT_NEEDED links mentioned inside DSOs that follow
  --no-copy-dt-needed-entries Do not copy DT_NEEDED links mentioned inside DSOs that follow
  --cref                      Output cross reference table
  --defsym SYMBOL=EXPRESSION  Define a symbol
  --demangle [=STYLE]         Demangle symbol names [using STYLE]
  --embedded-relocs           Generate embedded relocs
  --fatal-warnings            Treat warnings as errors
  --no-fatal-warnings         Do not treat warnings as errors (default)
  -fini SYMBOL                Call SYMBOL at unload-time
  --force-exe-suffix          Force generation of file with .exe suffix
  --gc-sections               Remove unused sections (on some targets)
  --no-gc-sections            Don't remove unused sections (default)
  --print-gc-sections         List removed unused sections on stderr
  --no-print-gc-sections      Do not list removed unused sections
  --gc-keep-exported          Keep exported symbols when removing unused sections
  --hash-size=<NUMBER>        Set default hash table size close to <NUMBER>
  --help                      Print option help
  -init SYMBOL                Call SYMBOL at load-time
  -Map FILE                   Write a map file
  --no-define-common          Do not define Common storage
  --no-demangle               Do not demangle symbol names
  --no-keep-memory            Use less memory and more disk I/O
  --no-undefined              Do not allow unresolved references in object files
  --allow-shlib-undefined     Allow unresolved references in shared libraries
  --no-allow-shlib-undefined  Do not allow unresolved references in shared libs
  --allow-multiple-definition Allow multiple definitions
  --no-undefined-version      Disallow undefined version
  --default-symver            Create default symbol version
  --default-imported-symver   Create default symbol version for imported symbols
  --no-warn-mismatch          Don't warn about mismatched input files
  --no-warn-search-mismatch   Don't warn on finding an incompatible library
  --no-whole-archive          Turn off --whole-archive
  --noinhibit-exec            Create an output file even if errors occur
  -nostdlib                   Only use library directories specified on
                                the command line
  --oformat TARGET            Specify target of output file
  --print-output-format       Print default output format
  --print-sysroot             Print current sysroot
  -qmagic                     Ignored for Linux compatibility
  --reduce-memory-overheads   Reduce memory overheads, possibly taking much longer
  --relax                     Reduce code size by using target specific optimizations
  --no-relax                  Do not use relaxation techniques to reduce code size
  --retain-symbols-file FILE  Keep only symbols listed in FILE
  -rpath PATH                 Set runtime shared library search path
  -rpath-link PATH            Set link time shared library search path
  -shared, -Bshareable        Create a shared library
  -pie, --pic-executable      Create a position independent executable
  --sort-common [=ascending|descending]
                              Sort common symbols by alignment [in specified order]
  --sort-section name|alignment
                              Sort sections by name or maximum alignment
  --spare-dynamic-tags COUNT  How many tags to reserve in .dynamic section
  --split-by-file [=SIZE]     Split output sections every SIZE octets
  --split-by-reloc [=COUNT]   Split output sections every COUNT relocs
  --stats                     Print memory usage statistics
  --target-help               Display target specific options
  --task-link SYMBOL          Do task level linking
  --traditional-format        Use same format as native linker
  --section-start SECTION=ADDRESS
                              Set address of named section
  -Tbss ADDRESS               Set address of .bss section
  -Tdata ADDRESS              Set address of .data section
  -Ttext ADDRESS              Set address of .text section
  -Ttext-segment ADDRESS      Set address of text segment
  -Trodata-segment ADDRESS    Set address of rodata segment
  -Tldata-segment ADDRESS     Set address of ldata segment
  --unresolved-symbols=<method>
                              How to handle unresolved symbols.  <method> is:
                                ignore-all, report-all, ignore-in-object-files,
                                ignore-in-shared-libs
  --verbose [=NUMBER]         Output lots of information during link
  --version-script FILE       Read version information script
  --version-exports-section SYMBOL
                              Take export symbols list from .exports, using
                                SYMBOL as the version.
  --dynamic-list-data         Add data symbols to dynamic list
  --dynamic-list-cpp-new      Use C++ operator new/delete dynamic list
  --dynamic-list-cpp-typeinfo Use C++ typeinfo dynamic list
  --dynamic-list FILE         Read dynamic list
  --warn-common               Warn about duplicate common symbols
  --warn-constructors         Warn if global constructors/destructors are seen
  --warn-multiple-gp          Warn if the multiple GP values are used
  --warn-once                 Warn only once per undefined symbol
  --warn-section-align        Warn if start of section changes due to alignment
  --warn-shared-textrel       Warn if shared object has DT_TEXTREL
  --warn-alternate-em         Warn if an object has alternate ELF machine code
  --warn-unresolved-symbols   Report unresolved symbols as warnings
  --error-unresolved-symbols  Report unresolved symbols as errors
  --whole-archive             Include all objects from following archives
  --wrap SYMBOL               Use wrapper functions for SYMBOL
  --ignore-unresolved-symbol SYMBOL
                              Unresolved SYMBOL will not cause an error or warning
  --push-state                Push state of flags governing input file handling
  --pop-state                 Pop state of flags governing input file handling
  --print-memory-usage        Report target memory usage
  --orphan-handling =MODE     Control how orphan sections are handled.
  @FILE                       Read options from FILE
/usr/bin/ld: supported targets: elf64-x86-64 elf32-i386 elf32-iamcu elf32-x86-64 a.out-i386-linux pei-i386 pei-x86-64 elf64-l1om elf64-k1om elf64-little elf64-big elf32-little elf32-big pe-x86-64 pe-bigobj-x86-64 pe-i386 plugin srec symbolsrec verilog tekhex binary ihex
/usr/bin/ld: supported emulations: elf_x86_64 elf32_x86_64 elf_i386 elf_iamcu i386linux elf_l1om elf_k1om i386pep i386pe
/usr/bin/ld: emulation specific options:
ELF emulations:
  --ld-generated-unwind-info  Generate exception handling info for PLT
  --no-ld-generated-unwind-info
                              Don't generate exception handling info for PLT
  --build-id[=STYLE]          Generate build ID note
  --compress-debug-sections=[none|zlib|zlib-gnu|zlib-gabi]
                              Compress DWARF debug sections using zlib
                               Default: none
  -z common-page-size=SIZE    Set common page size to SIZE
  -z max-page-size=SIZE       Set maximum page size to SIZE
  -z defs                     Report unresolved symbols in object files.
  -z muldefs                  Allow multiple definitions
  -z execstack                Mark executable as requiring executable stack
  -z noexecstack              Mark executable as not requiring executable stack
  -z globalaudit              Mark executable requiring global auditing
  --audit=AUDITLIB            Specify a library to use for auditing
  -Bgroup                     Selects group name lookup rules for DSO
  --disable-new-dtags         Disable new dynamic tags
  --enable-new-dtags          Enable new dynamic tags
  --eh-frame-hdr              Create .eh_frame_hdr section
  --no-eh-frame-hdr           Do not create .eh_frame_hdr section
  --exclude-libs=LIBS         Make all symbols in LIBS hidden
  --hash-style=STYLE          Set hash style to sysv, gnu or both
  -P AUDITLIB, --depaudit=AUDITLIB
			      Specify a library to use for auditing dependencies
  -z combreloc                Merge dynamic relocs into one section and sort
  -z nocombreloc              Don't merge dynamic relocs into one section
  -z global                   Make symbols in DSO available for subsequently
			       loaded objects
  -z initfirst                Mark DSO to be initialized first at runtime
  -z interpose                Mark object to interpose all DSOs but executable
  -z lazy                     Mark object lazy runtime binding (default)
  -z loadfltr                 Mark object requiring immediate process
  -z nocopyreloc              Don't create copy relocs
  -z nodefaultlib             Mark object not to use default search paths
  -z nodelete                 Mark DSO non-deletable at runtime
  -z nodlopen                 Mark DSO not available to dlopen
  -z nodump                   Mark DSO not available to dldump
  -z now                      Mark object non-lazy runtime binding
  -z origin                   Mark object requiring immediate $ORIGIN
				processing at runtime
  -z relro                    Create RELRO program header (default)
  -z norelro                  Don't create RELRO program header
  -z separate-code            Create separate code program header
  -z noseparate-code          Don't create separate code program header (default)
  -z common                   Generate common symbols with STT_COMMON type
  -z nocommon                 Generate common symbols with STT_OBJECT type
  -z stack-size=SIZE          Set size of stack segment
  -z text                     Treat DT_TEXTREL in shared object as error
  -z notext                   Don't treat DT_TEXTREL in shared object as error
  -z textoff                  Don't treat DT_TEXTREL in shared object as error
elf_x86_64: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z noreloc-overflow         Disable relocation overflow check
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
  -z ibtplt                   Generate IBT-enabled PLT entries
  -z ibt                      Generate GNU_PROPERTY_X86_FEATURE_1_IBT
  -z shstk                    Generate GNU_PROPERTY_X86_FEATURE_1_SHSTK
  -z bndplt                   Always generate BND prefix in PLT entries
elf32_x86_64: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z noreloc-overflow         Disable relocation overflow check
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
  -z ibtplt                   Generate IBT-enabled PLT entries
  -z ibt                      Generate GNU_PROPERTY_X86_FEATURE_1_IBT
  -z shstk                    Generate GNU_PROPERTY_X86_FEATURE_1_SHSTK
elf_i386: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
  -z ibtplt                   Generate IBT-enabled PLT entries
  -z ibt                      Generate GNU_PROPERTY_X86_FEATURE_1_IBT
  -z shstk                    Generate GNU_PROPERTY_X86_FEATURE_1_SHSTK
elf_iamcu: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
elf_l1om: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
elf_k1om: 
  -z noextern-protected-data  Do not treat protected data symbol as external
  -z dynamic-undefined-weak   Make undefined weak symbols dynamic
  -z nodynamic-undefined-weak Do not make undefined weak symbols dynamic
  -z call-nop=PADDING         Use PADDING as 1-byte NOP for branch
i386pep: 
  --base_file <basefile>             Generate a base file for relocatable DLLs
  --dll                              Set image base to the default for DLLs
  --file-alignment <size>            Set file alignment
  --heap <size>                      Set initial size of the heap
  --image-base <address>             Set start address of the executable
  --major-image-version <number>     Set version number of the executable
  --major-os-version <number>        Set minimum required OS version
  --major-subsystem-version <number> Set minimum required OS subsystem version
  --minor-image-version <number>     Set revision number of the executable
  --minor-os-version <number>        Set minimum required OS revision
  --minor-subsystem-version <number> Set minimum required OS subsystem revision
  --section-alignment <size>         Set section alignment
  --stack <size>                     Set size of the initial stack
  --subsystem <name>[:<version>]     Set required OS subsystem [& version]
  --support-old-code                 Support interworking with old code
  --[no-]leading-underscore          Set explicit symbol underscore prefix mode
  --[no-]insert-timestamp            Use a real timestamp rather than zero. (default)
                                     This makes binaries non-deterministic
  --add-stdcall-alias                Export symbols with and without @nn
  --disable-stdcall-fixup            Don't link _sym to _sym@nn
  --enable-stdcall-fixup             Link _sym to _sym@nn without warnings
  --exclude-symbols sym,sym,...      Exclude symbols from automatic export
  --exclude-all-symbols              Exclude all symbols from automatic export
  --exclude-libs lib,lib,...         Exclude libraries from automatic export
  --exclude-modules-for-implib mod,mod,...
                                     Exclude objects, archive members from auto
                                     export, place into import library instead.
  --export-all-symbols               Automatically export all globals to DLL
  --kill-at                          Remove @nn from exported symbols
  --output-def <file>                Generate a .DEF file for the built DLL
  --warn-duplicate-exports           Warn about duplicate exports.
  --compat-implib                    Create backward compatible import libs;
                                       create __imp_<SYMBOL> as well.
  --enable-auto-image-base           Automatically choose image base for DLLs
                                       unless user specifies one
  --disable-auto-image-base          Do not auto-choose image base. (default)
  --dll-search-prefix=<string>       When linking dynamically to a dll without
                                       an importlib, use <string><basename>.dll
                                       in preference to lib<basename>.dll 
  --enable-auto-import               Do sophisticated linking of _sym to
                                       __imp_sym for DATA references
  --disable-auto-import              Do not auto-import DATA items from DLLs
  --enable-runtime-pseudo-reloc      Work around auto-import limitations by
                                       adding pseudo-relocations resolved at
                                       runtime.
  --disable-runtime-pseudo-reloc     Do not add runtime pseudo-relocations for
                                       auto-imported DATA.
  --enable-extra-pep-debug            Enable verbose debug output when building
                                       or linking to DLLs (esp. auto-import)
  --enable-long-section-names        Use long COFF section names even in
                                       executable image files
  --disable-long-section-names       Never use long COFF section names, even
                                       in object files
  --high-entropy-va                  Image is compatible with 64-bit address space
                                       layout randomization (ASLR)
  --dynamicbase			 Image base address may be relocated using
				       address space layout randomization (ASLR)
  --forceinteg		 Code integrity checks are enforced
  --nxcompat		 Image is compatible with data execution prevention
  --no-isolation		 Image understands isolation but do not isolate the image
  --no-seh			 Image does not use SEH. No SE handler may
				       be called in this image
  --no-bind			 Do not bind this image
  --wdmdriver		 Driver uses the WDM model
  --tsaware                  Image is Terminal Server aware
  --build-id[=STYLE]         Generate build ID
i386pe: 
  --base_file <basefile>             Generate a base file for relocatable DLLs
  --dll                              Set image base to the default for DLLs
  --file-alignment <size>            Set file alignment
  --heap <size>                      Set initial size of the heap
  --image-base <address>             Set start address of the executable
  --major-image-version <number>     Set version number of the executable
  --major-os-version <number>        Set minimum required OS version
  --major-subsystem-version <number> Set minimum required OS subsystem version
  --minor-image-version <number>     Set revision number of the executable
  --minor-os-version <number>        Set minimum required OS revision
  --minor-subsystem-version <number> Set minimum required OS subsystem revision
  --section-alignment <size>         Set section alignment
  --stack <size>                     Set size of the initial stack
  --subsystem <name>[:<version>]     Set required OS subsystem [& version]
  --support-old-code                 Support interworking with old code
  --[no-]leading-underscore          Set explicit symbol underscore prefix mode
  --thumb-entry=<symbol>             Set the entry point to be Thumb <symbol>
  --[no-]insert-timestamp            Use a real timestamp rather than zero (default).
                                     This makes binaries non-deterministic
  --add-stdcall-alias                Export symbols with and without @nn
  --disable-stdcall-fixup            Don't link _sym to _sym@nn
  --enable-stdcall-fixup             Link _sym to _sym@nn without warnings
  --exclude-symbols sym,sym,...      Exclude symbols from automatic export
  --exclude-all-symbols              Exclude all symbols from automatic export
  --exclude-libs lib,lib,...         Exclude libraries from automatic export
  --exclude-modules-for-implib mod,mod,...
                                     Exclude objects, archive members from auto
                                     export, place into import library instead.
  --export-all-symbols               Automatically export all globals to DLL
  --kill-at                          Remove @nn from exported symbols
  --output-def <file>                Generate a .DEF file for the built DLL
  --warn-duplicate-exports           Warn about duplicate exports
  --compat-implib                    Create backward compatible import libs;
                                       create __imp_<SYMBOL> as well.
  --enable-auto-image-base[=<address>] Automatically choose image base for DLLs
                                       (optionally starting with address) unless
                                       specifically set with --image-base
  --disable-auto-image-base          Do not auto-choose image base. (default)
  --dll-search-prefix=<string>       When linking dynamically to a dll without
                                       an importlib, use <string><basename>.dll
                                       in preference to lib<basename>.dll 
  --enable-auto-import               Do sophisticated linking of _sym to
                                       __imp_sym for DATA references
  --disable-auto-import              Do not auto-import DATA items from DLLs
  --enable-runtime-pseudo-reloc      Work around auto-import limitations by
                                       adding pseudo-relocations resolved at
                                       runtime.
  --disable-runtime-pseudo-reloc     Do not add runtime pseudo-relocations for
                                       auto-imported DATA.
  --enable-extra-pe-debug            Enable verbose debug output when building
                                       or linking to DLLs (esp. auto-import)
  --large-address-aware              Executable supports virtual addresses
                                       greater than 2 gigabytes
  --disable-large-address-aware      Executable does not support virtual
                                       addresses greater than 2 gigabytes
  --enable-long-section-names        Use long COFF section names even in
                                       executable image files
  --disable-long-section-names       Never use long COFF section names, even
                                       in object files
  --dynamicbase			 Image base address may be relocated using
				       address space layout randomization (ASLR)
  --forceinteg		 Code integrity checks are enforced
  --nxcompat		 Image is compatible with data execution prevention
  --no-isolation		 Image understands isolation but do not isolate the image
  --no-seh			 Image does not use SEH. No SE handler may
				       be called in this image
  --no-bind			 Do not bind this image
  --wdmdriver		 Driver uses the WDM model
  --tsaware                  Image is Terminal Server aware
  --build-id[=STYLE]         Generate build ID

Report bugs to <http://www.sourceware.org/bugzilla/>
```

