# QEMU build
## build qemu:
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

make -j$(nproc) 2>&1 | tee build.log

## 添加一个pcie设备
查看qemu支持哪些设备：
qemu-system-x86_64 -device help | grep pci-te*

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
  scp -v -P 2222 -r your_folder cityday@localhost:/home/cityday/work 
  scp -P 2222 your_file cityday@localhost:/home/cityday/work

## copy file or folder from guest to host os
host os:
  将guest os的cityday@localhost:/home/cityday/work/Makefile 复制到当前host os folder, 新名叫your_file
  scp -P 2222 cityday@localhost:/home/cityday/work/Makefile your_file

  将guest os的cityday@localhost:/home/cityday/work 复制到当前host os folder下
  scp -r -P 2222 cityday@localhost:/home/cityday/work  .

## 如果scp连接被refuse了, 好像还是有问题，需要再try
guest os上修改sshd配置：
sudo vim /etc/ssh/sshd_config
以下item改为yes：
PermitRootLogin yes
PasswordAuthentication yes
修改之后需要：
sudo systemctl daemon-reload
sudo systemctl restart ssh
