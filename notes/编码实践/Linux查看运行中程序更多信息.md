# Linux查看运行中程序更多信息

通过查看 /proc/[PID]/ 目录查看更多有关程序的信息

更多命令见 **man proc**

```Cpp
/proc/[pid]/maps
A  file containing the currently mapped memory regions and their access permissions.  See mmap(2) for some further information about memory map‐
pings.

Permission to access this file is governed by a ptrace access mode PTRACE_MODE_READ_FSCREDS check; see ptrace(2).

The format of the file is:

address           perms offset  dev   inode       pathname
00400000-00452000 r-xp 00000000 08:02 173521      /usr/bin/dbus-daemon
00651000-00652000 r--p 00051000 08:02 173521      /usr/bin/dbus-daemon
00652000-00655000 rw-p 00052000 08:02 173521      /usr/bin/dbus-daemon
00e03000-00e24000 rw-p 00000000 00:00 0           [heap]
00e24000-011f7000 rw-p 00000000 00:00 0           [heap]
...
35b1800000-35b1820000 r-xp 00000000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a1f000-35b1a20000 r--p 0001f000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a20000-35b1a21000 rw-p 00020000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a21000-35b1a22000 rw-p 00000000 00:00 0
35b1c00000-35b1dac000 r-xp 00000000 08:02 135870  /usr/lib64/libc-2.15.so
35b1dac000-35b1fac000 ---p 001ac000 08:02 135870  /usr/lib64/libc-2.15.so
35b1fac000-35b1fb0000 r--p 001ac000 08:02 135870  /usr/lib64/libc-2.15.so
35b1fb0000-35b1fb2000 rw-p 001b0000 08:02 135870  /usr/lib64/libc-2.15.so
...
f2c6ff8c000-7f2c7078c000 rw-p 00000000 00:00 0    [stack:986]
...
7fffb2c0d000-7fffb2c2e000 rw-p 00000000 00:00 0   [stack]
7fffb2d48000-7fffb2d49000 r-xp 00000000 00:00 0   [vdso]

The address field is the address space in the process that the mapping occupies.  The perms field is a set of permissions:

    r = read
    w = write
    x = execute
    s = shared
    p = private (copy on write)

The offset field is the offset into the file/whatever; dev is the device (major:minor); inode is the inode on that device.  0 indicates that  no
inode is associated with the memory region, as would be the case with BSS (uninitialized data).

The  pathname  field  will  usually  be the file that is backing the mapping.  For ELF files, you can easily coordinate with the offset field by
looking at the Offset field in the ELF program headers (readelf -l).

There are additional helpful pseudo-paths:

     [stack]
            The initial process's (also known as the main thread's) stack.

     [stack:<tid>] (since Linux 3.4)
            A thread's stack (where the <tid> is a thread ID).  It corresponds to the /proc/[pid]/task/[tid]/ path.

     [vdso] The virtual dynamically linked shared object.  See vdso(7).

     [heap] The process's heap.

If the pathname field is blank, this is an anonymous mapping as obtained via mmap(2).  There is no  easy  way  to  coordinate  this  back  to  a
process's source, short of running it through gdb(1), strace(1), or similar.

Under Linux 2.0, there is no field giving pathname.

/proc/[pid]/mem
This file can be used to access the pages of a process's memory through open(2), read(2), and lseek(2).
    w = write
    x = execute
    s = shared
    p = private (copy on write)

The offset field is the offset into the file/whatever; dev is the device (major:minor); inode is the inode on that device.  0 indicates that  no
inode is associated with the memory region, as would be the case with BSS (uninitialized data).

The  pathname  field  will  usually  be the file that is backing the mapping.  For ELF files, you can easily coordinate with the offset field by
looking at the Offset field in the ELF program headers (readelf -l).

There are additional helpful pseudo-paths:

     [stack]
            The initial process's (also known as the main thread's) stack.

     [stack:<tid>] (since Linux 3.4)
            A thread's stack (where the <tid> is a thread ID).  It corresponds to the /proc/[pid]/task/[tid]/ path.

     [vdso] The virtual dynamically linked shared object.  See vdso(7).

     [heap] The process's heap.

If the pathname field is blank, this is an anonymous mapping as obtained via mmap(2).  There is no  easy  way  to  coordinate  this  back  to  a
process's source, short of running it through gdb(1), strace(1), or similar.

Under Linux 2.0, there is no field giving pathname.'
```

