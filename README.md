# -PoC-python-
记录一次 vsftpd 2.3.4 “笑脸后门”漏洞检测实验

最近在本地搭了个漏洞靶场，用 Python 写了个小脚本，检测 vsftpd 2.3.4 的“笑脸后门”（CVE-2011-2523）。下面把整个过程整理一下，方便以后回看，也分享给有需要的朋友。

一、漏洞简介

vsftpd 2.3.4 版本被植入了后门：当用户登录 FTP 时，如果用户名中包含 :) 这个笑脸符号，服务器就会在 6200 端口悄悄开一个 root 权限的 Shell。攻击者直接连上去就能拿到最高权限。
检测思路：先用带 :) 的用户名尝试登录，然后看 6200 端口有没有被打开。

二、实验环境

攻击机：Kali 虚拟机

靶机：Metasploitable 2 虚拟机（自带 vsftpd 2.3.4 漏洞）

网络模式：两台虚拟机都使用 NAT 模式，保证在同一网段

工具：Kali 自带的 Python3、nano 编辑器、nc

三、测试过程

在虚拟机软件中导入并启动 Metasploitable 2。

用默认账号登录靶机：用户名 msfadmin，密码 msfadmin。

执行 ifconfig，找到 eth0 的 inet addr，记下靶机 IP，例如 192.168.152.128/24。

注意：/24 是子网掩码位数，不是 IP 的一部分，ping 时要去掉。

回到 Kali，执行：ping 192.168.152.128

看到 64 bytes from ... 的回复，说明两台机器网络互通。另外，ping 默认会一直刷，可以按 Ctrl + C 停止。

<img width="546" height="104" alt="ee39a5f4e3c5e9334d90f3d64b8c4f40" src="https://github.com/user-attachments/assets/6e2b7668-5ca5-44be-abe6-99ca9982ddab" />

小坑记录：一开始把 <Metasploitable_IP> 这种占位符原样输入，或者把 /24 也带上，Kali 会报 parse error 或 名称或服务未知。ping 时只写纯 IP 即可。


四、编写并运行检测脚本

Kali 没有装 VS Code，直接用终端里的 nano 写脚本就够了。

  1. 创建脚本文件
    运行：nano ~/check_vsftpd.py
  
  2. 在终端里的 nano里粘贴代码：

<img width="1178" height="1528" alt="06e50daf-1b58-4fd2-b7e1-d01d54e4793f" src="https://github.com/user-attachments/assets/2a3fd1f9-244f-4659-b916-3231a51620e0" />

  
3. 保存退出

然后按 Ctrl + O 保存，回车确认
然后按 Ctrl + X 退出

4. 运行脚本
   输入：python3 ~/check_vsftpd.py
   
提示输入 IP 时，填入靶机 IP），然后回车，输出结果。
<img width="484" height="184" alt="562c6e1c6e3b1b07ed11a7c8c38c9d08" src="https://github.com/user-attachments/assets/e4145bb0-07ca-42a3-a74b-300a875ea9c2" />


五、结果验证

脚本运行后，6200 端口显示开放，说明后门被成功触发。

六、小结

笑脸后门的触发条件很简单：用户名里带上 :) 就行。检测的关键是看 6200 端口是否被打开。
整个实验在本地虚拟机中完成，靶机与外部网络隔离。
修复方法：把 vsftpd 升级到官方安全版本，或者改用 SFTP 等更安全的方案。
温馨提醒：这类测试一定要在自己搭建的隔离环境里做，不要对未经授权的系统扫描或利用。
