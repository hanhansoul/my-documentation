## A Comprehensive Study of the Past, Present, and Future of Data Deduplication

#### Introduction

1. 

#### REDUNDANT DATA REDUCTION AND DATA DEDUPLICATION

##### A Primer on Redundant Data Reduction

1. Huffman编码
2. LZ77 / LZ78
3. DEFLATE
4. Delta compression：Xdelta
   1. source code version control
   2. remote synchronization
   3. backup storage systems
5. Data deduplication



- byte level
- string level
- chunk level
- file level



##### Key Feature of Data Dedupliaction

1. LBFS：content-defined chunking
2. Venti：fixed-size chunking

---

1. Chunk-Level Duplication Identification
   1. fixed-size chunking：boundary-shift problem
   2. content-defined chunking（variable-size chunking）
2. Cryptographically Secure Hash-Based Fingerprinting
   1. SHA1 / SHA256

##### Basic Workflow of Data Deduplication

1. Chunking of data stream
2. Accleration of computation tasks
3. Indexing of fingerprints
4. Further compression
5. Data restore
6. Garbage collection
7. Security
8. Reliability



#### DATA DEDUPLICATION: AN IN-DEPTH EXAMINATION OF KEY TECHNOLOGIES

##### Chunking of data stream

1. Restrictions on max/min chunk size
   1. fixed-size chunking（FSC）
   2. content-defined chunking（CDC）
   3. Rabin algorithm
   4. Two-thresholds-two-divisors（TTTD）
   5. Regression chunking
   6. MAXP
   7. FingerDiff
2. Improving speed of CDC

   1. SampleByte
   2. LBFS
   3. Gear
   4. Asymmetric extremum
3. Further re-chunking the non-duplicate chunks

   1. Bimodal chunking
   2. Subchunk
   3. Frequency-based chunking（FBC）
   4. Metadata harnessing deduplication（MHD）




1. A low-bandwidth network file system
2. A framework for analyzing and improving content-based chunking algorithms
3. Primary data deduplication-large scale study and system design
4. Fingerdiff: Improved duplicate elimination in storage systems
5. Optimizing file replication over limited bandwidth networks using remote differential compression
6. EndRE: An end-system redundancy elimination service for enterprises
7. AE: An asymmetric extremum content defined chunking algorithm for fast and bandwidth-efficient data deduplication
8. Leap-based content defined chunking—Theory and implementation
9. Ddelta: A deduplicationinspired fast delta compression approach
10. Anchor-driven subchunk deduplication
11. Frequency based chunking for data de-duplication
12. Hysteresis re-chunking based metadata harnessing deduplication of disk images
13. Bimodal content defined chunking for backup streams

---

**Indexing of Fingerprints**

1. Locality-based
   1. DDFS：Bloom Filter + Locality preserved  caching
   2. Sparse Indexing
   3. MAD2
   4. SAM
   5. HPDS
   6. BLC
2. Similarity-based
   1. Extreme Binning
   2. Aronovich et al.
   3. SiLo
3. Flash-assisted
   1. DequeV1
   2. ChunkStash
   3. BloomStore
4. Cluster deduplication
   1. HYDRAstor
   2. DEBAR
   3. Dongle et al.
   4. Sigma-Dedupe
   5. Kaiser et al.
   6. Produck



**Post-Deduplication Compression**

1. Resemblance detection
   1. Super-feature
   2. REBL
   3. TAPER
   4. DE
   5. SIDC
   6. DARE
2. Delta encoding
   1. Xdelta
   2. Zdelta
   3. Ddelta



**Data Restore**

1. Primary storage systems
   1. IDedup
   2. POD
2. Backup storage systems
   1. CFL-SD
   2. CBR
   3. Capping
   4. RevDedup
   5. HAR
3. Cloud storage systems
   1. CARdedupe
   2. SAR
   3. NED



**Garbage Collection**

1. Convergent encryption and its variants

   1. Douglas et al.
   2. Farsite
   3. Pastiche
   4. MLE
   5. DupLESS
   6. Dekey
   7. SeeDep

2. Side channel attack

   1. Harnik et al.
   2. Gateway-based

3. Proofs of ownership (POW)

   1. PoWs

   2. s-POW

   3. Xu et al.

      

**Reliability**

1. Secondary storage
2. Primary storage
3. Cloud storage
4. Virtual machine
5. Network
6. SSD, multimedia, etc



#### RESOURCES AND PERSPECTIVES

#### SUMMARY



---

1. Alternatives for detecting redundancy in storage systems data
2. A low-bandwidth network file system
3. Bimodal content defined chunking for backup streams
4. P-dedupe: Exploiting parallelism in data deduplication system
5. Fingerprinting by random polynomials
6. A study on data deduplication in HPC storage systems
7. Some applications of Rabin’s fingerprinting method
8. Ddelta: A deduplicationinspired fast delta compression approach
9. QuickSync: Improving synchronization efficiency for mobile cloud storage services

---

