# icmpsh-s-linux
GNU/Linux version of the https://github.com/inquisb/icmpsh slave, plus the C
master from https://github.com/bdamele/icmpsh.

## Included files

- `icmpsh-s-linux.c` / `icmpsh-s-linux` - slave (source / prebuilt)
- `icmpsh-m.c` / `icmpsh-m` - master (source / prebuilt)

The prebuilt binaries are 32-bit i386, statically linked against a 2.6.9-era
glibc and tested on Red Hat Enterprise Linux 4 (kernel 2.6.9-5.EL i686).

`gcc icmpsh-s-linux.c -o icmpsh-s-linux`

might want to link it statically:

`gcc icmpsh-s-linux.c -o icmpsh-s-linux -static`

## Building for old kernels (e.g. RHEL 4 / 2.6.9-5.EL i686)

Build **natively on the target** (its own `gcc`/`glibc`), statically:

`gcc -O2 -static icmpsh-s-linux.c -o icmpsh-s-linux`

A binary built on a modern distro (even 32-bit `-m32`) links a modern glibc
and requires a new kernel, so it will **not** run on 2.6.9.

## Socket types / privileges

The program first tries an ICMP *ping* socket (`SOCK_DGRAM` + `IPPROTO_ICMP`),
which only exists on Linux >= 2.6.39. If that is unavailable (e.g. 2.6.9) it
falls back to a *raw* ICMP socket (`SOCK_RAW`), which works on all kernels but
**always requires root** (the checksum and IPv4 header handling are done in
software for raw sockets). On modern kernels a non-root user may still use it
if allowed by the net.ipv4.ping_group_range /proc/sys/ setting.

## Master (`icmpsh-m.c`)

Build natively (statically for old kernels):

`gcc -O2 -static icmpsh-m.c -o icmpsh-m`

**Before running the master, disable kernel ICMP echo replies on the master
host**, otherwise the OS's own echo reply races the master's crafted reply and
the slave will feed its own output back into the shell (endless
`/bin/sh: line N: ...` errors):

```
sysctl -w net.ipv4.icmp_echo_ignore_all=1
./icmpsh-m
```

Restore it afterwards with `sysctl -w net.ipv4.icmp_echo_ignore_all=0`.
This applies to the `icmpsh_m.py` master as well.

![Screen](SCREEN.PNG?raw=true)
