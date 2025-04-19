## 本地启动

Ory Kratos 用户认证
https://cloud.tencent.com/developer/article/1861413

目前在windows上就可以启动了

首先下载依赖
```bash
pnpm install
```

启动api D:\workspace\daily-api
1. 先启动数据库，可以使用docker-compose中的postgres和redis
```bash
docker run -d --name postgres01 -p 5432:5432 -e POSTGRES_PASSWORD=123456 -v /home/xu/workspace/pgdata:/var/lib/postgresql/data postgres:17-alpine

docker run -d --name redis01 -p 6379:6379 redis:latest

```
2. 修改.env文件
添加数据库配置
```bash
# 数据库相关配置
# postgres数据库连接配置
TYPEORM_HOST=127.0.0.1
TYPEORM_USERNAME=postgres
TYPEORM_PASSWORD=123456
TYPEORM_DATABASE=api
TYPEORM_PORT=5432
```
迁移数据库
```bash
pnpm run db:migrate:latest
# 运行后报错，连接数据库失败
# 还需要在 src/data-source.ts 中加载.env 中的配置
const dotenv = require('dotenv');
dotenv.config({ path: `.env` });
然后再运行就可以成功了
```


```bash
JWT_PUBLIC_KEY_PATH=../daily-apps/.cert/public.pem
JWT_PRIVATE_KEY_PATH=../daily-apps/.cert/key.pem
```

数据库配置
```bash
# 创建数据表
# 默认情况下test模式下会使用api_test数据库
set NODE_ENV=test

pnpm run db:migrate:latest
# 导入一些初始数据
pnpm run db:seed:import
```
## 以test模式启动
```bash
set NODE_ENV=test
pnpm run db:migrate:latest
pnpm run db:seed:import
# 这个不整会报错，但是不会影响启动
export GROWTHBOOK_CLIENT_KEY=your_growthbook_client_key
# 从daily-apps的docker-compose中看到他们有添加一个MOCK_USER_ID环境变量。
# 所以我们也在.env中添加 MOCK_USER_ID=testuser
pnpm run dev
```

## 以development模式启动
```bash
set NODE_ENV=development
pnpm run db:migrate:reset
pnpm run db:seed:import
# 这个不整会报错，但是不会影响启动
export GROWTHBOOK_CLIENT_KEY=your_growthbook_client_key
# 从daily-apps的docker-compose中看到他们有添加一个MOCK_USER_ID环境变量。
# 所以我们也在.env中添加 MOCK_USER_ID=testuser
pnpm run dev
```
## 以production模式启动
3. 运行`pnpm run dev`启动api
```bash
export NODE_ENV=production
# windows下配置环境变量

export NODE_ENV=test
export GROWTHBOOK_CLIENT_KEY=your_growthbook_client_key
pnpm run dev
```

启动报错1
```bash
stack_trace: "Error: GEOIP_PATH not set\n    at initGeoReader (D:\workspace\daily-api\src\common\geo.ts:280:13)\n    at run (D:\workspace\daily-api\bin\cli.ts:20:26)\n    at processTicksAndRejections (node:internal/process/task_queues:105:5)"
    logging.googleapis.com/insertId: "..........DNgR_V_atEtdUonoOq16fq"
    message: "Error loading GeoIP2 database"
```
解决办法：
`await initGeoReader();` 这一行注释掉

启动报错2
连接数据库失败
```bash
在跟目录下ormconfig.ts 加载环境变量
const dotenv = require('dotenv');
dotenv.config({ path: `.env` });
```

启动报错3
```bash
Error: ENOENT: no such file or directory, open 'D:\workspace\daily-api\src\common\geo.ts'
```
解决办法：
在 .env 中把下面的配置注释掉
```bash
# JWT_PUBLIC_KEY_PATH=../apps/.cert/public.pem
# JWT_PRIVATE_KEY_PATH=../apps/.cert/key.pem
```

发现注释掉也不行
```log
TypeError: The "path" argument must be of type string or an instance of Buffer or URL. Received undefined
```

尝试着自己生成证书
```bash
openssl genpkey -algorithm RSA -out ./.cert/key.pem -pkeyopt rsa_keygen_bits:2048
 openssl rsa -in .cert/key.pem -pubout -out .cert/public.pem
```
然后修改.env 中证书的路劲为新生成的路径
```bash
JWT_PUBLIC_KEY_PATH=.cert/public.pem
JWT_PRIVATE_KEY_PATH=.cert/key.pem
```



# 启动前台 D:\workspace\daily-apps
1. 修改.env文件，添加api地址
2.  修改环境变量 
NEXT_PUBLIC_API_URL=http://127.0.0.1:5000 
NEXT_PUBLIC_SUBS_URL=wss://192.168.0.111/graphql

3. 运行下面命令启动前台
```bash
cd packages\webapp
pnpm run dev:notls
```
## 报错1：
启动后，请求api的boot接口不通

api侧的csrf需要配置下，暂时关闭

关闭后，接口可以正常访问了，但是前台还是没有显示出来
## 报错2：
这是api侧的错误提示
```bash
[[object Object]] INFO (17680):
    severity: "INFO"
    reqId: "req-l"
    reason: "user not found"
    userId: "testuser"
    serviceContext: {
      "service": "api",
      "version": "latest"
    }
    logging.googleapis.com/insertId: ".........0vPtpqNgRCCCuu7ZOghGJ1T"
    message: "clearing authentication
```
这难道是web_app侧也需要启动生产模式吗？
清理web端 cookie就可以解决


## supabase改动
1. 认证流程最好还是在服务端通过api来完成
2. 在src/routes/auth 添加supabase认证相关的接口
3. 具体请求和supabase的交互在src/supabase.ts中完成
