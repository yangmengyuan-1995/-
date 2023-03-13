### 1、 shell基础

#### 1.1 shell相关常用命令

- pstree： 查看进程树
- cat /etc/shells : 查看系统拥有的解释器

#### 1.2 标准的脚本构成

- 1. 声明解释器

  - #！ /bin/bash

- 2. 注释

- 3. 具体代码

#### 1.3 脚本执行方式

- 1. 赋予脚本文件x执行权限，然后使用绝对路径或相对路径运行该文件
- 2. 使用解释器直接执行脚本，即使没有x权限
     - bash xxx.sh  含义： 新开启一个解释器执行脚本，脚本执行结束后，新开启的bash自动退出
- 3. 使用source命令执行脚本
     - source xxx.sh  含义： 不开启新解释器，使用当前解释器执行脚本
     - 可以简写为  .  xxx.sh

### 2、shell变量

####  2.1 变量分类

- 自定义变量

  -  定义变量： 变量名称=10  例子： a=10
  -  调用变量： $变量名称    例子: echo $a
  -   取消变量定义： unset  变量名称  例子： unset a
  - 变量和常量字符混合使用： ${变量名称}常量字符  例子: ${a}RMB

- 环境变量

  - echo $USER   查看当前用户名
  - echo $UID   查看当前用户id号 
  - echo $HOME  查看家目录
  - echo $PATH  查看命令程序存放路径
  - echo  $PS1  查看一级提示符
  - echo $PS2 查看二级提示符

- 位置变量

  - $1 $2 $3 ....     在脚本中定义，在脚本执行时按顺序传入

- 预定义变量

  - $$     显示脚本运行时的进程号
  - $#     显示脚本运行时位置变量的数量
  - $*     显示脚本运行时的所有位置变量
  - $?     查看上一条指令是否成功
  - $0     查看执行的脚本的名称
  - $!      查看最后一个放在后台的进程的进程号
    - sleep 100 &    将sleep命令放在后台运行
    - echo $!  可以查看sleep的进程号

- 常用命令：

  - env  ：查看所有环境变量

  - set   ：查看所有变量

    

#### 2.2 引号的作用

- 引号的作用：

  - ""  ：双引号，用于界定范围

    ```shell
    touch  a  b  # 系统会认为a和b是分开的，会创建出a和b两个文件
    touch "a b" # 系统会创建出a b这一个文件
    ```

  

  - ''  ： 单引号， 用于界定范围，但是会屏蔽特殊符号(会把变量调用识别为普通字符串)

    ```shell
    a=10
    echo "$a"  # 系统会把双引号内的$a识别为变量调用，输出a变量的值 10
    echo '$a' # 系统会把单引号内的$a识别为字符串，输出 $a
    ```

    

  - `` ：反撇号或$() ：可以将命令执行的标准输出作为字符串存储，因此称为命令替换

    ```shell
    echo date  # 系统会把date识别为普通字符串， 输出  date
    echo `date` # 系统会先执行date命令，然后将结果输出
    echo $(date) # 同反撇号
    ```

#### 2.3 终端回显控制

- read -p : 脚本执行时显示提示信息

  ```shell
  #!/bin/bash
  read -p "请输入用户名：" u
  useradd $u
  read -p "请输入密码：" p
  echo $p | passwd --stdin $u
  ```

  

- stty -echo： 关闭终端回显

- stty echo：打开终端回显

- 结合上面两条命令，修改脚本。让用户输入的密码不要在终端显示出来

  ```shell
  #!/bin/bash
  read -p "请输入用户名：" u
  useradd $u
  stty -echo  # 用户输入密码前关闭终端回显
  read -p "请输入密码：" p
  stty echo  # 用户输入密码后打开终端回显
  echo $p | passwd --stdin $u
  ```

#### 2.4 定义全局变量

- export ：提升局部变量为全局变量

  - expory a=10

