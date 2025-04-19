# 本地对此项目的修改
## 2021-06-13
1. cors配置给移除了
```bash
 app.register(cors, {
```
在这里直接return ture了
2. 部署后打不开登录的页面
需要将api以test模式启动即可。具体原因未知