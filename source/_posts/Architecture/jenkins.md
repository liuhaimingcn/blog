title: 使用jenkins进行项目的自动构建部署
date: 2016-12-21  
tags:
    - original
categories:
    - Architecture
---

# jenkins 简介
* Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，功能包括：持续的软件版本发布/测试项目和监控外部调用执行的工作。
* 官网地址地址： https://jenkins.io

<!-- more -->

# 下载安装启动

## CentOS 下用yum进行安装启动

``` sh
# 先更新源再安装最新版 jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins
# 启动
sudo service jenkins start
# 停止
sudo service jenkins stop
# 重启
sudo service jenkins restart
# 检查
sudo chkconfig jenkins on
```

## 下载war包放到tomcat中启动

* 服务器 yum 安装速度太慢了，最终我选择了这种方式，本地下载好war包传到服务器上的tomcat容器下，然后启动
* 下载地址：http://mirrors.jenkins.io/war-stable/latest/jenkins.war

# 初始化

* 在浏览器中输入url打开jenkins的后台控制页面
  ![初始化界面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.00.jpeg)
* 初始化成功后会自动生成一个管理员密码放到指定位置，根据页面提示复制密码粘贴到输入框就可以登录了
  ![初始登录界面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.01.jpeg)
* 登录成功后回让你选择插件的安装，可以选择建议的安装也可以自己进行选择，不清楚的话可以使用建议的安装
  ![初始登录界面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.02.jpeg)
* 由于建议安装的插件比较多，安装的过程有点慢，多等待一会
  ![初始登录界面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.06.jpeg)
* 安装的过程也可能因为网络等一些原因安装会失败，现在可以无视它，点击Continue，后面再进行手动的安装
  ![安装完成](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.30.jpeg)
* 安装完成后最好新创建一个管理员账户代替之前的临时自动生成的密码账户
  ![创建新的管理员账户](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.31.jpeg)
* 初始化完成，进入后台管理界面
  ![初始化完成](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.32.jpeg)
  ![后台管理界面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.322.jpeg)

# 安装插件

* 之前初始化的时候，有些插件安装失败，可以在用到的时候来手动修复它，没用到的话就可以暂且不理它，不影响jenkins的使用
* 点击左侧边栏的“系统管理”，就可以看了插件安装的一些错误信息
  ![插件错误信息](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.40.jpeg)
* 在“系统管理”中往下拉,找到“管理插件”点击进去就可以查看和管理所有的插件，点击“可选插件”显示所有jenkins支持的插件，在右上角的“过滤”输入框中，输入需要安装的插件名就可以筛选查找到想要的插件
  ![查找插件](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.54.jpeg)
* 选中要安装的插件输入框，点击安装就可以在线安装需要的插件，当然由于网络的原因也可能再次安装错误，或者安装的比较慢。我们可以点击插件名进入插件的主页，里面有该插件的详细信息并能下载hpi文件进行手动安装
  ![插件详情](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2011.58.jpeg)
* 在“管理插件”的页面中点击高级选项，我们可以在下面找到“上传插件”，上传下载好的插件，点击“上传”，系统就会自动上传安装该插件。
  ![上传插件](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2012.00.jpeg)

# gitlab的配置
* 集成gitlab，让jenkins能够直接读取修改gitlab中的代码，方便项目的构建
* 安装gitlab-plugin
* 在“系统管理” -> “系统设置“ -> “Gitlab” 中配置对应的gitlab信息
* 点击“Test Connection”测试下配置是否成功
  ![Gitlab配置](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2012.28.jpeg)
  ![Add Gitlab Credentials](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2012.27.jpeg)
  ![Gitlab API token](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2012.29.jpeg)

# Publish Over SSH
* 通过ssh连接远程服务器，并能执行脚本部署项目
* 安装publish-over-ssh
* 在“系统管理” -> “系统设置“ -> “Publish over SSH” 中配置对应的ssh信息
* Key中填登录远程服务器的密码([ssh免密码登录](https://www.google.com.hk/search?client=safari&rls=en&q=ssh%E5%85%8D%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95&ie=UTF-8&oe=UTF-8&gws_rd=cr,ssl))
* 点击"SSH Servers"后的“增加”按钮，新增一个远程服务器
* 点开“高级...”按钮，能进一步的配置端口等信息。
* 配置为Server信息后，点击"Test Configuration"按钮测试是否能够连接成功。
  ![ssh配置](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.21.jpeg)

# 监测代码变动自动部署

* 点击左侧边栏的“新建”按钮，新建一个任务。
* 填写项目的名称，并选择一种构建的方式，此时我们选择第一个，构建一个自由风格的软件项目，然后点击“OK”按钮创建任务，并进行详细的配置
  ![新建任务](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.28.jpeg)
* 默认设置里填写项目名和描述，并选择之前配置好的要连接的gitlab
  ![默认配置](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.32.jpeg)
* 配置源码，填写要构建项目的源码仓库地址，并指定要构建的分支
  ![配置源码](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.30.jpeg)
* 配置触发器，选择触发构建的方式，可以通过hook，根据jenkins提供的地址，放到gitlab中的hook配置中，就会自动触发构建。此时我们选择的是定时检测项目变动，如果检测到分支有新的变动就触发构建，如果感觉一分钟时间太频繁的话，可以自己设置时间频率。
  ![构建触发器](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.34.jpeg)
* 配置构建，构建选用的是“Invoke top-level Maven target”,填写对应的maven命令，就会自动执行maven命令进行侯建
* 配置构建后操作， 该行为会在构建完成后执行，我们选用的是“Send build artifacts over SSH”的方式，把构建完成的jar包发送到远程服务器上用ssh命令执行启动，此时jenkins所有机器的默认路径是任务所在的目录，远程机器的默认路径是之前publish-over-ssh中指定的文件地址。Source files指定要传送到远程服务器上的文件，remote directory指定的是传送到远程服务器上的文件地址，Remove prefix是值要去除的文件目录，不然传送到远程服务器也会带有该目录层级结构的。exec Command里输入的是在远程服务器上要执行的指令。
  ![构建部署配置](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2014.39.jpeg)
* 项目构建后会有构建历史，点击进去，选择“Console Output”就可以查看构建过程中的执行记录

# 项目回滚
* 上面虽然实现了项目的自动部署，但是有时部署失败的时候我们需要回滚到指定版本的构建，这样才能更灵活的进行项目的构建部署。我们可以选择“参数化的构建过程”进行传递不同的参数来选择是进行新的构建还是回滚
* 如果要在实现回滚，一定要在构建后将，构建完成的文件进行存档，方便以后回滚的时候使用
  ![构建后存档](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2015.27.jpeg)
* 使用参数化构建过程，让后面的脚步可以根据不同的变量执行不同的操作。添加“Choice”参数配置不同的选项，让选择发布还是回滚，添加“String Parameter”参数来传递要回滚的版本号。
  ![参数和构建过程](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2015.28.jpeg)
* 构建选择“Execute Shell”的方式，自己根据变量，自定义构建的脚本，此时如果是发布安装maven的构建过程进行新的构建，如果是回滚，知道历史构建后的文件，复制到当前构建结果目录。
  ![构建脚本](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2015.29.jpeg)
* 点击构建，根据不同的参数选择发布还是回滚，回滚的时候填写要回滚到的历史版本号
  ![构建页面](http://luobuxiadeyu.oss-cn-beijing.aliyuncs.com/2016-12-21%20at%2015.30.jpeg)

<br>