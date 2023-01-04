Use a generic target called ***i686-elf*** or ***arm-none-eabi***, which provides a toolchain targeting the `System V ABI`. 

The first thing is to set up a GCC #Cross-Compiler for **i686-elf** or **arm-none-eabi**.  We build a toolset running our host that can turn source code into object files for our target system.

- We will install our new compiler into `$HOME/opt/cross`
- We build everything out of the source directory tree, as is considered good practice.

### Installing Dependencies
``` Bash
sudo apt install build-essential
sudo apt install bison
sudo apt install flex
sudo apt install libgmp3-dev  # could have depend issues with this 
sudo apt install libmpc-dev
sudo apt install libmpfr-dev
sudo apt install texinfo
sudo apt install libcloog-isl-dev # no clue where to find this ? Builds without it 
sudo apt install libisl-dev
```

### Preparation
```bash
export PREFIX="$HOME/opt/cross"
export TARGET=arm-none-eabi # or replace with i686-elf 
export PATH="$PREFIX/bin:$PATH"
```
Add the installation prefix to the `PATH` of the current shell session. The prefix will configure the build process so that all the files of your cross-compiler environment end up in `$HOME/opt/cross`

### Binutils
Download the desired Binutils release [Index of /gnu/binutils](https://ftp.gnu.org/gnu/binutils/)
```bash
cd $HOME/src

curl https://ftp.gnu.org/gnu/binutils/binutils-2.39.tar.gz --output binutils-2.39.tar.gz

tar -zxvf binutils-2.39.tar.gz
```

Compile Binutils
```bash
cd $HOME/src

mkdir build-binutils
cd build-binutils
../binutils-2.39/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
make
make install
```

### GCC
Download the desired GCC release [Index of /gnu/gcc](https://ftp.gnu.org/gnu/gcc/)
```bash
cd $HOME/src

curl https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz --output gcc-12.2.0.tar.gz

tar -zxvf gcc-12.2.0.tar.gz
```

Build GCC
```bash
cd $HOME/src

# The $PREFIX/bin dir _must_ be in the PATH. We did that above.
which -- $TARGET-as || echo $TARGET-as is not in the PATH

mkdir build-gcc
cd build-gcc
../gcc-12.2.0/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c,c++ --without-headers
make all-gcc
make all-target-libgcc
make install-gcc
make install-target-libgcc
```
We build libgcc, a low-level support library that the compiler expects to be available at compile time. Linking against libgcc provides integer, floating point, decimal, stack unwinding (beneficial for exception handling) and other support functions.

## Using the Cross Compiler

We can now run our new 'naked' compiler. It does not yet have access to a C library or C runtime, so we cannot use any of the standard includes or create runnable binaries.
```bash
$HOME/opt/cross/bin/$TARGET-gcc --version
```

Use the new compiler simply by invoking `$TARGET-gcc`
```bash
export PATH="$HOME/opt/cross/bin:$PATH"
```
