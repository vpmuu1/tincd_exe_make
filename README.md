# tincd_exe_make
cross-compiling tinc for Windows under Linux using MinGW


https://www.tinc-vpn.org/examples/cross-compiling-64-bit-windows-binary/

changed:

wget https://github.com/gsliepen/tinc/archive/refs/tags/release-1.0.36.tar.gz

wget https://mirrors.aliyun.com/openbsd/LibreSSL/libressl-2.9.2.tar.gz

DESTDIR=$HOME/mingw64 make install


---------------------------------------------------------

Howto: cross-compiling tinc for Windows under Linux using MinGW

This howto describes how to create a 64-bit Windows binary of tinc. Although it is possible to compile tinc under Windows itself, cross-compiling it under Linux is much faster. It is also much easier to get all the dependencies in a modern distribution. Therefore, this howto deals with cross-compiling tinc with MinGW under Linux on a Debian distribution.
Overview

The idea is simple:

    Install 64-bit MinGW.
    Create a directory where we will perform all cross-compilations.
    Get all the necessary sources.
    Cross-compile everything.

Installing the prerequisites for cross-compilation

There are only a few packages that need to be installed as root to get started:


sudo apt-get install mingw-w64 git-core wget quilt

sudo apt-get build-dep tinc

Other Linux distributions may also have 64-bit MinGW packages, use their respective package management tools to install them. Debian installs the cross-compiler in /usr/x86_64-w64-mingw32/. Other distributions might install it in another directory however. Check in which directory it is installed, and replace all occurences of x86_64-w64-mingw32 in this example with the correct name from your distribution.
Setting up the build directory and getting the sources

We will create a directory called mingw64/ in the home directory. We use apt-get and wget to get the required libraries necessary for tinc, and use git to get the latest development version of tinc.


mkdir $HOME/mingw64

cd $HOME/mingw64

apt-get source liblzo2-dev zlib1g-dev libssl-dev

wget https://github.com/gsliepen/tinc/archive/refs/tags/release-1.0.36.tar.gz

Making cross-compilation easy

To make cross-compiling easy, we create a script called mingw64 that will set up the necessary environment variables so configure scripts and Makefiles will use the 64-bit MinGW version of GCC and binutils:

mkdir $HOME/bin

cat >$HOME/bin/mingw64 << 'EOF'

#!/bin/sh

PREFIX=x86_64-w64-mingw32

export CC=$PREFIX-gcc

export CXX=$PREFIX-g++

export CPP=$PREFIX-cpp

export RANLIB=$PREFIX-ranlib

export PATH="/usr/x86_64-w64-mingw32/bin:$PATH"

exec "$@"

EOF

chmod u+x $HOME/bin/mingw64

If $HOME/bin is not already part of your $PATH, you need to add it:

export PATH="$HOME/bin:$PATH"

We use this script to call ./configure and make with the right environment variables, but only when the ./configure script doesn’t support cross-compilation itself. You can also run the export commands from the mingw64 script by hand instead of calling the mingw64 script for every ./configure or make command, or execute $HOME/bin/mingw64 $SHELL to get a shell with these environment variables set, but in this howto we will call it explicitly every time it is needed.
Compiling LZO

Cross-compiling LZO is easy:

cd $HOME/mingw64/lzo2-2.08

./configure --host=x86_64-w64-mingw32

make

DESTDIR=$HOME/mingw64 make install

If it fails with a message about not passing the “ACC” test, create a symlink for the missing getopt.h file as mentioned above.
Compiling Zlib

Cross-compiling Zlib is also easy, but a plain make failed to compile the tests, so we only build the static library here:

cd $HOME/mingw64/zlib-1.2.8.dfsg

mingw64 ./configure --static

mingw64 make

DESTDIR=$HOME/mingw64 mingw64 make install


Compiling LibreSSL

Tinc can use either OpenSSL or LibreSSL. The latter is recommended.

wget https://mirrors.aliyun.com/openbsd/LibreSSL/libressl-2.9.2.tar.gz

cd $HOME/mingw/libressl-2.9.2

CC=x86_64-w64-mingw32-gcc ./configure --host=x86_64-w64-mingw32

make

DESTDIR=$HOME/mingw64 make install



Compiling tinc

Now that all the dependencies have been cross-compiled, we can cross-compile tinc. Since we use a clone of the git repository here, we need to run autoreconf first. If you want to cross-compile tinc from a released tarball, this is not necessary.

cd $HOME/mingw64/tinc

autoreconf -fsi

./configure --host=x86_64-w64-mingw32 --with-zlib=$HOME/mingw64/usr/local

make



