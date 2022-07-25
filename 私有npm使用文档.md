# 私有 npm 库使用文档
## 地址

  - `https://npm.xmov.ai/`

  - 可以网页访问，账号密码：xmov/Xmov0115

## 使用

> 注意：公司统一使用 `yarn` 作为包管理工具

### 设置 `registry`

```shell
$ yarn config set registry https://npm.xmov.ai
```

### 初始化你的包

```shell
$ yarn init
```

> 注意：对应在 git 上新建对应的仓库

### 包命名、变量命名

（待讨论）

### 登录 npm

```shell
$ yarn login
```

`username` 为上述 `xmov`，`email` 填写自己的企业邮箱

### 发布你的包

```shell
$ yarn publish
```

密码为上述 `Xmov0115`，每次发布需要更新 `version` 字段，参考 `npm version major/minor/patch` 命令

### 删除你的包

```shell
$ yarn unpublish ${package-name}@{version} --force
```


## 管理 registry 的工具 - [nrm](https://github.com/Pana/nrm)


## 一些说明

- 每次发布包时需要修改 `package.json` 里面的版本号，发包时版本号不能与库里已经拥有的包的版本号相同。版本号一般三位，建议：第一位为大版本，第二位为迭代版本，第三位为bug修复和优化

- 切换源为公司的私有库后，获取包时会经过私服查找是否存在，存在的话会获取私有库里的包，如果私有库里不存在会去 taobao 镜像里找

- `nrm` 的作用仅仅是帮助我们快捷的切换源地址，不用完全没有影响，不过你如果有经常切换多个源的需求的话可以用用

- 推送包时需要注意，有些配置可能会影响你推送包，比如 `package.json`` 里的 `private`` 参数。还有些情况推送成功，但是拉取的时候不能正常 install。这时候可以获取 npm 查看原因

- 其他问题钉钉联系 `@王凯`
