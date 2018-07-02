title: A Note on Linux Hugepages
date: 2018-07-01 18:30:20
comments: true
tags: ['Linux', 'Programming', 'Hugepage']
categories: Programming
---

Page table is where Linux stores virtual to physical page address translation, and its size can get huge when memory usage is high. One way to reduce the size of page tables, and reduce the number of page faults, is to use huge pages. I've been digging some information on hugepages for my own curiosity, and it looks like Linux has pretty good support for huge pages. And this blog serves as a quick note on my readings.

<!-- more -->

## Hugepages

The sysctl directory contains `/sys/kernel/mm/hugepages/hugepages-{pagesize}kB/` control files and information on hugepages, where pagesize could be 1048576 or 2048, corresponding to 1GB or 2MB of hugepage size.

To get information on hugepages on your Linux systems, the `hugepages` directory contains the controlling files:

- `nr_hugepages`
- `nr_hugepages_mempolicy`
- `nr_overcommit_hugepages`
- `free_hugepages`
- `resv_hugepages`
- `surplus_hugepages`.

You can also get hugepage-related information from `/proc/meminfo`:

```
    HugePages_Total:    2048
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
```

The [Kernel Documentation of Hugetlbpage](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt) contains the detailed information and explanation of the purpose and usage of hugepage files, as well as `meminfo` fields.


## Allocating Hugepages

The most convenient way to reserve hugepages on x86_64 Linux is to echo into the sysctl file `/sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages`, e.g.:

```
# echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

And for user to access and use the huge pages, Linux actually provides a quite convenient interface: the `hugetlbfs` file system, e.g.:

```
mount -t hugetlbfs nodev /mnt/huge
```

Which mounts a pseudo filesystem of type `hugetlbfs` on `/mnt/huge`, and uses the default huge pagesize specified by the system, and all files created inside the directory uses huge pages.

And after that, you can use hugepage-backed memory by creating files inside `/mnt/huge` directory. See example in Linux source tree: [hugepage-mmap.c](https://github.com/torvalds/linux/blob/master/tools/testing/selftests/vm/hugepage-mmap.c). The author takes the following steps:

- Open a file inside `/mnt` with read-write permission.
- Map memory to the file using `mmap`, with proper protection and flags set (`PROT_READ | PROR_WRITE` and `MAP_SHARED` in this case).
- Use the hugepage-backed memory as usual.
- Clean up memory and file.

## Enable Hugepage On Start

According to Linux [Kernel Documentation](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt):

```
System administrators may want to put this command in one of the local rc
init files.  This will enable the kernel to allocate huge pages early in
the boot process when the possibility of getting physical contiguous pages
is still very high.
```

And to quote the [LWN article](https://lwn.net/Articles/376606/):

```
If the huge page pool is statically allocated at boot-time, then this section
will not be relevant as the huge pages are guaranteed to exist. In the event
the system needs to dynamically allocate huge pages throughout its lifetime,
then external fragmentation may be a problem.
```

So to avoid external fragmentation and make sure that the hugepage allocation is always successful, we may want to reserve hugepages memory regions on boot time. This [Debian Wiki Page](https://wiki.debian.org/Hugepages) provides a way to do that. To reserve number of hugepages, add the following line in `/etc/sysctl.conf`:

```
vm.nr_hugepages = 1024
```

And to mount it automatically on system start, simply add to `/etc/fstab`:

```
hugetlbfs /hugepages hugetlbfs mode=1770,gid=2021 0 0
```

## Advanced Topics

There are other advanced topics, which probably will not be covered in the scope of this quick note:

- __Transparent Huge Pages__: system automatically decides if memory should be backed by hugepages, which make usage of hugepage memory much easier.
- __libhugetlbfs APIs__: `libhugetlbfs` provides programmer APIs to manage and access hugepage memory as well. Hugepage library utilities `hugectl` can overload Linux standard `shmget()` functions to allow huge pages to be used by allocating shared memory.
- __Text And Data__: `hugectl` has options to run an application, with its `text` and `data` section mapped by hugepages, which gives potential performance benefits.

These are ideas worth digging into in the future, for applications where hugepages can potential give a good performance boost.

## References

- [Kernel Documentation of Hugetlbpage](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)
- [LWN Intro to Huge Pages (1 Intro)](https://lwn.net/Articles/374424/)
- [LWN Intro to Huge Pages (2 Interfaces)](https://lwn.net/Articles/375096/)
- [LWN Intro to Huge Pages (3 Administration)](https://lwn.net/Articles/375096/)
- [Redhat Documentation on Performance Tuning: 5.2 Huge Pages And Transparent Huge Pages](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-memory-transhuge#s-memory-configure_hugepages)
- [hugepage-mmap.c: Hugepage example in Linux source tree](https://github.com/torvalds/linux/blob/master/tools/testing/selftests/vm/hugepage-mmap.c)
