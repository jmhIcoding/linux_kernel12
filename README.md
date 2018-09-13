本项目是国科大《操作系统高级教程》的学习项目

#编译内核

##获取内核代码

```
git clone https://github.com/jmhIcoding/linux_kernel12.git
git checkout check
```
其中 linux_kernel12的目录结构为：

```
·
|
|————source_code  原始linux-0.11代码,里面有vs2015的工程文件,可以方便的看代码
|
|----source_code_att 修改后的linux-0.11代码,里面的src中bootsec.s,setup.s,head.s使用AT&T格式编写,负责生成linux内核 image
```
##安装必要软件包

```
apt-get install qemu-system-i386 make gdb 
```
##编译内核

```
cd source_code_attr
make
make start
```
即可看到从qemu启动linux-0.11

```
 . http://bochs.sourceforge.net
 . http://www.nongnu.org/vgabios

cirrus-compatible VGA is detected

QEMU BIOS - build: 05/08/09
$Revision$ $Date$
Options: apmbios pcibios eltorito rombios32

ata1 master: QEMU DVD-ROM ATAPI-4 CD-Rom/DVD-Rom

Press F12 for boot menu.

Booting from Floppy...

Loading system ...

Ram disk: 1049600 bytes, starting at 0x400000
Loading 1048576 bytes into ram disk... done
10/1024 free blocks
286/341 free inodes
3446 buffers = 3528704 bytes buffer space
Free mem: 11533312 bytes
=== Ok.
[/usr/root]# ll
```
##内核调试方法

```
cd source_code_att
```
使用带调试参数的命令从qemu启动镜像：

```
make debug
```
可以发现 qemu 处于stoped状态，等待着gdb的接入。

另外开启一个终端，输入：`gdb` 打开gdb
在gdb里面输入：

```
(gdb)file ~/home/jmh/linux_kernel12/source_code_att/src/boot/bootsect.sym  #==>载入Bootsec代码
(gdb)target remote localhost:1234
(gdb)br *0x07c00   #===>在0x07c00加上断点，当eip=0x7c00即停下,然后我们就可以开始调试bootsec了,因为bios已经将bootsec共512字节拷贝到0x7c00.
(gdb)continue 
```
运行的结果：

```
(gdb) file boot/bootsect.sym
Reading symbols from boot/bootsect.sym...done.
(gdb) pwd
Working directory /home/jmh/linux_kernel12/source_code_att/0.11.
(gdb) target remote localhost:1234
Remote debugging using localhost:1234
0x0000fff0 in ?? ()
(gdb) break 0x7c00
Function "0x7c00" not defined.
Make breakpoint pending on future shared library load? (y or [n]) n
(gdb) break *0x7c00
Breakpoint 1 at 0x7c00
(gdb) c
Continuing.

Breakpoint 1, 0x00007c00 in ?? ()
(gdb)

```
可以看到系统停在了7c00处，
我们输入`ni` 进入下一条指令：

```
(gdb) ni
48              mov     $BOOTSEG, %ax
(gdb) l
43      # ROOT_DEV:     0x000 - same type of floppy as boot.
44      #               0x301 - first partition on first drive etc
45              .equ ROOT_DEV, 0x301
46              ljmp    $BOOTSEG, $_start
47      _start:
48              mov     $BOOTSEG, %ax
49              mov     %ax, %ds
50              mov     $INITSEG, %ax
51              mov     %ax, %es
52              mov     $256, %cx

```
可以看到开头的代码。
以此即可一步步的调试，观察OS.
