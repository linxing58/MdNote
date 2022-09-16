# Gitea+Jenkins+webhooks-多分支触发+备份构建前端资源_一个运维小青年的博客-CSDN博客_sshpublisherdesc
### Gitea+[Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020)+webhooks-前端自动化部署

###### jenkins中文汉化文档

```shell
https://blog.csdn.net/qq_37489565/article/details/104337073

安装Locale插件=>设置语言为zh_US=>重启=>设置语言为zh_CN=>刷新页面就可以了

```

**Jenkins的插件安装，在插件管理中安装Generic Webhook Trigger用于gitea构建触发器，Blue Ocean可以理解为Jenkins的一个皮肤（个人感觉界面看起来和操作使用很舒服）**

![](https://img-blog.csdnimg.cn/img_convert/992384fe6704e1112a1c726370bb558a.png)

安装插件  
![](https://img-blog.csdnimg.cn/img_convert/5d95d0b2a13e0747dd1231097de70554.png)

###### 第二个插件

![](https://img-blog.csdnimg.cn/img_convert/6a9c722bbfe42e64c63af5693ac78aa2.png)

###### 第三个插件

![](https://img-blog.csdnimg.cn/img_convert/b680abf8315476110ab6f48735d8212f.png)

**关于Jenkins准备工作，已经基本做完，下一步将使用gitea中的webhooks与Jenkins进行联系，达到代码自动部署的效果**

**首先，我们先点新建任务，进入到任务列表，接下来，我将分享两种构建的方式，分别是批处理命令构建和pipeline流水线语法的方式构建**

![](https://img-blog.csdnimg.cn/img_convert/f664d2c793706e82e96108d2a08f2ced.png)

###### 创建任务

![](https://img-blog.csdnimg.cn/img_convert/809a5e87e3ca37eb76827a00e23a9bc1.png)

### 自由风格项目构建

**在任务配置中输入gitea clone的地址，并且点击新建验证方式，我这里用的是用户名密码**

![](https://img-blog.csdnimg.cn/img_convert/80c4e17095fb015f0f86e58e70b0a89b.png)

###### 构建[触发器](https://so.csdn.net/so/search?q=%E8%A7%A6%E5%8F%91%E5%99%A8&spm=1001.2101.3001.7020)

```shell
http://JENKINS_URL/generic-webhook-trigger/invoke

```

![](https://img-blog.csdnimg.cn/img_convert/e01417f7854abb0a5c0d955a836b51e5.png)

###### TOken

```shell
vue_vite

```

![](https://img-blog.csdnimg.cn/img_convert/adea5d370c8fd4c217559df7e101ce95.png)

**然后在仓库设置中添加web钩子，设置请求的地址，地址与Jenkins构建触发器中示例地址一致**

![](https://img-blog.csdnimg.cn/img_convert/664495d625dd0e0d4a168f24b81fe844.png)

###### web钩子设置

![](https://img-blog.csdnimg.cn/img_convert/e60fb254723f090b724725b72e2eb27f.png)

配置选择  
![](https://img-blog.csdnimg.cn/img_convert/0bfdcf3dbc7d5ccea201ded70ae285d8.png)

###### **触发条件**

**我选择的是推送，即当前仓库收到推送信息就会通过webhooks通知Jenkins构建项目，最后测试一下是否能正常请求，请求成功后就会执行构建**

![](https://img-blog.csdnimg.cn/img_convert/2e9a9a8e4481f529a23ed09af979f3a3.png)

![](https://img-blog.csdnimg.cn/img_convert/161275fa037a6d257d42ba62cd0d99ed.png)

###### 我的完整成功推送

```
http://192.168.2.202:8080/jenkins/generic-webhook-trigger/invoke?token=dajiba

```

![](https://img-blog.csdnimg.cn/img_convert/63b2303b301f71db9dc577f0cf656d8a.png)

###### 配置构建

构建内容

```shell




cd /root/.jenkins/workspace/webhook-111111


npm  install 


npm run build


```

![](https://img-blog.csdnimg.cn/img_convert/9b722ac2592d807015270d6453648d85.png)

###### 更新代码，测试更新

随便更新一些代码上去

![](https://img-blog.csdnimg.cn/img_convert/6484665ddc59437e74d6c6a312674e95.png)

###### 上传代码

![](https://img-blog.csdnimg.cn/img_convert/adb75d5c425026e86e3289ed144300e2.png)

###### 发现jenkins在自动更新

![](https://img-blog.csdnimg.cn/img_convert/5e0a213ee50c65f48ada2becff83eb2d.png)

### 构建pipeline流水线语法

###### 创建任务

![](https://img-blog.csdnimg.cn/img_convert/9bcadd67853e270b2672192a9bd062ff.png)

###### 构建触发器

![](https://img-blog.csdnimg.cn/img_convert/b462ae90bd768f018791ee454328f48a.png)

###### 配置一个参数，如果以后还有项目，构建的时候可以灵活选择

```shell
single_project_name

/root/.jenkins/workspace/webhooks-rock

```

![](https://img-blog.csdnimg.cn/img_convert/20cccd78b0f16e2f133471bb93642127.png)

###### 对应定义的值

![](https://img-blog.csdnimg.cn/img_convert/17a522429c42cd83c74b7e83c1942ce2.png)

###### 发布到服务器语法

![](https://img-blog.csdnimg.cn/img_convert/82d25cb89216b5a4a87966f5d03b4c85.png)

###### **然后就是构建流水线脚本，这里放出我配置的一段供大家参考。** 

```
`pipeline { 
   agent any 
   stages { 
      stage('拉取代码') { 
          steps { 
              checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '7f49313d-880d-4a5e-836f-cef4bf2ec37a', url: 'http://192.168.2.204:3000/aike/test.git']]])
            } 
        }
      stage('选择node版本编译打包') { 
             steps { 
              nodejs('node') {
                sh '''cd ${single_project_name}
                      npm  install 
                      npm run build
                      tar -zcvf  ./front.end-levee.tar.gz   ./dist'''
              }
            } 
        } 
      stage('发布到服务器') { 
             steps { 
              sshPublisher(publishers: [sshPublisherDesc(configName: '192.168.2.204', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
            cd /a
            tar -xzf front.end-levee.tar.gz -C ./
            cp -r dist/* ./ 
            rm -rf   front.end-levee.tar.gz''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/a', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'front.end-levee.tar.gz')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            } 
        }             
   }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29


```

### 构建可以选择对应目录

![](https://img-blog.csdnimg.cn/img_convert/37e1e5b871ae80567b6357f3440db4c4.png)

#### 最终完整构建

![](https://img-blog.csdnimg.cn/img_convert/336a621bc17a66b6601b1ee616354dca.png)

###### 测试推送更新代码

![](https://img-blog.csdnimg.cn/img_convert/27ba0c4e497ae9bb2ddcff0754aaf29f.png)

###### 最终成功图

![](https://img-blog.csdnimg.cn/img_convert/306576e02f17c893b7417764714607d9.png)

### 备份+触发+pipeline，完整版

```
`这里讲一下我遇到的问题，
gitea仓库配置token，测试访问测试一直失败，连接超时

解决方法：确认jenkins是内网环境，gitea是公网环境，jenkins是能拉到公网的代码，需要设置内网穿透，把jenkins映射成公网

解决gitea仓库webhook钩子测试不成功问题

遇到问题：测试访问token     测试一直失败，连接超时
解决方法-1：采用小蝴蝶内网穿透，把jenkins映射成公网，配置gitea仓库-web钩子测试还是连接超时，
访问token是可以访问到的，猜想可能是跟gitea仓库有关系

解决方法-2：采用gitlab配置web钩子，去测试访问jenkins-token是可以正常访问的，把gitlab内网穿透
映射成公网，模拟gitea仓库进项配置访问测试，也是可以访问到Jenkins的

解决方法-3：配置公网gitee仓库  web钩子，去测试访问jenkins-token是可以正常访问的，
模拟gitea仓库进项配置访问测试，也是可以访问到Jenkins的

解决方法-4：把自己搭建的gitea仓库做个内网穿透，模拟公司gitea仓库放在公网，然后内网jenkins去拉取代码，
再把jenkins做个内网穿透，模拟gitea--web钩子测试token是否可以发送成功，测试是正常的

最终解决问题
把jenkins用花生壳做内网穿透，隐射成https去访问，配置公司gitea仓库的web钩子，再次测试token，
配置成映射的公网域名，访问不到，打开花生壳访问ip设置，把刚才的访问ip设置白名单，访问正常问题解决` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23


```

###### jenkins-pipeline完成截图配置

```shell
这个方法是把jenkinsfile放到代码仓库里面，jenkins回去识别代码里面的jenkinsfile文件，然后按照内容去执行

```

![](https://img-blog.csdnimg.cn/img_convert/e81f13224d49e5d782c25b96090bf9d0.png)

###### jenkinsfile文件存放在代码仓库位置

![](https://img-blog.csdnimg.cn/img_convert/376a0403a961e17a33a566eba4598537.png)

###### jenkinsfile文件样本

```
`pipeline { 
   agent any 
      triggers {
    GenericTrigger (
            // 构建时的标题
            causeString: 'Triggered by $ref',
            // 获取POST参数中的变量，key指的是变量名，通过$ref来访问对应的值，value指的是JSON匹配值（参考Jmeter的JSON提取器）
            // ref指的是推送的分支，格式如：refs/heads/master
            genericVariables: [[key: 'ref', value: '$.ref']],
            // 打印获取的变量的key-value，此处会打印如：ref=refs/heads/master
            printContributedVariables: true,
            // 打印POST传递的参数
            printPostContent: true,
            // regexpFilterExpression与regexpFilterExpression成对使用
            // 当两者相等时，会触发对应分支的构建
            regexpFilterExpression: '^refs/heads/(test1|release-test)$',
            regexpFilterText: '$ref',
            // 与webhook中配置的token参数值一致
            token: 'webhooks-river'
    )
}
   stages { 
    	stage("测试部署") {
            when {
                branch 'release-test'
            }
    	    steps {
                echo 'release-test branch'
    	    }
    	}
    	stage("生产部署") {
            when {
                branch 'test1'
            }
    	    steps {
                echo 'test1 branch'
    	    }
    	}
        stage('选择node版本编译打包') { 
             steps { 
              nodejs('node') {
                sh '''cd ${single_project_name}
                      npm  install 
                      npm run build
                      tar -zcvf  ./front.end-river.tar.gz   ./dist'''
              }
            } 
        } 
        stage('发布到服务器') { 
             steps { 
              sshPublisher(publishers: [sshPublisherDesc(configName: 'xxxxxxxxxx', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 
            '''source /etc/profile
            bash /back/river/back.sh
            find /data/cnhuafas/html/bim  -type f -not -name \'*.gz\' -delete            
            cd /data/cnhuafas/html/bim
            tar -xzf front.end-river.tar.gz -C ./
            cp -r dist/* ./ 
            rm -rf   front.end-river.tar.gz''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/a', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'front.end-river.tar.gz')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            } 
        }             
   }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62


```

###### gitea钩子设置

![](https://img-blog.csdnimg.cn/img_convert/912d8942fff7b7a2d65518b05ccdd8eb.png)

###### 测试通过

![](https://img-blog.csdnimg.cn/img_convert/4b0852cb7da8b3f6ae3af5ec8b43d2f8.png)

###### jenkins做的内穿穿透

花生壳

![](https://img-blog.csdnimg.cn/img_convert/a77ebc059e03e1517d52eefe65c06158.png)

###### jenkins构建的备份shell脚本

备份前端静态资源脚本，自己看着修改

```
``#!/bin/sh

dateTime=`date +%Y-%m-%d`
days=7
bakuser=user1
bakdir=/data/backupdata                 
bakdata=${bakuser}_${dateTime}.tar.gz   
baklog=${bakuser}_${dateTime}.log       

baksrcdir=/db/mysql/data                

remotePath=/backupdata/dbdata           
remoteIP="192.168.56.66"

cd ${bakdir}
mkdir -p ${bakuser}
cd ${bakuser}

echo "backup start at ${dateTime}" > ${baklog}  
echo "---------------------------" >> ${baklog}

tar -zcvf ${bakdata} ${baksrcdir} >> ${baklog}

find ${bakdir}/${bakuser} -type f -name "*.log" -exec rm -rf {} \;

find ${bakdir}/${bakuser} -type f -name "*.tar.gz" -mtime +$days -exec rm -rf {} \;

rsync -avzPL ${bakdir}/${bakuser}/${bakdata} ${remoteIP}:${remotePath}`` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33


```

###### 测试成功截图

![](https://img-blog.csdnimg.cn/img_convert/a003f58d0578eac299aab4d3dd09bdc1.png)

### 第二种直接写pipeline方式

完整版截图  
![](https://img-blog.csdnimg.cn/img_convert/34bcd2f7d3501dc799bc5df5a33b71e2.png)

###### jenkins-pipeline语法

```
`pipeline { 
   agent any 
   stages { 
      stage('拉取代码') { 
          steps { 
              checkout([$class: 'GitSCM', branches: [[name: '${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: '100b584e-7f67-466f-8b93-9b9038e117a0', url: 'xxxxxx']]])
            } 
        }
      stage('选择node版本编译打包') { 
             steps { 
              nodejs('nodejs') {
                sh '''cd ${single_project_name}
                      cnpm  install 
                      npm run build
                      tar -zcvf  ./front.end-rock.tar.gz   ./dist'''
              }
            } 
        } 
      stage('发布到服务器') { 
             steps { 
            sshPublisher(publishers: [sshPublisherDesc(configName: 'xxxxxxx', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''source /etc/profile
            bash /back.sh
            find /data/cnhuafas/html/zeus  -type f -not -name \'*.gz\' -delete
            cd /data/cnhuafas/html/zeus
            tar -xzf front.end-rock.tar.gz -C ./
            cp -r dist/* ./ 
            rm -rf   front.end-rock.tar.gz
            ''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/data/cnhuafas/html/zeus', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'front.end-rock.tar.gz')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            } 
        }             
   }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32


```

ck.tar.gz  
‘’', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: ‘\[, \]+’, remoteDirectory: ‘/data/cnhuafas/html/zeus’, remoteDirectorySDF: false, removePrefix: ‘’, sourceFiles: ‘front.end-rock.tar.gz’)\], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)\])  
}  
}  
}  
}

```

脚本其他的都跟上面一样的，谢谢查阅此文章，此文章填坑很多，动动你发财小手关注一下吧

```