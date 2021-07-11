# 使用C语言操作Lustre文件系统

[TOC]

**c lustreapi测试**

## 1、准备工作

- 确认运行 C 程序的环境已安装

	安装参考：[https://blog.csdn.net/weixin_42090356/article/details/90678158](https://blog.csdn.net/weixin_42090356/article/details/90678158)

- 确认 Lustre 文件系统已安装

- 确认所依赖的头文件已存在

```sh
[root@node1 lustre]# pwd
/usr/include/lustre

[root@node1 lustre]# ls
liblustreapi.h  ll_fiemap.h  lustreapi.h  lustre_barrier_user.h  lustre_lfsck_user.h  lustre_user.h
```

## 2、基本操作

### 2.1、llapi_layout_file_create

```c
// 在使用 `llapi_layout_file_create()` 创建文件时，使用它来为其分配一个新的布局。
struct llapi_layout *llapi_layout_alloc(void);

// 使用指定的布局和模式创建它。
int llapi_layout_file_create(const char *path, 
	                         int open_flags, int mode,
                             const struct llapi_layout *layout);
```    

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    const char *path = "/mnt/lustre/test.txt";
    // 使用默认布局配置
    const struct llapi_layout *layout = llapi_layout_alloc();

    llapi_layout_file_create(path, O_CREAT|O_RDWR, 0777, layout);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_file_create.c -lm -llustreapi -o llapi_layout_file_create.out

[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
lmm_stripe_count:  1
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              37           0x25                0
```

### 2.2、llapi_layout_file_open


```c
// 打开路径中的一个已存在的文件，或使用指定的布局和模式创建一个文件
int llapi_layout_file_open(const char *path, 
	                       int open_flags, mode_t mode,
			               const struct llapi_layout *layout);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *path = "/mnt/lustre/test03.txt";
    
    const struct llapi_layout *layout = llapi_layout_alloc();
    llapi_layout_stripe_size_set(layout, 4194304);
    llapi_layout_stripe_count_set(layout, 2);

    rc = llapi_layout_file_open(path, O_CREAT|O_RDWR, 0777, layout);
    printf("%d\n", rc);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_file_open.c -lm -llustreapi -o llapi_layout_file_open.out

[root@node1 cfile]# ./llapi_layout_file_open.out
3

[root@node1 cfile]# lfs getstripe /mnt/lustre/test03.txt
/mnt/lustre/test03.txt
lmm_stripe_count:  2
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              66           0x42                0
             0              66           0x42                0
```

### 2.3、llapi_layout_stripe_count_get、llapi_layout_stripe_count_set

```c
// 设置布局的stripe count
int llapi_layout_stripe_count_set(struct llapi_layout *layout, uint64_t count);

// 存储布局的stripe count
int llapi_layout_stripe_count_get(const struct llapi_layout *layout,
				  uint64_t *count);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    const char *path = "/mnt/lustre/test.txt";

    const struct llapi_layout *layout = llapi_layout_alloc();

    int rc;
    uint64_t count;
    int stripe_count = 2;		
    
    rc = llapi_layout_stripe_count_set(layout, stripe_count);
    printf("result flag: %d\n", rc);
	rc = llapi_layout_stripe_count_get(layout, &count);
	printf("result flag: %d\n", rc);
	printf("the stripe count is: %ld\n", count);
    
    llapi_layout_file_create(path, O_CREAT|O_RDWR, 0777, layout);

    return 0;
}
```
```sh
[root@node1 cfile]# gcc llapi_layout_stripe_count_set.c -lm -llustreapi -o llapi_layout_stripe_count_set.out

[root@node1 cfile]# ./llapi_layout_stripe_count_set.out
result flag: 0
result flag: 0
the stripe count is: 2

[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
lmm_stripe_count:  2
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0
        obdidx           objid           objid           group
             0              39           0x27                0
             1              39           0x27                0
```

### 2.4、llapi_layout_stripe_size_get、llapi_layout_stripe_size_set

```c
// 设置布局的stripe size
int llapi_layout_stripe_size_set(struct llapi_layout *layout, uint64_t size);

// 存储布局的stripe size
int llapi_layout_stripe_size_get(const struct llapi_layout *layout,
				 uint64_t *size);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    const char *path = "/mnt/lustre/test.txt";

    const struct llapi_layout *layout = llapi_layout_alloc();

    int rc;
    uint64_t size;
    int stripe_size = 4194304;		
    
    rc = llapi_layout_stripe_size_set(layout, stripe_size);
    printf("result flag: %d\n", rc);
	rc = llapi_layout_stripe_size_get(layout, &size);
	printf("result flag: %d\n", rc);
	printf("the stripe size is: %ld\n", size);
    
    llapi_layout_file_create(path, O_CREAT|O_RDWR, 0777, layout);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_stripe_size_set.c -lm -llustreapi -o llapi_layout_stripe_size_set.out

[root@node1 cfile]# ./llapi_layout_stripe_size_set.out
result flag: 0
result flag: 0
the stripe size is: 4194304

[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
lmm_stripe_count:  1
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0
        obdidx           objid           objid           group
             0              40           0x28                0
```

### 2.5、llapi_layout_pool_name_get、llapi_layout_pool_name_set【有疑问】

```c
// 设置OST池的名称，文件对象将被分配到这个池中
int llapi_layout_pool_name_set(struct llapi_layout *layout,
			                   const char *pool_name);

// 将pool_name_len长度的OST池的名称字符存储到缓存，返回存储的字节数
int llapi_layout_pool_name_get(const struct llapi_layout *layout,
			                   char *pool_name, 
			                   size_t pool_name_len);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{

	// const char *path = "/mnt/lustre/test.txt";

    const struct llapi_layout *layout = llapi_layout_alloc();

    int rc;
    char mypool[LOV_MAXPOOLNAME + 1] = { '\0' };
    size_t pool_name_len = sizeof(mypool);
    const char *pool_name = "testp";		
    
    rc = llapi_layout_pool_name_set(layout, pool_name);
    printf("result flag: %d\n", rc);
	rc = llapi_layout_pool_name_get(layout, mypool, pool_name_len);
	printf("the number of bytes stored: %d\n", rc);
	printf("the pool name is: %s\n", mypool);

	// llapi_layout_file_create(path, O_CREAT|O_RDWR, 0777, layout);
    
    return 0;
}
```
```sh
[root@node1 cfile]# gcc llapi_layout_pool_name_set.c -lm -llustreapi -o llapi_layout_pool_name_set.out

[root@node1 cfile]# ./llapi_layout_pool_name_set.out
result flag: 0
the number of bytes stored: 0        ???
the pool name is: testp
```

### 2.6、llapi_layout_ost_index_get、llapi_layout_ost_index_set【有疑问】

**stripe number???**

```c
// Set the OST index associated with stripe number a stripe_number to a ost_index.
// OST和stripe number相关联
int llapi_layout_ost_index_set(struct llapi_layout *layout, 
	                           int stripe_number,
			                   uint64_t index);

// Store the index of the OST where stripe number a stripe_number is stored in a index.
int llapi_layout_ost_index_get(const struct llapi_layout *layout,
			                   uint64_t stripe_number, 
			                   uint64_t *index);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{

	const char *path = "/mnt/lustre/test.txt";
    const struct llapi_layout *layout = llapi_layout_alloc();

    int rc;
 	
    rc = llapi_layout_ost_index_set(layout, 0, 0);
    printf("result flag: %d\n", rc);

    uint64_t ost0;
    rc = llapi_layout_ost_index_get(layout, 0, &ost0);
	printf("result flag: %d\n", rc);
	printf("the index is: %ld\n", ost0);

    // uint64_t ost1;
	// rc = llapi_layout_ost_index_get(layout, 1, &ost1);
	// printf("result flag: %d\n", rc);

	llapi_layout_file_create(path, O_CREAT|O_RDWR, 0777, layout);
    
    return 0;
}
```
```sh
[root@node1 cfile]# gcc llapi_layout_ost_index_set.c -lm -llustreapi -o llapi_layout_ost_index_set.out

[root@node1 cfile]# ./llapi_layout_ost_index_set.out
result flag: 0
result flag: 0
the index is: 0
```

```sh
#  rc = llapi_layout_ost_index_set(layout, 0, 1);
[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
lmm_stripe_count:  1
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              40           0x28                0

#  rc = llapi_layout_ost_index_set(layout, 0, 0);
[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
lmm_stripe_count:  1
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0
        obdidx           objid           objid           group
             0              41           0x29                0 
```

### 2.7、llapi_layout_comp_extent_set、llapi_layout_comp_add

```c
// 设置当前布局组件的范围
int llapi_layout_comp_extent_set(struct llapi_layout *layout,
				                 uint64_t start, uint64_t end);

// 将一个组件添加到现有的组合或平面布局中。
int llapi_layout_comp_add(struct llapi_layout *layout);

// 获取当前布局组件的开始和结尾的偏移量
// Fetch the start and end offset of the current layout component.
int llapi_layout_comp_extent_get(const struct llapi_layout *layout,
				                 uint64_t *start, uint64_t *end);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *path = "/mnt/lustre/test.txt";
    
    const struct llapi_layout *layout = llapi_layout_alloc();

    uint64_t start00 = 0;
    uint64_t end00 = 4 * 1024 * 1024;  // 4m

    uint64_t start01 = end00;
    uint64_t end01 = LUSTRE_EOF;   

	llapi_layout_comp_extent_set(layout, start00, end00);
	llapi_layout_stripe_size_set(layout, 4194304);
    llapi_layout_stripe_count_set(layout, 2);
	llapi_layout_comp_add(layout);

	llapi_layout_comp_extent_set(layout, start01, end01);
	llapi_layout_stripe_size_set(layout, 1048576);
    llapi_layout_stripe_count_set(layout, 1);
	llapi_layout_comp_add(layout);

    rc = llapi_layout_file_create(path, 0, 0777, layout);
    printf("%d\n", rc);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_comp_extent_set.c -lm -llustreapi -o llapi_layout_comp_extent_set.out

[root@node1 cfile]# ./llapi_layout_comp_extent_set.out
3

[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
  lcm_layout_gen:  3
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  2
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x45:0x0] }
      - 1: { l_ost_idx: 1, l_fid: [0x100010000:0x45:0x0] }

    lcme_id:             2
    lcme_flags:          init
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  1
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x46:0x0] }
```

### 2.8、llapi_layout_comp_extent_get、llapi_layout_comp_use

```c
// 获取当前布局组件的开始和结尾的偏移量
int llapi_layout_comp_extent_get(const struct llapi_layout *layout,
				                 uint64_t *start, uint64_t *end);

enum llapi_layout_comp_use {
	LLAPI_LAYOUT_COMP_USE_FIRST = 1,
	LLAPI_LAYOUT_COMP_USE_LAST = 2,
	LLAPI_LAYOUT_COMP_USE_NEXT = 3,
	LLAPI_LAYOUT_COMP_USE_PREV = 4,
};
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *path = "/mnt/lustre/test.txt";
    
    const struct llapi_layout *layout = llapi_layout_alloc();

    uint64_t start00 = 0;
    uint64_t end00 = 4 * 1024 * 1024;  // 4m

    uint64_t start01 = end00;
    uint64_t end01 = LUSTRE_EOF;   

	llapi_layout_comp_extent_set(layout, start00, end00);
	llapi_layout_stripe_size_set(layout, 4194304);
    llapi_layout_stripe_count_set(layout, 2);
	llapi_layout_comp_add(layout);

	llapi_layout_comp_extent_set(layout, start01, end01);
	llapi_layout_stripe_size_set(layout, 1048576);
    llapi_layout_stripe_count_set(layout, 1);
	llapi_layout_comp_add(layout);

    rc = llapi_layout_file_create(path, 0, 0777, layout);
    printf("%d\n", rc);

    uint64_t s1, e1;
    llapi_layout_comp_extent_get(layout, &s1, &e1);
    printf("start: %ld\n", s1);
    printf("end: %ld\n", e1);

    llapi_layout_comp_use(layout, LLAPI_LAYOUT_COMP_USE_FIRST);
    uint64_t s0, e0;
    llapi_layout_comp_extent_get(layout, &s0, &e0);
    printf("start: %ld\n", s0);
    printf("end: %ld\n", e0);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_comp_extent_get.c -lm -llustreapi -o llapi_layout_comp_extent_get.out

[root@node1 cfile]# ./llapi_layout_comp_extent_get.out
3
start: 4194304
end: -1
start: 0
end: 4194304
```

### 2.9、llapi_layout_comp_flags_set、llapi_layout_comp_flags_get

**uint32_t flags：组件的标志/索引**

```c
// 设置当前组件的指定标志，保持其他标志不变。
int llapi_layout_comp_flags_set(struct llapi_layout *layout, uint32_t flags);

// 获取当前组件的属性标志。
int llapi_layout_comp_flags_get(const struct llapi_layout *layout,
				uint32_t *flags);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *path = "/mnt/lustre/test.txt";
    
    const struct llapi_layout *layout = llapi_layout_alloc();

    uint64_t start00 = 0;
    uint64_t end00 = 4 * 1024 * 1024;  // 4m

    uint64_t start01 = end00;
    uint64_t end01 = LUSTRE_EOF;   

	llapi_layout_comp_extent_set(layout, start00, end00);
	llapi_layout_comp_add(layout);

	llapi_layout_comp_extent_set(layout, start01, end01);
	llapi_layout_comp_add(layout);

    rc = llapi_layout_file_create(path, 0, 0777, layout);
    printf("%d\n", rc);


    uint64_t s1, e1;
    llapi_layout_comp_extent_get(layout, &s1, &e1);
    printf("start: %ld\n", s1);
    printf("end: %ld\n", e1);
	llapi_layout_comp_flags_set(layout, 3);

    uint64_t s3, e3;
    uint32_t flags;
    llapi_layout_comp_extent_get(layout, &s3, &e3);
    printf("start: %ld\n", s3);
    printf("end: %ld\n", e3);
    llapi_layout_comp_flags_get(layout, &flags);
    printf("%ld\n", flags);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_comp_flags_set.c -lm -llustreapi -o llapi_layout_comp_flags_set.out

[root@node1 cfile]# ./llapi_layout_comp_flags_set.out
3
start: 4194304
end: -1
start: 4194304
end: -1
3
```

### 2.10、llapi_layout_file_comp_add

```c
// 添加布局组件到一个已存在的文件
int llapi_layout_file_comp_add(const char *path,
			                   const struct llapi_layout *layout);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *path = "/mnt/lustre/test.txt";
    
    const struct llapi_layout *layout;

    uint64_t start00 = 0;
    uint64_t end00 = 4 * 1024 * 1024;  // 4m

    uint64_t start01 = end00;
    uint64_t end01 = 8 * 1024 * 1024;   

    layout = llapi_layout_alloc();
    llapi_layout_stripe_count_set(layout, 1);

	llapi_layout_comp_extent_set(layout, start00, end00);

	llapi_layout_file_create(path, 0, 0777, layout);

	layout = llapi_layout_alloc();
	llapi_layout_stripe_count_set(layout, 2);
	
	llapi_layout_comp_extent_set(layout, start01, end01);

	llapi_layout_file_comp_add(path, layout);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_file_comp_add.c -lm -llustreapi -o llapi_layout_file_comp_add.out

[root@node1 cfile]# ./llapi_layout_file_comp_add.out

[root@node1 cfile]# lfs getstripe /mnt/lustre/test.txt
/mnt/lustre/test.txt
  lcm_layout_gen:  3
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x53:0x0] }

    lcme_id:             2
    lcme_flags:          init
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   8388608
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x55:0x0] }
      - 1: { l_ost_idx: 1, l_fid: [0x100010000:0x54:0x0] }
