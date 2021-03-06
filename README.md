# jdk8u-xcode10

How to compile JDK 8 with the latest Xcode on macOS Mojave
(stooke@redhat.com, March 2019)

Currently (March 2019), jdk8u can only be compiled with XCode 4, which won't run on the latest macOS.
This repo contains patches and information for setting up an environment to compile a JDK using the very latest tools.

### Caveats:
There is an issue during shutdown that has been solved in later JDKs, a crash in `PerfData::~PerfData()`.
This can be avoided by (1) commenting out the code on macOS (fine if you didn't start your JVM via JNI)
or by setting a command line option to not capture performance data in the first place.


## Install Prerequisites

These are also required for building JDK 11, so your efforts won't be wasted here.

Install XCode 10, autoconf, freetype and mercurial
Install a bootstrap JDK; either JDK7 or 8.  
If you have a system JDK 8, the build should find it

```
curl -O -L http://ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz
tar -xzf autoconf-2.69.tar.gz
cd autoconf-2.69
./configure
sudo make

curl -O https://nongnu.freemirror.org/nongnu/freetype/freetype-2.9.tar.gz
tar -xvf freetype-2.9.tar.gz
cd freetype-2.9
./configure
sudo make

curl -O https://www.mercurial-scm.org/release/mercurial-4.9rc0.tar.gz
tar -xvf mercurial-4.9rc0.tar.gz
cd mercurial-4.9rc0/
make local
```

## download the JDK and all subrepos

```
hg clone http://hg.openjdk.java.net/jdk8u/jdk8u-dev jdk8u-dev
cd jdk8u-dev
chmod 755 get_source.sh configure
./get_source.sh
```

## install the patches

```
cd jdk8u-dev
hg import --no-commit ../jdkbase-macos.patch
(you might get an error in patching generated_configure.sh; ignore it or delete the file)
cd jdk
hg import --no-commit ../../jdk-macos.patch
```

## configure the JDK

```
./configure --with-toolchain-type=clang
```
or 

```
./configure --with-toolchain-type=clang --with-debug-level=slowdebug
```

## build the JDK

```
make images COMPILER_WARNINGS_FATAL=false
```

## run!

```
./build/maxosx-x86_64-normal-server-release/images/j2sdk-image/bin/java
./build/maxosx-x86_64-normal-server-slowdebug/images/j2sdk-image/bin/java
```

Notice the crash at shutdown!
To avoid this, use the flag `-XX:-UsePerfData`.
```
./build/maxosx-x86_64-normal-server-release/images/j2sdk-image/bin/java -XX:-UsePerfData
```

For javac, use the `-J` option to pass this flag to the underlying JVM.
```
./build/maxosx-x86_64-normal-server-release/images/j2sdk-image/bin/javac -J-XX:-UsePerfData Foo.java
```

