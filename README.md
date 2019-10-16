# osblog
The Adventures of OS

Installation as follows (tested on Ubuntu 12.04 x64):

Pre-requesites:
---------------
```console
me@machine:$ sudo apt-get install libmpc-dev libmpfi-dev libgmp-dev libjemalloc-dev
me@machine:$ mkdir build-binutils && mkdir build-gcc-s1 build-gcc-s2 && mkdir build-linux && mkdir build-glibc-s1 build-glibc-s2 && mkdir build-qemu
me@machine:$ export PREFIX=/opt/riscv64_oslbog
me@machine:$ export SYSROOT=$PREFIX/sysroot # Note: You may need to create PREFIX and make it rwx for everyone

# Optional: install make 4.x (will be installed in /usr/local/bin/):
me@machine:$ wget http://ftp.gnu.org/gnu/make/make-4.1.tar.gz
me@machine:$ tar xvf make-4.1.tar.gz
me@machine:$ cd make-4.1/
me@machine:$ ./configure
me@machine:$ make -j16
me@machine:$ sudo make install
me@machine:$ cd ..
```


Cloning repos:
-------------
```console
me@machine:$ git clone -b binutils-2_32 --single-branch git://sourceware.org/git/binutils-gdb.git
me@machine:$ git clone -b gcc-9_2_0-release --single-branch https://github.com/gcc-mirror/gcc.git
me@machine:$ git clone -b v5.3 --single-branch https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
me@machine:$ git clone -b glibc-2.29 --single-branch git://sourceware.org/git/glibc.git
me@machine:$ git clone -b v4.1.0 --single-branch git://git.qemu-project.org/qemu.git
```

Compiling and Installing:
------------------------
```console
# gcc -- see https://gcc.gnu.org/wiki/InstallingGCC
# gcc stage 1
me@machine:$ cd gcc 
me@machine:$ ./contrib/download_prerequisites
me@machine:$ cd ../build-gcc-s1
me@machine:$ ../gcc/configure --target=riscv64-unknown-linux-gnu --prefix=$PREFIX --with-sysroot=$SYSROOT --with-newlib --without-headers --disable-shared --disable-threads --with-system-zlib --enable-tls --enable-languages=c --disable-libatomic --disable-libmudflap --disable-libssp --disable-libquadmath --disable-libgomp --disable-nls --disable-bootstrap --enable-checking=yes --disable-multilib --with-abi=lp64 --with-arch=rv64g
me@machine:$ make -j16
me@machine:$ make install
me@machine:$ cd ..

# linux headers
me@machine:$ cd linux
me@machine:$ wget https://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/eoan/patch/?id=9b4605005a307039e66de2db416ed826399e5103 -O kernel-9b4605005a307039e66de2db416ed826399e5103.patch
me@machine:$ git apply --stat kernel-9b4605005a307039e66de2db416ed826399e5103.patch
me@machine:$ make ARCH=riscv INSTALL_HDR_PATH=$PWD/../build-riscv64-unknown-linux-gnu-linux-headers defconfig
me@machine:$ make ARCH=riscv INSTALL_HDR_PATH=$PWD/../build-riscv64-unknown-linux-gnu-linux-headers headers_install
me@machine:$ cd ..

# glibc stage 1
me@machine:$ cd build-glibc-s1
me@machine:$ ../glibc/configure CXX=$PREFIX/bin/riscv64-unknown-linux-gnu-gcc CC=$PREFIX/bin/riscv64-unknown-linux-gnu-gcc --host=riscv64-unknown-linux-gnu --prefix=$SYSROOT/usr --enable-shared --with-headers=../build-riscv64-unknown-linux-gnu-linux-headers/include --disable-multilib --enable-kernel=3.0.0 --enable-languages=c,c++
me@machine:$ make install-headers
me@machine:$ cp -a ../build-riscv64-unknown-linux-gnu-linux-headers/include/* $SYSROOT/usr/include/
me@machine:$ cd ..

# glibc stage 2
me@machine:$ cd build-glibc-s2
me@machine:$ ../glibc/configure CXX=$PREFIX/bin/riscv64-unknown-linux-gnu-gcc CC=$PREFIX/bin/riscv64-unknown-linux-gnu-gcc --host=riscv64-unknown-linux-gnu --prefix=/usr --disable-werror --enable-tls --disable-nls --enable-shared --enable-obsolete-rpc --with-headers=$SYSROOT/usr/include --disable-multilib --enable-kernel=3.0.0 --enable-languages=c,c++
me@machine:$ make -j16
me@machine:$ make install install_root=$SYSROOT
# Note: original instructions asked for ln -s $SYSROOT/lib64 $SYSROOT/lib, but this failed and does not seem necessary
me@machine:$ cd ..

# gcc stage 2
me@machine:$ cd build-gcc-s2
me@machine:$ ../gcc/configure --target=riscv64-unknown-linux-gnu --prefix=$PREFIX --with-sysroot=$SYSROOT --with-system-zlib --enable-shared --enable-tls --enable-languages=c,c++ --disable-libmudflap --disable-libssp --disable-libquadmath --disable-nls --disable-bootstrap --disable-multilib --enable-checking=yes --with-abi=lp64
me@machine:$ make -j16
me@machine:$ make install
me@machine:$ cp -a $PREFIX/riscv64-unknown-linux-gnu/lib* $SYSROOT
me@machine:$ cd ..

# qemu
me@machine:$ cd build-qemu
me@machine:$ ../qemu/configure --prefix=$PREFIX --interp-prefix=$SYSROOT --target-list=riscv32-linux-user,riscv32-softmmu,riscv64-linux-user,riscv64-softmmu --enable-jemalloc --disable-werror
me@machine:$ make -j16
me@machine:$ make install

# wrapping up
me@machine:$ ln -s $PREFIX/bin/riscv64-unknown-linux-gnu-gcc $PREFIX/bin/riscv64-gcc
me@machine:$ ln -s $PREFIX/bin/riscv64-unknown-linux-gnu-g++ $PREFIX/bin/riscv64-g++
me@machine:$ ln -s $PREFIX/bin/riscv64-unknown-linux-gnu-objdump $PREFIX/bin/riscv64-objdump
me@machine:$ ln -s $PREFIX/bin/riscv64-unknown-linux-gnu-gdb $PREFIX/bin/riscv64-gdb

me@machine:$ cp -a $SYSROOT/lib/* $SYSROOT/usr/lib64/lp64/​
```

