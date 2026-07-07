# fuzzing-benchmark-seeds
Benchmark programs and initial seed corpora used in the fuzzing experiments.
# Benchmark Programs and Initial Seed Corpora

This repository provides the benchmark information, AFL instrumentation/build instructions, and initial seed corpora used in our fuzzing experiments.

This repository is intended to support the reproducibility of the experimental setup. It only includes the initial seed corpora for each target program, together with the corresponding benchmark source information, instrumentation/build procedures, and fuzzing command examples. The modified AFL implementation, complete raw fuzzing logs, and full experimental output directories are not included in this repository.

## Repository Structure

```text
fuzzing-benchmark-seeds/
├── README.md
├── as-new/
│   └── seeds/
├── bloaty/
│   └── seeds/
├── cxxfilt/
│   └── seeds/
├── libpng/
│   └── seeds/
├── libsndfile/
│   └── seeds/
├── lua/
│   └── seeds/
├── nm-new/
│   └── seeds/
├── objcopy/
│   └── seeds/
├── objdump/
│   └── seeds/
├── php/
│   └── seeds/
├── poppler/
│   └── seeds/
├── re2/
│   └── seeds/
├── readelf/
│   └── seeds/
├── size/
│   └── seeds/
├── sqlite3/
│   └── seeds/
└── strip-new/
    └── seeds/
```

Each `seeds/` directory contains the initial seed corpus used for the corresponding target program. In our experiments, all compared fuzzers used the same initial seed corpus for the same target program.

## Benchmark Classification

The target programs used in this study are divided into three categories: MAGMA, FuzzBench, and real-world programs.

### 1. MAGMA Benchmarks

The MAGMA benchmarks are used to evaluate the bug-triggering capability of fuzzers on vulnerability-injected target programs.

The following targets are included:

- libpng
- libsndfile
- lua
- php
- poppler
- sqlite3

### 2. FuzzBench Benchmarks

The FuzzBench benchmarks are used as additional public fuzzing targets to evaluate path exploration and testing capability.

The following targets are included:

- bloaty
- re2

### 3. Real-world Programs

The real-world programs are selected from binutils-2.40 to evaluate the applicability of the approach in practical scenarios.

The following 8 targets are included:

- as-new
- cxxfilt
- nm-new
- objcopy
- objdump
- readelf
- size
- strip-new

## Experimental Environment

All target programs were instrumented using AFL LLVM mode and compiled with AddressSanitizer enabled.

The general environment variables used in the experiments are shown below:

```bash
export AFL_PATH=/path/to/AFL
export PATH="$AFL_PATH:$PATH"

export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++

export AFL_USE_ASAN=1
export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export LSAN_OPTIONS='detect_leaks=0:symbolize=0'
export AFL_SKIP_CPUFREQ=1
```

In the following commands, `/path/to/` denotes the local experimental workspace and should be replaced with the actual path on the user's machine.

Before compiling the target programs, AFL LLVM mode should be built first:

```bash
make -C /path/to/AFL/llvm_mode
```

## Build Instructions for MAGMA Benchmarks

### 1. libpng

- Program: libpng
- Benchmark source: MAGMA
- Vulnerability patches: PNG001--PNG007
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `libpng_read_fuzzer`
- Initial seed directory: `libpng/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_libpng/libpng

patch -p1 --fuzz=3 < /path/to/magma/targets/libpng/patches/setup/libpng16.patch

for p in /path/to/magma/targets/libpng/patches/bugs/PNG0*.patch; do
  patch -p1 < "$p"
done
```

Note: Do not define `MAGMA_ENABLE_FIXES`; otherwise, the injected bugs may be disabled by the fixing branches. The macro `MAGMA_ENABLE_CANARIES` can be enabled if MAGMA canary logging is required.

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

cd /path/to/fuzzing_libpng/libpng
make distclean 2>/dev/null || make clean 2>/dev/null || true

export AS=/usr/bin/as
export AFL_PATH=/path/to/AFL
export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

./configure --disable-shared --enable-static
make -j"$(nproc)"
```

#### Build AFL Harness

```bash
cd /path/to/fuzzing_libpng/libpng

