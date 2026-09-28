RISIX - A POSIX Compatibility Layer for xv6-riscv
=================================================

A work-in-progress POSIX compatibility layer for xv6-riscv.
This project extends the educational xv6 kernel with POSIX-style
system calls, shell features, and user-space libraries.


Overview
--------

xv6-riscv is intentionally minimal. Its system call interface is
much smaller than POSIX, and several common APIs are missing.
RISIX adds them step by step, while keeping the original xv6
design and code style intact.


What Has Been Added
-------------------

System Calls
~~~~~~~~~~~~

  API         Status   Notes
  ----------  -------  --------------------------------------------
  lseek       Done     SEEK_SET, SEEK_CUR, SEEK_END
  dup2        Done     POSIX-compatible duplication
  waitpid     Done     Supports WNOHANG, POSIX status encoding
  pledge      Done     Sandbox mechanism (OpenBSD-inspired)

File System
~~~~~~~~~~~

  Feature              Status   Notes
  -------------------  -------  ---------------------------------
  O_APPEND             Done     >> redirection works correctly
  errno (method B)     Partial  Linux-style negative error codes
  struct xv6_dirent    Done     Renamed from struct dirent to
                                avoid POSIX conflict

User-Space Libraries
~~~~~~~~~~~~~~~~~~~~

  API                              Status  Notes
  -------------------------------  ------  ----------------------
  opendir/readdir/closedir         Done    POSIX-style traversal
  pmkdir                           Done    mkdir(path, mode)
  errno                            Done    Global var + codes

Shell
~~~~~

  Feature  Status  Notes
  -------  ------  ----------------------------------------
  >        Done    Rewritten using dup2
  >>       Done    Uses O_APPEND
  <        Done    Rewritten using dup2
  |        Done    Existing, verified with redirection

Sandbox
~~~~~~~

The sandbox command restricts a child process to a set of
allowed operation categories:

  sandbox stdio,rpath cat README
  sandbox stdio cat README
  sandbox 0 echo hello

Available categories: stdio, rpath, wpath, proc, mem.


Testing Programs
----------------

  Program         Purpose
  --------------  --------------------------------------------
  lseektests      Tests lseek
  dup2tests       Tests dup2 via stdout redirection
  waitpidtests    Tests waitpid with and without WNOHANG
  pmkdirtests     Tests pmkdir
  errnotests      Tests errno after failed open
  pls             POSIX-style ls using opendir/readdir


Building and Running
--------------------

  make clean
  make qemu

Then, in the xv6 shell:

  $ lseektests
  $ dup2tests
  $ waitpidtests
  $ pls
  $ sandbox stdio,rpath cat README


POSIX Compatibility Estimate
----------------------------

Rough estimate of POSIX API coverage:

  Metric                  Coverage
  ----------------------  --------
  System call count       ~3%
  Required POSIX APIs     ~15%
  Semantic conformance    ~6%

These numbers are approximate. The goal is not full POSIX
compliance, but practical compatibility for simple tools.


Design Notes
------------

- Kernel-side error codes follow the Linux convention:
  system calls return negative values such as -ENOENT.

- User-space wrappers convert these into errno and return -1,
  matching POSIX behavior.

- struct xv6_dirent is the original xv6 directory entry.
  POSIX struct dirent is provided separately in user/dirent.h.

- The pledge mechanism is inspired by OpenBSD, not POSIX.


Known Limitations
-----------------

- errno is only set for open. Other system calls still return -1.

- errno is a global variable, not thread-local
  (xv6 has no threads).

- pledge is one-directional: calling it again can loosen
  restrictions.

- exec is always allowed inside the sandbox.

- The shell does not support quoting
  ("stdio rpath" is not parsed).

- Arrow keys corrupt argv due to naive console input handling.


Acknowledgements
----------------

Based on xv6-riscv by MIT PDOS.
See https://github.com/mit-pdos/xv6-riscv for the original.