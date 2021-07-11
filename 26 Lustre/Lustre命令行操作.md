# Lustre命令行操作

[TOC]

#### 1、为单个文件指定文件布局

```sh
[root@node1 ~]# lfs setstripe -c -1 -S 4M /mnt/lustre/test3.txt
[root@node1 ~]# lfs getstripe /mnt/lustre/test3.txt
/mnt/lustre/test3.txt
lmm_stripe_count:  2
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              70           0x46                0
             0              70           0x46                0

```

#### 2、为目录指定文件布局

```sh
[root@node1 ~]# mkdir /mnt/lustre/test1/
[root@node1 ~]# lfs setstripe -c -1 -S 4M /mnt/lustre/test1/
[root@node1 ~]# lfs getstripe /mnt/lustre/test1
/mnt/lustre/test1
stripe_count:  -1 stripe_size:   4194304 stripe_offset: -1

[root@node1 ~]# touch /mnt/lustre/test1/test2.txt
[root@node1 ~]# lfs getstripe /mnt/lustre/test1/test2.txt
/mnt/lustre/test1/test2.txt
lmm_stripe_count:  2
lmm_stripe_size:   4194304
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 0    【表示文件的条带开始写入的ost的索引】
        obdidx           objid           objid           group
             0              69           0x45                0
             1              69           0x45                0
```

#### 3、确定文件\目录位于哪个MDT上

```sh
[root@node1 ~]# lfs getstripe -M /mnt/lustre/test1.txt
0
[root@node1 ~]# lfs getstripe -M /mnt/lustre/test1/
0
[root@node1 ~]# lfs getstripe --mdt-index /mnt/lustre/test1/
0
```

#### 4、创建PFL文件

```sh
[root@node1 lustre]# touch test1.txt
[root@node1 lustre]# lfs setstripe -E 4M -c 1 -S 4M -E -1 -c 1 -S 4M test1.txt
[root@node1 lustre]# lfs getstripe test1.txt
test1.txt
  lcm_layout_gen:  2
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x56:0x0] }

    lcme_id:             2
    lcme_flags:          0
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    65535
      lmm_stripe_offset: -1
[root@node1 lustre]# dd if=/dev/zero of=test1.txt count=4 bs=4M
记录了4+0 的读入
记录了4+0 的写出
16777216字节(17 MB)已复制，0.0375243 秒，447 MB/秒
[root@node1 lustre]# lfs getstripe test1.txt
test1.txt
  lcm_layout_gen:  3
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x56:0x0] }

    lcme_id:             2
    lcme_flags:          init
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x54:0x0] }

# ?????指定ost索引失败
[root@node1 lustre]# lfs setstripe -E 4M -c 1 -i 0 -E 8M -c 1 -i 1 test2.txt
[root@node1 lustre]# lfs getstripe test2.txt
test2.txt
lmm_stripe_count:  1
lmm_stripe_size:   1048576
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              88           0x58                0      
```

#### 4、删除、添加PFL组件

```sh
# 添加组件前，必须删除最后一个指定为 -E -1 或 -E EOF 组件
[root@node1 lustre]# lfs setstripe --component-del -I 2 test1.txt
[root@node1 lustre]# lfs getstripe test1.txt
test1.txt
  lcm_layout_gen:  4
  lcm_entry_count: 1
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x56:0x0] }

[root@node1 lustre]# lfs setstripe --component-add -E -1 -c 2 -o 0,1 test1.txt 
[root@node1 lustre]# lfs getstripe test1.txt
test1.txt
  lcm_layout_gen:  5
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x56:0x0] }

    lcme_id:             5
    lcme_flags:          0
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
[root@node1 lustre]# dd if=/dev/zero of=test1.txt count=4 bs=4M
记录了4+0 的读入
记录了4+0 的写出
16777216字节(17 MB)已复制，0.0518376 秒，324 MB/秒
[root@node1 lustre]# lfs getstripe test1.txt
test1.txt
  lcm_layout_gen:  6
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  1
      lmm_stripe_size:   4194304
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x56:0x0] }

    lcme_id:             5
    lcme_flags:          init
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x55:0x0] }
      - 1: { l_ost_idx: 1, l_fid: [0x100010000:0x57:0x0] }


````

#### 5、普通布局向组合布局迁移

```sh
[root@node1 lustre]# lfs setstripe -c 1 -S 128K test3.txt
[root@node1 lustre]# lfs getstripe test3.txt
test3.txt
lmm_stripe_count:  1
lmm_stripe_size:   131072
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              89           0x59                0

