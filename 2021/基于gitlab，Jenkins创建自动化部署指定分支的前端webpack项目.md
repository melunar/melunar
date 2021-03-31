# 基于gitlab，Jenkins创建自动化部署指定分支的前端webpack项目

> 当前大环境下，gitlab是大部分企业选择的代码托管平台，借助Jenkins对gitlab的构建支持，实现快速在指定环境上部署，本文拟定有两个部署环境（开发环境、正式环境）

<!--todo link-->
关于Jenkins的安装，参考上一篇文章[Linux环境下 Jenkins 的安装](https://github.com/melunar/melunar/blob/main/2021/Linux%E7%8E%AF%E5%A2%83%E4%B8%8B%20Jenkins%20%E7%9A%84%E5%AE%89%E8%A3%85.md)

## 准备工作

**插件准备**  
  
准备好可用的Jenkins环境，进入`系统管理-插件管理-可选插件`

我们的项目依赖gitlab，需要安装相关插件：搜索『gitlab』，选择安装`gitlab plugin 和 gitlab hook plugin`两个插件

我们的构建需要制定git仓库分支，需要branch参数指定分支：搜索「Git Parameter」，安装该插件

![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-plugin-install.png)

安装完成之后重启Jenkins，以使得插件生效


**node工具配置**  
 
基于当下潮流的前端框架，是离不开node环境和npm包管理器的，所以我们需要提前给Jenkins配置node环境，进入`系统管理-全局工具配置`  

点击`【新增nodeJs】`指定一个版本的nodejs，并提供一个别名（名字随意，便于自己认得就行），然后点击保存按钮退出。也可以在这里添加多个版本的node环境，在构建过程可以指定当前运行构建的node环境。我们这里以发稿日最新的稳定node版本`v15.4.0`为示例。

![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-node.png)

## 创建项目/job

### 创建项目

填入项目名称，选择自由风格（第一项），确定，进入下一步详细配置表单
![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-01.png)

### job配置

填写项目构建描述

#### 选择**参数化构建过程**

注意 我们这用到了准备工作中安装的git parameter 提供的参数配置功能。依次操作【`添加参数-git参数`】，参数名称我们这填branch（当然，这个是参数名是完全自定义），描述随便填，参数类型选择`分支`（这样，构建任务就知道这个参数是用来指定仓库分支的）  


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-git-param.png)

git parameter还支持很多别的参数，比如`flag、commit`等，可以根据自身需求选择参数类型  

最后我们添加一个文本类型的参数：remark 用于做构建备注


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-build-2.png)



文本型参数属于Jenkins内置参数类型，大家可以根据自己需求和兴趣对其进行更深入的研究，常用的有`凭据参数、文本参数、布尔类型参数、运行时参数等`，满足构建过程各种参数注入，提供编程式构建方案

### 源码管理

这里我们选择Git源，并输入Repository URL，就是我们gitlab项目的remote address


注意这里需要gitlab授权，在凭据添加里面需要先创建一个Jenkins凭据，如果创建的凭据是全局性的，一次创建，可以永久在Jenkins平台别的项目中继续使用。当然也可以在全局配置中创建凭据（`系统配置-全局配置`），这里就不细说了。    
![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-repository.png) 

添加凭据：选择全局凭据，类型我们这里选择`userName with password`（意思就是你在这里提供gitlab的用户和密码，Jenkins拿着它去向gitlab索取一切需要的权限），你也可以选择`gitlab api token`，该模式更加安全，token需要在gitlab创建和管理，感兴趣可以尝试。     填写用户名、密码和ID提交即可 


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-pingju.png)

然后我们在凭据添加选中刚才添加的凭据即可，源码管理初步完成

【oh 别忘了还有分支选择】
这里我们在指定分支填入`${branch}`, branch 就是我们上述添加的git参数，`${}`是Jenkins获取参数的模板格式。

![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-select-branch.png)

源码库浏览器，我们可以设置一下源码检出的Jenkins本地路径，一般填写项目根目录即可`./`(注意是相对路径)

### 构建触发器

关于构建触发器，就是根据你的需求，期望以什么样的形式触发构建，比如每一次push、通过restful api、通过Jenkins平台、通过定时器触发等等，我们该示例通过shell脚本执行构建，这里的触发器暂时就不选择了。 【ok，继续....】

### 构建环境

这里我们需要选择node环境，就是准备工作中配置的node版本，勾选`Provide Node & npm bin/ folder to PATH`, 选择我们想要的node环境


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-select-node.png)


### 构建

最后一步，也是最重要的一步

增加构建步骤，选择`执行shell`, 然后我们填写构建流程的shell脚本


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-shell-detail.png)


```
#!/bin/bash
echo '====== out shell start ======'
echo '====== play on branch: ======'
echo ${branch}
echo ${remark}
sh /var/lib/jenkins/workspace/我的项目名/jenkins-deploy.sh
echo '====== out shell end ======'
echo '====== start upload files ======'
## 测试环境部署路径
devPath='root@我的主机IP:/data/application/我的项目名-dev'
## 生产环境部署路径
masterPath='root@@我的主机IP:/data/application/我的项目名'
if [ ${branch} == 'origin/master' ]
then
	echo $masterPath
	scp -r ./dist/* $masterPath
fi
if [ ${branch} == 'origin/dev' ]
then
	echo $devPath
	scp -r ./dist/* $devPath
fi

```

`我的项目名/jenkins-deploy.sh`: 内容
```
#!/bin/bash
echo '=====shell start===='
node -v
npm -v
ls -al
npm i
pwd
npm run build
```

脚本很简单，大概意思就是先执行项目中的内置的一个 jenkins-deploy.sh 文件, 这个文件指定了该项目完整build过程，
然后根据git branch参数，如果是dev分支，把build的项目往测试环境发布，如果是master分支，就往生产环境发布

这里需要注意一个问题：**scp** 往指定服务器拷贝文件是需要服务器密码的，由于这里是静默shell脚本的执行，没有交互式密码输入的入口，我们需要在服务器上配置Jenkins所在服务器的免密ssh登录，并且注意要用`Jenkins`用户，因为Jenkins执行构建任务和shell脚本的宿主用户是`Jenkins`, 具体配置自行搜索，网上一大堆。

### 保存

最后保存一下，我们的job就创建完成了

然后我们来验证一下

## 执行构建

在Jenkins主页找到我们的job工程，进入详情页

点击`build with parameter`（如果我们没有配置构建参数，这里将会是`build`）


![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-build-1.png)

填写branch参数：选择你想构建的分支（这里的分支列表是从gitlab上事实同步下来的）  
填写remark参数  

![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-build-2.png)


开始构建  

在构建详情页，我们可以事实看到构建控制台输出，稍等片刻，如果看到构建控制台的`Finished: SUCCESS`, 那么恭喜你，顺利完成构建任务。

![示例图](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/blog--2020-3-31--step-success.png)

> 如果有错误发生，可以根据错误log排查问题，也欢迎通过评论区或者issue或者文章末尾的其他联系方式一起讨论



------

**更多关于我**
- 💻[博客](http://blog.lalapkp.cn)
- 🐱[Github](https://github.com/melunar)
- 🔨[掘金](https://juejin.cn/user/2612095355979405)
- 👱[关于我](http://www.lalapkp.cn/about)
- 🐒[CSDN](https://blog.csdn.net/Haoyong110?spm=1000.2115.3001.5343&type=1)

**微信公众号 [ 代表moon ]**

![代表moon](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/qrcode_for_gh_64a22fb6b2a0_344.jpg)

