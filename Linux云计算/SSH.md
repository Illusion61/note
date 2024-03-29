# SSH原理

## 1.1 简介

#### SSH(Secure SHell)

- SSH是一种安全传输数据的协议;用加密的方式在不安全的网络上进行安全的通信
- SSH可以用来远程登录服务器,智能机器(智能小车),嵌入式设备(监控)等

#### SSHD(Secure SHell Daemon)

- SSH的守护进程(Daemon),用来提供SSH服务
- 守护进程(Daemon):是在计算机系统中在**后台运行**的一种特殊进程,通常在系统启动的时候启动,在系统运行期间持续运行,独立于用户登录会话
  - 守护进程一般没有直接交互的用户界面
  - 守护进程提供长期服务,例如网络服务系统监控日志记录等
  - 守护进程通常以系统级别的任务运行,不受特定的用户登录或注销的影响

#### SFTP(Secure File Transfer Protocol)

- FTP:文件传输协议,用明文进行数据传输
- SFTP:加密文件传输协议,用SSH加密之后进行文件传输

## 2.1 哈希算法

- 哈希:将字符串/文件进行**单向**加密(无法还原),来得到特征字符串
- 用处:
  - **存储密码**,防止数据库泄露之后密码被别人知道
  - **校验文件**:文件用哈希算法得到一个特征码,如果文件发生改变特征码就会改变(下载校验,网盘秒传)
    - 下载校验防止被别人偷偷修改植入后门
    - 网盘对于md5相同的文件只会维护一个
  - 比较长字符串(大于几千位),将长字符串经过哈希之后一般会得到一个固定长度的短字符串(256位),方便比较
- 常见哈希算法:
  - MD5:将输入的消息进行计算和变换,生成一个128位的哈希值(不安全)
  - SHA-1:生成160位哈希值
  - **SHA256**/SHA512:分别生成256位和512位
  - CRC32:生成32位的哈希值(用于校验)
- 时间复杂度:O(n)且常数因子比较小

## 2.2 对称和非对称加密

SSH采用非对称加密算法传输数据

#### DES/3DES对称加密算法(扩展/选学)

- 对称加密算法(只有一个密钥):
  - 优点:加密强度比较大,几乎无法破解;加密速度快
  - 缺点:密钥不能丢失,否则自然就会被破解了
- 过程:发送方**用密钥A**将密码加密,传输,接收方**用同样的密钥A**解密密码

#### RSA非对称加密

- 非对称加密密钥分为:公钥(public-key),私钥(private-key)
  - 同一对公私钥:公钥加密只有私钥能解密,私钥加密只有公钥能解密
  - 优点:即使泄露了公钥和传输的加密信息,也不会被破解
  - 缺点:只能保护单向的数据传输,而且一旦私钥泄露信息也会被破解;加密速度慢
- 准备:接收方(服务器)事先**发送公钥**给发送方(客户端)
  - 公钥是可以分享给别人的,私钥必须要保护好
  - **公钥可以理解为一种加密的算法**,而且只能通过这个私钥才能解密
- 过程:发送公钥,发送方使用**公钥**将密码加密,加密数据传输,接收方用**私钥**将密码解密

## 2.3 SSH登录过程

SSH**登录**过程中采用**非对称加密**,**会话**过程中采用**对称加密**(速度更快)

#### 密码登录

0. 客户端发送ssh登录请求,与服务器建立连接

   ```
    ssh root@107.173.181.210 -p 58613
   ```

1. 服务器端本身拥有一对公钥私钥,**将公钥发送给客户端**

   - 指纹:是对公钥的摘要或者哈希值,用于验证公钥的真实性或者完整性???
   - 服务器端的公钥会记录在客户端的~/.ssh/known_hosts

   ```
   ED25519 key fingerprint is SHA256:DDBJK2SUJQbZnYVdxQaxqz8eGDXCkU+oiMKJCluSf0M.  #服务器会把指纹发过来
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  #是否继续连接,将服务器的公钥添加到本地
   Warning: Permanently added '[107.173.181.210]:58613' (ED25519) to the list of known hosts.
   ```
   - 查看服务器端的公钥和私钥

   ```bash
   cd /etc/ssh
   #在这个目录中,ssh_host_[算法名]*的文件就是公私钥文件
   ssh_host_ed25519_key #ed25519算法的私钥文件
   ssh_host_ed25519_key.pub #ed25519算法的公钥文件
   ```

   - 客户端当中~/.ssh/known_hosts当中的公钥和服务器端中/etc/ssh/ssh_host_[算法].pub的内容应该完全相同

