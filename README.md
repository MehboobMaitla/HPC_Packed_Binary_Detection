# Runtime Event Profiling vs Entropy for Packed Binary Detection

A small, reproducible prototype inspired by *Low-Entropy Packed Binary Detection via Accurate Hardware Events Profiling* (Leal, Ren, Li, Ming, HOST 2025).

## Question

Entropy is a common signal for detecting packed binaries, but it fails when a packed file is padded with low-entropy data. Does how a program behaves at run time stay informative when the file's entropy has been made to look normal? And does the answer hold against a stronger entropy baseline than whole-file entropy?

## What the notebook does

1. **Dataset.** About 20 benign Linux utilities. Each is packed with UPX, and each packed copy is padded with zeros to create a low-entropy packed variant. Every sample is executed once to confirm it still runs correctly.
2. **Entropy baselines.**
   * Whole-file Shannon entropy with a fixed threshold of 7.0.
   * Block-wise entropy: the file is split into 4 KiB blocks, and it is flagged if at least 5% of blocks exceed 7.0 bits. This is a fairer baseline, because padding dilutes the whole-file average but not the packed region itself.
3. **Runtime profiling.** Each sample is executed and profiled. The notebook tries, in order:
   * Valgrind cache and branch simulation (instruction counts, branch misprediction rate, cache miss rates)
   * `strace` memory events (counts of anonymous executable mappings, `mprotect` calls, `/proc/self/exe` reads, total system calls)
   * Basic resource usage as a last resort
4. **Runtime detector.** A random forest classifier on the profiling features, evaluated with grouped cross validation so that variants of the same program never appear in both training and test folds.
5. **Comparison.** Detection rate on UPX and low-entropy samples, false positive rate, overall accuracy, and profiling time per binary, for all three detectors side by side.
6. **Robustness checks.**
   * Runtime feature importance.
   * Whole-file entropy detection rate and false positives at thresholds 5, 6 and 7.
   * A list of the programs each detector misses on low-entropy samples.
7. **Plots.** Whole-file entropy view, block-wise entropy view and runtime event view, coloured by sample type.

## How to run

Open the notebook (`packed_binary_detection.ipynb`) in Google Colab and run all cells from the top. Setup installs `upx-ucl`, `valgrind` and `strace`. The cell that profiles the samples prints which method it used.

## Current status

* The whole-file entropy baseline behaves as expected: zero padding moves packed files from about 7.8 down to about 1.3, below the detection threshold.
* Runtime profiling currently runs through `strace` system call features. Packed and padded variants of the same program produce the same runtime profile, since padding does not change execution.
* The three-way comparison and robustness checks are implemented. Results from the latest run: `<fill in the comparison table from section 6 and the block-wise entropy result>`.
* The feature set is still being refined.

## Limitations

* The runtime signals are software visible events or simulated counters, not real hardware performance counters. Colab does not expose `perf` hardware events.
* Only UPX is used, with zero padding as the entropy evasion. Other packers and evasion techniques are not covered.
* The block size (4 KiB) and block-wise threshold (5% of blocks above 7.0 bits) were set by hand and not tuned.
* Samples are small, benign, well known system utilities. No malware is used.
* The dataset is small, so results are indicative only.

## Next steps

* Repeat the profiling with real hardware counters through `perf` on a bare metal Linux machine.
* Vary the padding size and add non-zero structured low-entropy padding (for example repeated text), which is harder for block-wise entropy to catch.
* Add more packers and more varied entropy evasion methods.
* Test stability of the signals across different CPU microarchitectures.
* Increase the number and variety of benign programs.

## Safety

All samples are benign system utilities. No malware is collected, executed or distributed by this project.
