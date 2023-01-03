# vue3 + vite中开发环境和生产环境全局变量配置_小绵杨Yancy的博客-CSDN博客_vite 全局变量
开发环境：也就是编码时运行的环境，即我们使用npm run dev或者npm run serve运行项目到本地时，项目处于的环境。

生产环境：[项目部署到服务器上](https://so.csdn.net/so/search?q=%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E5%88%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A&spm=1001.2101.3001.7020)后处于的环境，我们使用npm run build将项目打包以后，再运行项目，项目就运行在生产环境中了。

对于不同的环境，我们可以配置不同的环境变量，来实现开发和生产的兼容。

例如：  
开发环境时，我们可以请求自己本地的接口（‘/api’ proxy代理）。  
而部署到服务器上后，应该请求服务器提供的接口（‘http://xxxxxx/api/’ 真实接口）。  
我们通过设置axios的baseUrl可以实现，但是需要区分开发环境和生产环境，从而改变baseUrl。

在项目根目录下（与package.json同级）新建两个配置文件：  
![](https://img-blog.csdnimg.cn/c3cf05e2b3d14f8ba83149c94a4fc5b1.png)

```bash
.env.development：开发环境下的配置文件,执行npm run dev命令，会自动加载.env.development文件.

.env.production：生产环境下的配置文件,执行npm run build命令，会自动加载.env.production文件


```

.env.development文件：

```bash
ENV = 'development'

VITE_BASE_URL='/api'


```

.env.production

```bash
ENV = 'production'

VITE_BASE_URL = 'http://xxxxxx/api/'


```

这里的VITE\_BASE\_URL是项目上线后需要请求的服务器接口。

与vue-cli引用不同，vue-cli引用为：

```bash
process.env.变量名

```

而[vite](https://so.csdn.net/so/search?q=vite&spm=1001.2101.3001.7020)引用为:

```bash
import.meta.env.变量名

```

在配置axios时使用全局baseUrl:

```javascript
const service = axios.create({
    baseURL: import.meta.env.VITE_BASE_URL,
    timeout: 5000
})

```