- export -n 变量名：将全局变量降为局部变量

  - export -n a 

  ```shell
  a=10 # 局部变量
  echo $a  # 输出:10 
  bash  # 开启新的解释器
  echo $a  # 无输出，新的解释器获取不到之前解释器定义的值
  exit # 退出新解释器
  export b=10 # 创建新变量，并发布成全局变量
  bash # 开启新的解释器
  echo $b # 输出：10，因为b已经成为全局变量
  exit # 退出新解释器
  export -n b # 撤销全局变量，恢复局部变量(一定要回到创建变量的解释器环境才可以)
  bash # 开启新的解释器
  echo $b # 无输出
  ```

  

### 3、 shell数值运算

  shell 中有4种数值运算方式

#### 3.1 expr

- 计算结果会打印在控制台

- expr后要有一个空格， 运算符两侧要有一个空格

- 乘法运算符号*需要转义后再使用

  ```shell
  expr 1 + 1
  expr 2 - 1
  expr 2 \* 2  # 因为shell中*是代表匹配所有的通配符，想要让它进行乘法运算，需要将其转义
  expr 2 '*' 2 # 也可以使用单引号将*号括起来，取消它的特殊含义
  expr 4 / 2
  expr 10 % 3 # 取余数
  a=10
  expr $a + $a # 变量计算
  ```

  

#### 3.2 $[]

- 计算结果不会打印，要打印计算结果需要配合echo使用

- $[]内运算符两侧可以无空格

- $[]内进行乘法运算时不需要将*进行转义，可直接使用

- $[]内可以直接通过变量名称引用变量，不需使用"$变量名称"的方式来引用

- 它还有一种写法是：$(())

  ```shell
  echo $[1+1] # 与expr不同的是，$[]不会将计算结果显示在控制台，所以要配合echo使用
  echo $[1-1]
  echo $[1*1] # $[]的乘法运算不需要对*号进行转义
  echo $[1/1]
  echo $[10%3]
  a=10
  echo $[a*a] # 变量计算，$[]中直接写变量名称即可，不需再加$符号
  ```



#### 3.3 let

- 可以改变变量本身的值，不显示结果

  ```shell
  a=10
  let a=a+1
  echo $a  # 输出11
  let a=a++
  echo $a  # 输出12
  ```

  

#### 3.4 非交互式bc

- 因为bc计算器是交互式的，不适用于脚本内，所以可以通过echo加 | 的方式，将要计算的数据传递给bc，避免交互式界面

- bc计算器的好处是可以计算小数，前3种方式都不支持计算小数

  ```shell
  echo "1+2" | bc  # 使用bc进行非交互式计算。输出为：3
  echo "1.1 + 10" | bc # 可以计算小数。输出为：11.1
  echo "1+2;2+3" | bc # 多条计算命令中间用 ; 分隔。输出为： 3  5
  echo "10/3" | bc # 不定义小数点后长度，默认取整。输出为：3
  echo "scale=3;10/3" | bc # 定义小数点后长度为3为。输出为：3.333
  
  ```

  

### 4、语法基础

#### 4.1 条件测试的基本用法

- 语法格式

  - 使用 "test 表达式" 或者[ 表达式 ]都可以,使用中括号时，表达式两边至少要留一个空格。
  - 条件测试操作本身不显示除任何信息。测试的条件是否成立主要体现在命令执行后的返回状态。可以通过$?的值来做出判断，或者结合&& || 等逻辑操作显示出结果

- 字符串比较

  - 等于用 == 比较 
  - 不等于用  != 比较   

  - 运算符两边至少要留一个空格

  - -z 用来分析变量是否有值

  - -n 也可以分析变量是否有值。但变量两侧要加双引号

    ```shell
    a=
    [ -z "$a" ]
    echo $?  # 输出0，为真
    a=10
    [ -z "$a" ]
    echo $? # 输出1，为假
    ```

#### 4.2 逻辑运算符

- 逻辑运算符

  - &&   且   前边的任务执行成功，才会执行后面的任务
  - ||    或   前边的任务执行失败，才执行后面的任务
  - ;  前面的任务执行完毕后, 继续执行后面的任务，前后无逻辑关系