```Cpp
/proc/[pid]/io (since kernel 2.6.20)
This file contains I/O statistics for the process, for example:

    # cat /proc/3828/io
    rchar: 323934931
    wchar: 323929600
    syscr: 632687
    syscw: 632675
    read_bytes: 0
    write_bytes: 323932160
    cancelled_write_bytes: 0

The fields are as follows:

rchar: characters read
       The  number  of  bytes  which this task has caused to be read from storage.  This is simply the sum of bytes which this process passed to
       read(2) and similar system calls.  It includes things such as terminal I/O and is unaffected by whether or not actual physical  disk  I/O
       was required (the read might have been satisfied from pagecache).

wchar: characters written
       The number of bytes which this task has caused, or shall cause to be written to disk.  Similar caveats apply here as with rchar.

syscr: read syscalls
       Attempt to count the number of read I/O operations—that is, system calls such as read(2) and pread(2).

syscw: write syscalls
       Attempt to count the number of write I/O operations—that is, system calls such as write(2) and pwrite(2).

read_bytes: bytes read
       Attempt  to  count  the  number  of bytes which this process really did cause to be fetched from the storage layer.  This is accurate for
       block-backed filesystems.

write_bytes: bytes written
       Attempt to count the number of bytes which this process caused to be sent to the storage layer.

cancelled_write_bytes:
       The big inaccuracy here is truncate.  If a process writes 1MB to a file and then deletes the file, it will in fact perform  no  writeout.
       But  it  will  have  been  accounted as having caused 1MB of write.  In other words: this field represents the number of bytes which this
       process caused to not happen, by truncating pagecache.  A task can cause negative I/O too.  If this task  truncates  some  dirty  page‐
       cache, some I/O which another task has been accounted for (in its write_bytes) will not be happening.

Note: In the current implementation, things are a bit racy on 32-bit systems: if process A reads process B's /proc/[pid]/io while process B is updating one of these 64-bit counters, process A could see an intermediate result.'
Permission to access this file is governed by a ptrace access mode PTRACE_MODE_READ_FSCREDS check; see ptrace(2).
````

```Cpp
/proc/[pid]/limits (since Linux 2.6.24)
This file displays the soft limit, hard limit, and units of measurement for each of the process's resource limits (see getrlimit(2)).  Up to and
including  Linux 2.6.35, this file is protected to allow reading only by the real UID of the process.  Since Linux 2.6.36, this file is readable
by all users on the system.'
```

```Cpp
/proc/[pid]/net (since Linux 2.6.25)
See the description of /proc/net.
```

```Cpp
/proc/[pid]/oom_score (since Linux 2.6.11)
This file displays the current score that the kernel gives to this process for the purpose of selecting a process for the OOM-killer.  A  higher
score  means  that  the  process  is more likely to be selected by the OOM-killer.  The basis for this score is the amount of memory used by the
process, with increases (+) or decreases (-) for factors including:

* whether the process is privileged (-).

Before kernel 2.6.36 the following factors were also used in the calculation of oom_score:

* whether the process creates a lot of children using fork(2) (+);

* whether the process has been running a long time, or has used a lot of CPU time (-);

* whether the process has a low nice value (i.e., > 0) (+); and

* whether the process is making direct hardware access (-).

