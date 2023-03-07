#### 一、私有源优点

```
1. 安全性 (部署在内网，资产安全性高)
2. 复用性，提高开发效率，版本管理
3. 提升下载速度
4. 技术资产积累
```



#### 二、私有源有哪些

```
1. Npm 付费服务
2. Sinopia (不再维护)
3. Verdaccio (基于Sinopia的分支)
4. cnpm
```



#### 三、调研对比

```
verdaccio: 轻量级，安装使用简单，但不能定时同步官方源

cnpm: 可以定时同步官方源，但需要依赖外部服务mysql，淘宝源使用的就是cnpm
```

由于公司没有定时同步官方源的需求，最终选择了verdaccio，官方文档：[verdaccio](https://github.com/verdaccio/verdaccio)、
[cnpm](https://github.com/cnpm/cnpmjs.org)



#### 四、安装方法

1. 使用 npm/yarn 全局安装

```shell
// 安装
$ npm install -g verdaccio
$ yarn global add verdaccio

// 启动
$ verdaccio
```

> verdaccio 命令启动的是前台进程，使用 pm2 守护 verdaccio 进程

```shell
// 安装 pm2
$ npm install -g pm2

// 使用 pm2 启动 verdaccio
$ pm2 start verdaccio

// 查看 pm2 守护下的进程 verdaccio 的实时日志
$ pm2 show verdaccio
```



2. 使用docker镜像安装

```shell
V_PATH=/path/for/verdaccio; docker run -it --rm --name verdaccio \
  -p 4873:4873 \
  -v $V_PATH/conf:/verdaccio/conf \
  -v $V_PATH/storage:/verdaccio/storage \
  -v $V_PATH/plugins:/verdaccio/plugins \
  verdaccio/verdaccio
```



#### 五、配置文件解析

生产环境需要修改默认配置文件，以下对公司私有源配置文件进行详细解析

```yaml
# 包含所有包的目录路径,npm私服包的存放目录以及缓存地址
storage: /verdaccio/storage/data
# 包含plugins的目录路径,默认插件的文件位置
plugins: /verdaccio/plugins
# verdaccio的界面设置
web:
  enable: true # 可以在页面搜索
  title: npm # 标题
  gravatar: true # 支持 gravatar头像
# 权限
auth:
  # 密码验证方式htpasswd，加密后密码保存在文件里
  htpasswd:
    file: /verdaccio/storage/htpasswd
	# 禁止用户自行注册，只可以用公司账号密码登录验证
    max_users: -1
    algorithm: bcrypt
    rounds: 10
# 配置上游的npm服务器，主要用于请求的库不存在时可以去到上游服务器去获取，可以多配置下上游链路的链接
uplinks:
  npmjs:
    url: https://registry.npmmirror.com/  # 淘宝最新npm源
# 配置模块,access访问下载权限，pushlish包的发布权限，unpublish删除包权限
# "$all", "$anonymous", "$authenticated" 三个关键字:所有的,匿名的,验证过的
# 公司设置发包删包需要登录认证用户，所有人都可以访问下载
packages:
  '@*/*':
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
  '**':
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
# 服务超时时间，中间件等，此处是默认配置
server:
  keepAliveTimeout: 60
middlewares:
  audit:
    enabled: true
# 日志设置
logs:
  - {type: stdout, format: pretty, level: http}
# 路径前缀
url_prefix: '/'
# 安全设置，配置后可以在页面点击下载
security:
  api:
    legacy: false
    jwt:
      sign:
        expiresIn: 29d
```



#### 六，公司安装方法

公司使用k8s对容器进行统一编排管理

以下是部署yaml

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: npm

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: npm-verdaccio
  labels:
    app: verdaccio
  namespace: npm
data:
  config.yaml: |-
    storage: /verdaccio/storage/data
    plugins: /verdaccio/plugins
    web:
      enable: true
      title: npm
      gravatar: true
    auth:
      htpasswd:
        file: /verdaccio/storage/htpasswd
        max_users: -1
        algorithm: bcrypt
        rounds: 10
    uplinks:
      npmjs:
        url: https://registry.npmmirror.com/
    packages:
      '@*/*':
        access: $all
        publish: $authenticated
        unpublish: $authenticated
        proxy: npmjs
      '**':
        access: $all
        publish: $authenticated
        unpublish: $authenticated
        proxy: npmjs
    server:
      keepAliveTimeout: 60
    middlewares:
      audit:
        enabled: true
    logs:
      - {type: stdout, format: pretty, level: http}
    url_prefix: '/'
    security:
      api:
        legacy: false
        jwt:
          sign:
            expiresIn: 29d
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: verdaccio
  name: npm-verdaccio
  namespace: npm
spec:
  replicas: 1
  selector:
    matchLabels:
      app: verdaccio
  strategy:
    type: Recreate
    rollingUpdate: null
  template:
    metadata:
      labels:
        app: verdaccio
    spec:
      securityContext:
        runAsUser: 10001
        runAsGroup: 65533
      containers:
        - name: verdaccio
          image: "verdaccio/verdaccio:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 4873
              name: http
          livenessProbe:
            httpGet:
              path: /-/ping
              port: http
            initialDelaySeconds: 5
          readinessProbe:
            httpGet:
              path: /-/ping
              port: http
            initialDelaySeconds: 5
          resources:
            {}
          volumeMounts:
            - mountPath: /verdaccio/storage
              name: storage
              readOnly: false
            - mountPath: /verdaccio/conf/config.yaml
              name: config
              subPath: config.yaml
      nodeSelector:
        node: prod
      volumes:
      - name: config
        configMap:
          name: npm-verdaccio
      - name: storage
        hostPath:
          path: /data/nas/verdaccio
          type: DirectoryOrCreate
---
apiVersion: v1
kind: Service
metadata:
  name: npm-verdaccio
  labels:
    app: verdaccio
  namespace: npm
spec:
  ports:
    - port: 4873
      targetPort: http
      protocol: TCP
      name: npmport
      nodePort: 32767
  selector:
    app: verdaccio
  type: NodePort
```

