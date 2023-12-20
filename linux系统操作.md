### 1、安装命令

#### pacman命令

##### 安装

```shell
pacman -S package_name1 package_name2 ...
```

正则表达式安装

```shell
pacman -S $(pacman -Ssq package_regex)
```

用花括号扩展相似的包

```shell
pacman -S plasma-{workspace{,-wallpapers},pa}
```

##### 删除

不删除依赖

```shell
pacman -R<options> package_name
```

###### options:

- s: 同时删除不被其他包使用的依赖。
- sc: 删除软件包和所有依赖这个软件包的程序。（递归的删除）
- dd: 删除一个被其他软件包依赖的软件包，但是不删除依赖这个软件包的其他软件包。（会导致其他软件寄掉，乃至系统）
- n: 不备份配置文件

##### 升级

```shell
pacman -Syu
```

##### 查询

*pacman* 使用 `-Q` 参数查询本地软件包数据库， `-S` 查询同步数据库，以及 `-F`查询文件数据库。

查询已安装的软件包

```shell
pacman -Qs string1, string2...
```

显示依赖树，pactree在pacman-contrib包中

```shell
pactree package_name
```





#### apt安装命令

##### options:

- -h 帮助
- -y 安装过程自动选择yes
- -q 不显示安装过程

##### command:

- update 列出可更新的软件清单
- upgrade 升级软件包
- update [package_name] 升级指定的包
- install [package1] [package2] 安装软件包
- remove [package_name] 删除软件包
- lis --installed 列出所有已安装的包

### 2、文件操作

##### options

###### -r: 递归操作

###### -f: 强制操作

######   : 模糊匹配

​	例：'rm txt'删除所有txt文件

###### -p: 多级目录

#### ls列出目录

##### options:

###### -a: 显示全部文件

###### -d: 仅列出目录本身

###### -l: 列出详细信息

​	例drwxrwxr-x 2 chris chris 4096 3月 10 14:48 ddd

​	 1.第一个字母表示文件类型 

​		. 隐藏文件

​		d 目录文件

​		l 快捷方式

​	2.后续九个符号表示所有者权限，所有者所属组权限，其他		用户权限.

​		r 可读

​		w 可写

​		x 可执行

​		\- 无该权限

 3. "2"表示有几个文件夹或文件硬链接的个数
 4. chris 所有者
 5. chris 所有者所属的组
 6. 4096  如果是文件夹则显示4096。如果是文件则显示文件大小。
 7. 3月 10 14:48 修改时间
 8. ddd 文件(夹)名

-h 以KGMT形式显示

#### cd 更改目录

#### pwd 显示当前工作目录

#### mkdir [name] 创建文件夹

mkdir [directory_name]

##### options:

###### -p 多级目录

​	例：mkidir -p xxx/sss/ddd/ppp

#### touch 创建一个空文件

touch [filename]

#### rmdir 移除空文件夹

#### rm  移除文件或文件夹

##### options:

###### -r 递归删除

###### -f 强制删除

#### cp 复制文件(夹)

cp 原文件 路径

#### mv 移动文件(夹)

#### cat 从上到下查看文件

#### tac 从下到上查看文件

#### head -n 查看前n行

#### tail -n 查看后n行

#### nl 显示行号及内容

#### more 查看一屏，自动退出

- q 退出查看
- Enter 下翻一行
- 空格 下翻一屏

#### less 查看一屏，不自动退出

### 3、vim的使用

vim [filename]

#### 命令模式

| 命令          | 说明                  |
| ------------- | --------------------- |
| shift+z+z(ZZ) | 保存退出              |
| shift+z+q(ZQ) | 不保存退出            |
| dd            | 删除一行数据          |
| ndd           | 删除n行数据           |
| u             | 撤销                  |
| yy            | 复制                  |
| nyy           | 复制n行               |
| p             | 粘贴                  |
| G             | 定位到最后一行        |
| gg            | 定位到第一行          |
| ngg           | 定位到第n行           |
| $             | 定位到行末            |
| 0             | 定位到行首0           |
| k             | 上                    |
| j             | 下                    |
| h             | 左                    |
| l             | 右                    |
| ctrl + f      | 下翻一页              |
| ctrl + b      | 上翻一页              |
| ctrl + d      | 下翻半页              |
| ctrl + u      | 上翻半页              |
| x             | 删除光标右边的字符    |
| nx            | 删除光标右边n个字符   |
| X             | 删除光标左边的字符    |
| nX            | 删除光标左边的n个字符 |
| ctrl + r      | 反撤销                |

