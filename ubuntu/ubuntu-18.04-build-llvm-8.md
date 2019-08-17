# Ubuntu-18.04 - Build LLVM-8 for `ppc64el`

## Overview

This document describes how to build the LLVM toolchain from sources for the `ppc64el` host architecture.

## Procedure

### Step 01.00: Setup.

#### Step 01.01: Install required packages.

```bash
sudo apt-get install \
gcc-8 g++-8 \
libedit-dev \
libncurses-dev \
libxml2-dev \
python-dev \
python3-dev \
swig
```

Configure alternatives for gcc:
```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 700 --slave /usr/bin/g++ g++ /usr/bin/g++-7
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8
```

If you'll wish to change the default gcc version later on, run `sudo update-alternatives --config gcc`. It will bring a prompt similar to this, which lets you pick the version to be used:
```bash
sudo update-alternatives --config gcc

There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-8   800       auto mode
  1            /usr/bin/gcc-7   700       manual mode
  2            /usr/bin/gcc-8   800       manual mode
```


#### Step 01.01: Clone LLVM sources.

```bash
: '
Information about available software versions:
01. LLVM: 5.0, 6.0, 7, 8, 9.
'

LLVM_VERSION='8.0.1'
GIT_BRANCH=release/`echo $LLVM_VERSION | cut -d. -f1`.x
GIT_TAG="llvmorg-$LLVM_VERSION"

mkdir -p /project/software/tool/llvm
cd /project/software/tool/llvm

# clone, checking out release branch
git clone --single-branch --depth=1 --branch=$GIT_BRANCH  https://github.com/llvm/llvm-project.git llvm-project
mkdir -p llvm-project/build; cd llvm-project/build;
```

#### Step 01.02: Patch sources.

001: compiler-rt: cmake test big endian fix

```bash
nano /project/software/tool/llvm/llvm-project/compiler-rt/cmake/builtin-config-ix.cmake
# add the following line
include (TestBigEndian)
```

