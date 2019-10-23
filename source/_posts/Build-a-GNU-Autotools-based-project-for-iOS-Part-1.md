---
title: 'Building a GNU-Autotools-based Project for iOS: Part 1'
date: 2019-10-17 14:22:34
tags: build, iOS
---

The open-source movement started way before the rise of the mobile scene, leaving enormous great free libraries and components lying around. Lots of the reusable codebases are usually only "a build away" from becoming mobile-ready. One of the most popular build systems for the open-source projects is GNU Autotools.

Recently I tried to build a few renowned C/C++ open-source projects for iOS, only to find that:

1. Nothing works out of the box;
2. The scattered info on the Internet is often inconsistent or out of date.

... familiar pattern for a platform with breaking changes every so often, keeping everyone busy and unable to document things tightly. But here is my $0.02. Hope it'll help folks who got bruises out of trials-and-errors.

In Part 1, I'll focus on building static libs for iOS.


## The Environment

Check my setup before getting excited. You know why, having tried and failed upon many StackOverflow tips.

- macOS 10.14.6
- iOS 13.1
- Xcode 11.1

If you can't get my solution working for your own projects, e.g., when it's based on a more recent environment, it's quite possible that you need a next-gen tip.


## The Working Solution

For the impatient, here is the build script to place in the root folder of your autotool-based project

```sh
#! /bin/sh

#
# Build for iOS 64bit-ARM variants and iOS Simulator
# - Place the script at project root
# - Customize MIN_IOS_VERSION and other flags as needed
# 
# Test Environment
# - macOS 10.14.6
# - iOS 13.1
# - Xcode 11.1
#

Build() {
    # Ensure -fembed-bitcode builds, as workaround for libtool macOS bug
    export MACOSX_DEPLOYMENT_TARGET="10.4"
    # Get the correct toolchain for target platforms
    export CC=$(xcrun --find --sdk "${SDK}" clang)
    export CXX=$(xcrun --find --sdk "${SDK}" clang++)
    export CPP=$(xcrun --find --sdk "${SDK}" cpp)
    export CFLAGS="${HOST_FLAGS} ${OPT_FLAGS}"
    export CXXFLAGS="${HOST_FLAGS} ${OPT_FLAGS}"
    export LDFLAGS="${HOST_FLAGS}"

    EXEC_PREFIX="${PLATFORMS}/${PLATFORM}"
    ./configure \
        --host="${CHOST}" \
        --prefix="${PREFIX}" \
        --exec-prefix="${EXEC_PREFIX}" \
        --enable-static \
        --disable-shared  # Avoid Xcode loading dylibs even when staticlibs exist

    make clean
    mkdir -p "${PLATFORMS}" &> /dev/null
    make V=1 -j"${MAKE_JOBS}" --debug=j
    make install
}

echo "HI"

# Locations
ScriptDir="$( cd "$( dirname "$0" )" && pwd )"
cd - &> /dev/null
PREFIX="${ScriptDir}"/_build
PLATFORMS="${PREFIX}"/platforms
UNIVERSAL="${PREFIX}"/universal

# Compiler options
OPT_FLAGS="-O3 -g3 -fembed-bitcode"
MAKE_JOBS=8
MIN_IOS_VERSION=8.0

# Build for platforms
SDK="iphoneos"
PLATFORM="arm"
PLATFORM_ARM=${PLATFORM}
ARCH_FLAGS="-arch arm64 -arch arm64e"  # -arch armv7 -arch armv7s
HOST_FLAGS="${ARCH_FLAGS} -miphoneos-version-min=${MIN_IOS_VERSION} -isysroot $(xcrun --sdk ${SDK} --show-sdk-path)"
CHOST="arm-apple-darwin"
Build

SDK="iphonesimulator"
PLATFORM="x86_64-sim"
PLATFORM_ISIM=${PLATFORM}
ARCH_FLAGS="-arch x86_64"
HOST_FLAGS="${ARCH_FLAGS} -mios-simulator-version-min=${MIN_IOS_VERSION} -isysroot $(xcrun --sdk ${SDK} --show-sdk-path)"
CHOST="x86_64-apple-darwin"
Build

# Create universal binary
cd "${PLATFORMS}/${PLATFORM_ARM}/lib"
LIB_NAME=`find . -iname *.a`
cd -
mkdir -p "${UNIVERSAL}" &> /dev/null
lipo -create -output "${UNIVERSAL}/${LIB_NAME}" "${PLATFORMS}/${PLATFORM_ARM}/lib/${LIB_NAME}" "${PLATFORMS}/${PLATFORM_ISIM}/lib/${LIB_NAME}"

echo "BYE"
```


