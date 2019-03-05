### 常用命令

#### ls
用来显示目标列表

- >ls -l 文件名，文件类型、权限模式、硬连接数、所有者、组、文件大小和文件的最后修改时间
- ll按时间排序
    - >ll -t 降序
    - >ll -t|tac 升序
- ls
    - > -s 列出大小
    - > -S 按大小降序
    - > -r 反转
> ls -l --time-style='+%Y-%m-%d %H-%M-%S'

#### tail
用于列出输入文件中的尾部内容,默认显示最后10行

- >-f 显示文件最新追加内容
- >-c <N>  输出文件尾部N个字节的内容,N可以有后缀b(512字节),k,m
- >tail -n +20 file 输出从第20行至末尾的内容

#### netstat
用来打印Linux中网络系统的状态信息

- > -a或--all 显示所有连线中的Socket
- > -n或--numeric：直接使用ip地址，而不通过域名服务器
- > -p或--programs：显示正在使用Socket的程序识别码和程序名称

#### find
用来在指定目录下查找文件

linux中文件的时间有三种:
> mtime：文件最后内容修改时间。  
> ctime：文件最后属性改变时间，也可以叫status。可以使用-c代替。  
> atime：文件最后被读取时间，也可以叫access，use。可使用-u代替。

- > find -type f -ctime -1|xargs ls -l 列出1天内最后属性改变的文件
- > find -type f -mtime +5 -exec ls -l {} \; 列出5分钟以前内容修改的文件
- 时间之前不带符号刚好5天之前的文件
- exec 可以换成ok,会给出提示
```
注意一定是{} \;别漏掉了分号。{}是指find . -type f -mtime +5的返回的结果，\;是指命令尾部标记
```

#### xargs与exec,ok
- exec对每一个结果执行一次命令
- xargs全部塞给命令
- xargs可以通过-n来控制每次传递参数的个数