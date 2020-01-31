## Big Data Analytics using Spark

### Memory Latency
Computers consist of CPU and storage at the high level.

Step Latency: time it takes to complete one step.

Total Latency: time it takes to complete all the steps

Storage Types:
- Main Memory (RAM) is fast
- Spinning disk is slow

The major source of latency in big data analysis is reading and writing to storage.

Different types of storage offer different latency, capacity and price.

Big data analytics revolves around methods for organizing storage and computation in ways that maximize speed while minimizing cost.

> <strong>Q: Which of the following steps has the lowest step latency?</strong>
- Reading from memory
- Performing a CPU operation
- Writing to memory

> A: Performing a computational operation. (Note that Reading from memory and Writing to memory are the two steps that takes up the most memory for Big Data Analytics).

> <strong>Q: True or false: Main memory has a higher latency than a mechanical hard drive.</strong>

> A: False

> <strong>Q: Where is the result stored immediately following a multiplication operation?</strong>

> A: Registers

----

### Cache and Memory Hierarchy
#### Why Memory locality reduces latency?
<strong>Cache</strong>:is a hardware or software component that stores data so that future requests for that data can be served faster; the data stored in a cache might be the result of an earlier computation or a copy of data stored elsewhere. Cache is a static memory that is more expensive but it is much faster.

<strong>Cache hit</strong>: When CPU is able to find the memory location from the cache.
> Cache is effective if most accesses are hits, i.e. Cache hit rate is high.

<strong>Cache miss</strong>: When CPU is unable to find the memory location from the cache.

<strong>Temporal Locality</strong>: Multiple accesses to same address within a short time period.

<Strong>Spatial Locality</strong>: Multiple accesses to close-together addresses in short time period.
- Examples: calculating difference between two sums.
- Counting words by sorting.
- Benefits of Spatial Locality: memory is partitioned into blocks/lines rather than single bytes. So moving a block of memory takes much less time than moving each byte individually. Memory locations that are close to each other are highly likely to be in the same block. This leads to more cache hits.

In general, caching reduces storage latency by bringing relevant data close to the CPU. This requires that code exhibits access locality, i.e. temporal locality and spatial locality.

> <strong>Q: Suppose you have a 16KiB cache, with a 64 byte block size. You try to access a 4 byte integer and receive a cache miss. How much data will be copied to the cache from main memory?</strong>

> A: The amount of data copied to the cache will be the size of the variable being accessed. In this case, 4 bytes.

> <strong>Q:How does sorting data improve access latency?</strong>

> A: Spacial locality is improved because values are placed in consecutive memory locations during sorting and multiple values which will be accessed consecutively will be copied to the cache.

----

### Locality and Storage Access
#### Why is access locality important?
- <strong>Access Locality</strong> is the ability of a software to make good use of cache.
- Memory is broken up into pages.
- Software that uses the same or neighboring pages repeatedly has good access locality.
- Hardware is designed such that it would speed up such software.

<strong>Temporal Locality</strong>: repeated access to the same memory location.
- Example is a function with a parameter that computes long sequence of elements. We can think of the parameter as a parameter vector, i.e. the weights in a neural network. As we need each of the parameters in each computation, having the parameter in the cache will lead to a faster computation. If otherwise, each calculation will cause at least two cache misses and ultimately your program will be much slower.

<strong>Spatial Locality</strong>: Multiple access to close-together addresses
- Example is a function where you sum the squares of the difference between two consecutive sequence of numbers. In order to perform this calculation, there are two different approaches:
  1. Linked list: poor locality, i.e. given 6 numbers, 1,2,3,...,6, we will need 4 pages of memories where the numbers are distributed. In the end, you will need 4 pages to touch all 6 elements (1->2->...->6).
  2. Indexed array: good locality, i.e. given 6 numbers, 1,2,3,...,6, we will need only 2 pages since you will align the numbers consecutively (array of numbers with indexes).

In general, improved memory locality improves run-time. This is because memory is organized in pages.

> <strong>Q: What is the primary reason that a linked-list will cause more cache misses than an array?</strong>

> A: Linked-list elements are not stored in consecutive memory locations.
> Linked-list elements are not stored in consecutive memory locations (Remember memory is organized in pages).

> <strong>Q: Given the following code, determine which locality is present</strong>
```python
A = [0] * 10000
sum = 0
for i in range(1,10000):
  sum += A[i] + i
```
> A: Spacial Locality

> <strong>Q: Lack of temporal and spatial locality will cause cache misses (T/F).</strong>

