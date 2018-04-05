title: Linux调优
date: 2017-3-27  
tags:
    - original
categories:
    - Linux  
---

### 打开文件数设置

``` sh  
cat >> /etc/security/limits.conf << EOF
* soft nofile 65535
* hard nofile 65535
EOF
```

<!-- more -->

### Linux内核调优

``` sh  
cat >> /etc/sysctl.conf << EOF
# 该参数设置系统的TIME_WAIT的数量，如果超过默认值则会被立即清除
net.ipv4.tcp_max_tw_buckets = 20000
# 定义了系统中每一个端口最大的监听队列的长度，这是个全局的参数
net.core.somaxconn = 65535
# 对于还未获得对方确认的连接请求，可保存在队列中的最大数目
net.ipv4.tcp_max_syn_backlog = 262144
# 在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
net.core.netdev_max_backlog = 30000
# 能够更快地回收TIME-WAIT套接字。此选项会导致处于NAT网络的客户端超时，建议为0
net.ipv4.tcp_tw_recycle = 0
# 系统所有进程一共可以打开的文件数量
fs.file-max = 6815744
EOF
```

<br>