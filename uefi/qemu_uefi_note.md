# uefi note
## qemu command for uefi
### work fine, not hello word.
 qemu-system-x86_64 -pflash hda-contents/bios.bin \
 -m 4096 -smp 4 \
 -drive file=uefi_test_disk.img,format=qcow2 \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0

### work fine
qemu-system-x86_64 -pflash hda-contents/bios.bin \
 -m 4096 -smp 4 \
 -hda fat:rw:hda-contents \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0

### arm
qemu-system-aarch64 \
 -m 4096 \
 -cpu cortex-a72 \
 -smp 4 \
 -M virt \
 -bios /home/cityday/work/qemu/1888_qemu/qemu-study/arm-uefi/QEMU_EFI.fd \
 -hda fat:rw:arm-uefi \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0

### arm pflash
qemu-system-aarch64 \
 -m 4096 \
 -cpu cortex-a72 \
 -smp 4 \
 -M virt \
 -drive if=pflash,format=raw,unit=0,readonly=on,file=/home/cityday/work/qemu/1888_qemu/qemu-study/arm-uefi/QEMU_EFI.fd \
 -drive if=pflash,format=raw,unit=1,file=/home/cityday/work/qemu/1888_qemu/qemu-study/arm-uefi/QEMU_VARS.fd \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0


### build arm uefi
sudo apt install gcc-aarch64-linux-gnu

cd edk2_stable_202408/
export WORKSPACE=$PWD
export GCC5_AARCH64_PREFIX=/usr/bin/aarch64-linux-gnu-
source edksetup.sh 
build -a AARCH64 -t GCC5 -p ArmVirtPkg/ArmVirtQemu.dsc -b DEBUG 2>&1 | tee build.log

ls
ls Build/
ls ArmVirtPkg/ -l
ls Build/ArmVirtQemu-AARCH64/DEBUG_GCC5/FV/

### arm framebuffer display, need use "gvncviewer localhost" to login. press F2 to interrupt boot flow.
qemu-system-aarch64 \
 -m 4096 \
 -cpu cortex-a72 \
 -smp 4 \
 -M virt \
 -vnc :0 \
 -device ramfb \
 -device usb-ehci \
 -device usb-mouse \
 -device usb-kbd \
 -bios /home/cityday/work/qemu/1888_qemu/qemu-study/arm-uefi/QEMU_EFI.fd \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0 

qemu-system-aarch64 \
 -m 4096 \
 -cpu cortex-a72 \
 -smp 4 \
 -M virt \
 -vnc :0 \
 -device ramfb \
 -device usb-ehci \
 -device usb-mouse \
 -device usb-kbd \
 -bios /home/cityday/work/qemu/1888_qemu/qemu-study/arm-uefi/QEMU_EFI.fd \
 -chardev stdio,id=char0,logfile=./serial.log,signal=off \
 -serial chardev:char0 \
 -s -S

### gdb cmd
gdb-multiarch
set architecture aarch64
target remote :1234
layout src
add-symbol-file /home/cityday/work/uefi/edk2/edk2_stable_202408/Build/ArmVirtQemu-AARCH64/DEBUG_GCC5/AARCH64/ArmPlatformPkg/Sec/Sec/DEBUG/Sec.dll 0x2000

(gdb) info registers sp      # 栈指针
(gdb) info registers x0      # 通用寄存器 x0
(gdb) info registers cpsr    # 程序状态寄存器
(gdb) info registers pc
pc             0x0                 0x0

#### 反汇编当前程序计数器指向的内存地址处的指令
命令分解
    x：examine（检查）命令，用于显示内存内容
    /i：格式化选项，表示以指令（instruction）格式显示
    /10i：格式化选项，表示以指令（instruction）格式显示10条指令
    $pc：程序计数器（Program Counter）寄存器
(gdb) x/i $pc
(gdb) x/10i $pc

反汇编pc前5条指令，后10条指令
(gdb) disassemble $pc-5*4, $pc+10*4

### 单步执行
* step (C源码级单步) 执行到下一行C源代码（可能包含多条机器指令），如果遇到函数调用，会进入到函数内部。
* stepi (指令级单步) 执行一条机器指令，如果遇到函数调用，会进入到函数内部。
* next (C源码级单步) 执行到下一行C源代码（可能包含多条机器指令），如果遇到函数调用，不会进入到函数内部。
* nexti (指令级单步) 执行一条机器指令，如果遇到函数调用，不会进入到函数内部。
* 在汇编部分使用stepi/nexti，在C代码部分使用step/next
### SEC 阶段
(gdb) hbreak _ModuleEntryPoint

### ASM_FUNC 宏用于定义可以被 C 代码调用的汇编函数
ASM_FUNC(_ModuleEntryPoint)即定义了_ModuleEntryPoint函数
### ASM_PFX 宏用于为汇编符号添加正确的前缀，以匹配 C 编译器的名称修饰约定，确保汇编代码和 C 代码能够正确链接。


