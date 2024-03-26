# codecraft2024-docker-env
2024华为软件精英挑战赛 编译运行环境

## 环境说明

满足[2024年华为软件精英挑战赛【环境说明】](https://competition.huaweicloud.com/advance/1000042021/circumstance)

## 构建镜像

```shell
docker build -t judge -f docker/Dockerfile .
```

> 时间会很久

## 使用

先进入到你要评测的代码的项目目录中，然后执行：

```shell
docker run -it --rm --cpus 2 --memory 4g -v $(pwd):/home/judge/codecraft2024/ judge
```

在这个环境中跑你想测试的代码，和线上判题平台环境一致。