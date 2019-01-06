# release pack

Download the binary in the [release tab](https://github.com/rdhafidh/myc) of this project.

## To check
```
hv@pc1:/work$ sha256sum clang71glibc2.11_libgcc_s_unwinder.7z

ca750ca8274ae1180d6d67ae40081a10e2ad595eee358eb4e20bf9a105c41578 clang71glibc2.11_libgcc_s_unwinder.7z
```
```
hv@pc1:/work$ 7za x clang71glibc2.11_libgcc_s_unwinder.7z
```
```
hv@pc1:/work$ ldd clang26122018/bin/clang++
        linux-vdso.so.1 =>  (0x00007fffdd3ff000)
        libpthread.so.0 => /lib/libpthread.so.0 (0x00007f5892e14000)
        libz.so.1 => /usr/lib/libz.so.1 (0x00007f5892bfd000)
        librt.so.1 => /lib/librt.so.1 (0x00007f58929f4000)
        libdl.so.2 => /lib/libdl.so.2 (0x00007f58927f0000)
        libncursesw.so.6 => /media/sharedserverku/work/clang26122018/bin/../lib/libncursesw.so.6 (0x00007f589258a000)
        libm.so.6 => /lib/libm.so.6 (0x00007f5892307000)
        libc++.so.1 => /media/sharedserverku/work/clang26122018/bin/../lib/libc++.so.1 (0x00007f589203d000)
        libc++abi.so.1 => /media/sharedserverku/work/clang26122018/bin/../lib/libc++abi.so.1 (0x00007f5891e06000)
        libgcc_s.so.1 => /media/sharedserverku/work/clang26122018/bin/../lib/libgcc_s.so.1 (0x00007f5891bef000)
        libc.so.6 => /lib/libc.so.6 (0x00007f589188d000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f5893044000)
```	

## Reproducing 
```
hv@pc1:/work$ cmake -DCLANG_DEFAULT_RTLIB=compiler-rt -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
-DCLANG_DEFAULT_LINKER=lld -DSANITIZER_CXX_ABI=libc++ -DLLVM_TARGETS_TO_BUILD="X86;AMDGPU" \
-DLLVM_ENABLE_LIBXML2=OFF_ -DCMAKE_C_COMPILER=$CC -DCMAKE_CXX_COMPILER=$CXX \
-DCMAKE_CXX_FLAGS="-stdlib=libc++" -DPYTHON_EXECUTABLE=/opt/py27/bin/python \
-DCMAKE_BUILD_TYPE=Release  \
-DCMAKE_INSTALL_PREFIX=/media/work/clang26122018 -G "Ninja" ..
```

```
hv@pc1:/work$ ninja && ninja install

hv@pc1:/work$ cd /media/work/clang26122018/lib && nano libc++.so 
```

and append ```-lgcc_s``` at the end.
 
```
INPUT(libc++.so.1 -lc++abi -lgcc_s)
```

## Note
The reason to use explicit link libgcc_s is that to make proper safe hard runtime dependency.
It's based on gcc 4.8.5 libgcc unwinder and glibc 2.11, so that should theoritically forward support all current known
linux glibc based distribution. This process guarantee will void if the base OS glibc version for compilation ( for which generated code )
is higher than glibc target OS version for that app deployment, because every linker will impose additional min requirements of glibc 
version based on what OS glibc they use when compiling.

