# 使用C API操作HDFS的简单测试

[TOC]

### 1、简介

C APIs 作为 HDFS APIs 的子集 ，用来操作 HDFS 文件和文件系统。

头文件 hdfs.h 可在 `$HADOOP_HDFS_HOME/include/` 中找到。

### 2、配置环境变量

```sh
[root@node3 ~]# cat /etc/profile
export JAVA_HOME=/opt/jdk1.8.0_281
export HADOOP_HOME=/opt/hadoop-2.7.3

export CLASSPATH=$CLASSPATH:$($HADOOP_HOME/bin/hadoop classpath)

for i in /opt/hadoop-2.7.3/share/hadoop/tools/lib/*;
    do CLASSPATH=$i:"$CLASSPATH";
done
for i in /opt/hadoop-2.7.3/share/hadoop/common/lib/*;
    do CLASSPATH=$i:"$CLASSPATH";
done
for i in /opt/hadoop-2.7.3/share/hadoop/hdfs/lib/*;
    do CLASSPATH=$i:"$CLASSPATH";
done
for i in /opt/hadoop-2.7.3/share/hadoop/hdfs/*;
    do CLASSPATH=$i:"$CLASSPATH";
done
for i in /opt/hadoop-2.7.3/share/hadoop/common/*;
    do CLASSPATH=$i:"$CLASSPATH";
done

export CLASSPATH=$CLASSPATH

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/hadoop-2.7.3/lib/native:/opt/jdk1.8.0_281/jre/lib/amd64/server

export PATH=.:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:/opt/hadoop-2.7.3/lib/native:$JAVA_HOME/bin:$PATH
```

### 3、官方示例

```c
#include <hdfs.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv) {

    hdfsFS fs = hdfsConnect("hdfs://node1:9000/", 0);
    const char* writePath = "/in/testfile.txt";
    hdfsFile writeFile = hdfsOpenFile(fs, writePath, O_WRONLY |O_CREAT, 0, 0, 0);
    if(!writeFile) {
          fprintf(stderr, "Failed to open %s for writing!\n", writePath);
          exit(-1);
    }
    char* buffer = "Hello, World!";
    tSize num_written_bytes = hdfsWrite(fs, writeFile, (void*)buffer, strlen(buffer)+1);
    if (hdfsFlush(fs, writeFile)) {
           fprintf(stderr, "Failed to 'flush' %s\n", writePath);
          exit(-1);
    }
    hdfsCloseFile(fs, writeFile);
}
```

执行 `gcc main.c -I$HADOOP_HOME/include -L$HADOOP_HOME/lib/native -lhdfs -o main` 编译，执行 `./main.out` 启动。


