# QEMU edu test

## 查看qemu支持是否支持edu设备：
qemu-system-x86_64 -device help | grep edu
```bash
cityday@ubuntu24:~/work/qemu/ubuntu-2404$ qemu-system-x86_64 -device help | grep edu
name "edu", bus PCI
name "coreduo-v1-x86_64-cpu"
name "coreduo-x86_64-cpu"
```

## 运行edu设备
> qemu-system-x86_64 -monitor stdio -m 4096 -smp 4 -cpu host -enable-kvm -drive file=ubuntu24.qcow2,if=virtio,format=qcow2 -netdev user,id=net0,hostfwd=tcp::2222-:22 -device virtio-net-pci,netdev=net0 -device edu

## 登陆geust os，password: 0
```bash
ssh -p 2222 cityday@localhost
```

## 使用lspci查看edu信息
edu device id为0x11e8。可以用lspci找到其对应的bdf：
```bash
cityday@qemuubunt2404server:~$ lspci | grep 11e8
00:04.0 Unclassified device [00ff]: Device 1234:11e8 (rev 10)
```
通过-s参数，查看指定bdf的pci设备的信息。如果不用root权限， Capabilities部分看到的将是<access denied>。：
```bash
cityday@qemuubunt2404server:~$ sudo lspci -vvv -s 00:04.0
[sudo] password for cityday: 
00:04.0 Unclassified device [00ff]: Device 1234:11e8 (rev 10)
	Subsystem: Red Hat, Inc. Device 1100
	Physical Slot: 4
	Control: I/O+ Mem+ BusMaster- SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Interrupt: pin A routed to IRQ 11
	Region 0: Memory at fea00000 (32-bit, non-prefetchable) [size=1M]
	Capabilities: [40] MSI: Enable- Count=1/1 Maskable- 64bit+
		Address: 0000000000000000  Data: 0000
```
其中，- 表示相应功能被禁用；+ 表示相应功能已启用。
* Mem+： 设备可以响应内存空间访问（其 Region 0 就是内存映射的）。
* I/O+： 设备可以响应I/O端口访问。
* BusMaster- 表示该设备的 总线主控（DMA）功能被禁用。设备当前无法主动访问主机内存。通常，操作系统驱动在初始化设备后会启用此功能。
* INTx-： 当前INTx中断线未激活（但下面显示它连接到了IRQ 11）。
* SERR+： 系统错误报告功能已启用。
* Status 中的 Cap+： 表示该设备具有能力列表（Capabilities List），用于支持MSI/MSI-X中断、电源管理等高级功能。
* Interrupt: pin A routed to IRQ 11： 该设备使用传统的 INTx 中断（引脚A），被系统路由到中断请求线 IRQ 11。
* Region 0: Memory at fea00000 (32-bit, non-prefetchable) [size=1M]：
    * 操作系统（Guest OS）为该设备分配了一段 1MB大小 的物理内存区域。起始物理地址是 0xfea00000 。该区域是非预取的，通常用于设备控制寄存器的映射
* Capabilities: [40] MSI: Enable- Count=1/1 Maskable- 64bit+
    * 设备支持 MSI（Message Signaled Interrupts） 中断机制，这是一种优于传统INTx中断的现代中断方式。
    * Enable-：MSI 功能当前被禁用，设备仍在使用传统的 INTx 中断（IRQ 11）。Guest OS驱动未初始化MSI。
    * Count=1/1：设备支持1个MSI中断向量，最多可使用1个。
    * Maskable-：该MSI中断不可屏蔽。
    * 64bit+：支持64位地址的MSI消息。
    * Address: 0000000000000000 Data: 0000：MSI地址和数据寄存器当前为0，这进一步证实MSI未配置