- 整数比较

  - -eq  等于
  - -ne  不等于
  - -gt   大于
  - -ge  大于等于
  - -lt    小于
  - -le   小于等于

- 案例

  - 编写脚本，每2分钟检查系统登录的用户数量，如果超过3个人，则发邮件给管理员报警

    ```shell
    
    #!/bin/bash
    # 检查系统登录的用户数量
    # 可以用who命令列出当前登录用户
    # 把who命令的结果交给wc -l 去算出有几行
    users=`who | wc-l`
    [ $users -gt 3 ] && echo "当前登录用户大于3人" | mail -s test root
    
    ```

#### 4.3 识别文件/目录的状态

- -e 判断对象是否存在(不管是目录还是文件)
- -d 判断对象是否为目录(存在且是目录)
- -f 判断对象是否为文件(存在且是文件)
- -r  是否可读(对root无效) 
- -w 是否可写(对root无效) 
- -x  是否可执行

#### 4.4 if判断

```shell
#!/bin/bash
# if 单分支
if 条件测试 ; then
    命令序列
fi

# if 双分支
if 条件测试 ; then
    命令序列
else
    命令序列
fi

# if 多分支
if 条件测试 ; then
    命令序列
elif 条件测试 ; then
    命令序列2
elif 条件测试 ; then
    命令序列3
...
else 
    命令序列X
fi
```

#### 4.5 for 循环

```shell
for 变量名 in 值列表
do
  命令序列
done
```

#### 4.6 while 循环

```shell
while 条件测试
do
  任务列表
done
```

#### 4.7 case分支

```shell
case 变量 in
模式1)
    命令序列1;;
模式2)
    命令序列2;;
*)
    命令序列3
esac
```



### 5、 Nginx相关脚本

#### 5.1 Nginx安装脚本

- 需要先下载好tag包，放在/opt目录下

  ```shell
  #!/bin/bash
  # 安装依赖
  yum -y install gcc openssl-devel prce-devel
  tar -xf /opt/nginx-1.12.2.tar.gz
  cd /opt/nginx-1.12.2
  ./configure
  make 
  make install
  ```

- 安装过程中未指定安装目录，所以nginx默认安装在 /usr/local/nginx/目录下

  ```
  cd /usr/local/nginx/sbin
  # 启动nginx
  ./nginx
  ```

  

#### 5.2 Nginx控制脚本

- ```shell
  #!/bin/bash
  case $1 in 
  start)
      /usr/local/nginx/sbin/nginx;;
  stop)
      /usr/local/nginx/sbin/nginx -s stop;;
  restart)
      /usr/local/nginx/sbin/nginx -s stop
      /usr/local/nginx/sbin/nginx;;
  *)
      echo "st|stop"
  esac
  ```



### 6、 Shell字符串处理

#### 6.1 字符串截取

- ${var: start :length}

  ```shell
  a=abc
  echo ${a:0:2}  # ab
  echo ${a:1:2}  # bc
  
  #/bin/bash
  a=abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789
  for i in {1..8}
  do
  x=$[ RANDOM%62 ]
  pass=${a:x:1}
  pass2=$pass2$pass
  done
  
  ```

#### 6.2 字符串替换

- `${var/old/new} `：只会替换第一个匹配到的旧字符

- `${var//old/new}`：会替换所有匹配到的旧字符

  ```shell
  str=aabbcc
  echo ${str/aa/88} # 88bbcc
  echo ${str/a/8} # 8abbcc  替换时只会替换第一个匹配的字符，并不会全部替换
  # 如何替换所有匹配到的字符?
  str2 = aabbbbcc
  echo ${str2//b/8}  # aa8888cc
  ```

#### 6.3 字符串删除

- `${var#*关键词}`：从左往右删除第一个字符到第一个关键词位置的字符

- `${var##*关键词}`：从左往右删除第一个字符到最后一个关键词位置的字符

- `${var%关键词*}`：从右往左删除第一个字符到第一个关键词的位置的字符

