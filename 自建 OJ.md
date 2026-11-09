# 自建 OJ 分阶段学习路线（学习任务 + 配套资料绑定，一步一步来）
目标：可自定义外观 + 自定义评测 + 公网域名访问；优先路线：Hydro OJ 二次开发（成本最低，能同时学前段、判题、部署）
环境统一：Ubuntu 22.04（虚拟机先练，再上云服务器）

## 前置说明
- 阶段顺序不能跳，每阶段完成后再进入下一阶段
- 每个小节：【学习任务】→【配套资料】→【验收标准】（做完验收，再往下）

## 阶段 0：环境准备 (1~2 天)
目标：掌握 Linux、虚拟机基础，搭建本地实验环境

### 0.1 学习任务
1. 安装 VMware / VirtualBox，新建 Ubuntu 22.04 虚拟机
2. 学会基础命令：`cd` `ls` `mkdir` `cp` `rm` `apt` `systemd`
3. 学会使用 `sudo`，了解用户权限
4. 了解 Docker 基础概念：容器、镜像

配套资料：
5. Linux 入门：Linux 鸟哥私房菜（基础篇，看前 5 章即可）
6. Docker 官方入门文档：[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

验收标准：
- 虚拟机正常开机；能在 Ubuntu 里安装 docker；成功跑通 `docker run hello-world`

### 0.2 学习任务
学习网络基础：Nginx、域名、反向代理、HTTPS

配套资料：
1. Nginx 官方入门文档：[https://nginx.org/en/docs/beginners_guide.html](https://nginx.org/en/docs/beginners_guide.html)
2. Let’s Encrypt 免费证书教程（certbot）

验收标准：
- 在虚拟机用 Nginx 部署一个静态网页，访问 IP 能打开；知道反向代理是什么

## 阶段 1：本地部署 Hydro OJ（核心第一步，2~3 天）
目标：本地虚拟机跑完整 OJ，注册账号、提交 AC 代码，熟悉整套流程

### 1.1 学习任务
1. 阅读 Hydro 官方文档，了解架构：网站服务 + 评测机 judger
2. 使用官方一键脚本在 Ubuntu 虚拟机安装 Hydro

配套资料：
Hydro 官方文档：[https://hydro.ac/docs](https://hydro.ac/docs)

验收标准：
虚拟机浏览器打开 `http://本机IP:8080`，进入 OJ，新建题目，提交一段 C 语言代码，评测成功（AC）

### 1.2 学习任务
熟悉 Hydro 后台功能：创建题目、SPJ、比赛、用户权限

配套资料：Hydro 文档【题目管理、自定义判题 SPJ】章节

验收标准：
创建一道答案不唯一的题目，编写 SPJ，提交代码可以正常判对 / 判错

> ? 到这里：你已经拥有一个可以跑题目的 OJ。接下来开始【自定义外观】

## 阶段 2：修改 OJ 外观（自定义页面，3 ~ 7 天）
Hydro 前端是独立 Vue 项目，可以完全自定义页面、logo、首页、配色

### 2.1 学习任务
学习前端基础：Vue3 + Vite + CSS，看懂 Hydro 前端源码结构

配套资料：
1. Vue3 官方文档：[https://vuejs.org/guide/introduction.html](https://vuejs.org/guide/introduction.html)
2. Hydro 前端仓库：[https://github.com/hydro-dev/Hydro/tree/master/frontend](https://github.com/hydro-dev/Hydro/tree/master/frontend)

验收标准：
把 Hydro 首页 logo 替换成自己图片，修改首页文字，重新打包前端并部署，网页生效

### 2.2 学习任务
深度定制：修改页面布局、新增页面、调整主题样式

配套资料：Hydro 文档【自定义前端】章节

验收标准：
成功修改页面配色，新增一个自定义介绍页面，OJ 内可以正常跳转

## 阶段 3：深入评测系统（自定义评测逻辑，OJ 核心，5 ~ 10 天）
本阶段学习判题原理，理解沙盒，自己写自定义校验逻辑

### 3.1 学习任务
理解 OJ 完整评测流程：提交 -> 消息队列 -> 评测机拉取任务 -> 沙盒运行代码 -> 结果回传

配套资料：
1. 洛谷文章：一小时从零建立你自己的 Online Judge
[https://www.luogu.com.cn/article/025rwkjq](https://www.luogu.com.cn/article/025rwkjq)
2. 清华 OJ 课程实验文档（沙盒原理）：
[https://lab.cs.tsinghua.edu.cn/rust/2022/projects/oj/](https://lab.cs.tsinghua.edu.cn/rust/2022/projects/oj/)

验收标准：
口述清楚：TLE、MLE、RE、WA 分别是怎么被评测机检测出来

### 3.2 学习任务
学习 Linux 沙盒底层：cgroup（限制 CPU 内存）、seccomp（系统调用拦截）、namespaces 隔离

配套资料：
1. Linux man 手册 cgroup、seccomp
2. Judge0 文档（沙盒设计思路）：[https://judge0.com](https://judge0.com)

验收标准：
写一个简单 C 程序，使用 cgroup 限制内存，超出内存程序直接被杀掉（模拟 MLE）

### 3.3 学习任务
Hydro 自定义 SPJ、交互题评测开发

配套资料：Hydro 文档【Special Judge / 交互题】

验收标准：
手写 SPJ 程序，实现自定义判分逻辑（例如：答案包含指定字符串就算 AC）

## 阶段 4：公网上线！别人输入网址访问（2 ~ 3 天）
### 4.1 学习任务
1. 云服务器部署 Hydro（推荐轻量云，或者 Oracle 免费云做实验）
2. 域名解析：域名 A 记录指向服务器公网 IP
3. Nginx 反向代理，配置 HTTPS（certbot）

配套资料：
4. Hydro 文档【生产环境部署】
5. Certbot 官方文档：[https://certbot.eff.org/](https://certbot.eff.org/)

验收标准：
浏览器输入你的域名，直接打开 OJ，HTTPS 正常，其他人网络可以访问，提交代码正常判题

### 4.2 学习任务
安全加固（对外网站必须做）：评测机与 web 服务分离，防火墙配置，限制访问

配套资料：Hydro 文档【安全建议】

验收标准：
服务器防火墙只开放必要端口；评测机无法访问内网数据库

## 阶段 5（进阶可选：自研方向，适合学完 Hydro 之后）
如果你想从零手写 OJ，不是二次开发，在完成前面全部阶段再碰

### 5.1 学习任务
设计 OJ 架构：前端 Vue + 后端（Golang/Node） + Redis 任务队列 + 沙盒

配套资料：
1. System Design: Online Judge [https://vibeengines.com/systemdesign/online-judge-system-design](https://vibeengines.com/systemdesign/online-judge-system-design)
2. Judge0 源码：[https://github.com/judge0/judge0](https://github.com/judge0/judge0)（参考独立评测沙盒）

验收标准：
画出 OJ 架构图，画出：提交队列、worker 评测节点、沙盒模块

### 5.2 学习任务
手写极简版评测沙盒

配套资料：清华 OJ 实验文档

验收标准：
写程序能够编译运行用户代码，限制时间内存，捕获退出状态

# ? 汇总资料清单（方便一次性收藏）
1. Hydro OJ 官方文档：[https://hydro.ac/docs](https://hydro.ac/docs)
2. Hydro Github：[https://github.com/hydro-dev/Hydro](https://github.com/hydro-dev/Hydro)
3. Vue3 官方文档：[https://vuejs.org/guide/introduction.html](https://vuejs.org/guide/introduction.html)
4. 洛谷 OJ 搭建入门：[https://www.luogu.com.cn/article/025rwkjq](https://www.luogu.com.cn/article/025rwkjq)
5. 清华 OJ 沙盒实验：[https://lab.cs.tsinghua.edu.cn/rust/2022/projects/oj/](https://lab.cs.tsinghua.edu.cn/rust/2022/projects/oj/)
6. Judge0 官网 & 源码：[https://judge0.com](https://judge0.com) / [https://github.com/judge0/judge0](https://github.com/judge0/judge0)
7. Docker 入门：[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)
8. Nginx 入门：[https://nginx.org/en/docs/beginners_guide.html](https://nginx.org/en/docs/beginners_guide.html)
9. Certbot HTTPS：[https://certbot.eff.org/](https://certbot.eff.org/)

# 可选视频资源（辅助理解）
1. NOI 评测系统原理讲解视频（B 站）：搜索关键词：`NOI 评测系统 沙盒`
2. Hydro OJ 部署教程（B 站）：搜索关键词：`Hydro OJ 搭建`

需要我把这份md导出成纯文本，或者拆成单独的任务清单（可复制到Obsidian）吗？