/path/to/AFL/afl-clang-fast++ -O1 -g -fno-omit-frame-pointer -fsanitize=address \
  -I. -Icontrib/oss-fuzz \
  contrib/oss-fuzz/libpng_read_fuzzer.cc \
  contrib/oss-fuzz/afl_driver.cpp \
  .libs/libpng16.a -lz -lm \
  -o /path/to/fuzzing_libpng/libpng_read_fuzzer
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1

/path/to/AFL/afl-fuzz \
  -i libpng/seeds \
  -o output/libpng \
  -m none \
  -d \
  -- /path/to/fuzzing_libpng/libpng_read_fuzzer @@
```

---

### 2. libsndfile

- Program: libsndfile
- Benchmark source: MAGMA
- Vulnerability patches: SND001--SND025
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `sndfile_fuzzer`
- Initial seed directory: `libsndfile/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_libsndfile/libsndfile

patch -p1 --fuzz=3 < /path/to/magma/targets/libsndfile/patches/setup/libsndfile.patch

for p in /path/to/magma/targets/libsndfile/patches/bugs/SND*.patch; do
  patch -p1 < "$p"
done
```

Note: Do not define `MAGMA_ENABLE_FIXES`; otherwise, the injected bugs may be disabled by the fixing branches. The macro `MAGMA_ENABLE_CANARIES` can be enabled if MAGMA canary logging is required.

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

mkdir -p /path/to/bin
ln -sf /usr/bin/python3 /path/to/bin/python
export PATH=/path/to/bin:$PATH

cd /path/to/fuzzing_libsndfile/libsndfile
./autogen.sh

make distclean 2>/dev/null || make clean 2>/dev/null || true

export AS=/usr/bin/as
export AFL_PATH=/path/to/AFL
export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"
export ASAN_OPTIONS=detect_leaks=0

./configure --disable-shared --enable-ossfuzzers
make
```

The generated AFL target is:

```text
/path/to/fuzzing_libsndfile/libsndfile/ossfuzz/sndfile_fuzzer
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1
export AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1

/path/to/AFL/afl-fuzz \
  -i libsndfile/seeds \
  -o output/libsndfile \
  -m none \
  -d \
  -- /path/to/fuzzing_libsndfile/libsndfile/ossfuzz/sndfile_fuzzer @@
```

---

### 3. lua

- Program: Lua
- Benchmark source: MAGMA
- Vulnerability patches: LUA001--LUA004
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `lua`
- Initial seed directory: `lua/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_lua/lua

patch -p1 --fuzz=3 < /path/to/magma/targets/lua/patches/setup/lua.patch

for p in /path/to/magma/targets/lua/patches/bugs/LUA0*.patch; do
  patch -p1 < "$p"
done
```

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

cd /path/to/fuzzing_lua/lua
make clean 2>/dev/null || true

export AS=/usr/bin/as
export AFL_PATH=/path/to/AFL
export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export LDFLAGS="-fsanitize=address"

make \
  CFLAGS="$CFLAGS -std=c99 -DLUA_USE_LINUX -fno-stack-protector -fno-common" \
  MYLDFLAGS="$LDFLAGS -Wl,-E" \
  MYLIBS="-ldl"
```

The generated AFL target is:

```text
/path/to/fuzzing_lua/lua/lua
```

#### Stability Adjustments

To improve fuzzing stability, the following source-level adjustments were applied to Lua:

```text
1. Fixed the initialization seed of math.random.
2. Fixed the string hash seed.
3. Disabled the randomized pivot in table.sort.
4. Fixed time-related functions such as os.clock(), os.date(), and os.time().
5. Disabled os.execute and io.popen to avoid excessive subprocess creation.
6. Disabled os.tmpname and io.tmpfile to avoid excessive temporary file generation.
```

After these modifications, the target should be rebuilt using the same build commands.

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1
export AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1

/path/to/AFL/afl-fuzz \
  -i lua/seeds \
  -o output/lua \
  -m none \
  -d \
  -- /path/to/fuzzing_lua/lua/lua @@
```

---

### 4. php

- Program: PHP
- Benchmark source: MAGMA
- Vulnerability patches: PHP001--PHP016
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing targets: `parser`, `json`, `exif`, `unserialize`, `unserializehash`, `execute`
- Initial seed directory: `php/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_php/php-src

patch -p1 --fuzz=3 < /path/to/magma/targets/php/patches/setup/setup.patch

for p in /path/to/magma/targets/php/patches/bugs/PHP0*.patch; do
  patch -p1 < "$p"
done
```

