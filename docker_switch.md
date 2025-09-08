# 将 QCOW2 镜像转换为 Docker 容器的完整指南

## Table of contents
1. Pre-requisite
2. Create a Tar ball
3. Run Docker Import
4. Docker Run
5. Optimize your Image Size

> This is the recommended workflow for creating your own Docker image for your application using a qcow2 image

---

## 1. Pre-requisite
Install the following applications before running the below commands

```bash
yum install libguestfs libguestfs-tools
yum install qemu-kvm libvirt
systemctl start libvirtd
2. Create a Tar ball
You can convert your qcow2 images to a docker image by creating a tar ball.

Follow the below steps to create a tar ball using your qcow2 image:

You need to identify the device to which your root is mapped on the QCOW2 image

bash
guestfish add /apps/docker/azapp-test-ol7-20230126.qcow2 : run : list-filesystems
# 输出示例:
# /dev/sda1: vfat
# /dev/sda2: ext4
# /dev/vg00/opt: ext4
# /dev/vg00/root: ext4
# /dev/vg00/swap: swap
# /dev/vg00/tmp: ext4
# /dev/vg00/var: ext4
# /dev/vg00/var_log: ext4
# /dev/vg00/var_log_audit: ext4
# /dev/vg00/var_log_audit.d: ext4
# /dev/vg00/var_tmp: ext4
Use /dev/vg00/root as it is the device mount that contains the root contents.

bash
mkdir /mnt/
guestmount -a /apps/docker/azapp-test-ol7-20230126.qcow2 -m /dev/vg00/root /mnt/
cd /mnt/

# 查看挂载内容:
ls -l /mnt
# 输出示例:
# total 32
# lrwxrwxrwx.   1 root root    7 Oct 12  2021 bin -> usr/bin
# dr-xr-xr-x.   4 root root 4096 Dec 13  2021 boot
# drwxr-xr-x.   2 root root    6 Oct 12  2021 dev
# drwxr-xr-x. 103 root root 8192 Jan 11  2022 etc
# drwxr-xr-x.   5 root root   46 Dec 13  2021 home
# lrwxrwxrwx.   1 root root    7 Oct 12  2021 lib -> usr/lib
# lrwxrwxrwx.   1 root root    9 Oct 12  2021 lib64 -> usr/lib64
# drwxr-xr-x.   2 root root    6 Apr 10  2018 media
# drwxr-xr-x.   2 root root    6 Dec 13  2021 mnt
# drwx------.   2 root root    6 Jan 11  2022 oem
# drwxr-xr-x.  10 root root 4096 Dec 13  2021 opt
# drwxr-xr-x.   2 root root    6 Oct 12  2021 proc
# dr-xr-x---.   8 root root 4096 Jan 11  2022 root
# drwxr-xr-x.   2 root root    6 Oct 12  2021 run
# lrwxrwxrwx.   1 root root    8 Oct 12  2021 sbin -> usr/sbin
# drwxr-xr-x.   2 root root    6 Apr 10  2018 srv
# drwxr-xr-x.   2 root root    6 Oct 12  2021 sys
# drwxrwxrwt.   7 root root 4096 Jan 11  2022 tmp
# drwxr-xr-x.  13 root root 4096 Oct 12  2021 usr
# drwxr-xr-x.   3 root root   33 Jan 11  2022 var

# 创建tar包:
tar -czf /apps/docker/azapp-test-ol7-20230126-qcow2.tar.gz .

# 验证tar包:
ls -lh /apps/docker/azapp-test-ol7-20230126-qcow2.tar.gz
# 输出示例:
# -rw------- 1 root root 1.5G Jan 25 16:23 /apps/docker/azapp-test-ol7-20230126-qcow2.tar.gz
3. Run Docker Import
Once you have your qcow2 tar ball created, ran the docker import command with "EXPOSE 22" to ensure you can SSH into the container once the image is created

bash
cat /apps/docker/azapp-test-ol7-20230126-qcow2.tar.gz | sudo docker import -c "EXPOSE 22" - azapp-test-ol7-20230126-qcow2:0.0.1
# 输出示例:
# sha256:8bbcaa7a34e50893de1b804e48dc04733ad5ff7d91aba890644a8c12e2798070

# 验证镜像:
docker images
# 输出示例:
# REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
# azapp-test-ol7-20230126-qcow2                  0.0.1               8bbcaa7a34e5        24 seconds ago      4.08GB
# azapp-test-ol7-20230126                        0.0.1               569ef6347563        4 hours ago         2.66GB
# azapp-oel7-test-image                          0.0.2               5f60b5e9fbd0        13 days ago         3.24GB
# azapp-oel6-test-image                          0.0.2               13dad8fe7b85        2 weeks ago         2.3GB
4. Docker Run
The docker image has been created successfully now you can use docker run command to get inside the container

bash
docker run -it -v /git/:/home/rpmbuild/git -v /var/run/docker.sock:/var/run/docker.sock azapp-test-ol7-20230126-qcow2:0.0.1 bash

# 容器内查看文件系统:
ls -ltr
# 输出示例:
# total 4
# drwxr-xr-x   2 root root    6 Apr 11  2018 srv
# drwxr-xr-x   2 root root    6 Apr 11  2018 media
# drwxr-xr-x   2 root root    6 Oct 13  2021 run
# drwxr-xr-x  13 root root  155 Oct 13  2021 usr
# lrwxrwxrwx   1 root root    8 Oct 13  2021 sbin -> usr/sbin
# lrwxrwxrwx   1 root root    9 Oct 13  2021 lib64 -> usr/lib64
# lrwxrwxrwx   1 root root    7 Oct 13  2021 lib -> usr/lib
# lrwxrwxrwx   1 root root    7 Oct 13  2021 bin -> usr/bin
# dr-xr-xr-x   4 root root 4096 Dec 13  2021 boot
# drwxr-xr-x  10 root root  133 Dec 13  2021 opt
# drwxr-xr-x   2 root root    6 Dec 13  2021 mnt
# dr-xr-x---   1 root root   20 Jan 11  2022 root
# drwx------   2 root root    6 Jan 11  2022 oem
# drwxrwxrwt   7 root root  138 Jan 11  2022 tmp
# drwxr-xr-x   1 root root   66 Jan 25 23:34 etc
# dr-xr-xr-x 434 root root    0 Jan 25 23:34 proc
# dr-xr-xr-x  13 root root    0 Jan 25 23:34 sys
# drwxr-xr-x   1 root root   17 Jan 25 23:34 var
# drwxr-xr-x   1 root root   22 Jan 25 23:34 home
# drwxr-xr-x   5 root root  360 Jan 25 23:34 dev
5. Optimize your Image Size
Docker images can get very large very soon, and that will become a problem when pulling over the network or pushing on devices with limited storage. Here are a some suggestions on how to keep your image size small:

Reduce the number of RUN commands in your Docker. Each command adds a layer to the image, so consolidating the number of RUN can reduce the number of layers in the final image.

Note: Docker image layers are designed to be reusable, and will not be pushed or pulled if they've not changed.

Use --no-install-recommends when installing packages with apt-get install to disable installations of optional packages and save some disk space.

Remove tar balls or other archive files that were copied during the installation. Each layer is added on top of the others, so files that were not removed in a given RUN step will be present in the final image even if they are removed in a later RUN step.

Also, clean up your package lists that are downloaded with apt-get update by removing /var/lib/apt/lists/* in the same RUN step.

Create separate images for development and production. Production images should not include all of the libraries and dependencies pulled in by the build.

Use multi-stage builds (see Docker docs) and push only your prod image.

