# uefi code flow
记录ArmVirtPkg/ArmVirtQemu.dsc的flow
## sec
qemu官方文档：https://qemu-project.gitlab.io/qemu/system/arm/virt.html 中有定义arm virt board的裸机信息：
* Flash memory starts at address 0x0000_0000
* RAM starts at 0x4000_0000

arm刚上电的时候，mmu没有开启，虚拟地址等于物理地址。第一条指令从0x0开始执行。从ArmVirtPkg/ArmVirtQemu.fdf中可以看到，uefi在0x0的地址直接存放了一个跳转指令，跳转到0x1000执行。
```bash
#
# UEFI has trouble dealing with FVs that reside at physical address 0x0.
# So instead, put a hardcoded 'jump to 0x1000' at offset 0x0, and put the
# real FV at offset 0x1000
#
0x00000000|0x00001000
DATA = {
!if $(ARCH) == AARCH64
  0x00, 0x04, 0x00, 0x14   # 'b 0x1000' in AArch64 ASM
!else
  0xfe, 0x03, 0x00, 0xea   # 'b 0x1000' in AArch32 ASM
!endif
}

#
# The FV RegionType is a pointer to either one of the unique FV names that are defined in the [FV] section.
# 这里FV指向了[FV.FVMAIN_COMPACT]定义的FV. 这里表示：
# 在Flash的物理地址0x00001000处，分配$(FVMAIN_COMPACT_SIZE)大小的空间给名为FVMAIN_COMPACT的固件卷，
# 并将这个固件卷的基地址和大小分别存储到PCD变量PcdFvBaseAddress和PcdFvSize中。
#
0x00001000|$(FVMAIN_COMPACT_SIZE)
gArmTokenSpaceGuid.PcdFvBaseAddress|gArmTokenSpaceGuid.PcdFvSize
FV = FVMAIN_COMPACT
```
同时，也可以看到，从0x1000到FVMAIN_COMPACT_SIZE，非配给了FVMAIN_COMPACT的固件卷。

通过gdb可以看到0x1000处存的也是一条跳转指令，条转到0x5570, 即_ModuleEntryPoint的地址。
```bash
(gdb) info registers pc
pc             0x0                 0x0
(gdb) x/i $pc
=> 0x0:	b	0x1000
(gdb) stepi
0x0000000000001000 in ?? ()
(gdb) info registers pc
pc             0x1000              0x1000
(gdb) x/i $pc
=> 0x1000:	b	0x5570 <_ModuleEntryPoint>
```
这里是arm64,_ModuleEntryPoint定义在ArmPlatformPkg/Sec/AArch64/ModuleEntryPoint.S。