```

### 2.11、llapi_layout_get_by_path、llapi_layout_free

```c
// 返回一个指针，指向一个包含指定路径下文件布局的新分配的队列数据结构
struct llapi_layout *llapi_layout_get_by_path(const char *path, uint32_t flags);

// 释放布局所占内存
void llapi_layout_free(struct llapi_layout *layout);

// Return a pointer to a newly-allocated opaque data type containing the layout for the file referenced by open file descriptor \a fd.
// fd 打开文件的返回值
struct llapi_layout *llapi_layout_get_by_fd(int fd, uint32_t flags);

// Return a pointer to a newly-allocated opaque data type containing the layout for the file associated with Lustre file identifier string a fidstr.
struct llapi_layout *llapi_layout_get_by_fid(const char *path,
					                         const lustre_fid *fid,
					                         uint32_t flags);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{

    const char *path = "/mnt/lustre/test.txt";
    
    struct llapi_layout *layout;
    
    layout = llapi_layout_get_by_path(path, 0);

    int rc;
    uint64_t count;
    rc = llapi_layout_stripe_count_get(layout, &count);
    printf("result flag: %d\n", rc);
    printf("the stripe count is: %ld\n", count);

    llapi_layout_free(layout);
    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_layout_get_by_path.c -lm -llustreapi -o llapi_layout_get_by_path.out

[root@node1 cfile]# ./llapi_layout_get_by_path.out
result flag: 0
the stripe count is: 2
```

### 2.12、llapi_path2fid、llapi_layout_get_by_fid

```c
extern int llapi_path2fid(const char *path, lustre_fid *fid);

// Return a pointer to a newly-allocated opaque data type containing the layout for the file associated with Lustre file identifier string a fidstr. 
struct llapi_layout *llapi_layout_get_by_fid(const char *path,
					                         const lustre_fid *fid,
					                         uint32_t flags);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    int rc;
    lustre_fid fid;
    uint64_t count;
    const char *path = "/mnt/lustre/test.txt";

    rc = llapi_path2fid(path, &fid);
    printf("%d \n", rc);
    struct llapi_layout *layout = llapi_layout_get_by_fid(path, &fid, 0);

    llapi_layout_stripe_count_get(layout, &count);
    printf("%ld\n", count);
}
```

```sh
[root@node1 cfile]# gcc llapi_path2fid.c -lm -llustreapi -o llapi_path2fid.out

