---
title: "OpenJDK: Frequently Used Commands"
tags: jdk
---

We're using Ubuntu-24.04 and [OpenJDK-24] version.

## Dependencies

### Build Dependencies

A list of external libraries are required. See section `External Library Requirements`
in [OpenJDK build]. Here gives a simpler way:

```bash
$ sudo apt-get build-dep openjdk-21
```

### Boot JDK

A Boot JDK is required to bootstrap the build, and only a set of specific
versions are valid to build the target version. Check `boot-jdk.m4` file for more
for more details. Roughly boot JDK is looked in the order:

```text
1) --with-boot-jdk=/path/to/boot/jdk
2) JAVA_HOME
3) /usr/lib/jvm/jdk-* directory on Linux
```

Note that we can download the prebuilt releases from [OpenJDK GA Download].

### google test

gtest source is needed to run JTreg tests. And there is minimal version
requirement. See `LIB_TESTS_SETUP_GTEST` in JDK repo for more details.

```bash
# Download
git clone -b v1.15.2 https://github.com/google/googletest.git /tmp/gtest
# Pass to JDK configure
--with-gtest=/tmp/gtest
```

### Disassembly library: hsdis

- Blog-1: [Building hsdis in 2022. Jorn Vernee]
- Blog-2: [Developers disassemble! Use Java and hsdis to see it all]
- Prebuilt package: https://chriswhocodes.com/hsdis/

```bash
# Download d prebuilt package and put it under /usr/lib
sudo curl -Lo /usr/lib/hsdis-aarch64.so https://chriswhocodes.com/hsdis/hsdis-aarch64.so
```

### jtreg

JTreg binary is needed to run JDK conformance test and there is also minimal
version requirement. Check `LIB_TESTS_SETUP_JTREG` and `JTREG_MINIMUM_VERSION` in
JDK repo for more details. We look for JTreg binary in the order of

```text
1) --with-jtreg=/path/to/jtreg/library
2) JT_HOME
```

Here gives the instruction to build a jtreg. See section `Building jtreg` of
[Regression Test Harness for the JDK: jtreg] for more details.

```bash
# dependencies: install the prebuilt jdk in ubuntu
sudo apt-get install -y default-jdk

# download and build jtreg. note that 7.5.1+1 is the latest version.
cd /tmp
git clone https://github.com/openjdk/jtreg.git jtreg-tmp
cd jtreg-tmp
git checkout jtreg-7.5.1+1
bash make/build.sh --jdk /usr/lib/jvm/default-java
# build/images/jreg is our target
ls build/images/jtreg

# move it to ~/work/libs
cp -r build/images/jtreg /usr/local/lib

# the following configure option can be passed later
--with-jtreg=/usr/local/lib/jtreg
# OR we can simply export JT_HOME
export JT_HOME=/usr/local/lib/jtreg
```

### [JMH] jar

A set of jmh microbenchmarks are included inside OpenJDK code base.

```bash
# step-1: build the jar package
cd /path/to/your/jdk
sh make/devkit/createJMHBundle.sh

# step-2: the following configure option should be passed
--with-jmh=/path/to/your/jdk/build/jmh/jars
```

## Configure and Build

By default, the following commands can build one OopenJDK and test whether the OpenJDK works well.

```bash
# configure
mkdir -p jdk-build
cd jdk-build
bash /path/to/your/jdk/configure

# if succeeds, the configuration in details would be printed out.

# build
make images # by default, $nproc threads will start for building
make test-image

# check whether OpenJDK works or not
./jdk/bin/java --version
```
More options can be specify to configure:

```bash
# google test, jtreg and jmh, boot jdk
--with-gtest=/path/to/gtest
--with-jtreg=/path/to/jtreg
--with-jmh=/path/to/your/jdk/build/jmh/jars
--with-boot-jdk=/path/to/boot/jdk

# debug level, jvm varaints
--with-debug-level=release
--with-jvm-features=-compiler1
--disable-precompiled-headers

# specify GCC version, e.g.,
export CC=/usr/bin/gcc-12
export CXX=/usr/bin/g++-12
--with-toolchain-type=gcc

# specify LLVM version, e.g.,
export CC=/usr/lib/llvm-14/bin/clang
export CXX=/usr/lib/llvm-14/bin/clang++
--with-toolchain-type=clang

# specify compiler flags
--with-extra-cxxflags="-fsanitize=undefined"
--with-extra-cflags="-fsanitize=undefined"
--with-extra-ldflags="-fsanitize=undefined"
--with-extra-cflags="-Wno-nonnull -Wc++17-extensions"

# get the full compilation log
--with-extra-cflags=--verbose --with-extra-cxxflags=--verbose --with-extra-ldflags=--verbose
```

