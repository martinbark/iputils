[![Build Status](https://travis-ci.org/iputils/iputils.svg?branch=master)](https://travis-ci.org/iputils/iputils)
[![Coverity Status](https://scan.coverity.com/projects/1944/badge.svg?flat=1)](https://scan.coverity.com/projects/1944)

The iputils package is set of small useful utilities for Linux networking.

These tools are included in iputils
- arping
- clockdiff
- ipg
- ping
- rarpd
- rdisc
- tftpd
- tracepath
- traceroute6

If you still use [old version](http://www.skbuff.net/iputils/), please consider moving forward to new releases placed here.

This version also fully supports glibc, uClibc and musl-libc.

## Dependencies

To compile you need the following tools:

 * [Git]()
 * [CMake]() >= 2.8
 * [pkg-config]()

## Building

```sh
git clone https://github.com/iputils/iputils.git
cd iputils
mkdir build
cd build
cmake [options] ..
make
```

Where [options] are:

 * `-DCMAKE_BUIlD_TYPE=Release` Build in Release mode
 * `-DCMAKE_INSTALL_PREFIX=/path/to/install` default is `/usr/local`
 * `-DUSE_CAP=OFF` Capability support (with libcap).  Default is on
 * `-DUSE_IDN=OFF` IDN support.  Default is on
 * `-DUSE_NETTLE=OFF` nettle library for ipv6 ping.  Default is on
 * `-DUSE_GCRYPT=ON` libgcrypt library for ipv6 ping.  Default is off
 * `-DUSE_OPENSSL=ON` openssl library for ping6.  Default is off
 * `-DRDISC_SERVER=ON` rdisc server (-r option) support.  Default is off

<!-- vim: set tw=80: -->