[root@node1 cfile]# ./llapi_path2fid.out
0 
1
```

### 2.13、llapi_lov_get_uuids

```c
extern int llapi_lov_get_uuids(int fd, struct obd_uuid *uuidp, int *ost_count);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int i, fd;
	int osts_all;
	struct obd_uuid *uuidp;
	struct obd_uuid uuids[1024];
	char *path = "/mnt/lustre/test.txt";
	fd = open(path, O_RDONLY, 0644);

	llapi_lov_get_uuids(fd, uuids, &osts_all);
	for (i = 0, uuidp = uuids; i < osts_all; i++, uuidp++) {
        printf("UUID %d is %s\n",i, uuidp->uuid);
    }

}
```

```sh
[root@node1 cfile]# gcc llapi_lov_get_uuids.c -lm -llustreapi -o llapi_lov_get_uuids.out

[root@node1 cfile]# ./llapi_lov_get_uuids.out
UID 0 is lustrefs-OST0000_UUID
UUID 1 is lustrefs-OST0001_UUID
```

### 2.14、llapi_file_create_pool

```c
extern int llapi_file_create_pool(const char *name,  
                              unsigned long long stripe_size,
                              int stripe_offset, int stripe_count,
                              int stripe_pattern, char *pool_name);
```

```sh
# 先创建池，再将 OST 添加进入
[root@node2 ~]# lctl pool_new lustrefs.testpool
Pool lustrefs.testpool created
[root@node2 ~]# lctl pool_add lustrefs.testpool OST[0-1]
OST lustrefs-OST0000_UUID added to pool lustrefs.testpool
OST lustrefs-OST0001_UUID added to pool lustrefs.testpool
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
	int rc;
    const char *name = "/mnt/lustre/text10.txt";
    char *pool_name = "testpool";
    rc = llapi_file_create_pool(name, 4194304, -1, 2, 0, pool_name);

    printf("%d\n", rc);
    return 0;

}
```

```sh
[root@node1 cfile]# gcc llapi_file_create_pool.c -lm -llustreapi -o llapi_file_create_pool.out