> A: True

---

### Row-wise vs. Column-wise Scanning
#### Memory Locality, Rows vs. Columns
<strong>1.1 The effect of row vs column major layout</strong>:

Numpy arrays are by default organized in row-major order.
- a[i,j] and a[i,j+1] are placed in consecutive places in memory.
- a[i,j] and a[i+1,j] are 10 memory locations apart.

This means that scanning through an array row by row is more local (faster/efficient) than scanning column by column. Usually column by column operations takes twice longer than row by row operations.

<strong>1.2 Some experiments with row vs column scanning</strong>:

We want to see how the run time of two code snippets varies as n, size of array is changed. As the size of the matrix increases, i.e. from 1 x 1 to 1,000 x 1,000, we see that the ratio of run times of column by column over row by by increases exponentially. It is also important to note that the trend is very noisy. This is because individual runs differ from one to the other since the state of cache is varying very rapidly.

<strong>1.3 Conclusions</strong>:
- Traversing a numpy array column by column takes more time than row by row.
- The effect increases proportionally to the number of elements in the array (square of the number of rows or columns).
- The effect is highly variable between runs.

----

### Measuring Memory Latency
<strong>L1 Cache</strong> is "level-1" cache memory, usually built onto the microprocessor chip itself.

<strong>L2 Cache memory</strong> is on a separate chip (possibly on an expansion card) that can be accessed more quickly than the larger "main" memory.

<strong>Memory Latency</strong> is the time between when CPU issues the command to the time when command is complete. This time could be short if the element is already in the L1 Cache and very long if the element is in external memory (disk or SSD).
- Note CPU runs at about 10^-9.
- System time is accurate to micro-second, so 0KB is not actually zero.
- Avg SSD random access latency is 10^-5 - 10^-4.
- Avg memory access latency is 10^-7 - 10^-5.

> The bigger the array, farther you are from CPU, i.e. 10GB has to be stored in the hard drive

> <strong>SSD vs. HDD:</strong> SSD's are faster when it comes to raw speed. In terms of sequential speed, unless we are copying large files back and forth, the comparison is meaningless. SSD's are all about little data transactions that happen all the time, i.e. messenger, operating system, transactions where you do not have to physically move ahead across a disk SSD outperforms HDD. So in system responsiveness SSD is the fastest. In terms of playing music files, archive of pictures, HDD's are faster. Note as of 2014, with $160, you can buy 256GB SSD while you can by 4TB HDD. In terms of reliability, HDD's wear out faster as opposed to SDD's but if you write to a consumer grade model often, it will wear out quickly.


<strong>Sequential Access</strong>
- Random access degrades rapidly with the size of the blocks.
- Sequential access is much faster.
- Note 10GB to disk sequentially takes 8.9 seconds or less than 1 second for a gigabyte.
- Writing 1TB disk at this rate takes ~1000 seconds or about 16 minutes.
- When you want to write big blocks, SSD (Solid State Disk) is much faster than the main memory.
  - Why?
    - Byte rate for writing large blocks for memory is about 100MBps
    - Byte rate for writing large SSD blocks is about 1GBps
    - Memory: Sequential access: 100MBps, random access: 10^-9 for 10kb and 10^-6 - 10^-3 for 10GB.
    - SSD: Sequential access: 1GBps, random access: 10^-5 - 10^-3 for 10kb, 10^-4 - 10^-1 for 10GB.
- Having a big sized L3 Cache is helpful because the bigger the L3 cache, the more data you can put into cache at once.

<strong>bandwidth vs. latency</strong>: Bandwidth is the total number of gigabytes we can write in so many seconds while latency is a property of every write itself.
- Latency usually have long tails, so use log-log plots.
- Memory latency varies from 10^-9 - 10^-9 sec depending on access pattern.
- SSD latency for random access varies from 10^-5 - 10^-1
- When reading or writing large blocks, we care about **throughput** or **byte-rate** not **latency**.
  - Typical throughput: Memory (100MBps); SSD (1GBps)

**Impact on Big Data Analytics**
- Clock rate is stuck at around 3GHz, and will be stuck there for the foreseeable future.
> GHz is a clock frequency, a.k.a. **clock rate** or **clock speed**, representing a cycle of time. An oscillator circuit supplies a small amount of electricity to a crystal each second that is measured in kHz,MHz, or GHz. "Hz" is an abbreviation of Hertz.
- Faster computers/disks/networks are expensive.
- When we want to analyze big data, the focus is more on moving data around or data access instead of calculation.
- The cost-effective solution is to use cluster of many cheap computers with multi-cores and break up the data so that each computer has a small fraction of data.
  - Such solution is provided by the data-centers or the cloud



