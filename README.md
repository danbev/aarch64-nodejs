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

### Build a container
```console
$ docker build -t aarch64-nodejs .
```

### Running the container
```console
$ docker run -ti aarch64-nodejs bash
```

Cloning a specific version/tag of Node.js:
```console
$ git clone --depth 1 --branch v14.15.4 https://github.com/nodejs/node.git node
$ cd node
$ ./configure && make -j8
```

### Tagging and pushing to docker hub
```console
$ docker login -u <username>
$ docker tag aarch64-node-14.15.4 dbevenius/aarch64-node:14.15.4
$ docker push dbevenius/aarch64-node:14.15.4
```

### Testing yarn
This was actually the purpose of this repo to test an issue when running
yarn.
```console
[root@5fde00944ba7 node]# uname -m
aarch64

$ export PATH=/node:$PATH
$ node --version
v14.15.4

$ /node/deps/npm/bin/npm-cli.js --version
6.14.10

$ mkdir yarn-test && cd yarn-test
$ /node/deps/npm/bin/npm-cli.js init
$ /node/deps/npm/bin/npm-cli.js  install -g yarn --unsafe-perm
$ /node/out/bin/yarn init 
$ /node/out/bin/yarn install
yarn install v1.22.10
[1/4] Resolving packages...
success Already up-to-date.
Done in 3.11s.
/node/out/bin/yarn list
...
Done in 3.53s.
```
Now I want to try the same using a binary of Node:
```console
$ mkdir binary-node
$ cd binary-node/
$ curl https://nodejs.org/dist/v14.15.4/node-v14.15.4-linux-arm64.tar.gz | tar xpfz -
$ cd node-v14.15.4-linux-arm64/
$ export PATH=$PWD/bin:$PATH
$ which node
/node/binary-node/node-v14.15.4-linux-arm64/bin/node
$ mkdir sample
$ cd sample
$ npm init
$ npm install -g yarn --unsafe-perm
$ yarn add lodash
yarn add v1.22.10
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
info Direct dependencies
└─ lodash@4.17.20
info All dependencies
└─ lodash@4.17.20
Done in 14.88s.
[root@5fde00944ba7 sample]#
```
And that also works.


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

### AWS
For this I created and launched a new instance using RHEL 8 and arm64.
```console
$ ssh -i path_to_aws_key ec2-user@public_dns
```
Install utils and build node:
```console
$ sudo dnf install -y git gcc-c++ which make python3 openssl-devel procps-ng
$ git clone --depth 1 --branch v14.15.4 https://github.com/nodejs/node.git node
$ cd node
$ ./configure --debug && make -j2
```

Using Node.js binary on Centos aarch64
```console
$ ssh -i path_to_aws_key centos@public_ip
$ uname -a
Linux 4.18.0-193.6.3.el8_2.aarch64 #1 SMP Wed Jun 10 11:10:40 UTC 2020 aarch64 aarch64 aarch64 GNU/Linux
$ curl https://nodejs.org/dist/v14.15.4/node-v14.15.4-linux-arm64.tar.gz | tar xpfz -
$ cd node-v14.15.4-linux-arm64/
$ export PATH=$PWD/bin:$PATH
$ node --version
v14.15.4
$ npm --version
6.14.10
$ mkdir tmp && cd tmp
$ npm install -g yarn
$ npm init
$ yarn add lodash
yarn add v1.22.10
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[-] 0/1Segmentation fault (core dumped)
```

