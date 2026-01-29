# 1.1. Input/Output (I/O) baseline

## Exercise 1.1.1

**Measure of the Input/Output Operation per Second (IOPS)**

> [!todo] **Exercise 1.1.1**
> 
> Using the `dd` command, measure the upper limit of Input/Output Operation per Second (IOPS) on you exercise machine.
> 
> You have to use the smallest possible blocksize issuing 1 million I/O operations reading from **/dev/zero** and writing to **/dev/null**. To do this, use the following command:
> 
> `dd if=/dev/zero of=/dev/null bs=1 count=1000000`

**Explanation of the dd parameters:**

- **dd if=** `input-file`
- **of=** `output-file`
- **bs=** `block-size`
- **count=** `number of blocks`

Explain why this command measures the IOPS limit of your computer.
Try also the following command:

Bash

```
dd if=/dev/zero of=/dev/null bs=1000 count=1000
```

This commands still writes 1 million bytes but in blocks of 1000 bytes. Explain why it is faster.

What is the I/O pattern of the **dd** command?

Run the same command prefixed with **strace** to see the all the I/O operations done by the **dd** command:

Bash

```
strace dd if=/dev/zero of=/dev/null bs=1 count=1000000
strace dd if=/dev/zero of=/dev/null bs=1000 count=1000
```

> [!success] **Solution**
> 
> Bash
> 
> ```
> dd if=/dev/zero of=/dev/null bs=1 count=1M
> ```
> 
> This command reads a single byte (the smallest unit for the I/O system) from a virtual zero `(/dev/zero)` device and writes it to a virtual null `(/dev/null)` device. Both devices can be considered as latency-free and infinite bandwidth because they don't do any I/O operation. If we use a blocksize of 1 we measure the maximum I/O transaction rate we can execute in our environment.
> 
> Bash
> 
> ```
> strace dd if=/dev/zero of=/dev/null bs=1 count=100
> ...
> read(0, "\0", 1)                        = 1
> write(1, "\0", 1)                       = 1
> read(0, "\0", 1)                        = 1
> write(1, "\0", 1)                       = 1
> etc ...
> ```
> 
> One can see that **dd** opens both devices and then reads one byte from **/dev/zero** and writes one byte to **/dev/null** as expected!
> 
> If you want to calculate the number of IOPS of your computer, you need to measure the time it takes to perform the 1 million operations. Run the same command prefixed with **time** to display how much time it has taken to complete the **dd** command:
> 
> Bash
> 
> ```
> time dd if=/dev/zero of=/dev/null bs=1 count=1M
> ...	
> real    0m1.013s
> user    0m0.156s
> sys     0m0.855s
> ```
> 
> In this example, you can see that the computer can do more than 1 million I/O operations per second.

---

## Exercise 1.1.2

**Measure the memory bandwidth**

> [!todo] **Exercise 1.1.2**
> 
> Measure the memory speed using the dd command with a **1 million bytes blocksize** from /dev/zero to /dev/null. Explain the observed result.
> 
> `dd if=/dev/zero of=/dev/null bs=1M count=10000`
> 
> How would you measure the speed of your computer RAM memory ?

> [!success] **Solution**
> 
> Bash
> 
> ```
> dd if=/dev/zero of=/dev/null bs=1M count=100000
> 100000+0 records in
> 100000+0 records out
> 104857600000 bytes (105 GB, 98 GiB) copied, 2.04313 s, 51.3 GB/s
> ```
> 
> This command is limited by memory bandwidth because it **read** 1M bytes into memory and write them back to a null device that has no latency. It is essentially a memory copy operation.
> 
> The memory speed is directly indicated by the **dd** command, but you can also prefix the command with the time command and do the calculations yourself!

---

## Exercise 1.1.3

**Measure disk performance and disk caching**