> <strong>Q:What is latency?</strong>

> A: the time difference between the time at which the CPU is issuing a read or write command and, the time the command is complete.

> <strong>Q: How do we get a long tail distribution?</strong>

> A: if the probability of getting extreme values is much higher than what a normal distribution would give

> <strong>Q: What kind of distribution do we see with latency?</strong>

> A: Long tail distribution

----
### Memory Hierarchy
- Real systems have a several levels of storage types:
  - Top of Hierarchy: small and fast storages close to CPU
  - Bottom of Hierarchy: big and slow storages further from CPU
- Caching is used to transfer data between different levels of the hierarchy.
- To the programmer / compiler does not need to know
  - The hardware provides an **abstraction**: memory looks like a single large array (Memory hierarchy abstraction).
- Performance depends on program's access pattern.
- **Data processing Cluster**: collection of multiple computers linked through an ethernet connection.
  - Each computers share storage.
  - Locality: Data needed for CPU to perform computation and this data resides on the same computer as the CPU.
- Mechanism to transfer data between computers. In Spark, this distribution is called the RDD (Resilient Distribution Dataset)

> **Q: List different storage devices to their respective places in the memory hierarchy**

> A: CPU / L1 Cache / L2 Cache / L3 Cache / Main Memory / Disk Storage / Local Area Network

> **Q:Why do we use caching?**

> A: to transfer data between the different levels of the memory Hierarchy

> **Q: How is memory abstracted to the programmer?**

> A: as a large, single array
----

### History of Large Scale Computing
**Super computers**
- Usually called 'Cray','Deep Blue','Blue Gene',etc...
- Specialized hardwares that are extremely expensive.
- Designed to solve specialized problems such as predicting the weather and playing chess.

**Data Centers**
- Machines that are the physical side of the 'cloud-computing'. It is like a very big factory that has collections of commodity computers (100,000's).
- Created to provide computation for large and small organizations.

**How did this all start?**
- Started in 2003 where Larry Page and Sergey Brin developed a method for combining a large clusters of commodity computers into a big cluster and sharing the data across the cluster.
- Each file is broken into fixed-sized **chunks**.
- Each chunk is stored on multiple **chunk servers**.
- The location of the chunks is managed by the **master**.
  - Master computer knows the locations of the chunks and how to access them.

> **HDFS (Hadoop File Server)** is the storage unit of Hadoop that is used to store and process huge volumes of data on multiple datanodes. It is designed with low cost hardware that provides data across multiple hadoop clusters. It has fault tolerance and throughput. Large file is broken down into small blocks of data. HDFS has a default block size of 128MB which can be increased as per requirement. Multiple copies of each block are stored in the cluster in a distributed manner on different nodes.

> **GFS**: As the number internet users grew in the early 2000, Google faced the problem of storing increasing user data on its traditional data servers. Thousands of search queries were raised per second. There was a need for large, distributed highly fault tolerant file system to store and process the queries. The solution was the GFS. GFS consists of a single master and multiple chunk servers.

**Properties of GFS/HDFS**
- Uses commodity hardware: low cost per byte of storage.
- **Locality**: each piece of data are close to a CPU (you have access to many CPU's without moving the data).
- **Redundancy**: cluster can recover easily from server failure.
- **Simple abstraction**: the whole system is designed to function like a regular file system and the chunk mechanism is hidden.

**Map-Reduce**
- HDFS is a storage abstraction (way to store files on many machines).
- Map-Reduce is a computation abstraction that works well with HDFS.
- Allows programmer to specify parallel computation without knowing how the hardware is organized.

**Spark**
- Developed by Matei Zaharia, amplab, 2014.
- Hadoop uses shared file system (disk).
- On the other hand, Spark uses shared memory which has faster lower latency.

> **Q: What are supercomputers designed for?**

> A: Solve specialized, important problems.

> **Q:You are given a cluster of 5 servers, each with 500GB of storage. You run HDFS on these 5 servers with a redundancy factor of 4. What is the effective amount of total storage space available on the HDFS cluster?**

> A: 625GB

> **Q: Which of the following are NOT properties of HDFS?**

> A: Provides computational abstraction (Redundancy / Locality of data to CPU / Abstracts filesystem from user / uses commodity hardware).
