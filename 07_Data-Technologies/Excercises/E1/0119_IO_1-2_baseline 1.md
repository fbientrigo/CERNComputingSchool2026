# 1.2. I/O Boundness and Compression

## Exercise 1.2.1

**I/O or CPU bound?**

> [!todo] **Exercise 1.2.1**
> 
> The speed of an application can be limited by the processor speed (**CPU bound**) or it may be waiting for data transfers from disks, networks, or databases rather than actively using the CPU for calculations (**I/O bound**).
> 
> A good estimator to identify I/O or CPU bottlenecks is the ratio of **(cpu + system) / realtime**.
> 
> - If ratio **> 0.9**: CPU bound.
>     
> - If ratio **< 0.5**: I/O bound.
>     
> 
> **Preparation:** Create two 1 GB files with different contents:
> 
> 1. `dd if=/dev/zero of=/var/tmp/1G.null bs=1M count=1000`
>     
> 2. `dd if=/dev/urandom of=/var/tmp/1G.random bs=1M count=1000`
>     

**Q1** Using `md5sum`, `sha1sum`, and `sha256sum`, measure the time to compute hashes for both files. Are these applications CPU or I/O bound?

**Q2** Which component on your computer is the execution time defining bottleneck?

**Q3** What is the checksumming bandwidth of the three algorithms?

**Q4** Is it sensitive to the file contents?

**Q5** What is the effective blocksize used by these applications?

> [!success] **Solution**
> 
> **Q1** The computation is **CPU bound** (ratio > 0.9) when data is read from the buffer cache. This is verified by consistent execution times across runs.
> 
> **Q2** The performance is limited by the speed of the **CPU and Memory**.
> 
> **Q3** Bandwidth (Data / Realtime):
> 
> - **md5**: ~633 MB/s
>     
> - **sha1**: ~1280 MB/s
>     
> - **sha256**: ~1157 MB/s
>     
> 
> **Q4** No. Performance is identical for zero bytes or random bytes; hashing algorithms do not depend on data complexity.
> 
> **Q5** Using `strace -e read`, we see the blocksize is **32768 bytes (32kb)**.
> 
> Bash
> 
> ```
> strace -e read md5sum /var/tmp/1G.null
> # read(3, ...... , 32768) = 32768
> ```

---

## Exercise 1.2.2

**I/O Compression**

> [!todo] **Exercise 1.2.2**
> 
> **gzip** reduces storage volume but requires CPU time.
> 
> **Q1** Measure the time to compress the two files (1G or 100M) and compute compression speed (MB/s).
> 
> Bash
> 
> ```
> time gzip /var/tmp/100M.null
> time gzip /var/tmp/100M.random
> ```
> 
> **Q2** Look at the filesize after compression. What is the compression ratio?
> 
> **Q3** Measure the time to decompress and compute decompression bandwidth with respect to the **input stream**.
> 
> **Q4** Is the I/O pattern of `gzip`/`gunzip` independent of the file contents?

> [!success] **Solution**
> 
> **Q1** > * **1G.null**: ~159 MB/s (6.7s)
> 
> - **1G.random**: ~30 MB/s (35.8s)
>     
> 
> **Q2**
> 
> - **1G.null.gz**: ~1 MB (Ratio **1:947**). Gzip handles sequences of zeros extremely well.
>     
> - **1G.random.gz**: ~1 GB (Ratio **1:1.03**). Random data is essentially incompressible.
>     
> 
> **Q3** > * **null decompression**: Input bandwidth seems low (~15.9 KB/s) because the input file is tiny, but output bandwidth is high (1GB restored in 6.5s).
> 
> - **random decompression**: Input bandwidth ~138 MB/s.
>     
> 
> **Q4** **No, it depends on the compression ratio.**
> 
> Using `strace -c`:
> 
> - **Compression**: `read` calls are fixed by the input size/blocksize (32kb). `write` calls depend on the compressed size (few for null, many for random).
>     
> - **Decompression**: `read` calls depend on the compressed input size (few for null, many for random). `write` calls depend on the restored output size.
>     
