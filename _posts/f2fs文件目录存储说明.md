title: f2fs文件目录存储结构
tags:
	- f2fs
date: 2022/07/13
categories: f2fs
---

# f2f2文件目录存储结构

当前目录结构：

```
$ tree mnt/f2fs
mnt/f2fs
├── dir
│   ├── 6.29.cfx.pcap
│   └── testone
└── test

1 directory, 3 files
```

文件内容

```
mnt/f2fs/test:
hello world
mnt/f2fs/dir/testone
hellow zt
mnt/f2fs/dir/6.29.cfx.pcap 一个27MB的数据包文件
```

### NAT信息

查看inode信息

```
$ sudo dump.f2fs -n 0~-1 /dev/loop0
$ nvim dump_nat
  1 nid:    3   ino:    3   offset:    0    blkaddr:      4101  pack:2
  2 nid:    4   ino:    4   offset:    0    blkaddr:      4105  pack:2
  3 nid:    5   ino:    5   offset:    0    blkaddr:      4609  pack:2
  4 nid:    6   ino:    6   offset:    0    blkaddr:      4610  pack:2
  5 nid:    7   ino:    7   offset:    0    blkaddr:      4611  pack:2
  6 nid:    8   ino:    7   offset:    1    blkaddr:      4612  pack:2
  7 nid:    9   ino:    7   offset:    2    blkaddr:      4613  pack:2
  8 nid:   10   ino:    7   offset:    3    blkaddr:      5120  pack:2
  9 nid:   11   ino:    7   offset:    4    blkaddr:      4614  pack:2
 10 nid:   12   ino:    7   offset:    5    blkaddr:      4615  pack:2
 11 nid:   13   ino:    7   offset:    6    blkaddr:      4616  pack:2
 12 nid:   14   ino:    7   offset:    7    blkaddr:      4617  pack:2
 13 nid:   15   ino:    7   offset:    8    blkaddr:      4618  pack:2
 14 nid:   16   ino:    7   offset:    9    blkaddr:      4619  pack:2
```

从super block中可以得到信息，NAT的地址0xa00000，CP中的NAT journal地址在0x201000
00000400  10 20 f5 f2 01 00 0f 00  09 00 00 00 03 00 00 00  |. ..............|
00000410  0c 00 00 00 09 00 00 00  01 00 00 00 01 00 00 00  |................|
00000420  00 00 00 00 00 64 00 00  00 00 00 00 2a 00 00 00  |.....d......*...|
00000430  31 00 00 00 02 00 00 00  02 00 00 00 02 00 00 00  |1...............|
00000440  01 00 00 00 2a 00 00 00  00 02 00 00 00 02 00 00  |....*...........|
00000450  00 06 00 00 <font color=red>**00 0a 00 00**</font>  00 0e 00 00 00 10 00 00  |................|
00000460  03 00 00 00 01 00 00 00  02 00 00 00 83 8f 92 2c  |...............,|
00000470  60 18 4d 2e 85 c3 d4 8e  56 29 d9 dc 00 00 00 00  |`.M.....V)......|
00000480  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

查看NAT，红色是node的序号，蓝色是node的地址，绿色是版本号，接下来是下一个节点。

在这里3号node是根目录，4号node是dir目录，5号node是test文件，6号文件是testone，7号文件是6.29.cap

00a00000  00 00 00 00 00 00 00 00  00 00 01 00 00 00 01 00  |................|
00a00010  00 00 00 02 00 00 00 01  00 00 00 <font color=green>**00**</font> <font color=red>**03 00 00 00**</font>  |................|
00a00020  <font color=blue>**05 10 00 00**</font> 00 04 00 00  00 08 10 00 00 00 05 00  |................|
00a00030  00 00 01 12 00 00 00 06  00 00 00 02 12 00 00 00  |................|
00a00040  07 00 00 00 03 12 00 00  00 07 00 00 00 04 12 00  |................|
00a00050  00 00 07 00 00 00 05 12  00 00 00 07 00 00 00 00  |................|
00a00060  14 00 00 00 07 00 00 00  06 12 00 00 00 07 00 00  |................|
00a00070  00 07 12 00 00 00 07 00  00 00 08 12 00 00 00 07  |................|
00a00080  00 00 00 09 12 00 00 00  07 00 00 00 0a 12 00 00  |................|
00a00090  00 07 00 00 00 0b 12 00  00 00 00 00 00 00 00 00  |................|
00a000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|

* **当文件系统修改比较小时，会将修改数据保存在nat journal中，而不直接写入到SIT和NAT中，减少读写次数。**

### node   inode  文件存储

从下图的f2fs_node数据结构可以看出，f2fs_node分为三种inode、direct_node、indirect。第一种，node存储指向data的指针；第二种，direct_node存储指向node的指针，然后再由node存储指向data的指针

```
  1081 struct f2fs_node {
  1082     /* can be one of three types: inode, direct, and indirect types */
  1083     union {
  1084         struct f2fs_inode i;
  1085         struct direct_node dn;
  1086         struct indirect_node in;
  1087     };
  1088     struct node_footer footer;
  1089 };