[root@node1 cfile]# ./llapi_file_create_pool.out
UID 0 is lustrefs-OST0000_UUID
UUID 1 is lustrefs-OST0001_UUID

[root@node1 cfile]# lfs getstripe /mnt/lustre/text10.txt
/mnt/lustre/text10.txt
lmm_stripe_count:  2
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0
lmm_pool:          testpool
        obdidx           objid           objid           group
             0             132           0x84                0
             1             131           0x83                0
```

### 2.15、llapi_file_open_pool

```c
extern int llapi_file_open_pool(const char *name, int flags, int mode,
                                unsigned long long stripe_size,
                                int stripe_offset, int stripe_count,
                                int stripe_pattern, char *pool_name);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    int rc;
    const char *name = "/mnt/lustre/poolt";
    char *pool_name = "testpool";
    llapi_file_open_pool(name, O_RDWR, 0644, 4194304, -1, 2, 0, pool_name);

    const char *file = "/mnt/lustre/poolt/test.txt";

    llapi_file_create(file, 1048576, -1, 2 ,0);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_file_open_pool.c -lm -llustreapi -o llapi_file_open_pool.out

[root@node1 cfile]# ./llapi_file_open_pool.out
UID 0 is lustrefs-OST0000_UUID
UUID 1 is lustrefs-OST0001_UUID

