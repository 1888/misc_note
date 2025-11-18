# QEMU create ubuntu24.04
## install steps
1. 创建qcow2格式的虚拟磁盘，大小50G，磁盘文件名字为ubuntu24.qcow2
> qemu-img create -f qcow2 ubuntu24.qcow2 50G

2. 安装ubuntu镜像文件
> qemu-system-x86_64 -m 2048 -smp 2 -enable-kvm -boot d -hda ubuntu24.qcow2 -cdrom ubuntu-24.04-live-server-amd64.iso

其中各参数含义如下：
* -cdrom ubuntu-24.04-live-server-amd64.iso  # 指定安装镜像路径
* -boot d # 从光盘 (cdrom) 启动

3. 启动已安装的虚拟机
> qemu-system-x86_64 -m 4096 -smp 4 -cpu host -enable-kvm -drive file=ubuntu24.qcow2,if=virtio,format=qcow2 -netdev user,id=net0,hostfwd=tcp::2222-:22 -device virtio-net-pci,netdev=net0

其中各参数含义如下：
* -m 4096 # 为虚拟机分配 4096MB（4GB）内存
* -smp 4 # 配置 4 个 CPU 核心
* -cpu host # 使用与宿主机相同的 CPU 特性，提升性能
* -enable-kvm # 启用 KVM（Kernel-based Virtual Machine）加速，显著提高性能
* -drive file=ubuntu24.qcow2,if=virtio,format=qcow2 # 使用 ubuntu24.qcow2 作为虚拟磁盘文件, 使用 virtio 接口，优化磁盘 I/O 性能, 磁盘文件格式为 QCOW2（支持快照、动态扩容）
* -netdev user,id=net0,hostfwd=tcp::2222-:22 # 使用用户模式网络建立一个host network backend, id为net0, 将宿主机的2222端口映射到虚拟机的22端口（SSH）
* -device virtio-net-pci,netdev=net0 # 建立一个guest network frontend，使用 virtio 网络设备，关联到前面定义的 net0 网络backend。
 
其中网络后端和前端关系如下：虚拟机内部 → 前端设备（虚拟机内的网卡） → 后端网络栈 → 宿主机网络。 refer to [QEMU doc: Network emulation](https://qemu-project.gitlab.io/qemu/system/devices/net.html)

1. 查看是否有挂载整个50G的磁盘
```bash
cityday@qemuubunt2404server:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                         2:0    1    4K  0 disk 
sr0                        11:0    1 1024M  0 rom  
vda                       253:0    0   50G  0 disk 
├─vda1                    253:1    0    1M  0 part 
├─vda2                    253:2    0    2G  0 part /boot
└─vda3                    253:3    0   48G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   24G  0 lvm  /

```
看出vda3共48G，逻辑卷只有24G，挂在了根目录。需要扩展逻辑卷， 并调整文件系统大小以使用新的逻辑卷空间：
```bash
cityday@qemuubunt2404server:~$ sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <24.00 GiB (6143 extents) to <48.00 GiB (12287 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
cityday@qemuubunt2404server:~$ sudo resize2fs /dev/ubuntu-vg/ubuntu-lv 
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 6
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 12581888 (4k) blocks long.

cityday@qemuubunt2404server:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              392M  992K  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   48G  4.5G   41G  11% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/vda2                          2.0G  100M  1.7G   6% /boot
tmpfs                              392M   12K  392M   1% /run/user/1000
cityday@qemuubunt2404server:~$ lsblk 
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                         2:0    1    4K  0 disk 
sr0                        11:0    1 1024M  0 rom  
vda                       253:0    0   50G  0 disk 
├─vda1                    253:1    0    1M  0 part 
├─vda2                    253:2    0    2G  0 part /boot
└─vda3                    253:3    0   48G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   48G  0 lvm  /

```

sudo apt update
sudo apt upgrade
sudo apt install vim git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
sudo apt install libdw-dev

# 安装获取源码所需的工具
sudo apt install dpkg-dev

# 进入你的工作目录，然后获取当前内核版本的源码
# 假设你的内核版本是6.8.0-49-generic，对应的源码包名通常是linux-sources-6.8.0或类似
apt-get source linux-image-unsigned-$(uname -r)

cp /boot/config-$(uname -r) .config
# 更新配置以适应新源码（使用已有的 .config 文件作为基础，自动处理新版本内核新增的配置选项，对新增选项使用默认值）
make olddefconfig

make -j$(nproc) 2>&1 | tee build.log

build qemu:
  sudo apt install libslirp-dev
  sudo apt install gawk
  mkdir build
  cd build
  ../configure --target-list=x86_64-softmmu,aarch64-softmmu \
    --enable-slirp \
    --enable-system \
    --enable-kvm \
    --enable-virtfs \
    --enable-vnc \
    --enable-tools

make -j$(nproc)


make -j$(nproc) 2>&1 | tee build.log

## ssh 登陆
## guest os
```bash
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
```
## host os
```bash
ssh -p 2222 cityday@localhost
```

## copy file or folder from host to guest os
host os:
  scp -r your_folder -P 2222 cityday@localhost:/home/cityday/work
  scp your_file -P 2222 cityday@localhost:/home/cityday/work

## copy file or folder from guest to host os
host os:
  scp -r -P 2222 cityday@localhost:/home/cityday/work your_folder
  scp -P 2222 cityday@localhost:/home/cityday/work your_file

## 如果scp连接被refuse了, 好像还是有问题，需要再try
guest os上修改sshd配置：
sudo vim /etc/ssh/sshd_config
以下item改为yes：
PermitRootLogin yes
PasswordAuthentication yes
修改之后需要：
sudo systemctl daemon-reload
sudo systemctl restart ssh
