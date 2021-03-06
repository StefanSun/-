
# FAQ

## 索引
* HyperApp 问题
    * 常见安装错误
    * BBR 问题
    * 服务器错误
* 应用问题
    * 文件上传大小限制
* 爱国问题
    * 手机爱国电脑不爱国
    * 无法爱国的问题
* [GCP 问题](#gcp)
    * [结算账号被停用](proxy/get-started.md)
    * [密钥认证失败](proxy/get-started.md)


## HyperApp 常见错误排查

### 如何更新应用？

1. 在服务器下面的 Docker 监控里选择你的应用，轻点后选择 `Pull Image` 即可更新镜像，然后在应用那里 `更新配置`
2. 手动的话可以 `docker ps` 找到应用的镜像名字，然后 `docker pull 镜像`，然后在应用那里 `更新配置`

### 安装错误


#### 为什么在安装应用的时候.会卡在安装中非常久(这种情况多出现在国内VPS中)？

请先确定你的VPS可以正确联网再参考下列步骤:
* 方法1: 在hyperapp终端左下角第一个按钮中.选择安装阿里云国内Docker源
* 方法2: 阿里云用户请参考[使用阿里云镜像加速服务](Aliyun-docker.md)
* 终极解决办法: 远离国内VPS.
* 如果你使用的国外 VPS 依然非常慢，那估计是你VPS的网络比较慢，有些应用的镜像很大，下载需要时间。


#### 提示连接失败/无法连接

1. 请不要打开VPN连接hyperapp.（或者在对应的App中加一条规则，设置你的服务器IP直连）
2. 检查服务器IP是否变了（GCP 如果没有用静态IP，服务器关机后再重启IP可能会变）

#### 提示 transport read 或者 timeout

1. 不要开 VPN 使用 HyperApp，并且进行操作的时候不要切换应用.
2. timeout 可能是掉线了，刷新下服务器状态，看看应用状态是否改变。


#### Conflict

`docker: Error response from daemon: Conflict. The container name *** is already used by container ....`

应用已经安装过了，更新了配置要点击“更新配置”


#### docker command not found

1. 确保你 VPS 的操作系统支持 Docker。
2. HyperApp 会自动安装 Docker，出现这类问题可能是 Docker 安装失败，你可以手动执行 `curl -fsSL get.docker.com | sh` 安装最新docker


#### cannot connect to the Docker daemon

1. 请确保你的 Linux 版本符合要求
2. 请确保一定要先安装 BBR 再安装应用，如果你不幸搞错了顺序，参考下一条


#### 如果先安装了docker再安装bbr导致无法启动docker怎么办?

执行 `rm -rf /var/lib/docker/aufs` 清除docker网络配置.然后重新安装所有应用即可.


#### 操作Docker时提示权限不足(Permission Denied)

A: 当前用户没有docker权限，加到docker组才有，执行命令 `sudo usermod -aG docker $USER`


#### 使用bbr安装脚本的时候提示"Inappropriate ioctl for device"
使用键盘随便输入几个字符并且回车两下.原因是bbr安装脚本需要等待输入来执行下一步

---


## 应用问题


### 关于 Nginx 的上传限制

如果你使用了 `Nginx Proxy`，则默认有个2M的上传限制，你可以新建一个文件 `/srv/docker/vhost.d/default` 添加一行  `client_max_body_size 100m;` 来解决这个问题（一般情况下 Nginx 会自动重启，如果没有则需要手动重启下 Nginx Proxy）

你也可以直接运行下面的命令，会自动创建上面的所说的文件:

```sh
echo "client_max_body_size 100m;" > /srv/docker/nginx/vhost.d/default
```
* **除了 Nginx 外，PHP 还可以有自己的上传限制，请参考 https://github.com/waylybaye/HyperApp-Guide/issues/152**


## 爱国问题

### 手机可以电脑不行：
请先查看客户端日志，并贴出来用全局模式测非Chrome、Firefox浏览器（因为Chrome浏览器的很多插件会改代理设置）如果全局可以就能断定是pac规则或者浏览器的原因。


### 无法科学上网的问题

#### 排查服务端的问题

1. HyperApp 里面刷新一下服务器状态，看应用是否启动成功(显示 up xxx seconds/minutes/hours)
2. 点击 `查看日志` 看看应用日志中是否有错误提示
    * ss-libev 如果有一些连接错误的信心，一般是客户端的密码、加密方式、混淆没填对。
3. 如果上两步都没有问题，请检查
    * 是否是机器防火墙问题（参考下面的第二项）
    * 是否是客户端配置错误（参考下面的客户端排错）

#### 排查客户端的问题

* 是否使用了跟服务端一致的配置。建议通过扫描二维码的方式添加。
* 如果你用 Wingy 不要开 OBFS （除非你自己足够了解）
* 另外不要用 SSR 连 SS，也不要用 SS 连 SSR
* ss-libev 客户端的 http/tls 混淆必须要和服务端一致。（服务器如果设置的 tls 混淆，客户端也必须用 tls）
* 最后如果你的问题不在上述思路中.请仔细查看文章中涵盖的hyperapp常见问题以及解决方案.实在搞不懂再上群提问并且请注意你的语气.没有人有义务帮助你.

### 如何排查服务器是否有防火墙阻拦了某个端口？

1. 在 VPS 上运行: `telnet localhost PORT(要检查的端口)` 能通说明服务端没问题。不通说明服务端启动失败。
2. 在电脑上运行`telnet IP PORT(要检查的端口)` 能通说明防火墙没问题。不通说明是防火墙的问题。
3. 在 VPS 上运行 `lsof -i :端口` 来查看是否有进程正在监听该端口，如果有说明是防火墙屏蔽了端口访问，如果没有说明部署失败。


## 防火墙设置教程

* 临时禁用系统防火墙
  * CentOS7: service firewalld stop
  * CentOS6: service iptables stop
  * Ubuntu16: service ufw stop
* 永久禁用系统防火墙
  * CentOS7: systemctl disable firewalld
  * Ubuntu16: systemctl disable ufw
* 系统防火墙永久添加端口
  * CentOS7: firewall-cmd --permanent --zone=public --add-port=端口/tcp
* 平台防火墙
  * GCE: 网络 → 防火墙规则
    * 点击`创建防火墙规则`
      * 名称：随便输入一个名称
      * 目标：选择 `网络中的所有示例`
      * 来源过滤：`0.0.0.0/0`
      * 协议和端口：`指定的协议和端口` 下面输入 `tcp;udp:端口号`
  * AWS: EC2 控制面板 → 安全组
  * 阿里云: 云服务器 ECS → 安全组

## GCP

### 二维码显示不全，如何手动添加 GCP ？

1. 二维码上面有 `IP`，`用户名`，`端口` 信息，在 `HyperApp → 服务器` 中点击添加按钮，选择手动配置。
2. 在出现的窗口中，分别输入上面对应的字段，密码不填写，在最下面的密钥那里选择一个密钥（应该只有一个）。
3. 添加即可。但是稍后的可能会出现密钥认证失败的错误，所以建议将密钥名字改成你的 gmail 邮箱名字。或者按照教程，修改两处用户名一致即可。

## 厂商常见问题：

* 搬瓦工只支持 KVM虚拟化 的机器。
* Vultr/DigitalOcean/AWS 全都支持
* GCP没有领到试用金：重新打开试用页面再点击一次“Try it free”
* 50Kvm 的机器第一次登录需要记下下次登录的 SSH 端口
* DigitalOcean 的机器第一次登录需要改密码

## HyperApp 常见问题:

* bbr脚本执行后重启后发现执行`lsmod | grep bbr`发现输出空白.bbr没有正确启动.同时执行sysctl -p输出空白
* 根据使用者反馈的问题 ,初步解决方案是执行一下命令

​```bash
sudo su && cp /etc/sysctl.conf /etc/sysctl.conf.bak && rm -rf /etc/sysctl.conf && touch /etc/sysctl.conf && chmod 644 /etc/sysctl.conf && sudo echo -e "\n\n\n\nnet.core.default_qdisc = fq\n\n\nnet.ipv4.tcp_congestion_control = bbr" >> /etc/sysctl.conf && sysctl -p
```

​**如果这时候`lsmod | grep bbr`还没有出现tcp_bbr的话请自行根据网上linux文本编辑器教程编辑/etc/sysctl.conf.加上这两行**

​```bash
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

* 关于squid内存不足无法正确启动的解决办法:

   * Ubuntu使用如下命令创建swap

```bash
sudo su
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo "vm.swappiness = 10" » /etc/sysctl.conf
sysctl -p
```

* Centos请将第一个echo换成
  `echo "/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab`




## Hyperapp官方应用教程链接

https://www.hyperapp.fun

#### ~~部署v2ray等hyperapp应用可适当参考如下链接~~(打死这个不要脸的:)

https://vinga.ml
