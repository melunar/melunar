# Linux环境下安装 Jenkins

> 系统运行环境为 Ubuntu18.04，Jenkins 是依赖于 Java 的，所以必须在 Jenkins 节点机器上安装 java 环境


## 安装java环境

> 以下安装 Java JDK 在linux系统上的开源版本 Openjdk为例
> 当前的Jenkins版本不支持Java 10（和Java） 11）, 但不确定未来新版Jenkins是否会支持，尽量使用java8的版本

1. 更新软件包列表 `sudo apt-get update`
2. 安装openjdk-8-jdk `sudo apt-get install openjdk-8-jdk`
3. 查看java版本，看看是否安装成功：`java -version` 或 `javac`

## 安装Jenkins

使用下面的wget命令，导入 Jenkins 软件源的 GPG keys：

`wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -`

下一步，添加软件源到系统中：

`sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`

一旦 Jenkins 软件源被启用，升级apt软件包列表，并且安装最新版本的 Jenkins：
```
sudo apt update
sudo apt install jenkins
```

> 如果出现类似  
`E: The repository 'http://pkg.jenkins.io/debian-stable binary/ Release' does not have a Release file.`的错误，请执行  
`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9B7D32F2D50582E6`

> 如果出现类似错误：`The repository 'http://pkg.jenkins.io/debian-stable binary/ Release' no longer has a Release file` => 修复：`apt-get --auto-remove --purge dist-upgrade`

安装完成后，Jenkins 服务将会被自动启动。
可以通过打印服务状态来验证它：

```
systemctl status jenkins
```

## 访问和配置Jenkins

> 如出现无法访问的情景，请确认防火墙是否正确配置，配置防火墙`sudo ufw allow proto tcp from 192.168.121.0/24 to any port 8080` or `sudo ufw allow 8080` 使8080端口允许在任何地方访问  
如果你使用的是阿里、腾讯等ECS云服务，确保8080端口对外开放，详细配置参考相关云服务配置页面

Jenkins启动之后默认端口为`8080`, 浏览器访问：`http://ip_or_domain:8080`

打开之后会看到密码验证的界面，通过页面提示获取密码：`cat /var/lib/jenkins/secrets/initialAdminPassword`

![Jenkins](https://note.youdao.com/yws/public/resource/0adee7a13a025b76cc154a7ba77c51b4/xmlnote/DF0967252B014F9A8A5EE014AEA737A5/6496)

然后提示安装插件，我们这里选择【安装推荐的插件】

![Jenkins](https://note.youdao.com/yws/public/resource/0adee7a13a025b76cc154a7ba77c51b4/xmlnote/C44F6146F3BE4FF5B501A6440E7EEA1A/6503)

等待安装完成
![Jenkins](https://note.youdao.com/yws/public/resource/0adee7a13a025b76cc154a7ba77c51b4/xmlnote/131DC0342EEF48CBA6EBAC4D16DC9A1F/6509)

> 如果安装部分插件失败，可以采取重试策略，由于网络等其他原因导致无法安装的原因，也可以选择跳过

设置管理员账户（也可以跳过，默认使用admin账户）
![Jenkins](https://note.youdao.com/yws/public/resource/0adee7a13a025b76cc154a7ba77c51b4/xmlnote/DED0FA3695C94B23A434E8BFE4CCB8C6/6514)

后面都是一直【下一步】，直到看到如下主页面
![Jenkins](https://note.youdao.com/yws/public/resource/0adee7a13a025b76cc154a7ba77c51b4/xmlnote/9C4F15A59D96490FAA3978D655000B64/6522)



## 总结

关于Linux安装Jenkins的步骤就这样了，更多资讯请参考[Jenkins官网](https://www.jenkins.io/zh/)

------

**更多关于我**
- 💻[博客](http://lalapkp.cn)
- 🐱[Github](https://github.com/melunar)
- 🔨[掘金](https://juejin.cn/user/2612095355979405)
- 🐶[知乎](https://www.zhihu.com/people/hao-yong-21)
- 👱[关于我](http://www.lalapkp.cn/about)
- 🐒[CSDN](https://blog.csdn.net/Haoyong110?spm=1000.2115.3001.5343&type=1)

**微信公众号 [ 代表moon ]**

![代表moon](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/qrcode_for_gh_64a22fb6b2a0_344.jpg)
--------
