### ARM64 on linux
This project just contain notes and examples to help me setup and run examples
on aarm64.

### Try running a arm64v8 container
```console
$ docker run --rm -ti arm64v8/fedora uname -m
standard_init_linux.go:211: exec user process caused "exec format error"
```
So we need to install a few things go get this working...

### Install QEMU
```console
$ sudo dnf -y install qemu qemu-user qemu-user-static
$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

After this we should be able to run the same container we tried above:
```console
$ docker run --rm -ti arm64v8/fedora uname -m
aarch64
```

### Troubleshooting
I'm not sure how I managed to get my system into this state but somehow
I was not able to install qemu-user-static:
```console
$ sudo dnf -y install qemu-user-static
Last metadata expiration check: 3:12:04 ago on Sun 31 Jan 2021 08:26:52 AM CET.
Dependencies resolved.
================================================================================================================================================
 Package                                Architecture                 Version                                Repository                     Size
================================================================================================================================================
Installing:
 qemu-user-static                       x86_64                       2:4.1.1-1.fc31                         updates                        14 M

Transaction Summary
================================================================================================================================================
Install  1 Package

Total download size: 14 M
Installed size: 141 M
Downloading Packages:
qemu-user-static-4.1.1-1.fc31.x86_64.rpm                                                                        6.2 MB/s |  14 MB     00:02    
------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                           5.8 MB/s |  14 MB     00:02     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                        1/1 
  Installing       : qemu-user-static-2:4.1.1-1.fc31.x86_64                                                                                 1/1 
Error unpacking rpm package qemu-user-static-2:4.1.1-1.fc31.x86_64
  Verifying        : qemu-user-static-2:4.1.1-1.fc31.x86_64                                                                                 1/1 

Failed:
  qemu-user-static-2:4.1.1-1.fc31.x86_64
```
I turned out I had a /usr/bin/qemu-arm-static:
```console
$ sudo rm -rf /usr/bin/qemu-arm-static
```


