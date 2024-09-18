---
title: 解决 Linux 下使用 matplotlib 中文乱码问题
date: 2024-09-14 15:01:53
tags:
    - "matplotlib"
    - "python"
    - "linux"
excerpt: false
categories: "Python技巧"
---


# 1.出现的问题

以下面这个代码为例：
```Python
import numpy as np
import matplotlib.pyplot as plt

x = np.arange(1, 12)
y = x ** 2 + 4
plt.title("Matplotlib demo")
plt.xlabel("时间(分钟)")
plt.ylabel("金额($)")
plt.plot(x,y)
plt.show()
```


在 Linux 下使用 Python 的`matplotlib`包默认是会出现乱码的：
[![乱码.png](https://s21.ax1x.com/2024/09/14/pAuEYVI.png)](https://imgse.com/i/pAuEYVI)

# 2.解决方法

## 2.1 下载字体

网上常用的中文字体是`SimHei`，[下载地址](https://github.com/StellarCN/scp_zh/blob/master/fonts/SimHei.ttf)

## 2.2 查找存放字体的文件夹

```Python
import matplotlib
 
print(matplotlib.matplotlib_fname())   # 查找字体路径
```
运行这段代码会打印出字体存放路径，如：
```txt
/home/sky/miniconda3/envs/test/lib/python3.12/site-packages/matplotlib/mpl-data/matplotlibrc
```
删掉`matplotlibrc`这部分后然后进入目录下：
```bash
cd /home/sky/miniconda3/envs/test/lib/python3.12/site-packages/matplotlib/mpl-data/
```

然后进入存放字体的目录：
```bash
cd fonts/ttf 
```

把下载好的字体文件`SimHei.ttf`复制到`ttf`目录下：
```bash
cp ~/下载/SimHei.ttf .  
```

然后回到`mpl-data`目录下，修改`matplotlibrc`文件：
```bash
nano matplotlibrc
```

找到`font.serif`，`font.sans-serif`所在位置，如下如所示。在冒号后面加入`SimHei`，保存退出：
[![修改的地方](https://s21.ax1x.com/2024/09/15/pAuJFz9.md.png)](https://imgse.com/i/pAuJFz9)

## 2.3 清理缓存

查找缓存位置：
```Python
import matplotlib    
print(matplotlib.get_cachedir())
```

把缓存文件删除即可：
```bash
rm ~/.cache/matplotlib -rf
```

# 3.解决示例

[![正常输出.png](https://s21.ax1x.com/2024/09/14/pAuERiV.png)](https://imgse.com/i/pAuERiV)


## 3.1 参考教程

- [【Deepin20系统】Linux系统中永久解决matplotlib画图中文乱码问题和使用seaborn中文乱码问题](https://developer.aliyun.com/article/1577567)
- [解决Python使用matplotlib绘图时出现的中文乱码问题](https://cloud.tencent.com/developer/article/1877673)