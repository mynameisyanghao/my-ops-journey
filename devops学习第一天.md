### ***\*动手练习\****

\1. 完成 3 台虚拟机的创建，记录每台的 IP 地址，从管理机能免密登录到另外两台。

我使用的是windows环境使用的Vmware软件创建虚拟机，单台创建主机太麻烦，于是我使用ai写了一个脚本脚本可以用来批量创建虚拟机。

脚本如下：

@echo off

setlocal enabledelayedexpansion

chcp 65001 >nul

 

:: ===== 路径配置 =====

set "VMRUN=E:\vm17\vmrun.exe"

set "TEMPLATE=E:\unbuntuk8s\mb\k8s.vmx"

set "VM_ROOT=E:\unbuntuk8s\cluster"

set "GUEST_SCRIPT=%~dp0guest_setup.sh"

set "LOG=%~dp0run.log"

 

:: ===== 虚拟机登录用户和密码（Ubuntu默认无root，用yh用户+sudo提权）=====

set VM_ROOT_USER=yh

set VM_ROOT_PWD=passme

 

:: ===== 清空日志 =====

echo. > "%LOG%"

 

:: ===== 节点列表：虚拟机名 + IP最后一段 =====

for %%n in ("k8s-master 130" "k8s-node1 131" "k8s-node2 132") do (

  for /f "tokens=1,2" %%a in (%%n) do (

​    set "VM_NAME=%%a"

​    set "IP_LAST=%%b"

​    set "VM_DIR=!VM_ROOT!\%%a"

​    set "VMX=!VM_DIR!\%%a.vmx"

 

​    :: 目标目录不存在则创建（vmrun clone 要求目标目录已存在）

​    if not exist "!VM_DIR!" mkdir "!VM_DIR!"

 

​    echo ========================================== >> "%LOG%"

​    echo [开始] 克隆 !VM_NAME!  192.168.2.!IP_LAST! >> "%LOG%"

 

​    "%VMRUN%" -T ws clone "%TEMPLATE%" "!VMX!" full >> "%LOG%" 2>&1

​    if !errorlevel! neq 0 (

​      echo [错误] 克隆失败 !VM_NAME! - 检查模板路径、模板是否已关机、锁文件是否清理 >> "%LOG%"

​    ) else (

​      echo [开始] 启动 !VM_NAME! >> "%LOG%"

​      "%VMRUN%" -T ws start "!VMX!" >> "%LOG%" 2>&1

 

​      echo 等待 40 秒等待 open-vm-tools 就绪 ...

​      timeout /t 40 /nobreak

 

​      echo [开始] 上传脚本到 !VM_NAME! >> "%LOG%"

​      "%VMRUN%" -T ws -gu !VM_ROOT_USER! -gp !VM_ROOT_PWD! copyFileFromHostToGuest "!VMX!" "%GUEST_SCRIPT%" /tmp/guest_setup.sh >> "%LOG%" 2>&1

 

​      echo [开始] 配置主机名和静态IP !VM_NAME! >> "%LOG%"

​      "%VMRUN%" -T ws -gu !VM_ROOT_USER! -gp !VM_ROOT_PWD! runScriptInGuest "!VMX!" /bin/bash "echo '!VM_ROOT_PWD!' | sudo -S bash /tmp/guest_setup.sh !VM_NAME! 192.168.2.!IP_LAST!" >> "%LOG%" 2>&1

​      if !errorlevel! neq 0 (

​        echo [错误] 配置失败 !VM_NAME! - 检查yh密码/sudo权限、open-vm-tools是否安装 >> "%LOG%"

​      ) else (

​        echo [成功] !VM_NAME! 配置完成 >> "%LOG%"

​      )

​    )

  )

)

 

echo ========================================== >> "%LOG%"

echo [END] 全部执行完毕，结果见 run.log >> "%LOG%"

echo 全部执行完毕，请打开 run.log 查看结果

pause

配套sh脚本用于修改虚拟机信息内容如下：

\#!/bin/bash

\# 用法: bash /tmp/guest_setup.sh <主机名> <IP地址>

HOST=$1

IP=$2

 

echo "===== 设置主机名: $HOST ====="

hostnamectl set-hostname "$HOST"

 

echo "===== 写入 netplan 静态IP: $IP/24 ====="

cat > /etc/netplan/01-static.yaml <<EOF

network:

 version: 2

 ethernets:

  ens33:

   dhcp4: no

   addresses:

​    \- $IP/24

   routes:

​    \- to: default

​     via: 192.168.2.1

   nameservers:

​    addresses: [114.114.114.114,223.5.5.5]

EOF

 

echo "===== 应用网络配置 ====="

netplan apply

 

echo "===== 重建SSH密钥 ====="

rm -f /etc/ssh/ssh_host_*

dpkg-reconfigure openssh-server || true

 

echo "===== 完成: $HOST -> $IP ====="

 