[root@node1 lustre]# dd if=/dev/urandom of=/mnt/lustre/test3.txt bs=1M count=5
记录了5+0 的读入
记录了5+0 的写出
5242880字节(5.2 MB)已复制，0.04037 秒，130 MB/秒
[root@node1 lustre]# lfs getstripe test3.txt
test3.txt
lmm_stripe_count:  1
lmm_stripe_size:   131072
lmm_pattern:       1
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              89           0x59                0
# 两个组件的复合布局
[root@node1 lustre]# lfs migrate -E 1M -S 512K -c 1 -E -1 -S 1M -c 2 test3.txt
[root@node1 lustre]# lfs getstripe test3.txt
test3.txt
  lcm_layout_gen:  4
  lcm_entry_count: 2
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   1048576
      lmm_stripe_count:  1
      lmm_stripe_size:   524288
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x57:0x0] }

    lcme_id:             2
    lcme_flags:          init
    lcme_extent.e_start: 1048576
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x5a:0x0] }
      - 1: { l_ost_idx: 0, l_fid: [0x100000000:0x58:0x0] }

```

#### 6、一个组合布局迁移至另一个组合布局

```sh
[root@node1 lustre]# lfs setstripe -E 1M -S 512K -c 1 -E -1 -S 1M -c 2 test4.txt
[root@node1 lustre]# dd if=/dev/urandom of=/mnt/lustre/test4.txt bs=1M count=5
记录了5+0 的读入
记录了5+0 的写出
5242880字节(5.2 MB)已复制，0.0512445 秒，102 MB/秒
[root@node1 lustre]# lfs migrate -E 1M -S 1M -c 2 -E 4M -S 1M -c 2 -E -1 -S 3M -c 3 test4.txt
[root@node1 lustre]# lfs getstripe test4.txt
test4.txt
  lcm_layout_gen:  6
  lcm_entry_count: 3
    lcme_id:             1
    lcme_flags:          init
    lcme_extent.e_start: 0
    lcme_extent.e_end:   1048576
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x5d:0x0] }
      - 1: { l_ost_idx: 0, l_fid: [0x100000000:0x5a:0x0] }

    lcme_id:             2
    lcme_flags:          init
    lcme_extent.e_start: 1048576
    lcme_extent.e_end:   4194304
      lmm_stripe_count:  2
      lmm_stripe_size:   1048576
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 0
      lmm_objects:
      - 0: { l_ost_idx: 0, l_fid: [0x100000000:0x5b:0x0] }
      - 1: { l_ost_idx: 1, l_fid: [0x100010000:0x5e:0x0] }

    lcme_id:             3
    lcme_flags:          init
    lcme_extent.e_start: 4194304
    lcme_extent.e_end:   EOF
      lmm_stripe_count:  2
      lmm_stripe_size:   3145728
      lmm_pattern:       1
      lmm_layout_gen:    0
      lmm_stripe_offset: 1
      lmm_objects:
      - 0: { l_ost_idx: 1, l_fid: [0x100010000:0x5f:0x0] }
      - 1: { l_ost_idx: 0, l_fid: [0x100000000:0x5c:0x0] }
```

#### 7、查找组件数大于等于1的文件

```sh
[root@node1 lustre]# lfs find /mnt/lustre --component-count=+1
/mnt/lustre/test4.txt
/mnt/lustre/test3.txt
/mnt/lustre/test1.txt
```

#### 8、OST池操作

```sh
# mgs上
[root@node2 ~]# lctl pool_new lustrefs.test-pool
Pool lustrefs.test-pool created

[root@node2 ~]# lctl pool_add  lustrefs.test-pool OST[0-1]
OST lustrefs-OST0000_UUID added to pool lustrefs.test-pool
OST lustrefs-OST0001_UUID added to pool lustrefs.test-pool

[root@node2 ~]# lctl pool_list lustrefs
Pools from lustrefs:
lustrefs.test-pool

[root@node2 ~]# lctl pool_remove lustrefs.test-pool OST[0-1]
OST lustrefs-OST0000_UUID removed from pool lustrefs.test-pool
OST lustrefs-OST0001_UUID removed from pool lustrefs.test-pool

[root@node2 ~]# lctl pool_destroy lustrefs.test-pool
Pool lustrefs.test-pool destroyed
```