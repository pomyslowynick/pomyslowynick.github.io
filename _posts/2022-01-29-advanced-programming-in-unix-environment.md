Important ANSI C features:
- Function prototypes
- Generic pointers (void *)
- Abstract data types (size_t, pid_t) (abstract data types can be represented by the OS, instead of the type Int the OS can use the right equivalent)

"C always gives you enough rope to hang yourself"

Well written C should always handle errors 
- it should provide meaningful return values
- errno variable - acts as an integer value representing the reason for failure.
- `strerror` and `perror` are meaningful return values for meaningful error messages.

> root is not allowed to log in remotely

to mount a virtual disk
`mount /dev/wd0a /mnt` - at least in the NetBSD example.
`umount /mnt` to unmount

`echo $?` - print out the last returned value in the terminal

> If the compiler doesn't know what a function returns it will assume an integer.

`whoami` command has been deprecated, id is the modern version.

Year 2038 problem - the time integer value will be overflown again on 19-01-2038
64 bit value will be able to represent the next few billion years, goes to show how many more values can be represented with the growing powers of 2.

* clock time
* user CPU time
* system CPU time
blocked I/O time is not accounted in user nor system CPU time. 

`time` can be used to time the commands

What is `xargs`?

File I/O:
`unbuffered I/O` - done through system calls like open(2), read(2), executes in kernel space.
`buffered I/O`  - provided through library functions like fopen(3), fread(3), executes in user space.

System buffers the output, it doesn't print as user requests it but rather collects buffers to print at the most convenient time.

Ctrl+D sends EOF
Ctrl+C sends a SIGINT

`echo $$` - prints the PID of the shell

ctags and exctags

> Check your umask. Clean your tmp files. Read a man page. It's not a network problem. And don't "monkey around" with /etc/hosts.

Advice from a seasoned admin

Know tcpdump, be able to read a pcap.
## Week 02

### Segment 1 - File Descriptors
Interesting investigation into the allowed number of file descriptors on a platform, I always thought i was a fixed value. Turns out it depends on the platform and is dynamically allocated.
Writing and reading some code is the best way to further understanding and confirm that it is correct.

`fstat`
`ulimit -n`
`pstat -f`

``` C
_POSIX_OPEN_MAX

    The value of this macro is the most restrictive limit permitted by POSIX for the maximum number of files that a single process can have open simultaneously. Its value is 16.

```

POSIX standard has a list of minimum upper bounds for the system limit parameters.

### Segment 2 - open(2) and close(2)
`creat` was obsoleted by open, it used to be necessary to obtain a file descriptor, but as it wasn't atomic it lead to race conditions within the code.

`O_EXCL` - error if `O_CREAT` and file already exists (atomic exclusive open of a file). Without this if we test for the existence of a file followed by an open we can introduce a race condition again, and a vulnerability. This type of vulnerability where time passes between creation and read is known as ["TOCTOU"](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use), I assume that the delay can allow an attacker to replace the file in the meantime.

`open` is used to open a file and obtain a file descriptor for it, some systems support `openat` which can handle relative paths in an atomic fashion.
Apparently `openat` can be useful in some avoid `TOCTOU`s, it is left as an exercise to figure out how.

I think `openat` is preventing an attack vector where the relative directory structure changes during the program execution?
### Tool Tip: sizeof != strlen
Structs may be padded to improve their alignment in memory, 12 bytes struct might have sizeof 16 so that it fits better.

`define p sizeof($arg0)` alias for running the same command in GDB (can you persist that?)

`whatis` - get the type of a variable

C Flex arrays

`sizeof` 

### Week 02, Segment 3 - read(2), write(2), lseek(2)
You can't seek on a pipe, managing the file offset can be done with lseek.
File with a hole in it, a "sparse" file.

In the example prof. creates a file, writes first 10 bytes and then `lseek`s 10 megabytes forward to write 10 more bytes. As a result the 10 megabytes are just "empty" data, represented as nulls in hexdump.
`df .`  

The support for sparse files depends on the filesystem.

Beatiful example of how investigating the file size on a network file system can give a different value depending on the lag between the time it takes to actually save it.

`stat -f  "%k" filename` 