- `${var%%关键词*}`：从右往左删除第一个字符到最后一个关键词位置的字符

- 即： 

  - `#`号代表从左往右删除，一个代表删除到第一个关键词,两个代表删除到最后一个关键词
  - 因为`#`号是从左往右删, 要删除关键词左边所有的字符,需要用`*`号代替,所以`*`号要写在关键词前边
  - `%`号代表从右往左删除，一个代表删除到第一个关键词，两个代表删除到最后一个关键词
  - 因为`%`号是从右往左删, 要删除关键词右边所有的字符,需要用`*`号代替,所以`*`号要写在关键词后边
- 删除时会包含关键词在内一起删除

### 7、sed用法

- sed ： 流式编辑器，可以非交互式修改文本，逐行操作

#### 7.1 使用方法

- 前置命令 | sed 选项 (定址符) 指令
- sed | 选项 (定址符) 指令 文本



#### 7.2 选项

- -n :  屏蔽默认输出
- -r ：支持扩展正则
- -i ：写入文件

#### 7.3 指令

- p ： 输出指定内容
- d ： 删除指定内容
- s ：替换指定内容 ： sed 's/old/new/'
- a :  在行后追加一行
- i ：在行前添加一行
- c ：替换行

```shell
df | sed -n 'p' # 输出df指令生成的文本

cd /opt # 切换到opt目录
head -5 /etc/passwd txt # 将passwd前五行复制为新文件，文件名为txt

# p: 打印指定内容
# 使用行号当定址符
sed 'p' txt # 打印txt文件内容， 但是sed默认会把每一行打印两遍
sed -n 'p' txt # 使用-n屏蔽默认输出，这样打印出来的内容只有一遍
sed -n '1p' txt # 打印txt文件的第一行
sed -n '1, 3p' txt # 打印txt文件的第一行到第三行
sed -n '1, +4p' txt # 打印txt文件的第一行, 以及后边4行
sed -n '1p; 3p' txt # 打印txt文件的第一行；第三行
sed -n '1~2p' txt # 打印txt文件从第一行后，间隔2行才输出(即1,3,5...)

# 使用正则当定址符(指令前输入//， 两个斜杠之间写正则表达式)
sed -n '/^root/p' txt # 打印txt文件内以root开头的行
sed -n '/bash$/p' txt # 打印txt文件内以bash结尾的行
sed -n '=' txt # 打印txt文件的每一行的行号
sed -n '$=' txt # 打印txt文件的最后一行的行号(即总行数)

# d: 删除指定内容(因为未使用 -i 选项, 所以并不会真的删除文件内容, 只是把删除后的内容打印出来)
sed '1d' txt # 删除txt文件第一行
sed '1, 3d' txt # 删除txt文件第一行到第三行
sed '3, +2d' txt # 删除txt文件第三行，以及后边两行
sed '3d; 5d' txt # 删除txt文件第三行；第五行
....
sed -i '5d' txt # 删除txt文件第五行，并写入文件
sed '/^root/d' txt # 删除txt文件以root开头的行
sed -r '/bash|nologin/d' txt # 删除txt文件所有包含bash或nologin的行
sed '$d' txt # 删除txt文件最后一行
sed '/^$/d' txt # 删除txt文件中的空行 

# s ：替换指定内容 ： sed 's/old/new/'
vim txt2
2017 2011 2018
2017 2017 2020
2017 2017 2017
sed 's/2017/2019' txt2  # 为txt2文件的每一行执行替换操作，将2017替换为2019。但是只会替换每行的第一个结果
sed 's/2017/2019/g' txt2 # 将txt2文本中所有的2017替换为2019
sed 's/2017/2019/3'  # 将txt2文本中每行的第3个2017替换为2019
sed '3s/2017/2019/2;3s/2017/2019/2' txt2  # s前加行号可以替换指定行, 需要替换多个值可以用;分割。 但是要注意替换后的字符的索引下标

sed '1s#/bin/bash#/sbin/sh#' txt  # 当要替换的字符串或目标字符串中包含/符号时， 可以使用其它的特殊符号(例如# $ % 等)来分割
sed -n 's/2018/2019/p' txt2  #  只打印有替换动作的行


vim txt5
AAA
BBB
CCC
sed 'a 888' txt5
```