Note: Do not define `MAGMA_ENABLE_FIXES`; otherwise, the injected bugs may be disabled by the fixing branches. The macro `MAGMA_ENABLE_CANARIES` can be enabled if MAGMA canary logging is required.

#### Prepare AFL Driver

```bash
cp /path/to/fuzzing_libpng/libpng/contrib/oss-fuzz/afl_driver.cpp /path/to/fuzzing_php/afl_driver.cpp

/path/to/AFL/afl-clang-fast++ -O1 -g -fno-omit-frame-pointer -fsanitize=address \
  -c /path/to/fuzzing_php/afl_driver.cpp \
  -o /path/to/fuzzing_php/afl_driver.o
```

If `libc++` is not available on the system, modify `sapi/fuzzer/config.m4` and replace the C++ linker configuration for the fuzzer with:

```text
FUZZING_CC="$CXX"
```

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

cd /path/to/fuzzing_php/php-src

export AS=/usr/bin/as
export AFL_PATH=/path/to/AFL
export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1
export ASAN_OPTIONS=detect_leaks=0
export LSAN_OPTIONS=verbosity=0
export RE2C=/usr/bin/re2c

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export CXXFLAGS="$CFLAGS"
export EXTRA_CFLAGS="$CFLAGS -fno-sanitize=object-size"
export EXTRA_CXXFLAGS="$CXXFLAGS -fno-sanitize=object-size"

unset CFLAGS CXXFLAGS

LIB_FUZZING_ENGINE=/path/to/fuzzing_php/afl_driver.o ./configure \
  --disable-all \
  --enable-option-checking=fatal \
  --enable-fuzzer \
  --enable-exif \
  --enable-phar \
  --enable-intl \
  --without-pcre-jit \
  --disable-phpdbg \
  --disable-cgi \
  --with-pic

make
```

#### Optional Corpus and Dictionary Generation

```bash
cd /path/to/fuzzing_php/php-src

export ASAN_OPTIONS=detect_leaks=0
export LSAN_OPTIONS=verbosity=0

sapi/cli/php sapi/fuzzer/generate_unserialize_dict.php
sapi/cli/php sapi/fuzzer/generate_parser_corpus.php
sapi/cli/php sapi/fuzzer/generate_unserializehash_corpus.php
```

The generated AFL targets may include:

```text
/path/to/fuzzing_php/parser
/path/to/fuzzing_php/json
/path/to/fuzzing_php/exif
/path/to/fuzzing_php/unserialize
/path/to/fuzzing_php/unserializehash
/path/to/fuzzing_php/execute
```

#### AFL Run Command

Example for the `parser` target:

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1
export AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1
export TMPDIR=/path/to/fuzzing_php/tmp
mkdir -p /path/to/fuzzing_php/tmp

/path/to/AFL/afl-fuzz \
  -i php/seeds \
  -o output/php/parser \
  -m none \
  -d \
  -- /path/to/fuzzing_php/parser @@
```

Example for the `unserialize` target with a dictionary:

```bash
/path/to/AFL/afl-fuzz \
  -i php/seeds \
  -o output/php/unserialize \
  -x /path/to/fuzzing_php/dict/unserialize \
  -m none \
  -d \
  -- /path/to/fuzzing_php/unserialize @@
```

---

### 5. poppler

- Program: poppler
- Benchmark source: MAGMA
- Vulnerability patches: PDF001--PDF022
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `poppler_pdf_fuzzer`
- Initial seed directory: `poppler/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_poppler/poppler

for p in /path/to/magma/targets/poppler/patches/bugs/PDF*.patch; do
  patch -p1 --forward < "$p"
done
```

Note: Do not define `MAGMA_ENABLE_FIXES`; otherwise, the injected bugs may be disabled by the fixing branches. The macro `MAGMA_ENABLE_CANARIES` can be enabled if MAGMA canary logging is required.

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

export AS=/usr/bin/as
export AFL_PATH=/path/to/AFL
export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

POPPLER_SRC=/path/to/fuzzing_poppler/poppler
POPPLER_BUILD=/path/to/fuzzing_poppler/poppler/build

rm -rf "$POPPLER_BUILD"
mkdir -p "$POPPLER_BUILD"

