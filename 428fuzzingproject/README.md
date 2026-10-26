# Fuzzing cJSON with AFL++

CPT_S 428 project by Jose Hernandez, Reese Bewick, and Valeria Barrios.

We fuzz the cJSON parser (https://github.com/DaveGamble/cJSON) to look for
memory-safety bugs. The fuzzing harness (`fuzzing/afl.c`) and build scripts
(`fuzzing/afl.sh`, `fuzzing/afl-prepare-linux.sh`) come from the cJSON project.
Our team's additions are a custom seeds folder and specific results folders for future tests.

## Requirements

- Ubuntu 24.04.5
- AFL++ 4.09c
- CMake 3.28.3
- clang 18.1.3

Install on Ubuntu:

```bash
sudo apt install afl++ cmake clang
```

## Setup

```bash
git clone https://github.com/rbewick/cJSON.git
cd cJSON/fuzzing
./afl-prepare-linux.sh   # one-time system setting so AFL can detect crashes (needs sudo)
```

## Running the fuzzer

All commands are run from the root of the repo.

### 1. Build

```bash
cd fuzzing
./afl.sh        # builds afl-main; press Ctrl+C once the AFL screen appears
cd ..
```

### 2. Baseline run (cJSON's default seeds)

```bash
afl-fuzz -i fuzzing/inputs \
         -o team-fuzzing/results/baseline \
         -x fuzzing/json.dict \
         -V 2400 \
         -- fuzzing/afl-build/fuzzing/afl-main @@ yes
```

### 3. Custom seed run (Yet to be implemented)

```bash
afl-fuzz -i 428fuzzingproject/seeds \
         -o 428fuzzingproject/results/custom \
         -x fuzzing/json.dict \
         -V 2400 \
         -- fuzzing/afl-build/fuzzing/afl-main @@ yes
```

Both runs stop automatically after 40 minutes (`-V 2400`). They are identical
except for the seed folder, so their results can be compared directly.

**Warning:** `afl.sh` deletes the `afl-build/` folder each time it runs.
Copy your results out before rerunning.

## Inputs

- `fuzzing/inputs/`: the 14 seed files that come with cJSON
- `428fuzzingproject/seeds/`: our custom valid and invalid JSON inputs

## Seed file format

The AFL harness ( `fuzzing/afl.c` ) skips the first two characters of each file and uses them as option flags:

| Prefix | Meaning |
|---|---|
| `bf` | buffered printing, formatted output |
| `bu` | buffered printing, unformatted output |
| `uf` | unbuffered printing, formatted output |
| `uu` | unbuffered printing, unformatted output |

Example seed file:

```
bu{"name": "test", "values": [1, 2, 3]}
```

### What our custom seeds will test

- Deeply nested arrays and objects
- Very large and very small numbers
- Unicode escapes, including invalid ones
- Truncated and malformed documents

## Results

Each run saves its output to `428fuzzingproject/results/<run-name>/default/`:

- `fuzzer_stats`: summary numbers (executions, speed, coverage, crashes)
- `queue/`: inputs that reached new code
- `crashes/` and `hangs/`: inputs that crashed or hung the parser

To reproduce a crash:

```bash
fuzzing/afl-build/fuzzing/afl-main 428fuzzingproject/results/<run-name>/default/crashes/<file> yes
```

## Credentials

None required.
