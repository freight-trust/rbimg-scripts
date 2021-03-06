---
title: Reproducible Deterministic Build
version: v0.1.0
spdx: ISC
---

# Reproducible and Deterministic* Hypervisor Container Images

## Motivation

Hermetic
Reproducible
Deterministic

* nixpkg
* debian

### strftime()

The C and POSIX standards define for the strftime() function and the date utility
a notation for defining date and time representations. 
Here are some examples, of how they can be used to produce ISO 8601 output:

```bash
format string 	output
%Y-%m-%d 	      1999-12-31
%Y-%j 	        1999-365
%G-W%V-%u      	1999-W52-5
%H:%M:%S      	23:59:59
```


### Reproducible Builds 

```bash
export SOURCE_DATE_TZOFFSET = $(shell dpkg-parsechangelog -SDate | tail -c6)
```

`LC_ALL=C date -u -d '2015-10-21' `


In some cases, it is preferable to keep the original times for files that have
not been created or modified during the build process:

```bash
$ find build -newermt "@${SOURCE_DATE_EPOCH}" -print0 |
    xargs -0r touch --no-dereference --date="@${SOURCE_DATE_EPOCH}"
$ zip -r product.zip build
```

`--clamp-mtime` flag which will only set the time when the file is more recent
than the value specified with --mtime:

```bash
$ tar --mtime='2015-10-21 00:00Z' --clamp-mtime -cf product.tar build
```

#### Creating a Tarball

Works with GNU Tar 1.28
> note: Mac Users must use GNU Utils, as BSDTAR does not have some of these options. use `gtar` with `g` prefix denoting `GNU` core utils

```bash
$ tar --sort=name -cf product.tar build
```
For older versions or other archive formats, it is possible to use find and sort
to achieve the same effect:

```bash
$ find build -print0 | LC_ALL=C sort -z |
    tar --no-recursion --null -T - -cf product.tar
```

Tar offers a way to specify the user and group owning the file. Using `0/0` and
`--numeric-owner` is a safe bet, as it will effectively record 0 as values:

### Deterministic 
<!-- Lyapunov -->

> Empty Directory is created and archived to be embedded within 

```bash
$ mkdir -p build/.emptydir
$ tar --owner=0 --group=0 --numeric-owner -cf pkg.tar build/.emptydir
$ tar --owner=0 --group=0 --numeric-owner -cf product.tar build
```

Full example
The recommended way to create a Tar archive is thus:

```bash
$ tar --sort=name \
      --mtime="@${SOURCE_DATE_EPOCH}" \
      --owner=0 --group=0 --numeric-owner \
      --pax-option=exthdr.name=%d/PaxHeaders/%f,delete=atime,delete=ctime \
      -cf product.tar build
```

GNU ar and other tools from binutils have a deterministic mode which will use
zero for UIDs, GIDs, timestamps, and use consistent file modes for all files. It
can be made the default by passing the --enable-deterministic-archives option to
./configure


### Famous Timestamps

2009-01-13 12:46:54 PST == 2009-01-03 18:15:05 UTC
[https://web.archive.org/web/20130316013625/http://sourceforge.net/news/?group_id=244765](http://sourceforge.net/news/?group_id=244765)

#### Timetalbes

| **Text Date:Date in human-readable text**  | **Saturday, January 3, 2009 6:15:05 PM** |
|--------------------------------------------|------------------------------------------|
| RFC 822:RFC 822 formatted date             | Sat, 03 Jan 2009 18:15:05 \+0000         |
| ISO 8601:ISO 8601 formatted date           | 2009\-01\-03T18:15:05\+00:00             |
| UNIX Timestamp:seconds since Jan 1 1970    | 1231006505                               |
| Mac Timestamp:seconds since Jan 1 1904     | 3313851305                               |
| Microsoft Timestamp:days since Dec 31 1899 | 39816\.76047                             |
| FILETIME:100\-nanoseconds since Jan 1 1601 | 12875480105000000001C96DCF:33B61A80      |

#### Well Known Hashes

```hash
4b825dc642cb6eb9a060e54bf8d69288fbee4904

6ef19b41225c5369f1c104d45d8d85efa9b057b53b14b4b9b939dd74decc5321
```

```bash
$ GIT_AUTHOR_DATE="Thu, 01 Jan 1970 00:00:00 +0000" GIT_COMMITTER_DATE="Thu, 01 Jan 1970 00:00:00 +0000" git commit --allow-empty -m 'Initial commit'
```
```diff
-rawsz sha1:20
+rawsz sha256:32

-hexsz sha1:40
+hexsz sha256:64

-zero sha1:0000000000000000000000000000000000000000
+zero sha256:0000000000000000000000000000000000000000000000000000000000000000

-algo sha1:sha1
+algo sha256:sha256

-empty_blob sha1:e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
+empty_blob sha256:473a0f4c3be8a93681a267e3b1e9a7dcda1185436fe141f7749120a303721813

-empty_tree sha1:4b825dc642cb6eb9a060e54bf8d69288fbee4904
+empty_tree sha256:6ef19b41225c5369f1c104d45d8d85efa9b057b53b14b4b9b939dd74decc5321
```

The SHA2 `"6ef19b41225c5369f1c104d45d8d85efa9b057b53b14b4b9b939dd74decc5321"` is the new 
SHA1 `"4b825dc642cb6eb9a060e54bf8d69288fbee4904"` empty tree.


## License 

SPDX: ISC
