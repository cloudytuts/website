---
title: "Know What's in Your Container Images"
date: 2021-06-23T11:42:07-04:00
draft: false
author: serainville
description: Do you know what's running in your containers? Audit your images to avoid packages that leave you vulnerable to exploitation
tags:
- containers
- security
---

How well do you know the base container images running your services and tooling? It's a question that is largely ignored, as we often place an enormous amount of trust in them. Yet, securing your workloads and underlying infrastructure requires that we answer the question 'What is running in my containers?'

As developers we are largely satisfied that our application runs in an expected manner once we've containerized it. However, that leads to unknowingly including packages and services within the container runtime that opens us up to security exploitations. 

## Cataloging Container Packages
Unless you are working with scratch base image, your container is likely filled with packages that are not crucial to the functionality of your app. Knowing what resides inside of your chosen base images will help you secure your workloads.

A popular open-source tool for cataloging packages inside of images is [syft](https://github.com/anchore/syft), which scans images for any packages installed, including deb, NPM, Python, etc.

### Using Syft Container Package Scanner
The following shows the output of **syft** against an official WordPress image. It's a long list of packages, each bringing with them their own vulnerabilities. 

```shell {hl_lines=["19-21","35-39"]}
❯ syft wordpress:latest
 ✔ Parsed image            
 ✔ Cataloged packages      [223 packages]

NAME                       VERSION                       TYPE 
adduser                    3.118                         deb   
apache2                    2.4.38-3+deb10u4              deb   
apache2-bin                2.4.38-3+deb10u4              deb   
apache2-data               2.4.38-3+deb10u4              deb   
apache2-utils              2.4.38-3+deb10u4              deb   
apt                        1.8.2.3                       deb   
autoconf                   2.69-11                       deb   
base-files                 10.3+deb10u9                  deb   
base-passwd                3.5.46                        deb   
bash                       5.0-4                         deb   
binutils                   2.31.1-16                     deb   
binutils-common            2.31.1-16                     deb   
binutils-x86-64-linux-gnu  2.31.1-16                     deb   
bsdutils                   1:2.33.1-0.1                  deb   
bzip2                      1.0.6-9.2~deb10u1             deb   
ca-certificates            20200601~deb10u2              deb   
coreutils                  8.30-3                        deb   
cpp                        4:8.3.0-1                     deb   
cpp-8                      8.3.0-6                       deb   
curl                       7.64.0-4+deb10u2              deb   
dash                       0.5.10.2-5                    deb   
debconf                    1.5.71                        deb   
debian-archive-keyring     2019.1+deb10u1                deb   
debianutils                4.8.6.1                       deb   
diffutils                  1:3.7-3                       deb   
dpkg                       1.19.7                        deb   
dpkg-dev                   1.19.7                        deb   
e2fsprogs                  1.44.5-1+deb10u3              deb   
fdisk                      2.33.1-0.1                    deb   
file                       1:5.35-4+deb10u2              deb   
findutils                  4.6.0+git+20190209-2          deb   
fontconfig-config          2.13.1-2                      deb   
fonts-dejavu-core          2.37-1                        deb   
g++                        4:8.3.0-1                     deb   
g++-8                      8.3.0-6                       deb   
gcc                        4:8.3.0-1                     deb   
gcc-8                      8.3.0-6                       deb   
gcc-8-base                 8.3.0-6                       deb   
ghostscript                9.27~dfsg-2+deb10u4           deb   
gpgv                       2.2.12-1+deb10u1              deb   
grep                       3.3-1                         deb   
gzip                       1.9-3                         deb   
hostname                   3.21                          deb   
imagemagick-6-common       8:6.9.10.23+dfsg-2.1+deb10u1  deb   
init-system-helpers        1.56+nmu1                     deb   
libacl1                    2.2.53-4                      deb   
libapr1                    1.6.5-1+b1                    deb   
libaprutil1                1.6.1-4                       deb   
libaprutil1-dbd-sqlite3    1.6.1-4                       deb   
libaprutil1-ldap           1.6.1-4                       deb   
libapt-pkg5.0              1.8.2.3                       deb   
libargon2-1                0~20171227-0.2                deb   
libasan5                   8.3.0-6                       deb   
libatomic1                 8.3.0-6                       deb   
libattr1                   1:2.4.48-4                    deb   
libaudit-common            1:2.8.4-3                     deb   
libaudit1                  1:2.8.4-3                     deb   
libavahi-client3           0.7-4+deb10u1                 deb   
libavahi-common-data       0.7-4+deb10u1                 deb   
libavahi-common3           0.7-4+deb10u1                 deb   
libbinutils                2.31.1-16                     deb   
libblkid1                  2.33.1-0.1                    deb   
libbrotli1                 1.0.7-2+deb10u1               deb   
libbsd0                    0.9.1-2+deb10u1               deb   
libbz2-1.0                 1.0.6-9.2~deb10u1             deb   
libc-bin                   2.28-10                       deb   
libc-dev-bin               2.28-10                       deb   
libc6                      2.28-10                       deb   
libc6-dev                  2.28-10                       deb   
libcap-ng0                 0.7.9-2                       deb   
libcc1-0                   8.3.0-6                       deb   
libcom-err2                1.44.5-1+deb10u3              deb   
libcups2                   2.2.10-6+deb10u4              deb   
libcupsimage2              2.2.10-6+deb10u4              deb   
libcurl4                   7.64.0-4+deb10u2              deb   
libdb5.3                   5.3.28+dfsg1-0.5              deb   
libdbus-1-3                1.12.20-0+deb10u1             deb   
libde265-0                 1.0.3-1+b1                    deb   
libdebconfclient0          0.249                         deb   
libdpkg-perl               1.19.7                        deb   
libedit2                   3.1-20181209-1                deb   
libexpat1                  2.2.6-2+deb10u1               deb   
libext2fs2                 1.44.5-1+deb10u3              deb   
libfdisk1                  2.33.1-0.1                    deb   
libffi6                    3.2.1-9                       deb   
libfftw3-double3           3.3.8-2                       deb   
libfontconfig1             2.13.1-2                      deb   
libfreetype6               2.9.1-3+deb10u2               deb   
libgcc-8-dev               8.3.0-6                       deb   
libgcc1                    1:8.3.0-6                     deb   
libgcrypt20                1.8.4-5                       deb   
libgdbm-compat4            1.18.1-4                      deb   
libgdbm6                   1.18.1-4                      deb   
libglib2.0-0               2.58.3-2+deb10u2              deb   
libgmp10                   2:6.1.2+dfsg-4                deb   
libgnutls30                3.6.7-4+deb10u6               deb   
libgomp1                   8.3.0-6                       deb   
libgpg-error0              1.35-1                        deb   
libgs9                     9.27~dfsg-2+deb10u4           deb   
libgs9-common              9.27~dfsg-2+deb10u4           deb   
libgssapi-krb5-2           1.17-3+deb10u1                deb   
libheif1                   1.3.2-2~deb10u1               deb   
libhogweed4                3.4.1-1                       deb   
libicu63                   63.1-6+deb10u1                deb   
libidn11                   1.33-2.2                      deb   
libidn2-0                  2.0.5-1+deb10u1               deb   
libijs-0.35                0.35-14                       deb   
libisl19                   0.20-2                        deb   
libitm1                    8.3.0-6                       deb   
libjansson4                2.12-1                        deb   
libjbig0                   2.1-3.1+b2                    deb   
libjbig2dec0               0.16-1                        deb   
libjpeg62-turbo            1:1.5.2-2+deb10u1             deb   
libk5crypto3               1.17-3+deb10u1                deb   
libkeyutils1               1.6-6                         deb   
libkrb5-3                  1.17-3+deb10u1                deb   
libkrb5support0            1.17-3+deb10u1                deb   
liblcms2-2                 2.9-3                         deb   
libldap-2.4-2              2.4.47+dfsg-3+deb10u6         deb   
libldap-common             2.4.47+dfsg-3+deb10u6         deb   
liblqr-1-0                 0.4.2-2.1                     deb   
liblsan0                   8.3.0-6                       deb   
libltdl7                   2.4.6-9                       deb   
liblua5.2-0                5.2.4-1.1+b2                  deb   
liblz4-1                   1.8.3-1                       deb   
liblzma5                   5.2.4-1                       deb   
libmagic-mgc               1:5.35-4+deb10u2              deb   
libmagic1                  1:5.35-4+deb10u2              deb   
libmagickcore-6.q16-6      8:6.9.10.23+dfsg-2.1+deb10u1  deb   
libmagickwand-6.q16-6      8:6.9.10.23+dfsg-2.1+deb10u1  deb   
libmount1                  2.33.1-0.1                    deb   
libmpc3                    1.1.0-1                       deb   
libmpfr6                   4.0.2-1                       deb   
libmpx2                    8.3.0-6                       deb   
libncurses6                6.1+20181013-2+deb10u2        deb   
libncursesw6               6.1+20181013-2+deb10u2        deb   
libnettle6                 3.4.1-1                       deb   
libnghttp2-14              1.36.0-2+deb10u1              deb   
libnuma1                   2.0.12-1                      deb   
libonig5                   6.9.1-1                       deb   
libopenjp2-7               2.3.0-2+deb10u2               deb   
libp11-kit0                0.23.15-2+deb10u1             deb   
libpam-modules             1.3.1-5                       deb   
libpam-modules-bin         1.3.1-5                       deb   
libpam-runtime             1.3.1-5                       deb   
libpam0g                   1.3.1-5                       deb   
libpaper1                  1.1.28                        deb   
libpcre3                   2:8.39-12                     deb   
libperl5.28                5.28.1-6+deb10u1              deb   
libpng16-16                1.6.36-6                      deb   
libprocps7                 2:3.3.15-2                    deb   
libpsl5                    0.20.2-2                      deb   
libquadmath0               8.3.0-6                       deb   
librtmp1                   2.4+20151223.gitfa8646d.1-2   deb   
libsasl2-2                 2.1.27+dfsg-1+deb10u1         deb   
libsasl2-modules-db        2.1.27+dfsg-1+deb10u1         deb   
libseccomp2                2.3.3-4                       deb   
libselinux1                2.8-1+b1                      deb   
libsemanage-common         2.8-2                         deb   
libsemanage1               2.8-2                         deb   
libsepol1                  2.8-1                         deb   
libsigsegv2                2.12-2                        deb   
libsmartcols1              2.33.1-0.1                    deb   
libsodium23                1.0.17-1                      deb   
libsqlite3-0               3.27.2-3+deb10u1              deb   
libss2                     1.44.5-1+deb10u3              deb   
libssh2-1                  1.8.0-2.1                     deb   
libssl1.1                  1.1.1d-0+deb10u6              deb   
libstdc++-8-dev            8.3.0-6                       deb   
libstdc++6                 8.3.0-6                       deb   
libsystemd0                241-7~deb10u7                 deb   
libtasn1-6                 4.13-3                        deb   
libtiff5                   4.1.0+git191117-2~deb10u2     deb   
libtinfo6                  6.1+20181013-2+deb10u2        deb   
libtsan0                   8.3.0-6                       deb   
libubsan1                  8.3.0-6                       deb   
libudev1                   241-7~deb10u7                 deb   
libunistring2              0.9.10-1                      deb   
libuuid1                   2.33.1-0.1                    deb   
libwebp6                   0.6.1-2                       deb   
libwebpmux3                0.6.1-2                       deb   
libx11-6                   2:1.6.7-1+deb10u2             deb   
libx11-data                2:1.6.7-1+deb10u2             deb   
libx265-165                2.9-4                         deb   
libxau6                    1:1.0.8-1+b2                  deb   
libxcb1                    1.13.1-2                      deb   
libxdmcp6                  1:1.1.2-3                     deb   
libxext6                   2:1.3.3-1+b2                  deb   
libxml2                    2.9.4+dfsg1-7+deb10u1         deb   
libzip4                    1.5.1-4                       deb   
libzstd1                   1.3.8+dfsg-3+deb10u2          deb   
linux-libc-dev             4.19.181-1                    deb   
login                      1:4.5-1.1                     deb   
lsb-base                   10.2019051400                 deb   
m4                         1.4.18-2                      deb   
make                       4.2.1-1.2                     deb   
mawk                       1.3.3-17+b3                   deb   
mime-support               3.62                          deb   
mount                      2.33.1-0.1                    deb   
ncurses-base               6.1+20181013-2+deb10u2        deb   
ncurses-bin                6.1+20181013-2+deb10u2        deb   
openssl                    1.1.1d-0+deb10u6              deb   
passwd                     1:4.5-1.1                     deb   
patch                      2.7.6-3+deb10u1               deb   
perl                       5.28.1-6+deb10u1              deb   
perl-base                  5.28.1-6+deb10u1              deb   
perl-modules-5.28          5.28.1-6+deb10u1              deb   
pkg-config                 0.29-6                        deb   
poppler-data               0.4.9-2                       deb   
procps                     2:3.3.15-2                    deb   
re2c                       1.1.1-1                       deb   
sed                        4.7-1                         deb   
sensible-utils             0.0.12                        deb   
sysvinit-utils             2.93-8                        deb   
tar                        1.30+dfsg-6                   deb   
twentynineteen             2.0.0                         npm   
twentytwenty               1.7.0                         npm   
twentytwentyone            1.3.0                         npm   
tzdata                     2021a-0+deb10u1               deb   
ucf                        3.0038+nmu1                   deb   
util-linux                 2.33.1-0.1                    deb   
xz-utils                   5.2.4-1                       deb   
zlib1g                     1:1.2.11.dfsg-1               deb  
```

You should audit every package installed in any image that will be expected to run in production. The output above really only lists packages installed in the final layer of your images, though there could be many more hidden in other layers.

Using the `--scope` flag of Syft we can show any additional packages hidden within.

```shell
❯ syft wordpress:latest --scope all-layers
 ✔ Parsed image            
 ✔ Cataloged packages      [1175 packages]
```


### Security Implications of not Auditing Packages
The catalog above shows 223 packages installed in the latest WordPress Docker image (as of June 24th, 2021). While many of them are likely necessary for running a PHP application using Apache Web Server, not all are required for every use case. 

Take the following eight packages from the list into consideration. They are development tools used for compiling code written in C or C++, and in the case of a PHP or Apache, sometime needed when installing native extensions for expanded functionality. 

```shell
cpp                        4:8.3.0-1                     deb   
cpp-8                      8.3.0-6                       deb   
g++                        4:8.3.0-1                     deb   
g++-8                      8.3.0-6                       deb   
gcc                        4:8.3.0-1                     deb   
gcc-8                      8.3.0-6                       deb   
gcc-8-base                 8.3.0-6                       deb   
make                       4.2.1-1.2                     deb 
```

The inclusion of such tools in your final images provides a gateway for more serious attacks. One such example of an attack is known as the [DirtyCow](https://www.cs.toronto.edu/~arnold/427/18s/427_18S/indepth/dirty-cow/index.html), which was a Linux kernel privilege escalation vulnerability found in all versions of the Linux kernel. 

The vulnerability could easily be exploited by writing a simple C source file, and then compiling that code locally for use ([POC source files](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)).

### CURL
Another interesting find is the curl package. A very common program found on most Linux installations and container images. 

```text
curl                       7.64.0-4+deb10u2              deb
```

The tool provides a means for downloading files from remote sources. It fairly common and seemingly inconsequential addition to any system. 

However, it's a tool often used by malious actors to further an attack, by downloading additional files and tooling into your container. 

## Mitigating Attacks
The most important step for mitigating any potential attacks is understanding what attack vectors are available. We've identified two in the section above.

We now need to understand what our next steps should be, in an effort to mitigate any of those attacks.

1. Is the package needed for functionality?
2. How do I safely remove it?


### Remove Packages
Removing packages from base images is not as not a simple task. The reason for this is the result of the underlying structure of an container image. 

A single image is made of one or more filesystem layers, each one created by a command in your Dockerfile. Removing an item from the top layer does not remove it from the image. It will continue to exist in the layer it was originally added.

Two possible solution for addressing this issue are to:
1. Use multi-stage builds and copy required files from one stage to another.
2. Deleting the file in your Dockerfile and then using the `--squash` flag during the build.

### Handling Required Packages
While it would be nice to remove any package that provides a mechanism for attackers to exploit your systems, it's not always possible to do.

For the build tools (`cpp, g++, make`) an AppArmor profile can be employed to restrict file access.

For packages like `curl`, a network policy or firewall can be used to restrict which addresses can be called.

## Conclusion
We often take our base images for granted so long as our applications run. This often leaves us vulnerable to attacks that could have been easily avoided, had we audited the image and mitigated any potential attacks.