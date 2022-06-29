---
title: 使用GitHub Actions 自动部署hexo
date: 2022-06-29 10:08:14
categories:
- hexo
tags: 
- GitHub
---


# 使用GitHub Actions 自动部署hexo

##  1.创建hexo项目

- 配置node环境 [node官网]([Node.js 中文网 (nodejs.cn)](http://nodejs.cn/))

- 安装hexo项目 [安装文档]([文档 | Hexo](https://hexo.io/zh-cn/docs/))

## 2.创建github仓库

- 进入[GitHub]([GitHub](https://github.com/)) 注册并登录
- 点击new repository 创建一个新的仓库 

![image-20220629220231027](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629220231027.png)

![image-20220629220712032](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629220712032.png)

- 进入hexo创建好的博客目录下 注意是根目录
- 将全部代码git到刚刚创建好的blog仓库中

![image-20220629220920408](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629220920408.png)

## 3.创建自动部署文件

- 在.github文件夹下面创建 workflows文件夹 里面创建reploy.yml文件

```
name: Deploy Hexo Blog # Actions 显示的名字，随意设置

# 触发条件
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: checkout # 拉取当前执行 Actions 仓库的指定分支
      uses: actions/checkout@v2
      with:
        ref: main

    - name: 安装 Node环境 # 安装 Node 环境
      uses: actions/setup-node@v2
      with:
        node-version: "16.x"

    - name: 安装Hexo
      run: |
        npm install hexo-cli -g
        npm install

    - name: 设置密钥
      env:
        HEXO_DEPLOY_PRIVATE_KEY: ${{ secrets.HEXO_DEPLOY_PRIVATE_KEY }}  
      run: |
        mkdir -p ~/.ssh/
        echo "$HEXO_DEPLOY_PRIVATE_KEY" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan github.com >> ~/.ssh/known_hosts

    - name: 设置Git信息
      run: |
        git config --global user.name '创建密钥使用的user.name'
        git config --global user.email '创建密钥使用的email'

    - name: 自动部署hexo
      run: |
        hexo clean
        hexo generate
        hexo deploy

```

![image-20220629221447283](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629221447283.png)

## 4. 配置仓库私钥和公钥

### 1. 首先使用git命令配置用户名和邮箱避免出现不必要的错误

```
$ git config --global user.name "你的github用户名"
$ git config --global user.email "注册用的email"
```

### 2.生成 ssh 密钥

```
ssh-keygen -t rsa -C "xxxxx@xxxxx.com邮箱账号"  
```

- 进入C:\Users\你的用户名 \ .ssh\

![image-20220629222744789](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629222744789.png)

- id_rsa 为私钥
- id_rsa.pug 为公钥

### 3.进入当前blog仓库的setting中

![image-20220629222134809](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629222134809.png)

### 4.点击new 	repository secret 

![image-20220629222420699](https://raw.githubusercontent.com/xyh-study/blog-pic-repo/main/image-20220629222420699.png)

### 5.这个私钥的名称一定和配置文件中的${{ secrets.HEXO_DEPLOY_PRIVATE_KEY }} 对应上