#### 7.4 综合练习

```shell
vim txt3
aaa b7bb CCC
xxx y9yy ZZZ

# 1. 删除每一行第二个字符和最后一个字符
sed 's/.//2;s/.$//' txt3
# 2. 删除每一行中的数字
sed 's/[0-9]//g' txt3
# 3. 将每一行的首尾字母互相调换
sed -r 's/^(.)(.*)(.)$/\3\2\1' txt3  # 使用扩展正则将字符串分割为首字母、中间部分、最后一个字母
                                     # 然后使用\符号加索引号调用
# 4. 为每一个大写字母添加()
sed -r 's/([A-Z])/(\1)/g' txt3

# 5. 编写脚本， 实现vsftpd服务装包配置启服务的全过程，开启上传功能

#! /bin/bash
yum -y install vsftpd &> dev/null
sed -i 's/^#anon_u/anon_u/' /etc/vsftpd/vsftpd.conf
systemctl restart vsftpd
systemctl enable vsftpd
chmod o+w /var/ftp/pub
setenforce 0
systemctl stop firewalld
```

### 8、 awk

#### 8.1 使用方法

- 主要方法：
  - 格式1：前置命令 | awk [选项]  '[条件] {指令}'
  - 格式2：awk [选项] '[条件] {指令}'  文件

#### 8.2 基础用法

- 内置变量
  - $1 $2 $3 ....  行被分割后的列
  - NR ：行数
  - NF：列数
- 常量 
  - 需要用"" 括起来
- 其它
  - awk print 里 使用, 有空格效果。直接使用空格无效
- 正则
  - awk '/正则表达式/{print}' 文件名 ： 使用正则筛选文件内的行

```shell
vim awk_txt1
hello the world

awk '{print $1, $3}' awk_txt1  # awk默认以空格将一行分割为多列，使用$+数字，获取到分割后的列

head -1 /etc/passwd/ > awk_txt2
awk -F: '{print $5}' awk_txt2  # -F: 指定:作为分隔符，将一行分割为多列
                               # {print}  打印
                               # $5  第五列
awk -F[:/] '{print $9}' awk_txt2  # -F[:/] 指定: 或/ 为分割符，当两个符号之间没有其它字符时，awk                                     也会从中间分割开来
awk -F:/ '{print $2}' awk_txt2  # -F:/ 指定:/ 为分隔符

# 使用内置变量NR、NF以及常量
awk -F[:/] '{print "当前行有"NR"行, "NF"列"}' awk_txt2

# 查看网卡发送及接受的流量
ifconfig ens33 | awk '/RX p/{print "ens33网卡接受的流量是 " $5 "字节"}'
ifconfig ens33 | awk '/TX p/{print "ens33网卡发送的流量是 " $5 "字节"}'

```

#### 8.3 awk的处理时机

- awk会逐行处理文本，支持在处理第一行之前做一些准备工作，以及在处理完最后一行之后做一些总结性质的工作。在命令格式上分别体现如下
  - awk [选项] '[条件]{指令}'  文件
  - awk [选项] 'BEGIN{指令}{指令} END{指令}' 文件
    - BEGIN{ } 行前处理，读取文件内容前执行，指令执行1次
    - { } 逐行处理， 读取文件过程中执行，指令执行n次
    - END{ } 行后处理，读取文件结束后执行，指令执行1次 
  
- 练习：格式化输出passwd文件中的User、UID、Home 字段

  ```shell
  head -5 /etc/passwd > pwd_txt
  awk -F: 'BEGIN{print "User\tUID\tHome"} {print $1"\t"$3"\t"$6} END{print "总计：",NR,"行"}' pwd_txt
  ```

  

#### 8.4 awk处理条件