[root@node1 cfile]# lfs getstripe /mnt/lustre/poolt/test.txt
/mnt/lustre/poolt/test.txt
lmm_stripe_count:  2
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0
        obdidx           objid           objid           group
             0             133           0x85                0
             1             132           0x84                0

[root@node1 cfile]# touch /mnt/lustre/poolt/test02.txt
[root@node1 cfile]# lfs getstripe /mnt/lustre/poolt/test02.txt
/mnt/lustre/poolt/test02.txt
lmm_stripe_count:  2
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
lmm_pool:          testpool
        obdidx           objid           objid           group
             1             133           0x85                0
             0             134           0x86                0             
```

### 2.16、llapi_file_lookup【有疑问】

```c
extern int llapi_file_lookup(int dirfd, const char *name);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    int rc, dirfd;
    char *dir = "/mnt/lustre/poolt";
    const char *name = "/mnt/lustre/poolt/test.txt";
    
    dirfd = open(dir, O_RDWR, 0777);
    rc = llapi_file_lookup(dirfd, name);
    printf("%d\n", rc);

    return 0;
}
```

```sh
[root@node1 cfile]# gcc llapi_file_lookup.c -lm -llustreapi -o llapi_file_lookup.out

[root@node1 cfile]# ./llapi_file_lookup.out
-22  ？？？
```

### 2.17、llapi_file_open_param【有疑问】

```c
extern int llapi_file_open_param(const char *name, 
	                             int flags, mode_t mode,
				                 const struct llapi_stripe_param *param);

