---
layout: post
title: "gcc obj file symbols"
date: 2019-08-05 20:06:25 +0800
comments: true
categories:  [C, glibc, Linux]
---
自用

## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.2 LTS
Release:	18.04
Codename:	bionic

$ gcc --version
gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0

$ dpkg -l | grep libc6:amd64 
ii  libc6:amd64                                   2.27-3ubuntu1                                amd64        GNU C Library: Shared libraries
```

## 目錄

* [gcc7](#obsm-gcc)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtbegin.o](#obsm1)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o](#obsm2)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtbeginT.o](#obsm3)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtend.o](#obsm4)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o](#obsm5)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtfastmath.o](#obsm6)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadbegin.o](#obsm7)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadend.o](#obsm8)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadtable.o](#obsm9)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtprec32.o](#obsm10)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtprec64.o](#obsm11)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/crtprec80.o](#obsm12)
    * [/usr/lib/gcc/x86_64-linux-gnu/7/libasan_preinit.o](#obsm13)
* [glibc6](#obsm-glibc)
    * [/usr/lib/x86_64-linux-gnu/crt1.o](#obsm21)
    * [/usr/lib/x86_64-linux-gnu/crti.o](#obsm22)
    * [/usr/lib/x86_64-linux-gnu/crtn.o](#obsm23)
    * [/usr/lib/x86_64-linux-gnu/gcrt1.o](#obsm24)
    * [/usr/lib/x86_64-linux-gnu/grcrt1.o](#obsm25)
    * [/usr/lib/x86_64-linux-gnu/libtsan_preinit.o](#obsm26)
    * [/usr/lib/x86_64-linux-gnu/Mcrt1.o](#obsm27)
    * [/usr/lib/x86_64-linux-gnu/rcrt1.o](#obsm28)
    * [/usr/lib/x86_64-linux-gnu/Scrt1.o](#obsm29)

<a name="obsm-gcc"></a>
## gcc 7

<a name="obsm1"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtbegin.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 b completed.7697
0000000000000000 a crtstuff.c
0000000000000000 d .data
0000000000000000 t deregister_tm_clones
0000000000000070 t __do_global_dtors_aux
0000000000000000 t __do_global_dtors_aux_fini_array_entry
0000000000000000 D __dso_handle
0000000000000000 t .fini_array
00000000000000a0 t frame_dummy
0000000000000000 t __frame_dummy_init_array_entry
0000000000000000 t .init_array
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000000 n .note.GNU-stack
0000000000000030 t register_tm_clones
0000000000000000 t .text
                 U __TMC_END__
0000000000000000 d __TMC_LIST__
0000000000000000 d .tm_clone_table
```

<a name="obsm2"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 b completed.7697
0000000000000000 a crtstuff.c
                 w __cxa_finalize
0000000000000000 d .data
0000000000000000 d .data.rel.local
0000000000000000 t deregister_tm_clones
0000000000000090 t __do_global_dtors_aux
0000000000000000 t __do_global_dtors_aux_fini_array_entry
0000000000000000 D __dso_handle
0000000000000000 t .fini_array
00000000000000d0 t frame_dummy
0000000000000000 t __frame_dummy_init_array_entry
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 t .init_array
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000000 n .note.GNU-stack
0000000000000040 t register_tm_clones
0000000000000000 t .text
                 U __TMC_END__
0000000000000000 d __TMC_LIST__
0000000000000000 d .tm_clone_table
```

<a name="obsm3"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginT.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 b completed.7146
0000000000000000 a crtstuff.c
0000000000000000 d .data
                 w __deregister_frame_info
0000000000000000 t deregister_tm_clones
0000000000000070 t __do_global_dtors_aux
0000000000000000 t __do_global_dtors_aux_fini_array_entry
0000000000000000 D __dso_handle
0000000000000000 r .eh_frame
0000000000000000 r __EH_FRAME_BEGIN__
0000000000000000 t .fini_array
00000000000000b0 t frame_dummy
0000000000000000 t __frame_dummy_init_array_entry
0000000000000000 t .init_array
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000000 n .note.GNU-stack
0000000000000020 b object.7151
                 w __register_frame_info
0000000000000030 t register_tm_clones
0000000000000000 t .text
                 U __TMC_END__
0000000000000000 d __TMC_LIST__
0000000000000000 d .tm_clone_table
```

