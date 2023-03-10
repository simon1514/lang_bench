This project contains simple benchmarks for different languages, and an aggregator
of results to easily compare the languages.

# Presentation of the benchmarks.

json-ser: Json serialization

json-deser: Json deserialization

itoa: Converting integers to strings

nonVectoLoop: A loop that cannot get Loop Unrolling, or use simd instructions.

branchingNonVectoLoop: Same as nonVectoLoop but with branching.

trivialAutoVectoLoop: Adding an array of numbers.

complexAutoVectoLoop: Similar to trivialAutoVectoLoop but with more arithmetic ops.

branchingAutoVectoLoop: Similar to trivialAutoVectoLoop but with branching.

# Usage

From benchrunner/Dockerfile

```
sudo docker build  -t bench1 lang_bench/benchrunner ; sudo docker run -e LANG_BENCH_EFFORT=4 -it --rm bench1
```

From an IDE by running this file

```
benchrunner/App.kt
```

From gradlew

```bash
cd language_benchmarks/benchrunner
LANG_BENCH_EFFORT=4 ./gradlew run
```

# Env

The LANG_BENCH_EFFORT environment variable can be used to configure how long we are ready to wait for the result.
More effort means more reliable results.
The default is LANG_BENCH_EFFORT=1, which is not enough for quality results.

We assume that java, cargo and go are installed in /usr/bin.
Cargo can also be in ~/.cargo/bin/cargo.
For dev purposes, that location can be changed in

```
benchrunner/app/src/main/kotlin/benchrunner/App.kt
```

# Methodology

There are many ways we can build a benchmark.
Here are a few things that describe the benchmarks we have here (TLDR, they are DIY microbenchmarks)

- Unlike some other benchmarks, we dont measure the process execution time. Instead, what is
  measured is a loop running a small task many times.
- Each task is run many times before we start measuring it. We call that the warmup.
- We try to target only the cpu efficiency. Each task uses little memory, and no IO.
- These tasks cannot be optimized away by compilers since 1- they operate on nondeterministic data
  2- their result is eventually printed in stdout 3- each task iteration work on different data.
- All tasks compute a nonsensical integer that means nothing.
- These nonsensical integer results must be the same for the different implementation of a task, or
  the benchmark fails.
- The GC is run before starting the measurements in kotlin and go.

# Sample Result

```
Benchmark Summary (Smaller percentage is better) (Ignored 2 warmups) (effort=200) 
  json-ser                         scale:   772ms
    rust-serde                       mean: 16%, meanDiff:  0%
    kotlin-jvm-jsoniter              mean: 91%, meanDiff:  4%
    go-jsoniter                      mean:100%, meanDiff:  3%
  json-deser                       scale:  1628ms
    rust-serde                       mean: 44%, meanDiff:  1%
    kotlin-jvm-jsoniter              mean: 49%, meanDiff:  2%
    go-jsoniter                      mean:100%, meanDiff:  2%
  itoa                             scale:  5438ms
    rust                             mean: 30%, meanDiff:  1%
    kotlin-jvm-heapless              mean: 45%, meanDiff:  1%
    kotlin-jvm                       mean: 51%, meanDiff:  2%
    go                               mean:100%, meanDiff:  2%
  nonVectoLoop                     scale:   694ms
    kotlin-jvm                       mean: 87%, meanDiff:  3%
    go                               mean: 94%, meanDiff:  1%
    rust                             mean:100%, meanDiff:  2%
  branchingNonVectoLoop            scale:  1031ms
    rust                             mean: 58%, meanDiff:  0%
    kotlin-jvm                       mean: 76%, meanDiff: 13%
    go                               mean:100%, meanDiff:  1%
  complexAutoVectoLoop             scale:   384ms
    rust                             mean: 67%, meanDiff:  2%
    kotlin-jvm                       mean: 92%, meanDiff:  5%
    go                               mean:100%, meanDiff:  2%
  trivialAutoVectoLoop             scale:  7585ms
    rust                             mean:  9%, meanDiff:  0%
    kotlin-jvm                       mean: 35%, meanDiff:  1%
    go                               mean:100%, meanDiff:  1%
  branchingAutoVectoLoop           scale:  7038ms
    rust                             mean: 11%, meanDiff:  0%
    go                               mean: 83%, meanDiff:  2%
    kotlin-jvm                       mean:100%, meanDiff:  1%

```