The oom_score also reflects the adjustment specified by the oom_score_adj or oom_adj setting for the process.
```

```Cpp
/proc/[pid]/smaps (since Linux 2.6.14)
This file shows memory consumption for each of the process,s mappings.  (The pmap(1) command displays similar information, in a form that may be
easier for parsing.)  For each mapping there is a series of lines such as the following:

    00400000-0048a000 r-xp 00000000 fd:03 960637       /bin/bash
    Size:                552 kB
    Rss:                 460 kB
    Pss:                 100 kB
    Shared_Clean:        452 kB
    Shared_Dirty:          0 kB
    Private_Clean:         8 kB
    Private_Dirty:         0 kB
    Referenced:          460 kB
    Anonymous:             0 kB
    AnonHugePages:         0 kB
    ShmemHugePages:        0 kB
    ShmemPmdMapped:        0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    ProtectionKey:         0
    VmFlags: rd ex mr mw me dw

The first of these lines shows the same information as is displayed for the mapping in /proc/[pid]/maps.  The following lines show the  size  of
the  mapping, the amount of the mapping that is currently resident in RAM ("Rss"), the process,s proportional share of this mapping ("Pss"), the
number of clean and dirty shared pages in the mapping, and the number of clean and dirty private pages in the mapping.   "Referenced"  indicates
the  amount  of  memory  currently  marked  as referenced or accessed.  "Anonymous" shows the amount of memory that does not belong to any file.
"Swap" shows how much would-be-anonymous memory is also used, but out on swap.

The "KernelPageSize" line (available since Linux 2.6.29) is the page size used by the kernel to back the virtual memory area.  This matches  the
size  used by the MMU in the majority of cases.  However, one counter-example occurs on PPC64 kernels whereby a kernel using 64kB as a base page
size may still use 4kB pages for the MMU on older processors.  To distinguish the two attributes, the "MMUPageSize" line (also  available  since
Linux 2.6.29) reports the page size used by the MMU.

The "Locked" indicates whether the mapping is locked in memory or not.

The  "ProtectionKey" line (available since Linux 4.9, on x86 only) contains the memory protection key (see pkeys(7)) associated with the virtual
memory area.  This entry is present only if the kernel was built with the CONFIG_X86_INTEL_MEMORY_PROTECTION_KEYS configuration option.

The "VmFlags" line (available since Linux 3.8) represents the kernel flags associated with the virtual memory area, encoded using the  following
two-letter codes:

    rd  - readable
    wr  - writable
    ex  - executable
    sh  - shared
    mr  - may read
    mw  - may write
    me  - may execute
    ms  - may share
    gd  - stack segment grows down
    pf  - pure PFN range
    dw  - disabled write to the mapped file
    lo  - pages are locked in memory
    io  - memory mapped I/O area
    sr  - sequential read advise provided
    rr  - random read advise provided
    dc  - do not copy area on fork
    de  - do not expand area on remapping
    ac  - area is accountable
    nr  - swap space is not reserved for the area
    ht  - area uses huge tlb pages
    nl  - non-linear mapping
    ar  - architecture specific flag
    dd  - do not include area into core dump
    sd  - soft-dirty flag
    mm  - mixed map area
    hg  - huge page advise flag
    nh  - no-huge page advise flag
    mg  - mergeable advise flag

"ProtectionKey" field contains the memory protection key (see pkeys(5)) associated with the virtual memory area.  Present only if the kernel was
built with the CONFIG_X86_INTEL_MEMORY_PROTECTION_KEYS configuration option. (since Linux 4.6)

The /proc/[pid]/smaps file is present only if the CONFIG_PROC_PAGE_MONITOR kernel configuration option is enabled.
```

```Cpp
/proc/[pid]/statm
Provides information about memory usage, measured in pages.  The columns are:

    size       (1) total program size
               (same as VmSize in /proc/[pid]/status)
    resident   (2) resident set size
               (same as VmRSS in /proc/[pid]/status)
    shared     (3) number of resident shared pages (i.e., backed by a file)
               (same as RssFile+RssShmem in /proc/[pid]/status)
    text       (4) text (code)
    lib        (5) library (unused since Linux 2.6; always 0)
    data       (6) data + stack
    dt         (7) dirty pages (unused since Linux 2.6; always 0)