struct llapi_stripe_param {
	unsigned long long	lsp_stripe_size;
	char			*lsp_pool;
	int			lsp_stripe_offset;
	int			lsp_stripe_pattern;
	/* Number of stripes. Size of lsp_osts[] if lsp_specific is true.*/
	int			lsp_stripe_count;
	bool			lsp_is_specific;
	__u32			lsp_osts[0];
};
```

### 2.18、llapi_file_create

```c
extern int llapi_file_create(const char *name, unsigned long long stripe_size,
                             int stripe_offset, int stripe_count,
                             int stripe_pattern);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h> // .h 文件默认是在 /usr/include 目录下
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>// 这里表示 /usr/include/lustre/ 目录下的头文件      

int open_stripe_file()
{
        char *path = "/mnt/lustre/test.txt";
        int stripe_size = 65536;    /* System default is 4M */
        int stripe_offset = -1;     /* Start at default */
        int stripe_count = 1;  /*Single stripe for this demo*/
        int stripe_pattern = 0;     /* only RAID 0 at this time */
        int rc, fd;
             
        rc = llapi_file_create(path,
                        stripe_size,stripe_offset,stripe_count,stripe_pattern);
        /* result code is inverted, we may return -EINVAL or an ioctl error.
         * We borrow an error message from sanity.c
         */
        if (rc) {
                fprintf(stderr,"llapi_file_create failed: %d (%s) \n", rc, strerror(-rc));
                return -1;
        }
        /* llapi_file_create closes the file descriptor, we must re-open */
        fd = open(path, O_CREAT | O_RDWR | O_LOV_DELAY_CREATE, 0644);
        if (fd < 0) {
                fprintf(stderr, "Can't open %s file: %d (%s)\n", path, errno, strerror(errno));
                return -1;
        }
        return fd;
}

/* 如果文件不存在，会自动创建 */
int write_file(int fd)
{
    char *stng = "DEADBEEF";
    int cnt = 0;

    for( cnt = 0; cnt < 10; cnt++) { /* Size of the file in words */
        write(fd, stng, sizeof(stng));
    }
    return 0;
}

int close_file(int fd)
{
    if (close(fd) < 0) {
        fprintf(stderr, "File close failed: %d (%s)\n", errno, strerror(errno));
        return -1;
    }
    return 0;
}