## The Expected Results

Assuming you are at the project root, run the script and you should get:

- The static libs for arm64 family and iOS simulator under `./\_build/platforms/<ARCH>/lib`
- The universal binary for all architectures combined

Running a `lipo` check on the universal binary should give you something like these:

```sh
$ lipo -info /path/to/mylib/_build/arm/lib/libmylib.a
Architectures in the fat file: /path/to/mylib/_build/arm/lib/libmylib.a are: arm64 arm64e 
```

```sh
$ lipo -info /path/to/mylib/_build/x86_64-sim/lib/libmylib.a
Non-fat file: /path/to/mylib/_build/x86_64-sim/lib/libmylib.a is architecture: x86_64
```

```sh
$ lipo -info /path/to/mylib/_build/universal/libmylib.a
Architectures in the fat file: /path/to/mylib/_build/universal/libmylib.a are: x86_64 arm64 arm64e 
```

## The Lessons Learned

I feel obliged to write down the major gotchas that may help in the future.

### Compiler Executables

I lost most of my time to this. I started out by using the autotools compiler environment variables this way:

```sh
CC=clang
```

To my surprise, I then always ended up with builds holding `x86_64` instead of `arm64`. The correct way is now shown in the solution above. Before getting there, I was on the wrong track:  Fiddling with the architecture triplets that I copied from the Internet, unsure whether or not they could be trusted. I've tried numerous triplets, i.e., arch-vendor-os, to no avail. GNU is not big on documentation. The closest standard triplet lists I could find are:

- [Libtool platforms](http://git.savannah.gnu.org/cgit/libtool.git/tree/doc/PLATFORMS)
- [LLVM's llvm::Triple Class](http://llvm.org/doxygen/classllvm_1_1Triple.html)

These proved useless in my situation.

 To make it worse, `config.guess`  always gives me the wrong `x86_64` as well:

```sh
$ ./config.guess
x86_64-apple-darwin18.7.0
```

Knowing that autotools is just a wrapper over the real compilers, I've also tried to understand how the compiler works behind autotools. I created an Xcode project and watched the IDE build log, hunting for clues on magic flags. This proved useless as well, including the intriguing clang flag `-target arm64-apple-ios13.1`, not to be confused with the `--target` flag for `configure`.

The cross-compile idea also made me tweak the build/host/target combination over and over, only to find that having `--host` alone will suffice, the rest is implied by the assigned toolchain variables and flags.

### Bitcode

Since iOS 9, [enabling bitcode is required for library providers](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html#//apple_ref/doc/uid/TP40012582-CH35-SW2). However, to enabling bitcode without causing compiler errors such as:

```sh
ld: -bind_at_load and -bitcode_bundle (Xcode setting ENABLE_BITCODE=YES) cannot be used together
```

I have to put in a trick to signify the build machine version, which works around a supposed `libtool` bug:

```sh
export MACOSX_DEPLOYMENT_TARGET="10.4"
```

### Shared libs

Without adding `--disabled-shared`, autotools generate both dynamic and static libs for the project. This turns out to cue Xcode to try to load shared libs first (CRASH) even when I have not specified `-l` for the dylibs. So `--disabled-shared` is mandatory for using static libs.

## Conclusions

Autotools try to hide away compiler and OS details behind the magic `configure` command and its obscure options. In the Desktop era, the dirty work was done so well that all I needed was following the default trilogy steps. It's only until I get down to the dirty biz myself that I realize how much it takes to support a new platform. Although the new build systems like CMakes improve the cross-platform build environment,  the mobile devs inevitably bump into dinosaur autotools-based codebases, rigged with traps. Patience in research is the only cure, IMHO. 

In Part 2 (schedule TBD), I'll build a Framework for iOS using autotools.


## References

- [bitcode](https://stackoverflow.com/q/53121019/987846)
- [Compiler Executables](https://stackoverflow.com/q/26812060/987846)
- [Cross-compile for ARM with Autoconf](https://stackoverflow.com/questions/15234959/cross-compiling-for-arm-with-autoconf)
- [Shared Libs](https://stackoverflow.com/q/28679461/987846)



