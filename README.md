# github
搭建vue开发环境
1.安装node
(1) 下载
(2) 新增 node_global 和 node_cache 文件夹，打开cmd(需要在管理员权限打开cmd，否则添加命令无效)，输入以下命令
  npm config set prefix "D:\node\node_global"
  npm config set cache "D:\node\node_cache"
(3) 配置环境变量
  用户变量 path ：C:\Program Files\nodejs\node_global(如果为node_modules导致cnpm无法访问)
  系统变量 NODE_PATH ：C:\Program Files\nodejs\node_global\node_modules
  path：加入NODE_PATH
(4)检验安装是否成功：node -v
  若出现"'node' 不是内部或外部命令,也不是可运行的程序 或批处理文件"错误，说明环境变量没有配置成功
2.安装cnpm(同样需要在管理员权限下安装)
(1) npm install -g cnpm --registry=https://registry.npm.taobao.org
(2) cnpm -v 
3.安装vue-cli脚手架
(1) 安装vue
  cnpm install vue -g
  vue -V
(2)安装vue-cli
  cnpm install @vue/cli -g
问题1：Install fail! RunScriptError: post install error, please remove node_modules before retry!
解决：
4.创建新项目
  如果你仍然需要使用旧版本的 vue init 功能，需要执行npm install -g @vue/cli-init
  npm install -g @vue/cli-init
  vue init webpack my-project
5.安装工程依赖模块
  npm install
6.启动项目
  npm run dev