# PEI 阶段  
(gdb) hbreak PeiCore
# DXE 阶段
(gdb) hbreak DxeCore


## UEFI shell command
```bash
# exit form UEFI shell
reset -s
# switch to file system 0
fs0:
# backspace, delete the wrong input
CTRL+H
(if keyboard backspace is not working in UEFI shell)
```

## linux debug command
```bash
# view serial.log 
less -R serial.log
```


## uefi driver understanding
| items | linux driver     | uefi driver      |
| :---- | :----:           | -----:           |
| 单元格 |  module          | Image            |
| 单元格 |  device          | ControllerHandle |
| 单元格 |  device driver   | ImageHandle      |
| 单元格 |  file operation  | Protocol         |
| 单元格 |  match           | supported        |
| 单元格 |  probe           | start            |
| 单元格 |  remove          | stop             |


## PCD (Platform Configuration Database)
a mechanism for managing firmware settings and features that can be configured statically at build time or dynamically at runtime, allowing for platform customization, feature toggling, and user configuration within the UEFI firmware.

**PCDs are defined in the .dec (Declaration) file** and can be of different access types, such as FeatureFlag, FixedAtBuild, PatchableInModule, Dynamic, and DynamicEx, with their values specified 
```bash
  ## This flag is used to control build time optimization based on debug print level.
  #  Its default value is 0xFFFFFFFF to expose all debug print level.
  #  BIT0  - Initialization message.<BR>
  #  BIT1  - Warning message.<BR>
  #  BIT2  - Load Event message.<BR>
  #  BIT3  - File System message.<BR>
  #  BIT4  - Allocate or Free Pool message.<BR>
  #  BIT5  - Allocate or Free Page message.<BR>
  #  BIT6  - Information message.<BR>
  #  BIT7  - Dispatcher message.<BR>
  #  BIT8  - Variable message.<BR>
  #  BIT10 - Boot Manager message.<BR>
  #  BIT12 - BlockIo Driver message.<BR>
  #  BIT14 - Network Driver message.<BR>
  #  BIT16 - UNDI Driver message.<BR>
  #  BIT17 - LoadFile message.<BR>
  #  BIT19 - Event message.<BR>
  #  BIT20 - Global Coherency Database changes message.<BR>
  #  BIT21 - Memory range cachability changes message.<BR>
  #  BIT22 - Detailed debug message.<BR>
  #  BIT23 - Manageability debug message.<BR>
  #  BIT31 - Error message.<BR>
  # @Prompt Fixed Debug Message Print Level.
  # format: <PcdTokenSpaceGuidCName>.<PcdCName> | <DefaultValue> | <DatumType> | <Token>
  #         <Token> 一个唯一的16进制数字，通常从 0x00000001 开始递增。 这个Token在该PcdTokenSpaceGuidCName内必须是唯一的。
  #                 它和Token Space GUID一起，在编译时被用来唯一地标识这个PCD。
  gEfiMdePkgTokenSpaceGuid.PcdFixedDebugPrintErrorLevel|0xFFFFFFFF|UINT32|0x30001016
```
**overridden in the .dsc (Dsc) and .fdf (Flash Definition) files**
```bash
    gEfiMdePkgTokenSpaceGuid.PcdFixedDebugPrintErrorLevel   | 0x80000047
    gEfiMdePkgTokenSpaceGuid.PcdDebugPropertyMask           | 0x27
```
**PcdFixedDebugPrintErrorLevel is defined in MdePkg/MdePkg.dec with default value 0xFFFFFFFF.
if there is no dsc file to override it, all debug print levels are enabled.**

0x914AEBE7, 0x4635, 0x459b, { 0xAA, 0x1C, 0x11, 0xE2, 0x19, 0xB0, 0x3A, 0x10 
dmpstore -guid 914AEBE7-4635-459b-AA1C-11E219B03A10 PcdFixedDebugPrintErrorLevel



/home/cityday/work/uefi/edk2/edk2_stable_202408/ArmVirtPkg/Library/DebugLibFdtPL011Uart/DebugLib.c
/**
  Returns TRUE if any one of the bit is set both in ErrorLevel and PcdFixedDebugPrintErrorLevel.

  This function compares the bit mask of ErrorLevel and PcdFixedDebugPrintErrorLevel.

  @retval  TRUE    Current ErrorLevel is supported.
  @retval  FALSE   Current ErrorLevel is not supported.

**/
BOOLEAN
EFIAPI
DebugPrintLevelEnabled (
  IN  CONST UINTN  ErrorLevel
  )
{
  return (BOOLEAN)((ErrorLevel & PcdGet32 (PcdFixedDebugPrintErrorLevel)) != 0);
}

/**
  Returns the debug print error level mask for the current module.

  @return  Debug print error level mask for the current module.

**/
UINT32
EFIAPI
GetDebugPrintErrorLevel (
  VOID
  )
{
  //
  // Retrieve the current debug print error level mask from PcdDebugPrintErrorLevel.
  //
  return PcdGet32 (PcdDebugPrintErrorLevel);
}