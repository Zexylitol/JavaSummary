# Linux命令

## cat

cat（英文全拼：concatenate）命令用于连接文件并打印到标准输出设备上

```shell
cat [-AbeEnstTuv] [--help] [--version] fileName
```

| 参数                    | 说明                               |
| :---------------------- | :--------------------------------- |
| -n 或 --number          | 由 1 开始对所有输出的行数编号      |
| -b 或 --number-nonblank | 和 -n 相似，只不过对于空白行不编号 |



1. 显示redis.conf文件中的内容，不显示注释和空格

```shell
cat redis.conf | grep -v "#" | grep -v "^$"
```

## ps

ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器

```shell
ps [options] [--help]
```



1. 查看redis-开头的所有进程

```shell
ps -ef | grep redis-
```

## pstree

pstree 命令以树状图的方式展现进程之间的派生关系

```shell
pstree [OPTIONS]
```

| 参数 | 说明                   |
| :--- | :--------------------- |
| -a   | 显示每个程序的完整指令 |
| -p   | 显示程序识别码         |

1. 显示系统当前所有进程的进程ID和进程号

   ```shell
   pstree -p
   ```

2. 计算进程号5240下的线程数

   ```shell
   pstree -p 5240 | wc -l
   ```

## netstat

Linux netstat 命令用于显示网络状态。

