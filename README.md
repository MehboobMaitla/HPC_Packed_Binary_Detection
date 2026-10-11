# Runtime Event Profiling vs Entropy for Packed Binary Detection

A small, reproducible prototype inspired by *Low-Entropy Packed Binary Detection via Accurate Hardware Events Profiling* (Leal, Ren, Li, Ming, HOST 2025).

## Question

Entropy is a common signal for detecting packed binaries, but it fails when a packed file is padded with low-entropy data. Does how a program behaves at run time stay informative when the file's entropy has been made to look normal?

## What the notebook does

1. **Dataset.** About 20 benign Linux utilities. Each is packed with UPX, and each packed copy is padded with zeros to create a low-entropy packed variant. Every sample is executed once to confirm it still runs correctly.
2. **Entropy baseline.** Whole-file Shannon entropy with a fixed threshold of 7.0.
3. **Runtime profiling.** Each sample is executed and profiled. The notebook tries, in order:
   - Valgrind cache and branch simulation (instruction counts, branch misprediction rate, cache miss rates)
   - `strace` memory events (counts of anonymous executable mappings, `mprotect` calls, `/proc/self/exe` reads, total system calls)
   - Basic resource usage as a last resort
4. **Runtime detector.** A random forest classifier on the profiling features, evaluated with grouped cross validation so that variants of the same program never appear in both training and test folds.
5. **Comparison.** Detection rate on UPX and low-entropy samples, false positive rate, overall accuracy, and profiling time per binary.
6. **Plots.** Entropy view and runtime event view, coloured by sample type.

## How to run

Open the notebook in Google Colab and run all cells from the top. Setup installs `upx-ucl`, `valgrind` and `strace`. The cell that profiles the samples prints which method it used.

## Current status

- The entropy baseline behaves as expected: zero padding moves packed files from about 7.8 down to about 1.3, below the detection threshold.
- Runtime profiling currently runs through `strace` system call features. Packed and padded variants of the same program produce the same runtime profile, since padding does not change execution.
- Quantitative comparison of the two detectors is in progress, and the feature set is still being refined.

## Limitations

- The runtime signals are software visible events or simulated counters, not real hardware performance counters. Colab does not expose `perf` hardware events.
- Only UPX is used, with zero padding as the entropy evasion. Other packers and evasion techniques are not covered.
- Samples are small, benign, well known system utilities. No malware is used.
- The dataset is small, so results are indicative only.

## Next steps

- Repeat the profiling with real hardware counters through `perf` on a bare metal Linux machine.
- Add more packers and more varied entropy evasion methods.
- Test stability of the signals across different CPU microarchitectures.
- Increase the number and variety of benign programs.

## Safety

All samples are benign system utilities. No malware is collected, executed or distributed by this project.
