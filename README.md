# 搭建vue开发环境
# 1.安装node
## (1) 下载
## (2) 新增 node_global 和 node_cache 文件夹，打开cmd(需要在管理员权限打开cmd，否则添加命令无效)，输入以下命令
  ```js
  npm config set prefix "D:\node\node_global"
  npm config set cache "D:\node\node_cache"
  ```
## (3) 配置环境变量
  用户变量 path ：C:\Program Files\nodejs\node_global(如果为node_modules导致cnpm无法访问)
  系统变量 NODE_PATH ：C:\Program Files\nodejs\node_global\node_modules
  path：加入NODE_PATH
## (4)检验安装是否成功：node -v
  若出现"'node' 不是内部或外部命令,也不是可运行的程序 或批处理文件"错误，说明环境变量没有配置成功
# 2.安装cnpm(同样需要在管理员权限下安装)
## (1) npm install -g cnpm --registry=https://registry.npm.taobao.org
## (2) cnpm -v 
# 3.安装vue-cli脚手架
## (1) 安装vue
 ```js
 cnpm install vue -g
 vue -V
  ```
## (2)安装vue-cli
  cnpm install @vue/cli -g
问题1：Install fail! RunScriptError: post install error, please remove node_modules before retry!
解决：
# 4.创建新项目
  如果你仍然需要使用旧版本的 vue init 功能，需要执行npm install -g @vue/cli-init
  ```js
  npm install -g @vue/cli-init
  vue init webpack my-project
  ```
# 5.安装工程依赖模块
  npm install
# 6.启动项目
  npm run dev
# 按钮权限
两种解决办法:1.定义一个全局方法,配合v-if实现;2.使用自定义指令
## 1.定义一个全局方法,配合v-if实现;
在用户登录成功后,获取用户的按钮权限(数组格式),存储到store中
定义公共函数hasPermission
```js
  export function hasPermission(permission){
    let buttons = stores.getters.btns();
    return buttons.indexOf(permission);
  }
```
在main.js中引入
```js
import {hasPermission} from '.utils/hasPermission'
Vue.prototype.hasPerm = hasPermission;
```
在需要的按钮上使用即可
```js
<el-button v-if="hasPerm('sys:role:add')" type="primary" @click="addRole">
```
## 2.自定义指令：directives在全局main.js中注册
```js
//自定义指令
const has = Vue.directive('has',{})
// 权限检查方法
Vue.prototype.$_has = function(value){}
//暴露指令
export {has};
/*然后在main.js文件引入文件*/
import has from'./public/js/btnPermissions.js';
/*页面中按钮只需加v-has即可*/
<el-button @click='editClick' type="primary" v-has>编辑</el-button>
```
> 注意：自定义指令时如果使用bind：如果按钮权限不存在，删除按钮语句el.parentNode.removeChild(el)报错，因为el.parentNode为null，页面未渲染完成，解决方法：把bind改成inserted/ update /componentUpdated 就解决了
# 接口权限
一般使用jwt验证接口权限，登录后拿到token，并将token保存起来，再使用axios拦截器进行拦截，每次请求时头部携带token，如果没有则返回401，跳转到登录页面重新登录。
```js
  axios.interceptors.request.use(config => {
    config.headers['token'] = cookie.get('token')
    return config
  })
  axios.interceptors.response.use(res=>{},{response}=>{
    if (response.data.code === 40099 || response.data.code === 40098) { //token过期或者错误
        router.push('/login')
    }
  })
```
# 路由权限验证
## 1.通过router.beforeEach() 路由拦截的方式实现。
这种方式依赖于我们项目的路由表都是事先配置好的
![image](https://user-images.githubusercontent.com/46916809/182597119-676651be-c6a9-44fd-9fc0-a214043783b3.png)
在 router/index 中，通过router.beforeEach() 路由拦截去进行权限判断
```js
router.beforeEach((to, from, next) => {
    //to: 从哪个路由来
    //from: 去哪个路由
    //next：是一个方法，使用路由拦截，必须在后面添加next()，否则路由无法跳转
    //假设我们从后台获取的权限为：
    const list = ['index1', 'index2'];
    //如果没有匹配到，证明没有权限
    if(list.indexOf(to.name) === -1) {
        next('/login');
        ... //或者执行其他操作
    }
})
```
## 2.通过vue-router 官方提供的addRoutes()来进行动态路由注入，该方法只有vue-router的版本 >= 2.2才有效。
初始化的时候先挂载不需要权限控制的路由，比如登录页，404等错误页。如果用户通过URL进行强制访问，则会直接进入404，相当于从源头上做了控制
> 登录后，获取用户的权限信息，然后筛选有权限访问的路由，在全局路由守卫里进行调用addRoutes添加路由。
```js
  // permission judge function
  
```