更多内容参考：[https://blog.csdn.net/weixin_34010949/article/details/85903011](https://blog.csdn.net/weixin_34010949/article/details/85903011)


### 4、更多API测试

```c
// hdfsFileIsOpenForRead
int test01() {

    int flag;
    hdfsFS fs = hdfsConnect("hdfs://node1:9000/", 0);
    const char* filePath = "/in/testfile.txt";
    hdfsFile file = hdfsOpenFile(fs, filePath, O_RDONLY, 0, 0, 0);
    flag = hdfsFileIsOpenForRead(file);
    printf("%d\n", flag);

    hdfsCloseFile(fs, file);
    return flag;
}

// hdfsFileIsOpenForWrite
int test02() {

    int flag;
    hdfsFS fs = hdfsConnect("hdfs://node1:9000/", 0);
    const char* filePath = "/in/testfile.txt";
    hdfsFile file = hdfsOpenFile(fs, filePath, O_WRONLY, 0, 0, 0);
    flag = hdfsFileIsOpenForWrite(file);
    printf("%d\n", flag);

    hdfsCloseFile(fs, file);
}

// hdfsBuilderConnect
int test03() {

    int checkHDFS, disFlag;
    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    const char* filePath = "/in/testfile.txt";
    checkHDFS = hdfsExists(fs, filePath);
    printf("%d\n", flag);

}

// hdfsRead
int test04() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    const char* filePath = "/in/wc.txt";
    hdfsFile file = hdfsOpenFile(fs, filePath, O_RDONLY, 0, 0, 0);

    char* buffer = malloc(255);
    tSize num_readed_bytes = hdfsRead(fs, file, (void*)buffer, strlen(buffer)+1);

    printf("%s\n", buffer);

    hdfsCloseFile(fs, file);
    
}

// hdfsAvailable
int test05() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    const char* filePath = "/in/wc.txt";
    hdfsFile file = hdfsOpenFile(fs, filePath, O_RDONLY, 0, 0, 0);

    tSize num_available_bytes = hdfsAvailable(fs, file);

    printf("%d\n", num_available_bytes);

    hdfsCloseFile(fs, file);
    
}

// hdfsBuilderConnect
int test06() {

    struct hdfsBuilder *src_bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(src_bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(src_bld, 9000);
    hdfsFS srcFS = hdfsBuilderConnect(src_bld);

    struct hdfsBuilder *dst_bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(dst_bld, "hdfs://node5");
    hdfsBuilderSetNameNodePort(dst_bld, 9000);
    hdfsFS dstFS = hdfsBuilderConnect(dst_bld);

    int flag;
    const char* srcPath = "/in/wc.txt";
    const char* dstPath = "/in/";
    flag = hdfsCopy(srcFS,srcPath,dstFS,dstPath)

    printf("%d\n", flag);
    
}

// hdfsGetWorkingDirectory
int test07() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    char* workingDir = malloc(255);
    workingDir = hdfsGetWorkingDirectory(fs, workingDir, 255);

    printf("%s\n", workingDir);
    
}

// hdfsCreateDirectory
int test08() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    int flag;
    const char* Dir = "/in/test/";
    flag = hdfsCreateDirectory(fs, Dir);

    printf("%d\n", flag);
    
}

// hdfsGetPathInfo
int test09() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    hdfsFileInfo *dfi;
    const char* path = "/in/wc.txt";
    dfi = hdfsGetPathInfo(fs, path);


    printf("%s\n", dfi->mName);
    printf("%d\n", dfi->mSize);
    printf("%d\n", dfi->mReplication);
    
}

// hdfsListDirectory
// ???
// @return Returns a dynamically-allocated array of hdfsFileInfo
//         objects; NULL on error.
int test10() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    int i;
    int num = 3;
    int *numEntries = &num;
    // hdfsFileInfo *dfi[1000];
    hdfsFileInfo *dfi;
    const char* path = "/in/";
    dfi = hdfsListDirectory(fs, path,numEntries);

    printf("%s\n", dfi->mName);
    printf("%s\n", dfi->mOwner);
 
}

// hadoopReadZero
// zero-copy
int test11() {

    struct hdfsBuilder *bld = hdfsNewBuilder();
    hdfsBuilderSetNameNode(bld, "hdfs://node1");
    hdfsBuilderSetNameNodePort(bld, 9000);
    hdfsFS fs = hdfsBuilderConnect(bld);

    const char* inPath = "/in/wc.txt";
    const char* outPath = "/in/zero-copy.txt";

    hdfsFile file = hdfsOpenFile(fs, inPath, O_RDONLY, 0, 0, 0);

    struct hadoopRzOptions *opts = malloc(1024);
    hadoopRzOptionsSetSkipChecksum(opts, 100);
    hadoopRzOptionsSetByteBufferPool(opts, ELASTIC_BYTE_BUFFER_POOL_CLASS);
  
    int32_t maxLength = 512;
    struct hadoopRzBuffer* hrb;

    hrb = hadoopReadZero(file, opts, maxLength);

    // int32_t num_bytes = hadoopRzBufferLength(hrb);
    // printf("%d\n", num_bytes);

    const void* p;
    p = hadoopRzBufferGet(hrb);
    // printf("%s\n", p );

    hdfsFile outFile = hdfsOpenFile(fs, outPath, O_WRONLY |O_CREAT, 0, 0, 0);

    tSize num_written_bytes = hdfsWrite(fs, outFile, p, strlen(p)+1);
    if (hdfsFlush(fs, writeFile)) {
        fprintf(stderr, "Failed to 'flush' %s\n", writePath);
        exit(-1);
    }

    hdfsCloseFile(fs, inFile);
    hdfsCloseFile(fs, outFile);
}
```

理解zero-copy:

[https://www.zhifou.net/blogdetail/32](https://www.zhifou.net/blogdetail/32)

[https://blog.csdn.net/u013256816/article/details/52589524/](https://blog.csdn.net/u013256816/article/details/52589524/)