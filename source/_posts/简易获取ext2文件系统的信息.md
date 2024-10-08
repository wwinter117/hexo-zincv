---
title: 简易获取ext2文件系统的信息
tags:
  - 操作系统
  - 文件系统
  - ext2
date: 2023-4-26 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/redis-5fb76d3ec9fd434fe46a579d9c1c8a83.jpeg)

我的仓库地址：https://github.com/wwinter117/fetchext2.git

```
parallels@ubuntu:~/Dev/github/fetchext2$ ./fetchext2 vdisk-1k-100m 
--------------------------- [B0]BootBlock
...
--------------------------- [B1]SuperBlock
s_magic                        = 61267
s_inodes_count                 = 25584
s_blocks_count                 = 102400
s_r_blocks_count               = 5120
s_free_inodes_count            = 25573
s_free_blocks_count            = 94415
s_first_data_block             = 1
s_log_block_size               = 0
s_blocks_per_group             = 8192
s_inodes_per_group             = 1968
s_max_mnt_count                = 0
s_mtime                        = Thu Jan  1 08:00:00 1970
s_wtime                        = Sun Aug 18 12:59:11 2024
block size                     = 1024
inode size                     = 256

--------------------------- [B2]Group Desc-0
bg_block_bitmap                = 259
bg_inode_bitmap                = 260
bg_inode_table                 = 261
bg_free_blocks_count           = 7426
bg_free_inodes_count           = 1957
bg_used_dirs_count             = 2
bg_pad                         = 0

--------------------------- [B259]BMAP
0   : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
64  : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
128 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
192 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
256 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
320 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
384 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
448 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
512 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
576 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
640 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111 
704 : 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111100 
768 : ...

--------------------------- [B260]IMAP
0   : 11111111 11100000 00000000 00000000 00000000 00000000 00000000 00000000 
64  : ...

--------------------------- [B261]DIRS
drwxr-xr-x
i_uid                          = 0
i_gid                          = 0
i_size                         = 1024
i_ctime                        = Sun Aug 18 12:59:11 2024
i_links_count                  = 3
i_block[0] = 753
--------------------------- [B753]ENTRYS
type inode rec_len name_len name
d    2     12      1        .                   
d    2     12      2        ..                  
d    11    1000    10       lost+found          


--------------------------- [B2]Group Desc-1(not used)
--------------------------- [B2]Group Desc-2(not used)
--------------------------- [B2]Group Desc-3(not used)
--------------------------- [B2]Group Desc-4(not used)
--------------------------- [B2]Group Desc-5(not used)
--------------------------- [B2]Group Desc-6(not used)
--------------------------- [B2]Group Desc-7(not used)
--------------------------- [B2]Group Desc-8(not used)
--------------------------- [B2]Group Desc-9(not used)
--------------------------- [B2]Group Desc-10(not used)
--------------------------- [B2]Group Desc-11(not used)
--------------------------- [B2]Group Desc-12(not used)
```