\2. 注册 GitHub 账号，创建第一个仓库 my-ops-journey，往里面提交一个 README.md，内容是你未来 9 个月的目标清单。（PS:每一次更新都需要重新提交commit）

\# GitHub 创建仓库 `myopsjourney` + 提交 `README.md`

两种方式：**网页直接操作（最简单，不用装git）**；**本地git命令行（运维常用，以后工作用这个）**。

 

\## 方式一：网页操作（新手推荐）

\1. 登录 github.com，右上角点 **+ 下拉 → New repository（新建仓库）**

 

 

 

\2. 填写表单：

\- Repository name：`my-ops-journey`（**必须一模一样**）

\- Description（可选）：`我的运维学习旅程`

\- 选 **Public（公开）**

\- ✅ **勾选 Add a README file**（自动生成README.md）

\- 点击绿色按钮 **Create repository**

 

\> 完成！仓库建好，并且已经自带 README.md 文件。

 

\3. 修改/提交README内容

\- 点开仓库里的 `README.md`

\- 右上角铅笔图标 ✏️（编辑），写入内容，示例：

\```markdown

\# myopsjourney

运维学习记录，包含Linux、K8s、虚拟化实验。

\```

\- 下面填写提交信息：`feat: add readme 初始化仓库`

\- 点 **Commit changes** 提交，网页端就完成推送了。

 

\---

 

\## 方式二：本地 Git 命令行（运维标准流程，Windows / Ubuntu都适用）

\> 前提：系统已经安装 git；需要在github生成 **Personal access token（token）**，密码位置填token，不再填账号密码。

 

\### ① 第一次git全局配置（只执行一次）

\```bash

git config --global user.name "你的github用户名"

git config --global user.email "你的github注册邮箱"

\```

查看配置：

\```bash

git config --global --list

\```

 

\### ② github网页新建仓库（不要勾选 Add README！！留空）

\1. New repository，名字：`myopsjourney`，Public，**不要勾选Add README**，直接Create repository。

\2. 复制仓库地址，HTTPS格式：`https://github.com/你的用户名/my-ops-journey.git`

 

\### ③ 本地操作

\```bash

\# 新建本地文件夹

mkdir my-ops-journey

cd my-ops-journey

 

\# 初始化git仓库

git init

 

\# 创建README.md，写入内容

cat > README.md <<EOF

\# myopsjourney

运维学习记录，包含Linux、K8s、虚拟化实验。

EOF

 

\# 添加到暂存区

git add README.md

 

\# 提交本地commit

git commit -m "feat: init readme"

 

 

\# 关联远程github仓库（替换成你自己的仓库地址）

git remote add origin https://github.com/mynameisyanghao/my-ops-journey.git

 

\# 推送前需要确认节点对不对是不是main分支，一般github新建的项目默认main分支，如果不是就修改一下

 git branch -M main

 

\# 推送到github main分支

git push -u origin main

\```

\# 如果报错

Administrator@HX-C512 MINGW64 ~/Desktop (main)

$ git push -u origin main

To https://github.com/mynameisyanghao/my-ops-journey.git

 ! [rejected]     main -> main (fetch first)

error: failed to push some refs to 'https://github.com/mynameisyanghao/my-ops-journey.git'

hint: Updates were rejected because the remote contains work that you do not

hint: have locally. This is usually caused by another repository pushing to

hint: the same ref. If you want to integrate the remote changes, use

hint: 'git pull' before pushing again.

hint: See the 'Note about fast-forwards' in 'git push --help' for details.

 

\# 拉取远程代码合并，再推送

\# 拉取远程main分支，允许合并两个无关的仓库历史

git pull origin main --allow-unrelated-histories

git push -u origin main

 

 

\> 弹出账号密码框：用户名填github账号，**密码处粘贴你的Personal access token**。

 

\### 常见坑

\1. 如果网页端建仓库时勾选了README，本地push会报错：`fatal: refusing to merge unrelated histories`，解决：

\```bash

git pull origin main --allow-unrelated-histories

git push origin main

\```

\2. Windows要先安装Git for Windows；Ubuntu执行 `apt install git -y`。

\3. 新版github不再支持账号密码登录，必须用token。

 

\## 验证是否成功

浏览器访问：`https://github.com/你的用户名/myopsjourney`，页面能看到README.md渲染的文字，代表提交完成。

 

\### 给你一份适合这个仓库的README模板，直接复制用

\```markdown

\# myopsjourney

\> 我的运维学习笔记，记录Linux、VMware虚拟化、K8s实验踩坑。

 

\## 学习内容

\1. VMware Workstation批量创建虚拟机模板

\2. Ubuntu静态IP配置

\3. Git & GitHub使用

\4. Kubernetes集群搭建

 

\## 环境

\- VMware Workstation 17

\- Ubuntu 22.04

 

在 VS Code 里通过 Remote-SSH 连接管理机，截图留档——以后你的所有代码都在这里写。（推荐使用xshell更合适）