> [!todo] **Exercise 1.1.3**
> 
> **Q1** Measure the **disk write** performance using the **dd** command and a 1M blocksize writing a 1G file from **/dev/zero** to **/var/tmp/$USER.1G**.
> 
> **Q2** Explain the observed speed. Is it consistent with the speed of a harddisk or SSD? Which cache strategy is implemented by the operating system?
> 
> **Q3** Add **oflag=direct** to the **dd** command line and compare the result.
> 
> **Q4** Measure the maximum **disk read** performance using the **dd** command and a 1M blocksize reading a 1G file from **/var/tmp/$USER.1G** to **/dev/null**.
> 
> **Q5** Repeat the measurement adding **iflag=direct** to the **dd** command line and compare the results. Explain your observations

> [!tip] **Hint**
> 
> Bash
> 
> ```
> dd if=/dev/zero of=/var/tmp/$USER.1G bs=1M count=1K
> dd if=/dev/zero of=/var/tmp/$USER.1G bs=1M count=1K oflag=direct
> dd if=/var/tmp/$USER.1G of=/dev/null bs=1M count=1K
> dd if=/var/tmp/$USER.1G of=/dev/null bs=1M count=1K iflag=direct
> ```

> [!success] **Solution**
> 
> **Q1** The write performance is higher than the speed of the underlying disk.
> 
> **Q2** The operating system uses a write-back cache strategy, which boosts the observed bandwidth when writing until the memory buffer of the cache is full. Essentially you are writing into memory and not to the storage device. The write back to disk is asychronous and not visible to the application unless the cache is full.
> 
> **Q3** When **oflag=direct** is used the operating system write-back cache is bypassed and you see the real performance of the storage device.
> 
> **Q4** When reading the file, the data is read from the memory cache because the file was cached while writing it in the first run. This is why the read speed is much higher than the speed of the underlying disk.
> 
> **Q5** When we force direct I/O, we force to bypass the buffer cache and we see the real performance of the storage device.

---

## Exercise 1.1.4

**I/O Debugging**

> [!todo] **Exercise 1.1.4**
> 
> Inspect the I/O pattern of the command `yes` using the strace command. You have to use Control-C to terminate the command. You can redirect the output STDOUT to **/dev/null**. Which operations and blocksizes do you see?

> [!tip] **Hint**
> 
> Bash
> 
> ```
> strace yes > /dev/null
> ```

> [!success] **Solution**
> 
> Bash
> 
> ```
> strace yes > /dev/null
> ```
> 
> The command writes via 4k or 8k buffers (depending on OS) the character 'y' and a line feed in a loop if redirected.

---

## Exercise 1.1.5

**I/O Debugging**

> [!todo] **Exercise 1.1.5**
> 
> Measure the syscall tracing overhead using the strace command with the count option `-c` on the baseline measurement from [[1.1.1]] with 1 byte blocksize and 1M transactions. Compare the overhead when using large blocksizes like 1M in 10 transactions. Explain your observation

> [!tip] **Hint**
> 
> Bash
> 
> ```
> dd if=/dev/zero of=/dev/null bs=1 count=1000000
> strace -c dd if=/dev/zero of=/dev/null bs=1 count=1000000
> dd if=/dev/zero of=/dev/null bs=1M count=10
> strace -c dd if=/dev/zero of=/dev/null bs=1M count=10
> ```

> [!success] **Solution**
> 
> Bash
> 
> ```
> dd if=/dev/zero of=/dev/null bs=1 count=1000000
> strace -c dd if=/dev/zero of=/dev/null bs=1 count=1000000
> dd if=/dev/zero of=/dev/null bs=1M count=10
> strace -c dd if=/dev/zero of=/dev/null bs=1M count=10
> ```
> 
> The overhead per syscall is significant and changes the measured performance result by an order of magnitude. The effect is even more visible when you remove the `-c` option. One has to be careful with performance evaluations while using the strace command for such I/O cases. For applications with large blocksizes or for CPU bound applications the resulting overhead can often be neglegible, because the number of syscalls is low and the overhead is proportional to the number of syscall operations.