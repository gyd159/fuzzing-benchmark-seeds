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

## MAGMA Benchmarks

The MAGMA targets were prepared by applying the corresponding setup patches and vulnerability patches, and then compiled with AFL LLVM mode and AddressSanitizer.

Included programs:

- `libpng`: MAGMA libpng target with PNG vulnerability patches.
- `libsndfile`: MAGMA libsndfile target with SND vulnerability patches.
- `lua`: MAGMA Lua target with LUA vulnerability patches.
- `php`: MAGMA PHP target with PHP vulnerability patches.
- `poppler`: MAGMA poppler target with PDF vulnerability patches.
- `sqlite3`: MAGMA SQLite target with SQL vulnerability patches.

### MAGMA Build Process

The general patching process is shown below:

```bash
cd /path/to/target/source

patch -p1 --fuzz=3 < /path/to/magma/targets/<target>/patches/setup/<setup_patch>

for p in /path/to/magma/targets/<target>/patches/bugs/*.patch; do
  patch -p1 < "$p"
done
```

After applying the patches, the target programs were compiled using AFL LLVM mode and AddressSanitizer.

The initial seed directories are:

```text
libpng/seeds/
libsndfile/seeds/
lua/seeds/
php/seeds/
poppler/seeds/
sqlite3/seeds/
```

## FuzzBench Benchmarks

The FuzzBench targets were compiled using AFL LLVM mode and AddressSanitizer.

Included programs:

- `bloaty`: FuzzBench bloaty target.
- `re2`: FuzzBench RE2 target.

The initial seed directories are:

```text
bloaty/seeds/
re2/seeds/
```

## Real-world Programs

The real-world targets were selected from `binutils-2.40` and compiled using AFL LLVM mode and AddressSanitizer.

Included programs:

- `as-new`
- `cxxfilt`
- `nm-new`
- `objcopy`
- `objdump`
- `readelf`
- `size`
- `strip-new`

The initial seed directories are:

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

## General Build Environment

All target programs were instrumented using AFL LLVM mode and compiled with AddressSanitizer.

The general compilation environment is:

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

Before compiling the target programs, AFL LLVM mode should be built:

```bash
make -C /path/to/AFL/llvm_mode
```

## AFL Execution Example

The general AFL execution command is:

```bash
/path/to/AFL/afl-fuzz \
  -i <target>/seeds \
  -o output/<target> \
  -m none \
  -d \
  -- /path/to/target_binary @@
```

Example:

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
