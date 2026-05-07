# Problem
Linux uses its page cache to speed access to file data.  Data read from or written to files by processes is automatically cached in the page cache.  As the system requires RAM for other purposes, those cached pages will be freed up automatically.  However, long lived large files can create issues with some versions of the Linux kernel on some systems.

For example, a large log file for  a long running process, or a large file containing data for a long running process, you can run into corner cases.  One such corner I have seen is a long running process creates a very large log file.  The file's buffer cache (implemented in the page cache) has grown to be the size of the file (e.g. 10 GB).  Another process on the host, that is periodically (say once a minute) allocating and freeing memory encounters a failed malloc due to a lack of contiguous memory. 

That is, there is enough free memory for the process, but the memory is not contiguous and, probably because of the large file buffer, the kernel cannot create contiguous memory.   I have encountered this problem on Linux kernel versions 4.x and 5.x where, logically, the kernel should free up file buffers (i.e. page cache) but does not for reasons that are unclear.

# Solutions

## With root access
With root access, you can use the special system file named `/proc/sys/vm/drop_caches` to tell the kernel to free up the page cache.

First, run:
```
sync
```
Running `sync` will force the kernel to flush what it can to disk.

Then run:
```
echo 1 > /proc/sys/vm/drop_caches
```
Echoing `1` to `drop_caches` tells the Linux kernel to drop the page cache entries that are not strictly needed.

The downside of this is that it can have a global effect on the system that may be undesirable and may significantly impact system performance.   While we discussed a large page cache for a log file that is continuously growing or a large data file that has been fully read, there are systems that may be reading and writing over a large number of files where write-back and read caching is a real need.

In those cases, using the solution that does not require root access and is targeted to a specific file, makes more sense.
## Without root access

When root access is not available OR using system file `/proc/sys/vm/drop_caches` too risky, it is better to use call `posix_fadvise` to evict unneeded pages using `advice` flag `POSIX_FADV_DONTNEED` that can be used from utility program `vmtouch` (described below)

Using the call to `posix_fadvise` you specific the file descriptor for the file in question (e.g.. the large log file described above) and then specify the `offset` and `length` of the unneeded cached pages along with the flag `POSIX_FADV_DONTNEED`

Typically the `offset` would be zero and the `length` would be the full size of the file at the time of the call -- notably it can still be growing.

Note that while it seems odd, opening the existing file that is being appended to from another process is okay as you are directly the kernel to flush cached pages which are common to all file descriptors that point to the same on disk file.

The documentation for the `POSIX_FADV_DONTNEED` flag found in the `man` page for `posix_fadvise` is useful to read as is the the whole page:

`POSIX_FADV_DONTNEED`
- The specified data will not be accessed in the near future.

- `POSIX_FADV_DONTNEED`  attempts  to free cached pages associated with the specified region.  This is  useful, for example, while streaming large files.  A program may periodically request  the kernel to free cached data that has already been used, so that more useful cached pages are not discarded instead.

- Requests to discard partial pages are ignored.  It is preferable to preserve needed data than discard unneeded data.  If the application requires that data be considered for discarding, then `offset` and `len` must be page-aligned.

- The  implementation may attempt to write back dirty pages in the specified region, but this is not guaranteed.  Any unwritten dirty pages will not be freed.  If the  application  wishes  to  ensure that dirty pages will be released, it should call `fsync(2)` or `fdatasync(2)` first.


# Evict Cached Pages With Utility `vmtouch`

The `vmtouch` utility provides access to a variety of file m
anagement kernel features including `POSIX_FADV_DONTNEED`.  The package to be installed is usually named `vmtouch` and is easy to compile from source: https://github.com/hoytech/vmtouch

Simply run this command to evict all pages from a given file or files:

```
vmtouch -e <path-to-file> <...>
```

## Testing
To test that `vmtouch` works and to understand how to troubleshoot, start by seeing the state of memory by running `free -h`

```
> free -h
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       755Mi       5.5Gi        45Mi       1.8Gi       7.1Gi
Swap:          199Mi          0B       199Mi
```

`1.8` gigabytes are in use in the buffer cache.

Create a large file in `/var/tmp` and then `cat` it to put in into use on the system

```
> fallocate -l 2GiB /var/tmp/mylargefile

> cat /var/tmp/mylargefile > /dev/null
```

Now run `free -h` again
```
> free -h
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       771Mi       3.6Gi        45Mi       3.6Gi       7.1Gi
Swap:          199Mi          0B       199Mi
```

`free -h` shows the buffer cache use has, as expected, increased by 2 gigabytes.

Now evict the cached pages from `/var/tmp/mylargefile` using the `vmtouch -e` command

```
> vmtouch -e /var/tmp/mylargefile

           Files: 1
     Directories: 0
   Evicted Pages: 122071 (1G)
         Elapsed: 0.040787 seconds
```

Run `free -h` again

```
> free -h
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       746Mi       5.5Gi        45Mi       1.8Gi       7.1Gi
Swap:          199Mi          0B       199Mi
```

Note how the buffer cache use has dropped back down to 1.8 gigabytes.

Don't forget to remove the large file
```
> rm /var/tmp/mylargefile
```