```shell
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

| 参数           | 说明                                     |
| :------------- | :--------------------------------------- |
| -a或--all      | 显示所有连线中的Socket                   |
| -n或--numeric  | 直接使用IP地址，而不通过域名服务器       |
| -p或--programs | 显示正在使用Socket的程序识别码和程序名称 |

1. 查看3306端口的网络状态

   ```shell
   netstat -anp | grep 3306
   ```

2. 显示详细的网络状况

   ```shell
   netstat -a
   ```

3. 查看系统所有的网络服务

   ```shell
   netstat -anp | more
   ```

4. 查看机器上各种TCP状态的连接数

   ```shell
   netstat -n| awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
   LAST_ACK 6
   SYN_RECV 77
   CLOSE_WAIT 5793
   ESTABLISHED 8540
   FIN_WAIT1 143
   FIN_WAIT2 378
   CLOSING 29
   TIME_WAIT 22960
   ```

   

## top

Linux top命令用于实时显示 process 的动态

```shell
top [选项]
```

| 参数    | 说明                                       |
| :------ | :----------------------------------------- |
| -d 秒数 | 指定top命令每隔几秒更新，默认是3秒         |
| -i      | 使top不显示任何闲置或者僵死进程            |
| -p PID  | 通过指定监控进程ID来仅仅监控某个进程的状态 |

| 交互操作 | 说明                  |
| :------- | :-------------------- |
| p        | 以CPU使用率排序，默认 |
| M        | 以内存使用率排序      |
| N        | 以PID排序             |
| q        | 退出top               |



1. 显示指定的进程信息

   ```shell
   top -p PID
   ```

2. 解析top输出

   ```shell
   top - 02:47:51 up 37 min,  1 user,  load average: 0.00, 0.00, 0.02
   Tasks: 242 total,   1 running, 163 sleeping,   0 stopped,   0 zombie
   %Cpu(s):  0.1 us,  0.5 sy,  0.0 ni, 99.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
   KiB Mem :  4015904 total,  1655948 free,   777556 used,  1582400 buff/cache
   KiB Swap:  4191228 total,  4191228 free,        0 used.  2836412 avail Mem 
   
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
     1213 root      20   0  389412  83488  33308 S   2.3  2.1   0:10.04 Xorg       
    13262 yzz       20   0  682280  44216  35540 S   1.3  1.1   0:02.09 gnome-ter+ 
    12968 yzz       20   0 1296116 123296  86824 S   0.3  3.1   0:10.12 compiz     
    13606 yzz       20   0   43648   4172   3456 R   0.3  0.1   0:00.26 top        
        1 root      20   0  185436   6000   4008 S   0.0  0.1   0:04.98 systemd    
        2 root      20   0       0      0      0 S   0.0  0.0   0:00.08 kthreadd   
        4 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kworker/0+ 
        6 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 mm_percpu+ 
        7 root      20   0       0      0      0 S   0.0  0.0   0:00.08 ksoftirqd+ 
        8 root      20   0       0      0      0 I   0.0  0.0   0:00.49 rcu_sched  
   ```

   - `top - 02:47:51 up 37 min,  1 user,  load average: 0.00, 0.00, 0.02`

     - 表示系统的运行时间和平均负载
     - 当前时间
     - 系统已运行的时间
     - 当前登录用户的数量
     - 相应最近5、10和15分钟内的平均负载

   - `Tasks: 242 total,   1 running, 163 sleeping,   0 stopped,   0 zombie`

     - 显示全部进程的数量，正在运行、睡眠、停止、僵尸进程的数量

   - `%Cpu(s):  0.1 us,  0.5 sy,  0.0 ni, 99.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st`

     - 表示CPU状态，显示了不同模式下所占CPU时间的百分比
     - us, user： 运行(未调整优先级的) 用户进程的CPU时间
     - sy，system: 运行内核进程的CPU时间
     - ni，niced：运行已调整优先级的用户进程的CPU时间
     - wa，IO wait: 用于等待IO完成的CPU时间
     - hi：处理硬件中断的CPU时间
     - si: 处理软件中断的CPU时间
     - st：这个虚拟机被hypervisor偷去的CPU时间（译注：如果当前处于一个hypervisor下的vm，实际上hypervisor也是要消耗一部分CPU处理时间的）

   - ````shell
     KiB Mem :  4015904 total,  1655948 free,   777556 used,  1582400 buff/cache
     KiB Swap:  4191228 total,  4191228 free,        0 used.  2836412 avail Mem 
     ````

     - 显示内存使用率, 第一行是物理内存使用，第二行是虚拟内存使用(交换空间)
     - 物理内存显示如下：全部可用内存、已使用内存、空闲内存、缓冲内存
     - 交换部分显示的是：全部、已使用、空闲和缓冲交换空间

   - ```shell
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
     1213 root      20   0  389412  83488  33308 S   2.3  2.1   0:10.04 Xorg  
     ```

     - PID：进程ID，进程的唯一标识符
     - USER：进程所有者的实际用户名
     - PR：进程的调度优先级。这个字段的一些值是’rt’。这意味这这些进程运行在实时态
     - NI：进程的nice值（优先级）。越小的值意味着越高的优先级
     - VIRT：进程使用的虚拟内存
     - RES：驻留内存大小。驻留内存是任务使用的非交换物理内存大小
     - SHR：进程使用的共享内存
     - S：进程的状态，有以下不同的值:
       - D – 不可中断的睡眠态。
       - R – 运行态
       - S – 睡眠态
       - T – 被跟踪或已停止
       - Z – 僵尸态
     - %CPU：自从上一次更新时到现在任务所使用的CPU时间百分比
     - %MEM：进程使用的可用物理内存百分比
     - TIME+：任务启动后到现在所使用的全部CPU时间，精确到百分之一秒
     - COMMAND：运行进程所使用的命令



## ls

ls（英文全拼：list files）命令用于显示指定工作目录下之内容

| 参数 | 作用                                                         |
| :--- | :----------------------------------------------------------- |
| -a   | 显示所有文件及目录 (**.** 开头的隐藏文件也会列出)            |
| -l   | 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出 |

## cp

cp（英文全拼：copy file）命令主要用于复制文件或目录

```shell
cp [options] source dest
```

| 参数 | 说明                                                         |
| :--- | :----------------------------------------------------------- |
| -a   | 此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容 |
| -d   | 复制时保留链接。这里所说的链接相当于Windows系统中的快捷方式  |
| -f   | 覆盖已经存在的目标文件而不给出提示                           |
| -i   | 与-f选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答"y"时目标文件将被覆盖 |
| -p   | 除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。 |
| -r   | 若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件 |
| -l   | 不复制文件，只是生成链接文件                                 |



1. 复制一份redis-6379.conf命名为redis-6380.conf到同一目录下

```shell
cp redis-6379.conf redis-6380.conf
```

## rm

 rm（英文全拼：remove）命令用于删除一个文件或者目录

```shell
rm [options] name...
```

| 参数 | 说明                                             |
| :--- | :----------------------------------------------- |
| -i   | 删除前逐一询问确认                               |
| -f   | 即使原档案属性设为唯读，亦直接删除，无需逐一确认 |
| -r   | 将目录及以下之档案亦逐一删除                     |

1. 删除当前目录下的所有文件及目录

```shell
rm -rf *
```

## kill

1. 彻底杀死进程

```shell
kill -9 PID
```

## tail

tail 命令可用于查看文件的内容

```shell
tail [参数] [文件]  
```

| 参数      | 说明                                  |
| :-------- | :------------------------------------ |
| -f        | 循环读取                              |
| -n<行数>  | 显示文件的尾部 n 行内容               |
| -v        | 显示详细的处理信息                    |
| --pid=PID | 与-f合用,表示在进程ID,PID死掉之后结束 |

1. 显示info.log后20行

```shell
tail -n 20 info.log
```

2. 把 filename 文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要 filename 更新就可以看到最新的文件内容

```shell
tail -f filename
```

3.  倒数100行并写入实时监听文件写入模式

```shell
tail -100f  xxx.log
```

## grep

Linux grep 命令用于查找文件里符合条件的字符串。

```shell
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

| 参数                | 说明                 |
| :------------------ | :------------------- |
| -i 或 --ignore-case | 忽略字符大小写的差别 |
| -c 或 --count       | 计算符合样式的列数   |

1. 日志查找

```shell
grep forest xxx.log  
grep forest xxx.log yyy.log #多日志查找
```

2. 管道过滤关键字

```shell
 cat xxx.log | grep -i 'keyword' 
```

## more

Linux more 命令类似 cat ，不过会以一页一页的形式显示，更方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能（与 vi 相似），使用中的说明文件，请按 h ，q 退出 more，= 输出当前行的行号

