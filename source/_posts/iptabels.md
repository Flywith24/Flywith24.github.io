---
title: 【流水账】iptables 常用指令
date: 2019-08-29 10:10:35
categories: 
- ROM
tags: 
- ROM
- iptables
- framework
- 网络黑白名单
- 流水账
image: https://gitee.com/flywith24/Album/raw/master/img/20201015091556.png
---

> 写博客是个好习惯，但是写的人水平参差不齐，我见过最搞笑的就是博文的内容是其他博客的链接。本着不误人子弟的原则，我写博客一向很克制。流水账系列是我平时的一些记录，是 `how to` 类型的文章，网上相关的资料一搜一大把，仅供自己记录查找。
>
> 该文总结了 `iptables` 常用的指令。`iptables` 详细内容请看双印大佬 [iptables 详解系列](http://www.zsythink.net/archives/tag/iptables/page/2/)

<!-- more-->

## 规则查询
### 查看指定表中的规则
```shell
iptables -t filter -L
```

使用 `-t` 选项，指定要操作的表，使用 `-L` 选项，查看-t选项对应的表的规则， `-L` 选项的意思是，列出规则。所以，上述命令的含义为列出filter表的所有规则。

`filter` 可替换为 `raw` `mangle` `nat`

如果仅查看 `filter` 可以省略 `-t filter` ，当没有使用 `-t` 选项指定表时，默认为操作 `filter` 表，即 `iptables -L` 表示列出 `filter` 表中的所有规则。

### 查看指定表中的指定链的规则
```shell
iptables -L INPUT
```

只查看 `filter` 表中 `INPUT` 链的规则

```shell
iptables -vL INPUT
```

使用 `-v` 查看更多，更详细的信息

### 不让ip进行反解
```shell
 iptables -nvL
```

`iptables` 默认进行了名称解析，但是在规则非常多的情况下如果进行名称解析，效率会比较低。使用 `-n` 选项，表示不对 `ip` 地址进行名称反解，直接显示 `ip` 地址

只查看某个链的规则，并且不让 `ip` 进行反解，`iptables -nvL INPUT`

### 显示规则编号
```shell
iptables --line-number -nvL INPUT
或
iptables --line -nvL INPUT
```

## 规则管理
### 添加规则
<font color=#ff0000>注意：添加规则时，规则的顺序非常重要</font>
```shell
# 在指定表的指定链的尾部添加一条规则，-A选项表示在对应链的末尾添加规则
命令语法：iptables -t 表名 -A 链名 匹配条件 -j 动作
示例：iptables -t filter -A OUTPUT -s 192.168.10.225 -j DROP

# 在指定表的指定链的首部添加一条规则，-I选型表示在对应链的开头添加规则
命令语法：iptables -t 表名 -I 链名 匹配条件 -j 动作
示例： iptables -t filter -I OUTPUT -s 192.168.10.225 -j ACCEPT

# 在指定表的指定链的指定位置添加一条规则
命令语法：iptables -t 表名 -I 链名 规则序号 匹配条件 -j 动作
示例： iptables -t filter -I OUTPUT 3 -s 192.168.10.225 -j REJECT

# 设置指定表的指定链的默认策略（默认动作），并非添加规则
命令语法：iptables -t 表名 -P 链名 动作
示例：iptables -t filter -P FORWARD ACCEPT
```

### 删除规则
<font color="ff0000">注意：如果没有保存规则，删除规则时请慎重</font>

```shell
# 按照规则序号删除规则，删除指定表的指定链的指定规则，-D选项表示删除对应链中的规则
命令语法：iptables -t 表名 -D 链名 规则序号
示例：iptables -t filter -D INPUT 3

# 按照具体的匹配条件与动作删除规则，删除指定表的指定链的指定规则
命令语法：iptables -t 表名 -D 链名 匹配条件 -j 动作
示例：iptables -t filter -D INPUT -s 192.168.10.225 -j DROP

# 删除指定表的指定链中的所有规则，-F选项表示清空对应链中的规则
命令语法：iptables -t 表名 -F 链名
示例：iptables -t filter -F INPUT

# 删除指定表中的所有规则
命令语法：iptables -t 表名 -F
示例：iptables -t filter -F OUTPUT
```

### 修改规则
<font color="ff0000">注意：如果使用-R选项修改规则中的动作，那么必须指明原规则中的原匹配条件，例如源ip，目标ip等</font>
```shell
# 修改指定表中指定链的指定规则，-R选项表示修改对应链中的规则，使用-R选项时要同时指定对应的链以及规则对应的序号，并且规则中原本的匹配条件不可省略
命令语法：iptables -t 表名 -R 链名 规则序号 规则原本的匹配条件 -j 动作
示例：iptables -t filter -R INPUT 3 -s 192.168.10.225 -j ACCEPT
```
上述示例表示修改 `filter` 表中 `INPUT` 链的第3条规则，将这条规则的动作修改为 `ACCEPT` ， `-s`  `192.168.10.225` 为这条规则中原本的匹配条件，如果省略此匹配条件，修改后的规则中的源地址可能会变为 `0.0.0.0/0`

其他修改规则的方法：先通过编号删除规则，再在原编号位置添加一条规则

```shell
# 修改指定表的指定链的默认策略（默认动作），并非修改规则
命令语法：iptables -t 表名 -P 链名 动作
示例：iptables -t filter -P FORWARD ACCEPT
```

### 保存规则
```shell
# 保存规则命令，表示将iptables规则保存至/etc/sysconfig/iptables文件中
service iptables save
```

## 匹配条件

### 基本匹配条件

`-s` 用于匹配报文的源地址,可以同时指定多个源地址，每个 `ip` 之间用逗号隔开，也可以指定为一个网段

```shell
iptables -t filter -I INPUT -s 192.168.10.111,192.168.10.225 -j DROP
iptables -t filter -I INPUT -s 192.168.10.0/24 -j ACCEPT
iptables -t filter -I INPUT ! -s 192.168.10.0/24 -j ACCEPT
```

`-d` 用于匹配报文的目标地址,可以同时指定多个目标地址，每个 `ip` 之间用逗号隔开，也可以指定为一个网段

```shell
iptables -t filter -I OUTPUT -d 192.168.10.111,192.168.10.225 -j DROP
iptables -t filter -I INPUT -d 192.168.10.0/24 -j ACCEPT
iptables -t filter -I INPUT ! -d 192.168.10.0/24 -j ACCEPT
```

`-p` 用于匹配报文的协议类型,可以匹配的协议类型 `tcp` 、`udp` 、`udplite` 、`icmp` 、`esp` 、`ah` 、`sctp` 等

```shell
iptables -t filter -I INPUT -p tcp -s 192.168.10.146 -j ACCEPT
iptables -t filter -I INPUT ! -p udp -s 192.168.10.146 -j ACCEPT
```

### 扩展匹配条件

1. `tcp` 扩展模块

   `-p tcp -m tcp --sport` 用于匹配 `tcp` 协议报文的源端口，可以使用冒号指定一个连续的端口范围
   `-p tcp -m tcp --dport` 用于匹配 `tcp` 协议报文的目标端口，可以使用冒号指定一个连续的端口范围

    ```shell
    iptables -t filter -I OUTPUT -d 192.168.10.225 -p tcp -m tcp --sport 22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m tcp --dport 22:25 -j REJECT
    iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m tcp --dport :22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m tcp --dport 80: -j REJECT
    iptables -t filter -I OUTPUT -d 192.168.10.225 -p tcp -m tcp ! --sport 22 -j ACCEPT
    ```

2. `multiport` 扩展模块

   `-p tcp -m multiport --sports` 用于匹配报文的源端口，可以指定离散的多个端口号,端口之间用"逗号"隔开
   `-p udp -m multiport --dports` 用于匹配报文的目标端口，可以指定离散的多个端口号，端口之间用"逗号"隔开

   ```shell
   iptables -t filter -I OUTPUT -d 192.168.10.225 -p udp -m multiport --sports 137,138 -j REJECT
   iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m multiport --dports 22,80 -j REJECT
   iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m multiport ! --dports 22,80 -j REJECT
   iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m multiport --dports 80:88 -j REJECT
   iptables -t filter -I INPUT -s 192.168.10.225 -p tcp -m multiport --dports 22,80:88 -j REJECT
   ```

   

3. `udp` 模块

   `--sport` 匹配 `udp` 报文的源地址
   `--dport` 匹配 `udp` 报文的目标地址

   ```shell
   # 可以结合multiport模块指定多个离散的端口
   iptables -t filter -I INPUT -p udp -m udp --dport 137 -j ACCEPT
   iptables -t filter -I INPUT -p udp -m udp --dport 137:157 -j ACCEPT
   ```

   

4. `icmp` 模块

   `--icmp-type` 匹配 `icmp` 报文的具体类型

   ```shell
   iptables -t filter -I INPUT -p icmp -m icmp --icmp-type 8/0 -j REJECT
   iptables -t filter -I INPUT -p icmp --icmp-type 8 -j REJECT
   iptables -t filter -I OUTPUT -p icmp -m icmp --icmp-type 0/0 -j REJECT
   iptables -t filter -I OUTPUT -p icmp --icmp-type 0 -j REJECT
   iptables -t filter -I INPUT -p icmp --icmp-type "echo-request" -j REJECT
   ```

   

5. `iprange` 模块

   `--src-range` 指定连续的源地址范围
   `--dst-range` 指定连续的目标地址范围

   ```shell
   iptables -t filter -I INPUT -m iprange --src-range 192.168.10.127-192.168.10.146 -j DROP
   iptables -t filter -I OUTPUT -m iprange --dst-range 192.168.10.127-192.168.10.146 -j DROP
   iptables -t filter -I INPUT -m iprange ! --src-range 192.168.10.127-192.168.10.146 -j DROP
   ```

6. `stirng` 模块

   `--algo` 指定对应的匹配算法，可用算法为 `bm` 、`kmp`，此选项为必需选项。
   `--string` 指定需要匹配的字符串

   ```shell
   iptables -t filter -I INPUT -p tcp --sport 80 -m string --algo bm --string "baidu" -j REJECT
   iptables -t filter -I INPUT -p tcp --sport 80 -m string --algo bm --string "baidu" -j REJECT
   ```

   

7. `time` 模块 

   `--timestart` 用于指定时间范围的开始时间，不可取反
   `--timestop` 用于指定时间范围的结束时间，不可取反
   `--weekdays` 用于指定"星期几"，可取反
   `--monthdays` 用于指定"几号"，可取反
   `--datestart` 用于指定日期范围的开始日期，不可取反
   `--datestop` 用于指定日期范围的结束时间，不可取反

   ```shell
   iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --timestart 09:00:00 --timestop 19:00:00 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 443 -m time --timestart 09:00:00 --timestop 19:00:00 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time --weekdays 6,7 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time --monthdays 22,23 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time ! --monthdays 22,23 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time --timestart 09:00:00 --timestop 18:00:00 --weekdays 6,7 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time --weekdays 5 --monthdays 22,23,24,25,26,27,28 -j REJECT
   iptables -t filter -I OUTPUT -p tcp --dport 80  -m time --datestart 2017-12-24 --datestop 2017-12-27 -j REJECT
   ```

   

8. `connlimit` 模块

   `--connlimit-above` 单独使用此选项时，表示限制每个 `ip` 的链接数量。
   `--connlimit-mask`  此选项不能单独使用，在使用 `--connlimit-above` 选项时，配合此选项，则可以针对"某类 `ip` 段内的一定数量的 `ip` "进行连接数量的限制

   ```shell
   iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
   iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 20 --connlimit-mask 24 -j REJECT
   iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 10 --connlimit-mask 27 -j REJECT
   ```

   

9. `limit` 模块

   `--limit-burst` 此选项用于指定令牌桶中令牌的最大数量
   `--limit` 此选项用于指定令牌桶中生成新令牌的频率，可用时间单位有second、minute 、hour、day

   ```shell
   iptables -t filter -I INPUT -p icmp -m limit --limit-burst 3 --limit 10/minute -j ACCEPT
   iptables -t filter -A INPUT -p icmp -j REJECT
   ```

   

## 自定义链

### 创建自定义链
```shell
# 在filter表中创建yyz自定义链
iptables -N yyz
```
### 引用自定义链
```shell
# 在OUTPUT链中引用刚才创建的自定义链
iptables -I OUTPUT -j yyz
```
### 重命名自定义链
```shell
# 将yyz自定义链重命名为test
iptabels -E yyz test
```
### 删除自定义链
删除自定义链需要满足两个条件
1. 自定义链中没有被引用
2. 自定义链中没有任何规则

```shell
# 删除引用计数为0且不包含任何规则的test链
iptabels -X test
```