2. 客户端**验证公钥指纹**,如果与known_hosts当中的不一致,就会提示是否信任

3. 客户端也把自己的公钥发送给服务器

4. 客户端与服务器端采用**Diffie-Hellman算法根据双方的公钥计算**生成SessionKey,作为之后对称加密的密钥建立加密信道

5. 加密信道上面进行用户身份验证(保证验证信息不被第三方窃取),用户将密码通过SessionKey加密后发送给服务器


#### 中间人攻击(MITM)

- 什么是中间人攻击:**拦截**双方的通信,作为中间角色进行通信

- 中间人攻击要素:

  - **拦截**:能够**拦截**双方的通信,使得实际上双方发送的内容不能**直接抵达**
  - **理解**:能够**理解**通信的**加密内容**,而不是只看到一串乱码

- 常见的中间人攻击:

  - 离线MITM:拦截和阅读纸质邮件,并向同一位收信人发送伪造邮件
  - 网络MITM:免费公共WIFI伪造常见的电商网站和银行网站,在这些网站上面登录就会泄露密码

- **针对SSH的中间人攻击**:

  - 形式:劫持SSH连接的初始握手过程,将自己的公钥发送给客户端,并接收来自服务器的公钥

  - 解决:**查看指纹是否与之前储存的一致**(看公钥是否与历史公钥相比发生改变),如果不一致就会提醒是否信任

  - 指纹:公钥进行哈希(SHA256)之后的特征字符串,长度比公钥短(RSA公钥1024位)方便比较

    ```bash
    ssh-keyscan -p 58613 107.173.181.210 #ssh-keyscan获取服务器不同算法的公钥
    ssh-keygen -E SHA256 -lf /etc/ssh/ssh_host_ed25519_key.pub #获取服务器端的指纹
    ```

#### 免密登录

初次使用:

1. 客户端使用ssh-keygen生成公钥私钥对
2. 客户端使用ssh-copy-id将公钥发送给服务器,通过密码验证之后将公钥写入~/.ssh/authorized_keys当中

```
ssh-keygen #生成默认RSA密钥对(3072位)
#私钥文件设置密码?
ssh-copy-id -p 58613 root@107.173.181.210 #发送公钥给服务器添加入authorized_keys,需要输入密码或者密钥文件
```

用户登录:

1. 客户端将公钥发送给服务器,服务器在authorized_keys当中找到对应的公钥文件
2. 服务器生成随机的字符串,用相应的公钥进行加密后发送给客户端
3. 客户端用私钥解密字符串之后发送给服务器
4. 服务器查看客户端发送的字符串和自己生成的字符串是否一致

注意:

- **免密登录通俗理解:服务器存储加密规则(公钥),检验客户端是否有能力解密**

- 客户端如果有公钥文件就发送给服务器与authorized_keys当中进行查找,如果没有公钥文件就用私钥与服务器中的authorized_keys文件进行逐一验证直到找到相匹配的公钥

## 2.4 sshd服务配置

开机自动启动ssh守护进程(服务端)

```bash
systemctl enable ssh
systemctl restart ssh
```

/etc/ssh/sshd_config,配置完之后重启sshd服务

- 设置是否允许root登录

```conf
PermitRootLogin prohibit-password #Root用户禁止密码登录(只能免密登录)
PermitRootLogin yes/no #是否允许root用户登录
```

- 修改端口号:

```
Port 31649 #修改端口号
```

- 配置服务器端默认加密算法

```conf
HostKey /etc/ssh/ssh_host_rsa_key #默认使用rsa加密
HostKey /etc/ssh/ssh_host_ecdsa_key #默认使用ecdsa加密
HostKey /etc/ssh/ssh_host_ed25519_key #默认使用ed25519加密
```

- 禁止密码登录

```
PasswordAuthentication no
```

- 设定只允许特定IP上特定用户登录

```conf
#只允许107.173.181.210上的root用户和18.162.157.179上的所有用户进行ssh登录
AllowUsers root@107.173.181.210 *@18.162.157.179
```

不同非对称加密算法对比???

## 2.5 ssh登陆命令

- SSH客户端会默认使用~/.ssh/id_rsa作为私钥文件

```bash
ssh -p [端口号] -i [私钥文件路径] [用户名]@[IP地址]
```

- scp传输文件

```
scp -P [端口号] -r /本地路径/目录名 用户名@远程计算机IP:/远程路径/
scp -P 53269 -v -r  /usr/local/hadoop-3.3.6 root@18.141.44.29:/usr/local
```