cd "$POPPLER_BUILD"

cmake "$POPPLER_SRC" \
  -DCMAKE_BUILD_TYPE=Debug \
  -DBUILD_SHARED_LIBS=OFF \
  -DFONT_CONFIGURATION=generic \
  -DBUILD_GTK_TESTS=OFF \
  -DBUILD_QT5_TESTS=OFF \
  -DBUILD_CPP_TESTS=OFF \
  -DENABLE_GLIB=OFF \
  -DENABLE_GOBJECT_INTROSPECTION=OFF \
  -DENABLE_QT5=OFF \
  -DENABLE_LIBCURL=OFF \
  -DWITH_NSS3=OFF \
  -DENABLE_BOOST=OFF

make
```

#### Build AFL Harness

```bash
cp /path/to/fuzzing_libpng/libpng/contrib/oss-fuzz/afl_driver.cpp \
   /path/to/fuzzing_poppler/afl_driver.cpp

/path/to/AFL/afl-clang-fast++ -O1 -g -fno-omit-frame-pointer \
  -fsanitize=address \
  -I"$POPPLER_BUILD/cpp" -I"$POPPLER_SRC/cpp" \
  -I"$POPPLER_BUILD/poppler" -I"$POPPLER_SRC/poppler" \
  /path/to/magma/targets/poppler/src/pdf_fuzzer.cc \
  /path/to/fuzzing_poppler/afl_driver.cpp \
  "$POPPLER_BUILD/cpp/libpoppler-cpp.a" \
  "$POPPLER_BUILD/libpoppler.a" \
  -lfreetype -lcairo -lopenjp2 -lpng -ltiff -ljpeg -llcms2 -lz -lm -lpthread \
  -o /path/to/fuzzing_poppler/poppler_pdf_fuzzer
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1
export AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1

/path/to/AFL/afl-fuzz \
  -i poppler/seeds \
  -o output/poppler \
  -m none \
  -d \
  -- /path/to/fuzzing_poppler/poppler_pdf_fuzzer @@
```

---

### 6. sqlite3

- Program: SQLite
- Benchmark source: MAGMA
- Vulnerability patches: SQL001--SQL020
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `sqlite3_fuzz`
- Initial seed directory: `sqlite3/seeds/`

#### Apply Patches

```bash
cd /path/to/fuzzing_sqlite/sqlite

patch -p1 --fuzz=3 < /path/to/magma/targets/sqlite3/patches/setup/tclexec-ignore-stderr.patch

for p in /path/to/magma/targets/sqlite3/patches/bugs/SQL0*.patch; do
  patch -p1 < "$p"
done
```

#### Instrumented Build

```bash
make -C /path/to/AFL/llvm_mode

cd /path/to/fuzzing_sqlite/sqlite
make distclean 2>/dev/null || make clean 2>/dev/null || true

export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++
export AFL_USE_ASAN=1

export CFLAGS="-O1 -g -fno-omit-frame-pointer -fsanitize=address \
  -DSQLITE_MAX_LENGTH=128000000 \
  -DSQLITE_MAX_SQL_LENGTH=128000000 \
  -DSQLITE_MAX_MEMORY=25000000 \
  -DSQLITE_PRINTF_PRECISION_LIMIT=1048576 \
  -DSQLITE_DEBUG=1 \
  -DSQLITE_MAX_PAGE_COUNT=16384"

export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