<a name="obsm4"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtend.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtstuff.c
0000000000000000 d .data
0000000000000000 r .eh_frame
0000000000000000 r __FRAME_END__
0000000000000000 n .note.GNU-stack
0000000000000000 t .text
0000000000000000 D __TMC_END__
0000000000000000 d .tm_clone_table
```

<a name="obsm5"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtstuff.c
0000000000000000 d .data
0000000000000000 r .eh_frame
0000000000000000 r __FRAME_END__
0000000000000000 n .note.GNU-stack
0000000000000000 t .text
0000000000000000 D __TMC_END__
0000000000000000 d .tm_clone_table
```

<a name="obsm6"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtfastmath.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtfastmath.c
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_loc
0000000000000000 N .debug_ranges
0000000000000000 N .debug_str
0000000000000000 r .eh_frame
0000000000000000 t .init_array
0000000000000000 n .note.GNU-stack
0000000000000000 t set_fast_math
0000000000000000 t .text
0000000000000000 t .text.startup
```

<a name="obsm7"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadbegin.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 d .data
0000000000000000 r .gnu.offload_funcs
0000000000000000 r .gnu.offload_vars
0000000000000000 n .note.GNU-stack
0000000000000000 R __offload_func_table
0000000000000000 a offloadstuff.c
0000000000000000 R __offload_var_table
0000000000000000 t .text
```

<a name="obsm8"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadend.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 d .data
0000000000000000 r .gnu.offload_funcs
0000000000000000 r .gnu.offload_vars
0000000000000000 n .note.GNU-stack
0000000000000000 R __offload_funcs_end
0000000000000000 a offloadstuff.c
0000000000000000 R __offload_vars_end
0000000000000000 t .text
```

<a name="obsm9"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtoffloadtable.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 d .data
0000000000000000 n .note.GNU-stack
                 U __offload_funcs_end
                 U __offload_func_table
0000000000000000 a offloadstuff.c
0000000000000000 R __OFFLOAD_TABLE__
                 U __offload_vars_end
                 U __offload_var_table
0000000000000000 r .rodata
0000000000000000 t .text
```

<a name="obsm10"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtprec32.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtprec.c
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_ranges
0000000000000000 N .debug_str
0000000000000000 r .eh_frame
0000000000000000 t .init_array
0000000000000000 n .note.GNU-stack
0000000000000000 t set_precision
0000000000000000 t .text
0000000000000000 t .text.startup
```

<a name="obsm11"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtprec64.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtprec.c
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_ranges
0000000000000000 N .debug_str
0000000000000000 r .eh_frame
0000000000000000 t .init_array
0000000000000000 n .note.GNU-stack
0000000000000000 t set_precision
0000000000000000 t .text
0000000000000000 t .text.startup
```

<a name="obsm12"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/crtprec80.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 a crtprec.c
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_ranges
0000000000000000 N .debug_str
0000000000000000 r .eh_frame
0000000000000000 t .init_array
0000000000000000 n .note.GNU-stack
0000000000000000 t set_precision
0000000000000000 t .text
0000000000000000 t .text.startup
```

<a name="obsm13"></a>
### /usr/lib/gcc/x86_64-linux-gnu/7/libasan_preinit.o:

```
                 U __asan_init
0000000000000000 a asan_preinit.cc
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_str
0000000000000000 D __local_asan_preinit
0000000000000000 n .note.GNU-stack
0000000000000000 d .preinit_array
0000000000000000 t .text
```

<a name="obsm-glibc"></a>
## glibc6

<a name="obsm21"></a>
### /usr/lib/x86_64-linux-gnu/crt1.o:

```
0000000000000000 b .bss
0000000000000000 d .data
0000000000000000 D __data_start
0000000000000000 W data_start
0000000000000030 T _dl_relocate_static_pie
0000000000000000 r .eh_frame
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
0000000000000000 r .note.ABI-tag
0000000000000000 n .note.GNU-stack
0000000000000000 r .rodata.cst4
0000000000000000 T _start
0000000000000000 t .text
```

<a name="obsm22"></a>
### /usr/lib/x86_64-linux-gnu/crti.o:

```
0000000000000000 b .bss
0000000000000000 d .data
0000000000000000 T _fini
0000000000000000 t .fini
                 U _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000000 T _init