## 2.6 SSH的访问日志





# VPN原理和搭建

## 1.1 密钥化哈希

- 密钥与待哈希数据一起进行哈希运算得到一个

## 1.2 数字签名

- 数字签名过程:
  - 服务端对要发布的文件进行SHA256生成**校验码**
  - 服务端使用自己的**私钥**对**校验码**进行**加密**,制作成**数字证书**
  - 客户端用服务端的公钥解密数字证书,并比对文件的校验码和解密出来的校验码是否一致
- 数字签名作用:
  - **确保是私钥拥有者发布的,保证权威性**:因为只有这个服务端能够用私钥进行加密然后被公钥解密
  - **保证文件的完整性**:即使别人想要去修改文件也不行,因为没有私钥对修改后的文件哈希值进行加密

## 1.3 数字证书

- 数字签名的问题:中间人攻击,在一开始建立连接的时候就冒充权威机构发送公钥签发伪造的数据签名
- 数字证书:发布数字证书的机构很少,发布确保具有一定的权威性
  - 数字证书颁发过程:申请者向CA机构提交自己的姓名邮箱域名等数据,得到一个CA证书(包含相关信息以及**有效期**)
  - 数字证书怎么确保不能自己伪造:有CA的数字签名
  - 数字证书上面记录了相关的信息:IP地址,域名等等信息;如果只是复制数字证书但是内容与数字证书上面的不一致,那么就会失败
- 数字证书就是CA对发送者公钥和其他信息进行数字签名,之后把公钥和其他信息和CA数字签名组合在一起
- **CA数字证书:**
  - 组成:申请者公钥,其他信息,CA数字签名
  - 接收者步骤:
    - 用CA公钥解开CA的数字签名
    - 验证CA数字签名信息,比较哈希值是否一样(过期?域名一样?公钥一样?...):验证这个证书的权威性
    - 拿到发送者公钥,用公钥验证发送者数字签名:信息是否完整,是否是由私钥拥有者发布的
    - 进行正常通信

## 2.1 VPN基本概念

VPN:virtual-private-network(虚拟私有网络)

#### VPN的分类:

- SSL:SSL+HTTP =HTTPS
  - 传输层加密
  - 比较安全
  - 速度慢
- IPSec:
  - 网络层加密
  - 比较安全
- PPTP:容易被攻击窃取数据

#### IPSec VPN

- IpSec诞生背景:
  - 传统IP专线价格较高
  - IpSec-VPN基于加密线路传输
  - 部署非常方便,客户既能够上网又能与分支相互通信
  - IpSec-VPN在企业/校园网络分支之间使用
- 三个协议:
  - ISAKMP:协商,交换密钥
  - ESP:执行(加密,验证完整性)
  - AH:验证完整性无加密(不推荐适用)

## 2.2搭建Trojan协议VPN

- nginx搭建http代理

```nginx
server {
        listen 34;
        location / {
        	resolver 8.8.8.8;
            proxy_pass $scheme://$host$request_uri;
            proxy_set_header Host $host;
        }
}
```

#### Trojan协议简介

[原链接](https://p4gefau1t.github.io/trojan-go/basic/trojan/)

- `Shadowsocks`:
  - 使用**高匿名**的加密协议,数据包本身没有任何特征
  - 很难通过防火墙检测;因为GFW在检测到随机特征(大流量,随机字节流,高位端口)之后会进行主动检测(GFW**主动连接服务器的这个端口**检查通信对象是否是一个正常网站)
- `Trojan`:
  - 使用**常规**的TLS协议,看起来流量与HTTPS网站相同(HTTPS加密).TLS保证流量的保密性,完整性,防范中间人
  - 容易通过防火墙检测,除非针对特定网站使用无差别封锁:
    - 如果TLS连接建立失败,返回400 Bad Request
    - 如果识别为**非Trojan协议**或者**密码错误**:将会代理到HTTPS服务器上面去
    - 如果**识别为Trojan协议并且密码正确**:将会解析客户端的请求并**代理用户的流量**

#### Trojan-GO搭建

- 搭建web服务器
- 配置TLS/SSL(HTTPS)证书
- 配置server.json
  - localaddr/port:监听的地址和端口
  - remoteaddr/port:如果不是trojan协议或者密码错误时候使用的服务端口和ip

```trojan
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "your_awesome_password"
    ],
    "ssl": {
        "cert": "server.crt",
        "key": "server.key",
        "fallback_port": 1234
    }
}

```

