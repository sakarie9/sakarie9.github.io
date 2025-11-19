---
date: 2025-08-21T12:17:48+08:00
title: "Parsing PFS Files Used by the Artemis Engine"
tags:
  - Games
categories: 技术分享
summary: A comprehensive analysis of the PFS file format structure used by the Artemis game engine, including detailed breakdowns of headers, file entries, and offset calculations.
---

## Introduction

**PFS** is a proprietary archive format utilized by the [Artemis](https://www.ies-net.com/) visual novel engine. It functions as a container for game assets, enabling efficient packaging and distribution of multiple files within a single archive.

This document presents a comprehensive technical specification of the PFS file format, derived from reverse-engineering analysis. It aims to assist developers in building tools for extracting, inspecting, and modifying PFS archives.

### Related Tools

I have developed the following tools to facilitate working with PFS files:

- **[pf8](https://crates.io/crates/pf8)**: A Rust library offering robust encoding and decoding capabilities for PFS files.
- **[pfs-rs](https://github.com/sakarie9/pfs-rs)**: A command-line utility for packing and unpacking PFS archives.

{{< github repo="sakarie9/pfs-rs" showThumbnail=false >}}

## PFS File Format Structure

The PFS format employs a straightforward structure to organize file metadata and content. It begins with a **Header** section that indexes all contained files, followed immediately by the **File Data** section. This layout supports efficient random access to any file within the archive.

{{< alert >}}
**Note:** All multi-byte integer values in PFS files are stored in **little-endian** byte order.
{{< /alert >}}

## Header Structure

The header contains global archive information and the index of all files.

To represent the variable-length sections, we define the following symbols:

- **`N`**: The number of files (value of _File Count_).
- **`S_Entries`**: The total size in bytes of the _File Entries_ array.
- **`S_Offsets`**: The total size in bytes of the _File Size Offsets_ array (equal to `8 * N`).

| Offset                         | Size        | Field Name            | Description                                              |
| :----------------------------- | :---------- | :-------------------- | :------------------------------------------------------- |
| `0x00`                         | 3           | **Magic Number**      | File signature: `pf8` (or `pf6`).                        |
| `0x03`                         | 4           | **Index Size**        | Size of the index data (from `0x07` to end of header).   |
| `0x07`                         | 4           | **File Count** (`N`)  | Total number of files in the archive.                    |
| `0x0B`                         | `S_Entries` | **File Entries**      | Array of [File Entry](#file-entry-structure) structures. |
| `0x0B + S_Entries`             | 4           | **File Size Count**   | Number of size offsets (typically `N + 1`).              |
| `0x0F + S_Entries`             | `S_Offsets` | **File Size Offsets** | Array of 8-byte offsets pointing to file sizes.          |
| `0x0F + S_Entries + S_Offsets` | 8           | **Padding**           | Zero-filled padding bytes.                               |
| `0x17 + S_Entries + S_Offsets` | 4           | **Index End Offset**  | Pointer to _File Size Count_ (relative to `0x07`).       |

{{< mermaid >}}

%%{init: {'flowchart': {'nodeSpacing': 15, 'rankSpacing': 15}, 'themeVariables': {'fontSize': '12px'}}}%%
graph TD
Magic[0x00: Magic Number] --> IndexSize[0x03: Index Size]
IndexSize --> FileCount[0x07: File Count]
FileCount --> FileEntries[0x0B: File Entries]
FileEntries -- Size: S_Entries --> FileSizeCount[File Size Count]
FileSizeCount --> FileSizeOffsets[File Size Offsets]
FileSizeOffsets -- Size: S_Offsets --> Padding[Padding]
Padding --> EndOffset[Index End Offset]

    style FileEntries fill:#d9d,stroke:#333,stroke-width:2px
    style FileSizeOffsets fill:#0bf,stroke:#333,stroke-width:2px

{{< /mermaid >}}

## File Entry Structure

Each file in the archive is described by a variable-length entry in the _File Entries_ section.

> **Note:** The offsets listed below are relative to the start of the specific file entry.

| Relative Offset | Size | Field Name      | Description                                                                                                                               |
| :-------------- | :--- | :-------------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00`          | 4    | **Name Length** | The length of the filename in bytes.                                                                                                      |
| `0x04`          | _N_  | **File Name**   | The filename string. Length is defined by _Name Length_.                                                                                  |
| `0x04 + N`      | 4    | **Separator**   | A separator field, typically filled with null bytes (`0x00`).                                                                             |
| `0x08 + N`      | 4    | **Data Offset** | The absolute offset of the file's data from the beginning of the PFS file. See [Understanding Data Offsets](#understanding-data-offsets). |
| `0x0C + N`      | 4    | **File Size**   | The size of the file's data in bytes.                                                                                                     |

### Understanding Data Offsets

The **Data Offset** field specifies the absolute position of the file's content within the archive.

**Example:**
If a file entry has a **Data Offset** of `0xBD` and a **File Size** of `0x100`:

- The file's data begins at byte `0xBD` of the PFS file.
- The data ends at byte `0x1BD` (`0xBD + 0x100`).
- In Rust slice notation: `archive_data[0xBD .. 0x1BD]`.

## File Size Offset Structure

{{< mermaid >}}
%%{init: { "packet": { "bitsPerRow": 8 } } }%%
packet
+8: "File Size Offset"
+8: "File Size Offset"
+8: "..."
{{< /mermaid >}}

The **File Size Offsets** section provides a secondary way to navigate the file entries, specifically pointing to the size fields.

- Each entry is an **8-byte** integer.
- The value is an offset relative to the start of the _File Size Offsets_ array itself.
- It points to the `File Size` field of a specific File Entry.

**Example:**
If a File Size Offset entry contains `0x25`:

1. Locate the start of the _File Size Offsets_ array (let's say it's at absolute offset `0xF` for simplicity, though it varies).
2. Add `0x25` to that start position.
3. The resulting address points to the 4-byte `File Size` value of a file entry.

## Encryption

PFS archives (specifically the `pf8` variant) use a custom encryption scheme to protect file data. The encryption uses a stream cipher based on XOR operations with a dynamically generated key.

### Key Generation

The encryption key is derived from the archive's header index data using the SHA-1 hashing algorithm.

**Key Generation Process:**

1. **Source Data**: Extract the index data from the archive header

   - **Start Offset**: `0x07` (immediately after the magic number and index size fields)
   - **Length**: The value specified in the **Index Size** field (at offset `0x03`)
   - This includes the file count, all file entries, file size offsets, padding, and index end offset

2. **Hashing**: Apply SHA-1 hash to the index data

   - **Algorithm**: SHA-1
   - **Output**: 20-byte (160-bit) hash digest

3. **Result**: The 20-byte SHA-1 hash serves as the encryption key

```rust
// Key generation example
let index_size = read_u32_le(&archive_data[0x03..0x07]);
let index_data = &archive_data[0x07 .. 0x07 + index_size as usize];
let key = sha1(index_data); // 20 bytes
```

### Encryption Algorithm

The encryption uses a simple but effective XOR stream cipher with the generated key.

**Algorithm Characteristics:**

- **Method**: XOR (Exclusive OR) cipher
- **Key Stream**: The 20-byte key is repeated cyclically to match the data length
- **Scope**: Only file data is encrypted; headers and index structures remain in plaintext
- **Per-File Encryption**: Each file is encrypted independently, with the key stream starting from the beginning for each file

**Encryption Formula:**

{{< katex >}}

$$C_i = P_i \oplus K_{i \bmod L}$$

Where:

- \(C_i\) is the \(i\)-th byte of ciphertext (encrypted data)
- \(P_i\) is the \(i\)-th byte of plaintext (original data)
- \(K\) is the 20-byte encryption key
- \(L = 20\) (key length in bytes)
- \(i\) is the byte position **within the current file** (starting from 0 for each file)

**Encryption Example:**

```rust
// Each file is encrypted independently
for file in files {
    let mut file_data = read_file_data(file);

    // XOR with key starting from offset 0 for this file
    for (i, byte) in file_data.iter_mut().enumerate() {
        *byte ^= key[i % key.len()];
    }

    write_encrypted_data(file_data);
}
```

For large files that are processed in chunks, the offset is tracked within that file only:

```rust
let mut offset_in_file = 0;

while let Some(chunk) = read_chunk() {
    for (i, byte) in chunk.iter_mut().enumerate() {
        *byte ^= key[(offset_in_file + i) % key.len()];
    }
    offset_in_file += chunk.len();
}
```

### Decryption

Decryption is identical to encryption due to the XOR cipher's symmetric nature:

{{< katex >}}

$$P_i = C_i \oplus K_{i \bmod L}$$

Each file is decrypted independently using the same key, with the key stream restarting from position 0 for each file.
