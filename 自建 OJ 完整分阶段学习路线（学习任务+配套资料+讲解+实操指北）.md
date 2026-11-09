# 自建OJ完整分阶段学习路线

> 
> 适配：Linux + Docker + Hydro/QOJ/ClimberOJ（你之前关注的OJ系统栈），面向信奥CSP/NOIP训练场景；目标：从零基础搭出可提交C++代码、判题、排行榜、题目管理的在线判题平台；每阶段【学习任务 + 配套资料 + 讲解 + 实操指北】，命令一次性整块给到。
> 环境基线：Ubuntu 20.04/22.04（虚拟机桥接，当前你已经跑通Docker+Nginx）

## 前置总准备（阶段0，先一次性做完）

**学习任务**：确认基础环境，安装依赖、Docker、Docker Compose，固定虚拟机IP（桥接）
**配套资料**

- Docker官方文档：[https://docs.docker.com/compose/](https://docs.docker.com/compose/)
- Ubuntu网络配置：netplan文档
- 
**讲解**
自建OJ核心依赖：Docker隔离判题沙箱（防止恶意代码攻击宿主机）；OJ前端、后端、数据库、判题机全部容器化部署，方便迁移。桥接模式保证局域网其他电脑可以访问OJ站点。

**实操指北（整块复制执行）**

```
# 更新系统
sudo apt update && sudo apt upgrade -y
# 安装基础工具
sudo apt install -y git curl wget vim net-tools iputils-ping
# 安装docker+docker compose
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
sudo apt install -y docker-compose-plugin
# 将当前用户加入docker组，免sudo操作docker
sudo usermod -aG docker $USER
# 【重要】执行完上面命令需要重新登录终端（退出ssh/重新打开窗口），免sudo生效
# 验证docker
docker info
docker compose version
```

> 
> 可选：netplan配置静态IP（桥接环境，固定192.168.1.120不变，避免每次重启IP变）

```
sudo vim /etc/netplan/00-installer-config.yaml
```

netplan示例yaml（注意yaml缩进严格，空格不能用Tab）

```
network:
  ethernets:
    ens33:
      addresses: [192.168.1.120/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [223.5.5.5,8.8.8.8]
  version: 2
```

```
sudo netplan apply
ip a
```

---

## 阶段1：选型调研 + 预演容器基础（1~2天）

**学习任务**：对比主流开源OJ，吃透Docker容器、卷、网络概念；跑通容器持久化、端口映射（刚才nginx就是基础）
**配套资料**

1. HydroOJ文档 [https://hydro.js.org/](https://hydro.js.org/)
2. QOJ [https://github.com/QingdaoU/OnlineJudge](https://github.com/QingdaoU/OnlineJudge)
3. ClimberOJ（你自研OJ）> 
> 选型建议：
> 
> 
> - 快速部署、拿来即用，适合信奥训练：HydroOJ（推荐首选，文档完善，判题沙箱成熟）
> - 功能强大，可深度二次开发：QOJ
> - 自研学习、自定义逻辑：ClimberOJ（需要自己写判题核心，难度最高）
> **讲解**
> 容器：OJ的前端网页、后端API、MySQL数据库、判题机，全部独立容器；volume数据卷用来持久化题目、用户数据，容器删除数据不丢失。
> **实操指北（整块复制）**

```
# 练习：容器卷挂载（把本地目录挂载进容器，对应OJ题目文件持久化）
mkdir -p ~/oj_test/html
echo "<h1>OJ测试页面</h1>" > ~/oj_test/html/index.html
docker run -d -p 8081:80 -v ~/oj_test/html:/usr/share/nginx/html --name oj-nginx-test nginx
# 访问 [http://192.168.1.120:8081](http://192.168.1.120:8081) 验证
# 清理测试容器
docker stop oj-nginx-test
docker rm oj-nginx-test
```

**阶段验收**：理解 `-v`挂载、`-p`端口映射；能看懂docker compose yaml；选定OJ系统：Hydro推荐新手自建OJ。

---

## 阶段2：部署HydroOJ（容器化一键部署，核心阶段，2天）

**学习任务**：使用docker compose拉起完整HydroOJ服务；初始化管理员账号；网页登录OJ后台
**配套资料**：Hydro官方docker部署文档 [https://hydro.js.org/docs/install/docker](https://hydro.js.org/docs/install/docker)
**讲解**
Hydro包含：前端、后端、mongodb数据库、judger判题机。docker compose一次性编排所有服务；judger是核心：接收提交代码，在隔离沙箱编译运行C++，返回时间/内存、结果(AC/TLE/MLE/RE/WA)
**实操指北（整块复制执行）**

```
# 创建工作目录
mkdir -p ~/hydrooj && cd ~/hydrooj
# 下载docker-compose.yml
curl -O https://hydro.js.org/docker-compose.yml
# 启动全部服务
docker compose up -d
# 查看容器状态
docker compose ps
# 查看初始化管理员密码（容器日志）
docker compose logs hydro | grep "admin"
```

访问地址：`[http://192.168.1.120:8888](http://192.168.1.120:8888)`

1. 使用日志输出的管理员账号密码登录后台
2. 后台基础设置：网站名称、域名、提交限制、C++编译配置

> 
> 验证：在网页手动新建一道A+B题，提交C++代码，测试判题是否返回AC。

**阶段验收**：网页正常打开；管理员登录；A+B样题提交可正常判题。

---

## 阶段3：OJ基础运维 + 导入信奥题库（2~3天）

**学习任务**：OJ数据备份、恢复；批量导入CSP/NOIP真题；学会管理题目、测试数据、时限内存；理解判题沙箱原理
**配套资料**

- Hydro题库导入插件：[https://hydro.js.org/docs/plugins](https://hydro.js.org/docs/plugins)
- CSP历年测试数据包
**讲解**
判题沙箱：限制CPU时间、内存，禁止系统调用，防止恶意代码（fork炸弹、文件删除）破坏宿主机。OJ题目由：描述+输入样例+输出样例+测试数据+时间/内存限制组成。
**实操指北**

```
# 备份Hydro数据库（数据卷备份）
cd ~/hydrooj
docker compose exec hydro bash -c "mongodump --archive" > hydro-backup.archive
# 恢复备份命令
# docker compose exec -i hydro bash -c "mongorestore --archive" < hydro-backup.archive

# 查看OJ实时日志，排查提交报错
docker compose logs -f hydro
# 单独查看判题机日志
docker compose logs -f judger
```

操作网页后台：

1. 安装`hdisc`插件，批量导入题库
2. 创建分类：CSP-J、CSP-S、NOIP
3. 上传测试数据，设置时间限制、内存限制

**阶段验收**：成功批量导入真题；提交代码WA/TLE/RE能正常识别；会备份OJ数据。

---

## 阶段4：判题原理深入，扩展C++编译配置（2~3天）

**学习任务**：理解C++编译参数、沙箱资源限制；自定义编译器版本；调试判题出错；学习评测状态（AC WA TLE MLE RE CE）含义
**配套资料**

- GCC手册
- Hydro Judger源码 [https://github.com/hydrooj/judger](https://github.com/hydrooj/judger)
**讲解**
CE=编译错误；RE=运行时错误；TLE超时；MLE内存超限。提交代码流程：前端上传 → 后端存入数据库 → 判题机拉取任务 → 沙箱内g++编译 → 运行，对比输出和标准答案。
**实操指北**
网页后台操作：

1. 修改编译器配置：g++ -std=c++17 -O2（信奥标准编译参数）
2. 测试：故意写数组越界代码，验证RE；死循环验证TLE；超大数组验证MLE。

**阶段验收**：能人为构造各类评测结果，看懂判题日志定位代码错误。

---

## 阶段5：OJ进阶功能（1~2天）

**学习任务**：开启排行榜、提交统计、用户分组、比赛功能；配置Nginx反向代理，把8888端口映射到80（直接http访问不带端口号）
**配套资料**：Hydro插件市场文档
**讲解**
反向代理：Nginx接收80端口请求，转发给hydro的8888端口；适合对外访问，域名部署。
**实操指北（Nginx反向代理，整块复制）**

```
# 安装宿主机nginx
sudo apt install nginx -y
sudo vim /etc/nginx/sites-available/hydro
```

nginx配置文件内容

```
server {
    listen 80;
    server_name 192.168.1.120;
    location / {
        proxy_pass [http://127.0.0.1:8888](http://127.0.0.1:8888);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/hydro /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

现在直接访问 `[http://192.168.1.120](http://192.168.1.120)` 即可打开OJ，不用写:8888端口。

**阶段验收**：比赛创建、排行榜正常；80端口反向代理成功。

---

## 阶段6：二次开发 / 自研OJ ClimberOJ（长期，按需选择）

> 
> 如果你不想用现成Hydro，想写自己的ClimberOJ（你之前自研项目）
> **学习任务**：拆分OJ模块：前端页面、后端API、判题核心、沙箱、数据库；独立开发最小可用版本
> **配套资料**

1. libseccomp（系统调用过滤沙箱）
2. judge-server 开源判题机参考
3. NodeJS/Python/Golang后端，前端Vue
**讲解**
最小OJ模块：
4. 用户模块：注册登录
5. 题目管理模块
6. 提交模块（接收代码存入数据库）
7. 判题核心：调度任务，沙箱编译运行代码，对比输出
8. 结果回写前端展示
**实操指北**
先写最小原型：本地程序接收C++代码，调用g++编译，沙箱运行，对比输出。再拆微服务，容器化部署。

**阶段验收**：自研最小OJ，支持A+B提交判题。

## 阶段7：性能调优 + 安全加固（运维收尾）

**学习任务**：限制容器资源、安全加固、OJ压力测试，防止大量提交压垮服务器
**配套资料**：docker资源限制文档
**讲解**
给判题容器设置CPU/内存上限；关闭不必要端口；禁止宿主机权限泄露；防止恶意提交攻击。
**实操指北**
修改docker-compose.yml，给judger增加资源限制

```
services:
  judger:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

```
# 重启服务生效
docker compose up -d
```

# 学习路线选择建议

1. **快速搭建用于信奥训练（推荐）**：阶段0 → 阶段2 → 阶段3 → 阶段4 → 阶段5，用HydroOJ，1周左右可以完整跑起来用于CSP训练。
2. **自研ClimberOJ，学习底层判题原理**：全部7阶段，重点阶段6，先吃透Hydro/QOJ源码再动手写自己的。