## JTreg test

Everything about JTreg can be found in [Regression Test Harness for the JDK: jtreg].
As stated in section `Running tests using jtreg` in [Regression Test Harness for the JDK: jtreg],
there are three ways to run jtreg tests:

> 1) Using the 'test' make target
> 2) Use convenience scripts to run jtreg
> 3) Run jtreg.jar directly

Here we give examples for `1)` and `2)`.
**Prerequisites**: `--with-jtreg=` or `JT_HOME` must be specified during the JDK build.

```bash
# 1
# running the tests under XX and YY directories with AA and BB runtime options specified.
# note that XX or YY can be one specific test case or one test directory.
make test TEST="XX YY" JTREG="VM_OPTIONS=AA BB"
# e.g.,
make test TEST=test/hotspot/jtreg/compiler/c2/TestAbs.java

# here is another example:
# we usually use the following command to run vector related tests.
make test TEST="test/hotspot/jtreg/compiler/vectorapi/ test/jdk/jdk/incubator/vector/ test/hotspot/jtreg/compiler/vectorization/" \
     JTREG="VM_OPTIONS=-XX:MaxVectorSize=16 -Djdk.incubator.vector.test.loop-iterations=300"

# 2
# jtreg options can be found in the reference.
# can specify the jtwork
# -v:summary can dump all the test summary
# test case: -dir: PATH-to-src/test AA, AA can be one test case or one test dir.
/path/to/your/jtreg -agentvm -a -ea -esa -v:error,fail -ignore:quiet -timeoutFactor:32 -J-Xmx768m \
  -vmoption:-Xcomp -vmoption:-Xmx768m -vmoptions:-Djdk.incubator.vector.test.loop-iterations=300 \
  -testjdk:/path/to/your/jdk/build/images/ -w:/tmp/jtwork -dir:/path/to/your/jdk/test \
  jdk/jdk/incubator/vector/VectorReshapeTests.java
```

The following JTreg options or VM options might be helpful.

```bash
# set the max output
make test TEST="test/jdk/jdk/incubator/vector/Short64VectorTests.java" JTREG="MAX_OUTPUT=2500000"

# specify the error log file
-XX:ErrorFile=/tmp/aa.log
# e.g.,
make test JTREG="VM_OPTIONS=-XX:UseBranchProtection=standard -XX:ErrorFile=/tmp/aa.log" TEST="test/jdk/java/lang/Thread/virtual/Locking.java"

# disable one intrinsic
-XX:+UnlockDiagnosticVMOptions -XX:DisableIntrinsic=_compareUnsigned_i

# print out some log
make test JTREG="VM_OPTIONS=-Xlog:continuations=trace:file=/tmp/bb.log -XX:ErrorFile=/tmp/aa.log" TEST="jdk/internal/vm/Continuation/Fuzz.java"

# check one flag is set or not
java -XX:+PrintFlagsFinal -XX:UseAVX=3 -version | grep UseAVX
```

It's highly recommended to read [New Test Framework with IR Verification] as well.

## JMH test

**Prerequisites**: `--with-jmh=` must be specified during the JDK build.

Similarly, there are two ways to evaluate jmh case, using `make test` and running `java -jar` binary directly.

```bash
# 1
# run several specific jmh cases
make test TEST="micro:AA micro:BB" MICRO='VM_OPTIONS=XX YY'

# e.g.,
make test TEST="micro:vm.compiler.VectorShiftRight"
make test TEST="micro:FloatingScalarVectorAbsDiff.testVectorAbsDiffFloat micro:Longs.compareUnsignedDirect"
make test TEST=micro:MessageDigests.getAndDigest MICRO='VM_OPTIONS=-XX:+UnlockDiagnosticVMOptions -XX:+UseSHA3Intrinsics -XX:+UseSHA512Intrinsics;RESULTS_FORMAT=text'

# 2
# run java -jar
java -jar XX.jar AA

# here show several use examples:
./jdk/bin/java -jar ./images/test/micro/benchmarks.jar Integers.compareUnsigned
# specify -f -i options
./jdk/bin/java -jar ./images/test/micro/benchmarks.jar -f1 -i10 Integers.compareUnsigned
# we can further specify some VM options
./jdk/bin/java -jar -XX:+UnlockDiagnosticVMOptions -XX:DisableIntrinsic=_compareUnsigned_i ./images/test/micro/benchmarks.jar -f1 -i10 Integers.compareUnsigned
# we may use numactl to specify the CPU core, e.g.
numactl -C 140 ./jdk/bin/java -jar ./images/test/micro/benchmarks.jar -f1 -i10 Integers.compareUnsigned

# perf (important)
# add `-prof perfasm`, e.g.,
numactl -C 140 ./jdk/bin/java -jar ./images/test/micro/benchmarks.jar -f1 -i10 Integers.compareUnsigned -prof perfasm 2>&1 | tee /tmp/cmp-unsigned.log
```

