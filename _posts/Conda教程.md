---
title: Conda教程
date: 2024-07-19 03:37:03
tags:
    -"conda"
excerpt: "记录自己 conda 的使用过程。"
---


# 一、conda 指令

## 1.1 查看配置

```shell
conda config --show channels    # 查看镜像源
conda config --show-sources        # 查看配置文件内容
```

## 1.2 设置代理端口

```shell
# 添加代理地址端口
conda config --set proxy_servers.http http://127.0.0.1:10809
conda config --set proxy_servers.https http://127.0.0.1:10809

# 移除代理
conda config --remove-key proxy_servers
```

## 1.3 conda环境

```bash
# 创建（带名字）
conda create -n <conda_name> python=<版本号>
# 在指定文件路径下创建conda环境
conda create --yes --prefix /home/sky/桌面/pointTest/.conda python=3.11

# 激活conda环境
conda activate <conda_name>

# 回到base环境
conda deactivate

# 查看有哪些conda环境
conda info --envs

# 删除全部环境
conda remove -n env_name --all
# 删除指定环境
conda env remove -n env_name

# 重命名环境（将 --clone 后面的环境重命名成 -n 后面的名字）
conda create -n torch --clone py3      # 将 py3 重命名为 torch
```
注意：`--prefix/-p`不能与`--name/-n`同时使用！

## 1.4 下载库

```shell
# 从 conda-forge 渠道中提供的包安装
conda install -c conda-forge <package_name>

# 查询 conda-forge 中的包
conda search -c conda-forge <package_name>

# 安装指定版本的
conda install -c conda-forge <package_name>=<版本号>
```

### 1.4.1 安装GDAL库

```shell
#  安装 gdal 的依赖库 geos 和 proj
conda install geos proj
# 安装指定版本GDAL
conda install -c conda-forge gdal=3.2.1
```

## 1.5 迁移 conda 环境

将要迁移的环境打包

```shell
conda pack -n 虚拟环境名称 -o environment.tar.gz
```

如果报错：No command ‘conda pack’

```shell
# 尝试使用
conda install -c conda-forge conda-pack
```

复制压缩文件到新的电脑环境。进到conda的安装目录：/anaconda(或者miniconda)/envs/

```shell
# 对于 ubuntu 可以通过 whereis conda 查看 conda的安装路径
# cd 到 conda 的安装路径
mkdir environment

# 解压conda环境：
tar -xzvf environment.tar.gz -C  environment
```

# 二、pip 指令

## 2.1 使用临时镜像源下载库

```shell
pip install <package_name> -i <镜像源url>
```

# 三、镜像源

```shell
https://pypi.tuna.tsinghua.edu.cn/simple    # 清华
https://pypi.mirrors.ustc.edu.cn/simple        # 中科大
http://mirrors.aliyun.com/pypi/simple/        # 阿里云
http://pypi.douban.com/simple/            # 豆瓣
```
