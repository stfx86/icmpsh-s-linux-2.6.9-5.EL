# icmpsh-s-linux
GNU/Linux version of the https://github.com/inquisb/icmpsh slave

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

Works with e.g. the original icmpsh_m.py master (the original project linked above).

![Screen](SCREEN.PNG?raw=true)