002: [Bug 39696](https://bugs.llvm.org/show_bug.cgi?id=39696) - Workaround "error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression"

`libcxx` fails to compile with the following error:
```bash
libcxx/include/chrono:877:59: error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression
```

Apply this patch if you're building `llvm` from the `release/8.x` branch for `ppc64el` architecture.
This patch has already been applied to the llvm `master` and `release/9.x` branches, but it doesn't work
due to a wrong implementation. The correct fix for the [committed patch for llvm-project/libcxx](http://llvm.org/viewvc/llvm-project/libcxx/trunk/include/thread?view=patch&r1=357540&r2=357539&pathrev=357540) is shown below:

Create the following file in `llvm-project/libcxx/patch` folder:

`File: 001-libcxx-bug-39696-llvm-8.0.1-ppc64el.patch`
```diff
diff --git a/libcxx/include/thread b/libcxx/include/thread
index 8c0115f87..e439f60b9 100644
--- a/libcxx/include/thread
+++ b/libcxx/include/thread
@@ -435,7 +435,12 @@ sleep_for(const chrono::duration<_Rep, _Period>& __d)
     using namespace chrono;
     if (__d > duration<_Rep, _Period>::zero())
     {
+#if defined(_LIBCPP_COMPILER_GCC) && (__powerpc__ || __POWERPC__)
+        //  GCC's long double const folding is incomplete for IBM128 long doubles.
+        _LIBCPP_CONSTEXPR duration<long double> _Max = duration<long double>(ULLONG_MAX/1000000000ULL);
+#else
         _LIBCPP_CONSTEXPR duration<long double> _Max = nanoseconds::max();
+#endif
         nanoseconds __ns;
         if (__d < _Max)
         {
```

Apply the patch
```bash
cd /project/software/tool/llvm/llvm-project/libcxx
git apply patch/001-libcxx-bug-39696.patch
```

#### Step 01.03: Configure llvm.

Configure:
```bash
cd /project/software/tool/llvm/llvm-project/build/

# configure with no tests, no examples and no docs
ccmake \
-DCMAKE_INSTALL_PREFIX="/usr/local" \
-DCMAKE_BUILD_TYPE="Release" \
-DBUILD_SHARED_LIBS=ON \
-DLLVM_BUILD_BENCHMARKS=OFF \
-DLLVM_BUILD_DOCS=OFF \
-DLLVM_BUILD_EXAMPLES=OFF \
-DLLVM_BUILD_TESTS=OFF \
-DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;lld;lldb;libunwind;polly" \
-DLLVM_DEFAULT_TARGET_TRIPLE="powerpc64le-unknown-linux-gnu" \
-DLLVM_INCLUDE_TESTS=OFF \
-DLLVM_INCLUDE_EXAMPLES=OFF \
-DLLVM_INSTALL_TOOLCHAIN_ONLY=OFF \
../llvm
```

#### Step 01.04: Build.

Build:
```bash
make -j64
```

#### Step 01.05: [Optional] Test.

Re-configure:
```bash
cd /project/software/tool/llvm/llvm-project/build/

# optional: build tests. ensure that googletest is build as a shared library.
ccmake \
-DCMAKE_INSTALL_PREFIX="/project/software/tool/llvm/llvm-project/install" \
-DCMAKE_BUILD_TYPE="Release" \
-DBUILD_SHARED_LIBS=ON \
-DLLVM_BUILD_BENCHMARKS=OFF \
-DLLVM_BUILD_DOCS=OFF \
-DLLVM_BUILD_EXAMPLES=OFF \
-DLLVM_BUILD_TESTS=ON \
-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi;lldb;lld;polly" \
-DLLVM_DEFAULT_TARGET_TRIPLE="powerpc64le-unknown-linux-gnu" \
-DLLVM_INCLUDE_TESTS=ON \
-DLLVM_INCLUDE_EXAMPLES=OFF \
-DLLVM_INSTALL_TOOLCHAIN_ONLY=OFF \
../llvm
```

Test:
```bash
make -j64

# test
make -j64 check-all
```

#### Step 01.06: Install.

Install:
```bash
make -j64 install
```

Configure update alternatives for llvm binaries:
```bash
# configure update update-alternatives
sudo update-alternatives \
    --install /usr/local/bin/clang   clang   /usr/local/bin/clang-`echo $LLVM_VERSION | cut -d. -f1` 800
```

If you'll wish to change the default `clang` version later on, run `sudo update-alternatives --config clang`. It will bring a prompt similar to this, which lets you pick the version to be used:
```bash
sudo update-alternatives --config clang

There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-8   800       auto mode
  1            /usr/bin/gcc-7   700       manual mode
  2            /usr/bin/gcc-8   800       manual mode
```

Update `ldconfig`:
```bash
sudo ldconfig
```

#### Step 01.07: Verify installation.

Check `clang` version:
```
clang --version
clang version 8.0.1
Target: powerpc64le-unknown-linux-gnu
Thread model: posix
InstalledDir: /usr/local/bin
```

Compile and run a simple program:
```bash
cd /tmp
nano t.c
```

`File: /tmp/t.c`
```c
#include <stdio.h>
int main(int argc, char **argv) { printf("hello world\n"); }
```

Compile and run program:
```bash
$ clang t.c
$ ./a.out

hello world
```

---

## Issues

### GoogleTest

01. [Static build needs -fPIC CXXFLAG with CMake #854](https://github.com/google/googletest/issues/854)

Use `-fPIC` for `C_FLAGS` and `CXX_FLAGS` if building googletest as a static library:
```bash
ccmake \
-DCMAKE_INSTALL_PREFIX="/usr/local" \
-DCMAKE_BUILD_TYPE="Release" \
-DBUILD_GMOCK=ON \
-DBUILD_SHARED_LIBS=OFF \
-DCMAKE_CXX_FLAGS="-fPIC" \
-DCMAKE_C_FLAGS="-fPIC" \
../src
```

else, build googletest as a shared library:
```bash
ccmake \
-DCMAKE_INSTALL_PREFIX="/usr/local" \
-DCMAKE_BUILD_TYPE="Release" \
-DBUILD_GMOCK=ON \
-DBUILD_SHARED_LIBS=ON \
../src
```

---

## Blogs

### llvm

01. [Marshall's April Update - 20190501](https://cppalliance.org/marshall/2019/05/01/MarshallsMayUpdate.html)

- I spent a fair amount of time on Bug 39696 “Workaround “error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression”; which turned out to be a GCC bug on PowerPC machines.

---

## Issues

### llvm

01. [https://bugs.llvm.org/show_bug.cgi?id=39696](Bug 39696 - Workaround "error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression")

`libcxx` fails to compile with the following error:
```
In file included from /project/software/tool/llvm/llvm-project/libcxx/include/__mutex_base:15,
                 from /project/software/tool/llvm/llvm-project/libcxx/include/condition_variable:111,
                 from /project/software/tool/llvm/llvm-project/libcxx/src/condition_variable.cpp:14:
/project/software/tool/llvm/llvm-project/libcxx/include/chrono: In function ‘void std::__1::this_thread::sleep_for(const std::__1::chrono::duration<_Rep, _Period>&)’:
/project/software/tool/llvm/llvm-project/libcxx/include/thread:438:73:   in ‘constexpr’ expansion of ‘std::__1::chrono::duration<long double>(std::__1::chrono::duration<long long int, std::__1::ratio<1, 1000000000> >::max(), 0)’
/project/software/tool/llvm/llvm-project/libcxx/include/chrono:1064:64:   in ‘constexpr’ expansion of ‘std::__1::chrono::duration_cast<std::__1::chrono::duration<long double>, long long int, std::__1::ratio<1, 1000000000> >(__d)’
/project/software/tool/llvm/llvm-project/libcxx/include/chrono:916:67:   in ‘constexpr’ expansion of ‘std::__1::chrono::__duration_cast<std::__1::chrono::duration<long long int, std::__1::ratio<1, 1000000000> >, std::__1::chrono::duration<long double>, std::__1::ratio<1, 1000000000>, true, false>().std::__1::chrono::__duration_cast<std::__1::chrono::duration<long long int, std::__1::ratio<1, 1000000000> >, std::__1::chrono::duration<long double>, std::__1::ratio<1, 1000000000>, true, false>::operator()(__fd)’
/project/software/tool/llvm/llvm-project/libcxx/include/chrono:877:59: error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression
                            static_cast<_Ct>(__fd.count()) / static_cast<_Ct>(_Period::den)));
```

Ref:
- [patch](http://llvm.org/viewvc/llvm-project/libcxx/trunk/include/thread?view=patch&r1=357540&r2=357539&pathrev=357540)

02. [Bug 1538817 - g++ fails to compile libcxx on ppc64: error: ‘(9.223372036854775807e+18 / 1.0e+9)’ is not a constant expression](https://bugzilla.redhat.com/show_bug.cgi?id=1538817)

---

## Technotes

### llvm

01. [Getting Started: Building and Running Clang - llvm.org](http://clang.llvm.org/get_started.html)

02. [Getting Started with the LLVM System - llvm.org](http://llvm.org/docs/GettingStarted.html)

03. [Hacking on Clang - llvm.org](https://clang.llvm.org/hacking.html)

04. [LLVM Testing Infrastructure Guide - llvm.org](https://llvm.org/docs/TestingGuide.html)

---

## Related Links

### swig

01. [swig](https://github.com/swig/swig/wiki/Getting-Started)


## Related Topics

### CMake

01. [CMake: test endianness](https://cmake.org/pipermail/cmake/2016-January/062474.html)

```cmake
include (TestBigEndian)
```

### Ubuntu

01. [How to add slave to existing update-alternatives link group?](https://askubuntu.com/questions/964600/how-to-add-slave-to-existing-update-alternatives-link-group)

```bash
sudo update-alternatives --install "/usr/bin/java" "java" "/opt/jdk-10/bin/java" 10 \
    --slave "/usr/bin/jar"          "jar"           "/opt/jdk-10/bin/jar" \
    --slave "/usr/bin/jarsigner"    "jarsigner"     "/opt/jdk-10/bin/jarsigner" \
    --slave "/usr/bin/javac"        "javac"         "/opt/jdk-10/bin/javac" \
    --slave "/usr/bin/javadoc"      "javadoc"       "/opt/jdk-10/bin/javadoc" \
    --slave "/usr/bin/javap"        "javap"         "/opt/jdk-10/bin/javap" \
    --slave "/usr/bin/javaws"       "javaws"        "/opt/jdk-10/bin/javaws"
```

### gcc

01. [Install gcc-8 only on Ubuntu 18.04?](https://askubuntu.com/questions/1028601/install-gcc-8-only-on-ubuntu-18-04)
