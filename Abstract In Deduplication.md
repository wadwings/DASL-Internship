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

### Indexing

__Problem__ Index increase exponentially, overflowing the RAM capacity, and finally result in frequent accesses to the low-speed disk for index lookups
`Approximate deduplication` trades a slightly reduced accuracy for a higher lookup performance and lower memory footprint

#### Four general approaches 

- Locality-based approaches
  __Locality__ refers to the observation that similar or identical files
  _improve deduplication indexing performance_
- Similarity-based approaches
  __Similarity__ refers to the similar characteristics of files
  _reduce the RAM overhead_
- Flash-assisted approaches
  use a random-access flash memory as an alternative to disks to provide high-throughput I/Os
  _incur additional hardware costs_
- Cluster approaches
  assign data streams by data routing scheme with load balance & intra-node duplicate elimination
  _avail it scalable for massive storage system_

### Compression

overall compression rate will be higher than compressing every chunk individually.

post-compress can reach a __higher redundancy elimination ratio__ than re-chunking.(compared `re-chunking` with `post-deduplication delta compression`)

- meanwhile, it introduces extra overheads.

Right Now, we focus on the __post-deduplication delta compression__, of the three time-consuming stages

#### Resemblance detection

accurately detect a fairly similar candidate for delta compression

__Manber__ proposes that the similarity between two files is proportional to the fraction of fingerprints.
__Broder__ proposes that `multi-sample` files into `super-feature`, and use it to index similar files.

#### Delta encoding the similar chunks

delta compression is the time-consuming process of calculating the differences.

__Xdelta__ use a byte-wise sliding window to identify the matched strings

#### Additional delta compression challenges

`reading base-chunks`,`data restore`&`garbage collection` are important factors of compression.

### Data restore

- the chunks physically scattered in containers, with the poor random I/O performance of HDDs, decrease restore performance
- fragmented chunks make it difficult to do the GC works

#### Primary storage

it makes the read-latency-lengthening fragmentation issue extremely important.

__iDedup__ selectively deduplicate sequential duplicate disk blocks to reduce fragmentation and amortize the read latency

#### Backup storage

the restore speed can drop by orders

__Nam et al.__ selectively eliminate sequential and duplicate chunks with a quantitative metric.
__RevDedup__ eliminates duplicates from the previous backups while eliminates duplicates from the new backups.
		 thus shifts the fragmentation to old backups

#### Cloud storage

__CABdedupe__ identifies unmodified data among chronological versions of backup datasets.
__ASR__ stores unique chunks with high reference counts on SSDs.
__NED__ groups chunks into segments and identifies the fragmented segments before uploading them to the cloud. 

### Garbage collection

