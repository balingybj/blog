## man /proc/meminfo

该文件是系统内存使用情况的统计信息，内容很全面。一些常见的系统性能监控工具，比如 `free`、`top` 都是从这个文件读取信息。下面讲解每个参数的含义。

-   `MemTotal`，可使用的 RAM 总量（物理 RAM 减去少量预留的 bits 和内核二进制代码占用），这个值会接近物体内存总量。

-   `MemFree`，`LowFree` + `HighFree` 的和。

-   `Buffers`，原始磁盘块数据的暂存区，不会太大（20M 左右）。

-   `Cached`，磁盘文件的内存缓存（也就是页缓存），不包括 `SwapCached`。

-   `SwapCached`，曾经被交换出又被交换回内存，在 swap 文件中还有副本。（如果内存压力太大，这些页面不需要被交换出，因为 swap 文件中已经有它们的副本。这样可以减少 I/O。）

-   `Active`，最近频繁使用的内存，除非绝对必要，通常不会被回收。

-   `Inactive`，最近使用次数较少的内存，较可能会被回收。

-   `Active(anon)`，(since Linux 2.6.28) [To be documented.]

-   `Inactive(anon)`，(since Linux 2.6.28) [To be documented.]

-   `Active(file)`，(since Linux 2.6.28) [To be documented.]

-   `Inactive(file)`，(since Linux 2.6.28) [To be documented.]

-   `Unevictable`，(since Linux 2.6.28) (From Linux 2.6.28 to 2.6.30, CONFIG_UNEVICTABLE_LRU was required.)  [To be documented.]

-   `Mlocked`，(since Linux 2.6.28) (From Linux 2.6.28 to 2.6.30, CONFIG_UNEVICTABLE_LRU was required.)  [To be documented.]

-   `HighTotal`，(Starting  with  Linux  2.6.19,  CONFIG_HIGHMEM is required.) ，`Highmem` 的总量。`hgihmem` 是物理内存 ~860M（~ 表示大概） 以上的所有内存。`Highmem` 区域的内存用于用户空间进程，或者页缓存。内核必须使用间接技巧来访问该片内存区域，所以比访问 `lowmem` 慢。

-   `HighFree`，(Starting with Linux 2.6.19, CONFIG_HIGHMEM is required.)  Amount of free highmem.

-   `LowTotal`，(Starting  with  Linux  2.6.19,  CONFIG_HIGHMEM  is required.)  `lowmem` 的总量。`Lowmem` 可以用于和 `highmem` 一样的用途，但它还可以用于存储内核本身的数据结构。除此之外，`Slab` 中的所有数据都从该区域分配。但当`lowmen` 不够用时，问题也该出现了。

-   `LowFree`，(Starting with Linux 2.6.19, CONFIG_HIGHMEM is required.)  Amount of free lowmem.

-   `MmapCopy`，(CONFIG_MMU is required.)  [To be documented.]

-   `SwapTotal`，交换空间的总量。

-   `SwapFree`，当前未使用的交换空间。

-   `Dirty`，等待被写回磁盘的内存区域。

-   `Writeback`，正在被写回磁盘的内存区域

-   `AnonPages`，(since Linux 2.6.18)，映射到用户空间页表的非文件备份页。

-   `Mapped`，被映射到内存的文件，比如库文件。（参加 mmap 函数的文文档。）

-   `Shmem`，(since Linux 2.6.32) [To be documented.]

-   `Slab`，内核数据结构缓存。

-   `SReclaimable`，(since Linux 2.6.19)，`Slab` 的一部分，可能会被回收，比如缓存。

-   `SUnreclaim`，(since Linux 2.6.19)，`Slab` 的一部分，不能被回收。

-   `KernelStack`，(since Linux 2.6.19)，分配给内核的内存总量。

-   `PageTables`，(since Linux 2.6.18)，专用于最低水平页表的内存总量。

-   `Quicklists`，(since Linux 2.6.27)，(CONFIG_QUICKLIST is required.)  [To be documented.]

-   `NFS_Unstable`，(since Linux 2.6.18)，发送到服务器的 NFS 页面，但还未提交到稳定存储器。

-   `Bounce`，用于块设备 “bounce buffers” 的内存。

-   `WritebackTmp`，(since Linux 2.6.26) FUSE 的临时回写缓冲区。

-   `CommitLimit`， (since Linux 2.6.10)，当前系统上可供分配的内存总量，单位 kb。该限制只有当严格过量统计策略被允许时才有用（/proc/sys/vm/overcommit_memory 中写入 2）。该限制根据 `/proc/sys/vm/overcommit_memory` 下描述的公式来计算。更多细节，参考内核源文件 `Documentation/vm/overcommit-accounting`。

-   `Committed_AS`，当前系统上被分配的内存总量。该值是所有进程所申请的内存总和，即使她们尚未被使用。假如一个进程申请了 1GB 的内存（通过 malloc 或类似的 API），但是只使用了其中 300MB，只计 300MB 的内存使用，即使该进程已分配整个 1GB 的地址空间。

    这个“承诺”给 VM 的 1GB 内存可以在任何时候被分配给其他进程。如果系统允许了严格过量统计策略（mode 2 in IR /proc/sys/vm/overcommit_memory），超过 `CommitLimit` 的内存申请将不被允许。这可用于保证：进程一旦成功申请内存就不再会因为内存不足而失败。

-   `VmallocTotal`，vmalloc 内存区域总量。

-   `VmallocUsed`，已经使用的 vmalloc 区域总量。

-   `VmallocChunk`，vmalloc 区域空闲的最大连续块大小。

-   `HardwareCorrupted`，(since Linux 2.6.32)，(CONFIG_MEMORY_FAILURE is required.)  [To be documented.]

-   `AnonHugePages`，(since Linux 2.6.38)，(CONFIG_TRANSPARENT_HUGEPAGE is required.)。映射到用户空间进程页表的非文件备份的大内存页。

-   `HugePages_Total`,(CONFIG_HUGETLB_PAGE is required.) 巨大内存页池的大小。

-   `HugePages_Free`，(CONFIG_HUGETLB_PAGE is required.) 巨大内存页池中未被分配的页面数。

-   `HugePages_Rsvd`，(since Linux 2.6.17) (CONFIG_HUGETLB_PAGE  is  required.) 承诺要从池中分配的巨大内存页数量，但是还未分配。这些预留的巨大内存页可以保证应用在故障时能从池中分配一个巨大页面。

-   `HugePages_Surp`，(since Linux 2.6.24) (CONFIG_HUGETLB_PAGE  is  required.) 这是池中超过 `/proc/sys/vm/nr_hugepages` 中值大小的巨大页面数。剩余巨大页面的最大数量由 `/proc/sys/vm/nr_overcommit_hugepages` 控制。

-   `Hugepagesize`，(CONFIG_HUGETLB_PAGE is required.) 巨大内存页的大小。