```shell
more [-dlfpcsu] [-num] [+/pattern] [+linenum] [fileNames..]
```

| 参数      | 说明                           |
| :-------- | :----------------------------- |
| -num      | 一次显示的行数                 |
| +num      | 从第 num 行开始显示            |
| fileNames | 欲显示内容的文档，可为复数个数 |

1. 从第 20 行开始显示 testfile 内容

```shell
more +20 testfile
```



## less

less 与 more 类似，less 可以随意浏览文件，支持翻页和搜索，支持向上翻页和向下翻页

```shell
less [参数] 文件 
```

| 参数        | 说明                                  |
| :---------- | :------------------------------------ |
| -N          | 显示每行的行号                        |
| b           | 向上翻一页                            |
| 空格键      | 滚动一页                              |
| 回车键      | 滚动一行                              |
| Q           | 退出less 命令                         |
| -o <文件名> | 将less 输出的内容在指定文件中保存起来 |

1. 查看文件

```shell
less info.log
```

2. ps查看进程信息并通过less分页显示

```shell
ps -ef | less
```

3. 查看命令历史使用记录并通过less分页显示

```shell
history | less
```

## mv

Linux mv（英文全拼：move file）命令用来为文件或目录改名、或将文件或目录移入其它位置。

```shell
mv [options] source dest
mv [options] source... directory
```

| 参数 | 说明                                                         |
| :--- | :----------------------------------------------------------- |
| -b   | 当目标文件或目录存在时，在执行覆盖前，会为其创建一个备份     |
| -i   | 如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。 |
| -f   | 如果指定移动的源目录或文件与目标的目录或文件同名，不会询问，直接覆盖旧文 |
| -n   | 不要覆盖任何已存在的文件或目录                               |

1. 将文件 aaa 改名为 bbb 

   ```shell
   mv aaa bbb
   ```

2. 将 /usr/runoob 下的所有文件和目录移到当前目录下

   ```shell
   mv /usr/runoob/* .
   ```

## df

Linux df（英文全拼：disk free） 命令用于显示目前在 Linux 系统上的文件系统磁盘使用情况统计

```shell
df [选项]... [FILE]...
```

1. 查看文件系统分区的容量（以可读的形式）

   ```shell
   df -h
   ```

## du

Linux du （英文全拼：disk usage）命令用于显示目录或文件的大小。

du 会显示指定的目录或文件所占用的磁盘空间

```shell
du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size][--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]
```

| 参数                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| -x或--one-file-xystem  | 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过 |
| -h或--human-readable   | 以K，M，G为单位，提高信息的可读性                            |
| --max-depth=<目录层数> | 超过指定层数的目录后，予以忽略                               |

1. 查看当前路径下的文件占用空间大小

   ```shell
   du -h -x --max-depth=1
   ```


## whereis

Linux whereis命令用于查找文件.

该指令只能用于查找二进制文件、源代码文件和man手册页，一般文件的定位需使用locate命令

1. 查看mysql的安装位置

   ```shell
   whereis mysql
   ```

## wc

Linux wc命令用于计算字数。

利用wc指令我们可以计算文件的**Byte数、字数、或是列数**，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

```shell
wc [-clw][--help][--version][文件...]
```

| 参数                 | 说明          |
| :------------------- | :------------ |
| -c或--bytes或--chars | 只显示Bytes数 |
| -l或--lines          | 显示行数      |
| -w或--words          | 只显示字数    |
| --help               | 在线帮助      |
| --version            | 显示版本信息  |

1. 在默认的情况下，wc将计算指定文件的行数、字数，以及字节数

```shell
$ wc testfile           # testfile文件的统计信息  
3 92 598 testfile       # testfile文件的行数为3、单词数92、字节数598 

$ wc testfile testfile_1 testfile_2  #统计三个文件的信息  
3 92 598 testfile                    #第一个文件行数为3、单词数92、字节数598  
9 18 78 testfile_1                   #第二个文件的行数为9、单词数18、字节数78  
3 6 32 testfile_2                    #第三个文件的行数为3、单词数6、字节数32  
15 116 708 总用量                    #三个文件总共的行数为15、单词数116、字节数708 
```



# 经典面试题

## 1. 统计文件中出现次数最多的前10个单词

使用Linux命令或者shell实现：文件words存放英文单词，格式为每行一个英文单词（单词可以重复），统计这个文件中出现次数最多的前10个单词。

```shell
cat words.txt | sort | uniq -c | sort -k1,1nr | head -10
```

- `sort`: 对单词进行排序

- `uniq -c`: 显示唯一的行，并在每行行首加上本行在文件中出现的次数

- `sort -k1,1nr`: 按照第一个字段，数值排序，且为逆序

- `head -10`: 取前10行数据

## 2. 查看文件有多少行

```shell
wc -l filename # 就是查看文件里有多少行
wc -w filename # 看文件里有多少个word
wc -L filename # 文件里最长的那一行是多少个字。
```



# vim命令

1. 删除整行

```shel
dd
```

