# LDD3-Chapter8 Allocating Memory
## 关于kmalloc的重点
* kmalloc 函数原型为 void *kmalloc(size_t size, gfp_t flags), 声明在<linux/slab.h>。
* 用于向kernel申请一片size大小的内存，如果申请成功，返回值为首地址。如果申请失败，
* kmalloc能够分配的最大size是有限的。