#### 插入模式

| 按键 | 功能                           |
| ---- | ------------------------------ |
| i    | 进入插入模式                   |
| I    | 在第一个非空字符前进入插入模式 |
| esc  | 退出插入模式，进入命令模式     |
| a    | 在光标右侧插入                 |
| A    | 在光标所在行结尾插入           |
| s    | 删除光标所在位置的文字并插入   |
| S    | 删除光标所在行的文字并插入     |
| o    | 在光标下一行插入               |
| O    | 在光标所在上一行插入           |

#### 底线命令模式

| 命令                   | 功能                                                |
| ---------------------- | --------------------------------------------------- |
| :                      | 进入底线命令模式                                    |
| w                      | 保存                                                |
| q                      | 退出                                                |
| q!                     | 强制退出                                            |
| wq                     | 保存并退出                                          |
| e!                     | 放弃之前的修改                                      |
| /[内容]                | 搜索内容                                            |
| %s/原内容/新内容[/g]   | 所有行内容替换，g表示全局(默认只能替换一行中第一处) |
| m,ns/原内容/新内容[/g] | 从m行到第n行进行替换                                |
| set nu                 | 显示行数                                            |

### 4、用户操作

#### whoami 查看当前登录的用户名

#### su <用户名> 切换用户，不指定用户时默认切换到root用户

#### useradd <用户名>

##### options:

###### -d 指定家目录

###### -m 创建用户时也创建家目录

###### -s 指定shell，一般指定为/bin/bash

#### userdel <用户名> 删除用户

##### options:

###### -r 删除用户家目录

#### passwd <用户名> 设置用户的密码如没有指定用户则修改当前用户的密码

#### visudo = vim /ect/sudoers 修改sudoers文件，如给某用户添加sudo权限

#### groups <用户名> 查看用户或当前用户所在的分组

#### gpasswd 将用户添加到一个分组，或者从一个分组中删除

##### options

###### -a <用户名> <组名> 添加

###### -d <用户名> <组名> 删除

### 5、文件(夹)权限

#### chmod 用来修改权限

##### 1、参数

o:其他

u:所有者

g:所属组

a:全部

例:chmod o+w demo.txt 给其他用户添加对demo.txt的写入权限

chmod o=rwx demo.txt

##### 2、权限值

r  4

w  2

x  1

例:chmod 777 demo.txt 权限全开放

#### chgrp <组名> <文件名> 修改所属组

#### chown <用户名> <文件名> 修改	所有者

### 6、压缩解压

#### zip压缩

可以压缩文件夹

##### zip <目标文件> <原文件> 压缩为.zip格式。

例: zip a.zip 1.txt

##### unzip 压缩文件名 解压

#### gzip压缩

##### gzip <文件名> 压缩。

例: gzip 1.txt 用1.txt.gz替换1.txt

##### gunzip <压缩文件名> 解压

例: gunzip 1.txt.gz

##### options:

###### -k 保留原文件

###### -r 递归压缩文件夹里每个文件

#### bzip2压缩

不能压缩文件夹，用法与gzip类似

#### tar 打包命令

##### options:

###### -c 打包

###### -x 拆包

###### -t 不拆包查看内容

###### -f 指定文件

###### -v 显示过程

###### -z 使用gzip压缩解压.tgz格式

例: tar -zcvf test.tgz test 将test文件夹打包为.tgz压缩文件

例: tar -zxvf test.tgz 解压test.tgz

###### -j 使用bzip2压缩解压.tbz格式

