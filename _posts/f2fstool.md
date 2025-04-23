title: f2fstool函数说明
tags:
	- f2fs
date: 2022/06/21
categories: f2fs
---

# f2fs的基本数据结构
* sector: 扇区
* block: F2FS的数据存储的基本单位是block，大小为4KB，整个flash设备被格式化为多个block组成的结构。很多数据结构也被设计为4KB的大小，这是因为很多flash设备单次IO的读写都是基于4KB的倍数进行。
* segment: segment是管理block的结构，一个segment的大小是512个block，也就是2MB。
* section: 默认情况下一个segment等于一个section，section是GC的基本操作单位，每次GC都会从section中选出特定的segment进行回收。F2FS将section分为了6类，分别是hot-node，warm-node，cold-node，hot-data，warm-data，cold-data，hot->cold表示了数据的从高到低的修改频率，通过不同类型的section，进行gc的时候可针对使用hot的section进行gc，以降低gc的时间开销。
* zone: 默认情况一个zone等于一个section，与物理设备有关，大部分情况下用不上


# f2fs_configuration结构体
f2fs_configuration结构体在include/f2fs_fs.h文件中定义，f2fs文件系统初始化时默认参数也在include/f2fs_fs.h文件中定义。在lib/libf2fs.c的f2fs_init_configuration函数中进行初始化
```
struct f2fs_configuration {
	uint32_t reserved_segments;
	uint32_t new_reserved_segments;
	int sparse_mode;
	int zoned_mode;
	int zoned_model;
	size_t zone_blocks;
	double overprovision;
	double new_overprovision;
	uint32_t cur_seg[6];
	//一个section几个segment，默认为1个
	uint32_t segs_per_sec;
	//默认为1
	uint32_t secs_per_zone;
	//默认为1
	uint32_t segs_per_zone;
	uint32_t start_sector;
	uint32_t total_segments;
	uint32_t sector_size;
	uint64_t device_size;
	uint64_t total_sectors;
	uint64_t wanted_total_sectors;
	uint64_t wanted_sector_size;
	uint64_t target_sectors;
	uint64_t max_size;
	//一个block包含多少个扇区sector
	uint32_t sectors_per_blk;
	//一个segment包含多少个block
	uint32_t blks_per_seg;
	__u8 init_version[VERSION_LEN + 1];
	__u8 sb_version[VERSION_LEN + 1];
	__u8 version[VERSION_LEN + 1];
	char *vol_label;
	char *vol_uuid;
	uint16_t s_encoding;
	uint16_t s_encoding_flags;
	int heap;
	int32_t kd;
	int32_t dump_fd;
	//设备列表
	struct device_info devices[MAX_DEVICES];
	int ndevs;
	char *extension_list[2];
	const char *rootdev_name;
	int dbg_lv;
	int show_dentry;
	int trim;
	int trimmed;
	int func;
	void *private;
	int dry_run;
	//andorid 非内核运行检查
	int no_kernel_check;
	int fix_on;
	int force;
	int defset;
	int bug_on;
	int bug_nat_bits;
	bool quota_fixed;
	int alloc_failed;
	int auto_fix;
	int layout;
	int show_file_map;
	u64 show_file_map_max_offset;
	int quota_fix;
	int preen_mode;
	int ro;
	//andorid
	int preserve_limits;		/* preserve quota limits */
	int large_nat_bitmap;
	int fix_chksum;			/* fix old cp.chksum position */
	__le32 feature;			/* defined features */
	unsigned int quota_bits;	/* quota bits */
	time_t fixed_time;

	/* mkfs parameters */
	int fake_seed;
	uint32_t next_free_nid;
	uint32_t quota_inum;
	uint32_t quota_dnum;
	uint32_t lpf_inum;
	uint32_t lpf_dnum;
	uint32_t lpf_ino;
	uint32_t root_uid;
	uint32_t root_gid;

	/* defragmentation parameters */
	int defrag_shrink;
	uint64_t defrag_start;
	uint64_t defrag_len;
	uint64_t defrag_target;

	/* sload parameters */
	char *from_dir;
	char *mount_point;
	char *target_out_dir;
	char *fs_config_file;
#ifdef HAVE_LIBSELINUX
	struct selinux_opt seopt_file[8];
	int nr_opt;
#endif
	int preserve_perms;

	/* resize parameters */
	int safe_resize;

	/* precomputed fs UUID checksum for seeding other checksums */
	uint32_t chksum_seed;

	/* cache parameters */
	dev_cache_config_t cache_config;

	/* compression support for sload.f2fs */
	compress_config_t compress;
};
```