```

```
   994 struct f2fs_inode {
   995     __le16 i_mode;          /* file mode */
   996     __u8 i_advise;          /* file hints */
   997     __u8 i_inline;          /* file inline flags */
   998     __le32 i_uid;           /* user ID */
   999     __le32 i_gid;           /* group ID */
  1000     __le32 i_links;         /* links count */
  1001     __le64 i_size;          /* file size in bytes */
  1002     __le64 i_blocks;        /* file size in blocks */
  1003     __le64 i_atime;         /* access time */
  1004     __le64 i_ctime;         /* change time */
  1005     __le64 i_mtime;         /* modification time */
  1006     __le32 i_atime_nsec;        /* access time in nano scale */
  1007     __le32 i_ctime_nsec;        /* change time in nano scale */
  1008     __le32 i_mtime_nsec;        /* modification time in nano scale */
  1009     __le32 i_generation;        /* file version (for NFS) */
  1010     union {
  1011         __le32 i_current_depth; /* only for directory depth */
  1012         __le16 i_gc_failures;   /*
  1013                      * # of gc failures on pinned file.
  1014                      * only for regular files.
  1015                      */
  1016     };
  1017     __le32 i_xattr_nid;     /* nid to save xattr */
  1018     __le32 i_flags;         /* file attributes */
  1019     __le32 i_pino;          /* parent inode number */
  1020     __le32 i_namelen;       /* file name length */
  1021     __u8 i_name[F2FS_NAME_LEN]; /* file name for SPOR */
  1022     __u8 i_dir_level;       /* dentry_level for large dir */
  1023
  1024     struct f2fs_extent i_ext __attribute__((packed));   /* caching a largest extent */
  1025
  1026     union {
  1027         struct {
  1028             __le16 i_extra_isize;   /* extra inode attribute size */
  1029             __le16 i_inline_xattr_size; /* inline xattr size, unit: 4 bytes */
  1030             __le32 i_projid;    /* project id */
  1031             __le32 i_inode_checksum;/* inode meta checksum */
  1032             __le64 i_crtime;    /* creation time */
  1033             __le32 i_crtime_nsec;   /* creation time in nano scale */
  1034             __le64 i_compr_blocks;  /* # of compressed blocks */
  1035             __u8 i_compress_algrithm;   /* compress algrithm */
  1036             __u8 i_log_cluster_size;    /* log of cluster size */
  1037             __le16 i_padding;       /* padding */
  1038             __le32 i_extra_end[0];  /* for attribute size calculation */
  1039         } __attribute__((packed));
  1040         __le32 i_addr[DEF_ADDRS_PER_INODE]; /* Pointers to data blocks */
  1041     };
  1042     __le32 i_nid[5];        /* direct(2), indirect(2),
  1043                         double_indirect(1) node id */
  1044 };