./configure --disable-shared --enable-rtree
make clean
make -j"$(nproc)"
make sqlite3.c
```

#### Add AFL Driver

Create the following file:

```text
/path/to/fuzzing_sqlite/sqlite/test/afl_driver.c
```

with the following content:

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int LLVMFuzzerTestOneInput(const uint8_t* data, size_t size);

static uint8_t* read_file(FILE* f, size_t* out_size) {
  if (fseek(f, 0, SEEK_END) != 0) return NULL;
  long end = ftell(f);
  if (end <= 0) return NULL;
  if (fseek(f, 0, SEEK_SET) != 0) return NULL;

  size_t size = (size_t)end;
  uint8_t* data = (uint8_t*)malloc(size);
  if (!data) return NULL;

  if (fread(data, 1, size, f) != size) {
    free(data);
    return NULL;
  }

  *out_size = size;
  return data;
}

static uint8_t* read_stdin(size_t* out_size) {
  size_t cap = 4096, len = 0;
  uint8_t* data = (uint8_t*)malloc(cap);
  if (!data) return NULL;

  for (;;) {
    if (len == cap) {
      size_t new_cap = cap * 2;
      uint8_t* new_data = (uint8_t*)realloc(data, new_cap);
      if (!new_data) {
        free(data);
        return NULL;
      }
      data = new_data;
      cap = new_cap;
    }

    ssize_t n = read(STDIN_FILENO, data + len, cap - len);
    if (n <= 0) break;
    len += (size_t)n;
  }

  *out_size = len;
  return data;
}

int main(int argc, char** argv) {
  uint8_t* data = NULL;
  size_t size = 0;

  if (argc > 1 && argv[1] && argv[1][0] != '\0') {
    FILE* f = fopen(argv[1], "rb");
    if (!f) return 0;
    data = read_file(f, &size);
    fclose(f);
  } else {
    data = read_stdin(&size);
  }

  if (data && size > 0) LLVMFuzzerTestOneInput(data, size);
  free(data);
  return 0;
}
```

#### Build AFL Harness

```bash
cd /path/to/fuzzing_sqlite/sqlite

/path/to/AFL/afl-clang-fast -O1 -g -fno-omit-frame-pointer -fsanitize=address \
  -I. test/ossfuzz.c test/afl_driver.c sqlite3.o \
  -o /path/to/fuzzing_sqlite/sqlite3_fuzz \
  -pthread -ldl -lm
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'
export AFL_SKIP_CPUFREQ=1

/path/to/AFL/afl-fuzz \
  -i sqlite3/seeds \
  -o output/sqlite3 \
  -m none \
  -d \
  -- /path/to/fuzzing_sqlite/sqlite3_fuzz @@
```

## Build Instructions for FuzzBench Benchmarks

### 1. bloaty

- Program: bloaty
- Benchmark source: FuzzBench
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `fuzz_target_afl`
- Initial seed directory: `bloaty/seeds/`

#### Instrumented Build

```bash
export AFL_PATH=/path/to/AFL
export PATH="$AFL_PATH:$PATH"
export CC=afl-clang-fast
export CXX=afl-clang-fast++
export CFLAGS="-O1 -g -fsanitize=address -fno-omit-frame-pointer"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

mkdir -p /path/to/fuzzing_bloaty/work
cd /path/to/fuzzing_bloaty/work

cmake -DBUILD_TESTING=false /path/to/fuzzing_bloaty/bloaty
make -j2
```

#### Build AFL Harness

```bash
export AFL_PATH=/path/to/AFL
export PATH="$AFL_PATH:$PATH"
export CXX=afl-clang-fast++
export CXXFLAGS="-O1 -g -fsanitize=address -fno-omit-frame-pointer -std=c++17"

$CXX $CXXFLAGS \
  -I/path/to/fuzzing_bloaty/bloaty \
  -I/path/to/fuzzing_bloaty/bloaty/src \
  -I/path/to/fuzzing_bloaty/work/src \
  -I/path/to/fuzzing_bloaty/bloaty/third_party/abseil-cpp \
  -I/path/to/fuzzing_bloaty/bloaty/third_party/protobuf/src \
  -I/path/to/fuzzing_bloaty/bloaty/third_party/capstone/include \
  -I/path/to/fuzzing_bloaty/bloaty/third_party/re2 \
  /path/to/fuzzing_bloaty/bloaty/tests/fuzz_target.cc \
  /path/to/fuzzing_bloaty/bloaty/tests/fuzz_driver.cc \
  -o /path/to/fuzzing_bloaty/work/fuzz_target_afl \
  /path/to/fuzzing_bloaty/work/liblibbloaty.a \
  /path/to/fuzzing_bloaty/work/third_party/re2/libre2.a \
  /path/to/fuzzing_bloaty/work/third_party/capstone/libcapstone.a \
  /path/to/fuzzing_bloaty/work/third_party/protobuf/cmake/libprotobuf.a \
  -lz -lpthread
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'

/path/to/AFL/afl-fuzz \
  -i bloaty/seeds \
  -o output/bloaty \
  -m none \
  -d \
  -- /path/to/fuzzing_bloaty/work/fuzz_target_afl @@
```

