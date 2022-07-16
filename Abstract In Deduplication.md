# Abstract In Deduplication

## Abstract 

### Main Focus

1. the background and key features of data deduplication
2. summary about the state-of-the-art research in this field
3. the usage of deduplication technology in industrial application
4. open problems and trends on deduplication based storage system

### Background

1. The "data deluge"
2. a huge part of data are redundant

### Effect with Deduplication

1. reduce storage space
2. minimize transmission especially in low-bandwidth system

### Method of Deduplication

- Split input data stream into chunks
- identify each chunk using fingerprint generate by secure hash method
  - it can be fixed-size or variable
- remove deduplicate data chunks

### The data compression

Traditionally, the common approaches to do compress is using the dictionary model.
it will need to compare hash-match string byte by byte
which is used in small region because the time and space complexity

### Deduplication in large-scale storage system

deduplication is more efficient and scalable.
it process data in chunk or file-level, rather the compression do it in byte-level.
	deduplication use fingerprint to compare instead of directly compare in bytes.

### Workflow in Data Deduplication

there are five key stages:

- chunking
- fingerprinting
- indexing of fingerprints
- further compression (optional)
- storage management

<img src="C:\Users\Wings\Downloads\Documents\photo_2022-07-14_17-02-00.jpg" alt="photo_2022-07-14_17-02-00" style="zoom:50%;" />

1. Files are divided into chunks, which are uniquely represented by fingerprint each.
2. Store the unique chunk, and record the list of constituent chunks in metadata.
   - chunks are saved in different containers, which would cause heavy I/O loads during the procedure of restoring files.

### Some others

- The data deduplication mainly focuses on eliminating identical data, rather than identify the records that represent the same entity which is used widely.

## data reduplication

### The fundamental limit

$$
H(X) = E(I(X)) = ^{X^n}_{i=1}{P(x_i)I(x_i)}=-^{X^n}_{i=1}P(x_i)\log_bP(x_i)
$$

### Former Approaches

Early data compression approaches use the Statistical model based coding, the entropy coding.

- like the Huffman coding

new approaches called dictionary model based coding.

Delta compression was to target the compression of similar files.

### Key features

- eliminate redundancy at the chunk level
  - file-level deduplication was proposed earlier
    - chunk-level usually has better compression performance than file-level
- the schemes of chunk-level duplicate elimination and SHA1-based fingerprinting
- chunking approaches include __fixed-size chunking__ and __variable-size chunking__
  - fixed-size chunking has problem with boundary-shift problem
- __hash-based fingerprinting__
  - __SHA1__  the `hash collision probability` is lower than the `hard disk error probability` in a EB-scale data storage system.
  - Still, _The hash-based comparison approach is not risk-free_
- `Content-defined chunking`  and `secure-hash based fingerprinting` are widely used in commercial storage system.

### Workflow

1. chunking 
2. fingerprinting
3. indexing
4. compression
5. storage
   1. data restore
   2. garbage collection
   3. security
   4. reliability

### Content-Defined Chunking & secure-hash based Fingerprinting

> CDC uses a sliding-window technique on the content of files and computes a hash value of the window.
>
> The chunk breakpoint is determined if the hash value of this sliding window satisfies some pre-defined condition.

`Rabin algorithm` is the most widely used in CDC for computing fingerprint.
$$
Rabin(B_1,B_2,...,B_\alpha)=\{\sum^a_{x=1}B_xp^{\alpha-x}\}\mod{D}
$$

#### Three deficiencies

- high chunk size variance
  CDC with extremely small or large size chunk cause `higher metadata overhead`
  __Limits on MAX/MIN chunk size__

  - TTTD

  - Regression chunking

  - MAXP

  - FingerDiff

- computational overhead
  the frequent computations of fingerprint on sliding window are time-consuming

  - SampleByte

  - LBFS

  - Gear

  - AE

- inaccuracy of duplicate detection 
  CDC cannot accurately find the boundary between the changed regions

  - Bimodal chunking

  - Subchunk

  - Frequency-Based Chunking

  - Metadata Harnessing Deduplication

Chunking of unchanged data can result in different chunks when metadata is interspersed.

#### Acceleration of Computational Tasks

- use multicore processors or GPGPU to provide more computing resources
- divided into independent subtasks to run them parallelly