int main()
{
    int file,rc;
    printf("Open a file with striping\n");
    file = open_stripe_file();
    if ( file < 0 ) {
        printf("Exiting\n");
        exit(1);
    }

    printf("Write to the file...\n");
    rc = write_file(file);
    close_file(file);
    printf("Done\n");
}
```

执行：

```sh
# 编译
[root@node1 ~]# gcc main.c -lm -llustreapi

[root@node1 ~]# ls
a.out

# 执行
[root@node1 ~]# ./a.out 
Open a file with striping
Write to the file...
Done
```

执行完后，查看：

```sh
[root@node1 ~]# cd /mnt/lustre/

[root@node1 lustre]# ls
test.txt

[root@node1 ~]# cat /mnt/lustre/test.txt
DEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEF
```

### 2.19、llapi_file_open

```c
// 在 Lustre 文件系统上打开文件（或设备）。
extern int llapi_file_open(const char *name, 
                           int flags, int mode,
                           unsigned long long stripe_size, 
                           int stripe_offset,
                           int stripe_count, 
                           int stripe_pattern);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    int rc;
    char *path = "/mnt/lustre/test.txt";
    rc = llapi_file_open(path, O_RDWR, 0644,1048576, 0, 2, LOV_PATTERN_RAID0);
    if (rc < 0) {
        fprintf(stderr, "file open has failed, %s\n",strerror(-rc));
        return -1;
    }
    printf("Done\n");
    return 0;
}
```

### 2.20、llapi_file_get_stripe【有疑问】


```c
// 获取 Lustre 文件系统上的文件或目录的条带信息。
extern int llapi_file_get_stripe(const char *path, 
                                 struct lov_user_md *lum);
```

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <dirent.h>
#include <lustre/lustreapi.h>

int main()
{
    int i;
    int rc;
    int lum_size;

    const char *path = "/mnt/lustre/test.txt";
    struct lov_user_md_v1 *lum_file = malloc(1024);

    rc = llapi_file_get_stripe(path, lum_file);

    printf("Lov magic %u\n", lum_file->lmm_magic);
    printf("Lov pattern %u\n", lum_file->lmm_pattern);
    printf("Lov object %llu\n", lum_file->lmm_objects);
    printf("Lov stripe size %u\n", lum_file->lmm_stripe_size);
    printf("Lov stripe count %hu\n", lum_file->lmm_stripe_count);
    printf("Lov stripe offset %u\n", lum_file->lmm_stripe_offset);

    for (i = 0; i < lum_file->lmm_stripe_count; i++) {
        printf("Object index %d Object group %d\n", lum_file->lmm_objects[i].l_ost_idx, lum_file->lmm_objects[i].l_object_seq);
    }

    // ‘struct lov_user_md_v1’ has no member named ‘lmm_object_id’
    // printf("Lov object id %llu\n", lum_file->lmm_object_id);

    // l_object_id、l_object_seq
    // 报错：没有名为‘l_object_seq’的成员
    // for (i = 0; i < lum_file->lmm_stripe_count; i++) {
    //     printf("Object index %d Objid %llu\n", lum_file->lmm_objects[i].l_ost_idx, lum_file->lmm_objects[i].l_object_id);
    // }

}
```

```sh
[root@node1 ~]# gcc llapi_file_get_stripe.c -lm -llustreapi -o llapi_file_get_stripe.out
[root@node1 ~]# ./llapi_file_get_stripe.out
Lov magic 198249424
Lov pattern 1
Lov object 20901936
Lov stripe size 1048576
Lov stripe count 1
Lov stripe offset 0
```

参考地址：

