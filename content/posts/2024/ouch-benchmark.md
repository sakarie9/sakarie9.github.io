---
title: Ouch Benchmark
date: 2024-02-04
summary: Simple benchmark of ouch
tags: [Linux]
category: 技术随笔
---

{{< alert "circle-info" >}}
[**ouch**](https://github.com/ouch-org/ouch) stands for Obvious Unified Compression Helper.

It's a CLI tool for compressing and decompressing for various formats.
{{< /alert >}}

## **Supported formats**

| Format | .tar | .zip | 7z | .gz | .xz, .lzma | .bz, .bz2 | .lz4 | .sz (Snappy) | .zst | .rar |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Supported | ✓ | ✓¹ | ✓¹ | ✓² | ✓ | ✓ | ✓ | ✓² | ✓ | ✓³ |

✓: Supports compression and decompression.

✓¹: Due to limitations of the compression format itself, (de)compression can't be done with streaming.

✓²: Supported, and compression runs in parallel.

✓³: Due to RAR's restrictive license, only decompression and listing can be supported.
If you wish to exclude non-free code from your build, you can disable RAR support
by building without the `unrar` feature.

## Benchmarks

### Compress

| compress | tar | tar.gz | tar.zst | tar.xz | tar.bz2 | tar.lz4 | tar.sz | 7z | zip |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| time | 0.331 | 3.535 | 1.397 | 5:22.61 | 58.079 | 0.430 | 0.304 | 5:27.61 | 29.441 |
| size | 1G | 761M | 754M | 768M | 758M | 1G | 1G | 768M | 761M |
| cpu | 99% | 1170% | 99% | 99% | 99% | 99% | 641% | 99% | 99% |

| compress | tar | tar.gz | tar.zst | tar.xz | tar.bz2 | 7z | zip |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ouch | 0.331 | 3.535 | 1.397 | 5:22.61 | 58.079 | 5:27.61 | 29.441 |
| tar | 0.458 | 29.950 | 1.158 | 5:40.00 | 1:02.82 |  |  |
| p7zip |  |  |  |  |  | 43.131 |  |
| zip |  |  |  |  |  |  | 29.455 |

### Decompress

| decompress | tar | tar.gz | tar.zst | tar.xz | tar.bz2 | tar.lz4 | tar.sz | 7z | zip |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| time | 0.529 | 3.212 | 2.703 | 34.474 | 35.898 | 0.769 | 0.690 | 41.182 | 3.166 |
| cpu | 99% | 99% | 99% | 99% | 99% | 99% | 99% | 99% | 99% |

| decompress | tar | tar.gz | tar.zst | tar.xz | tar.bz2 | tar.lz4 | tar.sz | 7z | zip |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ouch | 0.529 | 3.212 | 2.703 | 34.474 | 35.898 | 0.769 | 0.690 | 41.182 | 3.166 |
| tar | 0.572 | 4.815 | 1.014 | 33.701 | 35.457 | 0.604 | 0.534 |  |  |
| p7zip |  |  |  |  |  |  |  | 30.890 |  |
| unzip |  |  |  |  |  |  |  |  | 4.885 |
