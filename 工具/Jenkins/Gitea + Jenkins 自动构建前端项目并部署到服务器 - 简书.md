# Gitea + Jenkins 自动构建前端项目并部署到服务器 - 简书
https://juejin.im/post/6887751398499287054?utm\_source=gold\_browser\_extension#heading-3

Gitea 用于构建 Git 局域网服务器，Jenkins 是 CI/CD 工具，用于部署前端项目。

配置 Gitea

下载[Gitea](https://links.jianshu.com/go?to=https%3A%2F%2Fdl.gitea.io%2Fgitea)，选择一个喜欢的版本，例如 1.13，选择gitea-1.13-windows-4.0-amd64.exe下载。

下载完后，新建一个目录（例如 gitea），将下载的 Gitea 软件放到该目录下，双击运行。

打开localhost:3000就能看到 Gitea 已经运行在你的电脑上了。

点击注册，第一次会弹出一个初始配置页面，数据库选择SQLite3。另外把localhost改成你电脑的局域网地址，例如我的电脑 IP 为192.168.0.118。

![](https://upload-images.jianshu.io/upload_images/4174952-e3ff8f02712806de.image)

![](https://upload-images.jianshu.io/upload_images/4174952-e9a2fdaccc01e866.image)

填完信息后，点击立即安装，等待一会，即可完成配置。

继续点击注册用户，第一个注册的用户将会成会管理员。

打开 Gitea 的安装目录，找到custom\\conf\\app.ini，在里面加上一行代码START\_SSH\_SERVER = true。这时就可以使用 ssh 进行 push 操作了。

8\. 如果使用 http 的方式无法克隆项目，请取消 git 代理。

![](https://upload-images.jianshu.io/upload_images/4174952-14436e6990f49643.image)

git config --global --unset http.proxygit config --global --unset https.proxy复制代码

配置 Jenkins

需要提前安装 JDK，JDK 安装教程网上很多，请自行搜索。

打开[Jenkins](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdownload%2F)下载页面。

![](https://upload-images.jianshu.io/upload_images/4174952-d116249f561bcef7.image)

安装过程中遇到Logon Type时，选择第一个。

![](https://upload-images.jianshu.io/upload_images/4174952-d85abd39404370c5.image)

端口默认为 8080，这里我填的是 8000。安装完会自动打开http://localhost:8000网站，这时需要等待一会，进行初始化。

按照提示找到对应的文件（直接复制路径在我的电脑中打开），其中有管理员密码。

6\. 安装插件，选择第一个。

![](https://upload-images.jianshu.io/upload_images/4174952-1313209973879117.image)

![](https://upload-images.jianshu.io/upload_images/4174952-6564e165175af8b8.image)

创建管理员用户，点击完成并保存，然后一路下一步。

8\. 配置完成后自动进入首页，这时点击Manage Jenkins->Manage plugins安装插件。

![](https://upload-images.jianshu.io/upload_images/4174952-901af9f1f9030568.image)

9\. 点击可选插件，输入 nodejs，搜索插件，然后安装。10. 安装完成后回到首页，点击Manage Jenkins->Global Tool Configuration配置 nodejs。如果你的电脑是 win7 的话，nodejs 版本最好不要太高，选择 v12 左右的就行。

![](https://upload-images.jianshu.io/upload_images/4174952-dc27e96807ff0bb8.image)

![](https://upload-images.jianshu.io/upload_images/4174952-c3d0c368c23d50ad.image)

创建静态服务器

建立一个空目录，在里面执行npm init -y，初始化项目。

执行npm i express下载 express。

然后建立一个server.js文件，代码如下：

constexpress =require('express')constapp = express()constport =8080app.use(express.static('dist'))app.listen(port,() =>{console.log(\`Example app listening at http://localhost:${port}\`)})复制代码

它将当前目录下的dist文件夹设为静态服务器资源目录，然后执行node server.js启动服务器。

由于现在没有dist文件夹，所以访问网站是空页面。

![](https://upload-images.jianshu.io/upload_images/4174952-f0bc909e3834b930.image)

不过不要着急，一会就能看到内容了。

自动构建 + 部署到服务器

下载 Jenkins 提供的 demo 项目[building-a-multibranch-pipeline-project](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjenkins-docs%2Fbuilding-a-multibranch-pipeline-project)，然后在你的 Gitea 新建一个仓库，把内容克隆进去，并提交到 Gitea 服务器。

2\. 打开 Jenkins 首页，点击新建 Item创建项目。

![](https://upload-images.jianshu.io/upload_images/4174952-f9226a4988ef51da.image)

3\. 选择源码管理，输入你的 Gitea 上的仓库地址。

![](https://upload-images.jianshu.io/upload_images/4174952-8ff7fb8cc2d88508.image)

![](https://upload-images.jianshu.io/upload_images/4174952-e299a083ec8ddbdc.image)

你也可以尝试一下定时构建，下面这个代码表示每 5 分钟构建一次。

![](https://upload-images.jianshu.io/upload_images/4174952-08ba308df6c1d633.image)

选择你的构建环境，这里选择刚才配置的 nodejs。

6\. 点击增加构建步骤，windows 要选execute windows batch command，linux 要选execute shell。

![](https://upload-images.jianshu.io/upload_images/4174952-85efc3add28f905f.image)

输入npm i && npm run build && xcopy .\\build\\\* G:\\node-server\\dist\\ /s/e/y，这行命令的作用是安装依赖，构建项目，并将构建后的静态资源复制到指定目录G:\\node-server\\dist\\ 。这个目录是静态服务器资源目录。

8\. 保存后，返回首页。点击项目旁边的小三角，选择build now。

9\. 开始构建项目，我们可以点击项目查看构建过程。

10\. 构建成功，打开http://localhost:8080/看一下结果。

11\. 由于刚才设置了每 5 分钟构建一次，我们可以改变一下网站的内容，然后什么都不做，等待一会再打开网站看看。

12\. 把修改的内容提交到 Gitea 服务器，稍等一会。打开网站，发现内容已经发生了变化。

使用 pipeline 构建项目

使用流水线构建项目可以结合 Gitea 的webhook钩子，以便在执行git push的时候，自动构建项目。

点击首页右上角的用户名，选择设置。

添加 token，记得将 token 保存起来。

打开 Jenkins 首页，点击新建 Item创建项目。

4\. 点击构建触发器，选择触发远程构建，填入刚才创建的 token。

5\. 选择流水线，按照提示输入内容，然后点击保存。

6\. 打开 Jenkins 安装目录下的jenkins.xml文件，找到<arguments>标签，在里面加上-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE\_CSRF\_PROTECTION=true。它的作用是关闭CSRF验证，不关的话，Gitea 的webhook会一直报 403 错误，无法使用。加好参数后，在该目录命令行下输入jenkins.exe restart重启 Jenkins。

7\. 回到首页，配置全局安全选项。勾上匿名用户具有可读权限，再保存。

打开你的 Gitea 仓库页面，选择仓库设置。

点击管理 web 钩子，添加 web 钩子，钩子选项选择Gitea。

目标 URL 按照 Jenkins 的提示输入内容。然后点击添加 web 钩子。

11\. 点击创建好的 web 钩子，拉到下方，点击测试推送。不出意外，应该能看到推送成功的消息，此时回到 Jenkins 首页，发现已经在构建项目了。

12\. 由于没有配置Jenkinsfile文件，此时构建是不会成功的。所以接下来需要配置一下Jenkinsfile文件。将以下代码复制到你 Gitea 项目下的Jenkinsfile文件。jenkins 在构建时会自动读取文件的内容执行构建及部署操作。

pipeline {    agent any    stages {        stage('Build') {            steps {  // window 使用 bat， linux 使用 sh                bat 'npm i'                bat 'npm run build'            }        }        stage('Deploy') {            steps {                bat 'xcopy .\\\\build\\\\\* D:\\\\node-server\\\\dist\\\\ /s/e/y' // 这里需要改成你的静态服务器资源目录            }        }    }}复制代码

每当你的 Gitea 项目执行push操作时，Gitea 都会通过webhook发送一个 post 请求给 Jenkins，让它执行构建及部署操作。

小结

如果你的操作系统是 Linux，可以在 Jenkins 打包完成后，使用 ssh 远程登录到阿里云，将打包后的文件复制到阿里云上的静态服务器上，这样就能实现阿里云自动部署了。具体怎么远程登录到阿里云，请看下文中的 《Github Actions 部署到阿里云》 一节。

Github Actions 自动构建前端项目并部署到服务器

如果你的项目是 Github 项目，那么使用 Github Actions 也许是更好的选择。

部署到 Github Page

接下来看一下如何使用 Github Actions 部署到 Github Page。

在你需要部署到 Github Page 的项目下，建立一个 yml 文件，放在.github/workflow目录下。你可以命名为ci.yml，它类似于 Jenkins 的Jenkinsfile文件，里面包含的是要自动执行的脚本代码。

这个 yml 文件的内容如下：

name:BuildandDeployon:# 监听 master 分支上的 push 事件push:branches:-masterjobs:build-and-deploy:runs-on:ubuntu-latest# 构建环境使用 ubuntusteps:-name:Checkoutuses:actions/checkout@v2.3.1with:persist-credentials:false-name:InstallandBuild# 下载依赖 打包项目run:|

          npm install

          npm run build-name:Deploy# 将打包内容发布到 github pageuses:JamesIves/github-pages-deploy-action@3.5.9# 使用别人写好的 actionswith:# 自定义环境变量ACCESS\_TOKEN:${{secrets.VUE\_ADMIN\_TEMPLATE}}# VUE\_ADMIN\_TEMPLATE 是我的 secret 名称，需要替换成你的BRANCH:masterFOLDER:distREPOSITORY\_NAME:woai3c/woai3c.github.io# 这是我的 github page 仓库TARGET\_FOLDER:github-actions-demo# 打包的文件将放到静态服务器 github-actions-demo 目录下复制代码

上面有一个ACCESS\_TOKEN变量需要自己配置。

打开 Github 网站，点击你右上角的头像，选择settings。

\>need-to-insert-img

点击左下角的developer settings。

\>need-to-insert-img

在左侧边栏中，单击Personal access tokens（个人访问令牌）。

\>need-to-insert-img

单击Generate new token（生成新令牌）。

输入名称并勾选repo。

拉到最下面，点击Generate token，并将生成的 token 保存起来。

打开你的 Github 项目，点击settings。

点击secrets->new secret。

创建一个密钥，名称随便填（中间用下划线隔开），内容填入刚才创建的 token。

将上文代码中的ACCESS\_TOKEN: ${{ secrets.VUE\_ADMIN\_TEMPLATE }}替换成刚才创建的 secret 名字，替换后代码如下ACCESS\_TOKEN: ${{ secrets.TEST\_A\_B }}。保存后，提交到 Github。

以后你的项目只要执行git push，Github Actions 就会自动构建项目并发布到你的 Github Page 上。

Github Actions 的执行详情点击仓库中的Actions选项查看。

具体详情可以参考一下我的 demo 项目**[github-actions-demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fwoai3c%2Fgithub-actions-demo)**。

\>need-to-insert-img

构建成功后，打开 Github Page 网站，可以发现内容已经发布成功。

Github Actions 部署到阿里云

初始化阿里云服务器

购买阿里云服务器，选择操作系统，我选的 ubuntu

在云服务器管理控制台选择实例->更多->密钥->重置实例密码（一会登陆用）

选择远程连接->VNC，会弹出一个密码，记住它，以后远程连接要用（ctrl + alt + f1~f6 切换终端，例如 ctrl + alt + f1 是第一个终端）

进入后是一个命令行 输入root（默认用户名），密码为你刚才重置的实例密码

登陆成功， 更新安装源sudo apt-get update && sudo apt-get upgrade -y

安装 npmsudo apt-get install npm

安装 npm 管理包sudo npm install -g n

安装 node 最新稳定版sudo n stable

创建一个静态服务器

mkdir node-server// 创建 node-server 文件夹cd node-server// 进入 node-server 文件夹npm init -y// 初始化项目npm i expresstouch server.js// 创建 server.js 文件vim server.js// 编辑 server.js 文件复制代码

将以下代码输入进去（用 vim 进入文件后按 i 进行编辑，保存时按 esc 然后输入 :wq，再按 enter），更多使用方法请自行搜索。

constexpress =require('express')constapp = express()constport =3388// 填入自己的阿里云映射端口，在网络安全组配置。app.use(express.static('dist'))app.listen(port,'0.0.0.0',() =>{console.log(\`listening\`)})复制代码

执行node server.js开始监听，由于暂时没有dist目录，先不要着急。

注意，监听 IP 必须为0.0.0.0，详情请看[部署Node.js项目注意事项](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.alibabacloud.com%2Fhelp%2Fzh%2Fdoc-detail%2F50775.htm)。

阿里云入端口要在网络安全组中查看与配置。

创建阿里云密钥对

请参考[创建SSH密钥对](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.alibabacloud.com%2Fhelp%2Fzh%2Fdoc-detail%2F51793.htm)和[绑定SSH密钥对](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.alibabacloud.com%2Fhelp%2Fzh%2Fdoc-detail%2F51796.htm%3Fspm%3Da2c63.p38356.879954.9.cf992580IYf2O7%23concept-zzt-nl1-ydb)，将你的 ECS 服务器实例和密钥绑定，然后将私钥保存到你的电脑（例如保存在 ecs.pem 文件）。

打开你要部署到阿里云的 Github 项目，点击 setting->secrets。

点击 new secret

secret 名称为SERVER\_SSH\_KEY，并将刚才的阿里云密钥填入内容。

点击 add secret 完成。

在你项目下建立.github\\workflows\\ci.yml文件，填入以下内容：

name:Buildappanddeploytoaliyunon:#监听push操作push:branches:# master分支，你也可以改成其他分支-masterjobs:build:runs-on:ubuntu-lateststeps:-uses:actions/checkout@v1-name:InstallNode.jsuses:actions/setup-node@v1with:node-version:'12.16.2'-name:Installnpmdependenciesrun:npminstall-name:Runbuildtaskrun:npmrunbuild-name:DeploytoServeruses:easingthemes/ssh-deploy@v2.1.5env:SSH\_PRIVATE\_KEY:${{secrets.SERVER\_SSH\_KEY}}ARGS:'-rltgoDzvO --delete'SOURCE:dist# 这是要复制到阿里云静态服务器的文件夹名称REMOTE\_HOST:'118.190.217.8'# 你的阿里云公网地址REMOTE\_USER:root# 阿里云登录后默认为 root 用户，并且所在文件夹为 rootTARGET:/root/node-server# 打包后的 dist 文件夹将放在 /root/node-server复制代码

保存，推送到 Github 上。

以后只要你的项目执行git push操作，就会自动执行ci.yml定义的脚本，将打包文件放到你的阿里云静态服务器上。

这个 Actions 主要做了两件事：

克隆你的项目，下载依赖，打包。

用你的阿里云私钥以 SSH 的方式登录到阿里云，把打包的文件上传（使用 rsync）到阿里云指定的文件夹中。

如果还是不懂，建议看一下我的[demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fwoai3c%2Fgithub-actions-aliyun-demo)。

作者：谭光志

链接：https://juejin.im/post/6887751398499287054

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。