[https://doc.lustre.org/lustre_manual.xhtml#dbdoclet.50438215_marker-1297700](https://doc.lustre.org/lustre_manual.xhtml#dbdoclet.50438215_marker-1297700)

[https://github.com/perrynzhou/kernel-note2/blob/e7525324ffff4ab804387edc952c7a45f1becea4/lustre-13/lustre/tests/llapi_layout_test.c](https://github.com/perrynzhou/kernel-note2/blob/e7525324ffff4ab804387edc952c7a45f1becea4/lustre-13/lustre/tests/llapi_layout_test.c)


--------------------------------------------


```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
/* 
   直接使用 C 语言的打开、读写函数，
   写到文件系统的挂载目录下，或写入到具有指定布局的目录下
   gcc cwriteread.c -lm -llustreapi -o  cwriteread.out
 */ 
main()
{
    int fd, size;
    //const char * pathname = "/mnt/lustre/test.txt";
    const char *pathname = "/mnt/lustre/test/test.txt";

    char s[] = "www.baidu.com\n", buffer[80];

    fd = open(pathname, O_WRONLY|O_CREAT);
    write(fd, s, sizeof(s));
    close(fd);

    fd = open(pathname, O_RDONLY);
    size = read(fd, buffer, sizeof(buffer));
    close(fd);

    printf("%s", buffer);
}
/*
   数据写入文件，且具有所在目录附带的布局信息；
   打印出 buffer 中的内容。
 */
```

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
/* 
   使用 C 语言的 lseek 函数实现随机读写
   gcc clseek.c -lm -llustreapi -o clseek.out
 */ 
main()
{
    int fd;
    const char * pathname = "/mnt/lustre/test/test.txt";
    char s[] = "google.com\n", buffer[80];

    // 从偏移量为 4 处开始写数据
    // 文件内容由 `www.baidu.com` 改变成为 `www.google.com`
    fd = open(pathname, O_WRONLY);
    lseek(fd, 4, SEEK_SET);
    write(fd, s, sizeof(s));
    close(fd);

    // 从偏移量为 4 处开始读数据
    // 打印 `google.com`
    fd = open(pathname, O_RDONLY);
    lseek(fd, 4, SEEK_SET);
    read(fd, buffer, sizeof(buffer));
    close(fd);
    printf("%s", buffer);
}
```

```c
#include <stdio.h>
/* 
   直接使用 C 语言的打开、读写函数，
   写到文件系统的挂载目录下，或写入到具有指定布局的目录下
   gcc cfwriteread.c -lm -llustreapi -o cfwriteread.out
 */
main()
{
    FILE *fp;
    const char *pathname = "/mnt/lustre/test/test.txt";
    char s[] = "www.baidu.com\n", buffer[80];

    fp = fopen(pathname, "wb+");
    fwrite(s, sizeof(s), 1, fp);    
    fclose(fp);

    fp = fopen(pathname, "rb");
    fread(buffer, sizeof(s), 1, fp); 
    printf("%s", buffer);
    fclose(fp);
}
/*
   数据写入文件，且具有所在目录附带的布局信息；
   打印出 buffer 中的内容。
 */
```

```c
#include <stdio.h>
/* 
   使用 C 语言的 fseek 函数实现随机读写
   gcc cfseek.c -lm -llustreapi -o cfseek.out
 */ 
main()
{
    FILE *fp;
    const char *pathname = "/mnt/lustre/test/test.txt";
    char s[] = "google.com\n", buffer[80];

    // 从偏移量为 4 处开始写数据
    // 文件内容由 `www.baidu.com` 改变成为 `www.google.com`
    fp = fopen(pathname, "wb+");
    fseek(fp, 4, SEEK_CUR);
    fwrite(s, sizeof(s), 1, fp);    
    fclose(fp);


    // 从偏移量为 4 处开始读数据
    // 打印 `google.com`
    fp = fopen(pathname, "rb");
    fseek(fp, 4, SEEK_CUR);
    fread(buffer, sizeof(s), 1, fp);    
    fclose(fp);
    printf("%s", buffer);
}
```

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <lustre/lustreapi.h>
/* 
   使用 lustre api 打开函数、和 C 的读写函数，在创建文件时指定文件布局
   gcc clwriteread.c -lm -llustreapi -o clwriteread.out
*/ 
main()
{
    int fd;
    const char *path = "/mnt/lustre/test/test.txt";
    int flags = O_CREAT | O_RDWR;
    int mode = 0644;
    int stripe_size = 65536;
    int stripe_offset = -1;
    int stripe_count = 2;
    int stripe_pattern = 0;

    fd = llapi_file_open(path, flags, mode,
                        stripe_size, stripe_offset,
                        stripe_count, stripe_pattern);

    char ibuf[] = "www.baidu.com\n", obuf[80];
    write(fd, ibuf, sizeof(ibuf));

    lseek(fd, 4, SEEK_SET);

    read(fd,obuf,sizeof(obuf));

    close(fd);
    printf("%s", obuf);
}
/*
   创建一个具有指定布局的文件，并写入数据；
   将读取位置移动到偏移量为4的位置处，并打印出 buffer 中的内容。
 */
```