/proc/[pid]/status
Provides much of the information in /proc/[pid]/stat and /proc/[pid]/statm in a format that's easier for humans to parse.  Here's an example:

    $ cat /proc/$$/status
    Name:   bash
    Umask:  0022
    State:  S (sleeping)
    Tgid:   17248
    Ngid:   0
    Pid:    17248
    PPid:   17200
    TracerPid:      0
    Uid:    1000    1000    1000    1000
    Gid:    100     100     100     100
    FDSize: 256
    Groups: 16 33 100
    NStgid: 17248
    NSpid:  17248
    NSpgid: 17248
    NSsid:  17200
    VmPeak:     131168 kB
    VmSize:     131168 kB
    VmLck:           0 kB
    VmPin:           0 kB
    VmHWM:       13484 kB
    VmRSS:       13484 kB
    RssAnon:     10264 kB
    RssFile:      3220 kB
    RssShmem:        0 kB
    VmData:      10332 kB
    VmStk:         136 kB
    VmExe:         992 kB
    VmLib:        2104 kB
    VmPTE:          76 kB
    VmPMD:          12 kB
    VmSwap:          0 kB
    HugetlbPages:          0 kB        # 4.4
    Threads:        1
    SigQ:   0/3067
    SigPnd: 0000000000000000
    ShdPnd: 0000000000000000
    SigBlk: 0000000000010000
    SigIgn: 0000000000384004
    SigCgt: 000000004b813efb
    CapInh: 0000000000000000
    CapPrm: 0000000000000000
    CapEff: 0000000000000000
    CapBnd: ffffffffffffffff
    CapAmb:   0000000000000000
    NoNewPrivs:     0
    Seccomp:        0
    Cpus_allowed:   00000001
    Cpus_allowed_list:      0
    Mems_allowed:   1
    Mems_allowed_list:      0
    voluntary_ctxt_switches:        150
    nonvoluntary_ctxt_switches:     545

The fields are as follows:

* Name: Command run by this process.

* Umask: Process umask, expressed in octal with a leading zero; see umask(2).  (Since Linux 4.7.)

* State: Current state of the process.  One of "R (running)", "S (sleeping)", "D (disk sleep)", "T (stopped)", "T (tracing stop)", "Z (zombie)",
  or "X (dead)".

* Tgid: Thread group ID (i.e., Process ID).

* Ngid: NUMA group ID (0 if none; since Linux 3.13).

* Pid: Thread ID (see gettid(2)).

* PPid: PID of parent process.

* TracerPid: PID of process tracing this process (0 if not being traced).

* Uid, Gid: Real, effective, saved set, and filesystem UIDs (GIDs).

* FDSize: Number of file descriptor slots currently allocated.

* Groups: Supplementary group list.

* NStgid : Thread group ID (i.e., PID) in each of the PID namespaces of which [pid] is a member.   The  leftmost  entry  shows  the  value  with
  respect to the PID namespace of the reading process, followed by the value in successively nested inner namespaces.  (Since Linux 4.1.)

* NSpid: Thread ID in each of the PID namespaces of which [pid] is a member.  The fields are ordered as for NStgid.  (Since Linux 4.1.)

* NSpgid: Process group ID in each of the PID namespaces of which [pid] is a member.  The fields are ordered as for NStgid.  (Since Linux 4.1.)

* NSsid:  descendant namespace session ID hierarchy Session ID in each of the PID namespaces of which [pid] is a member.  The fields are ordered
  as for NStgid.  (Since Linux 4.1.)

* VmPeak: Peak virtual memory size.

* VmSize: Virtual memory size.

* VmLck: Locked memory size (see mlock(3)).

* VmPin: Pinned memory size (since Linux 3.2).  These are pages that can,t be moved because something needs to directly access physical memory.

* VmHWM: Peak resident set size ("high water mark").