0000000000000000 t .init
0000000000000000 n .note.GNU-stack
0000000000000000 t .text
```

<a name="obsm23"></a>
### /usr/lib/x86_64-linux-gnu/crtn.o:

```
nm: /usr/lib/x86_64-linux-gnu/crtn.o: no symbols
```

<a name="obsm24"></a>
### /usr/lib/x86_64-linux-gnu/gcrt1.o:

```
                 U atexit
0000000000000000 b .bss
0000000000000000 b called.4604
0000000000000000 d .data
0000000000000000 D __data_start
0000000000000000 W data_start
0000000000000080 T _dl_relocate_static_pie
0000000000000000 r .eh_frame
                 U etext
                 U __GI_memcpy
                 U __GI_memmove
                 U __GI_memset
                 U _GLOBAL_OFFSET_TABLE_
0000000000000030 T __gmon_start__
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
                 U _mcleanup
                 U __monstartup
0000000000000000 r .note.ABI-tag
0000000000000000 n .note.GNU-stack
0000000000000000 r .rodata.cst4
0000000000000000 T _start
0000000000000000 t .text
```

<a name="obsm25"></a>
### /usr/lib/x86_64-linux-gnu/grcrt1.o:

```
                 U atexit
0000000000000000 b .bss
0000000000000000 b called.4604
0000000000000000 d .data
0000000000000000 D __data_start
0000000000000000 W data_start
0000000000000000 r .eh_frame
                 U etext
                 U _GLOBAL_OFFSET_TABLE_
0000000000000030 T __gmon_start__
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
                 U _mcleanup
                 U __monstartup
0000000000000000 r .note.ABI-tag
0000000000000000 n .note.GNU-stack
0000000000000000 r .rodata.cst4
0000000000000000 T _start
0000000000000000 t .text
```

<a name="obsm26"></a>
### /usr/lib/x86_64-linux-gnu/libtsan_preinit.o:

```
0000000000000000 b .bss
0000000000000000 n .comment
0000000000000000 d .data
0000000000000000 N .debug_abbrev
0000000000000000 N .debug_aranges
0000000000000000 N .debug_info
0000000000000000 N .debug_line
0000000000000000 N .debug_str
0000000000000000 D __local_tsan_preinit
0000000000000000 n .note.GNU-stack
0000000000000000 d .preinit_array
0000000000000000 t .text
                 U __tsan_init
0000000000000000 a tsan_preinit.cc
```

<a name="obsm27"></a>
### /usr/lib/x86_64-linux-gnu/Mcrt1.o:

```
nm: /usr/lib/x86_64-linux-gnu/Mcrt1.o: no symbols
```

<a name="obsm28"></a>
### /usr/lib/x86_64-linux-gnu/rcrt1.o:

```
0000000000000000 b .bss
0000000000000000 d .data
0000000000000000 D __data_start
0000000000000000 W data_start
0000000000000000 r .eh_frame
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
0000000000000000 r .note.ABI-tag
0000000000000000 n .note.GNU-stack
0000000000000000 r .rodata.cst4
0000000000000000 T _start
0000000000000000 t .text
```

<a name="obsm29"></a>
### /usr/lib/x86_64-linux-gnu/Scrt1.o:

```
0000000000000000 b .bss
0000000000000000 d .data
0000000000000000 D __data_start
0000000000000000 W data_start
0000000000000000 r .eh_frame
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
0000000000000000 r .note.ABI-tag
0000000000000000 n .note.GNU-stack
0000000000000000 r .rodata.cst4
0000000000000000 T _start
0000000000000000 t .text
```