```

```
  1071 struct node_footer {
  1072     __le32 nid;     /* node id */
  1073     __le32 ino;     /* inode nunmber */
  1074     __le32 flag;        /* include cold/fsync/dentry marks and offset */
  1075     __le64 cp_ver __attribute__((packed));      /* checkpoint version */
  1076     __le32 next_blkaddr;    /* next node page block address */
  1077 };
```

红色部分对应inode的i_name[F2FS_NAME_LEN]和i_dir_level，占256字节；蓝色部分对应inode的f2fs_extent i_ext，占12个字节。黑色部分对应inode的__len32  i_addr[DEF_ADDRS_PER_INODE]的部分。

\_\_attribute\_\_((paecked)) : 表示编译时不按照内存对齐的方式，以节约内存

0120c000  a4 81 00 01 00 00 00 00  00 00 00 00 01 00 00 00  |................|
0120c010  06 28 18 02 00 00 00 00  8d 21 00 00 00 00 00 00  |.(.......!......|
0120c020  06 48 c4 62 00 00 00 00  86 32 c4 62 00 00 00 00  |.H.b.....2.b....|
0120c030  86 32 c4 62 00 00 00 00  cc 71 67 0c 33 04 62 2c  |.2.b.....qg.3.b,|
0120c040  33 04 62 2c 9c 01 92 e4  00 00 00 00 00 00 00 00  |3.b,............|
0120c050  00 00 00 00 04 00 00 00  0d 00 00 00 <font color=red>36 2e 32 39</font>  |............6.29|
0120c060  <font color=red>2e 63 66 78 2e 70 63 61  70 00 00 00 00 00 00 00</font>  |.cfx.pcap.......|
0120c070  <font color=red>00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00</font>  |................|
*
0120c150  <font color=red>00 00 00 00 00 00 00 00  00 00 00 00</font> <font color=blue>00 0c 00 00</font>  |................|
0120c160  <font color=blue>00 24 00 00 00 14 00 00</font>  **00 38 00 00 01 38 00 00**  |.$.......8...8..|
0120c170  **02 38 00 00 03 38 00 00  04 38 00 00 05 38 00 00**  |.8...8...8...8..|
0120c180  **06 38 00 00 07 38 00 00  08 38 00 00 09 38 00 00**  |.8...8...8...8..|
0120c190  0a 38 00 00 0b 38 00 00  0c 38 00 00 0d 38 00 00  |.8...8...8...8..|
0120c1a0  0e 38 00 00 0f 38 00 00  10 38 00 00 11 38 00 00  |.8...8...8...8..|
0120c1b0  12 38 00 00 13 38 00 00  14 38 00 00 15 38 00 00  |.8...8...8...8..|
0120c1c0  16 38 00 00 17 38 00 00  18 38 00 00 19 38 00 00  |.8...8...8...8..|
0120c1d0  1a 38 00 00 1b 38 00 00  1c 38 00 00 1d 38 00 00  |.8...8...8...8..|
0120c1e0  1e 38 00 00 1f 38 00 00  20 38 00 00 21 38 00 00  |.8...8.. 8..!8..|
0120c1f0  22 38 00 00 23 38 00 00  24 38 00 00 25 38 00 00  |"8..#8..$8..%8..|
0120c200  26 38 00 00 27 38 00 00  28 38 00 00 29 38 00 00  |&8..'8..(8..)8..|

可以看一看0x3801000的内存，是6.29.cfx.pcap的文件数据

```
03801000  00 01 50 64 2b 95 5d 3d  c0 a8 0c 02 00 00 00 00  |..Pd+.]=........|
03801010  00 00 c0 a8 0c 80 00 00  00 00 00 00 00 00 00 00  |................|
03801020  00 00 00 00 00 00 00 00  22 1d bc 62 f5 bf 03 00  |........"..b....|
03801030  3c 00 00 00 3c 00 00 00  ff ff ff ff ff ff 2c 97  |<...<.........,.|
03801040  b1 c3 58 10 99 98 00 01  00 00 00 1d 00 00 00 04  |..X.............|
03801050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
03801070  00 00 00 00 22 1d bc 62  8e 04 04 00 3c 00 00 00  |...."..b....<...|
03801080  3c 00 00 00 ff ff ff ff  ff ff 00 22 46 2c 42 58  |<.........."F,BX|
03801090  08 06 00 01 08 00 06 04  00 01 00 22 46 2c 42 58  |..........."F,BX|
038010a0  c0 a8 0a 59 00 00 00 00  00 00 c0 a8 0a 8a 00 00  |...Y............|
038010b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
038010c0  22 1d bc 62 17 07 04 00  3c 00 00 00 3c 00 00 00  |"..b....<...<...|
038010d0  ff ff ff ff ff ff 50 64  2b 95 5d 3d 08 06 00 01  |......Pd+.]=....|
038010e0  08 00 06 04 00 01 50 64  2b 95 5d 3d c0 a8 0c 02  |......Pd+.]=....|
```

可以说，6.29.cfx的文件数据的索引保存在node7 ~ 16中。每一个node的offset  0x16c开始是文件数据的block地址。同时每个node的结尾都标注了该node的inode，都是inode7，同时标注offset，为还原文件做好了工作。



### 目录的结构

```
  1325 /* One directory entry slot representing F2FS_SLOT_LEN-sized file name */
  1326 struct f2fs_dir_entry {
  1327     __le32 hash_code;   /* hash code of file name */
  1328     __le32 ino;     /* inode number */
  1329     __le16 name_len;    /* lengh of file name */
  1330     __u8 file_type;     /* file type */
  1331 } __attribute__((packed));
  1332
  1333 static_assert(sizeof(struct f2fs_dir_entry) == 11, "");
  1334
  1335 /* 4KB-sized directory entry block */
  1336 struct f2fs_dentry_block {
  1337     /* validity bitmap for directory entries in each block */
  1338     /* (214 + 8 - 1) / 8 = 27 */
  1339     __u8 dentry_bitmap[SIZE_OF_DENTRY_BITMAP];
  1340     /* pagsize - (( size_of_dir_entry + f2fs_slot_len)*nr_entry_in_block + size_of_dir_entry_bitmap) */
  1341     /* 4096 - (( 11 + 8) * 214 + 27) = 3 */
  1342     __u8 reserved[SIZE_OF_RESERVED];
  1343     /* f2fs_dir_entry[214] len(32+32+16+8) */
  1344     struct f2fs_dir_entry dentry[NR_DENTRY_IN_BLOCK];
  1345     /* filename[214][8] */
  1346     __u8 filename[NR_DENTRY_IN_BLOCK][F2FS_SLOT_LEN];
  1347 };
```

目前目录对应的nid：

* / 根目录对应：nid3，对应的地址0x01005000。在0x01005168开始是data block pointer，指向的地址是0x1602000。红色部分是f2fs_dir_entry的hash_code，蓝色是ino(inode number)，绿色是文件名长度name_len，橘色是file_type。

01602000  0f 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
01602010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
01602020  00 00 03 00 00 00 01 00  02 00 00 00 00 03 00 00  |................|
01602030  00 02 00 02 <font color=red>**2a 8b 40 64**</font>  <font color=blue>**04 00 00 00**</font> <font color=green>**03 00**</font> <font color=orange>**02**</font> c9  |....*.@d........|
01602040  7c b4 e4 05 00 00 00 04  00 01 00 00 00 00 00 00  ||...............|
01602050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
01602950  2e 00 00 00 00 00 00 00  2e 2e 00 00 00 00 00 00  |................|
01602960  64 69 72 00 00 00 00 00  74 65 73 74 00 00 00 00  |dir.....test....|
01602970  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|



