# Benchmark Programs and Initial Seed Corpora

This repository provides the benchmark information, AFL instrumentation/build notes, and initial seed corpora used in our fuzzing experiments.

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

Each `seeds/` directory contains the initial seed corpus used for the corresponding target program.

## Benchmark Programs

The benchmark programs used in this study include MAGMA benchmarks, FuzzBench benchmarks, and real-world programs.

### MAGMA Benchmarks

The MAGMA targets were prepared by applying the corresponding setup patches and vulnerability patches, and then compiled with AFL LLVM mode and AddressSanitizer.

Included programs:

- `libpng`
- `libsndfile`
- `lua`
- `php`
- `poppler`
- `sqlite3`

Initial seed directories:

```text
libpng/seeds/
libsndfile/seeds/
lua/seeds/
php/seeds/
poppler/seeds/
sqlite3/seeds/
```

### FuzzBench Benchmarks

The FuzzBench targets were compiled with AFL LLVM mode and AddressSanitizer.

Included programs:

- `bloaty`
- `re2`

Initial seed directories:

```text
bloaty/seeds/
re2/seeds/
```

### Real-world Programs

The real-world targets were selected from `binutils-2.40` and compiled with AFL LLVM mode and AddressSanitizer.

Included programs:

- `as-new`
- `cxxfilt`
- `nm-new`
- `objcopy`
- `objdump`
- `readelf`
- `size`
- `strip-new`

Initial seed directories:

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

## General AFL Build Environment

The following environment was used for AFL LLVM instrumentation and ASan compilation:

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

AFL LLVM mode was built before compiling the target programs:

```bash
make -C /path/to/AFL/llvm_mode
```

## MAGMA Build Notes

For MAGMA targets, the corresponding setup patches and vulnerability patches were applied before compilation.

General patching process:

```bash
cd /path/to/target/source

patch -p1 --fuzz=3 < /path/to/magma/targets/<target>/patches/setup/<setup_patch>

for p in /path/to/magma/targets/<target>/patches/bugs/*.patch; do
  patch -p1 < "$p"
done
```

After patching, the target programs were compiled with `afl-clang-fast` or `afl-clang-fast++` and ASan.

## FuzzBench Build Notes

For FuzzBench targets, the corresponding FuzzBench target source and harness were compiled with AFL LLVM mode and ASan.

The main AFL targets used in the experiments were:

```text
bloaty: fuzz_target_afl
re2: fuzzer_afl
```

## Real-world Build Notes

The real-world targets were built from `binutils-2.40`.

Build command:

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

## AFL Running Format

The general AFL running format was:

```bash
/path/to/AFL/afl-fuzz \
  -i <target>/seeds \
  -o output/<target> \
  -m none \
  -d \
  -- /path/to/target_binary @@
```

For example:

```bash
/path/to/AFL/afl-fuzz \
  -i readelf/seeds \
  -o output/readelf \
  -m none \
  -d \
  -- /path/to/binutils/readelf @@
```

## Data Availability Statement

The benchmark information, AFL instrumentation/build notes, and initial seed corpora used in this study are available in this repository.
