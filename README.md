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
## 1.定义一个全局方法,配合v-if实现
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
这种方式依赖于我们项目的路由表都是事先配置好的:
```js
const router = new Router({
  routes: [{
        path: '/',
        redirect: '/index1'
    }, {
        path: '/index1',
        name: 'Index1',
        component: Index1
    }, {
        path: '/index2',
        name: 'Index2',
        component: Index2
    }, {
        path: '/index3',
        name: 'Index3',
        component: Index3
    }]
})
export default router;
```
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
const createRouter = () => new Router({
    // mode: 'history', // require service support
    scrollBehavior: () => ({ y: 0 }),
    routes: constantRoutes
});

const router = createRouter();
/** 然后我们访问A帐号有，B帐号没有的路由时,
重新实例化一个新的路由表，替换之前的路由表，然后将这个方法导出
重新拷贝固定路由，matcher 清除addRoutes添加的路由，不刷新的情况下，登录时初始化路由
*/
export function resetRouter() {
    const newRouter = createRouter();
    router.matcher = newRouter.matcher; // the relevant part
}
  // permission judge function
  function hasPermission(roles, permissionRoles) {
    if (roles.indexOf("admin") >= 0) return true; // admin permission passed directly
    if (!permissionRoles) return true;
    //判断动态路由中的meta中是否包含你当前用户的权限
    return roles.some(role => permissionRoles.indexOf(role) >= 0);
}
const whiteList = ["/login", "/authredirect"];

router.beforeEach((to, from, next) => {
    NProgress.start(); // start progress bar
    let token = sessionStorage.getItem("token");
    console.log({})
    if (token) {
        // determine if there has token
        /* has token*/
        if (to.path === "/login") {
            // next();
            //next({ path: '/' }) 会导致栈溢出?
            next({ path: '/login' })
            NProgress.done(); // if current page is dashboard will not trigger	afterEach hook, so manually handle it
        } else {
            if (store.getters.roles.length === 0) {
                // 先请求获取用户信息
                store.dispatch("getUserInfo").then(res => {
                    // 获取用户角色 note: roles must be a array! such as: ['editor','develop']
                    // const roles = res.data.roles;
                    const roles = res;
                    // 根据roles权限生成可访问的路由表
                    store.dispatch("generateRoutes", roles).then(() => {
                        // 动态添加可访问路由表,注意:保持命名一致,否则store.getters.addRoutes为undefined
                        // 清除动态路由,解决刷新页面路由重复
                        resetRouter();
                        router.addRoutes(store.getters.addRoutes);
                        next({...to, replace: true }); // hack方法 确保addRoutes已完成 ,
                    });
                }).catch(err => {
                    // store.dispatch("resetToken")
                    // Message.error(err || "Verification failed, please login again");
                    // next({ path: "/" });
                    store.dispatch("resetToken").then(() => {
                        Message.error(err || "Verification failed, please login again");
                        next({ path: "/" });
                    });
                });
            } else {
                // 没有动态改变权限的需求可直接next() 删除下方权限判断
                if (hasPermission(store.getters.roles, to.meta.roles)) {
                    next(); //
                } else {
                    next({ path: "/401", replace: true, query: { noGoBack: true } });
                }
                // 可删
            }
        }
    } else {
        /* has no token*/
        if (whiteList.indexOf(to.path) !== -1) {
            // 在免登录白名单，直接进入
            next();
        } else {
            next("/login"); // 否则全部重定向到登录页
            NProgress.done(); // if current page is login will not trigger afterEach hook, so manually handle it
        }
    }
});
```
### 问题一：vue路由跳转错误：Error: Redirected when going from "/login" to "/home" via a navigation guard.第一次点击登陆出现错误，第二次点击登陆正常进入主页
> 原因：先触发了守卫路由，然后再放置token，导致路由守卫中获取不到token就进行'/login'跳转
 解决方法：把push路由的方法放到存储token信息以后即可。
### 问题二：栈溢出
> 原因：命名错误，导致变量为undefined
# 菜单权限
## 1. 登录的时候后端就返回菜单，进到主界面的时候就进行了渲染
```js
/* 动态路由 */
export const asyncRoutes = [
  {
    path: "*",
    component: () => import('@/views/error-page/404'),
  }
]
/* 使用钩子函数对路由进行权限跳转 */
try {
    store.dispatch('user/getInfo').then(() => { // 拉取info
        store.dispatch('permission/generateRoutes').then(res => { // 生成可访问的路由表
            router.addRoutes(res) // 动态添加可访问路由表
            const redirect = decodeURIComponent(from.query.redirect || to.path)
            next({...to, replace: true })
        })
    }).catch(err => {
        console.log(err);
    })
}
/* 请求后台获取菜单，解析后加入到动态路由 */
const state = {
  routes: [],
  addRoutes: [],
  btnRoles: []
}
getters: {
  roles: state => state.roles,
  addRoutes: state => state.addRoutes
  permission_routes: state => state.routes
},
const mutations = {
  SET_ROUTES: (state, routes) => {
    state.addRoutes = routes
    state.routes = constantRoutes.concat(routes)
  },
}

/**
 * 由data生成权限菜单
 * @routes AsyncRoutes
 * @data  后端请求的菜单列表
 */
 export function generaMenu(routes, data) {
    data.forEach(item => {
        const menu = {
            path: item.url,
            children: [],
            name: item.name,
            isShow: true,
            meta: { title: item.name, keepAlive: true }
        }
        if (item.url.indexOf("token") >= 0) {
            menu.path = item.url + getToken()
        }
        if (item.component != null && item.component !== '') {
            menu.component = item.component === 'Layout' ? Layout : resolve => require([`@/views${item.component}`],             resolve)
        }
        if (item.redirect != null && item.redirect !== '') {
            menu.redirect = item.redirect
        }
        if (item.icon != null && item.icon !== '') {
            menu.meta.icon = item.icon
        } else {
            menu.meta.icon = 'task'
        }
        if (item.component === 'Layout') {
            generaMenu(menu.children, item.childList)
        }
        routes.push(menu)
    })
}

const actions = {
    generateRoutes({ commit }) {
        return new Promise(resolve => {
            getUserMenu().then(response => {
                let asyncBtns = []
                let menuArr = response.data.object.menuList
                let btnArr = response.data.object.btnList
                generaMenu(asyncRoutes, menuArr)
                console.log(asyncRoutes)
                commit('SET_ROUTES', asyncRoutes)
                resolve(asyncRoutes)
            })
        })
    }
}
根据store的路由数据，渲染菜单
import { mapGetters } from 'vuex'
computed: {
    ...mapGetters([
        'permission_routes',
        'sidebar'
    ]),
}
```
 
