# Keycloak快速上手指南，只需10分钟即可接入Spring Boot/Vue前后端分离应用实现
登录及身份认证是现代 web 应用最基本的功能之一，对于企业内部的系统，多个系统往往希望有一套 SSO 服务对企业用户的登录及身份认证进行统一的管理，提升用户同时使用多个系统的体验，Keycloak 正是为此种场景而生。本文将简明的介绍 Keycloak 的安装、使用，并给出目前较流行的前后端分离应用如何快速接入 Keycloak 的示例。

## Keycloak 是什么

> Keycloak 是一种面向现代应用和服务的开源 IAM（身份识别与访问管理）解决方案

Keycloak 提供了单点登录（SSO）功能，支持`OpenID Connect`、`OAuth 2.0`、`SAML 2.0`标准协议，拥有简单易用的管理控制台，并提供对 LDAP、Active Directory 以及 Github、Google 等社交账号登录的支持，做到了非常简单的开箱即用。

## Keycloak 常用核心概念介绍

首先通过官方的一张图来了解下整体的核心概念

![](https://s4.51cto.com/images/blog/202007/20/f1e6ef3f999fb55fec5f6826a0b75be5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

这里先只介绍 4 个最常用的核心概念：

1.  `Users`: 用户，使用并需要登录系统的对象
2.  `Roles`: 角色，用来对用户的权限进行管理
3.  `Clients`: 客户端，需要接入 Keycloak 并被 Keycloak 保护的应用和服务
4.  `Realms`: 领域，领域管理着一批用户、证书、角色、组等，一个用户只能属于并且能登陆到一个域，域之间是互相独立隔离的， 一个域只能管理它下面所属的用户

## Keycloak 服务安装及配置

### 安装 Keycloak

Keycloak 安装有多种方式，这里使用 Docker 进行快速安装

````null
docker run -d --name keycloak \
    -p 8080:8080 \
    -e KEYCLOAK_USER=admin \
    -e KEYCLOAK_PASSWORD=admin \
    jboss/keycloak:10.0.0```

访问[](http://localhost:8080/auth/)[http://localhost:8080](http://localhost:8080/)并点击Administration Console进行登录

![](https://s4.51cto.com/images/blog/202007/20/bcc3adcd582ecefbb8dec1e02e17d213.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 创建Realm

创建一个新的realm: demo，后续所有的客户端、用户、角色等都在此realm中创建

![](https://s4.51cto.com/images/blog/202007/20/4d0acb9a245b361c63b4bbb35f520fa2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

![](https://s4.51cto.com/images/blog/202007/20/378599ea54e1dc03ce0f3a6283ee9ef5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

![](https://s4.51cto.com/images/blog/202007/20/874172151d09e6206d73ac5fe9058391.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 创建客户端

#### 创建前端应用客户端

创建一个新的客户端：vue-demo，Access Type选择public

![](https://s4.51cto.com/images/blog/202007/20/18a5ba7df92d2eb252484eab46e602eb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 创建后端应用客户端

创建一个新的客户端：spring-boot-demo，Access Type选择bearer-only

![](https://s4.51cto.com/images/blog/202007/20/2af99e3c8114b6860252f9a2e109dc54.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

保存之后，会出现Credentials的Tab，记录下这里的secret，后面要用到

![](https://s4.51cto.com/images/blog/202007/20/e20bf8709adea2a9e4c4e0833a92b53d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 关于客户端的访问类型（Access Type）

上面创建的2个客户端的访问类型分别是public、bearer-only，那么为什么分别选择这种类型，实际不同的访问类型有什么区别呢？

事实上，Keycloak目前的访问类型共有3种：

`confidential`：适用于服务端应用，且需要浏览器登录以及需要通过密钥获取`access token`的场景。典型的使用场景就是服务端渲染的web系统。

`public`：适用于客户端应用，且需要浏览器登录的场景。典型的使用场景就是前端web系统，包括采用vue、react实现的前端项目等。

`bearer-only`：适用于服务端应用，不需要浏览器登录，只允许使用`bearer token`请求的场景。典型的使用场景就是restful api。

### 创建用户和角色

#### 创建角色

创建2个角色：ROLE\_ADMIN、ROLE\_CUSTOMER

![](https://s4.51cto.com/images/blog/202007/20/7753e1a42403c8e7ad2ea4119f249a48.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 创建用户

创建2个用户：admin、customer

![](https://s4.51cto.com/images/blog/202007/20/4ad5b9bddc46ce0a9625353208524231.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 绑定用户和角色

##### 给admin用户分配角色ROLE\_ADMIN

![](https://s4.51cto.com/images/blog/202007/20/bd7c5e12ac640bb8e2b7b5ee5b14f1fb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

##### 给customer用户分配角色ROLE\_CUSTOMER

![](https://s4.51cto.com/images/blog/202007/20/ed829d3b24196ed95d11adb2974fcfda.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

Vue应用集成Keycloak简明指南
-------------------

### 创建vue项目

```null
vue create vue-demo
````

### 添加官方 Keycloak js 适配器

```null
npm i keycloak-js --save
npm i axios --save
```

### main.js

```null
import Vue from 'vue'
import App from './App.vue'
import Keycloak from 'keycloak-js'

Vue.config.productionTip = false


const initOptions = {
  url: 'http://127.0.0.1:8080/auth',
  realm: 'demo',
  clientId: 'vue-demo',
  onLoad:'login-required'
}

const keycloak = Keycloak(initOptions)

keycloak.init({ onLoad: initOptions.onLoad, promiseType: 'native' }).then((authenticated) =>{
  if(!authenticated) {
    window.location.reload();
  } else {
    Vue.prototype.$keycloak = keycloak
    console.log('Authenticated')
  }

  new Vue({
    render: h => h(App),
  }).$mount('#app')

  setInterval(() =>{
    keycloak.updateToken(70).then((refreshed)=>{
      if (refreshed) {
        console.log('Token refreshed');
      } else {
        console.log('Token not refreshed, valid for '
            + Math.round(keycloak.tokenParsed.exp + keycloak.timeSkew - new Date().getTime() / 1000) + ' seconds');
      }
    }).catch(error => {
      console.log('Failed to refresh token', error)
    })
  }, 60000)

}).catch(error => {
  console.log('Authenticated Failed', error)
})
```

### HelloWorld.vue

```null
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div>
      <p>
        current user: {{user}}
      </p>
      <p>
        roles: {{roles}}
      </p>
      <p>
        {{adminMsg}}
      </p>
      <p>
        {{customerMsg}}
      </p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      user: '',
      roles: [],
      adminMsg: '',
      customerMsg: ''
    }
  },
  created() {
    this.user = this.$keycloak.idTokenParsed.preferred_username
    this.roles = this.$keycloak.realmAccess.roles

    this.getAdmin()
            .then(response=>{
              this.adminMsg = response.data
            })
            .catch(error => {
              console.log(error)
            })

    this.getCustomer()
            .then(response => {
              this.customerMsg = response.data
            })
            .catch(error => {
              console.log(error)
            })
  },
  methods: {
    getAdmin() {
      return axios({
        method: 'get',
        url: 'http://127.0.0.1:8082/admin',
        headers: {'Authorization': 'Bearer ' + this.$keycloak.token}
      })
    },
    getCustomer() {
      return axios({
        method: 'get',
        url: 'http://127.0.0.1:8082/customer',
        headers: {'Authorization': 'Bearer ' + this.$keycloak.token}
      })
    }
  }
}
</script>
```

`getAdmin()`及`getCustomer()`这 2 个方法内部分别请求 restful api

## Spring Boot 应用集成 Keycloak 简明指南

### 添加 Keycloak Maven 依赖

```null
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-starter</artifactId>
    <version>10.0.0</version>
</dependency>
```

### Spring Boot 配置文件

官方文档及网上大部分示例使用的都是 properties 格式的配置文件，而 yaml 格式的配置文件相对更简洁清晰些，此示例使用 yaml 格式的配置文件，内容如下

```null
server:
  port: 8082
keycloak:
  realm: demo
  auth-server-url: http://127.0.0.1:8080/auth
  resource: spring-boot-demo
  ssl-required: external
  credentials:
    secret: 2d2ab498-7af9-48c0-89a3-5eec929e462b
  bearer-only: true
  use-resource-role-mappings: false
  cors: true
  security-constraints:
    - authRoles:
        - ROLE_CUSTOMER
      securityCollections:
        - name: customer
          patterns:
            - /customer
    - authRoles:
        - ROLE_ADMIN
      securityCollections:
        - name: admin
          patterns:
            - /admin
```

除了几个必填的配置项外，另外需要注意的几个配置项如下

`credentials.secret`：上文添加客户端后 Credentials Tab 内对应的内容

`bearer-only`：设置为 true，表示此应用的 Keycloak 访问类型是 bearer-only

`cors`：设置为 true 表示允许跨域访问

`security-constraints`：主要是针对不同的路径定义角色以达到权限管理的目的

-   `/customer`：只允许拥有`ROLE_CUSTOMER`角色的用户才能访问
-   `/admin`：只允许拥有`ROLE_ADMIN`角色的用户才能访问
-   未配置的路径表示公开访问

### Controller 内容

```null
@RestController
public class HomeController {
    @RequestMapping("/")
    public String index() {
        return "index";
    }

    @RequestMapping("/customer")
    public String customer() {
        return "only customer can see";
    }

    @RequestMapping("/admin")
    public String admin() {
        return "only admin cas see";
    }
}
```

## 项目效果演示

分别启动前后端项目后，本地 8081 端口对应 vue 前端项目，本地 8082 端口对应 Spring Boot 实现的 restful api 项目

### 首次访问 vue 前端项目

第一次访问 vue 项目会跳转 Keycloak 登录页

![](https://s4.51cto.com/images/blog/202007/20/7202762c099f8460b0c667d27dc620c4.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 登录 admin 用户

![](https://s4.51cto.com/images/blog/202007/20/552d0d949b9162979f34f17940a2637f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 登录 customer 用户

![](https://s4.51cto.com/images/blog/202007/20/b65c6980e3b93807d009a9a483ab0587.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

## 总结

Keycloak 部署及接入简单，轻量的同时功能又不失强大，非常适合企业内部的 SSO 方案。

来源：`51CTO`

作者：`Java_老男孩`

链接：`https://blog.51cto.com/14230003/2511837` 
 [https://www.e-learn.cn/topic/3690763](https://www.e-learn.cn/topic/3690763)