---

### 2. re2

- Program: RE2
- Benchmark source: FuzzBench
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing target: `fuzzer_afl`
- Initial seed directory: `re2/seeds/`

#### Instrumented Build

```bash
export AFL_PATH=/path/to/AFL
export PATH="$AFL_PATH:$PATH"
export CC=afl-clang-fast
export CXX=afl-clang-fast++
export CFLAGS="-O1 -g -fsanitize=address -fno-omit-frame-pointer"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-fsanitize=address"

cd /path/to/fuzzing_re2/re2
make -j2
```

#### Build AFL Harness

```bash
export AFL_PATH=/path/to/AFL
export PATH="$AFL_PATH:$PATH"
export CXX=afl-clang-fast++
export CXXFLAGS="-O1 -g -fsanitize=address -fno-omit-frame-pointer"

/path/to/AFL/afl-clang-fast++ $CXXFLAGS \
  /path/to/fuzzbench/benchmarks/re2_fuzzer/target.cc \
  /path/to/fuzzing_re2/fuzz_driver.cc \
  -I /path/to/fuzzing_re2/re2 \
  /path/to/fuzzing_re2/re2/obj/libre2.a \
  -lpthread \
  -o /path/to/fuzzing_re2/fuzzer_afl
```

#### AFL Run Command

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0'

/path/to/AFL/afl-fuzz \
  -i re2/seeds \
  -o output/re2 \
  -m none \
  -d \
  -- /path/to/fuzzing_re2/fuzzer_afl @@
```

## Build Instructions for Real-world Programs

The real-world programs are selected from binutils-2.40. Eight target programs are used in the experiments:

- as-new
- cxxfilt
- nm-new
- objcopy
- objdump
- readelf
- size
- strip-new

The corresponding initial seed directories are:

```text
as-new/seeds/
cxxfilt/seeds/
nm-new/seeds/
objcopy/seeds/
objdump/seeds/
readelf/seeds/
size/seeds/
strip-new/seeds/
```

### binutils-2.40

- Program: binutils-2.40
- Benchmark source: RealWorld
- Instrumentation: AFL LLVM mode
- Sanitizer: AddressSanitizer
- Fuzzing targets: `as-new`, `cxxfilt`, `nm-new`, `objcopy`, `objdump`, `readelf`, `size`, `strip-new`

#### Install Dependencies

```bash
sudo apt-get update && sudo apt-get install -y \
  build-essential clang llvm bison flex texinfo \
  zlib1g-dev libzstd-dev
```

#### Instrumented Build

```bash
cd /path/to/real_project/build-afl-asan

export AFL_USE_ASAN=1
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0:allocator_may_return_null=1'

export CC=/path/to/AFL/afl-clang-fast
export CXX=/path/to/AFL/afl-clang-fast++

../binutils-2.40/configure \
  --disable-shared \
  --disable-nls \
  --disable-werror

make
```

#### AFL Run Command Example

The following command uses `readelf` as an example:

```bash
export ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0:allocator_may_return_null=1'
export AFL_SKIP_CPUFREQ=1

/path/to/AFL/afl-fuzz \
  -i readelf/seeds \
  -o output/readelf \
  -m none \
  -d \
  -- /path/to/real_project/build-afl-asan/binutils/readelf @@
```

Other binutils targets can be run in the same way by replacing the input seed directory, output directory, and target binary path. For example:

```text
as-new/seeds/      -> as-new
cxxfilt/seeds/     -> cxxfilt
nm-new/seeds/      -> nm-new
objcopy/seeds/     -> objcopy
objdump/seeds/     -> objdump
size/seeds/        -> size
strip-new/seeds/   -> strip-new
```

## Notes

- `/path/to/` should be replaced with the actual local experimental path.
- For the same target program, all fuzzers used the same initial seed corpus.
- All target programs were instrumented using AFL LLVM mode.
- All target programs were compiled with AddressSanitizer enabled.
- The modified AFL implementation used in this study is not included in this repository.
- Complete raw fuzzing logs are not included in this repository.
- Full experimental output directories are not included in this repository.
- This repository only provides benchmark information, instrumentation/build instructions, and initial seed corpora.

## Data Availability Statement

The benchmark information, AFL instrumentation and build instructions, and initial seed corpora used in this study are openly available in this repository.