## IdealGraphVisualizer: igv

Reference: [Improving the Ideal Graph Visualizer for better comprehension of Java's main JIT compiler]

### Build and install

Please check file `src/utils/IdealGraphVisualizer/README.md` in JDk repo for more details

```bash
# build igv on macOS

# install brew and openjdk-21
brew install openjdk@21

# install maven
brew install mvn

# download openjdk code base
mkdir ~/igv
cd ~/igv
git clone --branch master --depth 1 https://github.com/openjdk/jdk

# install igv
cd ~/igv/jdk/src/utils/IdealGraphVisualizer/
mvn install

# run igv
bash ./igv.sh
```

### usage

```bash
# the following options can be specified
-XX:PrintIdealGraphLevel=N
-XX:PrintIdealGraphAddress=XX
-XX:PrintIdealGraphPort=XX
-XX:PrintIdealGraphFile=filename
-XX:CompilerDirectivesFile=compiler_directive.txt

# e.g.,
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -Xcomp -XX:CompileCommand=dontinline,*Bar.sum -XX:CompileCommand=compileonly,*Bar.sum \
  -XX:PrintIdealGraphLevel=4 -XX:PrintIdealGraphAddress="10.123.123.45" -XX:PrintIdealGraphPort=4444 \
  -XX:PrintIdealGraphFile="file.xml" -XX:+PrintIdeal Bar.java
```

## Auto-generated files

### AArch64 assembler test

In some cases, we may add new AArch64 instruction and then the corresponding assembler test is added accordingly.

Take [8292587: AArch64: Support SVE fabd instruction] as an example. We updated
`aarch64-asmtest.py` and we should use the following command to generate the new `asmtest.out.h` file.

```bash
cd /path/to/your/jdk
cd test/hotspot/gtest/aarch64
python2 aarch64-asmtest.py | expand > asmtest.out.h
```

### AArch64 ad file

In some cases, we may add new matching rules for instruction selection in `AD` file.
 As the `ad` file is auto generated by the corresponding `m4` file, the following command is needed.

Take [8292587: AArch64: Support SVE fabd instruction] as an example.
We added one new rule, i.e. `vfabd_sve` in the `m4` file.

```bash
cd /path/to/your/jdk
cd src/hotspot/cpu/aarch64
m4 aarch64_vector_ad.m4 > aarch64_vector.ad # will override the original aarch64_vector.ad
```

## perf

TODO: libjvmti.so, async-profiler

## Misc

### usefule links

- [Chris Newland's site]: bytecode, JEP, VM options, VM intrinsics, GC
- [Java on Graviton by AWS]

TODO: useful url: jeps...

<!-- Links -->
[OpenJDK-24]: https://github.com/openjdk/jdk24u
[OpenJDK build]: https://openjdk.org/groups/build/doc/building.html
[OpenJDK test]: https://openjdk.org/groups/build/doc/testing.html
[OpenJDK GA Download]: https://jdk.java.net/archive/
[Building hsdis in 2022. Jorn Vernee]: https://jornvernee.github.io/hsdis/2022/04/30/hsdis.html
[Developers disassemble! Use Java and hsdis to see it all]: https://blogs.oracle.com/javamagazine/post/java-hotspot-hsdis-disassembler
[Regression Test Harness for the JDK: jtreg]: https://openjdk.org/jtreg/
[JMH]: https://github.com/openjdk/jmh-jdk-microbenchmarks
[New Test Framework with IR Verification]: https://cr.openjdk.org/~chagedorn/TestFramework/TestFramework.pdf
[Improving the Ideal Graph Visualizer for better comprehension of Java's main JIT compiler]: https://robcasloz.github.io/blog/2021/04/22/improving-the-ideal-graph-visualizer.html
[8292587: AArch64: Support SVE fabd instruction]: https://bugs.openjdk.org/browse/JDK-8292587
[Chris Newland's site]: https://byte-me.dev/
[Java on Graviton by AWS]: https://github.com/aws/aws-graviton-getting-started/blob/main/java.md