* VmRSS: Resident set size.  Note that the value here is the sum of RssAnon, RssFile, and RssShmem.

* RssAnon: Size of resident anonymous memory.  (since Linux 4.5).

* RssFile: Size of resident file mappings.  (since Linux 4.5).

* RssShmem: Size of resident shared memory (includes System V shared memory, mappings from tmpfs(5), and  shared  anonymous  mappings).   (since
  Linux 4.5).

* VmData, VmStk, VmExe: Size of data, stack, and text segments.

* VmLib: Shared library code size.

* VmPTE: Page table entries size (since Linux 2.6.10).

* VmPMD: Size of second-level page tables (since Linux 4.0).

* VmSwap: Swapped-out virtual memory size by anonymous private pages; shmem swap usage is not included (since Linux 2.6.34).

* HugetlbPages: Size of hugetlb memory portions.  (since Linux 4.4).

* Threads: Number of threads in process containing this thread.

* SigQ:  This field contains two slash-separated numbers that relate to queued signals for the real user ID of this process.  The first of these
  is the number of currently queued signals for this real user ID, and the second is the resource limit on the number of queued signals for this
  process (see the description of RLIMIT_SIGPENDING in getrlimit(2)).

* SigPnd, ShdPnd: Number of signals pending for thread and for process as a whole (see pthreads(7) and signal(7)).

* SigBlk, SigIgn, SigCgt: Masks indicating signals being blocked, ignored, and caught (see signal(7)).

* CapInh, CapPrm, CapEff: Masks of capabilities enabled in inheritable, permitted, and effective sets (see capabilities(7)).

* CapBnd: Capability Bounding set (since Linux 2.6.26, see capabilities(7)).

* CapAmb: Ambient capability set (since Linux 4.3, see capabilities(7)).

* NoNewPrivs: Value of the no_new_privs bit (since Linux 4.10, see prctl(2)).

* Seccomp:  Seccomp  mode of the process (since Linux 3.8, see seccomp(2)).  0 means SECCOMP_MODE_DISABLED; 1 means SECCOMP_MODE_STRICT; 2 means
  SECCOMP_MODE_FILTER.  This field is provided only if the kernel was built with the CONFIG_SECCOMP kernel configuration option enabled.

* Cpus_allowed: Mask of CPUs on which this process may run (since Linux 2.6.24, see cpuset(7)).

* Cpus_allowed_list: Same as previous, but in "list format" (since Linux 2.6.26, see cpuset(7)).

* Mems_allowed: Mask of memory nodes allowed to this process (since Linux 2.6.24, see cpuset(7)).

* Mems_allowed_list: Same as previous, but in "list format" (since Linux 2.6.26, see cpuset(7)).

* voluntary_ctxt_switches, nonvoluntary_ctxt_switches: Number of voluntary and involuntary context switches (since Linux 2.6.23).
```

```Cpp
/proc/[pid]/task (since Linux 2.6.0-test6)
This  is  a  directory  that contains one subdirectory for each thread in the process.  The name of each subdirectory is the numerical thread ID
([tid]) of the thread (see gettid(2)).  Within each of these subdirectories, there is a set of files with the same names and contents  as  under
the /proc/[pid] directories.  For attributes that are shared by all threads, the contents for each of the files under the task/[tid] subdirecto‐
ries will be the same as in the corresponding file in  the  parent  /proc/[pid]  directory  (e.g.,  in  a  multithreaded  process,  all  of  the
task/[tid]/cwd files will have the same value as the /proc/[pid]/cwd file in the parent directory, since all of the threads in a process share a
working directory).  For attributes that are distinct for each thread, the corresponding files under task/[tid] may have different values (e.g.,
various  fields  in  each of the task/[tid]/status files may be different for each thread), or they might not exist in /proc/[pid] at all.  In a
multithreaded process, the contents of the /proc/[pid]/task directory are not available if the main thread has already terminated (typically  by
calling pthread_exit(3)).
```
